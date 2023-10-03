---
title: "Design of Experiments"
subtitle: ""
date: 2023-09-18T17:41:23+02:00
draft: false
description: ""

tags: [DoE, sampling]
categories: [Surrogate modelling]
series: "Master's thesis"
weight: 5

featuredImage: ""
featuredImagePreview: "example_doe_2d_2.png"

bibFile: "library-bib.json" # path relative to project root
---

<!--more-->
In surrogate modelling, to get a uniform coverage of the parameter domain and get as much information about the objective value's variability per dimension, we need an adequate design of experiments (DoE), also known as a sampling plan. Such a design is used to determine the inputs with which we run the initial set of simulations that will be used to construct our initial surrogate model(s). Since we are considering a deterministic computer experiment, the DoE does not consider re-sampling for the same input.

# Latin Hypercube Sampling
The most popular technique is called Latin Hypercube Sampling (LHS). In this method, each dimensional axis gets divided into a predefined amount of bins equal to the total amount of samples. In each bin, only one sample will be placed. In 2D this means one sample will occupy a row and a column and no other sample can be placed in that row and column.  

Although this procedure will result in a so-called 'stratified' sampling plan, it is not necessarily optimally 'space-filling' or uniform as desired. Therefore the sampling plan is further optimised using some optimality criterium. An effective and popular choice is the maximin criterium, also called the Morris \& Mitchell criterium {{< cite "Morris1995">}}, in which we select the sampling plan that maximises the minimum (Euclidean) distance between any sampling location. Optimally, we search all possible LHS plans according to this criterion using a divide and conquer method {{< cite "Forrester2008" >}}. However, this is computationally expensive and in practice we can search for such a plan iteratively. The Python library [pyDOE2](https://pypi.org/project/pyDOE2/) provides the iterative DoE implementation.

In the multi-fidelity setting, now that we have found an optimised DoE for our lowest fidelity or fidelities, we would like to select a subset of that sampling plan to construct a DoE for the higher fidelity. To generate such an initial high-fidelity DoE, we greedily select points from the already defined lower-fidelity DoE in a similar fashion to {{< cite "Forrester2008-" >}} following the maximin principle. However, contrary to {{< cite Forrester2008 >}}, we take the distance to the input domain boundary into account and start with the location of the sample with the current best lower fidelity objective function. The total greedy selection algorithm is expressed in the pseudocode of Algorithm 1 below. An example result of the combined LHS and subset selection is given by [this image](#example_doe).

{{< image src="algorithm1.png" alt="Nested DoE algorithm" width="100%" linked=false >}}

In line 6 of the algorithm I multiply the distance by a factor of two. This is because the goal is to maximise the minimum distance to a source of information. Objective value information is supplied by two sources for distances between sampled locations. In the distance to the boundary, there is only one source of information. To compensate for this we should therefore divide the values of the distance matrix- or equivalently multiply the distances to the boundary by a factor of two. As an analogue for those familiarised with CFD, we effectively calculate the distance to a mirrored non-existing 'ghost point'.

{{< image src="example_doe_2d_2.png" alt="Nested DoE example" caption="Example of an iteratively optimised LHS experimental design (n = 20) with greedy maximin subset selection ($n_{subset} = 5$) according to Algorithm 1." width="80%" id="example_doe" linked=false >}}

## Size of the initial DoE
The size of the initial Design of Experiments is of large importance to the accuracy of the initial surrogate model and the cost of (initial) sampling. Too many samples may turn out to be overly costly. On the other hand, too few samples provide us with an inaccurate initial surrogate model, causing a less effective optimisation stage. According to e.g. {{< cite "Toal2015;Forrester2008-" >}}, $N = 10 \cdot d$ is an effective rule of thumb. 

Furthermore, {{< cite "Picheny2013-" >}} states that the size of the initial DoE in noisy optimisation is not critical to the performance of that optimisation, meaning that the effect of having more initial samples is counterbalanced by having to do fewer sequential sampling steps and vice versa. 

# Bibliography
{{< bibliography cited >}}
