---
title: "Multi-fidelity Kriging"
subtitle: ""
date: 2023-09-18T23:32:51+02:00
draft: false
description: ""

tags: []
categories: [Surrogate modelling]
series: "Master's thesis"

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-bib.json"
---

<!--more-->
In this post, we focus on the standard linear auto-regressive formulation of {{< cite "Gratiet2014-" >}} in the optimisation context of the EGO adaptive sampling algorithm, and are interested in the potential of this combination to reduce the costs of SBGO. {{< cite "Meliani2019;Korondi2021-" >}} use the recursive formulation of {{< cite "Gratiet2014-" >}} to introduce a multi-fidelity approach to the EGO algorithm and formulate level-selection criteria that balance the expected reduction of the prediction variance with the costs of sampling a given level. The multi-fidelity SBGO method of {{< cite "Meliani2019-" >}} will act as a reference to the method proposed in this thesis since both methods use a so-called nested Design of Experiments (DoE), a sampling strategy that provides the most complete information on the relation between the fidelities {{< cite "Kennedy2000" >}}.

{{< cite "Jones2001-" >}} discusses the relation of Kriging with respect to other surrogate modelling methods and explores possible improvements to EGO and the Expected Improvement criterion. As one of the most promising directions for further work, he hinted towards multi-fidelity simulation: using expensive high-precision simulations together with simulations of a lower precision but a much lower cost. This enables us to respectively balance exploitation and exploration of the search space and thereby cut costs while improving the quality of the surrogate.

Since then, Multi-Fidelity Kriging has received significant research effort, see for instance the review papers of {{< cite "Viana2014;Godino2016;Peherstorfer2018-" >}}. {{< cite "Kennedy2000-" >}} mathematically formulated the fundamentals, while {{< cite "Forrester2007-" >}} provides a more transparent overview. {{< cite "Gratiet2014-" >}} show that the model of {{< cite "Kennedy2000-" >}} can be equivalently but more cheaply formulated as recursively nested independent Kriging problems, one for each fidelity level. {{< cite "Perdikaris2017-" >}} improve upon this work by generalising the linear autoregressive[^1] formulation of {{< cite "Gratiet2014-" >}} to better accommodate non-linear-relations. 
[^1]:Meaning that the relations between fidelity levels and the regression parameters are directly solved in the linear system

# Multi-fidelity Kriging
The word 'fidelity' can be interpreted as the closeness of a simulation method to the truth. Simulations of a lower fidelity can for instance be obtained by reducing the number of mesh elements, using simpler governing equations, or by letting simulations only partially converge {{< cite "Palar2019" >}}. 

In general, a higher fidelity is more accurate but (much) more expensive than a lower fidelity solution. A multi-fidelity method tries to balance these different costs and accuracies to obtain a result that is more accurate and/or cheaper to obtain than a solution using only the highest fidelity. In the context of the [MF-EGO](/posts/mfego/) algorithm, it is said we can explore and exploit the objective function using the low- and high-fidelity levels respectively.

## Definition
We could describe the relation between two constitutive Kriging levels in a simplified form as {{< cite "Forrester2007" >}}:

\begin{equation}
Z_{\mathrm{e}}(\boldsymbol{x})=\rho Z_{\mathrm{c}}(\boldsymbol{x})+Z_{\mathrm{d}}(\boldsymbol{x})
\end{equation}

The subscripts respectively indicate 'expensive', 'cheap', and 'difference'. $\rho$ is a constant multiplicative scaling parameter. The difference model can be regarded as a surrogate model itself.

The auto-regressive method of {{< cite "Kennedy2000-" >}} can equivalently be recursively formulated to solve the problem at a much-reduced cost {{< cite "Gratiet2014" >}}. The recursive formulation of the Kriging predictor and MSE estimate then becomes:
<!-- \label{gratiet} -->

{{< raw >}}\begin{equation}\label{gratiet}
\hat{y}_{l}(\boldsymbol{x}_{l}^{\star}) = \hat{\mu}_{l} + \hat{\rho}_{l-1} \hat{y}_{l-1}(\boldsymbol{x}_{l}^{\star}) + r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1}[y_{l}(\boldsymbol{x}_{l}) - \boldsymbol{1} \hat{\mu}_{l} - \hat{\rho}_{l-1} \hat{y}_{l-1}(\boldsymbol{x}_{l})]
\end{equation}

\begin{equation}
s_{l}^{2}(\boldsymbol{x}_{l}^{\star}) = \hat{\rho}_{l-1}^{2} s_{l-1}^{2}(\boldsymbol{x}_{l}^{\star}) + \hat{\sigma}_{l}^{2}\left[1 - r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} r_{l} + \frac{\left[1 - r_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} r_{l}\right]^{2}}{\boldsymbol{1}_{l}^{\prime}(R_{l} + \lambda_l \mathbf{I})^{-1} \boldsymbol{1}_{l}}\right]
\end{equation}{{< /raw >}}

The effectiveness of this multi-fidelity formulation is typically exemplified using the somewhat contrived example of {{< cite "Forrester2007-" >}}:

{{< image src="Forrester2007MFshowcase.png" width="80%" alt="Forrester Example" caption="Image and function definition from {{< cite Forrester2007- >}}. By frequently sampling the cheap function ($y_c$) the Kriging approximation based on only four expensive datapoints ($y_e$) is significantly improved." linked=false >}}


## Usage conditions
The implicit presumption of multi-fidelity Kriging is that the difference model $Z_{\mathrm{d}}(\boldsymbol{x})$ is simpler and thus easier to predict than the true response surface. For this to be true we require a high correlation between the fidelity levels. {{< cite "Toal2015-" >}} provides an indication of the conditions in which bi-fidelity Kriging is more efficient or effective than single-fidelity Kriging:

- The correlation between the low and high fidelity function should be reasonably high, $r^2 > 0.9$
- More than 10 \% but no more than 80\% of the total evaluation budget should
be converted to cheap evaluations, $0.1 < f_r < 0.8$. 
- There should always be slightly more cheap data points than expensive with the inequality, $f_r > \frac{1.75}{1+\frac{1}{C_r}}$, giving a conservative bound for this condition

Here $f_r$ denotes the fraction of the number of expensive evaluations over the number of cheap evaluations and similarly, $C_r$ denotes the ratio of simulation costs. Although these criteria might not always be accurate, they at least provide some intuition to the practical use of multi-fidelity Kriging.

# Bibliography
{{< bibliography cited >}}
