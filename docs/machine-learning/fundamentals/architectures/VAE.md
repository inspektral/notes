# Variational AutoEncoder

A variational autoencoder is very similar to an autoencoder, but input vectors are projected in the latent space as multivariate gaussians:

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

Therefor they are conceptually quite different, the idea is that if we have gaussians we enforce the continuitiy of the latent space, which makes enables sampling from it.

## Training

At training time it is kinda fun, the vector generates mean and variance

- $z_\mu, z_\sigma = E(x)$
- $\sigma = e^{0.5 z_\sigma}$: we get the actual variance from the logvariance
- $z = z_\mu + \epsilon \odot \sigma$ ($\odot$ means elementwise multiplication)
- $\hat y = D(z)$

The computing of $z$ is called reparametrization trick, and it is done like this because if we were sampling from the gaussian that woulnd't be differentiable, so instead we add gaussian noise to it scaled by $\sigma$, this noise is still sampling but we're not interested in learning it, we just want to adjust the scaling of it and the centering. In this way we can sampling on a gaussian in a differentiable way.

It is quite important to understand how it works and how this enforces a continuous latent space: let's say that we have two different vectors $x_1, x_2$, we encode them and we get $(z_{1\mu}, z_{1\sigma}), (z_{2\mu}, z_{2\sigma})$. Now we sample the distributions and we get $z_1, z_2$. Since we added noise the sampled points won't land on exactly on their means, but somewhere nearby. Sometimes, for some $\epsilon_1, \epsilon_2$, they will be in the same are and that would be a point $z_{\epsilon}$ somwhere in between $z_{1\mu}, z_{2\mu}$. Now $z_\epsilon$ gets decoded to $\hat y_\epsilon$, but across many training steps that region will be visited sometimes from $x_1$ and some other times from $x_2$. Every time the reconstruction loss wil try to pull $\hat y_\epsilon$ towards the current target, and the decoder will learn to decode $z_\epsilon$ to something that is reasonable for both $x_1$ and $x_2$, a blent between the two. This is how smoothness is achieved, noise forces the decoder to handle not just the data points but also the space between them, this is the key of VAEs. 

## Probabilistic notation

Probabilistic notation is used a lot in this specifically because the latent space is a multivariate gaussian, and the fact that we're not outputting single points but distributions often gets shown with probability notation. Let's explore it a little bit:

- Probabilistic encoder: $q_{\phi}(z|x)$ it means the probability of $z$ (latent representation), given input vector $x$, this probability distribuition is a multivariate gaussian, so this is basically a fancy way to write $z_\mu, z_\sigma = E(x)$. $q_{\phi}$ means that it is an approximate ($q$ instead of $p$) distribution parametrized by $\phi$ which are the weights.
- Probabilistic decoder: $p_{\theta}(x|z)$ once z is computed (as shown above, just sampling from a multivariate gaussian) we can get the plug it into the decoder, and that means we're decoding from the latent space, generating a plausible (probable) $x$. The notation is a bit confusing here, because even if the decoder would be generating a distribution ($\mu, \sigma$) in the majority of applications we're just interest in reconstruction, which means a single vector, and the variance is ignored.

And now we have the weird one, at least IMHO, the prior: $p(z)$. This is like the shape of the latent space itself, and it is used for the loss. Since we have zero information about the data when establishing this we set it to a standard gaussian

### Losses

#### Reconstruction Loss

This is simple reconstruction loss:

$$MSE(x, x')$$

#### KL Divergence

Kullback-Leibler Divergence is similar to a distance between two distributions, but it's asymmetrical, it is actually a measurement of how much information is lost when approximating. This is the second main component of our loss. 

$$D_{KL}(Q||P) = \int q(x)\log\frac{q(x)}{p(x)}dx $$

Even if it might feel like a hack, this loss is actually a mathematical consequence of all the assumptions of the model, which follow from ELBO derivation. the idea is that we want our data points to be in a standard multivariate gaussian, and so at training time we will punish the distribution predicted by the encoder against our prior $\mathcal{N}(0,1)$, which is chosen as a reasonable assumption and allows to easily calculate $D_{KL}$. This has as a result that our predicted distribution will not collapse to a single point, will not be too flat and will be all pushed towards the $0$ of our latent space, keeping it as compact as possible.

#### Total loss

$$L_{VAE} = MSE(x, x') + D_{KL}(\mathcal{N}(z_\mu, z_\sigma)||\mathcal{N}(0,1))$$

## Inference

## References

<https://lilianweng.github.io/posts/2018-08-12-vae/>
