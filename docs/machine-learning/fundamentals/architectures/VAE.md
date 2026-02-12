# VAE

a varietional autoencoder is very similar to an autoencoder, but input vectors are projected in the latent space as multivariate gaussians:

- $n$: feature dimensions
- $l$: latent dimensions
- $x \in \mathbb{R}^n$: input vector
- $y \in \mathbb{R}^n$: truth
- $\hat y \in \mathbb{R}^n$: prediction
- $z_\mu \in \mathbb{R}^l$: latent mean vector
- $z_\sigma \in \mathbb{R}^l$: latent variance (actually logvariance so it can be negative)
- $\epsilon \in \mathbb{R}^l$: multivariate gaussian noise, one per every sample at training time
- $E$: encoder
- $D$: decoder

## Training

At training time it is kinda fun, the vector generates mean and variance

- $z_\mu, z_\sigma = E(x)$
- $\sigma = e^{0.5 z_\sigma}$ - we get the actual variance from the logvariance
- $z = z_\mu + \epsilon \odot \sigma$ - ($\odot$ means elementwise multiplication)
- $\hat y = D(z)$


### Losses

#### KL Divergence

## Inference
