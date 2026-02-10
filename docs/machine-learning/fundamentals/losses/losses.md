
# Losses

### What are we doing?

We are trying to build an algorithm, $f$, that given a suggestion will try to make  a guess, and we want to measure how close it got to the truth in order to make the algorithm better.

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

#### Metaphor

Let's an idea of a model could be an algorithm that given a picture from google street view guesses the coordinates where this picture was taken. Now of course an image is huge, it's like a million of pixels, and on the other side coordinates are just two numbers (and two letters).
So our x will be a matrix, a 2D array, a table that represent the pixels and the model is gonna do something to transform this in somthing that has the same shape as the truth, so 2 numbers and 2 letters. The following task is the focus of this page and it is to measure how far from the truth the model is, how bad its guesses are

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
