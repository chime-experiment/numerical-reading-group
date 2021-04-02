---
theme: solarized
title: "Numerical Reading Group: Eigendecompositions"
author: Richard Shaw
institute: UBC
date: Mar 31st 2021
toc: false
slide-level: 2
---

$$
\newcommand{\ensuremath}[1]{#1}
\newcommand\pdiff[1]{\frac{\partial}{\partial #1}}
\newcommand\diff[2]{\frac{d #1}{d #2}}
\newcommand{\curl}{\,\mbox{curl}\,}
\newcommand\del{\nabla}
\newcommand{\grad}[1][]{\ensuremath{\mathbf{\nabla} #1}}

\newcommand{\la}{\ensuremath{\left\langle}}
\newcommand{\ra}{\ensuremath{\right\rangle}}

\newcommand{\lv}{\ensuremath{\left\lvert}}
\newcommand{\rv}{\ensuremath{\right\rvert}}

\newcommand{\ls}{\ensuremath{\left[}}
\newcommand{\rs}{\ensuremath{\right]}}

\newcommand{\lp}{\ensuremath{\left(}}
\newcommand{\rp}{\ensuremath{\right)}}

\newcommand{\vecr}[1]{\mathbf{#1}}
\newcommand{\mat}[1]{\mathbf{#1}}

\newcommand{\hconj}{\ensuremath{\dagger}}

\newcommand{\va}{\vecr{a}}
\newcommand{\vc}{\vecr{c}}
\newcommand{\vf}{\vecr{f}}
\newcommand{\vk}{\vecr{k}}
\newcommand{\vn}{\vecr{n}}
\newcommand{\vr}{\vecr{r}}
\newcommand{\vs}{\vecr{s}}
\newcommand{\vt}{\vecr{t}}
\newcommand{\vu}{\vecr{u}}
\newcommand{\vv}{\vecr{v}}
\newcommand{\vw}{\vecr{w}}
\newcommand{\vx}{\vecr{x}}
\newcommand{\vy}{\vecr{y}}

\newcommand{\vdhat}{\hat{\vecr{d}}}
\newcommand{\vnhat}{\hat{\vecr{n}}}
\newcommand{\vphat}{\hat{\vecr{p}}}
\newcommand{\vuhat}{\hat{\vecr{u}}}
\newcommand{\vvhat}{\hat{\vecr{v}}}
\newcommand{\vxhat}{\hat{\vecr{x}}}
\newcommand{\vyhat}{\hat{\vecr{y}}}
\newcommand{\vzhat}{\hat{\vecr{z}}}

\newcommand{\vnt}{\tilde{\vecr{n}}}
\newcommand{\vvt}{\tilde{\vecr{v}}}
\newcommand{\vxt}{\tilde{\vecr{x}}}

\newcommand{\vnb}{\bar{\vecr{n}}}
\newcommand{\vvb}{\bar{\vecr{v}}}

\newcommand{\vA}{\vecr{A}}
\newcommand{\vE}{\vecr{E}}
\newcommand{\vH}{\vecr{H}}
\newcommand{\vS}{\vecr{S}}

\newcommand{\veps}{\vecr{\varepsilon}}

\newcommand{\mA}{\mat{A}}
\newcommand{\mB}{\mat{B}}
\newcommand{\mC}{\mat{C}}
\newcommand{\mE}{\mat{E}}
\newcommand{\mF}{\mat{F}}
\newcommand{\mG}{\mat{G}}
\newcommand{\mI}{\mat{I}}
\newcommand{\mJ}{\mat{J}}
\newcommand{\mK}{\mat{K}}
\newcommand{\mL}{\mat{L}}
\newcommand{\mM}{\mat{M}}
\newcommand{\mN}{\mat{N}}
\newcommand{\mP}{\mat{P}}
\newcommand{\mQ}{\mat{Q}}
\newcommand{\mR}{\mat{R}}
\newcommand{\mS}{\mat{S}}
\newcommand{\mT}{\mat{T}}
\newcommand{\mU}{\mat{U}}
\newcommand{\mV}{\mat{V}}
\newcommand{\mW}{\mat{W}}
\newcommand{\mX}{\mat{X}}
\newcommand{\mY}{\mat{Y}}

\newcommand{\mNh}{\mat{N}^{\scriptscriptstyle -\frac{1}{2}}}

\newcommand{\mBt}{\tilde{\mat{B}}}
\newcommand{\mCt}{\tilde{\mat{C}}}
\newcommand{\mNt}{\tilde{\mat{N}}}
\newcommand{\mQt}{\tilde{\mat{Q}}}
\newcommand{\mSt}{\tilde{\mat{S}}}

\newcommand{\mGamma}{\mat{\Gamma}}
\newcommand{\mLambda}{\mat{\Lambda}}
\newcommand{\mLambdat}{\tilde{\mat{\Lambda}}}
\newcommand{\mSigma}{\mat{\Sigma}}

\newcommand{\mBb}{\bar{\mat{B}}}
\newcommand{\mCb}{\bar{\mat{C}}}
\newcommand{\mFb}{\bar{\mat{F}}}
\newcommand{\mNb}{\bar{\mat{N}}}
\newcommand{\mSb}{\bar{\mat{S}}}

\newcommand{\mChat}{\hat{\mat{C}}}



\newcommand{\brsc}[1]{{\ensuremath{\scriptscriptstyle (#1)}}}


\newcommand{\calF}{\mathcal{F}}
\newcommand{\calG}{\mathcal{G}}
\newcommand{\calH}{\mathcal{H}}
\newcommand{\calP}{\mathcal{P}}
\newcommand{\calW}{\mathcal{W}}

\newcommand{\tmax}{\ensuremath{\text{max}}}
\newcommand{\tmin}{\ensuremath{\text{min}}}

\newcommand{\kprod}{\otimes}

\newcommand{\dhn}{d^2\hat{n}}

\newcommand{\hMpc}{\;h^{-1}\:\mathrm{Mpc}}
\newcommand{\ihMpc}{\;h\:\mathrm{Mpc}^{-1}}

\newcommand{\eps}{\varepsilon}

\newcommand{\prob}[1]{\mathrm{Pr}(#1)}

\mA
$$


## Overview

- Eigenvalue methods
- Singular Value Decompositions
- Numerical precision

## Eigenvalue Methods

- Iterative methods
  - Operations required $\sim O(M N_\text{ev} N_\text{mv})$
  - Useful when you can evaluate a matrix vector multiplication quickly i.e, $N_\text{mv} \ll N^2$, e.g. sparse matrix
  - ... or you only want a few eigenvalues/vectors, i.e. $N_\text{ev} \ll N$
- Dense methods
  - Faster if you want all the eigenvalues/vectors and the matrix is dense
  - What is done by `la.eigh`

## Eigenvalue Interpretations

- Use eigendecomposition to write a matrix as
  $$\mA = \mX \mLambda \mX^\hconj$$
  with $\mX$ an orthogonal/unitary matrix of eigenvectors and $\mLambda$ the diagonal array of eigenvalues
- As a transformation matrix we can think of this as
  $$\mA \vv = \mX \cdot (\mLambda \cdot (\mX^\hconj \vv))$$
  i.e.
  - Extract the components of $\vv$ along each eigenvector
  - Scale each component by the corresponding eigenvalue
  - Add components up along the eigenvectors to get the final vector


## Eigenvalue Interpretations

- Can also view as a sum of rank-1 outer products
  $$\mA = \sum_{i=0}^{N-1} \lambda_i \, \vx_i \otimes \vx_i^\hconj$$
- We can view a matrix as a sum of components ranked by
  their eigenvalue. Higher eigenvalues are more important to this matrix
- If we truncate the sum at $M \ll N$ to exclude small eigenvalues this is making a *low-rank approximation*
  $$\mA = \sum_{i=0}^{M-1} \lambda_i \, \vx_i \otimes \vx_i^\hconj$$


## Singular Value Decomposition

- Generalisation of the eigendecomposition to non-square, non-symmetric/hermitian matrices
  $$\mA = \mU \mSigma \mV^\hconj$$
- Nomenclature:
  - Diagonal entries of $\mSigma$ are the *singular values*
  - Columns of $\mU$ are *left singular vectors*
  - Columns of $\mV$ are *right singular vectors*
- This is **very** frequently used in CHIME
- Both interpretations still apply, either as transformation matrix, or as a general way of breaking up into rank-1 terms

## Low rank approximation with missing data

- Happens in CHIME frequently:
  - Calibration is looking for a rank-1 (or rank-2) approximation to the visibility matrix (see Kotekan `EigenVisIter`)
  - SVD foreground removal searches for a low rank approximation to the sky as it is dominated by foregrounds (draco `svdfilter.py`)
- In both cases we suffer from missing data
  - Short baselines in the vis matrix are contaminated by cross talk
  - Data has missing time/frequency samples for foregrounds
- Solution is an iterative scheme (~expectation-maximisation)
  - Initialise by zeroing out the bad data
  - Find a low-rank approximation
  - Fill the missing entries with the low rank solution
  - Iterate




## Covariance Matrices

- A very common class of matrix used in CHIME/Cosmology/Physics
- For some vector of random variables $\vx$ the covariance matrix $\mC$ is
  $$[\mC]_{ij} = \text{Cov}(x_i, x_j)$$
  and describes the correlations between all the different components of $\vx$
- This can be written in a more matrix-y notation as
  $$\mC = \la (\vx - \bar{\vx})(\vx - \bar{\vx})^\hconj \ra$$
- Covariance matrices are semi-positive-definite, i.e. all eigenvalues $\geq 0$

## Generating Gaussian realisations

- For simulations we frequently need to generate samples from a multi-variate Gaussian
  - described by mean $\bar{\vx}$ and covariance $\mC$
- Use for generating sky maps (`cora.core.skysim`), Gain fluctuations (`draco.synthesis.gain`) ...
- How can we do this?
  - Generating standard Gaussian samples (i.e. zero-mean, unit variance, uncorrelated) is easy
  - Linear combinations of Gaussian variables are still Gaussian

## Generating Gaussian realisations continued

- Let us define $\vs$ as a vector of standard Gaussian random samples. The most
  general linear transform we can do is
  $$\hat{\vx} = \va + \mA \vs$$
- Requiring $\la \hat{\vx} \ra = \bar{\vx}$ and noting that $\la \vs \ra = 0$, we see that $\va = \bar{\vx}$
- Let us look at the covariance of $\hat{\vx}$
  $$\begin{aligned}\hat{\mC} &= \la (\hat{\vx} - \bar{\vx}) (\hat{\vx} - \bar{\vx})^\hconj \ra \\
  & = \mA \la \vs \vs^\hconj \ra \mA^\hconj \\
  & = \mA  \mA^\hconj \end{aligned}
  $$


## Generating Gaussian realisations continued

- To produce random samples we need to find $\mA$ such that $\mA \mA^\hconj = \mC$
- One option is to use the **Cholesky decomposition** which factorizes a *positive-definite* matrix
  $$\mC = \mL \mL^\hconj$$
  where $\mL$ is lower triangular. This is essentially a restricted version of the *LU decomposition*
- Another option is to use the Eigendecomposition
  $$\mC = \mX \mLambda \mX^\hconj$$
  Define $\mA = \mX \mLambda^{1/2}$ with $\mLambda^{1/2}$ being diagonal with the root of the eigenvalues


## FLOPS count

- Almost all matrix-matrix operations are $O(N^3)$, but the constant factors matter in practice
  - Eigenvalues only $\sim 4 N^3 / 3$ FLOPS
  - Eigenvalues + eigenvectors by QR $\sim 9 N^3$ FLOPS
  - Eigenvalues + eigenvectors by Divide and Conquer $\sim 4 N^3$ FLOPS
  - Cholesky $\sim N^3 / 3$ FLOPS
- Cholesky is preferred.... or is it...


## Numerical Stability

- See Notebook


## Practical solution

- See `cora.util.nputil.matrix_root_manynull`
- Cholesky is so cheap that if you can do it, it pays off, always worth trying
- If not fall back to Eigenvalue method
  - Set small eigenvalues ($\lesssim 10^{-13} \lambda_\text{max}$) to zero explicitly
