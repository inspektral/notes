# Probability

## Probability distribuition density

From what a probability distribution density is just a function that maps the possible outcome to its probability. It makes sense that the total probability (the integral I guess) is 1. In the case of a continuous domain it feels a bit weird, I guess that in that case you always integrate and the probability is actually the area(?). 

## Uniform distribution

Well this is just flat, every result is equally probable.

## Normal distribution

$$\mathcal{N}(0,1)$$

## Entropy

## Perplexity

## Change of variable

I think that for this, thinking of a distribution as a cloud of points and we're projecting those clouds in new spaces with a function

- $p(x)$ probability density function (PDF)
- $f \rarr y = f(x)$: f is an inversible and differentiable function
- $p_x$: original PDF
- $p_y$: new, target PDF

Alright so i think this kinda makes sense now let's get to the point. Since we're just moving points around if we take a portion, and portion limits, in our $p_x$ and we run it thru our function the portion will contain the same points, but the portion could become bigger or smaller. A bit like having dots on a latex sheet and draw a circle around them, by stretching the latex the points can go far from each other in different directions but they will always stay inside the circle.
Let's try to formalize it:

$$p_x[x, x+dx] = p_y[f(x), f(x+dx)]$$

Left side:
- $\int_x^{x+dx}p_x(s)ds$: this are tiny rectangles $p_x(x)$ and tall, $dx$ wide:
- $p_x(x)dx$

Right side:
- $\int_{f(x)}^{f(x+dx)}p_y(s)ds$: this is a rectangle $p_y(f(x))$ tall and $f(x+dx)-f(x)$ wide:
- $p_y((f(x))(f(x+dx)-f(x))$: but $f(x+dx)-f(x)$ is the derivative($f'$) of $f(x)$:
- $p_y(f(x))f'(x)dx$

Join them back:
- $p_x(x)dx = p_y((f(x))f'(x)dx$
- $p_x(x) = p_y((f(x))(f'(x))$: if f is decreasing this would be negative but and we should reverse the integral but we just take the absolute value and everyone is happy
- $p_x(x) = p_y((f(x))|f'(x)|$: now we solve for y:
- $p_x(f^{-1}(y)) = p_y(y) |f'(f^{-1}(y))|$: here's theres the inverse function theorem: $y=f(x) \rarr (f^{-1})'(y)=\frac1{f'(x)}=\frac1{f'(f^{-1}(y))}$, basically the derivative of the inverse is the reciprocal of the derivative, it kinda makes sense.
- $p_x(x) = p_y(y)|\frac1{(f^{-1})'(y)}|$

and we usually want know $p_x$ and $f$ and we want to find $p_y$ so:
$$p_y(y) = p_x(f^{-1}(y))|(f^{-1})'(y)|$$

What does this mean?

Practically if we have p_x and f we can do the mapping between the two distribuitions,the first factor checks the density in the original distribution $p_x$ and the second factor adjusts for the stretching and compression happening in the area.

Example:

$$x \sim \mathcal{N}(0,1), y = 3x$$
$$f^{-1}(y) = \frac1 3 y, (f^{-1})' = \frac 1 3 $$
$$ p_y(y)= p_x(\frac 1 3 y) \frac 1 3$$
