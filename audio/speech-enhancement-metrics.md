---
layout: default
title: Losses
parent: audio 
---

# Metrics and Losses

### What are we doing?

We are trying to build an algorithm, $f$, that given a suggestion will try to make  a guess, and we want to measure how close it got to the truth in order to make the algorithm better.

### What? Why?

### Mmmh, okay go ahead...

We have four simple blocks:

- $y$: the __truth__, or that we don't know and we're trying to guess

- $x$: our input

- $f$: our algorithm, or __model__, to get close from to the truth with our input

- $\theta$: additional parameters for our algorithm, independent from the input

With those blocks we can build two key things:

- $\hat y = f(x, \theta)$ our guess, or __prediction__ with the available input

- $y - \hat y$: the distance between our guess and the truth, called __residual__

There is also an important thing to mention, we are being very general here and because of that we need to understand how 

### Intuition for those formulas

Stuff like $\frac{1}{N}$ and $\sum_{i=1}^N$ doesn't really matter, this is just means the average. the most another important thing is that $y_i-f(x_i)$ is just the distance from the truth, that's the input to our actual measurament. The point is that $y_i$ can be any number, and we are interested in reaching that number so we measure how far we are from it and then we calculcate our gradient. $y_i - f(x_i)$ is just placing ourself on the correct spot on the x axis, before appling the function whose slope we're interested about. 

## Mean Bias Error

Mean bias error (MBE): this is kinda of a dumb one, is just the difference sample by sample, not very useful since some are positive and some are negative and the loss can go to $0$ even if the samples are very different.

On the good side it is differentiable. 

$$
L_{MBE}=\frac{1}{N}\sum_{i=1}^N (y_i-f(x_i))
$$

## Mean Absolute Error

Mean Absolute Error (MAE), we now take the absolutte difference sample by sample, which makes much more sense but it has the problem that it is not differentiable, given that $\nexists f'(|x|)$ when $x = 0$. Besides that it is quite good to compare signals because it is equally sensitive to small and big differences. Which is not the case with squared or other.

$$
L_{MAE} = \frac{1}{N}\sum_{i=1}^N |y_i -f(x_i)|
$$

Anyway, in practice, when we match ground truth and theoretically cannot differentiate we can just pick either $1$ or $-1$ as a gradient and pretend to be off by an $\epsilon$. 

This thing is also called __L1__ (`torch.nn.L1Loss`) because it is  based on the $L_1$ norm of a vector, which is the sum of the magnitudes of its components. The distance $\sum|v-w|$ is also called __manhattan__ or __taxicab__ distance, because we're summing the absolute difference on each axis, so it's like travelling on a grid. 

Even another term is __Least Absolute Deviations (LAD)__. I honestly don't care why about this one, we already have enough different names.

## Mean Squared Error

$$
L_{MSE} = \frac{1}{N}\sum_{i=1}^N(y_i-f(x_i))^2
$$

- $L_2$ 

- `torch.nn.L2Loss`

## SNR

## SDR

## SI-SDR

## SI-SNR

## Perceptual Evaluation of Speech Quality

Also it is probably useless, all the models that I trained seems to have 0 STOI, so they're suppoesed to be perfect for this metric, but they're NOT GOOD ENOUGH.

## Short Time Objective Intelligibility

STOI (2011) is an intrusive metric, meaning that it requires a clean reference and it measures degradation.

`torchmetrics.audio.stoi`[[Short-Time Objective Intelligibility (STOI) &mdash; PyTorch-Metrics 1.8.2 documentation](https://lightning.ai/docs/torchmetrics/stable/audio/short_time_objective_intelligibility.html)]()

`pystoy` [[GitHub - mpariente/pystoi: Python implementation of the Short Term Objective Intelligibility measure](https://github.com/mpariente/pystoi)]()

``

### Steps

- downsampling to 10khz

- splitting into frames (256 samples), 50% overlap, hanning window

- FFT

- Grouping into 15 1/3 octave bands

- extract band energy

- create envelope matrix, looking at the 30 previous frames, at this point we have a matrix $E (15 \times 30)$, with an envelope for each band

- normalization, each row is normalized so the total energy in a specific row is the same in the clean and processed signal

- clipping

- per-row correlation: Pearson correlation is calculated between each row clean and processed

- frame correlation: the 15 correlations are averaged and we get the STOI for that fragment

- global correlation: the entire process is repeated sliing by one frame and then the global average is calculated giving us the final STOI $s \in [0,1]$, with a good value being $s \ge 0.85$ and a bad one being $s \le 0.60$.

### Pros and Cons

**Pros**:

- Perceptually informed
- non differentiable, due to clipping, normalization and also hard filters create a very  jagged gradient

**Cons**:

- Contrastive, we need a clean signal

- Disregard phase, so something like a vocoder still works while sounding completely unnatural, but this is by design, we're measuring intelligibility, not quality

- Normalization step can be problematic if a model attenuates or amplifies specific bands to the point of unintelligibility

### STOI-Net

### extended STOI

### Usage

## URGENT

## SIGMOS

## Speech Intelligibility Index
