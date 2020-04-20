---
type: post
title:  "Part 8: Computing the Adjoint"
date:   2020-04-18 13:03:00 -0600
categories: lie_algebra_tutorial
---

This is the seventh of a 8-part series of posts designed as a quick-start guide for students new
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
\def\grey#1{\color{gray}{#1}}
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


# Closed-Form Expressions for the Adjoint

In this section, I'm going to derive the Adjoint for each of our four
Lie Groups, but before we do that, let's first talk about a
concrete example of where the Adjoint is useful.

Let's consider some kind of nonlinear controller that is computing
error on the Lie algebra of $\SE\left(3\right)$, where we have some
desired state $\check{T}$ and our current state $T.$ We're going
to form some control law around the error between these two states.
Naturally, we want to use linear algebra as the backbone of our implementation,
so we'd like to use $\se\left(3\right)^{\vee}$ to represent our error.
We could compute the error in one of 4 ways:

$$
\begin{align*}
\dd_{a}^{\check{a}} & =\log\left(\check{\left(T_{a}^{b}\right)}^{-1}\cdot T_{a}^{b}\right)\\\\
\dd_{b}^{\check{b}} & =\log\left(\check{T_{a}^{b}}\cdot\left(T_{a}^{b}\right)^{-1}\right)\\\\
\dd_{\check{a}}^{a} & =\log\left(\left(T_{a}^{b}\right)^{-1}\cdot\check{T_{a}^{b}}\right)\\\\
\dd_{\check{b}}^{b} & =\log\left(\left(T_{a}^{b}\right)\cdot\left(\check{T}_{a}^{b}\right)^{-1}\right).
\end{align*}
$$

The choice of error depends largely on which frames are moving, and
which are not. Let's assume that in this case the $a$ frame is shared
between the transform (i.e. perhaps $a$ refers to the inertial frame),
and that the $b$ and $\check{b}$ frames are the current and desired
body frame of a robot. The goal of the controller in this case is
to make $b=\check{b}.$ In this case, it makes the most sense to define
our error as one of the following two options (or else the coordinate
frames will not match up):

$$
\begin{align*}
\dd_{b}^{\check{b}} & =\log\left(\check{T_{a}^{b}}\cdot\left(T_{a}^{b}\right)^{-1}\right),\\\\
\dd_{\check{b}}^{b} & =\log\left(T_{a}^{b}\cdot\left(\check{T}_{a}^{b}\right)^{-1}\right).
\end{align*}
$$

Driving either of these errors to zero will achieve our control objectives.
However, watch what happens when we want to compute the Jacobian of
our error with respect to our current state. We can find these Jacobians
using the chain rule:

$$
\begin{align*}
\frac{\partial}{\partial T_{a}^{b}}\dd_{b}^{\check{b}} & =\left.\frac{\partial\log\left(\x\right)}{\partial\x}\right|_{\x=\left(T_{a}^{b}\cdot\left(T_{a}^{\check{b}}\right)\right)}\cdot\frac{\partial}{\partial T_{a}^{b}}\left(\check{T_{a}^{b}}\cdot\left(T_{a}^{b}\right)^{-1}\right) & =A\cdot B^{\check{b}}\\\\
\frac{\partial}{\partial T_{a}^{b}}\dd_{\check{b}}^{b} & =\left.\frac{\partial\log\left(\x\right)}{\partial\x}\right|_{\x=\left(T_{a}^{b}\cdot\left(T_{a}^{\check{b}}\right)\right)}\cdot\frac{\partial}{\partial T_{a}^{b}}\left(T_{a}^{b}\cdot\left(\check{T_{a}^{b}}\right)^{-1}\right) & =A\cdot B^{b}
\end{align*}
$$


The expression for $A$ is in the [Table of Jacobians](/lie_algebra_tutorial/09-lie_group_tables/),
and it's the same for both parameterizations. However, we are left
with the two $B$ expressions. In the first case, we have to go through
our desired state to get to the current state. This isn't too bad,
we just look at the [Table of Jacobians](/lie_algebra_tutorial/09-lie_group_tables/) to
get (using the left Jacobian):

$$
\begin{align*}
B^{\check{b}} & =\frac{\partial}{\partial T_{a}^{b}}\left(\check{T_{a}^{b}}\cdot\left(T_{a}^{b}\right)^{-1}\right)\\\\
 & =-\Ad\left(\check{T_{a}^{b}}\cdot\left(T_{a}^{b}\right)^{-1}\right)
\end{align*}
$$

for the first cast, and

$$
\begin{align*}
B^{b} & =\frac{\partial}{\partial T_{a}^{b}}\left(T_{a}^{b}\cdot\left(\check{T_{a}^{b}}\right)^{-1}\right)\\\\
 & =I\in\R^{6\times6}.
\end{align*}
$$

for the second. Having a closed form expression for the Adjoint makes
computing these kinds of expressions very straight-forward.

Most modern forms of control and estimation rely heavily on quickly-computed
Jacobians. With a table of Jacobians for exp, log, Ad and a few other
Lie group expressions, plus the [Matrix Cookbook Petersen (2012)](https://www.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf),
it's pretty easy to chain-rule out even the hairiest Jacobians pretty
quickly to get analytical expressions. [Lie Group Tables](/lie_algebra_tutorial/09-lie_group_tables/)
contains most of these Jacobians for easy reference.

## Adjoint for $\SO\left(3\right)$

For all four Lie groups, we can derive the Adjoint by finding the
closed-form expression of Eq. \ref{eq:adjoint_start}. $\SO\left(3\right)$
is quite easy:

$$
\begin{align*}
\Ad\left(R\right) & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}R\cdot\exp\left(J_{1}\varepsilon\right)\cdot R^{-1} & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}R\cdot\left(I+\skew{\e_{1}\epsilon}+\cdots\right)\cdot R^{-1} & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}R\cdot R^{-1}+R\cdot\skew{\e_{1}\epsilon}\cdot R^{-1} & \cdots\end{bmatrix}\\\\
 & =\begin{bmatrix}R\cdot\skew{\e_{1}}\cdot R^{-1} & \cdots\end{bmatrix} & \grey{R\cdot R^{-1}=I}\\\\
 & =\begin{bmatrix}\skew{R\e_{1}} & \skew{R\e_{2}} & \skew{R\e_{3}}\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_2})}}\\\\
 & =R
\end{align*}
$$

We see here that $\SO\left(3\right)$ is *self-adjoint*, and
it makes sense why. Consider an $\SO\left(3\right)$ version of Eq.
\ref{eq:adjoint_show}. Rotating the vector is the natural way to
transform $\dd$ from the $a$ frame to the $b$ frame.

$$
\boxed{\Ad\left(R\right)=R}
$$


## Adjoint for Unit Quaternions

Remember, the Adjoint action is given by

$$
\Ad=\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\q\cdot\exp\left(J_{1}\varepsilon\right)\cdot\q^{-1}\right) & \cdots\end{bmatrix}.
$$

In the case of unit quaternions, we can see that they too, are self-adjoint.
To compute the adjoint of a quaternion, all we need to do is pre-
and post-multiply a pure quaternion of the lie algebra member by the
quaternion.

$$
\Ad\left(\q\right)=\q\cdot\dd\cdot\q^{-1}.
$$

However, sometimes we might want the matrix form of the adjoint. This
can be computed as follows:

$$
\begin{align*}
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\q\cdot\begin{pmatrix}0\\\\
\e_{1}\sin\left(\frac{\varepsilon}{2}\right)
\end{pmatrix}\cdot\q^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\begin{pmatrix}0\\\\
\frac{R^{\top}\e_{1}\varepsilon}{2}
\end{pmatrix} & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}R^{\top}\e_{1}\varepsilon & R^{\top}\e_{2}\varepsilon & R^{\top}\e_{3}\varepsilon\end{bmatrix}\\\\
 & =R^{\top}.
\end{align*}
$$

You could probably come up with this only just from intuition as well.
If you remember that quaternions and rotation matrices share a Lie
Algebra, and that the order of concatenation is backwards, then $R^{\top}$
is the logical choice.

$$
\boxed{\Ad\left(\q\right)=R^{\top}\left(\q\right)}
$$


## Adjoint for $\SE\left(3\right)$

Computing this adjoint is more complicated than pure rotation. Unfortunately,
$\SE\left(3\right)$ isn't self-adjoint[^1] so the limit gets a little more hairy. We have two main blocks: the
first three generators are the rotation generators, while the last
three are the translation generators. Let's consider these separately.
First, the rotation blocks

[^1]: This should be pretty obvious, seeing as $\se\left(3\right)$ is 6-dimensional while $\SE\left(3\right)$ is a group of $4\times4$ matrices.

$$
\begin{align*}
\Ad_{1-3} & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(T\cdot\exp\left(J_{1}\varepsilon\right)\cdot T^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(T\cdot\begin{bmatrix}\skew{\e_{1}\varepsilon} & 0\\\\
0 & 0
\end{bmatrix}\cdot T^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}R & \t\\\\
0 & 1
\end{bmatrix}\cdot\begin{bmatrix}\skew{\e_{1}\varepsilon} & 0\\\\
0 & 0
\end{bmatrix}\cdot\begin{bmatrix}R^{\top} & -R^{\top}\t\\\\
0 & 1
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}R\skew{\e_{1}\varepsilon} & 0\\\\
0 & 0
\end{bmatrix}\cdot\begin{bmatrix}R^{\top} & -R^{\top}\t\\\\
0 & 1
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}R\skew{\e_{1}\varepsilon}R^{\top} & -R\skew{\e_{1}\varepsilon}R^{\top}\t\\\\
0 & 0
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\skew{R\e_{1}\varepsilon} & -\skew{R\e_{1}\varepsilon}\t\\\\
0 & 0
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_2})}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\skew{R\e_{1}\varepsilon} & \skew{\t}R\e_{1}\varepsilon\\\\
0 & 0
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_1})}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}R\varepsilon\\\\
\skew{\t}R\varepsilon
\end{bmatrix}\\\\
 & =\begin{bmatrix}R\\\\
\skew{\t}R
\end{bmatrix}.
\end{align*}
$$

Now for the translation blocks

$$
\begin{align*}
\Ad_{4-6} & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(T\cdot\exp\left(J_{4}\varepsilon\right)\cdot T^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}R & \t\\\\
0 & 1
\end{bmatrix}\cdot\begin{bmatrix}0 & \e_{1}\varepsilon\\\\
0 & 0
\end{bmatrix}\cdot\begin{bmatrix}R^{\top} & -R^{\top}\t\\\\
0 & 1
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}0 & R\e_{1}\varepsilon\\\\
0 & 0
\end{bmatrix}\cdot\begin{bmatrix}R^{\top} & -R^{\top}\t\\\\
0 & 1
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}0 & R\e_{1}\varepsilon\\\\
0 & 0
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}0 & 0 & 0\\\\
R\e_{1}\varepsilon & R\e_{2}\varepsilon & R\e_{3}\varepsilon
\end{bmatrix}\\\\
 & =\begin{bmatrix}0\\\\
R
\end{bmatrix}.
\end{align*}
$$

We now concatenate these blocks to get the final Adjoint matrix:

$$
\boxed{\Ad\left(T\right)=\begin{pmatrix}R & 0\\\\
\skew{\t}R & R
\end{pmatrix}.}
$$

Note that some literature [Drummond (2014)](http://twd20g.blogspot.com/p/notes-on-lie-groups.html), [Eade (2019)](http://ethaneade.com/lie_groups.pdf) derives this
block-transposed. All that means is that they define generators 1-3
as the translation generators and 4-6 as the rotation generators.

Before moving on, let's take a second to develop a little bit of intuition
regarding the adjoint when it comes to rigid transforms. Consider
a rod rotating about a fixed origin, as shown in [Figure 6](#fig:rotating_adjoint).
Let us say we have the linear and angular velocity of coordinate frame
$a$, but we want the velocity of coordinate frame $b$. Because the
coordinate frames are fixed by some rigid transform, any angular rate
value of $a$ is the same for $b$, only rotated. However, to compute
the linear velocity of coordinate frame $b$, we must account for
the ``lever arm'' and the angular rate. This is what is going on
in the bottom-left corner of the Adjoint in $\SE\left(3\right).$


<a name="fig:rotating_adjoint"></a>
<div class="center">
<img  src="/lie_tutorial/rotating_bar.svg" style="width: 30vw; min-width: 120px;">
<figcaption class="center">Figure 6: Illustration of a rotating rigid body with two coordinate frames. The Adjoint of $\SE\left(3\right)$ will convert $\w$ and $\v$ from frame $a$ to frame $b$, and account for the change in translation. </figcaption>
</div>

## Adjoint of Dual Quaternions

As with unit quaternions, dual unit quaternions are also self-adjoint:

$$
\Ad\left(\qq\right)=\qq\cdot\exp\left(\d\right)\cdot\qq^{-1}.
$$

We have no problem dealing with six-vectors here (the problem with
$\SE\left(3\right),$which led us to compute that hairy limit). However,
just like unit quaternions, we can compute the matrix form of the
Adjoint using the limit expression. Again, let's split up the generators
into two groups to help keep things a little more concise

$$
\begin{align*}
\Ad_{1-3} & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\qq\cdot\exp\left(J_{1}\varepsilon\right)\cdot\qq^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}\cdot\begin{bmatrix}\begin{pmatrix}0\\\\
\e_{1}\sin\left(\frac{\varepsilon}{2}\right)
\end{pmatrix}\\\\
0\epsilon
\end{bmatrix}\cdot\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}\cdot\begin{bmatrix}\e_{1}\varepsilon\\\\
0\epsilon
\end{bmatrix}\cdot\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}^{-1}\right) & \cdots\end{bmatrix} & \grey{\sin\varepsilon\to\varepsilon}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\\\\
\epsilon\left(\e_{1}^{\wedge}\varepsilon\cdot\q_{d}\right)
\end{bmatrix}\cdot\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\epsilon\left(\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{d}^{-1}+\q_{r}^{-1}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{d}\right)
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\e_{1}^{\wedge}=\begin{pmatrix}0\\\\
\e_{1}
\end{pmatrix}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\epsilon\left(\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\begin{pmatrix}\frac{1}{2}\t^{\wedge}\cdot\q_{r}\end{pmatrix}^{-1}+\q_{r}^{-1}\cdot\e_{1}^{\wedge}\varepsilon\cdot\frac{1}{2}\t^{\wedge}\cdot\q_{r}\right)
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\frac{1}{2}\epsilon\left(\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\cdot\left(-\t^{\wedge}\right)+\q_{r}^{-1}\cdot\e_{1}^{\wedge}\varepsilon\cdot\t^{\wedge}\cdot\q_{r}\right)
\end{bmatrix}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\frac{1}{2}\epsilon\left(-\skew{R^{\top}\e_{1}\varepsilon}\t+R\skew{\e_{1}\varepsilon}\t\right)
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:quat_mult_matrix})}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\frac{1}{2}\epsilon\left(-R^{\top}\skew{\e_{1}\varepsilon}R\t+R\skew{\e_{1}\varepsilon}\t\right)
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_2})}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\frac{1}{2}\epsilon\left(R^{\top}\skew{R\t}\e_{1}\varepsilon-R\skew{\t}\e_{1}\varepsilon\right)
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_1})}}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\cdot\e_{1}^{\wedge}\varepsilon\cdot\q_{r}^{-1}\\\\
\frac{1}{2}\epsilon\left(\skew{\t}R^{\top}\e_{1}\varepsilon-R\skew{\t}\e_{1}\varepsilon\right)
\end{bmatrix}\right) & \cdots\end{bmatrix} & \grey{\textrm{(Eq. \ref{eq:skew_trick_1})}}\\\\
 & =\begin{bmatrix}R^{\top}\\\\
\skew{\t}R^{\top}
\end{bmatrix}.
\end{align*}
$$

Whew! Luckily, the translation generators are much easier:

$$
\begin{align*}
\Ad_{4-6} & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\qq\cdot\exp\left(J_{1}\varepsilon\right)\cdot\qq^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}\cdot\begin{bmatrix}0\\\\
\e_{1}\epsilon
\end{bmatrix}\cdot\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\left(\begin{bmatrix}0\\\\
\epsilon\left(\q_{r}\cdot\e_{q}\varepsilon\right)
\end{bmatrix}\cdot\begin{bmatrix}\q_{r}\\\\
\epsilon\q_{d}
\end{bmatrix}^{-1}\right) & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\begin{bmatrix}0\\\\
\epsilon\left(\q_{r}\cdot\e_{q}\varepsilon\cdot\q_{r}^{-1}\right)
\end{bmatrix} & \cdots\end{bmatrix}\\\\
 & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\log\begin{bmatrix}0\\\\
\epsilon\left(R^{\top}\e_{1}\varepsilon\right)
\end{bmatrix} & \cdots\end{bmatrix}\\\\
 & =R^{\top}.
\end{align*}
$$

Finally, we can write the full Adjoint for dual quaternions as

$$
\boxed{\Ad\left(\qq\right)=\begin{pmatrix}R^{\top} & 0\\\\
\skew{\t}R^{\top} & R^{\top}
\end{pmatrix}},
$$

which, unsurprisingly, is the same as the Adjoint of $\SE\left(3\right)$,
except with all the rotations transposed (due to the reverse order
of quaternions rotation with respect to $\SO\left(3\right).$) Note
that this is also the matrix inverse of the Adjoint of $\SE\left(3\right)$.

Now that we have each of the Adjoints, we can use the tables in [Lie Group Tables](/lie_algebra_tutorial/09-lie_group_tables/) to find Jacobians for almost any arbitrary expression
involving Lie groups!
