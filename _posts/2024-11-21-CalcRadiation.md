---
layout: post
title:  Modelling High-frequency Radiation with PIC 
categories: Research calc_radiation
excerpt: Particle-in-cell (PIC) simulations are a ubiquitous tool to model the dynamics of charged particles during laser-plasma interactions. However, they typically cannot resolve electromagnetic radiation in the ultraviolet and x-ray regimes for a number of reasons. Calc_radiation is a module I have written to enable the fully-relativistic PIC code EPOCH to simulate the emission of coherent radiation from charged particles.

header-includes:
  - \usepackage{algorithm2e}
---

Particle-in-cell (PIC) simulations form the backbone of computational plasma physics. They have proven incredibly useful in modelling everything from the interstellar medium to high intensity laser plasma interactions (LPI). There are two basic ideas behind the PIC method:
1. Electromagnetic fields exist on a *discrete* grid
2. Particles are grouped together using *macroparticles*.
<br>
![](/images/CalcRadiation/PIC_ApproxAndCycle.png)
*A summary of the particle-in-cell algorithm. a) Field quantities ($\vec{E}$ and $\vec{B}$, for example) exist on a discretized grid, while the positions and momenta of macroparticles can vary continuously. b) On each iteration, the algorithm loops through the four steps of the PIC cycle.*
<br>

In some sense, PIC is the simplest possible way one could think to simulate a plasma. Despite this apparent simplicity, PIC simulations give rise to a number of computational issues that limit their scope. Many books and innumerable papers have been written about the nuances of the PIC method, and every computational plasma physicist needs to keep their limitations in mind as they design their simulations. While PIC simulations are good for a lot things, they struggle to model high-frequency electromagnetic radiation.

The biggest reason PIC cannot simulate high-frequency radiation is the discrete timestep, $\Delta t$. PIC approximates Maxwell's equations as difference equations, so we introduce a $\Delta t$ in order to solve for $\vec{E}$ and $\vec{B}$. If we ignore numerical error, this means we're sampling the fields with a frequency $f_s = 1/\Delta t$. In the ideal case, the maximum frequency we can resolve is the Nyquist frequency, $f_\mathrm{max} = 0.5 f_s$. Anything with a higher frequency than $f_\max$ will be *aliased* to a frequency below the Nyquist limit. At this point, you may be asking yourself, 
>"why can't we just make $\Delta t$ smaller?

This can be answered with a simple scaling calculation. Imagine we wanted to perform a 1D simulation where an IR laser produces x-rays through betatron radiation. The walltime will be proportional to the number of macroparticles $N$, the length $X$ and resolution $\Delta x$ of the spatial grid, and the sampling length $T$ and resolution $\Delta t$. In order for the simulation to be numerically stable, we must have $\Delta x \approx c \Delta t$, where $c$ is the speed of light. Then we have,
$$
\begin{align}
    \mathrm{Walltime} = N \left( \frac{X}{\Delta x} \right)\left(\frac{T}{\Delta t} \right) \propto \frac{1}{(\Delta t)^2}
\end{align}
$$
To resolve 1 keV x-rays, we'd need to make $\Delta t$ 1000 times smaller than a simulation looking at that only resolves the IR laser. This means our walltime increases by a factor of 1,000,000! In 3D, this would be even higher. Even on cutting edge computing clusters, this simulation would take too long to be feasible. In reality, there are other reasons why we can't just make $\Delta t$ smaller, especially if we want to model coherent radiation. Electromagnetic waves travelling on a discrete grid don't have the same dispersion relation as light travelling in continuous space. This means that the speed of light for high frequency radiation is different than that of IR light, so there will be spurious interference that may affect the results of the simulation. Needless to say, we need a better solution than simply reducing $\Delta t$.

<hr>
# calc_radiation

Luckily, Einstein has come to save the day. When a relativistic charged particle interacts with laser, it experiences *time dilation*, meaning time passes slower in its reference frame than the lab frame. When it accelerates, it will release radiation with frequency $\omega$ in its rest frame. However, in the lab frame, the light is blue-shifted to a higher frequency depending on the particle's speed, $\omega_\mathrm{det} \approx 2\gamma^2 \omega$. Because of this, a virtual detector can have a much higher temporal resolution than the simulation itself.
$$
\begin{align}
    \Delta t_\mathrm{det.} \approx \frac{\Delta t_\mathrm{sim}}{2 \gamma^2}
\end{align}
$$

`calc_radiation` is a physics package I have written that takes advantage of this, enabling us to simulate XUV to soft x-ray radiation using PIC simulations. We have implemented `calc_radiation` in my [fork](https://github.com/bunzicker/epoch/tree/develop_calc_radiation) of the fully-relativistic PIC code [EPOCH](https://epochpic.github.io/). Our implementation closely follows the algorithm presented by Pardal, *et al.* in their paper ["RaDiO: An efficient spatiotemporal radiation diagnostic for particle-in-cell codes"](https://www.sciencedirect.com/science/article/pii/S0010465522003538). The basic steps are shown below in a pseudocode representation.

<pre id="calc_rad" class="pseudocode">
    % This quicksort algorithm is extracted from Chapter 7, Introduction to Algorithms (3rd edition)
    \begin{algorithm}
    \caption{Calculate Radiation}
    \begin{algorithmic}

    \FOR {t\_sim}
        \FOR {particle}
            \STATE $\vec{r}_{prev}$ = \CALL{GetLocation}{particle} 
            \STATE $\vec{\beta}_{prev}$ = \CALL{GetVelocity}{particle}
            \STATE

            \STATE $\vec{r}, \; \vec{\beta}$ =\CALL{PushParticle}{}
            \STATE $E_{part} = $ \CALL{GetEnergy}{$\vec{\beta}$}

            \STATE
            \STATE $\vec{a} = $ $(\vec{\beta} - \vec{\vec{\beta}_{prev}})/\Delta t$
            \STATE


            \FOR {pixel}
                \STATE $\vec{r}_{det} = $ \CALL{GetLocation}{pixel}
                \STATE $\vec{R} = \vec{r} - \vec{r}_{prev}$

                \STATE

                \STATE $t^{det} = t_{sim} + |\vec{R}|/c$
                \STATE

                \IF {$E_{part} \geq E_\min$ \AND $t_\min^{det} \leq t^{det} \leq t_\max^{det}$}
                    \STATE $\vec{\mathcal{E}} = $ \CALL{CalculateFieldAtDetector}{$\vec{\beta}, \vec{a}, \vec{R}$}
                    \STATE \CALL{InterpolateField}{$\vec{\mathcal{E}}, \; t^{det}, \; \vec{r}_{det}$}
                \ENDIF

            \ENDFOR
        \ENDFOR
    \ENDFOR

    <!-- \PROCEDURE{PushParticle}{}
    \ENDPROCDEURE -->
    \end{algorithmic}
    \end{algorithm}
</pre>
`calc_radiation` circumvents many of the problems associated with simulating high frequency radiation with PIC. By defining a virtual detector with its own time grid, we can resolve much higher frequencies than on the PIC grid--without decreasing $\Delta t$! This also lets us directly mimic experiment, because we can place the virtual detector at an arbitrary point in space. `calc_radiation` is also a dispersionless algorithm, because the high frequency light doesn't propagate to the grid. Instead, it is calculated directly at each pixel in the detector. This maintains the relative phase between different frequencies, which can be important if you're trying to study attosecond pulse generation, for example. In future posts, I will go into more detail about how `calc_radiation` works, the physics behind it, and some of the limitations. I'll also detail the key role it has played in my PhD. work simulating XUV radiation with tunable amounts of orbital angular momentum. For now though, I just wanted to introduce the code and show some of its capabilities.

A simple example demonstrating `calc_radiation` is high harmonic generation when a high intensity laser is incident on an overdense plasma. The laser will cause the electron density to oscillate, imparting a time-dependent phase shift in the reflected light, which produces XUV radaition. Typical PIC simulations cannot resolve this high frequency radiation without a very fine spatial grid. Using `calc_radiation`, we are able to resolve signifcantly higher frequencies than the PIC field solver. Results are shown below.

<br>
![](/images/CalcRadiation/PICVsCalcRad_HHG.png)
*The radiation emitted by a laser-driven overdense plasma using the PIC field solver (black) and calc_radiation. As can be seen, calc_radiation resolves much higher frequency radiation than the PIC grid. The most energetic frequencies are not affected by grid dispersion in calc_radiation, unlike the field solver that is affected heavily by aliasing.*

<!-- Render the pseudocode algorithm -->
<script>
    pseudocode.renderElement(document.getElementById("calc_rad"), 
    {lineNumber: true});
</script>
<script>
    pseudocode.renderClass("pseudocode");
</script>