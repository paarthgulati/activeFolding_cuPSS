# activeFolding_cuPSS

This is the pseduo spectral solver built atop the cuPSS library (arXiv:2405.02410) availabe at https://github.com/fcaballerop (v0.0.1) to solve continuum field equations for a mixture of active liquid crystals and a passive isotropic fluid in 2D on a regular lattice. 

Using the cuPSS library, we can solve generic stochastic PDEs. Here, I only include the solver (solvers/model_ALLPS_gravity) for the equations described below, as discussed in (link).

To use this solver, and the cuPSS library in general, the follwing dependencies are required:
- CUDA toolkit (11+)
- cuFFT
- cuRAND
- FFTW3

Once installed, you can compile the solver using the comp.sh file as:
```
comp.sh -s model_ALLPS_gravity
```

This compiles the solver "model_ALLPS_gravity" which will be used to solve the equations necessary to describe phase separation between an active and a passive fluid.

This can then be run using:
```
run/run_activeFolding.sh
```
This solves the follwing equations in 2D for the default parameters selected from solvers/model_ALLPS_gravity/default_parameters.txt and edited paramteres listed in run/run_activeFolding.sh

# Continuum model

The model is described in terms of a phase field $\phi$ describing the relative concentrations fo the two fluid phases, a nematic tensor field $Q$ describe the local orientation order, and a flow field $v$ as a Stokes flow generated by $\phi, Q$:

$$\partial_t \phi + v \cdot \nabla \phi = M \nabla^2 (a_\phi \phi + b_\phi \phi^3 + \kappa_\phi \nabla^2 \phi), $$

$$\partial_t Q = \dfrac{\lambda}{2} A  - \omega\cdot Q + Q\cdot \omega +\dfrac{1}{\gamma_Q} H, $$

$$0 = \eta \nabla^2 v - \nabla P + \nabla \cdot \sigma + f_g,$$

where the flow is assumed to be incompressible ($\nabla \cdot v = 0$), $A_{ij} = \dfrac{1}{2} (\partial_i v_j + \partial_j v_i ), \omega_{ij} = \dfrac{1}{2} (\partial_i v_j - \partial_j v_i )$ are the strain rate and the vorticity of the flow field respectively. The nematic molecular field, 
$$H_{ij} = a_Q \tilde{\phi} Q_{ij} + b_Q \rm{Tr}{Q^2} Q_{ij} + K_Q \nabla^2 Q_{ij},$$
is derived from a Landau-de Gennes free energy. Here, $\tilde{\phi} = (1+\phi/\phi_0)/2,$ where $\phi_0 = \sqrt{-a_\phi/b_\phi},$ is the volume fraction of the active phase.


The stress in the Stokes equation consists of the capillary stress ($\sigma^\phi$), elastic stress ($\sigma^e$) and the active stresses ($\sigma^a$) as:
$$\sigma^\phi = -\hat{k_\phi} (\nabla_i \phi \nabla_j \phi - \frac12 \delta_{ij} (\nabla \phi)^2)$$
$$\sigma^e = -\lambda H_{ij} + Q_{ik} H_{kj} - H_{ik}Q_{kj}$$
$$\sigma^a = \alpha \tilde{\phi} Q_{ij}$$
