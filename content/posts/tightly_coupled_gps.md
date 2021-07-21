---
type: post
title:  "Tightly-Coupled GPS State Estimation and RTK/Compassing"
date:   2018-11-20 12:05:00 -0600
categories: Scientific Computing
tags: ["GNSS", "GPS", "Kalman Filter", "RTK", "EKF"]
---

$$
\def\v{\mathbf{v}}
\def\u{\mathbf{u}}
\def\a{\mathbf{a}}
\def\b{\mathbf{b}}
\def\c{\mathbf{c}}
\def\w{\boldsymbol{\omega}}
\def\p{\mathbf{p}}
\def\t{\mathbf{t}}
\def\i{\mathbf{i}}
\def\j{\mathbf{j}}
\def\e{\mathbf{e}}
\def\k{\mathbf{k}}
\def\r{\mathbf{r}}
\def\d{\mathbf{d}}
\def\x{\mathbf{x}}
\def\y{\mathbf{y}}
\def\q{\mathbf{q}}
\def\qq{\Gamma}
\def\qr{\q_{r}}
\def\qd{\q_{d}}
\def\SO{\mathit{SO}}
\def\SE{\mathit{SE}}
\def\skew#1{\left\lfloor #1\right\rfloor _{\times}}
\def\norm#1{\left\Vert #1\right\Vert }
\def\grey#1{\textcolor{gray}{#1}}
\def\abs#1{\left|#1\right|}
\def\S{\mathcal{S}}
\def\dd{\boldsymbol{\delta}}
\def\Ad{\textrm{Ad}}
\def\SU{\mathit{SU}}
\def\R{\mathbb{R}}
\def\so{\mathfrak{so}}
\def\se{\mathfrak{se}}
\def\su{\mathfrak{su}}
$$

<style>
  .center {
	display: block;
	margin-left: auto;
	margin-right: auto;
	text-align: center;
	width: 80%;
  }

.side_by_side {
	display: block; /* shrink wrap the contents */
	margin: 0 auto; /* center via left/right margins */
	text-align: center;
	width: 100%;
  }
</style>

# Introduction

GPS (or, the more generic term, GNSS) is an amazing technology that has enabled huge advancements in autonomous technology.  However, most implementations of GPS I've seen on autonomous systems have been the garden variety "Buy a UBLOX chip and use it's sensor output".

While standard GPS receivers like the Ublox M8Q are inexpensive and easy to use, they black-box a lot of sensor fusion that can be greatly enhanced with additional sensors. In fact, I would argue that using a standard ublox receiver in an autonomous system is akin to using an IMU which only reported its estimated orientation, as opposed to acceleration and angular rate. In both cases, we are able to make much better use of raw sensor readings by fusing them with other sensors and leveraging knowledge of our particular system.  It just takes more work.

This document is designed to introduce “tightly coupled” GPS state estimation and RTK/GPS compassing in the context of autonomous vehicles. The purpose is to bring others with a firm understanding of Kalman Filters and state estimation theory up to speed, so to speak, on the relevant vocabulary and methods involved.

While I hope my explanation is complete enough to allow for full implementation of these algorithms, my intention is really to help others develop intuition about how these algorithms work, and make them less "black boxed".  Knowing what is going on in the guts of these systems can help autonomus systems designers better leverage the strengths and weaknesses of GNSS to build robust and effective systems.

# GNSS Basics

While GPS is probably the most common term for global satellite navigation, it is actually a slight misnomer. It's like calling adhesive bandages “band-aids.” The generic term is technically Global Navigation Satellite Systems (GNSS). There are several GNSS constellations and they are supported by different governments around the world. Each system is designed to provide robust positioning by itself, so they are all fully populated with enough satellites to service the entire world.  (Usually means that in any given location at any time you can see at least 6 of each constellation).

| Name    | Government    | Number of Satelites | Orbit Period       | # of Frequencies |
| ------- | ------------- | ------------------- | ------------------ | ---------------- |
| GPS     | United States | 30                  | $\approx$ 12 hours | 3                |
| Glonass | Russia        | 24                  | $\approx$ 12 hours | 3                |
| Galileo | EU            | 22                  | $\approx$ 14 hours | 3                |
| Beidou  | China         | 35                  | $\approx$ 13 hours | 3                |

In addition to the standard satellites, each of these countries have set up a few geostationary SBAS satellites that can be used in addition to the standard constellation. If we use a receiver that can receive signals from all of these constellations, we can usually see quite a few satellites at once. For example, [Figure 1](#fig:notation) shows all the navigation satellites visible from my house on an average Saturday afternoon.

<a name="fig:notation"></a>
<div class="center">
<img  src="/gnss_intro/visible_sats.png" style="width: 30vw; min-width: 30rem;">
<figcaption class="center">Figure 1: Satellites above my house on a Saturday afternoon</figcaption>
</div>

As you might imagine, there is a huge advantage to multi-constellation receivers, such as the F9-series chips from UBLOX.  These are a relatively new product offering (post 2018), but instead of being limited to five or six satellites of a single constellation, we can dozens.


# Signal Types

## P and C/A Codes
There are two main GNSS signals.  The first is the " P code".  Each satellite is equipped with a unique pseudo-random number generator with a known pattern.  This number (in binary form) is broadcast at a fixed rate from each satellite.  In the case of GPS, this number is broadcast at 10.23 million bits per second, and will repeat every 37 weeks.

Since we know the key, and a period for which we know that the signal is unique, we can convolve an estimated "chunk" of the signal against what we actually received.  If we match this signal, then we know when message was broadcast.  One of the main specs of a GNSS reciever is the number of simultaneous convolutions it can perform, and how fast these convolutions happen, since this will directly determine how long the receiver will take to "lock on" to the signal, and how many signals it can track simultaneously.

As you might imagine, trying to match a unique chunk of the $2^{14}$-bit P code can be hard.  So there is actually another signal overlayed on the P code called the C/A (or coarse acquisition) code.  This is a much lower-resolution signal sent at a slower rate.  The C/A code is used to "warm start" the receiver in its search for a lock in the P code.

<a name="fig:convolve_codes"></a>
<div class="center">
<img  src="/gnss_intro/pseudorange_time.svg" style="width: 30vw; min-width: 30rem;">
<figcaption class="center">Figure 2: Illustration of how the C/A and P code works</figcaption>
</div>

The P-code is actually encrypted for use by the military.  Civilian users are not authorized to use this code, so we use the C/A code alone.  Bummer, right?   With a modern receiver, Single-frequency C/A code-only users can expect absolute accurace between 1 and 3 meters, while P code users can expect 0.5-2m


## Navigation (Ephemeris)
Satellites also broadcast a navigation message (also known as ephemeris) to let us know where they are in the sky.  The satellites are continuously performing SLAM amongst themselves in space along with several ground stations on the earth, so they have a pretty good estimate of their position and velocity, and they publish this estimate every few minutes.  There are not a lot of noise sources in space, either, so these estimates are good for quite prolonged periods (up to 2 hours).

Most constellations provide us Keplerian Parameters.  These describe the shape and orientation of the elliptical orbit with respect to the equator, north pole, and prime meridian and the position of the satellite in the orbit, as well as its speed.  The exception is GLONASS, which provides us instead parameters to be used in a differential equation.

The ephemeris message is broadcast in a highly-compressed serialized format with significant error-checking mechanisms in place.  The format for this message (as well any official details for any of the constellations) are given in the "International Control Documents" (ICDs).  These are periodically updated as different bugs are discovered or functionality unlocked.

- [GPS ICD](https://www.gps.gov/technical/icwg/)
- [GLONASS ICD](http://russianspacesystems.ru/wp-content/uploads/2016/08/ICD-GLONASS-CDMA-General.-Edition-1.0-2016.pdf)
- [Galileo ICD](https://www.gsc-europa.eu/sites/default/files/sites/all/files/Galileo-OS-SIS-ICD.pdf)
- [Beidou ICD](http://en.beidou.gov.cn/SYSTEMS/ICD/)


# Signal Models

## Pseudorange

The psuedorange is the main workhorse of GNSS positioning.  The pseudorange is the result of locking onto the C/A-code and measuring $\tau$ in [Figure 2](#fig:convolve_codes).  However, because of the extreme distances involved in this measurement, there are two main effects that we need to take into account.

The first, and most significant, are atmospheric effects.  As light enters the atmosphere, it is diffracted by the ionosphere and troposphere.

<a name="fig:delays"></a>
<div class="center">
<img  src="/gnss_intro/delays.png" style="width: 30vw; min-width: 30rem;">
<figcaption class="center">Figure 3: Illustration atmospheric delays</figcaption>
</div>

A lot of work has been put into modeling these effects.  A good example of Ionosphere compensation is the [Klobuchar Algorithm](https://gssc.esa.int/navipedia/index.php/Klobuchar_Ionospheric_Model) and the [Saastamoinen Model](https://gssc.esa.int/navipedia/index.php/Galileo_Tropospheric_Correction_Model) is a commonly-used troposphere diffraction model.

Even with these models, however, errors in the psuedorange measurement are dominated by unmodeled atmospheric errors. Therefore a popular technique to mitigate these effects is to use multiple frequencies. Since diffraction is proportional to the frequency of light, we can compute the difference in the observed psuedoranges between the two frequencies and come up with much better models of our atmospheric error.  There are other advantages to multi-frequency GNSS, such as improved multipath mitigation, but those are out of the scope of this post.

To support this use-case, GNSS satellites simultaneously broadcast multiple codes on multple frequency bands.  For example, the GPS L1 and L2 bands are given below (there are other frequencies of GPS that are not open for civilian use).

| Name | Frequency   |
| ---- | ----------- |
| L1   | 1.56542 GHz |
| L2   | 1.2276 GHz  |

The second effect we need to consider is relative motion of the earth with respect to the satellite during the transmission of the signal.  Even at the speed of light, the signal will take about 70 milliseconds to reach the earth.  During this time, both the satellite and the receiver will move several meters.  This is also an example of the Sagnac effect.

<a name="fig:delays"></a>
<div class="center">
<img  src="/gnss_intro/earth-rotation.png" style="width: 30vw; min-width: 30rem;">
<figcaption class="center">Figure 4: Earth and Satellite motion (Sagnac) compensation</figcaption>
</div>

When we put this all together, we get the following model for a pseudorange measurement:

$$
\begin{equation}
\hat{\rho} \_{s} = \norm{\p_{s/e}^{e} \left(t_s \right)-\p_{b/e}^{e} \left(t_r \right)}+\frac{1}{c}\w_{e}^{\top}
\skew{\p_{s/e}^{e} \left(t_s \right)}\p_{b/e}^{e} \left(t_r \right)
+\delta^{atm} \left(\p_{s/e}^{e},\p_{b/e}^{e},t_r \right)+c \left(\tau_{b}-\tau_{s} \right)+\xi_{\rho}
\end{equation}
$$

where:
 - $\p_{s/e}^{e}$: Position of the satellite $s$ with respect to ECEF $e$, expressed in ECEF $e$.
 - $\p_{b/e}^{e}$: Position of the receiver $b$ with respect to ECEF $e$, expressed in ECEF $e$.
 - $t_s$: is the time the satellite sent the message.
 - $t_r$: is the time of reception.
 - $\tau_{b},\tau_{s}$: the clock bias of the receiver and satellite, respectively.
 - $\delta^{atm}\left(\p_{s/e}^{e},\p_{b/e}^{e},t_r\right)$: given by the Klobuchar Algorithm and Saastamoinen Model.
 - $\w_{e}$: the rotation of the earth.

## Doppler

As part of a modern GNSS receiver's P-code tracking loop, it also computes the doppler shift of the signal with respect to the known nominal frequency.  This is a very important observable, since it directly measures the relative velocity of the satellite with respect to the receiver along the line-of-sight vector.

The doppler signal is _not_ affected by atmospheric errors, since these errors are quasi-static.  Therefore, the doppler measurement can often be much more accurate than a pseudorange.

We model doppler as the derivative of the psuedorange, so it is given as

$$
\begin{align}
\dot{\hat{\rho}} \_{s} =&\left(\v_{s/e}^{e}\left(t\right)-\v_{b/e}^{e}\left(t\right)\right)^\top
\e_{s/b}^e+c\left( \dot{\tau_{b}} -\dot{\tau}_{s}\right) + \frac{1}{c}\w_e^\top \left(\skew{\v_{s/e}^e}\p_{b/e}^e + \skew{\p_{s/e}^e}\v_{b/e}^e\right) +\xi_{ \dot{\rho} }
\end{align}
$$

where:
 - $\v_{s/e}^{e}$: Velocity of the satellite $s$ with respect to ECEF $e$, expressed in ECEF $e$.
 - $\v_{b/e}^{e}$: Velocity of the receiver $b$ with respect to ECEF $e$, expressed in ECEF $e$.
 - $\dot{\tau_{b}}, \dot{\tau}_{s}$: recevier and satellite clock bias drift rate, respectively.


## Carrier Phase

The final measurement that we get from a GNSS receiver is the carrier phase angle.  Remember the C/A code?  Well, that code is carried by a radio wave with a specific frequency.  The carrier phase is the phase angle of this wave at a particular sample time. This makes the carrier phase signal an incredibly accurate ranging signal, accurate to sub-centimeter accuracy.

The carrier phase signal comes with a catch, though.  Look at Figure 
