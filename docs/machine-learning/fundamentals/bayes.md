# Bayes' Theorem

How new evidence changes belives, from prior to posterior.

## Formal statement

$$P(H|E) = \frac{P(E|H) P(H)}{P(E)}$$

- $H$: Hypothesis
- $E$: Evidence
- __Prior__: $P(H)$ probability of hypothesis $H$ without evidence
- __Likelihood__: $P(E|H)$ probability of evidence $E$ given hypothesis $H$
- __Posterior__: $P(H|E)$ probability of hypothesis $H$ given evidence $E$
- __Evidence__: $P(E)$ probability of evidence $E$

The idea is that probability of the hypothesis is updated with new evidence. Ideally we had the probability of hypothesis $H$, $P(H)$, and we now see evidence $E$ happening, and so we can update the probability $P(H|E)$ now that we know that $E$ has happened, knowing how probable $E$ is $P(E)$ and how likely is $E$ given $H$ $P(E|H)$.

To calculate the denominator we almost always have to sum the probability of $E$ when $H$ is true and when $H$ is not true ($\neg H$):

$$P(H|E) = \frac{P(E|H)P(H)}{P(E|H)P(H)+P(E|\neg H)P(\neg H)}$$

## Intuition with space

We start with the whole space of all possibilities, let's say $1 \times 1$, that is like $P(H) + P(\neg H) = 1$ or $P(E) + P(\neg E) = 1$. The probability of our hypothesis is a portion of this space, as is the probability of our evidence. Calculating the posterior means that now we know we are in the $E$ section of this space, and we want to know how likely it is that we are also in the $H$ portion, which is calculating how much of $E$ is also $H$.

## Inutition with sets
- $A$: all possibilities set
- $H \subseteq A$: subset of $A$ where $H$ is true
- $P(H) = \frac{|H|}{|A|}$: the prior probability of $H$ is the ratio between the size($|X|$) of $H$ and the one of $|A|$
- $E \subseteq A$: subset of $A$ where $E$ is true
- $P(E) = \frac{|E|}{|A|}$: the evidence probability of $E$ is the ratio between the size($|E|$) of $H$ and the one of $|A|$
- $P(E|H) = \frac{|E \subseteq H|}{|H|}$: the probability of evidence $E$ given $H$ (likelihood) is how much of $H$ is also $E$
- $P(H|E) = \frac{|H \subseteq E|}{|E|}$: the probability of $H$ given the evidence $E$ (posterior) is how much of $E$ is also $H$

Maybe this doesn't make much sense but to me it kinda helps thinking about sets
