# activeFolding_cuPSS

This is the pseduo spectral solver built atop the cuPSS library (arXiv:2405.02410) availabe at https://github.com/fcaballerop/cuPSS (v0.0.1) to solve continuum field equations for a mixture of active liquid crystals and a passive isotropic fluid in 2D on a regular lattice. 

Using the cuPSS library, we can solve generic stochastic PDEs. Here, I only include the solver (solvers/model_ALLPS_gravity) for the equations described below, as discussed in (arXiv:2407.04679).

To use this solver, and the cuPSS library in general, the follwing dependencies are required:
- CUDA toolkit (11+)
- cuFFT
- cuRAND
- FFTW3

Once downloaded, you can compile the solver and the required src/ code using the included compile file comp.sh as:
```
./comp.sh -s model_ALLPS_gravity
```
This compiles the solver "model_ALLPS_gravity" to bin/ which will then be used to solve the equations necessary to describe phase separation between an active and a passive fluid.

The compiled solver can then be conveniently run using:
```
./run/run_activeFolding.sh
```
This solves the model equations in 2D (described below) for the default parameters selected from solvers/model_ALLPS_gravity/default_parameters.txt and edited parametres listed in run/run_activeFolding.sh

It will instantiate an output directory, outDir = projects/activeFolding/{dirTree}. Here, {dirTree} can be modified in run_activeFolding.sh and is typically labelled using a subset of parameter values.
This will output all the simulation parameters used in outDir/sim_parameters.txt and the solver name/location in outDir/logf.txt. 
Additional details can be included in the log file from run_activeFolding.sh.
By default, only the phase field data is included in the output to outDir/data as csv files for each time step saved. Additional fields can be included in the output by modifying model_ALLPS_gravity.cu.

# Continuum model

The model is described in terms of a phase field $\phi$ describing the relative concentrations of the two fluid phases, a nematic tensor field $Q$ to describe the local orientation order, and a flow field $v$ as a Stokes' flow generated by $\phi, Q$:

$$\partial_t \phi + v \cdot \nabla \phi = M \nabla^2 (a_\phi \phi + b_\phi \phi^3 + \kappa_\phi \nabla^2 \phi), $$

$$\partial_t Q = \lambda A  - \omega\cdot Q + Q\cdot \omega +\dfrac{1}{\gamma_Q} H, $$

$$0 = \eta \nabla^2 v - \nabla P + \nabla \cdot \sigma + f_g,$$

where the flow is assumed to be incompressible ($\nabla \cdot v = 0$). $A_{ij} = \dfrac{1}{2} (\partial_i v_j + \partial_j v_i ), \omega_{ij} = \dfrac{1}{2} (\partial_i v_j - \partial_j v_i )$ are the strain rate and the vorticity of the flow field respectively. The nematic molecular field, 
$$H_{ij} = a_Q \tilde{\phi} Q_{ij} + b_Q \rm{Tr}{Q^2} Q_{ij} + K_Q \nabla^2 Q_{ij},$$
is derived from a Landau-de Gennes free energy. With $\phi_0 = \sqrt{-a_\phi/b_\phi}, \tilde{\phi} = (1+\phi/\phi_0)/2$ is the volume fraction of the active phase.


The stress in the Stokes equation consists of the capillary stress ($\sigma^\phi$), elastic stress ($\sigma^e$) and the active stresses ($\sigma^a$) as:
$$\sigma_{ij}^\phi = -\hat{k_\phi} (\nabla_i \phi \nabla_j \phi - \frac12 \delta_{ij} (\nabla \phi)^2)$$
$$\sigma_{ij}^e = -\lambda H_{ij} + Q_{ik} H_{kj} - H_{ik}Q_{kj}$$
$$\sigma_{ij}^a = \alpha \tilde{\phi} Q_{ij}$$


## Transition from asymmetric interface fluctuations to active folding:
Representative snapshots of the steady state configuration of the phase field are shown here. Here the active phase ($\tilde{\phi}=1$) is shown in cyan and the passive phase ($\tilde{\phi}=0$) in black, for increasing activity at two different values of the interface surface tension

![image](https://github.com/paarthgulati/activeFolding_cuPSS/assets/64762728/9f0ce2b5-f8f7-4d3f-8c48-d0f73b0f435d)

The figure captions have the parameter values listed in physical units (described in detail in the paper). To recreate these simulation results, use the default values included with the solver and set activtiy and phase field paramters as $\alpha^\rm{sim} = -|\alpha|/10$ and $-a_{\phi}^\rm{sim} = b_{\phi}^\rm{sim} =\gamma/10$ and $\kappa_\phi^\rm{sim} = \hat{\kappa_\phi}^\rm{sim} = 1.50 \times (\gamma/10)$.
