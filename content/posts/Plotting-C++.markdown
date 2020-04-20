---
type: post
title:  "Plotting C++ Code"
date:   2018-11-20 12:05:00 -0600
categories: Scientific Computing
---

I do a lot of scientific computing in C++, and often have a need for plotting the internals of my algorithms.  I love the Eigen Matrix Library, but directly plotting the contents of an Eigen Matrix can be difficult.  I've narrowed in on a workflow that I really like and want to share.

One example might be if I have a Matrix `X` which contains in each column the estimated state of the quadrotor at each time sample. with

$$
\mathbf{x} = \begin{bmatrix} p_{b/I}^I & q_b^I & v_{b/I}^I & \omega_{b/I}^I \end{bmatrix}^\top
$$

and another matrix `t` which contains the time of each sample.

I've found it's super easy to dump these matrices to file, then read them using MATLAB or python.  Here is the C++ code snippet I use to dump the matrices.

```C++
#include <fstream>

...

void log(const Eigen::MatrixXd& X, const Eigen::MatrixXd& t)
{
  std::ofstream xfile("state.bin");
  std::ofstream tfile("time.bin");
  xfile.write((char*)X.data(), sizeof(double)*X.rows()*X.cols());
  tfile.write((char*)t.data(), sizeof(double)*t.rows()*t.cols());
  xfile.close();
  tfile.close();
}
```

Reading and plotting them in MATLAB or python is also super easy!  Here is a code snippet to quickly load the data and plot the position states

### Python code
```py
import numpy as np
import matplotlib.pyplot as plt

est = np.reshape(np.fromfile('state.bin', dtype=np.float64), (-1,13))
time = np.fromfile('time.bin', dtype=np.float64)

figure(1); clf;
for i in range(3):
  subplot(3, 1, i+1)
  plot(time, est(i, :),)
```

### MATLAB code
``` matlab
%% read binary files
est_file = fopen(strcat(directory, 'state.bin'), 'r');
time_file = fopen(strcat(directory, 'time.bin'), 'r');

est = reshape(fread(est_file, 'double'), 13, []);
time = fread(est_file, 'double');

%% Plot position
figure(1); clf;
for i = 1:3
  subplot(3, 1, i)
  plot(time, est(i, :),)
end
```

# Realtime data
Another use I have for this workflow is in realtime applications, where I might be generate a sample each time a function runs.  It's super nice to be able to quickly log this data to a file and later view it.  The process for this is similar, but I use class variables to hold an open file while the program is running and close the log when the program exists.

```C++
#include <fstream>

class Logger
{
  ...
  std::ofstream xfile_;
  std::ofstream tfile_;
  ...
};

// open the logs (call during initialization)
void Logger::startlog(std::string)
{
  xfile_.open("state.bin");
  tfile_.open("time.bin");
}

// closes logs when Logger goes out of scope
void Logger::~Logger()
{
  xfile_.close();
  tfile_.close();
}

// Log a single sample (call from the main loop)
void Logger::log(const Eigen:VectorXd& X, double t)
{
  xfile.write((char*)X.data(), sizeof(double)*X.rows());
  tfile.write((char*)&t sizeof(double));
}
```

You can use the same python or MATLAB code to process this data.
