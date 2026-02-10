# DAC SE 1

Speech enhancement model, based on latent transformer in the DAC latent space. Accepted at ICASSP 2026. Inference seems super slow, in fact not working. Maybe it doesn't even sound that good, but the approach might be interesting. 

tags: __se__, __LM__, __transformer__, __latent__, __dac__, __code__, __weights__, __2025__

## Links

- github repo: [https://github.com/ETH-DISCO/DAC-SE1/tree/main?tab=readme-ov-file]()
- paper: [https://arxiv.org/pdf/2510.02187]()
- weights: [https://huggingface.co/disco-eth/DAC-SE1/tree/main]()

## Concept

Just use a Language Model, IE a transformer, on dac discrete tokens, without much adaptation could work good enough.

## Testing

- inference takes forever, probably just doesn't work as of now and needs some debugging
- audio demos don't sound that great
