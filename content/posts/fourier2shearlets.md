---
author: "Quinten Pinkhof"
title: "From Fourier to Shearlet"
date: "2026-01-10"
description: "What are Shearlets? Why would we care? And how do they relate to Fourier?"
tags: ["shearlets", "wavelets", "fourier"]
ShowToc: true
TocOpen: false
math: true
---
Nine months ago I started a PhD. Until now, I have been working with shearlets, and will probably remain working with them until my dissertation. It seems like the shearlet transformation is the most logical thing in the world now, but once upon a time it was quite the opposite. 

Through this post, I'd like to give researchers and enthusiasts an easy go-to for understanding shearlets, saving them interaction with dense mathematical papers (even though I got carried away and included some formulas).

My explanation rests on a basic understanding of the Fourier transform, so maybe let's just start with Fourier.

# Fourier
In its essence, the Fourier transform takes a signal or function $x(t)$ as an input, and gives you another function $X(f)$ which lets you look at $x(t)$, but through a 'frequency' lens ($f$) instead of a 'time' ($t$) lens. We say that $X(f)$ is the fourier equivalent of $x(t)$, with equivalent meaning that $X(f)$ describes the same thing as $x(t)$ and you can switch between any one of the two representations at any time. 

### What is watching through a 'frequency' lens? 
To understand this, let's first take a look at the vectors in figure 1. 
If you have two 2D linearly independent vectors, meaning they are not sitting on the same line[^1], we can combine the two vectors to form any other vector in the 2D space ($\vec{u}$ and $\vec{v}$ span the space). To do this, we scale the two vectors with scale factors $a$ and $b$ respectively (we say $\vec{z}$ is a linear combination of basis elements $\vec{u}$ and $\vec{v}$):

$$\vec{z} = a\vec{u} + b\vec{v}$$

So in a 2D space, we can break down any vector $\vec{z}$ into its $\vec{u}$ and $\vec{v}$ contributions, which are $a$ and $b$ respectively. This is called decomposition. The tough part is actually finding a way to calculate $a$ and $b$. If you know the coefficients and forgot the value of $\vec{z}$, you can just recalculate it.

{{< figure src="/fourier2shearlets/images/r2decomp.svg" align="center" caption="Figure 1">}}

Why does this matter for Fourier? 
Fourier uses a set of waves with different frequencies to decompose the signal f(t) into (~ basis elements $\vec{u}$ and $\vec{v}$ ), and the Fourier transform $X(f)$ tells you exactly what the contribution of the wave with frequency $f$ is (~ way to calculate the coefficients $a$ and $b$). 

One note: the Fourier contribution coefficient is a complex number. This way, we have a real and imaginary part working together to describe the amplitude of the wave (how tall it is) and the phase of the wave (where its peak starts)

### Why would anyone use this?
Why would we ever care about describing a signal in frequencies, and, therefore, the Fourier transform?

It boils down to two things:
1) There are multiple physical domains which work with frequencies. Think about a piano: each note is a frequency. If we can decompose a music piece in its frequency contributions, we know which notes are being used!
2) In the equivalent Fourier domain, mathematics works differently, and in some cases a hard operation in the time domain is an easy operation in the frequency domain. Sometimes performing the transform, the easy operation and the inverse transform is easier than performing the hard operation.

### The formula and resolution trade-off
The Fourier Transform takes the following form:

$$X(f) = \int_{-\infty}^{\infty} x(t) e^{-i2\pi f t} dt$$

The most complex part of this formula is $e^{-i2\pi f t}$. This is a short notation (Euler's formula) for 

$$ e^{-i2\pi f t} = \cos(-2\pi f t) + i\sin(-2\pi f t) = \cos(2\pi f t)  - i\sin(2\pi f t)$$

The real cosine and imaginary sine together describe a wave of frequency $f$ with an initial phase of zero and an amplitude of 1. By multiplying with $x(t)$, we scale the "base" wave at every point t, based on $x(t)$. If the 'peaks and valleys' of your signal $x(t)$ align with the 'peaks and valleys' of the sine and cosine waves, the product remains mostly positive, and the integral accumulates into a large coefficient. If they don't match, the product oscillates above and below zero, and the integral cancels itself out. The final result is a single complex number where the magnitude tells you the total strength of that frequency across the entire signal, and the angle tells you the specific phase shift required to align that 'base' wave with the actual patterns found in $x(t)$.


 If you understand this, it should be clear that we are calculating the $f$ wave's contribution to the **whole** signal. You want to know the frequency contribution to a smaller window? Well sure we can apply the Fourier transform to the smaller window signal and look at the frequency contributions. However, we need to watch many cycles of our base waves to know precisely which frequencies are active in our (partial) signal: if we look at a small window in time, a wave with 100 Hz and with 101 Hz almost look identical, making it harder to distinguish who contributes what. If we get more cycles to look at though, the difference becomes clearer. Think about it like trying to determine if a car is going $100\text{ km/h}$ or $101\text{ km/h}$ by watching it move for only $10\text{ cm}$. You just don't have enough "runway" to measure the difference accurately. This is a fundamental trade-off between time resolution and frequency resolution: the better you want to know when a pop (high frequency) happens, the worse you know which frequency the pop really is (its more like a blob of high frequencies than a peak). If you want to know more precisely what exact frequency is present, you need a large window and you will lose precision in knowing when that exact frequency is present. If you want to have $\Delta f$ frequency resolution (i.e. able to distinguish between $f$ and $f + \Delta f$), you need at least a time window of $\frac{1}{\Delta f}$ ([Gabor 1946](https://jcsphysics.net/lit/gabor1946.pdf)). 

Things become even trickier when you realise that a difference between 1 Hz and 2 Hz is relatively way larger than a difference between 10,000 Hz and 10,0001 Hz. It just so happens to be that humans perceive sound and light logarithmically, which means we are more sensitive to changes in low frequencies than in high frequencies, which means we need more precision at the bottom of the frequency scale! So we have to increase our window size to be able to accommodate these large low-frequency resolutions we desire, but in the process we lose time resolution -- and all this while we already had the desired high-frequency resolution!  This my friends, is what gave way to wavelets and shearlets.

### Side quest: Logarithmic perception
We say humans perceive light intensity and sound intensity logarithmically (Fechner 1860). This is the same thing as saying our senses only care about relative changes, not absolute changes. Why is this the same?

Scientists have figured out through that perception P has a logarithmic relationship with intensity I:

$$ P = \log(I) \text{\hspace{5mm}(very much simplified so)}$$ 

This corresponds to figure 2, where you can easily see that the difference in between perception between two high intensities is much smaller than the difference in perception between two low intensities. So logarithms have something to do with relativity. But we would like to proof that two equal relative intensity changes result in two equal absolute perception changes:

We have

$$ \Delta p_{12} = p_2 - p_1 = \log_{10}(i_2) - \log_{10}(i_1) $$

furthermore, since $10^{\log_{10}(x)} = x$, we also have $$10^{\log_{10}(i_2) - \log_{10}(i_2)} = \frac{10^{\log_{10}(i_2)}}{10^{\log_{10}(i_1)}} = \frac{i_2}{i_1} = 10^{\log_{10}(\frac{i_2}{i_1})}$$

which means

$$ \log_{10}(i_2) - \log_{10}(i_1) = \log_{10}(\frac{i_2}{i_1}) $$

which finally gets us to
$$\Delta p_{12} = \log_{10}(\frac{i_2}{i_1})$$ 

So if $\frac{i_2}{i_1} = \frac{i_4}{i_3}$ then $\Delta p_{12} = \Delta p_{34} $

Therefore, our use of a logarithmic relationship is correct, as this is exactly what we wanted perception to be: an equal relative increase in intensity is an equal absolute increase in perception.

{{< figure src="/fourier2shearlets/images/ylogx.svg" align="center" caption="Figure 2" >}}


# From Fourier to Wavelet
So we've established that Fourier has a fundamental problem: it forces us to choose between time resolution and frequency resolution, but we really want both! We want precise time localization for high frequencies (to know exactly when that drum hit happens) and precise frequency resolution for low frequencies (to distinguish between different bass notes). This seems impossible... until wavelets came along. Ingrid Daubechies is one of the key contributors to wavelet theory ([1992](https://doi.org/10.1137/1.9781611970104)), and she is a fellow Belgian, so get your information from her :). 

### The wavelet solution: adaptive windows
The key insight of wavelets is brilliantly simple: use different window sizes for different frequencies. 

- For high frequencies: use narrow windows (good time resolution, acceptable frequency resolution)
- For low frequencies: use wide windows (good frequency resolution, acceptable time resolution)

Think of it like using different camera exposures: for fast-moving objects (high frequency events) you use a fast shutter speed (narrow window), while for slow-moving objects (low frequency events) you use a longer exposure (wide window). 

This is called a multi-scale analysis (scale ~ frequency). Instead of using the same sine and cosine waves at all frequencies like Fourier does, wavelets use a mother wavelet $\psi(t)$ that gets stretched and squeezed to analyze different frequencies, and shifted in time to analyze different moments.



### The wavelet transform
The continuous wavelet transform looks like this:

$$W(a, b) = \frac{1}{\sqrt{|a|}} \int_{-\infty}^{\infty} x(t) \psi^*\left(\frac{t-b}{a}\right) dt$$

Let's break this down:
- $\psi(t)$ is the mother wavelet[^2] - a localized wave that decays to zero on both sides (unlike sine/cosine which go on forever). This property is important, as it allows for much better localization of frequenc (If it decays to zero without ever actually reaching it at any finite time we call these "Infinitely Supported" wavelets, otherwise we say "Compactly Supported").
- $a$ is the scale parameter - it stretches ($a > 1$) or compresses ($a < 1$) the wavelet. Large $a$ means wide wavelet (used for low frequencies), small $a$ means narrow wavelet (for high frequencies).
- $b$ is the translation parameter - it shifts the wavelet in time to analyze different parts of the signal
- $\psi^*$ denotes the complex conjugate (similar to Fourier)
- The $\frac{1}{\sqrt{|a|}}$ factor ensures energy is conserved across scales

So $W(a,b)$ tells you: "How much does the signal at time $b$ match the wavelet at scale characterized by $a$?", but be careful, larger $a$ means less time resolution and smaller $a$ means less frequency resolution!

### Why this works better
#### Perception
Remember our problem from the Fourier section? We needed $\Delta f$ frequency resolution which required a time window of $\frac{1}{\Delta f}$, but the higher the frequency, the less we care for audiovisual applications. 

With wavelets:
- At high frequencies (small $a$), the wavelet is narrow → we get good time resolution but coarser frequency resolution. But that's fine! At 10,000 Hz, we don't care if it's 10,000 or 10,001 Hz.
- At low frequencies (large $a$), the wavelet is wide → we get good frequency resolution but coarser time resolution. But that's fine! At 100 Hz, we want to distinguish 100 Hz from 101 Hz, and we don't mind if our time precision is a bit looser.

This matches human perception perfectly! The relative frequency resolution $\frac{\Delta f}{f}$ stays roughly constant across scales, which is exactly what our logarithmic perception needs.

#### Mother wavelet's family
You might think: why change wave types? Why don't we chop the signal $h$ times with $h$ different window lengths, apply the Fourier transform to each chopped window and then look at the high frequencies for small windows and low frequencies for large windows. And you'd be right, we could do this, but this quickly becomes a nightmare to manage: you can analyze frequencies, but how do you rebuild the signal (inverse transform)? How many times will you have to perform the fourier transform? Using a mother wavelet that is sheared an translated is just way more elegant.

You can swim across a pool in front crawl, or you can tread through it like your dog does (but what a cute dog!).

### The remaining problem
Wavelets solve the time-frequency resolution trade-off beautifully... for 1D signals. But what about 2D signals like images? We could just apply 1D wavelets horizontally and vertically, and indeed this works reasonably well. But it has a fundamental limitation: directional selectivity. Wavelets can tell you about horizontal features and vertical features, but what about diagonal edges? What about curves? An edge at 45 degrees gets split awkwardly between horizontal and vertical wavelet coefficients, leading to inefficiency. This is where shearlets come in, providing optimal directional sensitivity in 2D.

# From Wavelet to Shearlet

We ended the last section with a teaser: wavelets are amazing for 1D signals, but what happens when we move to 2D? Can we just apply wavelets separately in the horizontal and vertical directions? Sure, and it works... sort of. But there's a hidden inefficiency lurking in this approach. Easley, Labate and Lim ([2008](https://doi.org/10.1016/j.acha.2007.09.003), [Kutyniok et al. 2009](https://arxiv.org/abs/math/0605375)) are the founding fathers of the shearlet theory, and they explain clearly what I am saying below. 

### The directional problem
Imagine you have an image with a sharp diagonal edge - maybe the edge of a building against the sky. This edge is essentially a 1D feature (it varies in one direction and is constant perpendicular to that direction). Intuitively, you'd want to represent this edge efficiently with basis functions that are also aligned with that direction.

But here's the catch: if you apply wavelets horizontally and vertically (separable wavelets), that diagonal edge gets detected by both the horizontal and vertical wavelets. You need contributions from both directions to describe a single directional feature! It's like trying to describe a perfectly diagonal line using only horizontal and vertical Lego bricks - sure, you can approximate it, but it's going to require a lot more bricks than if you could just place one diagonal brick.

In mathematical terms, wavelets achieve the optimal sparse representation (few coefficients needed) for point singularities, but they are sub-optimal for line singularities (edges, curves). Since natural images are dominated by edges and curves rather than isolated points, this is a big deal!

### Enter the shear transformation
So how do we add directionality? The key insight is to introduce a shear transformation on top of the scaling. Let's build this up visually:

- **Isotropic scaling** (what Fourier uses): magnifies equally in all directions → no directional preference (like zooming in on an image while keeping the same aspect ratio)
- **Anisotropic scaling** (what wavelets use): different scaling in different directions → creates elongated shapes but still aligned with axes
- **Parabolic scaling + shear** (what shearlets use): anisotropic scaling combined with controlled tilting → creates elongated shapes at any orientation

The shear transformation is beautiful because you can continuously vary the orientation without rotation. Unlike rotation matrices which need trigonometric functions and can be computationally expensive, shear matrices are simple:

$$S_s = \begin{pmatrix} 1 & s \\ 0 & 1 \end{pmatrix}$$

where $s$ controls the amount of shear (and thus the orientation). When $s=0$, you get vertical orientation; as $s$ increases, the orientation tilts.

### The shearlet transform
The continuous shearlet transform combines three operations: scaling, shearing, and translation. For a 2D image $f(x_1, x_2)$, the shearlet transform is:

$$\mathcal{SH}_\psi f(a, s, t) = \langle f, \psi_{a,s,t} \rangle = \int_{\mathbb{R}^2} f(x) \overline{\psi_{a,s,t}(x)} dx$$

where the shearlet $\psi_{a,s,t}$ is defined as:

$$\psi_{a,s,t}(x) = a^{-3/4} \psi(A_{a}S_{s}(x-t))$$

Let's unpack this beast:
- $a > 0$ is the scale parameter (like wavelets, but now in 2D). Smaller $a$ means narrower shearlet for analyzing fine details (but losing frequency resolution!)
- $s \in \mathbb{R}$ is the shear parameter controlling the orientation of the shearlet
- $t = (t_1, t_2) \in \mathbb{R}^2$ is the translation parameter (where in the image we're looking)
- $A_a$ is the parabolic scaling matrix: $A_a = \begin{pmatrix} a & 0 \\ 0 & \sqrt{a} \end{pmatrix}$
- $S_s$ is the shear matrix we saw earlier
- $a^{-3/4}$ is the normalization factor ensuring energy conservation

The parabolic scaling is crucial! Notice how one direction scales by $a$ and the other by $\sqrt{a}$. This creates an elongated shape that gets increasingly needle-like as $a \to 0$, perfect for capturing directional features.

### Why parabolic scaling?
You might wonder: why $a$ and $\sqrt{a}$? Why not just $a$ in both directions? The answer lies in the optimal sparse approximation of edge singularities. 

Consider a smooth curve in an image. At fine scales (small $a$), we want our analyzing function to be:
- Very narrow across the edge (to localize it precisely) → scales as $a$
- Slightly wider along the edge (to capture its smoothness) → scales as $\sqrt{a}$

This creates a width-to-length ratio of approximately $\sqrt{a}:1$, giving us elongated "needles" at fine scales. Mathematical analysis shows this parabolic relationship is optimal for achieving the best decay rate of approximation coefficients for edge-like features.

### The shearlet advantage: optimal sparsity
The benefit of shearlets is reflect in its main property: Shearlet Transformations provide optimally sparse representations for the class of cartoon-like images (smooth regions separated by smooth edges). What does this mean? 

- Sparsity: essential image content is concentrated in a few large coefficients, with the rest of the coefficients being near-zero. This means you can approximate the image quite well by only using the N largest coefficients. 
- Optimal: The approximation error $\varepsilon_N$ when only using the N largest coefficients decays as $\varepsilon_N \leq C(\log(N)^{3})N^{-2}$ and this is faster than any other transformation for the class of cartoon like images (wavelets are $\varepsilon_N \leq CN^{-1}$ ).

Now you say 'Ok but I don't have many images that are cartoon like', and you'd be right to mention this! Real-world images can contain content (e.g., textures) that deviates from the cartoon-like class and may not admit sparse shearlet representations. However, edges carry the majority of semantically or diagnostically relevant information, and are a good approximation of the majority of natural images.


# The big picture
Let's zoom out and see the progression:
1. **Fourier**: Perfect for global frequency analysis, but a bad tool for time/space localization
2. **Wavelets**: Added multi-scale analysis → solved time-frequency trade-off for 1D
3. **Shearlets**: Added directionality with parabolic scaling → solved directional representation for 2D

Each transform builds on the previous one, adding capabilities while maintaining computational feasibility. Fourier gives us the frequency perspective, wavelets add the scale perspective, and shearlets add the directional perspective.



Note: Curvelets achieve the same thing as shearlets, but the set of analyzing functions is not cleanly derived from one family -- the shearlet transformation is way more elegant.




[^1]: More generally, a set of vectors is said to be linearly independent if none of the vectors can be formed through combination of the others. 
[^2]: There are many choices for the mother wavelet $\psi(t)$, each with different properties (symmetry, compactness, smoothnes...)

# References

[1] Dennis Gabor. ["Theory Of Communication"](https://jcsphysics.net/lit/gabor1946.pdf). (1946).

[2] Gustav Theodor Fechner. "Elemente der Psychophysik". (1860).

[3] Ingrid Daubechies. ["Ten Lectures on Wavelets"](https://doi.org/10.1137/1.9781611970104). (1992).

[4] Glenn Easley, et al. ["Sparse directional image representations using the discrete shearlet transform"](https://doi.org/10.1016/j.acha.2007.09.003). (2008).

[5] Gitta Kutyniok, Demetrio Labate. ["Resolution of the wavefront set using continuous shearlets"](https://arxiv.org/abs/math/0605375). arXiv preprint at arXiv:math/0605375 (2009).