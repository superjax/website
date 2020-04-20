---
type: post
title:  "Part 2: Matrix Exponential"
date:   2020-04-18 13:09:00 -0600
categories: math
tags: ["vectors", "notation", "Geometry"]
---

This is the second of a 8-part series of posts designed as a quick-start guide for students new
to the field of robotics and estimation, specifically on the use of
Lie groups to describe rotations and rigid body transformations. This is a web port of the full pdf document, which is hosted [here](https://drive.google.com/open?id=1T93xYi7iKqrW7a9KCJvoBB9no4j7zaDu).


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
</style>

# A Look at the Matrix Exponential

The matrix exponential is central to the rest of this document. In
this section, we are going to take a deeper look at the matrix exponential
to gain some intuition about what it's doing. If we can understand
what the exponential really means, then the reason we use it so often
in Lie Group theory will become obvious.

Let's start with the scalar, first-order differential equation

$$
\begin{equation}
\frac{\partial}{\partial t}x\left(t\right)=ax\left(t\right).\label{eq:first-order-diff_eq}
\end{equation}
$$

This equation has the simple solution,

$$
x\left(t\right)=x\left(t_{0}\right)e^{at},
$$
the derivation of which is a central part of any undergraduate differential
equations class. However, let us for a moment pretend that we don't
know about the natural number $e$, but we were tasked with solving
Eq. \ref{eq:first-order-diff_eq}. To make things more concrete, we
could stand in the shoes of Jacob Bernoulli as he studies the following
problem:

> An account starts with \\\\$1.00 and pays 100 percent interest per year. If the interest is credited once, at the end of the year, the value of the account at year-end will be \\\\$2.00. What happens if the interest is computed and credited more frequently during the year?

This problem is formulated as

$$
\$1.00\times\left(1+\frac{1}{n}\right)^{n},
$$

where $n$ is the number of intervals. Jacob Bernoulli noticed that
this series converges to what we now know as $e$, the natural number,
as $n\to\infty$. So what is going on here? The main factor at play
is that the amount of interest that the banker pays to the account
depends on how much money is *already in* the account. It just
so happens that this form of problem, where the derivative of some
state is directly proportional to its current value has this really
nice form that depends on the limit to this infinite series.

To bring this concept to a robotic application, let's consider two-dimensional
transforms. Let's say we have a unicycle robot [^1]  with inputs linear velocity $v$ and angular velocity $\omega$.
The dynamics of this system are given as

[^1]: This is a standard model in robotics for studying non-holonomic controls and consists of a planar robot that can only drive forward and turn.This model is also known as Dubin's car.

$$
\begin{equation}
\begin{pmatrix}\dot{x}\\\\
\dot{y}\\\\
\dot{\theta}
\end{pmatrix}=\begin{pmatrix}v\sin\theta\\\\
v\cos\theta\\\\
\omega
\end{pmatrix}.\label{eq:dubins_car}
\end{equation}
$$

Let's say that given a constant velocity and angular rate input, we
want to know where this robot is after one second. How should we solve
this? Well, one straight forward way is with good, old-fashioned Euler
integration

$$
\x\left[t+\delta t\right]=\x\left[t\right]+\delta t\:\dot{\x}\left[t\right].
$$

If we do this, we should get something like what is shown in [Figure 3](#fig:EulerIntegration).
In each trajectory, I have integrated
the same amount of time, but with smaller and smaller time steps.
If we think about what is happening from the point of view of the
robot, we could imagine that for each $\delta t,$ we drive forward
at some velocity, and then turn. We then repeat this, over and over
until we reach the end of the interval. In [Figure 3](#fig:EulerIntegration)
we can see that as our $\delta t$'s get smaller, we seem to be approaching
some ideal situation, where we are no longer separating the *drive forward* step from
the *turn* step. They are happening simultaneously,
and we are getting that natural curving motion that we intuitively
expect.

<a name="fig:EulerIntegration"></a>
<div class="center">
<img  src="/lie_tutorial/forward_integration.svg" style="width: 50vw; min-width: 120px;">
<figcaption class="center">Figure 3: Illustration of simple forward-mode numerical integration of a unicyle robot with different sized steps.</figcaption>
</div>

This is exactly the same situation as the compounding interest problem
discussed earlier, and we can set up our problem in such a way that
the matrix exponential solves it for us. The matrix exponential, similar
to the scalar exponential, is defined by the infinite series

$$
\begin{equation}
\exp\left(A\right)=\sum_{k=0}^{\infty}\frac{1}{k!}A^{k}.\label{eq:mat_exp_inf}
\end{equation}
$$

<a name="eq:mat_exp_inf_anchor"></a>
We will see in [Section 6](/lie_algebra_tutorial/06-closed_form_mat_exp) that for
many useful situations, this expression has a closed form. To solve
Eq. \ref{eq:dubins_car}, with the matrix exponential we first need
to convert the system into a matrix differential equation of the form

$$
\dot{\x}=\x A.
$$

In this particular example, we can combine the $2\times2$ rotation
matrix formed from $\theta$ and the translation states from our state
vector to form a $3\times3$ state matrix as in [^2]


[^2]: This is an example of $\SE\left(2\right),$ the special euclidean group with two dimensions. The top left $2\times2$ block is the planar rotation matrix.

$$
\begin{bmatrix}x\\\\
y\\\\
\theta
\end{bmatrix}\to\begin{bmatrix}\cos\left(\theta\right) & -\sin\left(\theta\right) & x\\\\
\sin\left(\theta\right) & \cos\left(\theta\right) & y\\\\
0 & 0 & 1
\end{bmatrix}.
$$

The dynamics of this matrix are given as:

$$
\begin{align*}
\frac{d}{dt}\begin{bmatrix}\cos\left(\theta\right) & -\sin\left(\theta\right) & x\\\\
\sin\left(\theta\right) & \cos\left(\theta\right) & y\\\\
0 & 0 & 1
\end{bmatrix} & =\begin{bmatrix}\cos\left(\theta\right) & -\sin\left(\theta\right) & x\\\\
\sin\left(\theta\right) & \cos\left(\theta\right) & y\\\\
0 & 0 & 1
\end{bmatrix}\cdot\begin{bmatrix}0 & -\omega & v\\\\
\omega & 0 & 0\\\\
0 & 0 & 0
\end{bmatrix}.\\\\
\dot{\x} & =\x A
\end{align*}
$$

Don't be too worried about how this works. We will talk about this
a lot more later. Just know that this is an equivalent expression
to Equation \ref{eq:dubins_car}. As soon as we do, the solution is given as
easily as

$$
\begin{align*}
\dot{\x} & =\x_{0}e^{At}\\\\
\begin{bmatrix}\cos\left(\theta\right) & \sin\left(\theta\right) & x\\\\
-\sin\left(\theta\right) & \cos\left(\theta\right) & y\\\\
0 & 0 & 1
\end{bmatrix}\left(t\right) & =\begin{bmatrix}\cos\left(\theta_{0}\right) & \sin\left(\theta_{0}\right) & x_{0}\\\\
-\sin\left(\theta_{0}\right) & \cos\left(\theta_{0}\right) & y_{0}\\\\
0 & 0 & 1
\end{bmatrix}\cdot\exp\begin{pmatrix}0 & -\omega t & vt\\\\
\omega t & 0 & 0\\\\
0 & 0 & 0
\end{pmatrix}.
\end{align*}
$$

This is precisely what Lie theory is all about. We can use the matrix
exponential to map to curvy structures (rotation matrices or transformations)
from linear systems (vectors) in an efficient and principled way.
Consequently, we can consider concepts of rotation and translation
simultaneously and how they interact.

[Next: Rotations](/lie_algebra_tutorial/03-rotations/)
