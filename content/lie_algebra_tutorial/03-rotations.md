---
type: post
title:  "Part 3: Rotations"
date:   2020-04-18 13:08:00 -0600
categories: math
tags: ["vectors", "notation", "geometry", "rotations"]
---

This is the third of a 8-part series of posts designed as a quick-start guide for students new
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

# Rotations

There are a couple of common conventions found in different fields
of modern literature when dealing with rotations. This section enumerates
a number of these conventions, the implications of using them, and
serves as a backdrop for much of the later work. It is important to
realize that not all fields of literature consider rotations in the
same way, and even within a single field, different conventions may
apply depending on the source. It's also important to realize that
none of these conventions is necessarily *right*. So long as
we effectively communicate what we are talking about and apply proper
methods mathematically, each of these conventions have different strengths,
which is why they have been adopted differently in different fields.

We will deal with both rotation matrices and unit quaternions, although
I will start with a brief explanation of Euler angles, mainly to argue
for why you shouldn't be using them.

## Euler Angles

Euler angles are often the first method that people are introduced
to when talking about rotations. Euler angles describe rotations about
subsequent axes, and there are actually several variations on the
order of these axes. The most common are the 3-2-1 Euler angles, typically
referred to yaw, pitch and roll angles. Despite Euler angles being
historically significant, when compared with modern methods, they
are usually a poor choice for describing a rotation. Euler angles
do not form a vector space [^1] , are numerically unstable in many real-life situations, and are computationally
inefficient. However, Euler angles are often the most intuitive to
look at, so a common workflow is to convert from rotation matrices
or quaternions to Euler angles for debugging problems or plotting.

[^1]: Euler angles do not follow the rules of associativity, commutativity (if you roll first, then pitch, you get a different rotation than if you first pitched, then rolled) or scalar multiplication (what would it mean to just "double" all my Euler angles?)

For reference, I have included the algorithms for converting rotation
matrices and unit quaternions to Euler angles with $\psi$, $\theta$
$\phi$ (referring to yaw, pitch, and roll, respectively). Note that
in the case that $\theta=\tfrac{\pi}{2},$ we get infinitely many
solutions. This condition is known as gimbal lock, and is a consequence
of the fact that for increasingly large values of pitch, roll and
yaw look more and more alike until they are indistinguishable. While
hitting gimbal lock would surely be catastrophic in an Euler-angle
based system, this problem has real consequences even far from actual
lock. This is because the representation becomes increasingly non-linear
as pitch angle grows. So even if a system never actually hits gimbal
lock, the risk of incurring significant linearization error is a quite
high for even moderate pitch angles.

Rotation matrices and unit quaternions, while perhaps less intuitive,
have none of these weaknesses. They can be easily cast into a vector
space (using their associated Lie algebra, which I will talk about
later), they are computationally efficient, and they are numerically
stable--everywhere. The downside is the intellectual hurdle of jumping
into matrix kinematics. I hope that this document is able to demystify
rotation matrices and unit quaternions to the point that they are
just as easy to use as Euler angles, and that future students don't
wade through the problems of Euler angles any longer than necessary.

<a name="alg:rot2euler"></a>
```py
def rot2Euler(R):
	if R[2,0] == 1 or R[2,0] == -1:
		phi = 0 # Anything, set to 0
		if (R[2,0] == -1):
			theta = pi/2.0
			psi = phi + atan2(R[0,1], R[0,2])
		else:
			theta = -pi/2.0
			psi = -phi + np.atan2(-R[0,1], -R[0,2])
	else:
		theta = -asin(R[2,0])
		psi = atan2((R[2,1]/cos(theta)), R[2,2]/cos(theta))
		phi = atan2((R[1,0]/cos(theta)), R[0,0]/cos(theta))
	return phi, theta, psi
```
<figcaption class="center">Algorithm 1: Computing Euler angles from a rotation matrix</figcaption>


<a name="alg:rot2euler"></a>
```py
def quat2Euler(q):
	s = 2*(q.o*q.y - q.x*q.z)
	if abs(s) > 1:
		theta = pi * sign(s)
		phi = 0 # anything, set to zero
	else:
		theta = asin(s)
		phi = atan2(2*(q.o*q.x + q.y*q.z), 1-2*(q.x**2 + q.y**2))
	psi = atan2(2*(q.o*q.z + q.x*q.y), 1-2*(q.y**2 + q.z**2))
	return phi, theta, psi

```
<figcaption class="center">Algorithm 2: Computing Euler angles from a unit quaternion</figcaption>


## Rotation Matrices

A rotation matrix is a member of the special orthogonal group $\SO\left(3\right)$.
This is the set of real $3\times3$ matrices with determinant $1$.
Another way of saying this, which may be a bit more intuitive, is
that the vectors making up each row and column are orthogonal to one
another, have unit length, and follow the right-hand rule [^2]. These vectors form the primary $x,y,z$ axis of a Cartesian coordinate
frame. These properties also give the result that

[^2]: This just means that if you take the $x$ vector and cross it with the $y$ vector, you get the $z$ vector. This comes from the constraint that $\det\left(A\right)=1.$ A matrix whose columns followed the left-hand rule would have a determinant -1

$$
RR^{\top}=I\quad\forall R\in\SO\left(3\right).
$$

The columns of $R_{a}^{b}$ contain the $x,y,z$ vectors of the $b$
frame, expressed in the $a$ frame, while the rows of the matrix contain
the basis vectors of the $a$ frame, expressed in the $b$ frame.
This makes this matrix the perfect mapping between $a$ space and
$b$ space because it directly encodes the change of basis from one
coordinate frame to another. This gives rise to the definition given
earlier:

$$
\begin{equation}
\r^{b}=R_{a}^{b}\r^{a},\label{eq:rot_formula}
\end{equation}
$$

and the fact that rotation matrices compound from right to left, as
in

$$
R_{a}^{c}=R_{b}^{c}R_{a}^{b}.
$$


## Passive Versus Active Rotations

<a name="fig:passive_rotation"></a>
<div class="side_by_side">
	<div class="column">
    <img  src="/lie_tutorial/rotation_passive.svg" style="width: 20vw; min-width: 120px;">
    <img  src="/lie_tutorial/rotation_active.svg" style="width: 20vw; min-width: 120px;">
	</div>
    <figcaption class="center">Figure 4: Illustration of a passive rotation (left) and an active rotation (right)</figcaption>
</div>

We are using a convention here, where we define $R$ as a *passive*
rotation. Consider Equation \ref{eq:rot_formula}. The vector quantity
$\r$ did not change during this rotation, it is simply being expressed
from another point of view. Some literature defines rotations as *active*.
[Figure 4](#fig:passive_rotation) illustrates the difference between
active and passive rotations. In the passive case (left), the point
$p$ does not change when rotating from frame $a$ to $b$. It remains
fixed while the perspective changes. In contrast to the active case
(right), point $p$ is moved to become a new object, point $p^{\star}$.
This interpretation is much more common in the computer graphics literature,
where a common operation is to generate some object at an arbitrary
origin, and then move that object to be rendered at some other location.
Engineering and robotics literature typically deal with passive rotations,
where the goal is often to reason about physical entities that cannot
be arbitrarily moved.

Confusingly, both kinds of rotations are typically denoted simply
as $R$, and readers are expected to infer passive or active convention.
However, active and passive rotations are actually the transpose of
one another, as

$$
R_{\text{active}}=R_{\text{passive}}^{\top}.
$$

This fact makes literature which may use one or the other convention
quickly accessible to any and all readers who know the rules and make
note of the choice in convention.

Readers should also note that our notation is not well-suited for
active transformations. If you try to perform an active rotation in
our notation, the coordinate frames will not line up, which can be
very confusing. This is a potential shortcoming of this notation if
the desire is to perform a lot of active transformations.

While rotation matrices are probably the most straight forward method
to represent rotations, they have some shortcomings. The first is
that they require nine parameters to describe three degrees of freedom.
This causes larger memory requirements than strictly necessary, and
requires more operations than necessary when concatenating matrices
With modern computing resources, this is less of a problem, however
the biggest disadvantage is that numerical errors can accumulate in
rotation matrices and cause unwanted scaling and shearing as the matrix
loses orthonormality. This error can be corrected by Gram-Schmidt
orthogonalization, but doing so can be computationally expensive.
As a result, rotation matrices are still much more efficient than
an Euler angle representation, but they are not as efficient as unit
quaternions.

## Unit Quaternions

A quaternion, is a hyper-complex number with four elements, commonly
written [^3]

[^3]: Sometimes $q_{w}$ is used instead of $q_{o}$.

$$
\q=q_{0}+q_{x}i+q_{y}j+q_{z}k.
$$

Here you can see two additional imaginary numbers, $j$ and $k$.
These new imaginary numbers follow the following rules

$$
i^{2}=j^{2}=k^{2}=ijk=-1,
$$

as well as the right-hand rule

$$
ij=k,\quad ji=-k,
$$

and so on, which gives rise to the definition of quaternion multiplication
as[^4]

[^4]: To derive this yourself, all you have to do is distribute out the
product and apply the rules above.

$$
\q^{a}\cdot\q^{b}=\begin{array}{c}
q_{0}^{a}q_{0}^{b}-q_{x}^{a}q_{x}^{b}-q_{y}^{a}q_{y}^{b}-q_{z}^{a}q_{z}^{b}\\\\
{\displaystyle +\left(q_{0}^{a}q_{x}^{b}+q_{x}^{a}q_{0}^{b}+q_{y}^{a}q_{z}^{b}-q_{z}^{a}q_{y}^{b}\right)i}\\\\
+\left(q_{0}^{a}q_{y}^{b}-q_{x}^{a}q_{z}^{b}+q_{y}^{a}q_{0}^{b}+q_{z}^{a}q_{x}^{b}\right)j\\\\
+\left(q_{0}^{a}q_{z}^{b}+q_{x}^{a}q_{y}^{b}-q_{y}^{a}q_{x}^{b}+q_{z}^{a}q_{0}^{b}\right)k.
\end{array}
$$

When representing rotations, quaternions must also satisfy the additional
constraint that the $l_{2}$ norm of all four elements is equal to
$1,$ hence the specification *unit* quaternions, and the use
of the group $\S^{3}$, the unit sphere in four dimensions (and has
three degrees of freedom).

Unit quaternions, unlike rotation matrices, multiply from left to
right, as in [^5]

[^5]: Because of the reverse order of concatenation, there is actually another convention for quaternions, first used by NASA JPL, where the quaternion imaginary numbers follow the *left-hand* rule instead of the right hand rule. This change makes unit quaternions concatenate right-to-left (to match rotation matrices). The left-handed definition is known as JPL notation, whereas the right-handed definition presented here is known as Hamilton notation. While this change may seem innocent on the surface, it has huge implications when we start talking about the associated Lie theory. Modern scientific literature in robotics and computer vision that use quaternions seem to almost always use Hamilton notation, although JPL is used in some early quaternion papers, mostly related to spacecraft attitude observers.

$$
\q_{a}^{c}=\q_{a}^{b}\cdot\q_{b}^{c}.
$$

For convenience, we will sometimes refer to the complex portion of
the quaternion as the quantity[^6]

[^6]: This is also often denoted with the *bar* notation $\bar{\q}$, however we are using this notation to mean the complex conjugate, as used in other literature involving complex numbers, so we use the *vect* notation here. Likely for historical reasons, a lot of literature refers to this imaginary portion as the *vector part.*

$$
\vec{\q}=\left[\begin{array}{ccc}
q_{x} & q_{y} & q_{z}\end{array}\right]^{\top}
$$

and write quaternions as the tuple of the real and complex portion,
as in

$$
\q=\left(\begin{array}{c}
q_{0}\\\\
\vec{\q}
\end{array}\right).
$$

This means that the somewhat unwieldy quaternion multiplication expression
above can be re-written as the matrix-like operation

$$
\begin{equation}
\q^{a}\cdot\q^{b}=\left(\begin{array}{cc}
-q_{0}^{a} & \left(-\vec{\q}^{a}\right)^{\top}\\\\
\vec{\q}^{a} & q_{0}^{a}I+\skew{\vec{\q}^{a}}
\end{array}\right)\left(\begin{array}{c}
q_{0}^{b}\\\\
\vec{\q}^{b}
\end{array}\right).\label{eq:quat_mult_matrix}
\end{equation}
$$

Splitting the quaternion into a vector and scalar portion also allows
us to interpret the quaternion in an axis-angle fashion. Given a unit
vector describing an axis of rotation $\r$ and angle about that axis
$\theta$ in radians, the equivalent quaternion is given as

$$
\begin{equation}
\vec{\q}=\r\sin\left(\frac{\theta}{2}\right),\quad q_{0}=\cos\left(\frac{\theta}{2}\right).\label{eq:quat_axis_angle}
\end{equation}
$$

This axis-angle interpretation makes it somewhat feasible to interpret
a unit quaternion by just looking at numbers. The axis-angle interpretation
also makes obvious that inverting a quaternion is the same as computing
the complex conjugate

$$
\q^{-1}=\bar{\q}=\begin{pmatrix}q_{0}\\\\
-\vec{\q}
\end{pmatrix}.
$$

There is actually another interesting thing about quaternions we can
see in Eq. \ref{eq:quat_axis_angle}. If you think about it, you can
actually see that unit quaternions are a double cover[^7] of $\SO\left(3\right).$ This is because for each $\left(\r,\theta\right)$,
$\left(-\r,-\theta\right)$ describes the same rotation.

[^7]: This just means that for every rotation matrix, there are two unit quaternions.

Finally, to convert from a quaternion to a rotation matrix, we employ
the following expression

$$
R\left(\q\right)=\left(2q_{0}-1\right)I-2q_{0}\skew{\vec{\q}}+2\vec{\q}\vec{\q}^{\top},
$$

and to convert a rotation matrix into its quaternion expression, we
can use [Algorithm 3](#alg:rot2quat).


<a name="alg:rot2quat"></a>
``` py
def rot2quat(R):
	tr = trace(R)

	if (tr > 0)
	{
		S = sqrt(tr + 1.0) * 2.0
		q = [0.25 * S, (R[1,2]-R[2,1]) / S, (R[2,0]-R[0,2]) / S, (R[0,1]-R[1,0]) / S]
	}
	else if ((R[0,0] > R[1,1]) && (R[0,0] > R[2,2]))
	{
		S = sqrt(1.0 + R[0,0]-R[1,1]-R[2,2]) * 2.0
		q=  [(R[1,2]-R[2,1]) / S, 0.25 * S, (R[1,0]+R[0,1]) / S, (R[2,0]+R[0,2]) / S]
	}
	else if (R[1,1] > R[2,2])
	{
		S = sqrt(1.0+R[1,1]-R[0,0]-R[2,2]) * 2.0
		q = [(R[2,0]-R[0,2]) / S, (R[1,0]+R[0,1]) / S, 0.25 * S, (R[2,1]+R[1,2]) / S]
	}
	else
	{
		S = sqrt(1.0+R[2,2]-R[0,0]-R[1,1]) * 2.0
		q = [(R[0,1]-R[1,0]) / S, (R[2,0]+R[0,2]) / S, (R[2,1]+R[1,2]) / S, 0.25 * S]
	}
	return q
```
<figcaption class="center">Algorithm 3: Algorithm for converting rotation matrix to quaternion</figcaption>




There are couple of ways to rotate a vector with a quaternion. The
first, is to compute the rotation matrix from the quaternion and then
use it to rotate the vector, as in

$$
\begin{equation}
\r^{b}=R\left(\q_{a}^{b}\right)\r^{a}.\label{eq:rot_matrix_multipliy}
\end{equation}
$$

This, however, is inefficient. Another way to rotate a vector is with
quaternion multiplication. We pad the vector with an extra zero in
the real component[^8] and perform post- and pre-multiplication by the quaternion and its
complex conjugate, as in

[^8]: A quaternion with no real component is known as a *pure quaternion.*

$$
\begin{equation}
\r^{b}=\left(\q_{a}^{b}\right)^{-1}\cdot\begin{pmatrix}0\\\\
\r^{a}
\end{pmatrix}\cdot\q_{a}^{b}.\label{eq:quat_rotation}
\end{equation}
$$

While this is more efficient than first computing the rotation matrix
and then performing the matrix-vector multiplication there is quite
a bit of redundant effort. If you remove this redundant effort, then
both Eq. \ref{eq:quat_rotation} and Eq. \ref{eq:rot_matrix_multipliy}
condense into the following expression:


$$
\begin{align*}
\r^{b} & =\textrm{rot}\left(\q_{a}^{b},\r^{a}\right)\\\\
       & =\r^{a}+q_{0}\mathbf{t}+\skew{\mathbf{t}}\vec{\q}_{a}^{b},\qquad\mathbf{t}=2\skew{\r^{a}}\vec{\q}_{a}^{b}.
\end{align*}
$$

If implemented efficiently, this method can be just as efficient as
the matrix-vector multiplication used when rotating a vector with
a rotation matrix. [^9]

[^9]: When implementing on hardware with AVX2 extensions (Intel and AMD cores produced after 2016), an entire double-precision quaternion can fit in the processors vector register. This, coupled with the vector complex number extensions makes usually makes rotating a vector with a quaternion *even more efficient than matrix-vector multiplication.*

## Relationship between $\SU\left(2\right)$ and Unit Quaternions

Unit quaternions are isomorphic[^10] to $2\times2$ complex matrices with $\det=1$. If we replace the
imaginary numbers with the following matrices,

[^10]: This means there for every unit quaternion, there is exactly one member of $\SU\left(2\right)$.

$$
\begin{align*}
\mathbf{1} & =\begin{pmatrix}1 & 0\\\\
0 & 1
\end{pmatrix},\qquad\qquad\i=\begin{pmatrix}0 & 1\\\\
-1 & 0
\end{pmatrix}\\\\
\j & =\begin{pmatrix}0 & i\\\\
i & 0
\end{pmatrix},\qquad\qquad\k=\begin{pmatrix}i & 0\\\\
0 & -i
\end{pmatrix}
\end{align*}
$$

then we can re-write the quaternion as the following $2\times2$ matrix:

$$
\begin{align*}
\q & =q_{0}\boldsymbol{1}+q_{x}\i+q_{y}\j+q_{z}\k\\\\
 & =\begin{pmatrix}q_{0}+q_{z}i & q_{x}+q_{y}i\\\\
-q_{x}+q_{y}i & q_{0}-q_{z}i
\end{pmatrix}.
\end{align*}
$$


We can see that this representation still follows the original rules
for the unit quaternion ($\i\j\k=-\boldsymbol{1}$), and instead of
unit norm, the matrix has unit determinant and we get the additional
constraint, $\q^{\dagger}\q=1$. This means that unit quaternions
when written as $2\times2$ matrices fulfill all the conditions of
$\SU\left(2\right)$, the special unitary group. This will show up
again when we talk about Lie groups.

## Body-Centric versus Inertial Representation

In robotics or computer vision, we are typically concerned with rotations
so that we can describe the movement of physical objects. When we
do this, it turns out that choice of coordinate frame also introduces
some interesting consequences that may or may not be intentional.
Rotational dynamics of any arbitrary coordinate frame $j$ with respect
to another coordinate frame $i$ is given by


$$
\dot{R} _i^{j}=R_i^{j} \skew{ \w _{i/j}^i}.
$$

However, let's study a concrete example: the dynamics of some rigid
body undergoing pure rotation. If we define some inertial frame $I$
and the moving, body frame as $b$, then the dynamics of the *body-centric*
rotation matrix (i.e. body$\to$inertial) are given by

$$
\begin{align}
\dot{R} _b^I=R_b^I \skew{ \w _{b/I}^b }, \label{eq:body_rot_dyn}
\end{align}
$$


and the *inertial* rotation matrix is (inertial$\to$body) evolves
according to

$$
\begin{align}d
\dot{R} _I^b=R _I^b \skew{ \w _{I/b}^I }. \label{eq:inertial_rotation_dynamics}
\end{align}
$$

This is all fine, except that we typically measure angular velocity
in the body coordinate frame $\w_{b/I}^{b}.$ Therefore, it is much
more convenient to express Eq. \ref{eq:inertial_rotation_dynamics}
in terms of body angular rates. If we do this, then we get

$$
\begin{align*}
\dot{R}_{I}^{b} & =R_{I}^{b}\skew{-R_{b}^{I}\w_{b/I}^{b}} & \grey{(\text{Negate and rotate } \w)}\\\\
 & =-\skew{\w_{b/I}^{b}}R_{I}^{b},
\end{align*}
$$

which can also be easily derived as the transpose of Eq \ref{eq:body_rot_dyn}.
The desire to express our angular rates in terms of the body frame
induces a similar effect in unit quaternion dynamics, except that
it is reversed due to the reversed order of concatenation:

$$
\begin{align}
\dot{\q} _I^b & =\frac{1}{2}\q _I^b\cdot\begin{pmatrix}0\\\\
\w _{b/I}^b
\end{pmatrix}\label{eq:quaternion_dynamics}\\\\
\dot{\q} _b^I & =-\frac{1}{2}\begin{pmatrix}0\\\\
\w _{b/I}^b
\end{pmatrix}\cdot\q _b^I.\nonumber
\end{align}
$$

Any advantage to expressing attitude in body-centric vs. inertial
coordinates will be use-case specific, so both representations are
used widely in literature. Because both representations are often
used, however, it can sometimes be confusing to compare two works
which may be expressing their attitude differently, especially if
neither work is specific about their representation.

## Right versus Left Invariance of Rotation Dynamics

Another non-trivial consequence of differing attitude representations
is whether the kinematics are left or right-invariant (LI or RI).
Body-centric rotation kinematics are left-invariant. This means that
if we have some constant matrix $A\in\SO\left(3\right)$, and we multiply
the kinematics (Eq. \ref{eq:body_rot_dyn}) by $A$ on the left, we
get a new differential equation with the same form as Eq. \ref{eq:body_rot_dyn}.

$$
\frac{d}{dt}\left(AR_{b}^{I}\right)=AR_{b}^{I}\skew{\w_{b/I}^{b}}
$$

One can see that we do not get a differential equation of the same
form if we multiply both sides of Eq. \ref{eq:body_rot_dyn} on the
*right* by $A.$

Inertial rotation kinematics, on the other hand, are right-invariant.
If we multiply both sides of Eq. \ref{eq:inertial_rotation_dynamics}
on the right by $A$, we get

$$
\frac{d}{dt}\left(R_{I}^{b}A\right)=-\skew{\w_{b/I}^{b}}R_{I}^{b}A,
$$

which has the same form as Eq. \ref{eq:inertial_rotation_dynamics}.
A similar effect can be observed in quaternion dynamics, but with
the order reversed.

The left-right invariance shows up because of our choice in representing
$\w$ in the body frame. I know this is jumping ahead a bit, but the
Adjoint representation will allow us to flip back and forth between
left- and right-vectors as necessary, making this distinction also
a matter of convenience.

## The Solution to the Rotational Dynamics Equation

Equations \ref{eq:body_rot_dyn} and \ref{eq:inertial_rotation_dynamics}
are first-order matrix differential equations. These are solved with
the matrix exponential we discussed in the previous Section
The solution to the body-centric equation is

$$
R_{b}^{I}\left(t\right)=R_{b}^{I}\left(t_{0}\right)\exp\left(\skew{\w_{b/I}^{b}}t\right)
$$

and the inertial dynamics are solved with

$$
R_{I}^{b}\left(t\right)=\exp\left(-\skew{\w_{b/I}^{b}}t\right)R_{I}^{b}\left(t_{0}\right).
$$

Quaternion dynamics are also easily solved in a similar fashion, as
in

$$
\begin{align*}
\q_{b}^{I}\left(t\right) & =\exp\begin{pmatrix}0\\\\
-\w_{b/I}^{b}t
\end{pmatrix}\cdot\q_{b}^{I}\left(t_{0}\right)\\\\
\q_{I}^{b}\left(t\right) & =\q_{I}^{b}\left(t_{0}\right)\cdot\exp\begin{pmatrix}0\\\\
\w_{b/I}^{b}t
\end{pmatrix}.
\end{align*}
$$

The infinite series form of the matrix exponential [here](matrix_exp.html#eq:mat_exp_inf_anchor) can be used to compute
these directly. However, efficient closed-form implementations are
derived in [Section 6](/lie_algebra_tutorial/06-closed_form_mat_exp)

[Next: Rigid Body Transforms](/lie_algebra_tutorial/04-transformations/)
