---
layout: default
title: Random
nav_order: 1
has_children: true
---

# Random notes

## CTC Loss

Connectionist Temporal Classification it is a loss used for RNNs, seems to be the go to for phonemes classification.
Basically ignores repetition and blanks in sequences, since in audio there are much more frames than phonemes.

`C C A - - T -` → `C A T` 
