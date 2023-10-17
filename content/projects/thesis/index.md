---
title: "Thesis project"
subtitle: ""
date: 2023-09-18T15:52:50+02:00
draft: false
description: ""
summary: "" 

tags: [CFD, Gaussian Process Regression, GPR, Kriging, surrogate modelling]
categories: [Surrogate modelling]
series: "Master's thesis"

featuredImage: "lifeboat.png"
# featuredImagePreview: "lifeboat.png"

bibFile: "library-bib.json" 
lightgallery: true
# citations, see : https://github.com/loup-brun/hugo-cite
---

In June of 2023, I defended my thesis project to obtain both a master's degree in Marine Engineering and Computer Science, awarded a 8.5 in both disciplines. This post provides a (shorter) adaptation of the thesis. The complete thesis titled "Multi-Fidelity Kriging Extrapolation" can be found on the [TU Delft repository](https://repository.tudelft.nl/islandora/object/uuid%3Ad30374fd-8213-40c7-bd72-f17b108d7759?collection=education). 

Before reading this article, it might be good to read up on the following subjects (from the post series "Master's thesis"):
- [Surrogate modelling](../../posts/surrogate_modelling/): what is surrogate modelling and what is its typical application in engineering?
- [Kriging](../../posts/kriging/), also known outside engineering as Gaussian Process Regression, is the surrugoate modelling technique at the core of the work of this thesis due to some beneficial characteristics with respect to its use together with expensive-to-evaluate objective functions.
- [Multi-fidelity Kriging](../../posts/mfkriging/): extending the Kriging surrogate to beneficially combine multiple infromation sources of different precision and cost.
- [(Multi-fidelity) Efficient Global Optimisation](../../posts/mfEGO/): an algorithm that uses the characteristics of (multi-fidelity) Kriging to converge to the global optimum solution of a design problem.

# Motivation
Automatic design procedures based on Computational Fluid Dynamics (CFD) simulations are becoming increasingly important in practical ship design {{< cite "Scholcz2015" >}}. For an efficient optimisation phase of such a design procedure, accurate high-dimensional surrogate modelling is essential {{< cite "Viana2014" >}}. A surrogate model provides a cheap substitute for an expensive (CFD) simulation, enabling us to feasibly approximate and explore the design space. To illustrate the effectiveness and promise of using surrogate models in naval optimisation routines, {{< cite "Scholcz2017-" >}} show that Surrogate-Based Global optimisation (SBGO) of a tanker`s hull reduces the required computational time from two weeks to only a day when compared with a direct optimisation algorithm. This is important since {{< cite "Raven2017-" >}} state that the time available for hull shape optimisation with the goal of, for instance, wave resistance minimisation is usually only 1 to 2 weeks or even less.

<!-- % Considerably, Multi-Fidelity SGBO still suffers from the \emph{curse of dimensionality} {{< cite "Palar2019" >}} just like its single-fidelity counterpart does, even though the multi-fidelity approach provides improvements. -->
However, SGBO suffers from the *curse of dimensionality* {{< cite "Palar2019" >}}. Here, the curse of dimensionality refers to the exponential growth of the design-space volume when the number of dimensions or design variables increases, thereby drastically increasing the amount of (expensive) simulations required to cover this volume and reliably construct the surrogate model. In other words, if we increase the number of design variables we optimise over, we drastically need to increase the number of CFD simulations, which is costly or even infeasible. Simultaneously, high-dimensional design spaces naturally arise in naval ship design {{< cite "Scholcz2017" >}}, making this curse of dimensionality a limiting problem for using SBGO in practice. 

Multi-fidelity modelling can improve the efficiency of SBGO, by using the usual expensive and precise data together with cheap but less accurate data. Results as early as that of {{< cite "Leifsson2013-" >}} show that optimised hull designs can be obtained at over 94\% lower computational cost when using multi-fidelity optimisation as compared to a direct high-fidelity optimisation algorithm. However, these results typically do not scale to more design variables.

We want to lessen the effects of the curse of dimensionality even further and enable ourselves to feasibly solve more complex problems of higher dimensionality, or to solve problems more cheaply and more accurately in general. {{< cite "Shan2010-" >}} survey strategies of solving high-dimensional expensive black-box problems. These strategies include problem decomposition, screening (e.g. Global Sensitivity Analysis), mapping (e.g. lossless projection of high-dimensional spaces onto lower-dimensional spaces; or multi-fidelity approaches that map a lower-fidelity onto a higher-fidelity model such as considered in this thesis), and (adaptive) space reduction.
From these, {{< cite "Shan2010-" >}} mark mapping as a promising approach and note that research does not take advantage of the characteristics of the underlying expensive functions. {{< cite "Wu2020-" >}} reiterate this conclusion, demonstrating that this research direction still holds relevance today.

During my thesis, I recognise that the *grid convergence* property of CFD solvers is currently an unused source of information that could further improve the performance of the multi-fidelity surrogate model and the corresponding optimisation process. Grid convergence states that the simulation solution converges to the true simulation solution as the grid is refined. CFD simulations reported by for instance {{< cite "Eijk2021wedge-" >}} show the clear presence of grid convergence. Therefore, this thesis explores a novel hierarchical multi-fidelity mapping method exploiting this grid convergence information to relieve the effects of the curse of dimensionality at the expensive high fidelity. 

## Relevance in Marine Engineering
<!-- % The latest and only {{< cite "Wang2023" >}} further methodological improvement on this work is given by {{< cite "Perdikaris2017-" >}} who generalise the linear autoregressive\footnote{Meaning that the relations between fidelity levels and the regression parameters are directly solved in the linear system} formulation of {{< cite "Gratiet2014-" >}} to better accommodate non-linear-relations.  -->

 <!-- % We do not use the method of {{< cite "Perdikaris2017-" >}} because the method of {{< cite "Gratiet2014-" >}} still is the most popular method {{< cite "Wang2023" >}} while having a ready-to-use implementation available {{< cite "SMT2019" >}}. -->

Optimisation methods based on Multi-Fidelity Kriging hold promise in maritime engineering. {{< cite "Raven2018;Raven2019-" >}} apply Multi-Fidelity Kriging in a ship hull form optimisation with a dense set of potential flow results and a coarse set of RANS data, and positively compared the result with the single-fidelity formulation. {{< cite "Leifsson2013-" >}} shows that optimised hull designs can be obtained at over 94\% lower computational cost when compared to a direct high-fidelity optimisation algorithm, even though they used the outdated multiplicative discrepancy model instead of a comprehensive discrepancy {{< cite "Godino2016" >}}. 
{{< cite "Korondi2019-" >}} performs reliability-based optimisation of a ducted propeller using an EGO-like strategy and a multi-fidelity Kriging surrogate. Although performed under compressible flow conditions, the same techniques are interesting to incompressible flows. {{< cite "Liu2022-" >}} uses viscous and potential flow calculations as the high- and low-fidelity respectively and shows the multi-fidelity Kriging approach can obtain a more optimal hydrodynamic hull form than the single-fidelity model alone. 

Although these are promising results, Multi-Fidelity Kriging still seems relatively rarely applied within maritime engineering, presumably because the increased complexity of the experimental setup is not plenty offset by the possible benefits, and the conditions for application seem very specific. There are still unresolved issues regarding when it is advantageous to use a multi-fidelity over the standard single-fidelity approach {{< cite "Toal2015;Godino2019;Korondi2021" >}} in terms of accuracy, cost, and assumed data characteristics. This work, through developing a new methodology, aims to increase the applicability of multi-fidelity surrogate modelling and SBGO in maritime engineering by providing clearer usage conditions while increasing the benefits of using these methodologies. Additionally, it is expected that the process of selecting the fidelities is simpler since the assumption of grid convergence as introduced in the [motivation](#motivation) implies restricting the scope of this process to a single CFD solver with simulations at different resolutions. 

# Method description
For the mathematical description of the method, I implore you to read the [thesis](https://repository.tudelft.nl/islandora/object/uuid%3Ad30374fd-8213-40c7-bd72-f17b108d7759?collection=education). In the discussion and recommendations of the thesis I formulate ways to improve the method and avoid (all) the problems currently encountered during the method testing and results. I might still perform these since I believe there is plenty potential left on the table.

It is however, important to understand the following assumption:

*The objective value differences implied by the grid convergence property of a CFD solver are proportional from one point in the design space to another.*

Remember the definition given in the motivation:
Grid convergence states that the simulation solution converges to the true simulation solution as the grid is refined.

The above assumption expresses that this convergence will progress in a similar fashion across the design space. 

If no convergence is present at all for a design, we in fact have a converged and thus accurate solution already to which the proposed method will mathematically default.

If, on the other hand, there is convergence but it does not fully conform to this assumption, the proposed method might not function as desired. However, this scenario is cheaply and reliably testable, as we will see [later](#when-to-use-the-proposed-method).

Without further ado, I would like to give an non-mathematical example that gives some intuïtve insight in the proposed method's functioning and capability in the next subsection.

## Pedagogical example
To show the use and effectiveness of the proposed method, I would like to re-iterate the example of {{< cite Forrester2007- >}}, and then slightly change their multi-fidelity transformation function.
The promise of Multi-Fidelity Kriging (MFK) per the contrived example of {{< cite Forrester2007- >}} is shown by [Fig. 3](#fig-mfk-promise).

{{< image src="MFK_promise.png" caption="Figure 3: On the left we see two single-fidelity Kriging surrogate models created at the same budget, and on the right the MFK model that can reproduce the desired high fidelity function almost exactly, but at the same cost." id="fig-mfk-promise" >}}

This result looks amazing! However, when we only slightly change the lower fidelity function (and thereby the relation between the levels), this result completely breaks down as is shown by [Fig. 4](#fig-changed-functions)

{{< image src="changed_functions.png" caption="Figure 4: On the left we see the original synthetic functions per fidelity and on the right the same high fidelity fuction but a changed low fidelity function. The sampled DoE locations remain the same." id="fig-changed-functions">}}

The accompanying change in functions is given by the following python snippet:

```python 
def HF_function(x): # high-fidelity function
    return ((6*x - 2)**2)*np.sin((12*x - 4))  

def LF_function(x): # low-fidelity function
    return 0.5 * HF_function(x) + (x-0.5) * 10. - 5
    # or:  0.5 * ((6*x - 2)**2)*np.sin((12*x - 4.0)) + (x - 0.5) * 10. - 5

def LF_function_adapted(x):
    return 0.5 * ((6*x - 2)**2)*np.sin((12*x - 4.9)) + (x - 0.5) * 10. - 5
```

So, with respect to the high-fidelity, the adapted low-fidelity function only experiences a small shift in the sinusoïdal term and a resulting shift in its minimum. This should not be a problem for a robust method, right?
However, if we now use the same MFK methodology, we get quite bad results as shown in [Fig 5.](#fig-MFK-changed-functions)!

{{< image src="MFKgoodtobad.png" caption="Figure 5: On the left we see the MFK solution on the original functions of {{< cite Forrester2007- >}}, while on the right we see the MFK surrogate on the adapted lower fidelity functions. The MFK surrogate now is far from the high-fidelity truth we wanted to predict, and so we have an undesirable inaccurate surrogate model." id="fig-MFK-changed-functions" >}}

Now, normally one might say the lower fidelity does not represent the high fidelity truth enough, in which case we could opt for a level with a higher fidelity than the current low fidelity. So, let's pick a medium-fidelity, in this case a simple average between the medium- and high-fidelity. Corresponding formula is given by the python snippet below.  

```python 
def MF_function(x):
    return LF_function_adapted(x) + (HF_function(x) - LF_function_adapted(x))/2
```

The result is shown by [Fig. 6](#fig-mfk-to-proposed). After adding the medium-fidelity, the MFK (left) surrogate performs as bad as seen before. What is worse: it believes it is quite accurate too! Clearly, the underlying relationships between the fidelity levels cannot be captured by the MFK method. This is were the proposed method comes in (right side of the image)! It is perfectly able to capture and leverage the relations between the fidelities to exactly re-create the high-fidelity truth. What is more: it can do so with less high-fidelity datapoints (cheaper!), given proper validity of the assumption the method makes on these fidelity relations.

{{< image src="MFKtoProposed.png" caption="Figure 6" id="fig-mfk-to-proposed" >}}

Sure enough, we trade one required data property for the other, but this will extent the applicable range of multi-fidelity modelling. Moreover, wether or not the proposed method's assumption on the fidelity relations is valid can be cheaply and reliably tested, as we will see [later](#when-to-use-the-proposed-method).


# Experimental setup
In the experimental setup to show the efficacy of the proposed method, I use two types of test cases:
- One using synthetic optimisation (minimisation) functions, altered to include noise and transformed to multi-fidelity variants using various relationships between the fidelity levels. In total, around a 1000 variations were used of the analytical functions shown in [Fig. 1](#BraninRosenbrock). 
{{< image src="Branin_Rosenbrock.png" id="BraninRosenbrock" alt="Branin and Rosenbrock synthetic objective functions." linked=false caption="Figure 1: From left to right, the full synthetic objective functions used for minimisation by surrogate modelling on a DoE with a limited amount of function evaluations: [2D Branin](https://www.sfu.ca/~ssurjano/branin.html), [2D and 5D Rosenbrock](https://www.sfu.ca/~ssurjano/rosen.html).">}}
- One where the objective function is retrieved using actual [CFD simulations](../../posts/cfd/), adapted to my needs and limitations. The solver is in development and limited to 2D scenarios. Shape variations are produced using a one-to-one parameterisation using Non-Uniform Rational B-Splines (NURBS). Fidelity levels are realised by using various levels of grid resolution for simulations.

For the complete experimental setup of the experiments using the synthetic objective functions I refer to my thesis document. 

In this post I will focus on the workflow of the approach using CFD simulations. Below flowchart shows the aspects involved with creating the initial surrogate model (based on the initial DoE, without entering the optimisation loop). The further sections corresponding to the experimental setup will follow this flowchart.

{{< mermaid >}}
graph TD;
    A(Problem formulation) --> B(<a href='../../posts/doe/'> Design of Experiments </a>)
    A(Problem formulation) --> C(NURBS shape parameterisation)
    A --> E(CFD environment setup)
    subgraph CFD simulation routines
    C --> D(In-grid body reconstruction) 
    E --> F(Simulation)
    D --> F
    F --> G(Output data processing)
    end
    G --> H(Objective function)
    H --> K( )
    B --> K
    K --> Z{Surrogate model}
{{< /mermaid >}}

## Lifeboat case: Problem formulation
The optimisation case mimics the shape optimisation of a free-fall lifeboat: a rescue vessel that drops off an offshore drilling platform into the sea in case of emergency. We only 'mimic' a complete optimisation because the used CFD solver, although specialised for these types of scenario, is under development and restricted to 2D problems. The optimisation goal is *to minimise the maximum (de-)acceleration* the lifeboat is subjected to when impacting with a flat body of water, to guarantee the safety of its passengers. This goal is achieved by changing the bottom shape of the lifeboat.

The main dimensions, dropping height (and thereby effective vessel speed at the moment of impact), and weight of this lifeboat are taken from the recordholding [FF1200 lifeboat of Palfinger Marine](https://www.palfingermarine.com/en/boats-and-davits/life-and-rescue-boats/free-fall-lifeboats#abd64e4eccollapse2) (the lifeboat seen at the start of this page).

With the goal of testing the proposed method, two variations of the same scenario are used:
1) The lifeboat falls perfectly vertically onto the water.
2) The lifeboat ludricously falls horizontally onto the water.

Setting up more scenarios was not feasible given the time constraints of the thesis.

## Shape parameterisation
The shape parameterisation of the lifeboat is done using Non-Uniform Rational B-Splines (NURBS, see {{< cite NURBSbook- >}}) using 2 parameters. The limitation of 2 parameters is due to the 2D nature of the CFD solver and the requirement of one-to-one parameter-shape mappings. [Fig. 2](#fig-nurbs) shows the used range of possible shape variations of the 2D lifeboat bottom. To create the B-Splines I used the [geomdl](https://github.com/orbingol/NURBS-Python) python package. 

{{< image width="60%" alt="Example NURBS Parameterisations" src="NURBSgrid.png" caption="Figure 2: Some example shape parameterisations, with the parameters $x_i$ in the interval $[0,1]$" id="fig-nurbs" >}}

In the creation of these shapes, the numerical stability and feasibility of the CFD solver imposes some constraints:
- The keel cannot be too flat, to avoid flat-plate impacts that impose extreme high pressures. Although this is already something occuring in the real-world, the CFD solver has the additional factor of an discretised grid with limited (vertical) resolutions able to capture the water displacements.
- The keel cannot be too sharp, due to problems otherwise encountered during the [interface reconstruction calculations](../../posts/cfd/#interface-reconstruction).
- The bilge cannot transition to be horizontal, again to avoid flat-plate impacts.
The relevant outermost angles have therefore been reduced by 10 degrees.

## In-grid body reconstruction
The parameterised lifeboat shape must be mapped onto the numerical grid of the CFD solver, to enable actually simulating the impact of the falling lifeboat. The grid definition itself should be symmetric over the lifeboat and in a domain large enough to not inflict boundary effects, next to solver-specific requirements. 

So, the continuous NURBS shape must be transformed to its counterpart in a discretised numerical grid. This is done through the following steps:

{{< mermaid >}}
graph LR;
    A(NURBS) --> C("<a href=>Signed Distance Function (SDF)</a>")
    C --> D(<a href='../../posts/cfd/#fig-cutcell_definition'>Body volume fraction per grid cell</a>)
    D --> E(<a href='../../posts/cfd/#interface-reconstruction'>Interface reconstruction</a>)
{{< /mermaid >}}

From a NURBS shape parameterisation an Signed Distance Function ([SDF](https://www.labri.fr/perso/nrougier/python-opengl/#signed-distance-fields)) is created. This function provides the shortest distances to the
body boundary with negative values defined to be inside the body and positive values outside the body. My implementation uses the {{< highlight-inline python compute_sdf >}} function of the [glumpy library](https://github.com/glumpy/glumpy).

This SDF function is then used to project a body-fraction field upon the simulation grid corresponding to the NURBS shape. Lastly, based on this fraction field the body is reconstructed in the grid using the Piecewise Linear Interface Calculation ([PLIC](../../posts/cfd/#interface-reconstruction)) technique.

## Output data processing \& Objective function
With the simulation setup done, we can actually run a CFD simulation and generate output data relevant to our problem and objective function.
<!-- TODO -->
TODO

## Fidelity level definition
A necessary step to transform the process of the flowchart into its multi-fidelity counterpart is to determine how the fidelity levels are determined. By definition of the method proposed during my thesis, which assumes that the *grid-convergence* provides information we can use, the construction of these fidelity levels is done by varying the resolution of the numerical grid. The proposed method uses three such fidelity levels:
- The lowest grid-resolution (low-fidelity) cannot be rougher than required for achieving numerical stability during the CFD simulation
- The highest grid-resolution (high-fidelity) should be such that the result is converged and can be considered the 'truth'. In this choice, computational budget can also be a significant limiting factor.
- The in-between grid-resolution (medium-fidelity) should be chosen such that there is some convergence of the objective value present over the fidelity levels.

Like for any multi-fidelity experimental setup, this step is crucial, and turned out to be challenging.


## Experimental method
### Simulation starting conditions
The surrogate model can be initialised in different ways depending on the surrogate modelling method used. 
- The MFK method, due to limitations in solving the underlying GLS problem, needs to start with a minimum of 3 high-fidelity samples in the initial DoE. 
- The proposed surrogate modelling method on the other hand can create a surrogate model with as little as 1 high-fidelity sample. To more fairly compare the proposed method with MFK but also consider this potential competetive capability, the proposed method is started using both 1 and 3 high-fidelity samples in the initial DoE.

### Surrogate performance measurement
The surrogate's accuracy is measured by comparing the predicted value with the real sampled results by using the Root Mean Squared Error (RMSE) as done in {{< cite Toal2015- >}}:

\begin{equation}
\mathrm{RMSE}=\sqrt{\frac{1}{n} \sum_{i=1}^{n}\left(y_{e_{i}}-y_{c_{i}}\right)^{2}}
\end{equation}

However, I prefer the normalised RMSE\% for better comparison between different optimisation cases:

\begin{equation}\label{eqn:NRMSE}
\mathrm{NRMSE}=\frac{\mathrm{RMSE}}{\max(y_e)-\min(y_e)}
\end{equation}

RMSEs are calculated using the high-fidelity `truth' sampled on the full initial DoE $y_e$, where subscript $i$ indexes such that high-fidelity samples used in the surrogate model are excluded to prevent a bias for surrogate models that use a lot of high-fidelity samples. 


# Results
<!-- Falling wedge best design: high specific mass -->
<!-- {{< youtube id="-TzZRLxXXPQ" autoplay="true" >}} -->
<!-- https://youtu.be/-TzZRLxXXPQ -->

<!-- Falling wedge best design: low specific mass -->
<!-- https://youtu.be/jgT5-tSlykU -->


## Synthetic cases \& structured experiments
In short, in the synthetic toy-case experiments the proposed method consistently outperforms the reference SBGO methodology of {{< cite Meliani2019- >}} (multi-fidelity optimisation based on MFK) in terms of optimisation cost, NRMSE of the surrogate and closeness of the found design to the optimum, even when confronted with severe noise or when deteriorating the validity of the assumption upon which the proposed method relies. This is because the proposed method, as a surrogate, is more accurate under the given circumstances than the traditional multi-fidelity MFK surrogate of {{< cite Gratiet2014- >}}, the surrogate at the basis of the SBGO method of {{< cite Meliani2019- >}}. 

These findings confirm the potential of the proposed method under a more varied and complex set of scenarios than was seen [before](#pedagogical-example).

## Lifeboat cases
In this section, we will be looking at two things:
- The competitiveness of optimisation the proposed surrogate modelling method compared to optimisation using the MFK surrogate.
- The quality of the optimisation result and its found designs

{{< image src="EVA_high_mass_fixed.png" caption="Plotted results at the optimisation end for the vertically dropping lifeboat. Left: The proposed method starting with 3 high-fidelity samples that was able to find a better design than all other setups. Right: The reference MFK method, which ends with a high NRMSE due to a wrongly inferred difference trend model. More samples in the initial DoE might decrease the issue." id="fig-eva-vertical" >}}

{{< image src="EVA_low_mass_fixed.png" caption="Plotted results at the optimisation end for the horizontally dropping lifeboat. On the left: The proposed method starting with 1 high-fidelity sample that has after optimisation has the best cost statistics and the best ending NRMSE. On the right: the reference method, which like the vertical case ends with a high NRMSE." id="fig-eva-horizontal" >}}

In both cases, the MFK surrogate knows a large NMRSE (also visually), caused by a wrongly inferred difference trend model. Around the optimum, there is no to little convergence of the objective value (meaning a difference of $\approx 0$), while elsewhere there is convergence present (meaning differences $>0$) with differences that seem to stabalise when moving further from the optimum design. With three starting samples in the initial DoE, the MFK methodology therefore infers a parabolic difference model, and faultly maps this to the rest of the domain. 

The proposed surrogate on the other hand does not suffer from this problem. This is partly because the method recognises undesirable conditions and defaults to the medium-fidelity, which is this case is - especially around the optimum design where little to no convergence is present - a stable and accurate alternative.

### Best designs per optimisation case and their hydrodynamic explanation
The following shows renders of the simulations that correspond to the best designs found during the vertically and horizontally falling lifeboat cases, respectively.

In the case of the 'vertically falling' case, the different orientation is encoded by a higher specific mass of the body in the simulation. We therefore do not actually see a more vertical structure as compared to the horizontally falling case. To the simulation outcome, both methods are completely equal.

<!-- Falling wedge best design: high specific mass -->
#### Falling wedge: best vertically falling design
{{< vimeo 873147687 >}}
{{< image src="filtered_high_mass.png" >}}

<!-- Falling wedge best design: low specific mass -->
#### Falling wedge: best horizontally falling design
{{< vimeo 873143175 >}}
{{< image src="filtered_low_mass.png" >}}


# When to use the proposed method
TODO


<script src="https://player.vimeo.com/api/player.js"></script>


# Bibliography
{{< bibliography cited >}}
