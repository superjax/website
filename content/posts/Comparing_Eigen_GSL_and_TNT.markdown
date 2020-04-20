---
type: post
title:  "Comparing Eigen, GSL and TNT"
date:   2017-10-20 09:17:00 -0600
categories: Scientific Computing
---

For a project I'm working on, I need to reduce memory usage on an embedded processor which is currently using Eigen for a non-symmetric eigenvalue decomposition.  As all you guys know, that is finding $\lambda$ and $\mathbf{x}$ such that for an arbitrary square matrix $A$

$$ A\mathbf{x} = \lambda\mathbf{x} $$

There are a number of generic algorithms for solving this problem if $A$ is symmetric.  This is an easier case becasue all the eigenvalues will be real. However, the non-symmetric case can be a lot more complicated.

I poked around the net for a while and ran into two possible alternative libraries.

 1. The [Template Numerical Toolkit (TNT)](http://math.nist.gov/tnt/) is the successor the LAPACK++,Sparselib++, and IML++ packages.  This seemed like a pretty strong option, given the success of it's predecessors, it's ''header-only-ness'' and relatively small source-code size.
 2. The [GNU Scientific Library (GSL)](https://www.gnu.org/software/gsl/) seemed to be the most ''bare-metal'', with the trade-off of being the least usable.  Because of this, I suspected that GSL would be the fastest and least memory-intensive of the bunch.

# Symmetric Eigenvalue Decomposition

To test these three libraries, I wrote a C++ program to run the three in parallel and time their execution time of solving the eigenvalue decomposition problem.  For each matrix size, I ran 25 tests and plotted the average of the time to perform the operation.  I used the ''Pei Matrix'' from [A test matrix for inversion procedures (1962)](https://dl.acm.org/citation.cfm?id=368975).  This matrix is created as a matrix of all ones, plus an identity matrix multiplied by some constant.

$$
A =
\begin{bmatrix}
  1 & 1 & \cdots & 1 \\\\
  1 & 1 & \cdots & 1 \\\\
  \vdots & \vdots & \ddots & \vdots \\\\
  1 & 1 & \cdots & 1 \\\\
\end{bmatrix} +
\begin{bmatrix}
    \alpha & 0 & \cdots & 0 \\\\
    0 & \alpha & \cdots & 0 \\\\
    \vdots & \vdots & \ddots & \vdots \\\\
    0 & 0 & \cdots & \alpha \\\\
\end{bmatrix}
$$

I ran this matrix at all sizes $5 < N < 200$ and got the following results.  (Note that the results are plotted on a log scale.)  The test source code can be found on my [github](https://github.com/superjax/matrix_comparison)
<style>
  .center {
	display: block;
	margin-left: auto;
	margin-right: auto;
	text-align: center;
	width: 50%;
  }

.side_by_side {
	display: block; /* shrink wrap the contents */
	margin: 0 auto; /* center via left/right margins */
	text-align: center;
	width: 100%;
  }
</style>

<div class="side_by_side">
	<div class="column">
    <img  src="/matrix_compare/symmetric_eval_decomp_compare.png" style="width: 30vw; min-width: 120px;">
    <img  src="/matrix_compare/symmetric_eval_decomp_compare_small.png" style="width: 30vw; min-width: 120px;">
	</div>
</div>

I found these results very surprising, and they seem to agree with [Erwin Kalvelagen](http://yetanothermathprogrammingconsultant.blogspot.com/2009/06/gsl-vs-lapack-performance.html) where he showed LAPACK severerly outperforming GSL in LU decomposition.

Another thing I found surprsing was the blips both TNT and GSL experienced at matrices size 128 and 192.  These numbers seem a little bit suspicious to be mere coincidence. At any rate, GSL takes about twice as long as TNT and Eigen to perform symmetric eigenvalue decomposition.  For small matrices, TNT is the clear winner, but it seems that Eigen starts to outperform TNT at larger matrices.

# Asymmetric Eigenvalue Decomposition

I performed the same test as before, but with an asymmetric $A$ matrix.  I generated $A$ with the following C++ code

```C++
Eigen::MatrixXd mat(size, size);
mat.setRandom();
mat += Eigen::MatrixXd::Identity(size, size);
```

It's sorta like the Pei matrix, but with a random twist.  At any rate, this matrix should most certainly be non-symmetric and mostly-postive (at least for small $N$).  This time, I created 25 random matrices for each size, and averaged the amount of time it took to solve for the eigenvalues and eigenvectors.

<div class="side_by_side">
	<div class="column">
    <img  src="/matrix_compare/asymmetric_eval_decomp_compare.png" style="width: 30vw; min-width: 120px;">
    <img  src="/matrix_compare/asymmetric_eval_decomp_compare_small.png" style="width: 30vw; min-width: 120px;">
	</div>
</div>

In this case, it appears that Eigen again beats out TNT, and both totally smash GSL.  Surprising.
