# ML-random notes

I write stuff here, ideally they are to be moved once they make more sense in the bigger context

## CTC Loss

Connectionist Temporal Classification it is a loss used for RNNs, seems to be the go to for phonemes classification.
Basically ignores repetition and blanks in sequences, since in audio there are much more frames than phonemes.

`C C A - - T -` → `C A T` 

## Multinomial sampling

From logits we can do probabilities (softmax), at that point we can sample from thisdistribution with multinomial sampling
instead of argmax. This from what I understand works well but it is not differentiable so it is to be used with care

## Gumbel softmax

We can do a crazy trick that makes sampling differentiable, basically we to the apply some noise and then we do the softmax, and this is like sampling but differentiable. which kinda makes sense:
`softmax{1 + gumbel(0,1)}`

The softmax is soft and gumbel is similar to gaussian noise, with the forward pass we get the index, with the backward pass we get the gradient, this makes it fully differentiable and very important to properly explore the gradient

Interesting video to dig deeper: [https://www.youtube.com/watch?v=c65nlNgfFrE]()

## Autocorrelation Issues

The data has similar patterns in different segments. That's obvious, especially for audio but it is good to be clear.

## Span masking

Since in audio there is a lot of autocorrelation if we just mask a frame it's very easi to predict, just copy and paste the previous one. That works most of the time but it is clearly useless, to solve this what is done it is called span masking, which means that we mask subsequent frames all together, so that actual changes and information is masked, if a model is successful on this it means that it is learning quite a lot about how to fill in the blanks.


