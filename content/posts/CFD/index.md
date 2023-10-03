---
title: "Computational Fluid Dynamics: brief method overview"
subtitle: ""
date: 2023-10-02T21:48:35+02:00
draft: false
description: ""
summary: "" 

tags: [CFD]
categories: [CFD]
series: "Master's thesis"

featuredImage: ""
featuredImagePreview: ""

bibFile: "library-EVA-bib.json" 
---

<style>
#fig-cutcell_definition {
    margin-top: 1rem;
}
</style>

This post provides a brief overview of the Computational Fluid Dynamics (CFD) solver during my [thesis](../../projects/thesis/), which is dubbed EVA and developed by {{< cite "EVA_Eijk2022-" >}}. EVA is able to solve two-phase (in-)compressible fluid flows and uses consistent two-way fluid-structure interactions (FSI), where applying this to moving bodies together with a Volume-of-Fluid (VoF) cut-cell method on a staggered arrangement of variables is a novelty of EVA {{< cite "EVA_Eijk2022" >}}.

The goal of this post is to provide sufficient insight into the CFD solver used in this work, such that there is some understanding of the simulation process and the related challenges encountered in this thesis.
This post is not meant to provide details on the level of complete reproducibility of the CFD code. A more detailed and complete description of EVA is given by {{< cite "EVA_Eijk2022-" >}}.

<!-- %
%- novelty of Eva: EVA is consistent: mass en momentum fluxes are matched; zuzio, but then for moving bodies / FSI
%- omdat huidige velocityveld exact resultaat geeft, blijft EVA consistent -> novelty?
% -->

This post will consider some core concepts of EVA in the following order:

- [Governing equations](#governing-equations).
- [Cell-labelling, interface reconstruction, and cut-cell definition](#cell-labelling-interface-reconstruction-and-cut-cell-definition).
- [Transport of fluid and bodies](#transport-of-the-fluids-and-body).
- [Solving method](#solving-method).


<!-- %Cell labelling is nodig voor interface reconstruction (en cut-cell identification).
%interface reconstruciton is een subroutine van transport
%transport wordt expliciet gedaan met het oude velocity field $u^n$
%adhv de laatste interface reconstructie na transport wordt de cut-cell voor momentum-equations gedefnieerd.
%momentum equations worden opgelost. -->

# Governing Equations
EVA solves the discrete 2D Navier-Stokes equations for mass and momentum conservation in partial differential conservative form, see $\cref{eqn:continuity_equation}{1}$ and $\cref{eqn:momentum_equation}{2}$. $\cref{eqn:continuity_equation}{1}$ is regularly referred to as the continuity equation. EVA employs the one-fluid formulation with a single velocity field, a single pressure field and a thereof resulting fluid mixture density, thereby only requiring one set of equations for all the fluids combined.

\begin{equation}\label{eqn:continuity_equation}
\frac{\partial \rho}{\partial t}+\nabla \cdot(\rho \mathbf{u})=0
\end{equation}
\begin{equation}\label{eqn:momentum_equation}
\frac{\partial(\rho \mathbf{u})}{\partial t}+\nabla \cdot(\rho \mathbf{u}\otimes\mathbf{u})+\nabla p-\nabla \cdot\left(\mu\left(\nabla \mathbf{u}+\nabla \mathbf{u}^{T}\right)-\frac{2}{3} \mu \nabla \cdot \mathbf{u}\right)-\rho \mathbf{f}=0
\end{equation}

Here $\mathbf{u} = \left(u,v\right)$ $[m/s]$ is the fluid velocity vector, $p$ the fluid pressure, $\nabla = \left(\frac{\partial}{\partial x},\frac{\partial}{\partial y}\right)$ is the gradient operator, $\mu$ $[Pa\cdot s]$ the dynamic viscosity of the fluid (mixture), and $\mathbf{f}$ $[kg/ m^2 s^2]$ represents the body forces comprised of the gravitational acceleration and capillary stresses. Lastly, $\rho$ $[kg/m^3]$ is the fluid mixture density, of which {{< cite "EVA_Eijk2022-" >}} provide a detailed description.

<!-- %EVA uses the one-fluid formulation to represent multi-phase flows, where the main advantage is that $$\cref{eqn:continuity_equation,eqn:momentum_equation}{TODO}${TODO}$ can be formulated for all fluids at once instead of requiring a linked set of these equations for each fluid, thereby simplifying the linear system of equations. In the one-fluid formulation all the fluids and materials are described using a single velocity field under the no-slip assumption, a single density field and one pressure field.  -->

To achieve consistent mass-momentum transport at the interfaces, EVA additionally solves a temporary auxiliary continuity equation {{< cite "EVA_Zuzio" >}}. 
<!-- % Applying this consistent two-way FSI methodology to moving bodies is a novelty of EVA {{< cite "EVA_Eijk2022" >}}. -->

<!-- % two phases was fluid and material interfaces -->
Because we are involved with the transport of two phases the transport equation $\cref{eqn:VoF_transport}{3}$ is solved. Here, the one-fluid velocity field results in the assumption of no-slip between the fluids and therefore the fluid velocity can be used instead of the interface velocity.

\begin{equation}\label{eqn:VoF_transport}
\frac{\partial F}{\partial t}+\mathbf{u} \cdot \nabla F=0
\end{equation}
Here, $F(x,t) = 0$ provides the location of the interface in space and time. To solve $\cref{eqn:VoF_transport}{3}$, EVA employs the Volume of Fluid (VoF) method {{< cite "EVA_Hirt1981" >}}, for which the continuous colour function $F$ is replaced by a discrete volume fraction $C$ with a value between 0 and 1; now indicating the ratio of the cell volume occupied by the corresponding phase. The discretized $\cref{eqn:VoF_transport}{3}$ is then solved using an [advection scheme](#fluxing-schemes). 

<!-- %Body displacement method (wel degelijk belangrijk ook voor thesis) -->
The rigid body is displaced explicitly using: 
<!-- %by integrating the body velocity $\mathbf{u}_b$ over time $t$ -->
\begin{equation}
\frac{\partial \mathbf{x}_b}{\partial t}=\mathbf{u}_b
\end{equation}

Here, the body velocity $\mathbf{u}_b$ is found through Newton's second law, using the known constant body mass $m_b$:

\begin{equation}
m_b \frac{\partial \mathbf{u}_b}{\partial t}=\mathbf{F}_b
\end{equation}

$\mathbf{F}_b$ is composed of the gravitational force and the normal pressures of the fluid integrated over the body boundary surface. Viscous stresses are neglected in the calculation of the body forces since EVA focuses on the dynamics of impacts: an event within a short-time period during which viscous effects such as boundary layers cannot develop. 

<!-- %- Consistent: the mass and momentum fluxes are matched through an auxiliary continuity equation within the velocity control volumes (zuzio) -->

# Cell-labelling, interface reconstruction, and cut-cell definition
The [one-fluid formulation](#governing-equations) that EVA uses relies on the accurate identification and definition of the (cells on the) interface between the different phases to perform correct fluid calculations. This section introduces the steps, definitions and notations needed to achieve this.

<!-- %VoF -> interface -> cutcell kan uitdrukking geven aan interface maar cutcells zijn vooral belangrijk voor correct momentum.  -->
Roughly, the procedure is as follows: 

- [Cell labelling](#cell-labelling): Based on a known (updated) Volume of Fluid fraction field $C$, each cell is given a label defining its composition and relation to neighbouring cells.
- [Interface reconstruction](#interface-reconstruction): Using the cell labelling and the volume fraction field, the fluid and material interfaces are geometrically reconstructed.
- [Cut-cell definition](#cut-cell-definition): The reconstructed interfaces are then used to define the cut-cell geometries, which are eventually used to correctly solve the momentum equations.

## Cell labelling
Cell labelling is used by EVA to distinguish between different possible scenarios and configurations of a (cut-)cell, indicating what computational routines should be used during the fluid calculations and interface reconstruction. 

The cell labelling is explained by means of [Fig. 1](#fig-cutcell_labelling), while I refer to {{< cite "EVA_Eijk2022-" >}} for a complete description of the cell labelling process.

{{< image src="cutcell_labelling_updated.png" width="40%" caption="Figure 1: Figure is adapted from {{< cite EVA_Eijk2022- >}}, and is an enhanced version of the labelling scheme of {{< cite EVA_Kleefsman- >}}. Present cell labels are: E(mpty), B(ody), S(urface), F(luid), (C)orner.  The C(orner) cell label has been introduced by {{< cite EVA_Eijk2022- >}} for fluid cut-cells that are diagonally connected to exactly one E(mpty) cell. Taking such corner cells into account improves the accuracy of the reconstruction and simulation {{< cite EVA_Eijk2022 >}}." id="fig-cutcell_labelling" >}}
<!-- %More detailed definitions of all labels are found in {{< cite "EVA_Eijk2022-" >}} -->

## Interface reconstruction
The interface is being approximated by a line (or a plane in three dimensions) using the Piecewise-Linear Interface Calculation (PLIC) method {{< cite "EVA_Youngs1984;EVA_Duz" >}}. For each 2D cell the interface is approximated using:

\begin{equation}
\boldsymbol{m\cdot x} = m_xx + m_yy = \alpha
\end{equation}

Here $\boldsymbol{m}$ is the local surface normal, $\boldsymbol{x}$ is the position vector of a point on the interface and $\alpha$ is the plane constant placing the line at a distance from the origin of the cell. $\boldsymbol{m}$ is calculated using the neighbouring volume fraction field and its gradients, where {{< cite "EVA_Duz-" >}} considers various methods to do so and EVA makes use of the cell labelling. In 2D, the plane constant $\alpha$ is analytically calculated by matching the volume resulting from the combination between the surface normal $m$ and the plane constant $\alpha$ with the required fraction volume $C_f$.
<!-- % In 3D, we are confronted with additional challenges {{< cite "EVA_Duz" >}}. -->

During the interface reconstruction, EVA is able to use the appropriate calculations by using the cell-labelling, where {{< cite "EVA_Eijk2022-" >}} show that the introduced C-labelling provides significant improvements. 

<!-- %Furthermore, EVA corrects the PLIC reconstruction by matching the body apertures of neighbouring cells if needed while maintaining a constant fraction volume.% $C_b$ and $C_f$. -->


## Cut-cell definition
Given a known [reconstructed fluid interface](#interface-reconstruction), this section defines the thereof resulting cut-cell geometry {{< cite "EVA_Kleefsman;EVA_Fekken" >}}. The material interface cuts through a regular fixed Cartesian grid-cell and thus divides the cell into different regions, which is why such a cell is called a cut-cell. The cut-cell geometry provides the definitions and control volumes that are important for correct momentum calculations at the body interface, seeing that the body is rigid and should not partake in the fluid calculations. 


<!-- %During fluid-structure interaction simulations in multi-phase flows we are involved with the interfaces between different fluids or structures.  -->

<!-- %verkeerde volgorde: VoF -> interface -> cutcell kan uitdrukking geven aan interface maar cutcells zijn vooral belangrijk voor correct momentum. -->

[Fig. 2](#fig-cutcell_definition) visually provides the full cut-cell definition using two situations:                                
- Subfigures (a), (b): Air is cut by the body (dark grey)
- Subfigure (c): $\hspace{13pt}$ Additionally, the air is cut by fluid (light grey) 

{{< image src="Cutcell definition.PNG" id="fig-cutcell_definition" caption="Figure 2: Figure from {{< cite EVA_Eijk2022- >}}. Various properties corresponding to the cut-cell with index (i, j) are portrayed. The mass control volume is indicated by the red dashed line, with its volume indicated by $V_c$. The solid body is indicated by the dark grey area. (b): $u_b$ and $v_b$ are the body velocities. The centre of the control volume is given by o. (c): Fluid is indicated by a light grey area." >}}

In the first scenario we are involved with the body volume fraction $C_{b,i,j}$ while in the second scenario we are additionally involved with the fluid volume fraction $C_{f,i,j}$. These volume fractions describe the portion of the cell volume ($=\delta x_i \delta x_j$ in a 2D structured grid) that is occupied by the corresponding phase.
<!-- %Note that these volume fractions are the discretisation of the Volume of Fluid variable $F$ as used in $\cref{#governing-equations,ssec:EVA_fluxing}{TODO}$. -->

[Fig. 2a](#fig-cutcell_definition) portrays the body apertures $a_b$, which defines the portion of each of the cell's edges that is open to flow between cells and defines the size of the control volume. $\delta x_i$ and $\delta y_j$ are respectively the horizontal and vertical size of grid cell (i,j).

In this section, I do not consider the added complexity introduced by the methodology of {{< cite "EVA_Zuzio-" >}}, used by EVA to simultaneously be momentum- and mass-conserving. This methodology defines additional control volumes that leave us with a more complicated sub-grid cut-cell formulation. For an account on this matter, I refer to {{< cite "EVA_Zuzio;EVA_Eijk2022-" >}}.


# Transport of the fluids and body
This section summarizes the methods that EVA uses for solving the transport equation $\cref{eqn:VoF_transport}{3}$ for both the fluid- and body volume fraction fields using a known velocity field $\boldsymbol{u}$ and the [interface reconstruction procedure](#interface-reconstruction). 

First, a [definition of flux](#flux-definition), whereafter the various advection or [fluxing schemes](#fluxing-schemes) of EVA are discussed. Lastly, I discuss the [controller](#cfl-controller) that ensures numerical stability during fluxing.
 
## Flux definition
Flux is the transport or flow of a quantity through a surface. In the case of EVA, that surface is the side of a cell defined in the structured grid as a 2D line with unit depth. The transported quantity is a material or fluid represented by the volume fractions $C_b$ or $C_f$. 

For an uncut fluid cell in a structured grid, the (average) fluid flow over that side is represented using a single velocity normal to the surface. The horizontal flux over the right side of cell (i,j) is then:
<!-- %It is the surface integral of the normal velocities over that surface, see $\cref{eqn:flux}. = \iint_S \boldsymbol{u}{TODO}$\cdot \boldsymbol{\hat{n}}\text{ }dS -->

\begin{equation}\label{eqn:flux}
\delta C_{f,i+\frac{1}{2},j} = u_{i+\frac{1}{2},j}\cdot \delta t
\end{equation}  

In the case of a cut-cell the flux calculation is more complicated due to the presence of interfaces. [Fig. 3](#fig-flux_explanation) depicts the scenario of a cut-cell with both a body and fluid interface, where we are again interested in the flux of the fluid fraction over the right cell boundary. The amount of fluxed fluid is limited by both the body-fluid interface and the air-fluid interface.

{{< image 
	src="flux_explanation2.png" width="50%" caption="Figure 3: Figure from {{< cite EVA_Eijk2022 >}}. Flux in a showcase cut-cell at location (i,j) in the structured grid. The solid body is indicated by the dark grey area and the fluid by the light grey area. The fluxed fluid amount is illustrated by the blue cross-hatched area."
	id="fig-flux_explanation" >}}
<!-- %Martins caption text: Flux calculated in a cut cell representation of a continuity control volume. Body is indicated by . Fluid (liquid) is indicated by . Center (i, j) is given by . The amount of fluid being transported (fluxed) is hatched with (–). -->

## Fluxing schemes
This section summarizes the methods that EVA uses for solving the discretized transport equation $\cref{eqn:VoF_transport}{3}$ for both the fluid- and body volume fraction fields. {{< cite "EVA_Duz-" >}} provide an elaborate account of these methods and identifies two approaches: using an unsplit advection scheme or an operator-split advection scheme.  

In the unsplit advection method, hereafter referred to as UNSPLIT, the body fraction $C$ and corresponding body interface is propagated within a single update $C^n \rightarrow C^{n+1}$ according to a known velocity field. Although this only requires one [fluid interface reconstruction](#interface-reconstruction) step and thus is cheap, we encounter problems for multi-directional body velocities. For instance, with 2D diagonal body movement the same interfacial volume can be doubly advected while none of the corresponding body fraction is actually displaced diagonally, but only vertically and horizontally in an isolated fashion. This process is called overlapping {{< cite "EVA_Comminal2015" >}} and results in mass loss. Similarly non-overlapping creates gaps leading to diffusion. Thereby, UNSPLIT is prone to mass conservation problems and does not inherently preserve the body shape and interface.

On the other hand, operator-split advection schemes flux the body volumes sequentially and separately per direction. As such they are inherently mass conserving and able to advect volumes to diagonally neighbouring cells within a single timestep. 

EVA and {{< cite "EVA_Duz" >}} consider two operator-split advection schemes:

- MACHO: Multidimensional Advective-Conservative Hybrid Operator
- COSMIC: Conservative Operator Splitting for Multidimensions with Inherent Constancy

COSMIC considers each sequential update ordering permutation at once, while MACHO only considers one such permutation per timestep. Instead, MACHO alternates between these permutations during subsequent timesteps to reduce directional bias. Therefore, COSMIC is in principle more accurate and reliable but much more expensive, requiring additional interface reconstructions for each permutation. 

The 2D MACHO scheme is formulated:

\begin{equation}\label{eqn:MACHO_permutation}
\begin{aligned}
C^{\*} &= C^{n} - \Delta t \frac{\partial u C^{n}}{\partial x} + \Delta t C^{n} \frac{\partial u}{\partial x}, \\\\
C^{n+1} &= C^{n} - \Delta t \left(\frac{\partial u C^{n}}{\partial x} + \frac{\partial v C^{\*}}{\partial y}\right),
\end{aligned}
\end{equation}

To clarify, in 2D MACHO alternates between $\cref{eqn:MACHO_permutation}{8}$ and:

\begin{equation}
\begin{aligned}
C^{\*} &=C^{n}-\Delta t \frac{\partial v C^{n}}{\partial y}+\Delta t C^{n} \frac{\partial v}{\partial y}, \\\\
C^{n+1} &=C^{n}-\Delta t\left(\frac{\partial u C^{\*}}{\partial x}+\frac{\partial v C^{n}}{\partial y}\right),
\end{aligned}
\end{equation}


The 2D COSMIC scheme is formulated:
\begin{equation}
\begin{aligned}
C^{X} &=C^{n}-\Delta t \frac{\partial u C^{n}}{\partial x}+\Delta t C^{n} \frac{\partial u}{\partial x} \\\\
C^{Y} &=C^{n}-\Delta t \frac{\partial v C^{n}}{\partial y}+\Delta t C^{n} \frac{\partial v}{\partial y}, \\\\
C^{n+1} &=C^{n}-\Delta t\left[\frac{\partial}{\partial x}\left(u \frac{C^{n}+C^{Y}}{2}\right)+\frac{\partial}{\partial y}\left(v \frac{C^{n}+C^{X}}{2}\right)\right]
\end{aligned}
\end{equation}

In 2D, UNSPLIT requires one interface reconstruction (once per time-step update), MACHO requires two, and COSMIC requires three interface reconstructions. In 3D however, UNSPLIT still requires one interface reconstruction, MACHO requires three, and COSMIC requires a significant amount of 10 interface reconstructions.
For the full definitions of MACHO and COSMIC in 3D, I refer to {{< cite "EVA_Duz" >}}.

## CFL controller
To maintain numerical stability during fluxing, EVA employs a time-step controller based on the Courant–Friedrichs–Lewy (CFL) condition {{< cite "EVA_CFL" >}}. The CFL controller prevents fluxing more fluid than there is present inside a cell. It decreases the time-step size $\Delta t$ if the flux is too high anywhere in the grid, and similarly increases it when fluxes can be higher without violating the condition. This ensures that at any time during the simulation we use the appropriate timestep size, thereby balancing numerical efficiency and stability. The CFL condition in 1D is defined as:
\begin{equation}
\frac{u \Delta t}{\partial x} \overset{?}{<=} CFL_{max}
\end{equation}

Since EVA operates in a structured grid we choose to apply this rule per dimension. The dimensionless number $CFL_{max}$ is called the Courant number. For $CFL_{max} = 1$ (with the valid range of $C_{max}$ defined by $0<CFL_{max} <= 1$), the condition entails that the displacement of some piece of fluid cannot be larger than a grid cell's size a given dimension. In other words, we should not be able to 'skip' an entire grid cell during a timestep, which would lead to numerical instability. 
<!-- %If the condition is not met somewhere within the grid, we need to reduce the timestep size $\Delta t$ until the condition is being met everywhere.  -->

Generally, a $CFL_{max}$ lower than 1 provides more numerical stability and is often required. Of course, this potentially comes at the cost of smaller used time-steps and therefore higher simulation times.

# Solving method
This section considers how EVA solves the Navier-Stokes equations ($\cref{eqn:continuity_equation}{1}$ and $\cref{eqn:momentum_equation}{2}$).
In basis, to solve the discretized Navier-Stokes equations EVA employs the so-called \textit{pressure correction} projection approach {{< cite "EVA_Eijk2022" >}}. EVA does so by using a second-order upwind space integration scheme and the Adam-Bashforth time integration scheme {{< cite "EVA_Eijk2022" >}}. The complete solving method of EVA includes additional steps to calculate effective densities at the VoF interface to enable compressible fluid calculations. These density calculations are in turn related to the method of {{< cite "EVA_Zuzio-" >}} for consistent calculations at the fluid interfaces. Since these topics are considered too in-depth and non-relevant for the goals of this work, I refer to {{< cite "EVA_Marnix;EVA_Eijk2022-" >}} for a complete description of the solving method. Here, I will provide a conceptual explanation of the pressure correction method.

The pressure correction approach consists of what seems a basic iterative
procedure between the velocity and the pressure fields {{< cite "EVA_Hirsch2007" >}} but which turns out to be directly solvable. For an initial approximation of the pressure field, the momentum equation $\cref{eqn:momentum_equation}{2}$ is solved to determine an (intermediate) velocity field $u^*$ containing all explicit terms like convection, diffusion and body forces. The obtained intermediate velocity field does not satisfy the divergence-free property imposed by the continuity equation $\cref{eqn:continuity_equation}{1}$ and should therefore be corrected. Since this velocity correction impacts the pressure field by virtue of the same momentum equation, a related pressure correction should be defined by requiring the corrected velocity to satisfy the continuity equation. The resulting system of equations leads to a Poisson equation for the pressure correction, which can be solved using a linear system solver of choice. 

For a full explanation, derivations and implementation details of the pressure correction method, I refer to {{< cite EVA_Hirsch2007- 625-637 >}}.

# Bibliography
{{< bibliography cited >}}

