---
type: post
title:  "Part 4: Rigid-Body Transforms"
date:   2020-04-18 13:07:00 -0600
categories: math
tags: ["vectors", "notation", "geometry", "rotations", "transforms"]
---

This is the fourth of a 8-part series of posts designed as a quick-start guide for students new
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

# Transformations

Now we venture into the world of rigid body transformations. These
allow us to represent both translation and rotation simultaneously.
As with rotations, there are two popular representations that I will
consider in this document, the first is matrix-based, the set of $4\times4$
matrices, commonly known as $\SE\left(3\right)$. The second (like
rotations) will be based on complex numbers, dual unit quaternions.
There are actually other representations, but we will see later all
of these groups share the same Lie Algebra! Therefore, we can choose
the representation that we feel most comfortable with, or is the most
efficient. It really doesn't matter, which is a great thing.

## Homogeneous Transform Matrices

Homogeneous transforms take the form of $4\times4$ matrices. These
are members of the Special Euclidean group of three dimensions, or
$\SE\left(3\right)$. This is by far the most common representation
in robotics; it's efficient, and the Lie group/Lie algebra is probably
the most straight forward. A homogeneous transform matrix has the
following block structure

$$
T_{a}^{b}=\begin{bmatrix}R_{a}^{b} & \t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix},
$$

where you have the rotation matrix in the upper left corner and the
translation vector in the right column.

Let's walk through in a block-wise format what happens when two transforms
are multiplied together. All we need to do is follow the matrix multiplication
rules and see how the coordinate frames work out. I found this to
be a useful exercise when I first got started. It really hit home
to me how the matrices concatenate in the same way as rotation matrices,
and the potentially confusing parameterization of the translation
vector. This procedure is carried out as:

$$
\begin{align*}
T_{a}^{c} & =T_{b}^{c}\cdot T_{a}^{b}\\\\
 & =\begin{bmatrix}R_{b}^{c} & \t_{c/b}^{c}\\\\
0 & 1
\end{bmatrix}\cdot\begin{bmatrix}R_{a}^{b} & \t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix}\\\\
 & =\begin{bmatrix}R_{b}^{c}R_{a}^{b} & R_{b}^{c}\t_{b/a}^{b}+\t_{c/b}^{c}\\\\
0 & 1
\end{bmatrix}\\\\
 & =\begin{bmatrix}R_{a}^{c} & \t_{c/a}^{c}\\\\
0 & 1
\end{bmatrix}.
\end{align*}
$$

So we see, we get what we expect: all the coordinate frames align
the way we want.

We can also perform the same exercise for taking the inverse. Remember
that the inverse of a matrix is

$$
A^{-1} = \frac{1}{\det\left(A\right)\textrm{Adj}{\left(A\right)}}
$$

where $\textrm{Adj}\left(A\right)$ refers to the adjugate of $A.$ Because
it gets a little nasty, I'm going to skip actually computing the adjugate,
but if you work it out, you can see how this apparently super convenient
result just pops out:

$$
\begin{align*}
\left(T_{a}^{b}\right)^{-1} & =\begin{bmatrix}R_{a}^{b} & \t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix}^{-1}\\\\
 & =\frac{1}{\det\left(T_{a}^{b}\right)}\textrm{Adj}\left(T_{a}^{b}\right)\\\\
 & =\frac{1}{1}\begin{bmatrix}\left(R_{a}^{b}\right)^{\top} & -\left(R_{a}^{b}\right)^{\top}\t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix}\\\\
 & =\begin{bmatrix}R_{b}^{a} & -R_{b}^{a}\t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix}\\\\
 & =\begin{bmatrix}R_{b}^{a} & \t_{a/b}^{a}\\\\
0 & 1
\end{bmatrix}\\\\
 & =T_{b}^{a}.
\end{align*}
$$

Boom.

If you didn't notice, it's really important to note is that the translation
vector is expressed in the *destination* frame ($b$), rather
than the origin frame ($a$) of the rotation matrix $R_{a}^{b}$.
This is sometimes a problem for me, since I personally would prefer
to express both rotation and translation in the same frame, i.e. $R_{I}^{b}$
and $\t_{b/I}^{I}$. Because we can transform either component into
whatever frame we want, this is all a matter of convenience in the
end. But it's important to realize that the translation and rotation
components are expressed in different frames and we may need perform
additional manipulations to get what we want.

## Passive and Active Transformations

The same notion of passive and active transformations discussed with
rotations applies here. To perform a passive transformation with $\SE\left(3\right)$,
we pad the vector to be rotated with an extra $1$ at the bottom,
and multiply by our transform matrix. Just for completeness, let us
walk through what happens when we do this in a block-wise format

$$
\begin{align*}
\r_{b/p}^{b} & =T_{a}^{b}\cdot\r_{a/p}^{a}\\\\
 & =\begin{bmatrix}R_{a}^{b} & \t_{b/a}^{b}\\\\
0 & 1
\end{bmatrix}\cdot\begin{bmatrix}\r_{a/p}^{a}\\\\
1
\end{bmatrix}\\\\
 & =\begin{bmatrix}R_{a}^{b}\r_{a/p}^{a}+\t_{b/a}^{b}\\\\
1
\end{bmatrix}\\\\
 & =\r_{b/p}^{b}.
\end{align*}
$$

This process is also illustrated on the left of [Figure 5](#fig:passive_transform).
Doing this at least once is another exercise that I found to be very
helpful when first trying to use $\SE\left(3\right).$ Similar to
a passive rotation, a passive transformation results in the vector
that points to the same object, but from the perspective of a different
coordinate frame. In our example here, point $p$ does not move, we
just computed how to get to $p$ from the $b$ frame, instead of the
$a$ frame.

<a name="fig:passive_transfrom"></a>
<div class="side_by_side">
	<div class="column">
    <img  src="/lie_tutorial/transform_passive.svg" style="width: 20vw; min-width: 120px;">
    <img  src="/lie_tutorial/transform_active.svg" style="width: 20vw; min-width: 120px;">
	</div>
    <figcaption class="center">Figure 5: Illustration of a passive transformation (left) and an active transformation (right)"</figcaption>
</div>


An *active* transformation is what we use if we want to move
$p.$ We can see what this looks like on the right of [Figure 5](#fig:passive_transform).
To do this, all we do is multiply our padded vector by the inverse
of our transformation matrix. This is much more common in computer
graphics than in robotics, and as with the active rotation case, our
notation is not well-suited for this operation. However, this can
be a valid operation, depending on the circumstance.

Transform matrices are a natural extension to rotation matrices. However,
just like rotation matrices, the complex number representation is
more efficient. This brings us to our next transform parameterization:
dual unit quaternions.

## Dual Unit Quaternions

The use of dual unit quaternions is another approach to representing
a rigid body transform that relies on complex numbers. As one might
expect, it can also be considered as two copies of $\SU\left(2\right)$
as well. Dual quaternions are more popular in the computer graphics
and physics community, and for the same reasons that quaternions often
out-perform rotation matrices, dual quaternions are often yield the
most efficient representation for rigid body transformations.

Before I jump into dual quaternions, I'm going to first introduce
dual numbers. Dual numbers are similar in many ways to imaginary numbers.
The idea is that we define a scalar $\epsilon$ as a mathematical
construct that splits a numerical representation into two components.
For example, we might have the dual number

$$
z=r+d\epsilon,
$$

where $r$ is the real part, and $d$ is the dual part. We also define
$\epsilon$ as having special rules where $\epsilon^{2}=0,$ but $\epsilon\neq0.$
Again, this is similar to complex numbers where we use $i$ to distinguish
imaginary components from real components. The dual operator $\epsilon$
is used in much the same way. It should be noted that $\epsilon$
in this context is just a special symbol. It is *not* related
to small perturbations, as it is used in other areas of mathematics.
Trying to interpret it in this way can sometimes be confusing.

Dual numbers follow these basic rules. If two dual numbers are added,
then the real part and dual part are simply summed separately (just
like imaginary numbers). For example,

$$
x=a+b\epsilon,\quad y=c+d\epsilon
$$

$$
x+y=\left(a+c\right)+\left(b+d\right)\epsilon.
$$

Multiplication also is performed similarly to complex multiplication,
but with the rule that $\epsilon^{2}=0$. For example,

$$
\begin{align*}
x\cdot y & =ac+ad\epsilon+bc\epsilon+bd\epsilon^{2}\\\\
 & =ac+\left(ad+bc\right)\epsilon.
\end{align*}
$$

Finally, dual variables have their own concept of conjugate, given
in the expected way and denoted as $\left(\cdot\right)^{*}.$ For
example,

$$
\begin{align*}
z & =r+d\epsilon\\\\
z^{*} & =r-d\epsilon.
\end{align*}
$$

If $r$ and $d$ are also complex variables, then we actually have
three forms of conjugation. Imaginary conjugation, dual conjugation,
and imaginary-dual conjugation, given as

$$
\begin{align*}
\bar{z} & =\bar{r}+\bar{d}\epsilon & \text{imag}\\\\
z^{*} & =r-d\epsilon. & \text{dual}\\\\
\bar{z}^{*} & =\bar{r}-\bar{d}\epsilon. & \text{imag+dual}
\end{align*}
$$

Now we are ready to talk about dual quaternions, $\mathbb{D}\S^{3}$
and $\mathbb{D}\SU\left(2\right)$. [^1]
We can represent 3D transforms with the dual unit quaternion pair

[^1]: The $\mathbb{D}$ here refers to the dual formulation.

$$
\qq=\q_{r}+\epsilon\q_{d},
$$

where $\q_{r}$ is called the real part and $\q_{d}$ is the dual
part. The real part contains the rotation, while the dual part is
one-half the pure quaternion containing the translation half rotated,
as in[^2].

[^2]: This may seem like a weird parameterization, and some (like myself)
may question if there is a simpler way, rather than handling this
half-rotated vector. It comes from using the simplest form of the
underlying Lie Algebra, and finding the natural expression of that
group in dual quaternions. In my experience, trying to form a convenient
group and mangling the underlying algebra to match, while certainly
possible, results in losing a lot of the beauty of Lie Theory. It's
better to just use the natural expression, starting at the algebra
and working up.

$$
\begin{align*}
\q_{r} & =\q_{a}^{b}\\\\
\q_{d} & =\frac{1}{2}\begin{pmatrix}0\\\\
\t_{b/a}^{a}
\end{pmatrix}\cdot\q_{a}^{b}.
\end{align*}
$$

If we combine the quaternion algebra operations with the dual number,
we get dual quaternion arithmetic. Dual quaternion multiplication
is given as one might expect, with

$$
\qq_{1}\circ\qq_{2}=\q_{1r}\cdot\q_{2r}+\left(\q_{1r}\cdot\q_{2d}+\q_{1d}\cdot\q_{2r}\right)\epsilon.
$$
Let's walk through this together and make sure the coordinate frames
all line up. Before we do this, however, we need the following trick.
If we take the quaternion rotation formula, and split it up, we get
an expression for dealing with half-rotated vectors:

$$
\begin{align}
\t^{b} & =\left(\q_{a}^{b}\right)^{-1}\cdot\t^{a}\cdot\q_{a}^{b}\nonumber \\\\
\q_{a}^{b}\cdot\t^{b} & =\q_{a}^{b}\cdot\left(\q_{a}^{b}\right)^{-1}\cdot\t^{a}\cdot\q_{a}^{b}\nonumber \\\\
\q_{a}^{b}\cdot\t^{b} & =\t^{a}\cdot\q_{a}^{b}.\label{eq:quat_switch_trick}
\end{align}
$$

Now can work through the dual quaternion arithmetic, and see how dual
quaternion multiplication results in concatenating transforms.

$$
\begin{align*}
\qq_{a}^{b}\circ\qq_{b}^{c} & =\q_{a}^{b}\cdot\q_{b}^{c}+\frac{\epsilon}{2}\left(\q_{a}^{b}\cdot\begin{pmatrix}0\\\\
\t_{c/b}^{b}
\end{pmatrix}\cdot\q_{b}^{c}+\begin{pmatrix}0\\\\
\t_{b/a}^{a}
\end{pmatrix}\cdot\q_{a}^{b}\cdot\q_{b}^{c}\right)\\\\
 & =\q_{a}^{c}+\frac{\epsilon}{2}\left(\begin{pmatrix}0\\\\
\t_{c/b}^{a}
\end{pmatrix}\cdot\q_{a}^{b}\cdot\q_{b}^{c}+\begin{pmatrix}0\\\\
\t_{b/a}^{a}
\end{pmatrix}\cdot\q_{a}^{b}\cdot\q_{b}^{c}\right) & \grey{\textrm{(Eq. \ref{eq:quat_switch_trick})}}\\\\
 & =\q_{a}^{c}+\frac{\epsilon}{2}\left(\begin{pmatrix}0\\\\
\t_{c/a}^{a}
\end{pmatrix}\cdot\q_{a}^{c}\right).
\end{align*}
$$

Inverting (or computing the complex conjugate of) a dual quaternion
is as simple as inverting the two components. To work through this
operation, line by line, we will also need to use the trick from Eq.
\ref{eq:quat_switch_trick}.

$$
\begin{align*}
\left(\qq_{a}^{b}\right)^{-1} & =\left(\q_{a}^{b}\right)^{-1}+\frac{\epsilon}{2}\left(\begin{pmatrix}0\\\\
\t_{b/a}^{a}
\end{pmatrix}\cdot\q_{a}^{b}\right)^{-1}\\\\
 & =\q_{b}^{a}+\frac{\epsilon}{2}\left(\left(\q_{a}^{b}\right)^{-1}\cdot\begin{pmatrix}0\\\\
\t_{b/a}^{a}
\end{pmatrix}^{-1}\right)\\\\
 & =\q_{b}^{a}+\frac{\epsilon}{2}\left(\q_{b}^{a}\cdot\begin{pmatrix}0\\\\
\t_{a/b}^{a}
\end{pmatrix}\right)\\\\
 & =\q_{b}^{a}+\frac{\epsilon}{2}\left(\begin{pmatrix}0\\\\
\t_{a/b}^{b}
\end{pmatrix}\cdot\q_{b}^{a}\right) & \grey{\textrm{(Eq. \ref{eq:quat_switch_trick})}}\\\\
 & =\qq_{b}^{a}
\end{align*}
$$

Transforming a vector is done by creating a pure quaternion (with
no rotational component) from the translation vector, then pre- and
post-multiplying by the dual quaternion and its dual-complex conjugate.
If we work through the steps we can see how this works:

$$
\begin{align*}
\r_{p/b}^{b} & =\left(\bar{\qq_{a}^{b}}\right)^{*}\circ\r_{p/a}^{a}\circ\qq_{a}^{b}\\\\
 & =\left[\q_{b}^{a}-\frac{\epsilon}{2}\left(\t_{a/b}^{b}\cdot\q_{b}^{a}\right)\right]\circ\left[\boldsymbol{1}+\epsilon\r_{p/a}^{a}\right]\circ\left[\q_{a}^{b}+\frac{\epsilon}{2}\left(\t_{b/a}^{a}\cdot\q_{a}^{b}\right)\right]\\\\
 & =\left[\q_{b}^{a}-\frac{\epsilon}{2}\left(\t_{a/b}^{b}\cdot\q_{b}^{a}\right)\right]\circ\left[\left(\boldsymbol{1}\cdot\q_{a}^{b}\right)+\frac{\epsilon}{2}\left(\boldsymbol{1}\cdot\left(\t_{b/a}^{a}\cdot\q_{a}^{b}\right)+2\r_{p/a}^{a}\cdot\q_{a}^{b}\right)\right]\\\\
 & =\left(\q_{b}^{a}\cdot\q_{a}^{b}\right)+\frac{\epsilon}{2}\left(\q_{b}^{a}\cdot\left(\left(-\t_{b/a}^{a}\cdot\q_{a}^{b}\right)+2\r_{p/a}^{a}\cdot\q_{a}^{b}\right)+\left(\t_{a/b}^{b}\cdot\q_{b}^{a}\right)\cdot\q_{a}^{b}\right)\\\\
 & =\boldsymbol{1}+\frac{\epsilon}{2}\left(\q_{b}^{a}\cdot\left(-\t_{b/a}^{a}\cdot\q_{a}^{b}\right)+\q_{b}^{a}\cdot2\r_{p/a}^{a}\cdot\q_{a}^{b}+\left(\t_{a/b}^{b}\cdot\q_{b}^{a}\right)\cdot\q_{a}^{b}\right)\\\\
 & =\boldsymbol{1}+\frac{\epsilon}{2}\left(-\t_{b/a}^{b}+2\r_{p/a}^{b}+\t_{a/b}^{b}\right)\\\\
 & =\boldsymbol{1}+\frac{\epsilon}{2}\left(2\r_{p/a}^{b}-2\t_{b/a}^{b}\right)\\\\
 & =\boldsymbol{1}+\epsilon\left(\r_{p/b}^{b}\right).
\end{align*}
$$


## Dynamics and Invariance

As with rotations, the same concept of left and right invariance also
applies to transforms. If we parameterize rigid body dynamics in body-centric
coordinates, we get the left-invariant dynamics

$$
\dot{T} _b^I=T _b^I \begin{pmatrix} \skew{ \w _{b/I}^b} & \v _{b/I}^b\\\\
0 & 1
\end{pmatrix}.
$$

and the inertial coordinates have right-invariant dynamics, given
as

$$
\dot{T} _I^b=\begin{pmatrix} -\skew{ \w _{b/I}^b } & -\v _{b/I}^b\\\\
0 & 1
\end{pmatrix}T_I^b
$$

Like ordinary quaternions and rotation matrices, dual quaternions
also concatenate opposite $\SE\left(3\right)$, so the order of the
dynamic equations is also flipped, as in

$$
\begin{align*}
\dot{\qq}_{b}^{I} & =-\frac{1}{2}\left(\begin{pmatrix}0\\\\
\w_{b/I}^{b}
\end{pmatrix}+\epsilon\begin{pmatrix}0\\\\
\v_{b/I}^{b}
\end{pmatrix}\right)\circ\qq_{b}^{I}\left(t_{0}\right)\\\\
\dot{\qq}_{I}^{b} & =\frac{1}{2}\qq_{I}^{b}\circ\left(\begin{pmatrix}0\\\\
\w_{b/I}^{b}
\end{pmatrix}+\epsilon\begin{pmatrix}0\\\\
\v_{b/I}^{b}
\end{pmatrix}\right).
\end{align*}
$$


[Next: Lie Groups](/lie_algebra_tutorial/05-lie_groups/)
