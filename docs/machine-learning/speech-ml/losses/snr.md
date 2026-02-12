# SNR
 Which means Signal to Noise Ratio, and it is an intrusive (needs reference) metric for speech enhancement models and a lot of other things, it is very old and very used.
 
## Definition
- $x$: vector, clean speech recording
- $r$: vector, noise recording
- $y = x + r$: vector, noisy mixture
- $\hat x$: vector, enhanced speech from noisy mixture

$$ SNR = 10log_{10}\left(\frac{\sum x_i^2}{\sum (x_i - \hat x_i)^2}\right)$$
