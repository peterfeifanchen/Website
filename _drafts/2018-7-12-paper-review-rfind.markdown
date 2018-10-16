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
receive - the paper will mention this later). The modulation transforms bits into symbols 
(e.g. MQAM). These symbols are then sent one by one to the pulse-shaper which modifies the signal
such that 1) it meets the signals bandwidth limitations and 2) protect it against intersymbol 
interference when it convolutes with the transmission channel h(t). 

<span style="color:blue">[Discontinuity and noise experiment]</span>
<span style="color:blue">[Spectrum of pulse-shaping with raised cosine vs. square wave experiment]</span>

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
more tractable. The Continuous Time Fourier Transform (CTFT) takes a time domain signal x(t) that may be periodic in T or aperiodic and returns its fourier transform X(jω):

\\[ X(j\omega) = {\int_{-\infty}^\infty x(t)e^{-jwt} dt} \\]
\\[ x(t) = {\int_{-\infty}^\infty X(j\omega)e^{jwt} dw} \\]

Fourier transform can be found by taking the fourier series of a periodic signal and taking the 
limit of its period T to infinity. In the case where x(t) is periodic, its Fourier transform and 
its fourier series are equivalent. CTFT are related to fourier series of continuous signals by:

\\[ x(t) = {\sum_k a_ke^{jkw_ot}} \\]
\\[ a_k = \frac{1}{T}X(j\omega)\Bigr|_{w=kw_o} \\]
\\[ w_o = \frac{2\pi}{T} \\]

However, in real modern digitized systems, we seldom deal with continuous time signals. We 
usually sample the signal (on receive) into its discrete representation and pulse shape a 
discrete time signal into a continuous time signal (on transmit) for transmission over the
channel. Fourier transform for discrete time signals are called Discrete Time Fourier Transforms
(DTFT):

\\[ x[n] = \frac{1}{2\pi}\int_{2\pi}X(e^{jw})e^{jwn}dw \\]
\\[ X(e^{jw}) = \sum_{n=-\infty}^{\infty}x[n]e^{-jwn} \\]

Like CTFT, DTFT is derived by taking the Discrete Fourier Series (DFS) of a periodic discrete 
signal with its period taken to infinity. Like DFS, DTFT is periodic over \\( 2\pi \\) since 
\\( e^{jw} = e^{j(w+2\pi)} \\). It's higher frequency components are around multiples of 
\\( \pi \\) while lower frequency components are at multiples of \\( 2\pi \\).

The discrete time signal must be produced from a sampling rate that is twice the maximum
bandwidth of a bandlimited continuous ttime signal (Nyquist Theorem). As sampling in time domain 
represents a train of delta functions, in the frequency domain, this is equivalent to the 
convolution of the CTFT of the signal with the CTFT of the delta train 
\\( X(e^{jw}) = \frac{2\pi}{N}\sum_{k=-\infty}^{\infty}\delta(w-\frac{2{\pi}k}{N}) \\). Here,
N = T or the sampling period. 

Finally, DTFT is actually transformed into Discrete Fourier Transforms (DFT). This is because
for DTFT of a discrete time signal is actually continous, this makes digital processing still
hard. DFT however are discrete both in time and frequency domains allowing easy manipulation. A
detailed description of DFT and its relationship with DTFT can be found 
[here](http://www.cambridge.org/ba/files/3113/6681/5698/4421_Chapter_12_-_Discrete_Fourier_transform.pdf).

In summation, DFT can be thought of as the DTFT of a windowed time-domain sampled signal which
is then sampled in the frequency domain at interval of \\( {\Delta}f \\). Like DTFT, DFT are 
periodic over \\( 2\pi \\). For a M-point DFT, each DFT bin represents \\( \frac{2\pi}{M} \\) 
for \\( {\Delta}f = \frac{1}{MT_s} \\) where \\( T_s \\) is the sampling rate and 
\\( \frac{1}{T_s} \\). The equations for DFT then becomes:

\\[ x[n] = \frac{1}{M}\sum_k^{M-1}X[k]e^{j2{\pi}kn/M} \; for \; 0 \leq k \leq N-1 \\]
\\[ X[k] = \sum_k^{N-1}X[k]e^{j2{\pi}kn/M} \; for \; 0 \leq k \leq M-1 \\]

The DFT has an interesting property since it is the result of the frequency-space 
sampling of the signal's DTFT. The sampling caues time-domain signal to be aliased at an 
interval M. If this interval is longer than the signal length, then no aliasing occurs. 
DFT is completely described over the interval M. Otherwise, another name for this time domain
aliasing is circular convolution. This means that for y[n] = x[n] * h[n], when we multiply 
their DFT, we are performing a circular, not linear convolution, unless the DFT length M is 
greater than the length of x and h combined. Further, if a signal falls in between two DFT
bins, rather than a single delta, it becomes a peak spread throughout the adjacent bins. 

<span style="color:blue">[ CTFT->DTFT->DFT experiment ]</span>
<span style="color:blue">[ DFT carry-add filter experiment ]</span>

**Pulse Shaping**
Pulse shaping is basically the D/A converter for the baseband signal. Basically, the discrete
time signal is convoluted with a basic pulse. This can be a on-off (square wave), a sinc or
the many different combinations in between. The shape of the time-domain pulse must have the
property of being exactly equal to the discrete time signal at intervals \\( nT_s \\). In a 
way pulse shaping is similar to low-pass filtering. Ideal pulse will be flat in its passband
and cut-off sharperly to eliminate leaks into adjacent frequencies. 

<span style="color:blue">[ Pulse Shaping experiment ]</span>

**Match Filter**
On the receive side, we would like to maximize the signal-to-noise ratio of our desired signal.
For a given time domain signal \\( y(t) = \int_{-\infty}^{\infty} \\)

More detailed information on matched filters can be found [here](https://dsp.stackexchange.com/questions/9094/understanding-the-matched-filter).

<span style="color:blue">[ Match Filter Experiment ]</span>

**Channel Estimation**

<span style="color:blue">[ Match Filter + Channel Estimation Experiment ]</span>

**Bandwidth and TOF** 

[ Bandwidth TOF experiment -> Power Delay Profile ]

**RFID** 

**Phase Noise and Carrier Frequency Offset**

[ Symbol experiment with Phase Noise and CFO ]

**Basic Antenna Theory**

<h1>Solution</h1>

<h1>Implementation</h1>

<h1>Tidbits</h1>

I got curious on how AoA is calculated. 

