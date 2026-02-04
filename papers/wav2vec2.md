---
layout: default
title: wav2vec 2.0
parent: Papers
---

# wav2vec 2.0

Basically audio in, quantized tokens out

## Architecture

There's a CNN and then a transformer

### Quantizer

it works with codebooks, there are two codebooks, each of them 320 long, a vector is chosen
from each codebook and then they are conccatenated (adn then a linear projection but whatever)

## Training

Everything is trained together:

- CNN takes audio (20ms) and outputs non-quantized vector(z)
- z get quantized with a quantizer module (q)
- z gets relative positional encoding with a small CNN (? i think it's here)
- z get masked (49%) and go into the transformer
- transformer is trained to guess the quantized missing token
- Loss 1: contrastive so positive target and distractors
- Loss 2: diversity, entropy of the probability distribution of the codebooks
- fine tuning step with labeled data and ctc loss

## Result

We throw away the quantizer and we just use the CNN and transformer,
this leads to get context aware tokens and works very well 
