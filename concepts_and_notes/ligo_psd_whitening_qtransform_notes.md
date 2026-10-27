# LIGO Signal Processing Notes: PSD, Whitening, and Q-Transforms

## Table of Contents

1. [Signal Model: Signal + Noise](#1-signal-model-signal--noise)
2. [Noise, Mean, Variance, and Power](#2-noise-mean-variance-and-power)
3. [Fourier Transform Refresher](#3-fourier-transform-refresher)
4. [Positive and Negative Frequencies](#4-positive-and-negative-frequencies)
5. [From Fourier Amplitudes to Power](#5-from-fourier-amplitudes-to-power)
6. [Power Spectral Density (PSD)](#6-power-spectral-density-psd)
7. [Why PSD Has Units of 1/Hz](#7-why-psd-has-units-of-1hz)
8. [One-Sided vs Two-Sided PSDs](#8-one-sided-vs-two-sided-psds)
9. [Amplitude Spectral Density (ASD)](#9-amplitude-spectral-density-asd)
10. [How a PSD Is Estimated in Practice](#10-how-a-psd-is-estimated-in-practice)
11. [LIGO's Frequency-Dependent Noise](#11-ligos-frequency-dependent-noise)
12. [Whitening](#12-whitening)
13. [Why Whitening Helps](#13-why-whitening-helps)
14. [Time-Frequency Analysis](#14-time-frequency-analysis)
15. [Short-Time Fourier Transform](#15-short-time-fourier-transform)
16. [Q-Transform](#16-q-transform)
17. [Whitening vs Q-Transform](#17-whitening-vs-q-transform)
18. [LIGO Processing Pipeline at a Glance](#18-ligo-processing-pipeline-at-a-glance)
19. [Notation Glossary](#19-notation-glossary)
20. [Do Not Confuse These](#20-do-not-confuse-these)
21. [Exercises](#21-exercises)

---

## 1. Signal Model: Signal + Noise

A useful starting model for detector data is

$$
h(t) = s(t) + n(t),
$$

where:

- $h(t)$: measured detector strain as a function of time
- $s(t)$: astrophysical gravitational-wave signal
- $n(t)$: detector noise

The noise $n(t)$ is the random time-varying process itself.

Its variance is a property of that process.

---

## 2. Noise, Mean, Variance, and Power

The expected value, or mean, of the noise is

$$
E[n(t)].
$$

A zero-mean noise process satisfies

$$
E[n(t)] = 0.
$$

This does **not** mean the noise is zero. Positive and negative fluctuations can cancel on average.

Example:

$$
n=
\begin{cases}
+1 & \text{with probability }1/2,\\
-1 & \text{with probability }1/2.
\end{cases}
$$

Then

$$
E[n]=0
$$

but

$$
E[n^2]=1.
$$

Variance is

$$
\operatorname{Var}(n)
=
E[n^2] - E[n]^2.
$$

For zero-mean noise,

$$
E[n]=0,
$$

so

$$
\boxed{
\operatorname{Var}(n)=E[n^2].
}
$$

In signal processing, squared amplitude is often called **power**. For zero-mean noise, average power and variance are therefore closely related.

### Important distinction

- **Noise** = the random process $n(t)$
- **Noise variance** = a numerical measure of how strongly that process fluctuates

In ML, people sometimes use "noise" as shorthand for a noise scale or noise variance. In signal processing, keeping these distinct is useful.

---

## 3. Fourier Transform Refresher

The Fourier transform converts a time-domain function into a frequency-domain representation.

Using one common convention,

$$
\tilde n(f)
=
\int_{-\infty}^{\infty}
n(t)e^{-2\pi i f t}\,dt,
$$

where:

- $t$: time in seconds
- $f$: frequency in hertz
- $i$: imaginary unit, satisfying $i^2=-1$
- $\tilde n(f)$: Fourier transform of $n(t)$

The tilde $\tilde{}$ means "Fourier-domain version of."

The inverse transform reconstructs the time-domain function from its frequency components.

Different textbooks use different Fourier conventions. Some use angular frequency

$$
\omega = 2\pi f
$$

instead of $f$, and some distribute factors of $2\pi$ differently between the forward and inverse transforms.

The physical content is the same; the formulas differ by convention.

---

## 4. Positive and Negative Frequencies

The Fourier transform of a real-valued time signal contains both positive and negative frequencies.

For a real-valued function $n(t)$,

$$
\tilde n(-f)
=
\overline{\tilde n(f)},
$$

where the overbar denotes the **complex conjugate**.

If

$$
z=a+ib,
$$

then

$$
\overline z=a-ib.
$$

Therefore,

$$
|\tilde n(-f)|^2
=
|\tilde n(f)|^2.
$$

So the Fourier **power** at negative frequency mirrors the power at positive frequency.

The negative-frequency side does not contain new power information for a real-valued signal.

---

## 5. From Fourier Amplitudes to Power

The Fourier coefficient $\tilde n(f)$ is generally complex.

Its magnitude squared is

$$
|\tilde n(f)|^2.
$$

If

$$
\tilde n(f)=a+ib,
$$

then

$$
|\tilde n(f)|^2
=
a^2+b^2.
$$

This is always a nonnegative real number.

The squared magnitude tells us how much squared amplitude is associated with that frequency.

Very schematically,

$$
n(t)
\longrightarrow
\tilde n(f)
\longrightarrow
|\tilde n(f)|^2.
$$

This is the basic object from which a power spectrum is built.

---

## 6. Power Spectral Density (PSD)

The **power spectral density** tells us how the variance of a random process is distributed across frequency.

For detector noise, write

$$
S_n(f)
$$

for the noise PSD.

A useful interpretation is

$$
\boxed{
S_n(f)
=
\text{noise variance per unit frequency near } f.
}
$$

The PSD is therefore a **variance density over frequency**.

This is analogous to a probability density:

$$
P(a<X<b)
=
\int_a^b p(x)\,dx.
$$

Likewise, for a PSD,

$$
\operatorname{Var}
\left(
\text{noise in frequency band }[f_1,f_2]
\right)
=
\int_{f_1}^{f_2}
S_n(f)\,df.
$$

For a one-sided PSD,

$$
\boxed{
\operatorname{Var}(n)
=
\int_0^\infty S_n(f)\,df.
}
$$

So:

$$
\boxed{
\text{area under the PSD}
=
\text{noise variance}.
}
$$

That is the core fact to remember.

### What is the "density" a density of?

The density is of the **variance**, not of the raw noise values.

Analogy:

- mass density = mass per unit space
- probability density = probability per unit variable
- PSD = variance or power per unit frequency

---

## 7. Why PSD Has Units of 1/Hz

LIGO strain is

$$
h(t)=\frac{\Delta L}{L},
$$

so it is dimensionless.

Therefore squared strain is also dimensionless.

The PSD is squared strain per hertz:

$$
[S_n(f)]
=
\frac{\text{strain}^2}{\mathrm{Hz}}.
$$

Because strain is dimensionless,

$$
\boxed{
[S_n(f)] = \frac{1}{\mathrm{Hz}}.
}
$$

Since

$$
1\ \mathrm{Hz}=1/\mathrm{s},
$$

it follows dimensionally that

$$
\frac{1}{\mathrm{Hz}}=\mathrm{s}.
$$

But it is usually clearer to retain the unit $1/\mathrm{Hz}$, because its meaning is "per unit frequency."

### Why this is consistent with integration

Since

$$
\operatorname{Var}(n)
=
\int S_n(f)\,df,
$$

the units are

$$
\frac{1}{\mathrm{Hz}}
\times
\mathrm{Hz}
=
1,
$$

which is dimensionless squared strain.

---

## 8. One-Sided vs Two-Sided PSDs

A **two-sided PSD** includes both positive and negative frequencies:

$$
-\infty < f < \infty.
$$

Its total variance is

$$
\operatorname{Var}(n)
=
\int_{-\infty}^{\infty}
S_{n,\text{two-sided}}(f)\,df.
$$

A **one-sided PSD** keeps only nonnegative frequencies:

$$
f\ge 0.
$$

Its total variance is

$$
\operatorname{Var}(n)
=
\int_0^\infty
S_{n,\text{one-sided}}(f)\,df.
$$

For a real-valued signal, the positive and negative frequency powers are equal.

Therefore, away from special points such as $f=0$,

$$
\boxed{
S_{n,\text{one-sided}}(f)
=
2\,S_{n,\text{two-sided}}(f).
}
$$

The factor of 2 simply preserves the same total area after discarding the negative-frequency half.

LIGO commonly uses **one-sided PSDs**.

---

## 9. Amplitude Spectral Density (ASD)

The **amplitude spectral density** is the square root of the PSD:

$$
\boxed{
\operatorname{ASD}(f)
=
\sqrt{S_n(f)}.
}
$$

If the PSD has units

$$
1/\mathrm{Hz},
$$

then the ASD has units

$$
\boxed{
1/\sqrt{\mathrm{Hz}}.
}
$$

For LIGO, the ASD is often displayed because its vertical scale is closer to an amplitude scale than the squared-amplitude PSD.

Example:

$$
S_n(f)=4\times10^{-46}/\mathrm{Hz}
$$

implies

$$
\operatorname{ASD}(f)
=
2\times10^{-23}/\sqrt{\mathrm{Hz}}.
$$

---

## 10. How a PSD Is Estimated in Practice

A stripped-down conceptual derivation begins with a finite noise sample $n(t)$.

Take the Fourier transform:

$$
n(t)
\longrightarrow
\tilde n(f).
$$

Then square its magnitude:

$$
\tilde n(f)
\longrightarrow
|\tilde n(f)|^2.
$$

For an observation of duration $T$, a two-sided PSD is schematically

$$
\boxed{
S_n(f)
\propto
\frac{|\tilde n(f)|^2}{T}.
}
$$

The exact constants depend on Fourier-transform convention.

For a one-sided PSD, positive-frequency values are approximately doubled:

$$
S_{n,\text{one-sided}}(f)
\propto
\frac{2}{T}
|\tilde n(f)|^2,
\qquad f>0.
$$

### Why divide by $T$?

The Fourier coefficient scale depends on the observation duration.

Dividing by $T$ gives a quantity whose area over frequency corresponds to average squared amplitude, i.e. variance for zero-mean noise.

This is connected to **Parseval's theorem**.

Very schematically,

$$
\int |n(t)|^2\,dt
=
\int |\tilde n(f)|^2\,df.
$$

Divide by the observation duration $T$:

$$
\frac{1}{T}
\int |n(t)|^2\,dt
=
\int
\frac{|\tilde n(f)|^2}{T}
\,df.
$$

For zero-mean noise, the left-hand side is approximately the variance.

Thus the normalized squared Fourier amplitudes naturally produce a density whose integral is the variance.

### Practical PSD estimation

Real noise is random, so a single periodogram is jagged.

In practice, one usually:

1. divides the data into segments,
2. windows each segment,
3. computes Fourier transforms,
4. squares magnitudes,
5. normalizes,
6. averages across segments.

Welch's method is a common example.

---

## 11. LIGO's Frequency-Dependent Noise

LIGO noise is **not** a monotonic function of frequency.

Very roughly:

- low frequencies: seismic, suspension, and gravity-gradient noise are important
- middle frequencies: detector sensitivity is best
- high frequencies: photon shot noise becomes increasingly important

The result is a broad "bucket"-shaped sensitivity curve.

There are also narrow spectral lines from effects such as:

- mains power
- calibration tones
- mechanical resonances
- suspension resonances
- instrumental couplings

Therefore,

$$
\boxed{
\text{LIGO has a frequency-dependent noise floor.}
}
$$

The same raw Fourier amplitude can have very different significance at two different frequencies.

---

## 12. Whitening

Whitening rescales Fourier components according to the expected noise level at each frequency.

Suppose the measured strain has Fourier transform

$$
\tilde h(f).
$$

A whitened frequency-domain signal is schematically

$$
\boxed{
\tilde h_{\text{white}}(f)
\propto
\frac{\tilde h(f)}
{\sqrt{S_n(f)}}.
}
$$

Since

$$
\sqrt{S_n(f)}
$$

is the amplitude spectral density, whitening divides each Fourier component by the typical noise amplitude at that frequency.

So whitening converts something like

> "How large is this Fourier component?"

into something closer to

> "How large is this Fourier component relative to the expected noise at this frequency?"

---

## 13. Why Whitening Helps

Suppose two frequencies have the same raw Fourier amplitude.

That does **not** mean they are equally interesting.

If 30 Hz is usually extremely noisy and 100 Hz is usually quiet, then the same observed amplitude may be ordinary at 30 Hz but unusual at 100 Hz.

Whitening compensates for this.

After whitening, the background noise is approximately flattened across frequency.

That is why the term **white** is used: white noise has approximately equal power per unit frequency over the band of interest.

A useful shorthand is

$$
\boxed{
\text{whitening}
=
\text{noise weighting}.
}
$$

---

## 14. Time-Frequency Analysis

An ordinary Fourier transform tells us what frequencies are present but not when they occur.

For a chirping gravitational-wave signal, that timing information is crucial.

A compact binary inspiral may sweep through frequencies like

$$
35\ \mathrm{Hz}
\rightarrow
50\ \mathrm{Hz}
\rightarrow
100\ \mathrm{Hz}
\rightarrow
200\ \mathrm{Hz}.
$$

An ordinary Fourier transform reports that these frequencies were present.

It does not directly show that the frequency increased with time.

For that, we need a time-frequency representation.

---

## 15. Short-Time Fourier Transform

The **short-time Fourier transform**, or STFT, applies Fourier analysis to short overlapping windows of the time series.

Conceptually:

1. take a short segment,
2. Fourier transform it,
3. move the window,
4. repeat.

This produces a representation with both time and frequency axes.

The central tradeoff is:

- longer windows -> better frequency resolution, worse time resolution
- shorter windows -> better time resolution, worse frequency resolution

This is a manifestation of the time-frequency uncertainty tradeoff.

---

## 16. Q-Transform

The Q-transform is another time-frequency decomposition.

Instead of analyzing all frequencies with one fixed window size, it uses localized oscillatory wave packets with different time and frequency extents.

A common parameter is

$$
\boxed{
Q=\frac{f}{\Delta f},
}
$$

where:

- $f$: central frequency
- $\Delta f$: characteristic frequency bandwidth

Large $Q$:

- narrow frequency bandwidth
- better frequency resolution
- poorer time resolution

Small $Q$:

- wider frequency bandwidth
- poorer frequency resolution
- better time resolution

The Q-transform effectively asks:

> Is there unusually large signal energy near this time, frequency, and timescale?

This is well suited to transient signals such as gravitational-wave chirps.

---

## 17. Whitening vs Q-Transform

These solve different problems.

### Whitening

Purpose:

$$
\boxed{
\text{compensate for frequency-dependent detector noise}
}
$$

Operation:

$$
\tilde h(f)
\longrightarrow
\frac{\tilde h(f)}
{\sqrt{S_n(f)}}.
$$

### Q-transform

Purpose:

$$
\boxed{
\text{show where signal energy occurs in time-frequency space}
}
$$

Operation:

$$
h(t)
\longrightarrow
Q(t,f).
$$

A useful mnemonic:

$$
\boxed{
\text{whitening = noise weighting}
}
$$

$$
\boxed{
\text{Q-transform = time-frequency decomposition}
}
$$

Whitening is often performed before the Q-transform because it makes unusual transient energy easier to distinguish from the frequency-dependent background.

---

## 18. LIGO Processing Pipeline at a Glance

A simplified conceptual pipeline is:

$$
h(t)
=
s(t)+n(t)
$$

$$
\downarrow
$$

Fourier transform

$$
\tilde h(f)
$$

$$
\downarrow
$$

Estimate detector noise PSD

$$
S_n(f)
$$

$$
\downarrow
$$

Whiten

$$
\tilde h_{\text{white}}(f)
\propto
\frac{\tilde h(f)}
{\sqrt{S_n(f)}}
$$

$$
\downarrow
$$

Return to the time domain if needed

$$
h_{\text{white}}(t)
$$

$$
\downarrow
$$

Q-transform or another time-frequency transform

$$
Q(t,f)
$$

$$
\downarrow
$$

Visualize transient structures such as the gravitational-wave chirp.

---

## 19. Notation Glossary

### Time-domain quantities

$$
t
$$

Time, usually in seconds.

$$
h(t)
$$

Measured strain time series.

$$
s(t)
$$

Astrophysical signal component.

$$
n(t)
$$

Noise component.

### Statistical notation

$$
E[X]
$$

Expected value of random variable $X$.

$$
\operatorname{Var}(X)
$$

Variance of $X$.

### Frequency-domain notation

$$
f
$$

Ordinary frequency in hertz.

$$
\omega
$$

Angular frequency, defined by

$$
\omega=2\pi f.
$$

$$
\tilde n(f)
$$

Fourier transform of $n(t)$.

The tilde indicates a frequency-domain representation.

$$
\overline z
$$

Complex conjugate of $z$.

$$
|z|
$$

Magnitude of complex number $z$.

$$
|z|^2
$$

Squared magnitude of $z$.

### Spectral quantities

$$
S_n(f)
$$

Noise power spectral density.

$$
\sqrt{S_n(f)}
$$

Noise amplitude spectral density.

$$
df
$$

Infinitesimal frequency interval used in integration.

$$
\Delta f
$$

Finite frequency bandwidth.

### Q-transform

$$
Q
$$

Dimensionless ratio

$$
Q=\frac{f}{\Delta f}.
$$

---

## 20. Do Not Confuse These

### Noise vs noise variance

$$
n(t)
$$

is the random process.

$$
\operatorname{Var}(n)
$$

is a numerical property describing its fluctuation size.

---

### Fourier transform vs PSD

$$
\tilde n(f)
$$

is the Fourier transform.

$$
S_n(f)
$$

is based on normalized squared Fourier amplitudes and describes variance per unit frequency.

---

### PSD vs ASD

$$
\text{PSD}=S_n(f)
$$

has units

$$
1/\mathrm{Hz}.
$$

$$
\text{ASD}=\sqrt{S_n(f)}
$$

has units

$$
1/\sqrt{\mathrm{Hz}}.
$$

---

### One-sided vs two-sided PSD

Two-sided:

$$
-\infty<f<\infty.
$$

One-sided:

$$
f\ge 0.
$$

The one-sided version doubles the positive-frequency power so the total integrated variance stays the same.

---

### Whitening vs Q-transform

Whitening adjusts for the detector's frequency-dependent noise.

The Q-transform localizes energy in both time and frequency.

---

### Frequency $f$ vs angular frequency $\omega$

$$
\omega=2\pi f.
$$

The two are related but use different units:

- $f$: hertz
- $\omega$: radians per second

---

## 21. Exercises

These exercises are meant to be a gentle first pass, not a test of endurance.
For each one, write down what you expect **before** calculating or running code.
That prediction is often the most useful part of the exercise.

For the Python exercises, the core tools are:

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal
```

The final exercise optionally uses GWpy, as in the notebooks in
`notebooks/gwosc_tutorials/`.

### A. Conceptual checks

Answer each question in one to three sentences. Try not to use equations unless
an equation makes the answer clearer.

#### A1. Zero-mean noise

A detector's noise has zero mean. Does that mean the detector is noiseless?
What quantity would you examine to describe the typical size of its
fluctuations?

#### A2. Reading a PSD

Detector A has a larger PSD than detector B near 100 Hz. What does that tell you
about the two detectors near 100 Hz? Does it necessarily tell you which detector
is quieter at 30 Hz?

#### A3. Area versus height

Two narrow frequency bands have the same width. The PSD in the first band is
three times as high as the PSD in the second. How do their contributions to the
total variance compare? What changes if the first band is only half as wide?

#### A4. PSD versus ASD

You see a LIGO sensitivity plot with vertical units of
$1/\sqrt{\mathrm{Hz}}$. Is it showing a PSD or an ASD? How would you obtain the
other quantity?

#### A5. One-sided spectra

Why can a one-sided PSD omit negative frequencies without losing power
information for a real-valued time series? Why must most positive-frequency
values be doubled?

#### A6. Same amplitude, different significance

A Fourier component has the same raw amplitude at 30 Hz and 100 Hz, but the
noise ASD is ten times larger at 30 Hz. At which frequency is the component more
significant relative to the expected noise? What operation makes this
comparison more direct?

#### A7. Fourier transform versus time-frequency transform

Two signals contain exactly the same set of frequencies. In one signal the
frequency rises with time; in the other it falls with time. Can an ordinary
power spectrum reliably distinguish them? What kind of representation can?

#### A8. Choosing a window or $Q$

For each goal below, say whether you would prefer finer time resolution or finer
frequency resolution:

1. determining the arrival time of a very short glitch,
2. separating two long-lived tones at 100 Hz and 101 Hz,
3. following a rapidly changing chirp.

Then state whether each goal generally suggests a shorter or longer STFT window.
For a Q-transform tile at fixed central frequency, relate the same choices to
smaller or larger $Q$.

### B. Whiteboard calculations and short proofs

Show your work and keep track of units.

#### B1. Mean, variance, and RMS

Consider the four equally likely noise values

$$
-2,\quad -1,\quad 1,\quad 2.
$$

Compute:

1. the mean,
2. $E[n^2]$,
3. the variance,
4. the root-mean-square value $\sqrt{E[n^2]}$.

Explain why the RMS and standard deviation agree here.

#### B2. Integrating a flat PSD

A one-sided PSD is constant from 20 Hz to 120 Hz:

$$
S_n(f)=4\times10^{-46}/\mathrm{Hz},
$$

and zero outside that band. Compute the variance and the RMS strain. Sketch the
PSD and identify geometrically where the variance appears in your sketch.

#### B3. Converting PSD to ASD

At 100 Hz, suppose

$$
S_n(100\ \mathrm{Hz})=9\times10^{-46}/\mathrm{Hz}.
$$

Compute the ASD, including its units. If the PSD becomes four times larger,
by what factor does the ASD change?

#### B4. One-sided and two-sided normalization

A two-sided PSD is

$$
S_2(f)=S_0
\quad\text{for}\quad -B<f<B,
$$

and zero elsewhere.

1. Integrate $S_2(f)$ over all frequencies.
2. Write the corresponding one-sided PSD $S_1(f)$ for $0<f<B$.
3. Integrate $S_1(f)$ and verify that the variance is unchanged.

Do not worry about the isolated point $f=0$; a single point has zero width in
the integral.

#### B5. Complex Fourier amplitudes

For

$$
z_1=3+4i,
\qquad
z_2=-1+2i,
$$

compute $\overline z$, $|z|$, and $|z|^2$ for each. Which of these quantities
is used to build a power spectrum?

#### B6. Prove conjugate symmetry

For a real discrete time series $x_0,\ldots,x_{N-1}$, define

$$
X_k
=
\sum_{j=0}^{N-1}
x_j e^{-2\pi i jk/N}.
$$

Starting from the definition of $X_{N-k}$, show that

$$
X_{N-k}=\overline{X_k}.
$$

Hint: use $e^{-2\pi i j}=1$ and the fact that $x_j$ is real. Then explain in
one sentence why the positive- and negative-frequency powers are equal.

#### B7. Power in a sinusoid

Let

$$
x(t)=A\cos(2\pi f_0t).
$$

Use

$$
\cos^2\theta
=
\frac{1+\cos(2\theta)}{2}
$$

to show that the long-time average of $x(t)^2$ is $A^2/2$. If a one-sided PSD
contains all of this sinusoid's power at $f_0$, what must the total area of that
spectral peak be?

#### B8. Why whitening flattens noise

Suppose, schematically, that the expected frequency-domain noise power obeys

$$
E\left[|\tilde n(f)|^2\right]=C S_n(f),
$$

where $C$ is a constant determined by the Fourier convention. Define

$$
\tilde n_{\mathrm{white}}(f)
=
\frac{\tilde n(f)}{\sqrt{S_n(f)}}.
$$

Compute $E[|\tilde n_{\mathrm{white}}(f)|^2]$. Why does the result justify the
word "white"? State one practical reason that real whitened data will not be
perfectly flat.

### C. Python exercises

Use synthetic data first. It lets you know the correct answer in advance and
keeps data access and detector complications out of the way.

#### C1. Estimate simple noise statistics

Generate Gaussian noise with known mean and standard deviation:

```python
rng = np.random.default_rng(1234)
x = rng.normal(loc=0.0, scale=2.0, size=1_000)
```

Without using `np.var` or `np.std`, compute the sample mean, mean square,
variance, and standard deviation from their definitions. Repeat for sample sizes
$10$, $100$, $1{,}000$, and $100{,}000$.

Self-check: the estimated mean should tend toward 0, and the variance should tend
toward 4 as the sample size grows. Do not expect the error to decrease
monotonically on every run.

#### C2. Find tones with an FFT

Sample this signal for 2 seconds at 1024 samples per second:

$$
x(t)
=
\cos(2\pi 50t)
+
0.5\cos(2\pi 120t).
$$

1. Construct the time array with `np.arange`.
2. Compute `np.fft.rfft(x)` and its frequencies with `np.fft.rfftfreq`.
3. Plot the magnitude and identify the two largest non-DC peaks.
4. Predict and then confirm the frequency-bin spacing.
5. Compute the full `np.fft.fft(x)` and numerically check conjugate symmetry.

Self-check: the peaks should be at 50 Hz and 120 Hz, and the bin spacing should
be $1/T=0.5$ Hz.

#### C3. Implement the DFT directly

For a small input with $N=32$, implement

$$
X_k
=
\sum_{j=0}^{N-1}
x_j e^{-2\pi i jk/N}
$$

using either two loops or a matrix of complex exponentials. Do not call an FFT
inside your implementation. Compare the result with `np.fft.fft` using
`np.allclose`.

Then time your implementation and NumPy's FFT for a few increasing values of
$N$. Keep the sizes modest if you use loops. Describe the scaling you observe;
you do not need to prove the FFT's complexity.

#### C4. Build a one-sided periodogram from scratch

Generate zero-mean noise, optionally with one sinusoid added. For $N$ samples at
sampling frequency $f_s$, use the following normalization:

$$
P_k
=
\frac{|X_k|^2}{f_sN},
$$

where `X = np.fft.rfft(x)`. Double every bin except DC and, when $N$ is even,
the Nyquist bin.

1. Compute the frequency spacing $\Delta f=f_s/N$.
2. Verify numerically that

   $$
   \sum_k P_k\,\Delta f
   \approx
   \frac{1}{N}\sum_j x_j^2.
   $$

3. Compare your result with
   `signal.periodogram(x, fs=fs, scaling="density", detrend=False)`.

This equality is the discrete version of "area under the PSD equals power."

#### C5. See why Welch averaging helps

Create several independent realizations of Gaussian noise and compute a raw
periodogram for each. Plot them together. Then estimate each PSD with

```python
f, pxx = signal.welch(x, fs=fs, nperseg=1024)
```

Compare the visual roughness of the raw periodograms and Welch estimates.
Experiment with `nperseg=256`, `1024`, and `4096`.

Questions:

1. Which choice gives the finest frequency spacing?
2. Which choice provides more segments to average for a fixed data length?
3. What tradeoff do you see between frequency resolution and estimate
   variability?

#### C6. Whiten synthetic colored noise

Make colored noise by shaping white noise in the frequency domain:

1. generate white noise $w(t)$,
2. compute `W = np.fft.rfft(w)`,
3. multiply `W` by a smooth, positive frequency-dependent curve $A(f)$,
4. transform back with `np.fft.irfft`.

For example, choose a curve that is large at low frequencies and approximately
flat at high frequencies. Estimate the colored noise PSD with Welch's method.
Interpolate that PSD onto the `rfft` frequencies, then form

```python
X_white = X / np.sqrt(psd_interpolated)
x_white = np.fft.irfft(X_white, n=len(x))
```

Plot the Welch PSD before and after whitening on log-log axes. Ignore the
overall vertical normalization and focus on the shape.

Questions:

1. Is the whitened PSD flatter?
2. What happens where the PSD estimate is very small?
3. Why might you impose a minimum PSD value before dividing?

#### C7. Compare time-frequency resolutions

Use `signal.chirp` to make a signal whose frequency rises from 30 Hz to 300 Hz.
Add Gaussian noise, then make two spectrograms using `signal.spectrogram`:

- one with a short `nperseg`,
- one with a long `nperseg`.

Before plotting, predict which spectrogram will localize rapid changes in time
and which will resolve frequency more sharply. Confirm your prediction from the
plots. This is an STFT exercise, but it develops the same time-frequency
intuition needed for the Q-transform.

#### C8. Optional: use GWpy on gravitational-wave data

After loading a GWpy `TimeSeries` named `strain`, as demonstrated in
`notebooks/gwosc_tutorials/index.ipynb`, try:

```python
psd = strain.psd(fftlength=4)
white = strain.whiten()
hq = strain.q_transform()
```

1. Plot `psd` and `psd ** 0.5`. Label their units as PSD and ASD.
2. Compare a short interval of `strain` and `white`.
3. Plot `hq` and identify any localized time-frequency structure.
4. Repeat the PSD or whitening with a different estimation duration. Describe
   what changes and what stays qualitatively the same.

This exercise uses library implementations. The earlier exercises explain the
main ideas those methods package together.

### Numerical checkpoints

Use these only after attempting the corresponding problems:

- B1: mean $=0$, variance $=E[n^2]=2.5$, RMS $=\sqrt{2.5}$.
- B2: variance $=4\times10^{-44}$ and RMS strain $=2\times10^{-22}$.
- B3: ASD $=3\times10^{-23}/\sqrt{\mathrm{Hz}}$; quadrupling PSD doubles ASD.
- B4: both PSD conventions integrate to $2BS_0$.
- B5: $|z_1|^2=25$ and $|z_2|^2=5$.
- B7: the one-sided spectral peak has total area $A^2/2$.
- B8: the expected whitened power is the frequency-independent constant $C$.

---

# Minimal Memory Version

If only a few facts survive, keep these:

$$
\boxed{
h(t)=s(t)+n(t)
}
$$

$$
\boxed{
\operatorname{Var}(n)=E[n^2]
\quad\text{for zero-mean noise}
}
$$

$$
\boxed{
S_n(f)
=
\text{variance density over frequency}
}
$$

$$
\boxed{
\operatorname{Var}(n)
=
\int S_n(f)\,df
}
$$

$$
\boxed{
\operatorname{ASD}(f)
=
\sqrt{S_n(f)}
}
$$

$$
\boxed{
\text{whitening}
\approx
\frac{\text{Fourier amplitude}}
{\text{expected noise amplitude}}
}
$$

$$
\boxed{
\text{Q-transform}
=
\text{time-frequency decomposition}
}
$$

$$
\boxed{
\text{whitening and Q-transform solve different problems}
}
$$
