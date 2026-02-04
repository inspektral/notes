# Short Time Objective Intelligibility

STOI (2011) is an intrusive metric, meaning that it requires a clean reference and it measures degradation.

`torchmetrics.audio.stoi`[Short-Time Objective Intelligibility (STOI) — PyTorch-Metrics 1.8.2 documentation](https://lightning.ai/docs/torchmetrics/stable/audio/short_time_objective_intelligibility.html)

`pystoy` [GitHub - mpariente/pystoi: Python implementation of the Short Term Objective Intelligibility measure](https://github.com/mpariente/pystoi)

## Steps

- downsampling to 10khz

- splitting into frames (256 samples), 50% overlap, hanning window

- FFT

- Grouping into 15 1/3 octave bands

- extract band energy

- create envelope matrix, looking at the 30 previous frames, at this point we have a matrix E (15 \times 30), with an envelope for each band

- normalization, each row is normalized so the total energy in a specific row is the same in the clean and processed signal

- clipping

- per-row correlation: Pearson correlation is calculated between each row clean and processed

- frame correlation: the 15 correlations are averaged and we get the STOI for that fragment

- global correlation: the entire process is repeated sliing by one frame and then the global average is calculated giving us the final STOI s \in [0,1], with a good value being s \ge 0.85 and a bad one being s \le 0.60.

## Pros and Cons

**Pros**:

- Perceptually informed
- non differentiable, due to clipping, normalization and also hard filters create a very jagged gradient (it can be fixed tho, with soft clipping and epsilon padding)

**Cons**:

- Contrastive, we need a clean signal

- Disregard phase, so something like a vocoder still works while sounding completely unnatural, but this is by design, we're measuring intelligibility, not quality

- Normalization step can be problematic if a model attenuates or amplifies specific bands to the point of unintelligibility

## Local STOI and STOI-gram

## STOI-Net

## extended STOI

## Usage
