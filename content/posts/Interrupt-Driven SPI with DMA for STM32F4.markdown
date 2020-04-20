---
type: post
title:  "Interrupt Driven SPI using DMA on the STM32F4"
date:   2017-10-13 09:17:00 -0600
categories: Embedded Computing
---

I recently have been working on my driver library [airbourne](https://github/roslifght/airbourne_f4) for STM32F4-based flight control boards, like the openpilot [REVO](https://hobbyking.com/en_us/openpilot-cc3d-revolution-revo-32bit-flight-controller-w-integrated-433mhz-oplink.html).  I want a driver library that is licensed under BSD-3, and doesn't compromise modularity, or processor time.  This post is about my full duplex asynchronous SPI driver, which uses DMA under the hood.  As I put this together I realized that there could be more tutorials on how to set this up.  There are some great resources out there already, which I used to get my driver working such as [this post](https://javakys.wordpress.com/2014/09/04/how-to-implement-full-duplex-spi-communication-using-spi-dma-mode-on-stm32f2xx-or-stm32f4xx/) by Javakys.  But I thought I would add and explanation of my implementation to the world.

The most important sensor to a flight controller is the IMU.  In the case of the REVO, the IMU is the MPU600, attached to SPI1, so I'm going to assume that we are using SPI1, and the associated DMA Streams.  It wouldn't be too hard to do a find and replace to switch it to whichever SPI peripheral you need.  I also wanted to code this up as a C++ class.  That way I can use polymorphism, and other C++ niceties.  There are a few hacks that I had to use in order to properly link the C-based interrupt routines, but I think it's a pretty nice solution.  All files here are licensed under a BSD-3 license.

All of this has been added to my [airbourne](https://github/roslifght/airbourne_f4) library on github, as well as an example file which reads from the IMU and prints the measurements over the USB cord.

Let's start with the Header File:

```C++
#pragma once

#include "revo_f4.h"  // includes the standard library, and the time functions millis() and micros()

#include "gpio.h" // my GPIO class abstraction

class SPI
{

public:
  SPI(SPI_TypeDef *SPI);

  void set_divisor(uint16_t new_divisor); // Set the SPI speed
  void enable(); // set the select pin high
  void disable(); // set the select pin low
  bool transfer(uint8_t *out_data, uint8_t num_bytes, uint8_t* in_data); // send and receive bulk data
  uint8_t transfer_byte(uint8_t data); // send and receive a single byte
  void transfer_complete_cb(); // function which is called at the completion of a SPI bulk transfer
  void register_complete_cb(void (*cb)(void)); // a method to set a user-defined callback for SPI bulk transfer complete

private:
  bool busy_ = false;
  SPI_TypeDef*	dev;
  GPIO mosi_;
  GPIO miso_;
  GPIO sck_;
  GPIO nss_;

  DMA_InitTypeDef DMA_InitStructure_;

  void (*transfer_cb_)(void) = NULL;
};
```

The header file isn't too complicated, but it just defines the abstraction.  Let's look at each function implementation.

## Constructor
```C++
SPI* SPIptr; // This is a global pointer, which will allow us to link up the C-based IRQs, defined in the standard library, to our class implementation

SPI::SPI(SPI_TypeDef *SPI) {

  GPIO_InitTypeDef gpio_init_struct;
  SPI_InitTypeDef  spi_init_struct;

  if (SPI == SPI1)
  {
    // Connect the Global pointer to this object
    SPIptr = this;
    // Configure the Select Pin
    nss_.init(GPIOA, GPIO_Pin_4, GPIO::OUTPUT);

    disable();

    // Set the AF configuration for the other pins
    GPIO_PinAFConfig(GPIOA, GPIO_PinSource5, GPIO_AF_SPI1);
    GPIO_PinAFConfig(GPIOA, GPIO_PinSource6, GPIO_AF_SPI1);
    GPIO_PinAFConfig(GPIOA, GPIO_PinSource7, GPIO_AF_SPI1);

    // Initialize other pins
    sck_.init(GPIOA, GPIO_Pin_5, GPIO::PERIPH_OUT);
    miso_.init(GPIOA, GPIO_Pin_6, GPIO::PERIPH_OUT);
    mosi_.init(GPIOA, GPIO_Pin_7, GPIO::PERIPH_OUT);

    dev = SPI1;

    // Set up the SPI peripheral
    SPI_I2S_DeInit(dev);
    spi_init_struct.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
    spi_init_struct.SPI_Mode = SPI_Mode_Master;
    spi_init_struct.SPI_DataSize = SPI_DataSize_8b;
    spi_init_struct.SPI_CPOL = SPI_CPOL_High;
    spi_init_struct.SPI_CPHA = SPI_CPHA_2Edge;
    spi_init_struct.SPI_NSS = SPI_NSS_Soft;
    spi_init_struct.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_64;  // 42/64 = 0.65625 MHz SPI Clock
    spi_init_struct.SPI_FirstBit = SPI_FirstBit_MSB;
    spi_init_struct.SPI_CRCPolynomial = 7;
    SPI_Init(dev, &spi_init_struct);
    SPI_CalculateCRC(dev, DISABLE);
    SPI_Cmd(dev, ENABLE);

    // Wait for any transfers to clear (this should be really short if at all)
    while (SPI_I2S_GetFlagStatus(dev, SPI_I2S_FLAG_TXE) == RESET);
    SPI_I2S_ReceiveData(dev); //dummy read if needed

    // Configure the DMA
    DMA_InitStructure_.DMA_FIFOMode = DMA_FIFOMode_Disable ;
    DMA_InitStructure_.DMA_FIFOThreshold = DMA_FIFOThreshold_1QuarterFull ;
    DMA_InitStructure_.DMA_MemoryBurst = DMA_MemoryBurst_Single ;
    DMA_InitStructure_.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;
    DMA_InitStructure_.DMA_MemoryInc = DMA_MemoryInc_Enable;
    DMA_InitStructure_.DMA_Mode = DMA_Mode_Normal;
    DMA_InitStructure_.DMA_PeripheralBurst = DMA_PeripheralBurst_Single;
    DMA_InitStructure_.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;
    DMA_InitStructure_.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;

    DMA_InitStructure_.DMA_PeripheralBaseAddr = (uint32_t)(&(SPI1->DR));
    DMA_InitStructure_.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
    DMA_InitStructure_.DMA_Priority = DMA_Priority_High;

    NVIC_InitTypeDef NVIC_InitStruct;
    NVIC_InitStruct.NVIC_IRQChannel = DMA2_Stream3_IRQn;
    NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 0x02;
    NVIC_InitStruct.NVIC_IRQChannelSubPriority = 0x02;
    NVIC_Init(&NVIC_InitStruct);
  }
}

```

Now, we have set up the SPI device, and we have a class object which encapsulates all of this information.  Now, we often need to change the speed of the SPI on the fly, the MPU6000 wants us to configure the device at a lower speed than we ultimately read from it, so we need the following function

## set_divisor

```C++
void SPI::set_divisor(uint16_t new_divisor) {
  SPI_Cmd(dev, DISABLE);

  const uint16_t clearBRP = 0xFFC7;

  uint16_t temp = dev->CR1;

  switch(new_divisor) {
  case 2:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_2;
    break;
  case 4:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_4;
    break;
  case 8:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_8;
    break;
  case 16:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_16;
    break;
  case 32:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_32;
    break;
  case 64:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_64;
    break;
  case 128:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_128;
    break;
  case 256:
    temp &= clearBRP;
    temp |= SPI_BaudRatePrescaler_256;
    break;
  }
  dev->CR1 = temp;

  SPI_Cmd(dev, ENABLE);
}

```

## Enable and Disable

Now, we need the ability to enable and disable the device, this is done by setting the chip select pin (or `NSS`) pin high or low.  This is using my `GPIO` class abstraction, but it's pretty obvious what is going on if you want to use the standard library directly.

```C++
void SPI::enable() {
  nss_.write(GPIO::HIGH);
}

void SPI::disable() {
  nss_.write(GPIO::LOW);
}
```

## Transfer byte

When we are configurinig devices, we typically want to send one byte at a time, and wait for the response, so we know what to do to keep going.  Therefore, the `transfer_byte` method is blocking, and returns the read byte.   That way, configuration can take place just one line after another, one byte at a time.  Hopefully, we only configure the device once, and use asynchronous methods for the bulk data read.  It also can detect a failure to read or write from the device, so we don't try to perform bulk reads where they will fail.

```C++
uint8_t SPI::transfer_byte(uint8_t data)
{
  uint8_t byte = data;
  uint16_t spiTimeout;

  spiTimeout = 0x1000;

  while (SPI_I2S_GetFlagStatus(dev, SPI_I2S_FLAG_TXE) == RESET)
  {
    if ((spiTimeout--) == 0)
      return false;
  }

  SPI_I2S_SendData(dev, data);

  spiTimeout = 0x1000;

  while (SPI_I2S_GetFlagStatus(dev, SPI_I2S_FLAG_RXNE) == RESET)
  {
    if ((spiTimeout--) == 0)
      return false;
  }
  // Pack received data into the same array
  data = (uint8_t)SPI_I2S_ReceiveData(dev);

  return byte;
}
```

## Bulk transfer

Now, we're really getting into the meat of the driver.  This is the bulk transfer.  We're going to use DMA to fire off a big data transfer, behind the scenes.  If we have our interrupts set up and the callbacks registered, then this function will exit very quickly, and when the transfer is done, the callbacks will get called.

```C++
bool SPI::transfer(uint8_t* out_data, uint8_t num_bytes, uint8_t* in_data)
{
  busy_ = true;

  // Configure the DMA
  DMA_DeInit(DMA2_Stream3); //SPI1_TX_DMA_STREAM
  DMA_DeInit(DMA2_Stream2); //SPI1_RX_DMA_STREAM

  DMA_InitStructure_.DMA_BufferSize = (uint16_t)(num_bytes);

  /* Configure Tx DMA */
  DMA_InitStructure_.DMA_Channel = DMA_Channel_3;
  DMA_InitStructure_.DMA_DIR = DMA_DIR_MemoryToPeripheral;
  DMA_InitStructure_.DMA_Memory0BaseAddr = (uint32_t) out_data;
  DMA_Init(DMA2_Stream3, &DMA_InitStructure_);

  /* Configure Rx DMA */
  DMA_InitStructure_.DMA_Channel = DMA_Channel_3;
  DMA_InitStructure_.DMA_DIR = DMA_DIR_PeripheralToMemory;
  DMA_InitStructure_.DMA_Memory0BaseAddr = (uint32_t) in_data;
  DMA_Init(DMA2_Stream2, &DMA_InitStructure_);

  //  Configure the Interrupt
  DMA_ITConfig(DMA2_Stream3, DMA_IT_TC, ENABLE);

  enable();

  DMA_Cmd(DMA2_Stream3, ENABLE); /* Enable the DMA SPI TX Stream */
  DMA_Cmd(DMA2_Stream2, ENABLE); /* Enable the DMA SPI RX Stream */

  /* Enable the SPI Rx/Tx DMA request */
  SPI_I2S_DMACmd(SPI1, SPI_I2S_DMAReq_Rx, ENABLE);
  SPI_I2S_DMACmd(SPI1, SPI_I2S_DMAReq_Tx, ENABLE);

  SPI_Cmd(SPI1, ENABLE);
}
```

## Transfer Complete callbacks

So, we are going to use a function pointer to register a user-defined callback for when the transfer is complete.  This will be called when the DMA has completed the data transfer.  There are also a number of things we need to clean up when the data is done, so we'll take care of that here

```C++
void SPI::register_complete_cb(void (*cb)())
{
  transfer_cb_ = cb;
}


void SPI::transfer_complete_cb()
{
  disable();
  DMA_ClearFlag(DMA2_Stream3, DMA_FLAG_TCIF3);
  DMA_ClearFlag(DMA2_Stream2, DMA_FLAG_TCIF2);

  DMA_Cmd(DMA2_Stream3, DISABLE);
  DMA_Cmd(DMA2_Stream2, DISABLE);

  SPI_I2S_DMACmd(SPI1, SPI_I2S_DMAReq_Rx, DISABLE);
  SPI_I2S_DMACmd(SPI1, SPI_I2S_DMAReq_Tx, DISABLE);

  SPI_Cmd(SPI1, DISABLE);

  busy_ = false;
  if (transfer_cb_)
    transfer_cb_();
}
```

## C-Based IRQ Handler
This function is defined in the standard peripheral library, so we have to link against it with the `extern "C"` key.  However, because of our global pointer, we can direct the callback into our class.

```C++

extern "C"
{

void DMA2_Stream3_IRQHandler()
{
  if (DMA_GetITStatus(DMA2_Stream3, DMA_IT_TCIF3))
  {
    DMA_ClearITPendingBit(DMA2_Stream3, DMA_IT_TCIF3);
    SPIptr->transfer_complete_cb();
  }
}

}
```

## Putting it all together

So, here are some excerpts from my `MPU6000` class, which includes a pointer to a `SPI` object as a member.  The `MPU6000` class is a little bit complicated by the fact that there is an external interrupt pin which goes high every time the IMU has new data to read.  I am actually firing off a bulk read from within this interrupt, that way I always have the most up-to-date data.  Setting up that interrupts is a little bit beyond the scope of this post, so if you're interested, go ahead and check it out in the [airbourne](https://github.com/rosflight/airbourne_f4) library.  In the meantime, I'm just going to copy enough of the class methods below to see how the SPI driver works.

```C++
MPU6000* IMU_Ptr;
void data_transfer_cb(void)
{
  IMU_Ptr->data_transfer_callback();
}

MPU6000::MPU6000(SPI* spi_drv)
{
  IMU_Ptr = this;
  spi = spi_drv;

  write(MPU_RA_PWR_MGMT_1, MPU_BIT_H_RESET);
  delay(150);

  write(MPU_RA_PWR_MGMT_1, MPU_CLK_SEL_PLLGYROZ);
  write(MPU_RA_USER_CTRL, MPU_BIT_I2C_IF_DIS);
  write(MPU_RA_PWR_MGMT_2, 0x00);
  write(MPU_RA_SMPLRT_DIV, 0x00);
  write(MPU_RA_CONFIG, MPU_BITS_DLPF_CFG_98HZ);
  write(MPU_RA_ACCEL_CONFIG, MPU_BITS_FS_4G);
  write(MPU_RA_GYRO_CONFIG, MPU_BITS_FS_2000DPS);
  write(MPU_RA_INT_ENABLE, 0x01);

  spi->set_divisor(2); // 21 MHz SPI clock (within 20 +/- 10%)

  // set the accel and gyro scale parameters
  accel_scale_ = (4.0 * 9.80665f) / ((float)0x7FFF);
  gyro_scale_= (2000.0 * 3.14159f/180.0f) / ((float)0x7FFF);

  spi->register_complete_cb(&data_transfer_cb);
}

void MPU6000::write(uint8_t reg, uint8_t data)
{
  spi->enable();
  spi->transfer_byte(reg);
  spi->transfer_byte(data);
  spi->disable();
}

void MPU6000::data_transfer_callback()
{
  new_data_ = true;
  acc_[0] = (float)((int16_t)((raw[1] << 8) | raw[2])) * accel_scale_;
  acc_[1] = (float)((int16_t)((raw[3] << 8) | raw[4])) * accel_scale_;
  acc_[2] = (float)((int16_t)((raw[5] << 8) | raw[6])) * accel_scale_;

  temp_  = (float)((int16_t)((raw[7] << 8) | raw[8])) / 340.0f * 36.53f;

  gyro_[0]  = (float)((int16_t)((raw[9]  << 8) | raw[10])) * gyro_scale_;
  gyro_[1]  = (float)((int16_t)((raw[11] << 8) | raw[12])) * gyro_scale_;
  gyro_[2]  = (float)((int16_t)((raw[13] << 8) | raw[14])) * gyro_scale_;
}

void MPU6000::read(float (&accel_data)[3], float (&gyro_data)[3], float* temp_data)
{
  accel_data[0] = acc_[0];
  accel_data[1] = acc_[1];
  accel_data[2] = acc_[2];
  gyro_data[0] = gyro_[0];
  gyro_data[1] = gyro_[1];
  gyro_data[2] = gyro_[2];
  *temp_data = temp_;
}

void MPU6000::exti_cb()
{
  imu_timestamp_ = micros();
  raw[0] = MPU_RA_ACCEL_XOUT_H | 0x80;
  spi->transfer(raw, 15, raw);
}

extern "C"
{

void EXTI4_IRQHandler(void)
{
  EXTI_ClearITPendingBit(EXTI_Line4);
  IMU_Ptr->exti_cb();
}

}

```
