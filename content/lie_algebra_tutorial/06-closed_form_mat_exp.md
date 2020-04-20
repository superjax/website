---
type: post
title:  "Part 6: Computing the Matrix Exponential"
date:   2020-04-18 13:05:00 -0600
categories: math
tags: ["vectors", "notation", "geometry", "rotations", "lie"]
---

This is the sixth of a 8-part series of posts designed as a quick-start guide for students new
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

# Computing the Matrix Exponential

Now that we have introduced the primary four 3D Lie groups used in
our field, the rest of this document focuses on making efficient implementations
in software. We'll start in this section by deriving closed-form expressions
for the exponential and logarithm of each group. The next section
focuses on the Adjoint, and computing Jacobians of expressions involving
Lie group operations.

As noted in Section [2](matrix_exp.html), the matrix exponential
is defined with an infinite series. Given a square matrix $A$, the
matrix exponential of $A$ is defined as

$$
\begin{align*}
\exp\left(A\right) & =I+A+\frac{1}{2}A^{2}+\frac{1}{6}A^{3}+\cdots\\\\
 & =\sum_{k=0}^{\infty}\frac{1}{k!}A^{k}.
\end{align*}
$$

and the matrix logarithm is defined as the inverse operation.

The matrix exponential will converge for any square matrix, however,
the matrix logarithm may not. For example, consider a rotation matrix
formed by a 180 degree rotation in terms of its axis-angle approximation.
You could rotate around any axis 180 degrees to get this rotation.
Computing the logarithm of such a rotation therefore has an infinite
number of solutions.

If you wanted to, you could just go ahead and compute this series
up to a high enough order, and you would get the right answer[^1]. However, for all of the Lie groups discussed above, the matrix exponential
has a closed form that make computing it much, much faster. In this
section, we will derive these forms for each of the Groups discussed.

[^1]: In fact, this is a really great way to check your implementation for accuracy.

<style>
.alert-note {
    border-radius: 10px;
    margin-bottom: 1rem;
    color: #3b3b3b;
    background-color: #c3dfeb;
    border-color: #74b7d4;
    padding: 20px;
  }
</style>

<div class="alert-note">
this section is going to be a bit dry, as it is pretty
much entirely nitty-gritty algebra. You may want to just skip to the
section you're most interested in."
</div>


## Closed-Form Matrix Exponential for $\SO\left(3\right)$

We know that the combination of the generators will give us a skew-symmetric
matrix, therefore the matrix exponential of $\so\left(3\right)$ will
look like

$$
\begin{align*}
\exp\left(\skew{\w}\right) & =I+\skew{\w}+\frac{1}{2!}\skew{\w}^{2}+\frac{1}{3!}\skew{\w}^{3}+\cdots.
\end{align*}
$$

If we collect the even and odd pairs together, we will get

$$
\exp\left(\skew{\w}\right)=I+\sum_{k=0}^{\infty}\left[\frac{\skew{\w}^{2k+1}}{\left(2k+1\right)!}+\frac{\skew{\w}^{2k+2}}{\left(2k+2\right)!}\right],
$$

and we use the property of skew-symmetric matrices that

$$
\skew{\w}^{3}=-\norm{\w}^{2}\skew{\w},
$$

then we can rewrite the coefficients of this series in terms of the norm of $\w$ squared and $\skew w$ as

$$
\begin{align*}
\theta^{2} & =\w^{\top}\w\\\\
\skew{\w}^{2k+1} & =\left(-1\right)^{k}\theta^{2k}\skew{\w}\\\\
\skew{\w}^{2k+2} & =\left(-1\right)^{k}\theta^{2k}\skew{\w}^{2}.
\end{align*}
$$

Let's plug these coefficients back into the equation and munge around
a little bit, factoring out the $\skew{\w}$ and $\skew{\w}^{2}$
terms until we get

$$
\begin{align}
\exp\left(\skew{\w}\right) & =I+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}\skew{\w}}{\left(2k+1\right)!}\right)+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}\skew{\w}^{2}}{\left(2k+2\right)!}\right)\nonumber \\\\
 & =I+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}}{\left(2k+1\right)!}\right)\skew{\w}+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}}{\left(2k+2\right)!}\right)\skew{\w}^{2}\nonumber \\\\
 & =I+\left(1-\frac{\theta^{2}}{3!}+\frac{\theta^{4}}{5!}+\cdots\right)\skew{\w}+\left(\frac{1}{2!}-\frac{\theta^{2}}{4!}+\frac{\theta^{4}}{6!}+\cdots\right)\skew{\w}^{2}.\label{eq:rodrigues_taylor_series}
\end{align}
$$

At this point, we can refer to the [Table of Taylor Series Expansions](/lie_algebra_tutorial/09-lie_group_tables/) and
recognize the sinc function and a cosine-related function. We can
plug these expressions into Eq. \ref{eq:rodrigues_taylor_series}
and get the well-known Rodrigues formula,

$$
\begin{equation}
\boxed{\exp\left(\skew{\w}\right)=I+\left(\frac{\sin\left(\theta\right)}{\theta}\right)\skew{\w}+\left(\frac{1-\cos\left(\theta\right)}{\theta^{2}}\right)\skew{\w}^{2}.}\label{eq:so3_exp}
\end{equation}
$$

This formula will have numerical problems if $\theta\approx0,$ so
best practice is to use the Taylor series version with two or three
terms if you've got a small $\theta.$

Now, if you wanted to compute the matrix logarithm for $\SO\left(3\right),$
you can just compute the inverse of Eq. \ref{eq:so3_exp}, as

$$
\boxed{\begin{array}{cc}
\log\left(R\right) & =\frac{\theta}{2\sin\theta}\left(R-R^{\top}\right),\end{array}}
$$
where

$$
\theta=\cos^{-1}\left(\frac{\textrm{tr}\left(R\right)-1}{2}\right).
$$

This equation has two points where numerical issues come into play,
the first is when $\theta\approx0,$ the other is when $\theta\approx\pi$.
As with Eq. \ref{eq:so3_exp}, just use the Taylor series of $\log\left(R\right)$
if $\theta$ is close to these values (See [Table of Taylor Series Expansions](/lie_algebra_tutorial/09-lie_group_tables/)).
As noted earlier, when $\theta=\pi$, then there are actually infinitely
many solutions to the $\log\left(R\right),$ in this case, many implementations
just choose one.

## Closed-Form Exponential for $\S^{3}$

Let's start with the definition of the quaternion exponential

$$
\begin{align}
\exp\left(\w^{\wedge}\right) & =\sum_{k=0}^{\infty}\frac{\left(\w^{\wedge}\right)^{k}}{k!}, & \w^{\wedge} & =\begin{pmatrix}0\\\\
\w
\end{pmatrix}\label{eq:quat_exp_series}
\end{align}
$$

where the term $\left(\q\right)^{k}$ refers to multiplying $\q$
with itself $\left(\q\cdot\q\cdot\cdots\right)$ $k$ times. We also
note that

$$
\begin{align*}
\left(\w^{\wedge}\right)^{2} & =\left(\w_{x}i+\w_{y}j+\w_{z}k\right)\cdot\left(\w_{x}i+\w_{y}j+\w_{z}k\right)\\\\
 & =-\w_{x}^{2}-\w_{d}^{2}-\w_{z}^{2}\\\\
 & =-\norm{\w}^{2}
\end{align*}
$$

Therefore, if $\theta=\norm{\w}$, then

$$
\begin{align*}
\left(\w^{\wedge}\right)^{2} & =-\theta^{2}, & \left(\w^{\wedge}\right)^{3} & =-\theta^{2}\w^{\wedge} & \left(\w^{\wedge}\right)^{4} & =\theta^{4}\cdots.
\end{align*}
$$

Now, we can rewrite the series in Eq. \ref{eq:quat_exp_series} as

$$
\begin{align}
\exp\left(\w^{\wedge}\right) & =\sum_{k=0}^{\infty}\frac{\left(\w^{\wedge}\right)^{k}}{k!}\nonumber \\\\
 & =1+\w^{\wedge}-\frac{\theta^{2}}{2!}-\frac{\theta^{2}}{3!}\w^{\wedge}+\frac{\theta^{4}}{4!}+\frac{\theta^{4}}{5!}\w^{\wedge}-\cdots\nonumber \\\\
 & =1+\frac{\theta}{\theta}\w^{\wedge}-\frac{\theta^{2}}{2!}-\frac{\theta^{3}}{3!\theta}\w^{\wedge}+\frac{\theta^{4}}{4!}+\frac{\theta^{5}}{5!\theta}\w^{\wedge}-\cdots\nonumber \\\\
 & =\left(1-\frac{\theta^{2}}{2!}+\frac{\theta^{4}}{4!}\cdots\right)+\frac{1}{\theta}\left(\theta-\frac{\theta^{3}}{3!}+\frac{\theta^{5}}{5!}\cdots\right)\w^{\wedge}\nonumber \\\\
 & =\cos\left(\theta\right)+\frac{\sin\left(\theta\right)}{\theta}\w^{\wedge}.\label{eq:quat_exp}
\end{align}
$$

As with the rotation matrix exponential, we will want to use the Taylor
series approximation if $\theta\approx0$ to avoid numerical errors.
(See [Table of Taylor Series Expansions](/lie_algebra_tutorial/09-lie_group_tables/)).

Computing the closed-form logarithm is done by inverting Eq. \ref{eq:quat_exp}
and is given by

$$
\log\left(\q\right)=\textrm{atan2}\left(\norm{\vec{\q}},q_{0}\right)\frac{\q}{\norm{\vec{\q}}}.
$$

Because of the fact that quaternion dynamics have a $\frac{1}{2}$
in front of them (See Eq. \ref{eq:quaternion_dynamics}), it is common
practice in both physics and robotics applications to embed the $\frac{1}{2}$
into the $\exp$ function itself such that[^2]

[^2]: You might recognize this as the axis-angle conversion to unit quaternions (Eq. \ref{eq:quat_axis_angle})

$$
\boxed{\exp\left(\w\right)=\cos\left(\frac{\norm{\w}}{2}\right)+\sin\left(\frac{\norm{\w}}{2}\right)\frac{\w}{\norm{\w}}}
$$

$$
\boxed{\log\left(\q\right)=2\tan^{-1}\left(\frac{\norm{\vec{\q}}}{q_{0}}\right)\cdot\frac{\vec{\q}}{\norm{\vec{\q}}}.}
$$

This is actually the same as if we redefined the generators of the
unit quaternion to all be $\frac{1}{2}J_{i}$. This is fine, and totally
proper, however we just need to be explicit about this choice.

## Closed-Form Matrix Exponential for $\SE\left(3\right)$

To derive the closed-form expression for the matrix exponential of
$\SE\left(3\right)$ we start with the series expression for the matrix
exponential and play with it until we can find patterns we can convert
into known expressions. In the case of $\SE\left(3\right)$, this
procedure starts out as follows:

$$
\begin{align*}
\exp\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix} & =I+\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix}+\frac{1}{2!}\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix}^{2}+\frac{1}{3!}\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix}^{3}+\cdots\\\\
 & =I+\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix}+\frac{1}{2!}\begin{pmatrix}\skew{\w}^{2} & \skew{\w}\v\\\\
0 & 1
\end{pmatrix}+\frac{1}{3!}\begin{pmatrix}\skew{\w}^{3} & \skew{\w}^{2}\v\\\\
0 & 1
\end{pmatrix}+\cdots\\\\
 & =\begin{pmatrix}\exp\left(\skew{\w}\right) & V\v\\\\
0 & 1
\end{pmatrix}\\\\
V & =I+\frac{1}{2}\skew{\w}+\frac{1}{3!}\skew{\w}^{2}.
\end{align*}
$$

At this point, we can see that the upper right block is just the rotation
matrix, but we have this matrix $V$ that couples some amount of angular
rate in with linear velocity. This is the term that describes the
screw motions we expect from this exponential map. We can find a closed-form
expression for $V$ by splitting the even and odd pairs which leads
to:

$$
\begin{align*}
V & =I+\sum_{k=0}^{\infty}\left(\frac{\skew{\w}^{\left(2k+1\right)}}{\left(2k+2\right)!}+\frac{\skew{\w}^{\left(2k+2\right)}}{\left(2k+3\right)!}\right)\\\\
 & =I+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}}{\left(2k+2\right)!}\right)\skew{\w}+\sum_{k=0}^{\infty}\left(\frac{\left(-1\right)^{k}\theta^{2k}}{\left(2k+3\right)!}\right)\skew{\w}^{2}\\\\
 & =I+\left(\frac{1}{2}-\frac{\theta^{2}}{4!}+\frac{\theta^{4}}{6!}+\cdots\right)\skew{\w}+\left(\frac{1}{3!}-\frac{\theta^{2}}{5!}+\frac{\theta^{4}}{7!}+\cdots\right)\skew{\w}^{2}\\\\
 & =I+\left(\frac{1-\cos\left(\theta\right)}{\theta^{2}}\right)\skew{\w}+\left(\frac{\theta-\sin\left(\theta\right)}{\theta^{3}}\right)\skew{\w}^{2}.
\end{align*}
$$

In summary, the final form for the matrix exponential is given as

$$
\boxed{\exp\begin{pmatrix}\skew{\w} & \v\\\\
0 & 1
\end{pmatrix}=\begin{pmatrix}\exp\left(\skew{\w}\right) & V\v\\\\
0 & 1
\end{pmatrix},}
$$

where

$$
V=I+\left(\frac{1-\cos\left(\theta\right)}{\theta^{2}}\right)\skew{\w}+\left(\frac{\theta-\sin\left(\theta\right)}{\theta^{3}}\right)\skew{\w}^{2},
$$

and the logarithm has this closed-form:

$$
\boxed{\log\begin{pmatrix}R & \t\\\\
0 & 1
\end{pmatrix}=\begin{pmatrix}\log\left(R\right) & V^{-1}\t\\\\
0 & 0
\end{pmatrix},}
$$

where

$$
V^{-1}=I-\frac{1}{2}\skew{\w}+\frac{1}{\theta^{2}}\left(1-\frac{\theta\sin\theta}{2\left(1-\cos\theta\right)}\right)\skew{\w}^{2}.
$$

As with the other results in this document, Taylor series expansions
for the numerically unstable trigonometric terms can be found in Table
\ref{tab:taylor_series}.

## Closed-Form Exponential for $\mathbb{D}\S^{3}$

To derive the exponential map of the dual quaternion, we start with
the exponential map for the ordinary quaternion, except with dual
numbers. For some syntactical relief, I'm going to abuse notation
a little bit and define $\w$ and $\v$ as pure quaternions and use
the $\tilde{\left(\cdot\right)}$ syntax to remind us that a value
has both real and dual components. The dual exponential map is given
as

$$
\begin{align}
\exp\left(\w+\epsilon\v\right) & =\cos\left(\frac{\tilde{\phi}}{2}\right)+\sin\left(\frac{\tilde{\phi}}{2}\right)\frac{\left(\w+\epsilon\v\right)}{\tilde{\phi}} & \tilde{\phi} & =\norm{\w+\epsilon\v}.\label{eq:dual_quat_exp_1}
\end{align}
$$

This is absolutely correct, but unwieldy. Let's use the identities
from [Table of Dual Number Identities](/lie_algebra_tutorial/09-lie_group_tables/) to break this into the real
and dual parts. First the dual angle can be re-written as

$$
\begin{align*}
\tilde{\phi} & =\sqrt{\left(\w+\epsilon\v\right)^{\top}\left(\w+\epsilon\v\right)}\\\\
 & =\sqrt{\left(\w\right)^{\top}\left(\w\right)+\epsilon2\w^{\top}\v}\\\\
 & =\sqrt{\w^{\top}\w}+\frac{\w^{\top}\v}{\sqrt{\w^{\top}\w}}\epsilon\\\\
 & =\theta+\frac{\gamma}{\theta}\epsilon.
\end{align*}
$$

where $\theta$ is the angle of rotation that we saw for the ordinary
quaternion and $\gamma$ is $\w^{\top}\v$. With this result, we can
now split the dual sin and cos functions as

$$
\begin{align*}
\cos\left(\frac{\tilde{\phi}}{2}\right) & =\cos\left(\frac{\theta}{2}+\epsilon\frac{\gamma}{2\theta}\right)\\\\
 & =\cos\frac{\theta}{2}-\epsilon\frac{\gamma}{2\theta}\sin\frac{\theta}{2}\\\\
\sin\left(\frac{\tilde{\phi}}{2}\right) & =\sin\frac{\theta}{2}+\epsilon\frac{\gamma}{2\theta}\cos\frac{\theta}{2},
\end{align*}
$$

and the sinc-like function expands as

$$
\begin{align*}
\frac{\sin\frac{\tilde{\phi}}{2}}{\tilde{\phi}} & =\frac{\sin\frac{\theta}{2}+\epsilon\frac{\gamma}{2\theta}\cos\frac{\theta}{2}}{\theta+\frac{\gamma}{\theta}\epsilon}\\\\
 & =\frac{\sin\frac{\theta}{2}+\epsilon\frac{\gamma}{2\theta}\cos\frac{\theta}{2}}{\theta+\frac{\gamma}{\theta}\epsilon}\left(\frac{\theta-\frac{\gamma}{\theta}\epsilon}{\theta-\frac{\gamma}{\theta}\epsilon}\right)\\\\
 & =\frac{\theta\sin\frac{\theta}{2}+\epsilon\left(\frac{\gamma}{2\theta}\theta\cos\frac{\theta}{2}-\frac{\gamma}{\theta}\sin\frac{\theta}{2}\right)}{\theta^{2}}\\\\
 & =\frac{\sin\frac{\theta}{2}}{\theta}+\epsilon\gamma\left(\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}}\right).
\end{align*}
$$

We can finally expand and factor Eq. \ref{eq:dual_quat_exp_1} into
the real and dual components as follows

$$
\begin{align*}
\exp\left(\w+\epsilon\v\right) & =\cos\frac{\tilde{\phi}}{2}+\frac{\sin\frac{\tilde{\phi}}{2}}{\tilde{\phi}}\left(\w+\epsilon\v\right)\\\\
 & =\left(\cos\frac{\theta}{2}-\epsilon\frac{\gamma}{2\theta}\sin\frac{\theta}{2}\right)+\left(\frac{\sin\frac{\theta}{2}}{\theta}+\epsilon\gamma\left(\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}}\right)\right)\left(\w+\epsilon\v\right)\\\\
 & =\left(\cos\frac{\theta}{2}-\epsilon\frac{\gamma}{2\theta}\sin\frac{\theta}{2}\right)+\left(\frac{\sin\frac{\theta}{2}}{\theta}\w+\epsilon\left(\gamma\left(\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}}\right)\w+\frac{\sin\frac{\theta}{2}}{\theta}\v\right)\right)\\\\
 & =\left(\cos\frac{\theta}{2}+\frac{\sin\frac{\theta}{2}}{\theta}\w\right)+\epsilon\left(-\frac{\gamma}{2\theta}\sin\frac{\theta}{2}+\gamma\left(\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}}\right)\w+\frac{\sin\frac{\theta}{2}}{\theta}\v\right)
\end{align*}
$$


$$
\boxed{\exp\left(\w+\epsilon\v\right)=\begin{pmatrix}\cos\frac{\theta}{2}\\\\
\frac{\sin\frac{\theta}{2}}{\theta}\w
\end{pmatrix}+\epsilon\begin{pmatrix}-\frac{\gamma}{2\theta}\sin\frac{\theta}{2}\\\\
\gamma\left(\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}}\right)\w+\frac{\sin\frac{\theta}{2}}{\theta}\v
\end{pmatrix},}
$$

which is the final form of the matrix exponential for the dual quaternion
representation.

In similar fashion, we can derive the dual quaternion logarithm by
expanding the dual form of the ordinary quaternion expression. We
start with the ordinary quaternion logarithm

$$
\log\left(\qq\right)=2\tan^{-1}\left(\frac{\norm{\tilde{\vec{\q}}}}{\tilde{q_{0}}}\right)\frac{\tilde{\vec{\q}}}{\norm{\tilde{\vec{\q}}}},
$$

which can now separate into real and dual parts.

First, let us start by separating the expression for the norm of the
dual quaternion imaginary component. For this derivation, I have defined

$$
\begin{align*}
\r & =\vec{\q}_{r} & r_{0} & =q_{r0} & \d & =\vec{\q}_{d} & d_{0} & =q_{d0}
\end{align*}
$$

to clean up the expressions somewhat. Let's start first with the norm
of the dual complex portion:

$$
\begin{align*}
\norm{\tilde{\vec{\q}}} & =\sqrt{\left(\r+\epsilon\d\right)^{\top}\left(\r+\epsilon\d\right)}\\\\
 & =\sqrt{\r^{\top}\r+\epsilon2\r^{\top}\d}\\\\
 & =\sqrt{\r^{\top}\r}+\frac{\gamma}{\sqrt{\r^{\top}\r}}\epsilon & \grey{\gamma} & \grey{=\r^{\top}\d}\\\\
 & =\theta+\frac{\gamma}{\theta}\epsilon. & \grey{\theta} & \grey{=\sqrt{\r^{\top}\r}},
\end{align*}
$$

We can use this to separate the argument of the arc-tangent function

$$
\begin{align*}
\frac{\norm{\tilde{\vec{\q}}}}{\tilde{q}_{0}} & =\frac{\theta+\frac{\gamma}{\theta}\epsilon}{r_{0}+\epsilon d_{0}}\\\\
 & =\frac{\theta+\frac{\gamma}{\theta}\epsilon}{r_{0}+\epsilon d_{0}}\left(\frac{r_{0}-\epsilon d_{0}}{r_{0}-\epsilon d_{0}}\right)\\\\
 & =\frac{r_{0}\theta}{r_{0}^{2}}+\epsilon\frac{\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)}{r_{0}^{2}}\\\\
 & =\frac{\theta}{r_{0}}+\epsilon\frac{\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)}{r_{0}^{2}},
\end{align*}
$$

the screw angle:

$$
\begin{align*}
\tan^{-1}\left(\frac{\norm{\tilde{\bar{\q}}}}{\tilde{q}_{0}}\right) & =\tan^{-1}\left(\frac{\theta}{r_{0}}+\epsilon\frac{\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)}{r_{0}^{2}}\right)\\\\
 & =\tan^{-1}\left(\frac{\theta}{r_{0}}\right)+\epsilon\left(\frac{\frac{\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)}{r_{0}^{2}}}{\left(\frac{\theta}{r_{0}}\right)^{2}+1}\right)\\\\
 & =\tan^{-1}\left(\frac{\theta}{r_{0}}\right)+\epsilon\left(\frac{\frac{\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)}{r_{0}^{2}}}{\frac{\theta^{2}+r_{0}^{2}}{r_{0}^{2}}}\right)\\\\
 & =\tan^{-1}\left(\frac{\theta}{r_{0}}\right)+\epsilon\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right), & \grey{\left(\theta^{2}+r_{0}^{2}=1\right)}
\end{align*}
$$

and the screw axis:

$$
\begin{align*}
\frac{\tilde{\bar{\q}}}{\norm{\tilde{\bar{\q}}}} & =\frac{\r+\epsilon\d}{\theta+\frac{\gamma}{\theta}\epsilon}\\\\
 & =\frac{\r+\epsilon\d}{\theta+\frac{\gamma}{\theta}\epsilon}\left(\frac{\theta-\frac{\gamma}{\theta}\epsilon}{\theta-\frac{\gamma}{\theta}\epsilon}\right)\\\\
 & =\frac{\r}{\theta}+\epsilon\frac{\d\theta-\frac{\gamma}{\theta}\r}{\theta^{2}}.
\end{align*}
$$

With these expressions, we can write the full form of the dual quaternion
logarithm as

$$
\begin{align*}
\log\left(\qq\right) & =2\tan^{-1}\left(\norm{\tilde{\bar{\q}}},\tilde{q_{0}}\right)\frac{\tilde{\bar{\q}}}{\norm{\tilde{\bar{\q}}}}\\\\
 & =2\left(\tan^{-1}\left(\frac{\theta}{r_{0}}\right)+\epsilon\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)\right)\left(\frac{\r}{\theta}+\epsilon\frac{\d\theta-\frac{\gamma}{\theta}\r}{\theta^{2}}\right)\\\\
 & =2\left(\frac{\r}{\theta}\tan^{-1}\left(\frac{\theta}{r_{0}}\right)\right)+2\epsilon\left(\tan^{-1}\left(\frac{\theta}{r_{0}}\right)\left(\frac{\d\theta-\frac{\gamma}{\theta}\r}{\theta^{2}}\right)+\left(r_{0}\frac{\gamma}{\theta}-d_{0}\theta\right)\frac{\r}{\theta}\right)\\\\
 & =2\left(\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}\r\right)+2\epsilon\left(\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}\left(\d-\frac{\gamma}{\theta^{2}}\r\right)+\left(r_{0}\frac{\gamma}{\theta^{2}}-d_{0}\right)\r\right).
\end{align*}
$$

Again, to make things a little more compact, we can define

$$
\begin{align*}
a & =\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}, & b & =\frac{\gamma}{\theta^{2}},
\end{align*}
$$

and simplify the expression above to get

$$
\begin{align*}
\log\left(\qq\right) & =2\left(a\r+\epsilon\left(a\left(\d-b\r\right)+\left(r_{0}b-d_{0}\right)\r\right)\right)\\\\
 & =2a\r+2\epsilon\left(a\d+\left(\left(r_{0}-a\right)b-d_{0}\right)\r\right).
\end{align*}
$$

This gives us the final form of the logarithm for the dual quaternion:

$$
\boxed{\log\left(\qq\right)=2\begin{pmatrix}a\r\\\\
a\d+\left(\left(r_{0}-a\right)b-d_{0}\right)\r
\end{pmatrix}.}
$$

To deal with the case that $\theta$ is small, we have to re-arrange
the expression so we can find a numerically stable Taylor series expression [Dantam (2018)](http://www.neil.dantam.name/papers/dantam2018practical.pdf).

$$
\begin{align*}
\log\left(\qq\right) & =2a\r+2\epsilon\left(a\d+\left(\left(r_{0}-a\right)b-d_{0}\right)\r\right)\\\\
 & =2a\r+2\epsilon\left(\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}\d+\left(\left(r_{0}-\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}\right)\frac{\gamma}{\theta^{2}}-d_{0}\right)\r\right)\\\\
 & =2a\r+2\epsilon\left(\frac{\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta}\d+\left(\gamma\frac{\theta\cos\left(\frac{\theta}{2}\right)-\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta^{3}}-d_{0}\right)\r\right) & \grey{\left(r_{0}=\cos\left(\frac{\theta}{2}\right)\right)}\\\\
 & =2a\r+2\epsilon\left(a\d+\left(\gamma\frac{\theta\cos\left(\frac{\theta}{2}\right)-\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta^{3}}-d_{0}\right)\r\right).
\end{align*}
$$

At this point, we can pump the nasty trig expression into Wolfram
Alpha to get a useful Taylor series representation to use when $\theta\approx0.$

$$
\frac{\theta\cos\left(\frac{\theta}{2}\right)-\tan^{-1}\left(\frac{\theta}{r_{0}}\right)}{\theta^{3}}=\frac{5}{24}-\frac{379}{1920}\theta^{2}+\frac{46073}{322560}\theta^{4}-\frac{127431}{1146880}\theta^{6}
$$

The Taylor series expression for $\frac{1}{\theta}\tan^{-1}\left(\frac{\theta}{r_{0}}\right)$
is given in [Table of Taylor Series Expansions](/lie_algebra_tutorial/09-lie_group_tables/). It should be noted that
this equation is highly non-linear, which is why for this series I
have added the 6th-order term.

[Next: The Adjoint](/lie_algebra_tutorial/07-adjoint_as_jacobian/)
