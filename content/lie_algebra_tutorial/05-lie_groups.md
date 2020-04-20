---
type: post
title:  "Part 5: Lie Groups"
date:   2020-04-18 13:06:00 -0600
categories: math
tags: ["vectors", "notation", "geometry", "rotations", "lie"]
---

This is the fifth of a 8-part series of posts designed as a quick-start guide for students new
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

# Lie Groups

Lie group theory is a mature discipline deeply rooted in mathematics
and theoretical physics. It was developed to better understand and
model nature at the most fundamental levels. In fact, much of particle
physics can be modeled using Lie group theory, including special relativity
and quantum dynamics. While we don't often need to properly model
Minkowski space, we can learn a lot about modeling physical systems
by piggybacking on physics literature.

If you can't tell from the last 4 sections, we end up doing a lot
of reasoning about coordinate frames in robotics. We often need to
reason about how they move, and how they relate to one another. We
also find ourselves trying to infer things about these coordinate
frames given noisy sensor measurements. There are a variety of ways
that we could do this, but the real problems show up when we need
to this in real time under realistic computational limitations.

To meet actual computational requirements, we generally want to be
able to leverage linear algebra and all of its tools. Unfortunately,
however, coordinate frame transformations are not vectors, so it's
not completely obvious how we can use these tools in a principled
manner. Luckily, we have Lie group theory that can bridge this gap.
We will see in this section how we can use Lie groups to take non-vector
group objects, and map them into a vector space where we can perform
linear algebra and reason about things in an efficient manner, then
map our results back into the group so we can do useful things.

As a further motivating example, consider the case where we might
want to represent our uncertainty of some estimate of a rotation matrix.
In general, assuming that our uncertainty of some vector quantity
$\x$ can be represented with a multivariate Gaussian distribution,
the covariance of this distribution is computed as

$$
\Sigma_{\x}=E\left[\left(\x-\hat{\x}\right)\left(\x-\hat{\x}\right)^{\top}\right],
$$

where $\hat{\x}$ was our best estimate and $\x$ was the true value.
However, if we consider putting our rotation matrix $R$ into this
equation, what does $R-\hat{R}$ even mean? Subtraction is not a sensible
operation when talking about rotation matrices. First off, it would
no longer be a rotation matrix, as it would not longer have unit determinant,
and second, it would have nine parameters, when we expect there to
only be three[^1]. Lie group theory solves this problem for us, as we will soon find
out.

[^1]: Using Euler angles doesn't get you out of this, either. It's just more subtle. Think about what it really means to subtract two "vectors" of Euler angles. The intermediate axes defining the Euler angle parameterization will not line up, so it's not a sensible operation.

## Group Theory

Before we launch into Lie groups, let us consider groups generically.
A group is simply a set of members which follow the set of four axioms.
The first is that they are *closed* under some group operator.
This means that if I had two group members $g_{1}$ and $g_{2}$ and
I act on these members with the group operator $\circ$, then the
result should also be a part of the group. For example,

$$
a\circ b=c,\qquad\qquad\forall a,b,c\in G.
$$

Secondly, groups are associative, as in

$$
\left(a\circ b\right)\circ c=a\circ\left(b\circ c\right),\qquad\qquad\forall a,b,c\in G
$$
and there must exist some *identity* element, as
$$
g\circ I=g,\qquad\qquad\forall g\in G.
$$
Finally, each member of the group must have a corresponding *inverse* such that

$$
g\circ g^{-1}=I\qquad\qquad\forall g\in G.
$$

We use groups all the time, maybe without knowing it. Some commonly
used groups in robotics are vectors (closed under vector addition),
rotation matrices (closed under matrix multiplication), and quaternions
(closed under quaternion multiplication). Some less-commonly used
groups could include binary vectors closed under bit-wise-XOR, positive
definite matrices closed under matrix addition and integers closed
under addition[^2].

[^2]: In case you need another reason to stop using Euler angles, Euler
angles aren't even a group, unless, of course, the group operator
is defined as ``convert to rotation matrix, multiply, and then convert
back to Euler angles''}

## Lie Group Definition

A Lie group is a group that is also a differentiable manifold whose
operator $\circ$ and inverse are smooth maps. What this means is
that every group element $A$ induces a map from one group element
$B$ to another $C$ via some operator and that this mapping
is differentiable everywhere. For example, let us say we have

$$
C=A\circ B
$$

where $A$, $B$, and $C$ all belong to the same group $G$ with
$\circ$ as the group operator. $G$ would be a Lie group if and only
if there is some way we can express infinitesimal changes in $C$
with respect to infinitesimal changes in $B$ for any $A$ or $B$
in the group.

Of all the groups mentioned above, vectors, rotation matrices, quaternions
and positive definite matrices are also Lie groups, while binary vectors
closed under XOR and integers are not, because the mappings are not
differentiable[^3]

[^3]:These are obvious cases because there is no notion of an "infinitesimal change" of an integer or binary vector, however there are other cases that are less obvious, such as the unit sphere in three dimensions which differentiable everywhere except a single point. (See the ``hairy ball theorem'').

Each Lie groups has an associated Lie algebra, typically indicated
by using the fraktur font, (e.g. $\so\left(3\right)).$ A Lie algebra
is a vector space $\mathfrak{g}$ equipped with a binary operation
called the *Lie bracket,* $[\cdot,\cdot]$. For matrix Lie algebras,
the Lie bracket is given as

$$
\left[A,B\right]=AB-BA.
$$

The Lie bracket abides by the following rules:

 * Bilinearity: $\left[aX+bY,Z\right]=a\left[X,Z\right]+b\left[Y,Z\right]$
 * Anticommutativity: $\left[X,Y\right]=-\left[Y,X\right]\quad\forall X,Y\in\mathfrak{g}$
 * The Jacobi Identity: $\left[X,\left[Y,Z\right]\right]+\left[Z,\left[X,Y\right]\right]+\left[Y,\left[Z,X\right]\right]=0\quad\forall X,Y,Z\in\mathfrak{g}$

One of the fundamental tenants of Lie theory is that every member
of a Lie group can be formed by taking the exponential map of the
Lie algebra, that is, we can form every element of our Lie group $A\in G$,
by passing *at least* one member of the Lie algebra through the
exponential map

$$
A=\exp\left(\a\right)\quad a\in\mathfrak{\mathfrak{g}},\:A\in G.
$$

When working in a Lie algebra we also have something analogous to
the basis of the Lie group, except instead of basis vectors, we have
*generators*. The generators are the building blocks of the group,
and typically encode the degrees of freedom in a orthonormal way.
Since the Lie algebra is a vector space, every element of the algebra
can be represented as linear combination of the generators, therefore
each member of the group can be represented as the exponential map
of a linear combination of the generators.

Important note: just as with vector spaces, the generators don't *have*
to be orthonormal. It's often more convenient to write them that way,
but so long as the generators are independent, then we are able to
get to every member of the algebra (and therefore, the group also,
through the exponential map) from a linear combination of the generators.

There is an important theorem that we will use a little later called
the Baker-Campbell-Hausdorff theorem (BCH). This theorem states that
the solution $Z$ to

$$
e^{X}e^{Y}=e^{Z},
$$

can be expressed as a power series involving commutators $X$ and
$Y$ in the Lie bracket. The first couple terms of this series is
given as

$$
Z=X+Y+\frac{1}{2}\left[X,Y\right]+\frac{1}{12}\left[X,\left[X,Y\right]\right]-\frac{1}{12}\left[Y,\left[X,Y\right]\right]+\dots.
$$

This formula is central to many proofs in the Lie group-Lie algebra
correspondence.

## The Generators of $\SO\left(3\right)$

The easiest way to come up with the generators of a Lie group is to
consider the infinitesimal transformations that the group can accommodate.
For example, the generators of $\so\left(3\right)$ are typically
given by thinking about the way a rotation matrix changes as the angle
of rotation approaches zero:

$$
\begin{align*}
J_{1} & =\begin{pmatrix}0 & 0 & 0\\\\
0 & 0 & -1\\\\
0 & 1 & 0
\end{pmatrix} & J_{2} & =\begin{pmatrix}0 & 0 & 1\\\\
0 & 0 & 0\\\\
-1 & 0 & 0
\end{pmatrix} & J_{3} & =\begin{pmatrix}0 & -1 & 0\\\\
1 & 0 & 0\\\\
0 & 0 & 0
\end{pmatrix}.
\end{align*}
$$

However, we can prove that these generators are correct with this
simple exercise. We know that all elements of $\SO\left(3\right)$
must follow these conditions:

$$
\begin{align*}
A^{\top}A & =I & \det\left(A\right) & =1.
\end{align*}
$$

We also know that every element of $\SO\left(3\right)$ can be written
in terms of some linear combination of generators $J$. as in

$$
\begin{align*}
A & =\exp\left(\theta J\right).
\end{align*}
$$

We can back out the structure of $J$ through putting it into the
definition conditions of $\SO\left(3\right)$. The first condition
yields

$$
\begin{align*}
A^{\top}A & =I\\\\
\exp\left(\theta J\right)^{\top}\exp\left(\theta J\right) & =I\\\\
J^{\top}+J & =0.
\end{align*}
$$

If we use the second condition (and the identity $\det\left(\exp\left(A\right)\right)=\exp\left(\textrm{tr}\left(A\right)\right)$)
we see

$$
\begin{align*}
\det\left(A\right) & =1\\\\
\det\left(e^{\theta J}\right) & =1\\\\
\textrm{tr}\left(J\right) & =0.
\end{align*}
$$

So, the generators for the algebra $\so\left(3\right)$ must be anti-symmetric,
traceless $3\times3$ matrices. Three linearly independent matrices
fulfilling these conditions *and* the rules for a lie algebra
are[^3]

[^3]: Specifically, we need generators that obey the right-hand rule through the Lie bracket so they can satisfy the Jacobi Identity.

$$
\begin{align*}
J_{1} & =\begin{pmatrix}0 & 0 & 0\\\\
0 & 0 & -1\\\\
0 & 1 & 0
\end{pmatrix} & J_{2} & =\begin{pmatrix}0 & 0 & 1\\\\
0 & 0 & 0\\\\
-1 & 0 & 0
\end{pmatrix} & J_{3} & =\begin{pmatrix}0 & -1 & 0\\\\
1 & 0 & 0\\\\
0 & 0 & 0
\end{pmatrix}.
\end{align*}
$$

Therefore, these must be valid generators of $\so\left(3\right).$
Note that there is actually an infinite set of valid generators. By
convention we typically choose the simplest generators for the same
reason that we typically use unit vectors to describe coordinate bases.

As mentioned before, all members of a Lie group can be created by
computing the exponential of a linear combination of the generators.
Furthermore, because the generators are orthogonal, $\so\left(3\right)$
is isomorphic to $\R^{3}$. Therefore, to keep things concise, we
can actually form a vector whose coefficients are then multiplied
by the generators. In the case of $\SO\left(3\right)$, this is the
same as computing the skew-symmetric matrix of a vector. But in general,
the $\left(\cdot\right)^{\wedge}$ and $\left(\cdot\right)^{\vee}$
notation is used to indicate this mapping. For example, for $\so\left(3\right):$

$$
\begin{align*}
\left(\v^{\wedge}\right) & =\sum_{i}J_{i}\e_{i}^{\top}\v\\\\
 & =J_{1}\v_{x}+J_{2}\v_{y}+J_{3}\v_{z}\\\\
 & =\begin{pmatrix}0 & -\v_{z} & \v_{y}\\\\
\v_{z} & 0 & -\v_{x}\\\\
-\v_{y} & -\v_{x} & 0
\end{pmatrix},
\end{align*}
$$


$$
\left(\v^{\wedge}\right)^{\vee}=\v.
$$
This notation is used in some literature [Barfoot (2019)](http://asrl.utias.utoronto.ca/~tdb/bib/barfoot_ser17.pdf), [Sola (2019)](https://arxiv.org/abs/1812.01537),
but not in others [Drummond (2014)](http://twd20g.blogspot.com/p/notes-on-lie-groups.html), [Eade (2019)](http://ethaneade.com/lie_groups.pdf). Since the two representations
(the matrix algebra and the vector) are isomorphic, the mapping to
the matrix Lie algebra is often skipped notationally, and $\exp\left(\v\right)$
is written while $\exp\left(\v^{\wedge}\right)$ would technically
be more correct. Since computing the matrix exponential of $\v\in\R^{3}$
is actually not possible given the definition above, the mapping of
the vector to the matrix Lie algebra is implied in literature where
the mapping is ignored.

## The Generators of $\SU\left(2\right)/\S^{3}$

We can follow the same procedure that we performed to derive the generators
of $\SO\left(3\right)$ to get the generators of $\SU\left(2\right)/\S^{3}$.
First, let's start with the conditions of $\SU\left(2\right)$

$$
U^{\dagger}U=UU^{\dagger}=1\qquad\qquad\det\left(U\right)=1.
$$

Now, as before, let us write these conditions in terms of the generators,
and use BCH in tandem with the fact that $\left[U^{\dagger},U\right]=0$
to get

$$
\begin{align}
U^{\dagger}U & =1\nonumber \\\\
\left(\exp\left(iJ\right)\right)^{\dagger}\exp\left(iJ\right) & =1\nonumber \\\\
\exp\left(-iJ^{\dagger}\right)\exp\left(iJ\right) & =1\nonumber \\\\
\exp\left(-iJ^{\dagger}+iJ+\frac{1}{2}\left[J^{\dagger},J\right]\right)+\dots & =1 & \grey{\textrm{(BCH)}}\nonumber \\\\
\exp\left(-iJ^{\dagger}+iJ\right) & =1\nonumber \\\\
-iJ^{\dagger}+iJ & =0\nonumber \\\\
J^{\dagger} & =J.\label{eq:HermetianSU2}
\end{align}
$$

Now, if we look at the second condition, (again, leveraging $\det\left(\exp\left(A\right)\right)=\exp\left(\textrm{tr}\left(A\right)\right)$),
we can see

$$
\begin{align}
\det\left(\exp\left(iJ\right)\right) & =1\nonumber \\\\
\exp\left(i\textrm{tr}\left(J\right)\right) & =1\nonumber \\\\
\textrm{tr}\left(J\right) & =0.\label{eq:tracelessSU2}
\end{align}
$$

So the generators must be Hermetian (Eq. \ref{eq:HermetianSU2}),
and traceless (Eq. \ref{eq:tracelessSU2}) complex $2\times2$ matrices.
The standard basis for this set is

$$
\begin{align*}
J_{1} & =\begin{pmatrix}0 & 1\\\\
1 & 0
\end{pmatrix} & J_{2} & =\begin{pmatrix}0 & -i\\\\
i & 0
\end{pmatrix} & J_{3} & =\begin{pmatrix}1 & 0\\\\
0 & -1
\end{pmatrix}.
\end{align*}
$$

This choice of generators corresponds with infinitesimal rotations
in $\SU\left(2\right),$ and because of the isomorphism between $\SU\left(2\right)$
and $\S^{3}$ (unit quaternions), we have also just derived the generators
for unit quaternions as well:

$$
\begin{align*}
J_{1} & =i & J_{2} & =j & J_{3} & =k.
\end{align*}
$$

This means that the $\left(\cdot\right)^{\wedge}$ operations for
$\SU\left(2\right)$ and $\S^{3}$ are given as

$$
\w^{\wedge}=\begin{pmatrix}\w_{z} & \w_{x}-\w_{y}i\\\\
\w_{x}+\w_{y}i & \w_{z}
\end{pmatrix},
$$

and

$$
\w^{\wedge}=\begin{pmatrix}0\\\\
\w
\end{pmatrix},
$$

respectively, while the $\left(\cdot\right)^{\vee}$operator is defined
as the inverse of either operation.

There is a very interesting relationship between $\su\left(2\right)$
and $\so\left(3\right)$. Both algebras have the following property:

$$
\begin{align*}
\left[J_{1},J_{2}\right] & =J_{3} & \left[J_{2},J_{1}\right] & =-J_{3}\\\\
\left[J_{2},J_{3}\right] & =J_{1} & \left[J_{3},J_{2}\right] & =-J_{1}\\\\
\left[J_{3},J_{1}\right] & =J_{2} & \left[J_{1},J_{3}\right] & =-J_{2}.
\end{align*}
$$

It turns out, this is indicative of a more fundamental result. One
can say that $\SU\left(2\right)$ and $\SO\left(3\right)$ have the
same Lie algebra, because we define Lie algebras by their Lie bracket.
Forgive the extended quotation, but this summary from [*Physics From Symmetry*](https://link.springer.com/book/10.1007/978-3-319-19201-7) explains this concept in
detail, as well as giving a very nice motivation for why we use Lie
Algebras in the first place.

> There is precisely one simply-connected Lie group corresponding to each Lie algebra. This simply-connected group can be thought of as the \textquotedbl mother\textquotedbl{} of all those groups having the same Lie algebra, because there are maps to all other groups with the same Lie algebra from the simply connected group, but not vice versa. We could call it the mother group of this particular Lie algebra, but mathematicians tend to be less dramatic and call it the covering group. All other groups having the same Lie algebra are said to be covered by the simply connected one. $SU(2)$ is the double cover of $SO(3)$. This means there is a two-to-one map from $SU(2)$ to $SO(3)$.
>
> Furthermore, $SU(2)$ is the three sphere, which is a simply connected manifold. Therefore, we have found the \textquotedbl most important\textquotedbl{} group belonging to the Lie algebra. We can get all other groups belonging to this Lie algebra through maps from $SU(2)$.
>
> We can now understand what manifold $SO(3)$ is. The map from $SU(2)$ to $SO(3)$ identifies with two points of $SU(2)$, one point of $SO(3)$. Therefore, we can think of $SO(3)$ as the top half of $\S^{3}$
>
> We can see, from the point of view that Lie groups are manifolds that $SU(2)$ is a more complete object than $SO(3)$. $SO(3)$ is just "part" of the complete object.
>...
>
>To describe nature at the most fundamental level, we must use the covering group, instead of any of the other groups that one can map to from the covering group. We are able to derive the representations of the most fundamental group, belonging to a given Lie algebra, by deriving representations of the Lie algebra. We can then put the matrices representing the Lie algebra elements (the generators) into the exponential function to get matrices representing group elements.
>
>Herein lies the strength of Lie theory. By using pure mathematics we are able to reveal something fundamental about nature. The standard symmetry group of special relativity hides something from us {[}the spin of elementary particles{]}, because it is not the most fundamental group belonging to this symmetry. The covering group of the Poincare group is the fundamental group and therefore we will use it to describe nature.

What Schwichtenberg is talking about is the fact that the double cover
of the Lorentz group is able to represent the spin of a particle,
while the standard Lorentz group cannot. This is analogous to the
difference between the use of $\S^{3}/\SU\left(2\right)$ versus $\SO\left(3\right)$.
In robotics, we are typically unconcerned with the bottom half of
the $\S^{3}$ unit sphere, so there is no fundamental drawback to
using $\SO\left(3\right)$ like there is in particle physics. However,
there is something about nature and pure mathematics that encodes
rotations in a double cover, and $\SO\left(3\right)$ only tells us
half of the story.

## The generators of $\SE\left(3\right)$

As mentioned before, $\SE\left(3\right)$ is the set of rigid body
transforms. Lie theory as developed by physicists has very little
to say about $\SE\left(3\right)$, as it is actually a subset of the
set of $4\times4$ matrices, known classically as the Lorentz group.
The Lorentz group (or the complexified version, octonians) are used
to encode not only rigid body transforms, but also the dilation of
space and time due to relativistic effects. In robotics, we can usually
restrict ourselves to time and space-preserving transformations, so
we can use a simplified version of the Lorentz space generators.[^4]

[^4]: The bottom row of the $4\times4$ matrix is used in the Lorentz group,
whereas is it not used in $\SE\left(3\right).$ The Lorentz group
generators continue the skew-symmetric pattern we are used to in this
bottom row and right column. For more information about using Lie
groups to model spacetime, [Schwichtenberg 2015](https://link.springer.com/book/10.1007/978-3-319-19201-7) is a phenomenal
introductory text to Lie groups in the context of theoretical physics.
It includes an excellent explanation of the Lorentz group and its
double-cover.

The generators of $\SE\left(3\right)$ are just the basis of infinitesimal
transformations along each degree of freedom.

$$
\begin{align*}
J_{1} & =\begin{pmatrix}0 & 0 & 0 & 0\\\\
0 & 0 & -1 & 0\\\\
0 & 1 & 0 & 0\\\\
0 & 0 & 0 & 0
\end{pmatrix} & J_{2} & =\begin{pmatrix}0 & 0 & 1 & 0\\\\
0 & 0 & 0 & 0\\\\
-1 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0
\end{pmatrix} & J_{3} & =\begin{pmatrix}0 & -1 & 0 & 0\\\\
1 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0
\end{pmatrix}\\\\
J_{4} & =\begin{pmatrix}0 & 0 & 0 & 1\\\\
0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0
\end{pmatrix} & J_{5} & =\begin{pmatrix}0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 1\\\\
0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0
\end{pmatrix} & J_{6} & =\begin{pmatrix}0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 0\\\\
0 & 0 & 0 & 1\\\\
0 & 0 & 0 & 0
\end{pmatrix}.
\end{align*}
$$

The use of these generators defines the $\left(\cdot\right)^{\wedge}$
operator for $\SE\left(3\right)$ as

$$
\begin{pmatrix}\w\\\\
\v
\end{pmatrix}^{\wedge}=\begin{pmatrix}\skew{\w} & \v\\\\
0 & 0
\end{pmatrix},
$$

and $\left(\cdot\right)^{\vee}$ as the inverse operation.

## The generators of $\mathbb{D}\SU\left(2\right)$ and $\mathbb{D}\S^{3}$

As with $\SE\left(3\right),$ we use only the subset of the two copies
of $\su\left(2\right)$ necessary to encode time and space-preserving
3D transformations. The generators for this group are

$$
\begin{align*}
J_{1} & =\begin{pmatrix}0 & 1\\\\
1 & 0
\end{pmatrix}+\epsilon\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix} & J_{2} & =\begin{pmatrix}0 & -i\\\\
i & 0
\end{pmatrix}+\epsilon\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix} & J_{3} & =\begin{pmatrix}1 & 0\\\\
0 & -1
\end{pmatrix}+\epsilon\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix}\\\\
J_{4} & =\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix}+\epsilon\begin{pmatrix}0 & 1\\\\
1 & 0
\end{pmatrix} & J_{5} & =\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix}+\epsilon\begin{pmatrix}0 & i\\\\
i & 0
\end{pmatrix} & J_{6} & =\begin{pmatrix}0 & 0\\\\
0 & 0
\end{pmatrix}+\epsilon\begin{pmatrix}1 & 0\\\\
0 & -1
\end{pmatrix}.
\end{align*}
$$

As $\mathbb{D}\SU\left(2\right)$ is isomorphic to $\mathbb{D}S^{3}$
(dual unit quaternions), the generators for dual quaternions are given
by

$$
\begin{align*}
J_{1} & =i & J_{2} & =j & J_{3} & =k\\\\
J_{4} & =\epsilon i & J_{5} & =\epsilon j & J_{5} & =\epsilon k.
\end{align*}
$$

By this point, we should already be able to tell that the $\left(\cdot\right)^{\wedge}$
operator for these groups is given by

$$
\begin{pmatrix}\w\\\\
\v
\end{pmatrix}^{\wedge}=\begin{pmatrix}\w & \w_{x}-\w_{y}i\\\\
\w_{x}+\w_{y}i & \w_{z}
\end{pmatrix}+\epsilon\begin{pmatrix}\v_{z} & \v_{x}-\v_{y}i\\\\
\v_{x}+\v_{y}i & \v_{z}
\end{pmatrix}
$$

for $\mathbb{D}\SU\left(2\right)$ and

$$
\begin{pmatrix}\w\\\\
\v
\end{pmatrix}^{\wedge}=\begin{pmatrix}0\\\\
\w
\end{pmatrix}+\epsilon\begin{pmatrix}0\\\\
\v
\end{pmatrix}
$$

for $\mathbb{D}\S^{3}$.

[Next: Computing the Exponential](/lie_algebra_tutorial/06-closed_form_mat_exp/)
