---
type: post
title:  "Part 7: Adjoint As Jacobian of Group Action"
date:   2020-04-18 13:04:00 -0600
categories: math
tags: ["vectors", "notation", "geometry", "lie", "adjoint"]
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


# The Adjoint as the Jacobian of Group Action


Most people's first introduction to calculus typically focuses on
computing derivatives. For completeness, we're going to start here
and work our way to computing Jacobians of the Lie group action. The
formula for computing the derivative of a scalar function $f$, is
given as\footnote{Note, to be consistent with the vast majority of literature on this
topic, I am using $\varepsilon$ to mean a small perturbation. This
is a separate and distinct concept from $\epsilon,$ the dual number.}

$$
\frac{\partial f}{\partial x}=\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\left(f\left(x+\varepsilon\right)-f\left(x\right)\right).
$$

In conversational English, this means, "How does the output of the
function $f$ change if I bump the input a tiny bit?" This is very
naturally adapted for a vector function $g\in\mathbb{R}^{m}\to\mathbb{R}^{n}$
as

$$
\begin{align*}
\frac{\partial g}{\partial\x} & =\lim_{\varepsilon\to0}\frac{1}{\varepsilon}\begin{bmatrix}\dd_{1}\left(\x,\varepsilon\right) & \dd_{2}\left(\x,\varepsilon\right)\cdots\dd_{m}\left(\x,\varepsilon\right)\end{bmatrix}\in\mathbb{R}^{n\times m}\\\\
\dd_{i}\left(\x,\varepsilon\right) & =g\left(\x+\e_{i}\varepsilon\right)-g\left(\x\right)
\end{align*}
$$

Again, in conversational English, we're saying that $g$ is a function
that takes in $m$-vectors and produces $n$-vectors. We want to know
what $g$ looks like along each of the input axes, so we're going
to make a matrix where each column is the derivative along each of
these directions. This is known as the Jacobian matrix, and can also
be interpreted as, "If I change the input in some way, what does
it do to the output?" For example, if I go further along the $\e_{1}$-axis,
would I expect the output to go up or down? This would be captured
in the first column of the Jacobian. This relationship lets us look
at Jacobians as a sort of mapping from input space to output space,
and lets us quickly compute how changes in input should affect the
output of the function $g$.

The concept of differentiation works quite naturally with Lie groups
as well. For example, consider the case of some function $h\in\SE\left(3\right)\to\mathbb{R}^{n}.$
What we want is a matrix that tells us how the output of $h$ behaves
as we tweak the argument of $h$ along the *generators *of $\se\left(3\right).$
To compute this, we use the fact that every member of the group can
be described by the exponential of some linear combination of generators,
as in\footnote{Assuming that $\x$ is in the bijective region of $\exp$. }

$$
\begin{align*}
T & =\exp\left(\x^{\wedge}\right), & \x^{\wedge} & =\log\left(T\right).
\end{align*}
$$

When we write it this way, we can see how we might perturb $\x$ (and
therefore, $T$) in the $J_{i}$ direction:

$$
\begin{align*}
T_{i}^{+} & =\exp\left(\x^{\wedge}+J_{i}\varepsilon\right)\\\\
 & \approx T\cdot\exp\left(J_{i}\varepsilon\right). & \grey{\left(\text{BCH}\vert\varepsilon\to0\right)}
\end{align*}
$$

Next, we need to think about how we might compute the difference between
$T_{i}^{+}$ and the original $T$. Again, looking to the Lie Algebra,
we should expect the difference $\dd_{i}\in\se\left(3\right)$ to
be

$$
\begin{align}
\exp\left(\dd_{i}\right) & =\exp\left(\x^{\wedge}\right)\cdot\exp\left(J_{i}\varepsilon\right)\cdot\exp\left(-\x^{\wedge}\right)\nonumber \\\\
\dd_{i} & =\log\left(T\cdot\exp\left(J_{i}\varepsilon\right)\cdot T^{-1}\right)^{\vee}.\label{eq:adjoint_start}
\end{align}
$$

If we make a matrix out of the six $\dd_{i}$ for $\se\left(3\right)$,
one for each generator $J_{i},$ and take the limit as $\varepsilon\to0$,
we're going to get a $6\times6$ matrix that describes how the generators
of the *output* of $T$ change as we perturb the *input.*
To be specific, lets attach coordinate frames to our transform $T_{a}^{b}$.
The resulting matrix will tell us how things change along the generators
in the $b$ frame if we perturb along the generators the $a$ frame.
This has a special name--the Adjoint--and we can think about it
as the Jacobian of the group action by a member of the group.[^1]

[^1]:We have a problem here with overloaded terms. The *Jacobian* in the sense that we are talking about refers to the differential of the function "group action on a vector" with respect to the group. Unfortunately, the term Jacobian has another meaning in Lie theory, and it is computed with only the right half of the Adjoint operation. Since this is quick-start guide, I'm choosing to ignore the Lie group interpretation in an effort to draw the analogy between functions of vectors and functions of Lie groups.

We can come at the Adjoint in another way: because the Adjoint is
the Jacobian of the group action of a Lie group, we can use it to
map vectors from the "output" of a group action to the "input",
and vice-versa. In mathematical terms\footnote{for the rest of this section, I'm dropping the $\left(\cdot\right)^{\wedge}$
so we can focus on the coordinate frame notation. The mapping from
the vector to algebra should be implied by context.}:

$$
T\cdot\exp\left(\dd\right)=\exp\left(\Ad_{T}\cdot\dd\right)\cdot T.
$$

See how we have "moved" $\dd$ from the "input" of $T$ (the
right side) to the "output" (the left side)? If we attach coordinate
frames to $T$ this becomes even more clear:

$$
\begin{align}
T_{a}^{b}\cdot\exp\left(\dd^{a}\right) & =\exp\left(\Ad\left(T_{a}^{b}\right)\cdot\dd^{a}\right)\cdot T_{a}^{b}.\nonumber \\\\
 & =\exp\left(\dd^{b}\right)\cdot T_{a}^{b}.\label{eq:adjoint_show}
\end{align}
$$

The Adjoint transforms $\dd$ from $a$ to $b.$

Remember all that talk about left-invariance and right-invariance?
This property of the Adjoint of the Lie group lets us swap back and
forth however we need to. This is really helpful when computing Jacobians
of complex expressions involving Lie Groups.

[Next: Computing the Adjoint](/lie_algebra_tutorial/08-computing_the_adjoint/)
