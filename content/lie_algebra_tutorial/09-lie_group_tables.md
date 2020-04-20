---
type: post
title:  "Tables of Useful Lie Group Identities"
date:   2020-04-18 13:11:00 -0600
tags: ["lie", "rotations", "transforms", "jacobians", "adjoint"]
---

This is a bunch of tables that are useful for computing jacobians of the Lie Groups $SO(3)$, $\mathcal{S}^3$, $SE(3)$ or $\mathbb{D}\mathcal{S}^3$

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
.jac_table {
	margin-left: auto;
	margin-right: auto;
	border-collapse: separate;
	border-spacing: 15px;
	text-align: center;
    min-width: 350px;
    background-color: #FFF;
	padding-top: 15px;
	padding-bottom: 15px;
	padding-left: 25px;
    padding-right: 25px;
    margin-top: 0px;
    margin-bottom: 0px;
  }
  table td, table th{
      border: none;
  }
  table th {
      padding-top none;
      border: none;
  }
.center {
	display: block;
	margin-left: auto;
	margin-right: auto;
	text-align: center;
	width: 80%;
  }
.rcorners2{
	border-radius: 25px;
	border: 2px solid #e74c3c;
	padding: 20px;
  }
.rcorners3{
	border-radius: 25px;
	border: 2px solid #e74c3c;
	padding: 20px;
	width: min-content;
	margin-left: auto;
	margin-right: auto;
	text-align: center;
	display: block;
  }
</style>

# Useful Tables

<figcaption class="center">Table 1: Dual Number Formulae</figcaption>
<a name="tab:dual_trig_functions"></a>
<div markdown="span" class="center rcorners3">
$$
\begin{align*}
f\left(r+d\epsilon\right) & =f\left(r\right)+\epsilon d\left(f^{\prime}\left(r\right)\right)\\
\cos\left(r+d\epsilon\right) & =\cos r-\epsilon d\sin r\\
\sin\left(r+d\epsilon\right) & =\sin r+\epsilon d\cos r\\
\arctan\left(r+d\epsilon\right) & =\arctan r+\epsilon\frac{d}{r^{2}+1}\\
\left(r+d\epsilon\right)^{2} & =r^{2}+\epsilon2rd\\
\sqrt{r+d\epsilon} & =\sqrt{r}+\epsilon\frac{d}{2\sqrt{r}}
\end{align*}
$$
</div>
<br />
<br />

<figcaption class="center">Table 2: Taylor series expansions</figcaption>
<a name="tab:taylor_series"></a>
<div markdown="span" class="center rcorners3">
$$
\begin{align*}
\sin\left(\theta\right) & =\theta-\frac{1}{3!}\theta^{3}+\frac{1}{5!}\theta^{5}\cdots\\
\cos\left(\theta\right) & =1-\frac{1}{2}\theta^{2}+\frac{1}{4!}\theta^{4}\cdots\\
\cos\left(\frac{\theta}{2}\right) & =1-\frac{1}{8}\theta^{2}+\frac{1}{46080}\theta^{4}\cdots\\
\frac{\sin\left(\theta\right)}{\theta} & =1-\frac{1}{3!}\theta^{2}+\frac{1}{5!}\theta^{4}\cdots\\
\frac{\sin\left(\frac{\theta}{2}\right)}{\theta} & =\frac{1}{2}-\frac{1}{48}\theta^{2}+\frac{1}{3840}\theta^{4}+\cdots\\
\frac{\theta}{\sin\left(\theta\right)} & =1+\frac{1}{3!}\theta^{2}+\frac{7}{360}\theta^{4}\cdots\\
\frac{1-\cos\left(\theta\right)}{\theta^{2}} & =\frac{1}{2}-\frac{1}{24}\theta^{2}+\frac{1}{720}\theta^{4}\cdots\\
\frac{\cos\theta-\frac{\sin\theta}{\theta}}{\theta^{2}} & =-\frac{1}{3}+\frac{1}{30}\theta^{2}-\frac{1}{840}\theta^{4}\cdots\\
\frac{\frac{1}{2}\cos\frac{\theta}{2}-\frac{\sin\frac{\theta}{2}}{\theta}}{\theta^{2}} & =-\frac{1}{24}+\frac{1}{960}\theta^{2}-\frac{1}{107520}\theta^{4}\cdots\\
\frac{\tan^{-1}\left(\frac{\theta}{y}\right)}{\theta} & =\frac{1}{y}-\frac{1}{3y^{3}}\theta^{2}+\frac{1}{5y^{5}}\theta^{4}
\end{align*}
$$
</div>
<br />
<br />

<figcaption class="center">Table 3: Group-Group Jacobians</figcaption>
<a name="tab:lie_identities_group_operations"></a>
<div class="rcorners3 center" markdown="span">
{{<table "table jac_table">}}
| Expression                                  | Left Jacobian                     | Right Jacobian                         |
| ------------------------------------------- | --------------------------------- | -------------------------------------- |
| $\frac{\partial}{\partial\x}\y\cdot\x$      | $\Ad\left(\y\right)^{-1}$         | $I$                                    |
| $\frac{\partial}{\partial\x}\x^{-1}\cdot\y$ | $-\Ad\left(\x\right)^{-1}$        | $-\Ad\left(\x^{-1}\cdot\y\right)^{-1}$ |
| $\frac{\partial}{\partial\x}\y\cdot\x^{-1}$ | $-\Ad\left(\y\cdot\x^{-1}\right)$ | $-\Ad\left(\x\right)$                  |
| $\frac{\partial}{\partial\x}\x\cdot\y$      | $I$                               | $\Ad\left(\y\right)^{-1}$              |
{{</table>}}
$$
\begin{align*}
\x,\y & \in G
\end{align*}
$$
</div>
<br />
<br />

<figcaption class="center">Table 4: Useful Jacobians for $\SO\left(3\right)$</figcaption>
<a name="tab:lie_identities_group_operations"></a>
<div class="rcorners3 center" markdown="span">
{{<table "table jac_table">}}
| Expression                                              | Left Jacobian                             | Right Jacobian                            |
| ------------------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| $\frac{\partial}{\partial\x}R \cdot \v$                 | $-\skew{R\cdot\v}$                        | $-R\cdot\skew{\v}$                        |
| $\frac{\partial}{\partial\x}R^{\top}\cdot\v$            | $R^{\top}\skew{\v}$                       | $\skew{R^{\top}\cdot\v}$                  |
| $\frac{\partial}{\partial\w}\exp\left(\skew{\w}\right)$ | $aI+b\skew{\w}+c\w\w^{\top}$              | $aI-b\skew{\w}+c\w\w^{\top}$              |
| $\frac{\partial}{\partial R}\log\left(R\right)$         | $I-\frac{1}{2}\skew{\dd}+e\skew{\dd}^{2}$ | $I+\frac{1}{2}\skew{\dd}+e\skew{\dd}^{2}$ |
{{</table>}}
<br />

$$
R\in\SO\left(3\right),\quad\v,\w\in\mathbb{R}^{3}\quad\dd=\log\left(R\right)\quad\theta=\norm{\dd}
$$

$$
a=\frac{\sin\theta}{\theta}\quad b=\frac{1-\cos\left(\theta\right)}{\theta^{2}}\quad c=\frac{1-a}{\theta^{2}}\quad e=\frac{b-2c}{2a}
$$

</div>
<br />
<br />

<figcaption class="center">Table 4: Useful Jacobians for $\SE\left(3\right)$</figcaption>
<a name="tab:lie_identities_group_operations"></a>
<div class="rcorners3 center" markdown="span">
{{<table "table jac_table">}}
| Expression                                                                            | Left Jacobian                                                    | Right Jacobian                                                                                                |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| $\frac{\partial}{\partial\x}T\cdot\v$                                                 | $\begin{pmatrix}-\skew{R\cdot\v+\t} & I\end{pmatrix}$            | $\begin{pmatrix}-R\cdot\skew{\v} & R\end{pmatrix}$                                                            |
| $\frac{\partial}{\partial\x}T^{-1}\cdot\v$                                            | $\begin{pmatrix}R^{\top}\cdot\skew{\v} & -R^{\top}\end{pmatrix}$ | $\begin{pmatrix}\skew{R^{\top}\cdot\left(\v-\t\right)}                                       -I\end{pmatrix}$ |
| $\frac{\partial}{\partial\boldsymbol{\xi}}\exp\left(\boldsymbol{\xi}^{\wedge}\right)$ | $\begin{pmatrix}A & 0\\\\   D & A\end{pmatrix}$                  | $\begin{pmatrix}A^{\top} & 0 \\\\ D^{\top} & A^{\top}\end{pmatrix}$                                           |
| $\frac{\partial}{\partial T}\log\left(T\right)$                                       | $\begin{pmatrix}E & 0\\\\  -E\cdot D\cdot E & E\end{pmatrix}$    | $\begin{pmatrix}E^{\top} & 0\\\\-\left(E\cdot D\cdot E\right)^{\top} & E^{\top}\end{pmatrix}$                 |
{{</table>}}
<br />

$$
T\in\SE\left(3\right),\quad\v\in\mathbb{R}^{3},\quad\boldsymbol{\xi}=\begin{pmatrix}\w & \v\end{pmatrix}^{\top}\in\se\left(3\right)
$$

$$
a=\frac{\sin\theta}{\theta}\quad b=\frac{1-\cos\left(\theta\right)}{\theta^{2}}\quad c=\frac{1-a}{\theta^{2}}\quad d=\w^{\top}\v\quad A=\frac{\partial}{\partial\w}\exp\left(\skew{\w}\right)
$$

$$
B=\w\v^{\top}+\v\w^{\top}\quad C=\left(c-b\right)I+\left(\frac{a-2b}{\theta^{2}}\right)\skew{\w}+\left(\frac{b-3c}{\theta^{2}}\right)\w\w^{\top}
$$

$$
D=b\skew{\v}+cB+dC\quad E=\frac{\partial}{\partial R}\log\left(R\right)
$$
</div>
<br />
<br />

<figcaption class="center">Table 4: Useful Jacobians for $\S^3$</figcaption>
<a name="tab:lie_identities_group_operations"></a>
<div class="rcorners3 center" markdown="span">
{{<table "table jac_table">}}
| Expression                                                  | Left Jacobian                             | Right Jacobian                            |
| ----------------------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| $\frac{\partial}{\partial\x}\q^{-1}\cdot\v^{\wedge}\cdot\q$ | $R\left(\q\right)^{\top}\cdot\skew{\v}$   | $\skew{R\left(\q\right)^{\top}\cdot\v}$   |
| $\frac{\partial}{\partial\x}\q\cdot\v^{\wedge}\cdot\q^{-1}$ | $-\skew{R\left(\q\right)\cdot\v}$         | $-R\left(\q\right)\cdot\skew{\v}$         |
| $\frac{\partial}{\partial\w}\exp\left(\w^{\wedge}\right)$   | $aI+b\skew{\w}+c\w\w^{\top}$              | $aI-b\skew{\w}+c\w\w^{\top}$              |
| $\frac{\partial}{\partial\q}\log\left(\q\right)$            | $I-\frac{1}{2}\skew{\dd}+e\skew{\dd}^{2}$ | $I+\frac{1}{2}\skew{\dd}+e\skew{\dd}^{2}$ |
{{</table>}}
<br />

$$
\q\in\S^{3}\quad\v,\w\in\mathfrak{s}^{3}\quad\dd=\log\left(\q\right)\quad\theta=\norm{\dd}
$$

$$
a=\frac{\sin\theta}{\theta}\quad b=\frac{1-\cos\left(\theta\right)}{\theta^{2}}\quad c=\frac{1-a}{\theta^{2}}\quad e=\frac{b-2c}{2a}
$$

</div>
<br />
<br />

<figcaption class="center">Table 4: Useful Jacobians for $\mathbb{D}\S^3$</figcaption>
<a name="tab:lie_identities_group_operations"></a>
<div class="rcorners3 center" markdown="span">
{{<table "table jac_table">}}
| Expression                                                                            | Left Jacobian                                                                  | Right Jacobian                                                                                    |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| $\frac{\partial}{\partial\x}\qq^{-1}\cdot\v^{\wedge}\cdot\qq$                         | $\begin{pmatrix}R\left(\q_{r}\right)\skew{\v} & R\left(\q\right)\end{pmatrix}$ | $\begin{pmatrix}\skew{R\left(\q_{r}\right)\left(\v-\t\right)} & -I\end{pmatrix}$                  |
| $\frac{\partial}{\partial\x}\qq\cdot\v^{\wedge}\cdot\qq^{-1}$                         | $\begin{pmatrix}-\skew{R\left(\q_{r}\right)^{\top}}\v+\t & I\end{pmatrix}$     | $\begin{pmatrix}R\left(\q_{r}\right)^{\top}\skew{-\v} & R\left(\q_{r}\right)^{\top}\end{pmatrix}$ |
| $\frac{\partial}{\partial\boldsymbol{\xi}}\exp\left(\boldsymbol{\xi^{\wedge}}\right)$ | $\begin{pmatrix}A & 0\\\\ D & A \end{pmatrix}$                                 | $\begin{pmatrix}A^{\top} & 0\\\\ D^{\top} & A^{\top} \end{pmatrix}$                               |
| $\frac{\partial}{\partial T}\log\left(\qq\right)$                                     | $\begin{pmatrix}E & 0\\\\ -E\cdot D\cdot E & E \end{pmatrix}$                  | $\begin{pmatrix}E^{\top} & 0 \\\\ \left(-E\cdot D\cdot E\right)^{\top} & E^{\top}\end{pmatrix}$   |
{{</table>}}
<br />

$$
\qq\in\mathbb{D}\S^{3},\quad\boldsymbol{\xi}=\begin{pmatrix}\w & \v\end{pmatrix}^{\top}\in\mathfrak{ds}^{3}
$$

$$
a=\frac{\sin\theta}{\theta}\quad b=\frac{1-\cos\left(\theta\right)}{\theta^{2}}\quad c=\frac{1-a}{\theta^{2}}\quad d=\w^{\top}\v\quad A=\frac{\partial}{\partial\w}\exp\left(\skew{\w}\right)
$$

$$
B=\w\v^{\top}+\v\w^{\top}\quad C=\left(c-b\right)I+\left(\frac{a-2b}{\theta^{2}}\right)\skew{\w}+\left(\frac{b-3c}{\theta^{2}}\right)\w\w^{\top}
$$

$$
D=b\skew{\v}+cB+dC\quad E=\frac{\partial}{\partial R}\log\left(R\right)
$$

</div>
<br />
<br />



## Left vs Right Jacobians

For now, just read [Micro Lie Theory (Sola 2019)](https://arxiv.org/pdf/1812.01537.pdf). They have a great discussion
on this topic.

Most of the time, I end up using the left Jacobian for the
matrix groups $\SO\left(3\right)$ and $\SE\left(3\right)$ and the
right Jacobian for the complex number groups $\S^{3}$ and $\mathbb{D}\S^{3}$,
but this is because I prefer to express rotations in the inertial
frame. Depending on how you choose to parameterize your problem, however,
you may want to use something else. If things are still hazy after
reading Sola, write unit tests of your implementations and make sure
everything is self-consistent.
