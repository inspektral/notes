# ParaGSE

PARAllel Generative Speech Enhancement

## Links 

- paper: <https://arxiv.org/pdf/2602.01793>
- code: <https://github.com/fliu215/ParaGSE>
- demo: <https://anonymity225.github.io/ParaGSE/>

## Codec

They use this interesting codec: G-MDCTCodec (Group Modified Cosine Transform Codec) based on GVQ (Group Vector Quantization). This codec outputs multiple independent vectors that then get quantized

#### MDCTCodec

Without the grouping the codec works like this:

```mermaid
flowchart LR
  I[audio] --> |48khz| S[MDCT]
  S --> |1.2khz|L[Latent]
  L --> Q[Quantizer]
  Q <-->|6kbps| T[tokens]
  Q --> D[Decoder]
  D --> iS[MDCT]
  iS --> o[audio]
```

examples: <https://pb20000090.github.io/MDCTCodecSLT2024/>

Honestly it doesn't sound that good...

#### G-MDCTCodec

```mermaid
flowchart LR
  A[audio] --> E[Encoder]
  E --> V1[vector 1] --> Q1[quantizer] --> T1[Token 1]
  E --> V2[vector 2] --> Q2[quantizer] --> T2[Token 2]
  E --> V3[vector 3] --> Q3[quantizer] --> T3[Token 3]
  E --> V4[vector 4] --> Q4[quantizer] --> T4[Token 4]
```
