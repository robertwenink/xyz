---
title: "Thesis project"
subtitle: ""
date: 2023-09-14T15:52:50+02:00
lastmod: 2023-09-14T15:52:50+02:00
draft: true
description: ""

tags: [CFD, Gaussian Process Regression, GPR, Kriging, surrogate modelling]
categories: [Surrogate modelling]

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-bib.json" # path relative to project root
# citations, see : https://github.com/loup-brun/hugo-cite
---
In June of 2023, I defended my thesis project to obtain both a master's degree in Marine Engineering and Computer Science, awarded a 8.5 in both disciplines. The complete thesis titled "Multi-Fidelity Kriging Extrapolation" can be found on the [TU Delft repository](https://repository.tudelft.nl/islandora/object/uuid%3Ad30374fd-8213-40c7-bd72-f17b108d7759?collection=education). In this article, I highlight some parts of this thesis that are of general interest.



# Kriging
Kriging, also known as Gaussian Process Regression, is at the core of the work of this thesis and should therefore be understood to some extent. In this section, the following subjects are treated:
- The formulation of Ordinary Kriging, which mathematically introduces the core concepts of the Kriging predictor and the associated mean-squared-error estimate.
- How can the interpolating Kriging formulation be adapted to be regressing in the presence of noise in the input data.
- How is the Ordinary Kriging formulation modified to retrieve Universal Kriging, the Kriging formulation used in the thesis.
- A short consideration of the hyperparameter tuning process, an optimisation problem in itself and essential to the functioning of the Kriging surrogate.


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

\begin{equation}
\ln (L) = -\frac{n}{2} \ln \left(\sigma^{2}\right)-\frac{1}{2} \ln (|\mathbf{R}|)-\frac{(\mathbf{y}-\mu)^{\prime} \mathbf{R}^{-1}(\mathbf{y}-\mu)}{2 \sigma^{2}}
\end{equation}


Setting the derivatives of the log-likelihood with respect to the process variance $\sigma^2$ and mean $\mu$ to 0 and solving we get their optimal values as a function of correlation matrix $\boldsymbol{R}$:

\begin{equation}
\widehat{\mu} =\frac{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{y}}{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{1}}
\end{equation}

\begin{equation}
\widehat{\sigma}^{2} =\frac{(\boldsymbol{y}-\hat{\mu})^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\widehat{\mu})}{n}
\end{equation}

The hat indicates an estimated quantity. Substituting these expression back into the log-likelihood \cref{eqn:loglikelihood}, we retrieve the 'concentrated log-likelihood':

\begin{equation}
-\frac{n}{2} \ln \left(\widehat{\sigma}^{2}\right)-\frac{1}{2} \ln (|\boldsymbol{R}|) 
\end{equation}

With this being a function of only $\boldsymbol{R}$, we maximize this expression to retrieve the unknown hyperparameters $\Theta$ and $p$ for each dimension. This is a global optimisation problem in itself, but with much cheaper function evaluations. The tuning process is discussed [later](#hyperparameter-tuning).

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

\begin{equation}
\widehat{y}\left(\boldsymbol{x}^{*}\right)=\widehat{\mu}+\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1}(\boldsymbol{y}-\widehat{\mu})
\end{equation}

The associated MSE estimate is found directly from the Gaussian Process formulation {{< cite "Sacks1989" >}}:
\begin{equation}
s^{2}\left(\boldsymbol{x}^{*}\right)=\widehat{\sigma}^{2}\left[1-\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}+\frac{\left(1-\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r}\right)^{2}}{\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{1}}\right]
\end{equation}

The last term can be regarded as a very small correction due to uncertainty in the value of $\mu$ {{< cite "Jones2001" >}} and is not found when following the non-Gaussian derivation given here. This term is generally omitted.

Note that if we calculate Eq. (11) at a sampled point $x^* = x_i$, then $r$ is a column of $R$ and thus $\boldsymbol{R}^{-1} \boldsymbol{r}$ is the $\textit{i}$th unit vector such that $\boldsymbol{r}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r} = r_i = 1$ (correlation with self is 1) and $\boldsymbol{1}^{\prime} \boldsymbol{R}^{-1} \boldsymbol{r} = 1$. Consequently, the prediction variance equates to 0 at sampled locations and thus Eq. (10) is an interpolating formulation.

## Including and handling noise
In deterministic numerical experiments, contrary to physical experiments, we are involved with errors that are of a repeatable nature. When we vary the input during the experiment we can observe fluctuating errors in the output that \textit{appears to be} noise. {{< cite "Forrester2006noisy-" >}} determines three reasons that are the source of this type of noise:
- discretisation errors
- incomplete convergence
- inaccurate application of boundary conditions

Machine-precision round-off errors are of a lesser impact. Similar to {{< cite "Forrester2006noisy-" >}}, the last reason is of no concern to us. 

Discretisation errors are numerical artefacts that are the result of the fact that we need to solve our governing equations on a discretized grid or mesh instead of being able to use a continuous analytic solution. For example, the volume of fluid (VoF) and cut-cell method of the CFD solver we use might produce local pressure peaks {{< cite "Kleefsman2005;Fekken2004" >}}.

Errors due to incomplete convergence, or rather varying levels of convergence, might in our case be observed through the enforcement of the CFL criterium {{< cite "CFL" >}}, in which we ensure numerical stability by varying the timestep. Simulations where due to a slightly different problem a smaller time step was required, might for instance see a higher level of convergence in the part of the simulation domain where the CFL criterium would not have been violated either way.

Would we keep using the [interpolating formulation](#derivation-and-definition-of-ordinary-kriging), in the presence of numerical noise we could encounter situations in which we overfit our data and we would retrieve spurious response surfaces. This would especially be apparent for samples very near to each other.

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

The benefits of Universal Kriging over Ordinary Kriging are clearly observed when no correlating samples are near, in which case the response surface converges back to the Kriging mean $\widehat{\mu}$. For Ordinary Kriging, the response surface then converges to the 'naïve' constant mean, whereas with Universal Kriging it converges to another trend model which in this case is a GLS model. As a result, the resulting response surface will be smoother and more stable. 

Since the Kriging extrapolation method as proposed in this thesis is mainly dependent on proper noise estimates at sampled locations and not directly on the response surface estimates, in principle using either Ordinary- or Universal Kriging would suffice. However, since the Universal Kriging implementation of {{< cite "SMT2019-" >}} provides a more stable and reliable solution than the own Ordinary Kriging implementation provided, Universal Kriging will be used throughout this thesis.

## Hyperparameter tuning
To obtain a correctly functioning and optimal Kriging correlation function and response surface, it is essential to well-tune the involved hyperparameters using (some form of) the concentrated log-likelihood function Eq. (7). If we are [regressing noise](#including-and-handling-noise), we need to simultaneously tune a parameter $\lambda$, where $\lambda$ is a function of $\sigma$. If additionally Universal Kriging is used, the results of the Generalised Least Squares solution are to be included in the Kriging mean and correlation function, as {{< cite "Korondi2021-" >}} clearly describes. 

The matrix inversion involved with solving the log-likelihood function many times during the hyperparameter optimisation process makes this process the main performance bottleneck for Kriging in either high dimensional input or large sample sizes. As a result, much research can be found on improving upon this aspect of Kriging. In our own implementation of Ordinary Kriging, a multi-start hill-climbing procedure had been used in combination with a genetic algorithm. However, since switching to the Universal Kriging solution of {{< cite "SMT2019" >}}, their much more refined routines are used. Since the hyperparameter tuning process is not the main subject of this thesis, I refer to {{< cite "SMT2019-" >}} for further reading.

Lastly, {{< cite "Toal2008-" >}} states that the hyperparameters should be re-tuned each or at least every other model update. Failing to do so could drastically deteriorate the performance of the model. Therefore, in this thesis, we re-tune the hyperparameters during each model update.


# Multi-fidelity Kriging
The word 'fidelity' can be interpreted as the closeness of a simulation method to the truth. Simulations of a lower fidelity can be obtained by reducing the number of mesh elements, using simpler governing equations, or by letting simulations only partially converge {{< cite "Palar2019" >}}. In general, a higher fidelity is more accurate but (much) more expensive than a lower fidelity solution. A multi-fidelity method tries to balance these different costs and accuracies to obtain a result that is more accurate and/or cheaper to obtain than a solution using only the highest fidelity. In the context of the EGO algorithm, it is said we can explore and exploit the objective function using the low- and high-fidelity levels respectively.

Strictly speaking, in this thesis we will adopt a 'multi-level' approach since we will use the same solver at different levels of mesh coarseness. However, the methods provided here could be used by combining different fidelities or sources of information, as long as the main assumption equally holds. Therefore we justify the use of the term 'multi-fidelity', which is semantically often much clearer. 

Throughout this report, we use three fidelities when considering the proposed method, which is the minimum amount required to observe grid convergence. We often refer to the 'two lower fidelities', alternatively called the low and medium fidelity, which are used to predict the 'high fidelity' truth.

## Definition
We could describe the relation between two constitutive Kriging levels in a simplified form as {{< cite "Forrester2007-" >}}:

\begin{equation}
Z_{\mathrm{e}}(\boldsymbol{x})=\rho Z_{\mathrm{c}}(\boldsymbol{x})+Z_{\mathrm{d}}(\boldsymbol{x})
\end{equation}

The subscripts respectively indicate 'expensive', 'cheap', and 'difference'. $\rho$ is a constant multiplicative scaling parameter. The difference model can be regarded as a surrogate model itself.

The auto-regressive method of {{< cite "Kennedy2000-" >}} can equivalently be recursively formulated to solve the problem at a much-reduced cost {{< cite "Gratiet2014" >}}. The recursive formulation of the Kriging predictor and MSE estimate then becomes:
<!-- \label{eqn:Gratiet_formulation} -->
{{< raw >}}
\begin{equation}
\hat{y}_{l}(\boldsymbol{x}_{l}^{\star}) = \hat{\mu}_{l} + \hat{\rho}_{l-1} \hat{y}_{l-1}(\boldsymbol{x}_{l}^{\star})
+ r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1}[y_{l}(\boldsymbol{x}_{l}) - \boldsymbol{1} \hat{\mu}_{l} - \hat{\rho}_{l-1} \hat{y}_{l-1}(\boldsymbol{x}_{l})]
\end{equation}

\begin{equation}
s_{l}^{2}(\boldsymbol{x}_{l}^{\star}) = \hat{\rho}_{l-1}^{2} s_{l-1}^{2}(\boldsymbol{x}_{l}^{\star})
+ \hat{\sigma}_{l}^{2}\left[1 - r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} r_{l} + \frac{\left[1 - r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} r_{l}\right]^{2}}{\boldsymbol{1}_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} \boldsymbol{1}_{l}}\right]
\end{equation}
{{< /raw >}}

The effectiveness of this multi-fidelity formulation is typically exemplified using the somewhat contrived example of {{< cite "Forrester2007-" >}}:

{{< image src="Forrester2007MFshowcase.png" width="80%" alt="Forrester Example" caption="Image and function definition from {{< cite Forrester2007- >}}. By frequently sampling the cheap function ($y_c$) the Kriging approximation based on only four expensive datapoints ($y_e$) is significantly improved." linked=false >}}


## Usage conditions
The implicit presumption of multi-fidelity Kriging is that the difference model $Z_{\mathrm{d}}(\boldsymbol{x})$ is simpler and thus easier to predict than the true response surface. For this to be true we require a high correlation between the fidelity levels. {{< cite "Toal2015-" >}} provides an indication of the conditions in which bi-fidelity Kriging is more efficient or effective than single-fidelity Kriging:

- The correlation between the low and high fidelity function should be reasonably high, $r^2 > 0.9$
- More than 10 \% but no more than 80\% of the total evaluation budget should
be converted to cheap evaluations, $0.1 < f_r < 0.8$. 
- There should always be slightly more cheap data points than expensive with the inequality, $f_r > \frac{1.75}{1+\frac{1}{C_r}}$, giving a conservative bound for this condition

Here $f_r$ denotes the fraction of the number of expensive evaluations over the number of cheap evaluations and similarly, $C_r$ denotes the ratio of simulation costs. Although these criteria are not all directly used in this thesis, we believe they provide some intuition to the practical use of multi-fidelity Kriging.


<!-- Add bibliography, cited entries only -->
# Bibliography
{{< bibliography cited >}}

<!--more-->
