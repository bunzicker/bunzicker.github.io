---
layout: post
title:  Phase Diversity - Phase Retriveal
categories: Research  PDPR
excerpt: Often the phase of a laser is just as important as the intensity distribution, but great care must be taken to measure both simulataneously. One method to do so is a phase retrieval algorithm known as Phase Diversity - Phase Retrieval.
---

Often when we perform measurements using light, we just want to know how bright a laser is or perhaps the light's spatial distribution. We can do this using a photodiode or a camera to image the beam. Unfortunately, these types of measurements only capture the light's intensity and completely miss its phase. But in many cases, the phase of a beam of light can be just as important as the intensity, especially in my work. 

Much of the work I have done in the latter half of my PhD. has focused on light with orbital angular momentum (OAM). This is a special type of light where the phase twists around in a spiral along the propagation axis, so it is vital that we be able to measure the phase reliably. 

<br>
![](/images/PhaseRetrieval/PhaseFronts.png)
*Phase isosurfaces for a light wave (a) with and (b) without orbital angular momentum (OAM). In beams with OAM, surfaces of constant phase corkscrew around the propagation axis several times depending on the amount of OAM they posses.*
<br>

However, phase measurements are often difficult. The most common way to measure phase is through interferometry, where a beam with unknown phase is interfered with a beam whose phase is well characterized. This technique has been broadly applied across every domain of physics, chemistry, metrology, and more to measure spatial, temporal, and spectral phase distributions in a number of different geometries (such as Mach-Zehnder, Michelson, and Fabry-Perot interferometers). Yet interferometry can be difficult for a number of reasons. Firstly, it requires a special setup that increases the complexity of an experiment. This is particularly restrictive at ultrahigh intensities where the beam is $\approx 6$ inches in diameter. Optics of this size can be very expensive, and experimental chambers are rarely big enough to fit them. My work presents another difficulty; interferometry works best when the two beams have the same wavelength. This becomes a big issue when you're trying to study extreme ultraviolet (XUV) light. Very few facilities have an XUV source co-located with a petawatt-class laser, so we have to find another solution to measure the phase of XUV light with OAM.

## Phase Diversity - Phase Retrieval
An alternative way to measure the spatial phase distribution is to use a phase retrieval algorithm. In [`hedp.pdpr`](https://github.com/bunzicker/hedp/tree/main/pdpr), I have implemented a phase diversity - phase retrieval (PDPR) algorithm that uses the known intensity distributions and a propagator to determine the complex fields.

The basic idea behind PDPR is demonstrated below. First we measure the laser's intensity distribution ${I_i}$ at several locations ${z_i}$. The magnitude of the complex electric field is related to the intensity $\|\mathcal{E}\| = \sqrt{I_i}$. Then we propagate the complex field in one plane to next and constrain the calculated intensity using the measured intensity. Eventually, this process will converge to the correct phase.

<br>
![](/images/PhaseRetrieval/PDPR_Figure_NoFields.png)
<br>


### Rayleigh-Sommerfeld Diffraction Integral
There are many possible propagators that can be used in PDPR. The most general one is the Rayleigh-Sommerfeld diffraction integral. The dynamics of a laser are governed by the paraxial Helmholtz equation.
    \begin{equation}
    \nabla^2 \mathcal{E} = -\frac{\omega^2}{c^2} \mathcal{E}
    \end{equation}
where $\mathcal{E}$ is the complex electric field. We can use a Green's function to write a propagator that is an exact solution to the wave equation
    \begin{align}
        \mathcal{E}_1 = \frac{|z_1 - z_0|}{2 \pi} \int dx' dy' \; \frac{\mathcal{E}_0 (x', y', z_0 )}{r^2} \exp \left( i k r \right) \left[ik - \frac{1}{r} \right]
        \label{eq:RS_Integral}
    \end{align}
where $k = \omega / c$ is the laser wavenumber and $r = \sqrt{(x - x')^2 + (y - y')^2 + (z_1 - z_0)^2}$. This propagator can be interpreted through Huygens's principle: every point on the plane $z = z_0$ is a source of spherical waves that interfere in the $z_1$ plane to form $\mathcal{E}_1$.

Even though Eq. \ref{eq:RS_Integral} may look complicated, it can be efficiently calculated using the Fast Fourier Transform (FFT) and the convolution theorem. Closer analysis of the form of $\mathcal{E}_1$ reveals that is actually a convolution between $\mathcal{E}_0$ and a transfer function $h(x, y)$
    \begin{align}
        \mathcal{E}_1 = \mathcal{E}_0 * h(x, y)
        \label{eq:Convolution}
    \end{align}
where 
    \begin{align}
        h(x, y; \Delta z) = \frac{|\Delta z|}{2 \pi}  \frac{1}{R^2} \exp \left( i k R \right) \left[ik - \frac{1}{R} \right]
    \end{align}
and $R = \sqrt{x^2 + y^2 + (\Delta z)^2}$. If we Fourier transform Eq. \ref{eq:Convolution} using the FFT, we can then use the convolution theorem to write 
    \begin{align}
        \mathcal{E}_1 = \mathcal{F}^{-1} \Big[ \mathcal{F}(\mathcal{E}_0) \mathcal{F}(h) \Big]
    \end{align}
where $\mathcal{F}$ and $\mathcal{F}^{-1}$ are the Fourier and inverse Fourier transforms, respectively. This lets us calculate $\mathcal{E}_1$ very efficiently, even for relatively large images.

### hedp.pdpr

`hedp` is a python library that has a number of packages I've written throughout my PhD. One of the subpackages is `hepd.pdpr` where I have implemented the PDPR algorithm to measure the phase of light using intensity distributions at several different locations. The main function is `recover_phase`, and a pseudo-code implementation is shown below. Variables of the form $\mathcal{E}_i$ are the guess electric fields, while $U_i$ are fields where the intensity constraint has been applied.

1. Initialize $U_0$ using a random phase, $U_0 = \sqrt{I_0}*\exp \left( i \; \mathrm{Rand}(x', y')\right)$
2. Calculate $\mathcal{E}_1$ by propagating $U_0$ using `propagator`.
3. Apply the intensity constraint, $U_1 = \sqrt{I_1} \exp\left(i \; \arg{\mathcal{E}_1} \right)$ 
4. Calculate $\mathcal{E}_0$ by propagating $U_1$.
5. Apply the intensity constraint on $U_0$ 
6. Calculate error between $\mathcal{E}_i$ and $\sqrt{I_i}$. Exit code if error is small enough.
7. Repeat Steps 2 - 6 until convergence.

`recover_phase` has been implemented to handle an arbitrary number of images. The only limitation is that the $(x, y)$ coordinates of every image must be the same. This is consequence of using the FFT to calculate $\mathcal{E}_i$. For more information, I suggest reading the code itself.

### Results

As a first test, I of my PDPR algorithm, I have repeated the measurements presented in Chapter 8 of Dr. Nicholas Czapla's [thesis](https://etd.ohiolink.edu/acprod/odb_etd/etd/r/1501/10?clear=10&p10_accession_num=osu1658486928321502) where he measured the OAM of the Scarlet laser using a different PDPR implementation. He found highly efficient conversion to the $\ell = 1$ Laguerre-Gaussian mode using a stepped spiral phase mirror. My PDPR algorithm obtains the same results while only spending 1% the computational time. 

<br>
![](/images/PhaseRetrieval/PDPR_NickData.png)
*The intensity (top) and recovered phase (bottom) of the Scarlet laser with OAM at three different locations ($z = $ -20, 0, and 20 $\mathrm{\mu m}$). The contours in the phase plots delineate the 10% intensity contour for each location.*
<br>