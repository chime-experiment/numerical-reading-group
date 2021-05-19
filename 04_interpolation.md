---
theme: solarized
title: "Numerical Reading Group: Interpolation"
author: Richard Shaw
institute: UBC
date: April 14th 2021
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
\newcommand{\vd}{\vecr{d}}
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

- Point of interpolation
- Preconditioning transforms
- Examples of usage in pipeline
- Implementation in CHIME
- Higher dimensions

## Interpolation

- Useful when you want to have an approximation to a set of highly accurate "data"
- Never use on actual data!
- Only use for interpolation is to provide a *fast* approximation to a function that is otherwise very expensive to compute
  - Results of numerical simulations
  - Integral equations
- Typically you may want to evaluate the function millions of times, but an exact calculation might takes seconds.


## Pretransforming

See notebook


## Examples in pipeline

- Driftscan beam code (`cylbeam.py`)
  - Solves a Fraunhofer diffraction problem to get the points (~1000 samples)
  - Normal linear interpolation
  - Interpolates on this when constructing the beam (~evalulated millions of times)
- Correlation function in new LSS code (`lss-dev` branch, `corrfunc.py`)
  - Calculates a massive Hankel transform to get each sample (~10000 samples)
  - Save results to disk
  - Uses sinh interpolation
  - Evaluated millions of times to generate random field (see session 2)


## Examples in pipeline 2

- Poisson sampling in cora (`poisson.py`)
- To generate samples from a 1D probability distribution function $p(x)$
- Define the cumulative distribution function
  $$F(x) = \int_{-\infty}^x p(x) dx$$
- Generate a random number $r$ from $[0, 1]$
- The random sample $x = F^{-1}(r)$
- Practical implementation
  - Perform integrals for a grid $x_i$ to get normal CDF $F(x_i)$, then perform interpolation
  backwards on $(F(x_i), x_i)$.
  - This gives an approximate *inverse* function which we can use as above


## Extrapolation

- Generally should be avoided, but can be useful on occasion
- Cubic splines (especially with natural boundary conditions) give reasonable extrapolation *provided* you have pretransformed correctly

## Interpolation in CHIME

- Cubic spline interpolation in `cora.util.cubicspline`
- Fast, multithreaded cython code


## High dimensional interpolation

- These interpolation techniques extend to higher dimensions
- In 2D:
  - Bilinear interpolation is the product of two linear interpolations (fast implementation in `cora.util.bilinearmap`)
  - Bicubic interpolation
- Also extended to higher dimensions, but you should really start to think if this is
  the correct approach if you actually need this



## Homework

- Group project:
  - Profiling
  - Tests
  - Modernise?
  - Parallel spline solve
  - Correlated location find
  - Expose data and y2