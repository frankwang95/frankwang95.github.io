---
layout: post
title: "The Asymptotic Error of Euler Discretization"
date: 2019-09-22
tags: ODEs Discretization Numerical Analysis
---

Suppose that we have two infinitely differentiable functions $$h: [0,1] \rightarrow \mathbb{R}$$ and $$f: \mathbb{R} \rightarrow \mathbb{R}$$ which satisfy the following initial value problem:

$$h'(t) = f(h(t)) \qquad \qquad  h(0) = h_0$$

This initial value problem admits a unique solution from the implied Lipschitz continuity of $$f$$. In today's discussion, we will derive asymptotic bounds on the error of a numerical solver used on such IVPs. We remark that while this analysis generalizes to wider domains than $$[0,1]$$, accumulating errors from Taylor approximations requires that all derivatives of $$h$$ remain bounded on the entire domain which we get for free when it is compact.

There are a variety of numerical methods for solving ODEs like the one above. One of the simplest of these methods is Euler discretization which is a forward estimating method which given an integer resolution $$N$$, estimates $$h$$ at discrete points in time $$k = 0, 1, 2, \ldots N$$ via the following recursive relation:

$$\hat h_0 = h_0 \qquad \qquad \hat h_k = \hat h_{k - 1} + \frac{1}{N} f(\hat h_{k - 1})$$

This method estimates $$\hat h_k \approx h(k/N)$$ on the basis of a first-order Taylor approximation:

$$h\left(\frac{k + 1} {N}\right) \approx h(\frac{k}{N}) + \frac{1}{N} h'\left(\frac{k}{N}\right) \approx \hat h_k + \frac{1}{N} f(\hat h_k) = \hat h_{k + 1}$$

Furthermore, use of the Taylor theorem gives us a a good method to measure the asymptotic behavior of our method's error which we write as $$\epsilon_k = \hat h_k - h(k / N)$$. Given the good-behavior and compact domain of $$h$$, we know  that the first-order Taylor series of $$h$$ centered at some point $$a$$ admits the following approximation:

$$h(a + \Delta t) = h(a) + h'(a) (x - a) + \Theta (\Delta t^2)$$

If we were to assume that the Euler estimate of the previous time step is exact, this equation directly indicates to us that the error incurred by Euler discretization at each step is of order $$O(N^{-2})$$ because we can think of $$\hat h_k$$ as the estimate for $$h(k / N)$$ given by the first-order Taylor series centered around $$({k - 1}) / N$$. This is called the local truncation error of Eulers discretization, which we will denote $$\tilde\epsilon_k$$. Our discussion here justifies the bound $$\vert\tilde\epsilon_k\vert \sim O(N^{-2})$$.

The local truncation error does not give the true error $$\epsilon_k$$ because it relies on the assumption that $$\hat h_{k-1} = h(k / N)$$ which is not true in general. We expect that these local truncation errors may accumulate over time meaning that our error will get larger with $k$. However, we can show that the accumulation of the local truncation error accumulates linearly over time so that $$\vert \epsilon_k \vert \sim O(k N^{-2})$$. This error relation exhibits the important property that our error is $$O(N^{-1})$$ when we seek to numerically compute the ODE solution for a particular point in time.

This result rests upon the following recurrsive relation:

$$\vert \epsilon_k \vert \leq \left(1 + \frac{L}{N}\right) \vert \epsilon_{k - 1} \vert + \vert \tilde \epsilon_k \vert$$

where $$L$$ is the Lipschitz constant of $$f$$. The derivation is as follows:

$$\begin{align}
\vert \epsilon_k \vert &= \left\vert \hat h_k - h\left(\frac{k}{N}\right) \right\vert \\
&\leq \left\vert \hat h_{k - 1} + \frac{1}{N} f(\hat h_{k - 1}) - h\left(\frac{k - 1} {N}\right) - \frac{1}{N} f\left(h\left(\frac{k - 1}{N}\right)\right) - \tilde \epsilon_k \right\vert \\
&\leq \frac{1}{N} \left\vert f(\hat h_{k - 1}) - f\left(h\left(\frac{k - 1}{N}\right)\right) \right\vert + \vert \epsilon_{k - 1} \vert + \vert \tilde \epsilon_k \vert \\
&\leq \frac{L}{N} \vert \epsilon_{k - 1} \vert + \vert \epsilon_{k - 1} \vert + \vert \tilde \epsilon_k \vert \\
&= \left(1 + \frac{L}{N}\right) \vert \epsilon_{k + 1} \vert + \vert \tilde \epsilon_k \vert
\end{align}$$

We can expand the recursive relation to to completely express $$\vert \epsilon_k \vert$$ in terms of proceeding local truncation errors. Then, since the second-derivative $$f''$$ is bounded in our compact interval, each $$\vert \tilde \epsilon_i \vert$$ is bounded uniformly above by some $$\vert \epsilon \vert \sim O(N^{-2})$$ via Taylor's Theorem. From here, the relation simplifies naturally:

$$\vert \epsilon_k \vert \leq \sum_{i=1}^k \vert \tilde\epsilon_i \vert \sum_{j=0}^{k - i} \left( \frac{L}{N} \right)^j \leq \vert \epsilon \vert \sum_{i=1}^k \sum_{j=0}^{k-i} \left( \frac{L}{N} \right)^j \sim O(k N^{-2})$$

When we are trying to compute an estimate of $$h$$ at $$t = 1$$ without a loss of generality, then as we take larger $$N$$, we must linearly increase $$k$$ as well so that $$k = N$$. This shows that the error of our estimate at a fixed time $$t$$ is in fact $$O(N^{-1})$$ as we discussed.
