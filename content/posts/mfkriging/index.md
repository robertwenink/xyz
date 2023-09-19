---
title: "Multi-fidelity Kriging"
subtitle: ""
date: 2023-09-18T23:32:51+02:00
lastmod: 2023-09-18T23:32:51+02:00
draft: false
description: ""

tags: []
categories: []

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-bib.json"
---

<!--more-->


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

Here $f_r$ denotes the fraction of the number of expensive evaluations over the number of cheap evaluations and similarly, $C_r$ denotes the ratio of simulation costs. Although these criteria are not all directly used in this thesis, we believe they provide some intuition to the practical use of multi-fidelity Kriging.

# Bibliography
{{< bibliography cited >}}
