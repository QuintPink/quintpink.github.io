---
author: "Quinten Pinkhof"
title: "WIP - From Fourier to Shearlet"
date: "2026-01-10"
description: "What are Shearlets? Why would we care? And how do they relate to Fourier?"
tags: ["shearlets", "fourier"]
ShowToc: true
TocOpen: false
math: true
---
Nine months ago I started a PhD. Until now, I have been working with shearlets, and will probably remain working with them until my dissertation. It seems like the shearlet transformation is the most logical thing in the world now, but once upon a time it was quite the opposite. 

Through this post, I'd like to give researchers and enthusiasts an easy go-to for understanding shearlets, saving them interaction with dense mathematical papers (apologies to my promotor, who writes some of those dense shearlet papers). 
My explanation rests on a basic understanding of the Fourier transform, so maybe let's just start with Fourier.

### Fourier
In its essence, the Fourier transform takes a signal or function $x(t)$ as an input, and gives you another function $X(f)$ which lets you look at $x(t)$, but through a 'frequency' lens ($f$) instead of a 'time' ($t$) lens. We say that $X(f)$ is the fourier equivalent of $x(t)$, with equivalent meaning that $X(f)$ describes the same thing as $x(t)$ and you can switch between any one of the two representations at any time. 

#### What is watching through a 'frequency' lens? 
To understand this, let's first take a look at the vectors in figure 1. 
If you have two 2D linearly independent vectors, meaning they are not sitting on the same line[^1], we can combine the two vectors to form any other vector in the 2D space ($\vec{u}$ and $\vec{v}$ span the space). To do this, we scale the two vectors with scale factors $a$ and $b$ respectively (we say $\vec{z}$ is a linear combination of basis elements $\vec{u}$ and $\vec{v}$):

$$\vec{z} = a\vec{u} + b\vec{v}$$

So in a 2D space, we can break down any vector $\vec{z}$ into its $\vec{u}$ and $\vec{v}$ contributions, which are $a$ and $b$ respectively. This is called decomposition. The tough part is actually finding a way to calculate $a$ and $b$. If you know the coefficients and forgot the value of $\vec{z}$, you can just recalculate it.

{{< figure src="/fourier2shearlets/images/r2decomp.svg" alt="z=au+bv" align="center" caption="2D vector decomposition as an analog to Fourier decomposition" />}}

Why does this matter for Fourier? 
Fourier uses a set of waves with different frequencies to decompose the signal f(t) into (~ basis elements $\vec{u}$ and $\vec{v}$ ), and the Fourier transform $X(f)$ tells you exactly what the contribution of the wave with frequency $f$ is (~ way to calculate the coefficients $a$ and $b$). 

One note: the contribution coefficient is a complex number. This way, we have a real and imaginary part working together to describe the amplitude of the wave (how tall is it) and the phase of the wave (where should its peak start)

#### Why would anyone use this?
Why would we ever care about describing a signal in frequencies, and, therefore, the Fourier transform?

It boils down to two things:
1) There are multiple physical domains which work with frequencies. Think about a piano: each note is a frequency. If we can decompose a music piece in its frequency contributions, we know which notes are being used!
2) In the equivalent Fourier domain, mathematics works differently, and in some cases a hard operation in the time domain is an easy operation in the frequency domain. Sometimes performing the transform, the easy operation and the inverse transform is easier than performing the hard operation.

#### The formula and resolution trade-off
The Fourier Transform can be described with the following beautiful formula (or daunting, depending on who you ask).

$$X(f) = \int_{-\infty}^{\infty} x(t) e^{-i2\pi f t} dt$$

The most complex part of this formula is $e^{-i2\pi f t}$. This is a short notation (using Euler's formula) for 

$$ e^{-i2\pi f t} = cos(-2\pi f t) + isin(-2\pi f t) = cos(2\pi f t)  - isin(2\pi f t)$$

The real cosine and imaginary sine together describe a wave of frequency $f$ with an initial phase of zero and an amplitude of 1. By multiplying with $x(t)$, we scale the "base" wave at every point t, based on $x(t)$. If the 'peaks and valleys' of your signal $x(t)$ align with the 'peaks and valleys' of the sine and cosine waves, the product remains mostly positive, and the integral accumulates into a large coefficient. If they don't match, the product oscillates above and below zero, and the integral cancels itself out. The final result is a single complex number where the magnitude tells you the total strength of that frequency across the entire signal, and the angle tells you the specific phase shift required to align that 'base' wave with the actual patterns found in $x(t)$.


 If you understand this, it should be clear that we are calculating the $f$ wave's contribution to the **whole** signal. You want to know the frequency contribution to a smaller window? Well sure we can apply the Fourier transform to the smaller window signal and look at the frequency contributions. However, we need to watch many cycles of our base waves to know precisely which frequencies are active in our (partial) signal: if we look at a small window in time, a wave with 100 Hz and with 101 Hz almost look identical, making it harder to distinguish who contributes what. If we get more cycles to look at though, the difference becomes clearer. Think about it like trying to determine if a car is going $100\text{ km/h}$ or $101\text{ km/h}$ by watching it move for only $10\text{ cm}$. You just don't have enough "runway" to measure the difference accurately. This is a fundamental trade-off between time resolution and frequency resolution: the better you want to know when a pop (high frequency) happens, the worse you know which frequency the pop really is (its more like a blob of high frequencies than a peak). If you want to know more precisely what exact frequency is present, you need a large window and you will lose precision in knowing when that exact frequency is present. If you want to have $\Delta f$ frequency resolution (i.e. able to distinguish between $f$ and $f + \Delta f$), you need at least a time window of $\frac{1}{\Delta f}$ 

Things become even trickier when you realise that a difference between 1 Hz and 2 Hz is relatively way larger than a difference between 10,000 Hz and 10,0001 Hz. It just so happens to be that humans perceive sound and light logarithmically, which means we are more sensitive to changes in low frequencies than in high frequencies, which means we need more precision at the bottom of the frequency scale! So we have to increase our window size to be able to accommodate these large low-frequency resolutions we desire, but in the process we lose time resolution -- and all this while we already had the desired high-frequency resolution! This my friends, is what gave way to wavelets and shearlets.

#### Side quest: Logarithmic perception
We say humans perceive light intensity and sound intensity logarithmically. This is the same thing as saying our senses only care about relative changes, not absolute changes. Why is this the same?

Scientists have figured out through that perception P has a logarithmic relationship with intensity I:

$$ P = log(I) $$ (very much simplified)

This corresponds to figure 2, where you can easily see that the difference in between perception between two high intensities is much smaller than the difference in perception between two low intensities. So logarithms have something to do with relativity. But we would like to proof that two equal relative intensity changes result in two equal absolute perception changes:

We have

$$ \Delta p_{12} = p_2 - p_1 = log_{10}(i_2) - log_{10}(i_1) $$

furthermore, since $10^{log_{10}(x)} = x$, we also have $$10^{log_{10}(i_2) - log_{10}(i_2)} = \frac{10^{log_{10}(i_2)}}{10^{log_{10}(i_1)}} = \frac{i_2}{i_1} = 10^{log_{10}(\frac{i_2}{i_1})}$$

which means

$$ log_{10}(i_2) - log_{10}(i_1) = log_{10}(\frac{i_2}{i_1}) $$

which finally gets us to
$$\Delta p_{12} = log_{10}(\frac{i_2}{i_1})$$ 

So if $\frac{i_2}{i_1} = \frac{i_4}{i_3}$ then $\Delta p_{12} = \Delta p_{34} $

Therefore, our use of a logarithmic relationship is correct, as this is exactly what we wanted perception to be: an equal relative increase in intensity is an equal absolute increase in perception.



#### Discrete vs continuous
Before we move on, there is something of note: most natural signals are continuous, but if we don't know their mathematical formula, we need to describe the signal using measured data points and interpolation. This happens often, so we want to have a way to analyze these discrete signals.

What does this change for Fourier? Not much: 
- the signal $x(t)$ becomes an N-length series of points $x_n$.
- the integral becomes a summation.
- the continuous wave frequency $\omega$ is replaced by frequency index $k$. This index is dimensionless (no Hertz), it needs to know at which rate $f_s$ you sampled your signal.
- $X_k$ tells us the contribution of frequency XX to the series $x_{0:N-1}$.


$$X_k = \sum_{n=0}^{N-1} x_n e^{-i \frac{2\pi}{N} kn}$$


[^1]: More generally, a set of vectors is said to be linearly independent if none of the vectors can be formed through combination of the others. 

### Fourier, Wavelet, and Shearlet walked into a bar...



