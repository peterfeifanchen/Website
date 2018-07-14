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

\\[ r = {\sum_i \alpha_i s(t-\tau)} = s(t)*h(t) \\]

where h(t) is a series of delta functions:

\\[ h(t) = {\sum_i \delta(t-\tau_i)} \\]

Each path is modified by α<sub>i</sub> which encapsulates path loss (function of distance) and
attenuation (medium/reflection). The sum of these paths (r in the above equation) captures the 
effect called fading. 

The function s(t) is the transmitted waveform. Mathematically, this contains the information that
we wish to transmit (modulation), and pulse-shaping (on transmit) or matched filtering (on 
receive- thepaper will mention this later). The modulation transforms bits into symbols 
(e.g. MQAM). These symbols are then sent one by one to be pulse-shaper which modifies the signal
such that 1) it meets the signals bandwidth limitations and 2) protect it against intersymbol 
interference when it convolutes with the transmission channel h(t). 

[Discontinuity and noise experiment]
[Spectrum of pulse-shaping with raised cosine vs. square wave experiment]

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
signals usually take place in the Fourier domain. 

**Fourier**

Convolution in the time domain becomes multiplication in the Fourier domain, making analysis
more tractable. 

The Continuous Time Fourier Transform (CTFT) takes a time domain signal x(t) that may be 
periodic in T or aperiodic and returns its fourier transform X(jω):

\\[ X(j\omega) = {\int_{-\infty}^\infty x(t)e^{-jwt} dt} \\]
\\[ x(t) = {\int_{-\infty}^\infty X(j\omega)e^{jwt} dw} \\]

The fourier transform of a signal exists if x(t) 1) is absolutely integrable, 2) have a finite
number of maxima/minima within a finite interval (no asymptotes) and 3) finite number of 
finite discontinuities within a finite interval. Fourier transform can be found by taking the 
fourier series of a periodic signal and taking the limit of its period T to infinity. In the 
case where x(t) is periodic, its Fourier transform and its fourier series are equivalent. 
CTFT are related to fourier series of continuous signals by:

\\[ x(t) = {\sum_k a_ke^{jkw_ot}} \\]
\\[ a_k = \frac{1}{T}X(j\omega)\Bigr|_{w=kw_o} \\]
\\[ w_o = \frac{2\pi}{T} \\]

However, in real modern digitized systems, we seldom deal with continuous time signals. We 
usually sample the signal (on receive) into its discrete representation and pulse shape a 
discrete time signal into a continuous time signal (on transmit) for transmission over the
channel. Fourier transform for discrete time signals are called Discrete Time Fourier Transforms
(DTFT):

They are related to CTFT by:

Where the discrete time signal must be produced from a sampling rate that is twice the maximum
bandwidth of a bandlimited signal.

Finally, DTFT is actually transformed into Discrete Fourier Transforms (DFT). This is because
for DTFT of a discrete time signal is actually continous, this makes digital processing still
hard. DFT however are discrete both in time and frequency domains.

[Wikipedia picture]

[ CTFT->DTFT->DFT experiment ]
[ Sampling experiment, does down-converting first help? ]

**Bandwidth and TOF** 

[ Bandwidth TOF experiment ]

**RFID** 

**Phase Noise and Carrier Frequency Offset**

[ Symbol experiment with Phase Noise and CFO ]

**Basic Antenna Theory**

**Channel Estimation**

<h1>Contribution</h1>

<h1>Solution</h1>


<h1>Implementation</h1>

<h1>Evaluation</h1>

<h1>Related Work</h1>

<h1>TidBits</h1>
