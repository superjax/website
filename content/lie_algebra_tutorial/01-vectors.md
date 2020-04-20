---
type: post
title: "Part 1: Vectors"
date:   2020-04-18 13:10:00 -0600
tags: ["vectors", "notation", "Geometry"]
---

This is the first of a 8-part series of posts designed as a quick-start guide for students new
to the field of robotics and estimation, specifically on the use of
Lie groups to describe rotations and rigid body transformations. This is a web port of the full pdf document, which is hosted [here](https://drive.google.com/open?id=1T93xYi7iKqrW7a9KCJvoBB9no4j7zaDu).

# Introduction

The
topic of Lie groups is fundamental to much of modern physics, and
therefore has a deep and rich history going back several decades.
At a high level, Lie theory connects the \emph{symmetries} of nature
with differential equations. Rotations and rigid body transformations
are two examples of such natural symmetries. The use of Lie group
theory has recently become prevalent in state-of-the-art methods in
robotics and state estimation, since it provides a natural way to
connect the symmetries that often come up in robotics (rotations and
rigid transformations) to differential equations.

Unfortunately, rigorous Lie theory also comes with a lot of vocabulary
and subtlety that can discourage the uninitiated. Therefore, I'm are
going to deliberately gloss over a lot of the nuance and subtlety
that exists in the ideas of Lie Groups, and attempt to supply \emph{just
enough} information to give you the vocabulary and intuition necessary
to understand and implement most of the state-of-the art robotics
literature. Pretty much everything in this document comes from the
following papers: \cite{Barfoot2019,Drummond2014,Ethan2019,Sola2019,schwichtenberg_2015}.
These are all excellent resources that can take your understanding
to the next level, and I would recommend any and all of them if you're
interested in being more rigorous in the theory.

One difference with this document when compared with others is the
use of extra notation throughout. This notation is much more detailed
than a lot of other literature on robotics or computer vision, but
the explicit syntax clarifies what is going on, and makes it easier
for someone like me (who likes to think of these ideas in terms of
physical relationships, rather than abstract mathematical ideas) to
visualize and understand what is going on. The extra notation has
also allowed me to make distinctions between different fields of research
that may have different conventions when it comes to transformations.

As many of my fellow graduate students and I used to say, ``The two
hardest problems in robotics are coordinate frames and naming things.''
While this won't help you much with the second problem, this will
hopefully help dispel a lot of the confusion I encountered at first.

# Vectors

Let us first describe the notation used in the rest of the document.
Succinctly, it is described as follows:
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

$$
\begin{aligned}
a & \quad\text{scalar quantity }a\in\R\\\\
\v & \quad\text{vector quantity }\v\\\\
\r_{a/b}^{c} & \quad\text{the vector }\r, \text{ representing some quantity (e.g. position) of point }a \text{ with respect to }\\\\
& \quad\text{point } b, \text{expressed in frame } c.\\\\
R_{a}^{b} & \quad\text{a transformation or rotation that executes a change of basis from frame } a \text{ to frame} b\\\\
\skew{\v} & \quad\text{The skew-symmetric matrix formed from }\v\end{aligned}
$$

## Vector Notation

If that's all you need, then great! You can carry on to the next section.
However, for people like me, let's dig into this notation and (hopefully)
let it sink in. Consider the illustration in [Figure 1](#fig:notation).
We have some point $a$ and another point $b$. Let $\r_{a/b}$ be
the vector which describes the position of $a$ with respect to $b$.
Note that we could flip the vector around by negating the quantity,
which means that for any arbitrary frame of reference

<a name="fig:notation"></a>
<div class="center">
<img  src="/lie_tutorial/frame.svg" style="width: 30vw; min-width: 120px;">
<figcaption class="center">Figure 1: Illustration of a vector with notation</figcaption>
</div>

Although it may be sometimes hard to actually draw some quantities
clearly (like the angular rate between two reference frames), in engineering,
we typically define vectors as some quantity with respect to some
other reference. For example, the angular velocity of a rotating body
with respect to the earth might be written as $\w_{b/E}$. While it
may be difficult to visualize, we could also consider the opposite
case: the angular velocity of the earth with respect to the rotating
body, $\w_{E/b}$. This can be a fun mind bender, turning frames of
reference around in your head. It turns out that even in this case,
the *earth-centric* representation is just the negative of the
*body-centric* representation.

$$
\w_{E/b}=-\w_{b/E}.
$$

The next piece of information that we need is the frame of reference
it is described in (its basis). For example, in [Figure 1](#fig:notation),
we could describe $\r$ in any coordinate frame we want. We could
use frame $a$, $b$ or $c$, it doesn't really matter, so long as
we always do all operations between vectors represented in the same
frame.

We will use the superscript notation to describe the frame of reference,
as in

$$
\r_{a/b}^{c},
$$

which in words means ``the position of point $a$ with respect to
point $b$, expressed in frame $c$.'' Pretty much any vector quantity
we want to manipulate in robotics will have this concept of frame
of reference, even if it is hard to visualize such as angular rate
and acceleration.\footnote{The discussion in this documents is strictly limited to vector spaces
over real numbers. There are more abstract definitions of a vector
space.}

## Rotating Vectors

I'm going to jump ahead a little bit and introduce the concept of
a rotation as a change of basis. Let us say that we have some $\r$
expressed in the $c$ frame, but we want to express it in the $b$
frame. How do we do that? Easy, we simply change the basis, as in

$$
\r_{a/b}^{b}=R_{c}^{b}\mathbf{r}_{a/b}^{c}.
$$

I will get into the details of how this actually works a little later,
but for now, just believe me that we can change this basis. The notation
is such that you can *cancel out* the frames, as in

$$
\require{cancel}
\r_{a/b}^{b}=R_{\cancel{c}}^{b}\mathbf{r}_{a/b}^{\cancel{c}}.
$$

So what if I want to go all the way to frame $a$ but I only have
rotations from $c\to b$ and $b\to a$? That's easy, just
compose the rotations, and everything cancels out.

$$
\mathbf{r}_{a/b}^{a}=R_{\cancel{b}}^{a}R_{\cancel{c}}^{\cancel{b}}\r_{a/b}^{\cancel{c}}.
$$

Again, I'm deliberately skimming over most of the actual math. We'll
get into this a lot more later, but we first need to get the mechanics
of working with this notation.

## Composing Vectors
Let's now take a minute to remember what defines a vector space. From
Wikipedia:

> A vector space is a collection of objects called vectors, which may be added together and multiplied (scaled) by numbers, called scalars.


All vector spaces abide by the rules shown in [Table 1](#tab:vector_rules}.
If you've taken a course in linear algebra, these will not be new
to you. However, for me, up to this point, I had never worked with
objects that were *not *in a vector space, so the rules seemed
obvious and redundant. We like vector spaces because computers are
well-suited for solving linear algebra problems. There are high-performance
libraries such as BLAS (basic linear algebra subprograms) and methods
for performing matrix decompositions (SVD, LU, QR, Cholesky) that
allow us to solve complicated problems quickly and accurately.


<a name="tab:vector_rules"></a>

| Axiom                                     | Meaning                                                                                                                      |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Associativity                             | $\a+\left(\b+\c\right)=\left(\a+\b\right)+\c$                                                                                |
| Commutativity                             | $\a+\b=\b+\a$                                                                                                                |
| Identity of addition                      | There exists an element $\boldsymbol{0}$ such that $\a+\boldsymbol{0}=\a$ for every vector in the space                      |
| Inverse element of addition               | for every vector $\a$ in the space, there is one and only one vector   $-\a$ such that $\a+\left(-\a\right)=\boldsymbol{0}.$ |
| distributivity of scalar multiplication   | $\left(v+u\right)\a=v\a+u\a$\\ $v\left(\a+\b\right)=v\a+v\b$                                                                 |
| Identity element of scalar multiplication | $1\a=\a$                                                                                                                     |

<figcaption class="center">Table 1: The rules of a vector space</figcaption>

It turns out that rotations and rigid-body transformations do *not*
lie in a vector space. This means that we end up performing manipulations
to get our problems into a vector space where we can use powerful
linear algebra techniques, and the distinction between vector spaces
and non-vector spaces will become very important.

Let us say we have three bodies, each with an associated coordinate
frame $a$, $b$, and $c$, as shown in [Figure 2](#fig:vector_compose).
Let's also say that we have only the orange vectors. (the vector from
$b\to a$ and the vector from $c\to b$) but we want the green vector
(the one that goes all the way from $c\to a$).

<a name="fig:vector_compose"></a>
<div class="center">
<img  src="/lie_tutorial/vector_composition.svg" style="width: 30vw; min-width: 120px;">
<figcaption class="center">Figure 2: Illustration of a vector triangle</figcaption>
</div>


Well, this would be easy if they were all represented in the same
coordinate frame

$$
\r_{a/c}=\r_{a/b}+\r_{b/c},
$$

but they aren't. However, we just have to remember that vectors can
only be added or subtracted if they are in the same frame, so we just
rotate them into a common frame before doing the additions, like this:

$$
\r_{a/c}^{c}=R_{b}^{c}\left(\r_{b/c}^{b}+R_{a}^{b}\r_{a/b}^{a}\right).
$$

From a theoretical perspective, the choice of coordinate frame doesn't
matter, however in practice there is often a choice of frame that
makes things easier.

## Skew-symmetric matrices

Before going much further, I also need to introduce skew-symmetric
matrices, and the the skew-symmetric matrix operator\footnote{There are a variety of symbols used to communicate this operation.
$\v_{\times}$ and $\left(\v\right)^{\times}$ are also commonly used.} $\skew{\v}$. It is defined as

$$
\skew{\v}=\left[\begin{array}{ccc}
0 & -\v_{z} & \v_{y}\\\\
\v_{z} & 0 & -\v_{x}\\\\
-\v_{y} & \v_{x} & 0
\end{array}\right],\quad\v=\begin{pmatrix}\v_{x}\\\\
\v_{y}\\\\
\v_{z}
\end{pmatrix}
$$
and is related to taking the cross-product between two vectors as

$$
\a\times\b=\skew{\a}\b.
$$

The skew-symmetric matrix has some really interesting and helpful
properties. The first is that the operator is anti-commutable, that
is

$$
\begin{equation}
\skew{\a}\b=-\skew{\b}\a.\label{eq:skew_trick_1}
\end{equation}
$$

The second is that rotation matrices can be moved in and out of the
skew-symmetric operator with\footnote{This is also called the Adjoint representation of $R$}

$$
\begin{equation}
\skew{R\v}=R\skew{\v}R^{-1}.\label{eq:skew_trick_2}
\end{equation}
$$

Finally, you could probably see from inspection that the transpose
of a skew-symmetric matrix is its negative

$$
\left\lfloor \v\right\rfloor _{\times}^{\top}=-\skew{\v}.
$$

These are all super useful tricks that show up all the time. I'll
try to point out when I'm using one of these tricks, but I might miss
some.

[Next: The Matrix Exponential](/lie_algebra_tutorial/02-matrix_exp/)
