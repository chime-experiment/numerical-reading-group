---
theme: solarized
title: "Numerical Reading Group: Linear Regression"
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

- Linear regression intro
- Pseudo inverses
- Wiener Filter
- Gibbs sampling


## Linear Regression

- General formulation is
  $$\vd = \mR \vs + \vn$$
  with $\vd$ the vector of data we have measured, $\vs$ the parameters we want to infer (or the signal), and $\vn$ the noise. The matrix $\mR$ is mapping between the two, it tells us how the signal generates the data.
- Many problems fit into this sort:
  - Telescope observations/map making
  - Delay transforms with RFI masking...

## Standard Solution

- The standard solution is to say we want to find the signal $\hat{\vs}$ that minimises the residuals (or noise)
  $$\begin{aligned}
  \pdiff{\vs} \lp \vd - \mR\vs \rp^\hconj \lp \vd - \mR \vs \rp & = 0 \\
  - \lp \vd - \mR\vs \rp^\hconj \mR & = 0
  \end{aligned}$$
  this implies that
  $$\mR^\hconj \vd = \lp\mR^\hconj \mR \rp \vs$$
- Assuming the term in parentheses is invertible, this gives the solution
  $$\hat{\vs} = \lp\mR^\hconj \mR \rp^{-1} \mR^\hconj \vd$$


## Pseudo-inverse

- What if $\mR^\hconj \mR$ is not invertible?
- What does that mean?
  - $\mR$ has a null space, i.e. there are modes of $\vs$ that do not effect $\vd$
  - $\vs$ has more degrees of freedom (i.e. dimensions) than $\vd$
- How to deal with this? Use the (Moore-Penrose) pseudo-inverse, $\mR^+$
  - In terms of the SVD, if $\mR = \mU \mSigma \mV^\hconj$ then $\mR^+ = \mV \mSigma^+ \mU^\hconj$ where
    $$\mSigma_{ii}^+ = \begin{cases} 1 / \sigma_i & \sigma_i > 0 \\ 0 & \sigma_i = 0\end{cases}$$
- *Minimum power* regularisation, i.e. minimum $\lv \vs \rv$ while satisfying  $\mR^\hconj \vd = \lp\mR^\hconj \mR \rp \vs$.

## Statistical Inversion

- It's more satisfying to view this whole problem as one of *statistical* inversion,
  i.e. given noisy measurements of $\vd$ what can I infer about $\vs$
- First we need to define a few things, we're going to assume most probability
  distributions are multi-variate Gaussians, i.e. some random vector $\vx$ has a probability distribution
  $$\begin{aligned}p(\vx) & = \frac{1}{\lv 2\pi \mX\rv} \exp{\lp -\frac{1}{2} \lp \vx - \bar{\vx}\rp^\hconj \mX^{-1} \lp \vx - \bar{\vx}\rp \rp} \\
  & = \calG(\vx - \bar{\vx}, \mX)\end{aligned}$$
  with the mean $\bar{\vx} = \la \vx \ra$ and covariance
  $\mX = \la (\vx - \bar{\vx}) (\vx -  \bar{\vx})^\hconj \ra$
- Note for the most part we will drop the means (they can be reinserted easily later on).

## Statistical Inversion continued

- First we'll assume the noise $\vn$ is Gaussian
  $$p(\vn) = \calG(\vn, \mN)$$
  and noting that the the noise $\vn = \vd - \mR \vs$ we can write the probability of the data (given the signal) as
  $$p(\vd | \vs) = \calG(\vd - \mR \vs, \mN)$$

## Bayes Theorem

- So we have
  $$p(\vd | \vs) = \calG(\vs - \mR \vs, \mN)$$
  but this isn't quite what we want to know, we are interested in knowing about the
  unknown signal $\vs$ given our fixed observation of the data $\vd$, i.e. we want
  $p(\vs | \vd)$
- This the key issue of *Bayesian inference*, to connect the two, we use *Bayes theorem*
  $$p(\vs | \vd) \propto p(\vd | \vs) p(\vs)$$
- What is $p(\vs)$? This is the *prior*, our knowledge of what the signal is if we hadn't measured it.

## Statistical Inversion

- Let us make the assumption that the signal has a Gaussian distribution (often a less reasonable assumption than  for the noise), i.e.
  $$p(\vs) = \calG(\vs, \mS)$$
- This means that
  $$p(\vs | \vd) \propto \exp{\ls -\frac{1}{2} (\vd - \mR \vs)^\hconj \mN^{-1} (\vd - \mR \vs) - \frac{1}{2} \vs^\hconj \mS^{-1} \vs \rs}$$
  and after a bunch of matrix manipulation (completing the square) is
  $$p(\vs | \vd) = \calG(\vs - \mC \mR^\hconj \mN^{-1} \vd, \mC)$$
  $$\mC = \ls \mS^{-1} + \mR^\hconj \mN^{-1} \mR \rs^{-1}$$


## Statistical Inversion

- Let us take the mean of that distribution
  $$\la \vs \ra = \ls \mS^{-1} + \mR^\hconj \mN^{-1} \mR \rs^{-1} \mR^\hconj \mN^{-1} \vd$$
  this is a statistically motivated linear regression. Compare to the
  $$\hat{\vs} = \lp\mR^\hconj \mR \rp^{-1} \mR^\hconj \vd$$
- This is a **Wiener filter** and is an extremely important technique used in CHIME.
- For modes where:
  -  $S > N$ do a linear regression
  -  $S < N$ then $\vs \rightarrow 0$


## Wiener Filtering in CHIME

- Wiener filtering is the key method used for map-making
  - $\vd$ is the visibility data
  - $\vs$ is the map of the sky
  - $\mR$ is the beam transfer matrices/synthetic beam
  - $\mN$ encodes thermal noise
- See tasks in `draco.analysis.mapmaker/ringmapmaker`.
- Wiener filtering is the key method used regridding data onto the sidereal day
  - $\vd$ is the visibility data in time
  - $\vs$ is the visibility data on the sidereal day
  - $\mR$ encodes the mapping of samples from one to another
  - $\mN$ is thermal noise and RFI mask
- See tasks in `draco.analysis.sidereal`


## Gibbs sampling

- What if you don't know what $\mS$ should be?
- If you knew what $\vs$ is, you could try to estimate $\mS = \la \vs \vs^\hconj \ra$.
- Chicken and egg problem!
- One way to do this is Gibbs sampling, choose a starting guess for $\mS_0$, then
  - Use a Wiener filter to estimate $\vs_i$ using $\mS_i$
  - Update the guess for $\mS_{i+1}$ using $\vs_i$
  - Loop
  - (Gibbs sampling also adds in random fluctuations as it's a Markov-chain Monte-Carlo process)


## Gibbs sampling in CHIME

- Is a useful technique for estimating both the signal ($\vs$), and its covariance/power spectrum ($\mS$)
- In CHIME we use this for delay spectrum estimation in the presence of an RFI mask
  - $\vd$ is the data as a frequency spectrum
  - $\vs$ is the data in delay space (which we will ignore)
  - $\mN$ encodes the RFI mask
  - $\mS$ encodes the delay power spectrum (which is what we want to solve for)
- See `draco.analysis.delay.DelaySpectrumEstimator`