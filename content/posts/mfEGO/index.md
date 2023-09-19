---
title: "(Multi-fidelity) Bayesian Optimisation"
subtitle: ""
date: 2023-09-18T23:35:22+02:00
lastmod: 2023-09-18T23:35:22+02:00
draft: true
description: ""

tags: []
categories: []

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-bib.json" 
---

<!--more-->
# (Multi-fidelity) Efficient Global optimisation
The development of the proposed method has been in coherence with some of the characteristics of the Efficient Global optimisation (EGO) algorithm and its paradigm will be used in this thesis. This section summarises its definition.

{{< image src="algorithm.png" alt="EGO algorithm" width="100%" linked=false >}}

## Expected improvement
The expected improvement is the expectation of those parts of the normally distributed variable $Y = N(\hat{y},s^2)$ of the Kriging estimator that retrieves a better result than the current best objective value $f_{min}$:

\begin{equation}
\mathrm{E}[I(\boldsymbol{x})] = \mathrm{E}\left[\max \left(f_{\min }-Y, 0\right)\right]
\end{equation}

Using integration by parts we retrieve the formulation of {{< cite "Jones1998-" >}}:

\begin{equation}
\mathrm{E}[I(\boldsymbol{x})]=\left(f_{\min }-\hat{y}\right) \Phi\left(\frac{f_{\min }-\hat{y}}{s}\right)+s \phi\left(\frac{f_{\min }-\hat{y}}{s}\right)
\end{equation}

Here $\Phi$ and $\phi$ are the standard normal Gaussian cumulative distribution function and probability density function respectively. A practical implementation can be found in Algorithm 1.

In [this post](/posts/kriging/#including-and-handling-noise) we saw that unless $\lambda = 0$ the variance at sampled points is not zero. 
In such a scenario, the expected improvement is non-zero too and the EGO algorithm might re-sample previously sampled locations. Since we assess deterministic computer experiments, this would provide us with no extra information. Therefore, if we want to use a regressing Kriging model in combination with the EGO algorithm we should perform re-interpolation {{< cite "Forrester2006noisy" >}}, which is informally but equivalently performed as:

1. Build a regressing Kriging model based on the noisy but deterministic observations $y$
2. Build an interpolating Kriging model using the predictions $\widehat{y}$ of the regressing Kriging model
3. Variance and Expected Improvement is 0 at sampled points again

A benchmark of Kriging-based infill criteria for noisy
optimisation is found in {{< cite "Picheny2013-">}}. The expected improvement criterion combined with re-interpolation {{< cite "Forrester2006noisy" >}} turns out to be the best option for deterministic computer experiments.


## Infill search routine
Searching the response surface for the maximum Expected Improvement is an auxiliary optimisation problem involved with the Efficient Global optimisation routine. This routine is not often described extensively since the Kriging prediction function and thereby the expected improvement are very cheap to evaluate when compared to for instance the hyperparameter tuning process.

{{< cite "Jones2001-" >}} uses a multi-start hillclimbing algorithm with each starting point between a neighbouring pair of sample locations. This is a simple and effective strategy for single-objective design problems. In this thesis, a similar approach is used where the starting points are determined by an LHS DoE with a sample size of 40.

## Multi-fidelity: Level selection strategy
To extend the EGO algorithm to use a multi-fidelity surrogate and thereby improve the performance of SBGO, we should additionally provide a way to decide which fidelity level we should sample once we have selected a location by means of the expected improvement criterion. Otherwise, we will still be exhaustively sampling the high-fidelity, which we aimed to avoid by using a multi-fidelity approach. {{< cite "Meliani2019-" >}} proposes such a methodology using a fully nested DoE, which means all lower fidelities are sampled when and where a higher fidelity is sampled. We summarise their method here using the notation used in this thesis.

First, the expected variance reduction of sampling a level in a nested fashion is given:

\begin{equation}
s_{\text{red}}^{2}\left(l, x^{\*}\right)=\sum_{i=0}^{l} s_{\delta, i}^{2}\left(x^{\*}\right) \prod_{j=i}^{l-1}\hat{\rho}_{j}^{2}
\end{equation}

Here $s_{\delta, i}^{2}$ is the prediction variance of level i, or the last term of $\pcref{gratiet}{2}{/posts/mfkriging/}$. The term $\hat{\rho}_{j}^{2}$ can similarly be identified there. The cost of sampling a level is defined by:

\begin{equation}
cost_{total}(l)=\sum_{i=0}^{l} c_{i}
\end{equation}

Lastly, we find the level we want to sample using:

\begin{equation}
l_{sample}=\underset{l \in(0, \ldots, t)}{\operatorname{argmax}} \quad \frac{s_{\text {red }}^{2}\left(l, x^{*}\right)}{\operatorname{cost}_{\text {total }}(l)^{2}}
\end{equation}

{{< cite "Korondi2021-" >}} adopts a similar approach for non-nested DoE's that reduces to the formulation of {{< cite "Meliani2019-" >}} for nested DoE's.


# Bibliography
{{< bibliography cited >}}
