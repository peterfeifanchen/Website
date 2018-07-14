---
layout: post
title:  "Minding the Billions: Ultra-wideband Localization for Deployed RFID Tags"
date:   2018-7-12 15:40:56 -0800
categories: systems
tags: [ Paper Review ]
---

<span style="color:red">Disclaimer:</span> All images, tables and graphs are sourced 
from the original paper itself and are not my own. The full link to the paper can be 
found [here](http://www.mit.edu/~fadel/papers/RFind-paper.pdf). 

<h1>Problem</h1>

Precise localization can enable smart environments. RFIDs are cheap and ubiquitous,
making them ideal for use in localization. Traditional methods use angle of arrival
(AoA) or received signal strength (RSS) to narrow the location to tens of cm. More
recent methods modify the environment with reference tags and move the RFID reader
over many wavelengths to narrow the location to cm-scale. This paper instead uses
time of flight (TOF) to accurately pinpoint RFID location to within centimeters.
By using multiple antennas, 2D/3D localization can be performed without the need to
modify the RFID tags or the environment. 

The challenge of using TOF multipath: reflected radio signals from the RFID may not 
take the line-of-sight (LOS), but bounce off of walls in the environment.
Multipaths in the environment can be resolved with a sufficient large bandwidth. 
However, for centimeter-level accuracy, this would require GHz of bandwidth, while RFIDs
are allocated only tens of KHz (resulting in accuracy of kilometers). 

<h1>Background</h1>

In order to understand the paper, we will cover extensive ground in wireless propagation,
mathematics (Fourier Transforms), the relationship between bandwidth and TOF, RFID operation,
phase noise, antenna systems and channel estimation. 

**Wireless Transmission**

A signal that propagates can be modeled as waves spreading from a radiating source. These
waves, like water waves, bounce off of walls and interfere with each other at a point. The
received signal can be modeled as the sum of these interfering waves at the point of the
receiver:

<a href="https://www.codecogs.com/eqnedit.php?latex=r&space;=&space;\sum_i&space;\alpha_is(t-\tau_i)&space;=&space;s(t)&space;*&space;h(t)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?r&space;=&space;\sum_i&space;\alpha_is(t-\tau_i)&space;=&space;s(t)&space;*&space;h(t)" title="r = \sum_i \alpha_is(t-\tau_i) = s(t) * h(t)" /></a>

where h(t) is a series of delta functions:

<a href="https://www.codecogs.com/eqnedit.php?latex=h(t)&space;=&space;\sum_i&space;\delta(t-\tau_i)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?h(t)&space;=&space;\sum_i&space;\delta(t-\tau_i)" title="h(t) = \sum_i \delta(t-\tau_i)" /></a>.

Each path is modified by α<sub>i</sub> which encapsulates path loss (function of distance) and
attenuation (medium/reflection). The sum of these paths (r in the above equation) captures the 
effect called fading. 

The function s(t) is the transmitted waveform. Mathematically, this contains the information that
we wish to transmit (modulation) and pulse-shaping (transmit) or matched filtering (receive - the
paper will mention this later). The modulation transforms bits into symbols (e.g. MQAM). These 
symbols are then sent one by one to be pulse-shaper which modifies the signal such that 1) it 
meets the signals bandwidth limitations and 2) protect it against intersymbol interference when
it convolutes with the transmission channel h(t). [Discontinuity and noise experiment] 

Below, an example of a two path model is shown. This is what we call <span style="variable">
multipath</span> nature of wireless signals. While the path loss and attenuation of a signal 
varies predictably, the fading caused by multipath is a random result of the environment and 
positioning of the transmitter and receiver. The issue with using TOF to caculate distance can
so be rephrased as one of how to decompose the received signal such that we see not the 
superposition of mulitple path but only that of the line-of-sight (LOS). 

<figure>
<img src="/assets/Papers/RFind/raytrace.jpg">
<figcaption>Example for two paths between receiver and transmitter</figcaption>
</figure>

The sum of these two paths can be alternatively expressed as the "convolution" of s(t) and h(t).
Since calculating, reasoning and writing convolutions can be annoying, analysis of wireless 
signals are usually take place in the Fourier domain as expressed below. 

**Fourier**

Continuous Time Fourier Transforms

Discrete Time Fourier Transforms

Discrete Fourier Transforms

**Bandwidth and TOF** 

**RFID** 

**Phase Noise**

**Basic Antenna Theory**

**Channel Estimation**

<h1>Contribution</h1>

<h1>Solution</h1>


<h1>Implementation</h1>

<h1>Evaluation</h1>

<h1>Related Work</h1>

<h1>TidBits</h1>
