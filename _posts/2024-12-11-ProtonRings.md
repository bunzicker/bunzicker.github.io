---
layout: post
title:  Proton Rings from Late-forming Ballistic Sheath Fields
categories: Research
excerpt: My paper describing how ring patterns form in the angular distribution of TNSA proton beams has been published in Physics of Plasmas!
---

My paper describing how ring patterns form in the angular distribution of laser-driven proton beams has been published in [Physics of Plasmas](https://pubs.aip.org/aip/pop/article/31/12/123106/3325445/Proton-rings-from-late-forming-ballistic-sheath?searchresult=1)! This paper is the result of a collaboration between Ohio State and ELI-NP--home to the most powerful laser in the world--and represents the culmination of about two years of work. 

Laser-driven ion acceleration is the one of the most promising potential applications in high energy density physics. For many years, people have tried to produce high quality proton beams with high fluence, low emittance, and tailored energy distributions for use in radiography, healthcare, fusion energy, and more. The most robust ion acceleration mechanism is target normal sheath acceleration (TNSA). In TNSA, an ultrahigh intensity laser heats a thin foil to very high temperatures, so that the electrons can move around freely, forming a *sheath* around the ion population. Now that there is a net charge distribution there is a very strong electric field (> 10$^(12)$ V/m) on the rear surface of the target that accelerates positively charged ions away. TNSA can produce proton beams that have peak energies greater than 100 MeV/u, relatively high fluence, and simple models predict that they should have very small divergence due to the planar symmetry of the target. 

<br>
![](/images/ProtonRings/TNSA_Figure.png)
*A pictorial representation of TNSA. The plasma sheath (blue) accerates protons (yellow) away from the target. The near planar symmetry of the sheath results in protons travelling pretty much perpendicularly to the target.*
<br>

Experiments around the world have successfully generated TNSA proton beams using petawatt-class laser systems, and they often observe ring patterns in the proton angular distribution rather than a well-collimated beam. In a collaboration with a team from ELI-NP, we measured the ring patterns that form when the Scarlet laser is incident on thin liquid crystal targets. To determine their origin, I performed a series of particle-in-cell simulations using the fully-relativisitc plasma simulation code EPOCH, which allowed us to track the dynamics of the high energy TNSA protons as they move away from the target. We found that rings do not form during the laser-plasma interaction (LPI) as we expected. Instead, they form while the beam is in-flight a fairly long time after the LPI.
<br>
<img src='/images/ProtonRings/ExpSetupAndRCF.png' width = 500 />
*a) The experimental setup. b-e) The energy-dependent proton angular distribution measured using radiochromic film stacks. Notice the robust ring pattern that grows as the proton energy increases.*
<br>
<br>
<img src="/images/ProtonRings/Spectrograms.png" height="600"/>
*We tracked the a) divergence, b) longitudinal velocity, and c) radial velocity distributions throughout the simulation for about 12,000 TNSA macroparticles. Notice that the proton divergence increases rapidly at $\approx 80$ fs (indicated by the black line), while the longitudinal velocity has already saturated by this point. This shows that the longitudinal and radial accelerations are decoupled from one another, meaning ring formation happens a long time after the LPI!* 
<br>

The mechanism behind ring formation is actually very similar to TNSA. The proton beam is formed by hot electrons pulling the protons away from the target, so the beam is composed of both protons and hot electrons. These electrons will form another sheath field around the proton beam, and because of the beam's cylindrical symmetry, the field points radially outwards. Once it is far from the target, the dynamics of the beam are determined primarily by this radial sheath field. 
<br>
<img src='/images/ProtonRings/MpDivergenceAndEperp.png' width = 500>
*a) The structure of the proton beam. b) The radial electric field strength in a plane $4 \; \mu$m behind the target. The peak field lies along the edge of the proton cone.*
<br>
In our paper, we develop a phenomenological model to make qualitative predictions about the energy-dependent divergence. We also demonstrate that these proton rings can be eliminated by reducing the preplasma scale length before the arrival of the main pulse. I encourage you to read the paper for more information, and reach out if you have any more questions. 

Lastly, I couldn't have done this without the help of my co-authors: Nick Czapla, Petru Ghenuche, Dan Stutman, Florin Negoita, Domenico Doria, Calin Ur, Mihail Cernaianu and Douglass Schumacher. 