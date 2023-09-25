---
title: "Kriging (Gaussian Process Regression)"
subtitle: ""
date: 2023-09-14T15:52:50+02:00
lastmod: 2023-09-14T15:52:50+02:00
draft: false
description: ""

tags: [CFD, Gaussian Process Regression, GPR, Kriging, surrogate modelling]
categories: [Surrogate modelling]

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-bib.json"
# citations, see : https://github.com/loup-brun/hugo-cite
---

# Kriging
Kriging, also known as a form of Gaussian Process Regression {{< cite "Rasmussen2006" >}}, is a particularly popular surrogate modelling technique due to two reasons. First, Kriging provides the best linear unbiased prediction (BLUP) when the assumption of Gaussian correlations between data samples holds {{< cite "Sacks1989" >}}. Second, Kriging provides an estimation of the mean squared error corresponding to that prediction {{< cite "Forrester2008" >}}. Kriging and Gaussian Process Regression are the most popular surrogate models for use in Bayesian optimisation {{< cite "Palar2019" >}}.

Kriging earns its name from {{< cite "Krige1951;Krige1952-" >}}, who used a precursor method to estimate gold concentrations based on only a few boreholes. It was further mathematically developed by {{< cite "Matheron1962-" >}} and has been popularised for use outside the field of geostatistics by {{< cite "Sacks1989-" >}}. {{< cite "Jones2001-" >}} and {{< cite "Forrester2008-" >}} provide a gentle and intuitive introduction to (Ordinary) Kriging, which will be used as the main reference material here. 

In this post, the following subjects are treated:
- The [formulation of Ordinary Kriging](#derivation-and-definition-of-ordinary-kriging), which mathematically introduces the core concepts of the Kriging predictor and the associated mean-squared-error estimate.
- How to [regress noise](#including-and-handling-noise) while using Kriging.
- How to modify the Ordinary Kriging formulation to retrieve [Universal Kriging](#universal-kriging-taking-a-prior-on-the-mean), which is a more generalised Kriging formulation.
- A short consideration of the [hyperparameter tuning process](#hyperparameter-tuning), which is an optimisation problem in itself and essential to the correct functioning of the Kriging surrogate.


## Derivation and definition of Ordinary Kriging
The derivations in this section are largely adapted from {{< cite "Jones2001;Forrester2008-" >}}, and provide an intuitive and insightful introduction to the (Ordinary) Kriging formulation. For a derivation in a fully Bayesian setting, I refer to {{< cite "Rasmussen2006-" >}}.

Following the principle of a Gaussian Process, it is typically assumed that data at locations $\mathbf{x}=\{x_0,...x_n\}$ is the result of sampling a stochastic process $\boldsymbol{Y}(x) = \mu + \mathcal{N}(0,\sigma^2)$. We further assume realisations of $\boldsymbol{Y(x)}$ that are spatially near to each other are correlated. We describe this correlation using a covariance matrix $\boldsymbol{\Sigma}$, composed of an (unknown) process variance $\sigma$ and the correlation matrix $\boldsymbol{R}$:

\begin{equation}
\boldsymbol{\Sigma}\left(\boldsymbol{Y}\right)=\sigma^{2} \boldsymbol{R}
\end{equation}

To define the correlation matrix $\boldsymbol{R}$, {{< cite "Jones2001" >}} and many others use the so-called "Kriging" kernel:

\begin{equation}
\mathbf{R}\left[\mathbf{Y}(x^{i}), \mathbf{Y}(x^{j})\right]=
\exp \left(-\sum_{\ell=1}^{d} \theta_{\ell}\left|\mathbf{x}^{i \ell}-\mathbf{x}^{j \ell}\right|^{p_{\ell}}\right)
\end{equation}

Other kernels are possible as well but are not necessarily beneficial to our type of problem {{< cite "Jones2001;Forrester2006noisy;Forrester2008" >}}. A more exhaustive list of covariance kernels can be found in {{< cite "Rasmussen2006-" >}}.

Our observations are deterministic within the context of computer simulations, meaning that repeated simulations with the same input will provide the same output {{< cite "Sacks1989" >}}.
So, instead of random variables $\boldsymbol{Y}(x_{i}), \boldsymbol{Y}(x_{j})$ we use observations $\boldsymbol{y}$. Then the likelihood of the data and the corresponding log-likelihood become:

\begin{equation}
L = p\left(y;\mu,\sigma^2,\boldsymbol{R}\right) = \frac{1}{(2 \pi)^{\frac{n}{2}}\left(\sigma^{2}\right)^{\frac{n}{2}}|\boldsymbol{R}|^{\frac{1}{2}}} \exp \left[\frac{-(\boldsymbol{y}-\mu)^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\mu)}{2 \sigma^{2}}\right]
\end{equation}

\begin{equation}\label{eqn:loglikelihood}
\ln (L) = -\frac{n}{2} \ln \left(\sigma^{2}\right)-\frac{1}{2} \ln (|\mathbf{R}|)-\frac{(\mathbf{y}-\mu)^{\prime} \mathbf{R}^{-1}(\mathbf{y}-\mu)}{2 \sigma^{2}}
\end{equation}


Setting the derivatives of the log-likelihood with respect to the process variance $\sigma^2$ and mean $\mu$ to 0 and solving we get their optimal values as a function of correlation matrix $\boldsymbol{R}$:

\begin{equation}
\widehat{\mu} =\frac{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{y}}{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{1}}
\end{equation}

\begin{equation}
\widehat{\sigma}^{2} =\frac{(\boldsymbol{y}-\hat{\mu})^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\widehat{\mu})}{n}
\end{equation}

The hat indicates an estimated quantity. Substituting these expression back into the log-likelihood $\cref{eqn:loglikelihood}{4}$, we retrieve the 'concentrated log-likelihood':

\begin{equation}
-\frac{n}{2} \ln \left(\widehat{\sigma}^{2}\right)-\frac{1}{2} \ln (|\boldsymbol{R}|) 
\end{equation}

With this being a function of only $\boldsymbol{R}$, we maximize this expression to retrieve the unknown hyperparameters $\Theta$ and $p$ for each dimension. This is a global optimisation problem in itself, but with much cheaper function evaluations. The tuning process is discussed [later](#hyperparameter-tuning) in this post.

We have now 'trained' the Kriging model using our data. Suppose we want to predict $y^* $at location $x^*$, adding their correlation to $\boldsymbol{R}$, then:

\begin{equation}
\widetilde{\mathbf{R}} = \begin{pmatrix}
\mathbf{R} & \mathbf{r} \\\\
\mathbf{r}^{\prime} & 1
\end{pmatrix}
\end{equation}

Using this augmented correlation matrix $\widetilde{\boldsymbol{R}}$ instead of $\boldsymbol{R}$ in the log-likelihood, then separating the known $\boldsymbol{R}$ from the unknown $\boldsymbol{r}$ and setting the derivative with respect to $y^* $to 0, we retrieve the expression (see {{< cite "Forrester2008-" >}} for the full derivation):

\begin{equation}
\left[\frac{-1}{\widehat{\sigma}^{2}\left(1-\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}\right)}\right]\left(y^{*}-\widehat{\mu}\right)+\left[\frac{\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\boldsymbol{1} \widehat{\mu})}{\widehat{\sigma}^{2}\left(1-\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}\right)}\right]=0
\end{equation}

Solving for $y^*$, we finally retrieve the Kriging predictor:

\begin{equation}\label{eqn:kriging_prediction}
\widehat{y}\left(\boldsymbol{x}^{*}\right)=\widehat{\mu}+\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\widehat{\mu})
\end{equation}

The associated MSE estimate is found directly from the Gaussian Process formulation {{< cite "Sacks1989" >}}:
\begin{equation}\label{eqn:kriging_variance}
s^{2}\left(\boldsymbol{x}^{*}\right)=\widehat{\sigma}^{2}\left[1-\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}+\frac{\left(1-\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}\right)^{2}}{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{1}}\right]
\end{equation}

The last term can be regarded as a very small correction due to uncertainty in the value of $\mu$ {{< cite "Jones2001" >}} and is not found when following the non-Gaussian derivation given here. This term is generally omitted.

Note that if we calculate $\cref{eqn:kriging_variance}{11}$ at a sampled point $x^* = x_i$, then $r$ is a column of $R$ and thus $\boldsymbol{R}^{-1} \boldsymbol{r}$ is the $\textit{i}$th unit vector such that $\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r} = r_i = 1$ (correlation with self is 1) and $\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r} = 1$. Consequently, the prediction variance equates to 0 at sampled locations and thus $\cref{eqn:kriging_prediction}{10}$ is an interpolating formulation.

## Including and handling noise
In contrast to Gaussian Process Regression, Kriging in principle is an interpolating formulation, which causes problems in the presence of noisy data. To solve this, {{< cite "Forrester2006noisy-" >}} show how to introduce regression terms to the Kriging formulation in coherence with the EGO method through a process called re-interpolation. However, lets first discuss the sources of noise. 

In deterministic numerical experiments (based on simulations that always retrieve the same answer), contrary to physical experiments, we are involved with errors that are of a repeatable nature. When we vary the input during the experiment we can observe fluctuating errors in the output that *appears to be* noise. {{< cite "Forrester2006noisy-" >}} determines three reasons that are the source of this type of noise:
- discretisation errors
- incomplete convergence
- inaccurate application of boundary conditions

Machine-precision round-off errors are of a lesser impact. Similar to {{< cite "Forrester2006noisy-" >}}, the last reason is normally not of a concern (assume the simulation tool and its setup is correct). 

Discretisation errors are numerical artefacts that are the result of the fact that we need to solve our governing equations on a discretized grid or mesh instead of being able to use a continuous analytic solution. For example, the volume of fluid (VoF) and cut-cell method of the CFD solver we use might produce local pressure peaks {{< cite "Kleefsman2005;Fekken2004" >}}.

Errors due to incomplete convergence, or rather varying levels of convergence, might in our case be observed through the enforcement of the CFL criterium {{< cite "CFL" >}}, in which we ensure numerical stability by varying the timestep. Simulations where due to a slightly different problem a smaller time step was required, might for instance see a higher level of convergence in the part of the simulation domain where the CFL criterium would not have been violated either way.

Would we keep using the interpolating formulation of $\cref{eqn:kriging_prediction}{10}$, in the presence of numerical noise we could encounter situations in which we overfit our data and we would retrieve spurious response surfaces. This would especially be apparent for samples very near to each other.

Therefore, to accommodate this apparent noise we adapt the correlation matrix such that each sample does not have an exact relation to itself. We do this by adding a regression constant $\lambda$ such that $R_{regression} = R + \lambda I$. The $\lambda$ can be simultaneously tuned together with the other hyperparameters to retrieve a data-informed estimate of the noise levels. The adapted correlation matrix can directly be plugged into the expressions we saw [earlier](#derivation-and-definition-of-ordinary-kriging) to retrieve the set of equations:

\begin{equation}
\hat{\mu}_{r}=\frac{\mathbf{1}^{\prime}(\mathbf{R}+\lambda \mathbf{I})^{-1} \mathbf{y}}{\mathbf{1}^{\prime}(\mathbf{R}+\lambda \mathbf{I})^{-1} \mathbf{1}}
\end{equation}

\begin{equation}
\widehat{\sigma}_r^{2} =\frac{(\mathbf{y}-\hat{\mu})^{\prime} (\mathbf{R}+\lambda \mathbf{I})^{-1}(\mathbf{y}-\widehat{\mu})}{n}
\end{equation}
{{< raw >}}
\begin{equation}
\hat{y}_{r}(\mathbf{x}_{n+1})=\hat{\mu}_{r}+\mathbf{r}^\prime(\mathbf{R}+\lambda \mathbf{I})^{-1}(\mathbf{y}-\mathbf{1}\hat{\mu}_{r})
\end{equation}

\begin{equation}
\hat{s}_{r}^{2}(\mathbf{x}_{n+1})=\hat{\sigma}_{r}^{2}\left[1+\lambda-\mathbf{r}^{\prime}(\mathbf{R}+\lambda \mathbf{I})^{-1} \mathbf{r}\right] 
\end{equation}
{{< /raw >}}

## Universal Kriging: Taking a prior on the mean $\mu$
In the second paragraph of [the Ordinary Kriging derivation](#derivation-and-definition-of-ordinary-kriging) we chose to use a constant mean in the formulation of our stochastic process $\boldsymbol{Y}$. Although the linear system of Ordinary Kriging is the best linear unbiased predictor {{< cite "Sacks1989" >}}, we can additionally replace the constant mean $\widehat{\mu}$ with a polynomial such that $\boldsymbol{Y}(x) = \mu(x) + \mathcal{N}(0,\sigma^2)$ where $\mu(x) = x^{\prime}\boldsymbol{w}$. The weights $w$ are fitted by means of the generalised least-squares method. This more general approach is called Universal Kriging and was already used in the original derivations of {{< cite "Sacks1989-" >}}. {{< cite "Korondi2021-" >}} gives a clear mathematical description. 
<!-- %In a full Bayesian setting it is equivalent to taking a prior on the mean $\widehat{\mu}$ {{< cite "Rasmussen2006" >}}. -->

The benefits of Universal Kriging over Ordinary Kriging are clearly observed when no correlating samples are near, in which case the response surface converges back to the Kriging mean $\widehat{\mu}$. For Ordinary Kriging, the response surface then converges to the 'naïve' constant mean, whereas with Universal Kriging it converges to another trend model which usually is a Generalised Least Squares (GLS) model. As a result, the resulting response surface will be smoother and more stable. There are quite some Universal Kriging or GPR implementations available. I have used and contributed to that of {{< cite "SMT2019-" >}} ([github](https://github.com/SMTorg/smt)).

## Hyperparameter tuning
To obtain a correctly functioning and optimal Kriging correlation function and response surface, it is essential to well-tune the involved hyperparameters using (some form of) the concentrated log-likelihood function $\cref{eqn:loglikelihood}{7}$. If we are [regressing noise](#including-and-handling-noise), we need to simultaneously tune a parameter $\lambda$, where $\lambda$ is a function of $\sigma$. If additionally Universal Kriging is used, the results of the Generalised Least Squares solution are to be included in the Kriging mean and correlation function, as {{< cite "Korondi2021-" >}} clearly describes. 

The matrix inversion involved with solving the log-likelihood function many times during the hyperparameter optimisation process makes this process the main performance bottleneck for Kriging in either high dimensional input or large sample sizes. As a result, much research can be found on improving upon this aspect of Kriging. In our own implementation of Ordinary Kriging, a multi-start hill-climbing procedure had been used in combination with a genetic algorithm. However, since switching to the Universal Kriging solution of {{< cite "SMT2019" >}}, their much more refined routines are used.

Lastly, in [dynamic sampling optimisation](/posts/mfEGO/) routines {{< cite "Toal2008-" >}} states that the hyperparameters should be re-tuned each or at least every other model update. Failing to do so could drastically deteriorate the performance of the model. Therefore, it is best to re-tune the hyperparameters during each model update.

# Bibliography
{{< bibliography cited >}}
