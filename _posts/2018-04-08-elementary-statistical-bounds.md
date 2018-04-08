---
layout: post
title: "Common Statistical Identities for Deriving Theoretical Error Bounds"
date: 2018-04-08
tags: bounds theory chebyshev markov
comments: true
---

In this blog post, we derive four basic, fundamental identities frequently used in proofs of theoretical error bounds for statistical models and machine learning. Later on we'll use some of these tools in looking at asymptotic properties of random forests as well as general min-max bounds.

# Markov's Inequality

**Theorem (Markov's Inequality):** For a nonnegative random variable $$X$$ and $$a > 0$$, we have the following identity:

  $$\mathbb{P}(X \geq a) \leq \frac{\mathbb{E}(X)}{a}$$

*Proof:* Let $$f$$ give the probability density function of $$X$$. We can relax $$\mathbb{P}(X \geq a)$$ by adding a nonnegative quantity:

  $$\mathbb{P}(X \geq a) = \int_a^\infty f(x) dx \leq \int_0^a \frac{x f(x)}{a} dx + \int_a^\infty f(x) dx$$

Then since $$\frac{x}{a} > 1$$ on the range $$(a, \infty)$$, we can relax the latter by a factor of $$\frac{x}{a}$$, giving us our desired result:

  $$\ldots \leq \int_0^a \frac{x f(x)}{a} dx + \int_a^\infty \frac{x f(x)}{a} dx = \frac{1}{a} \int_0^\infty x f(x) dx = \frac{\mathbb{E}(X)}{a}$$

The final equality requires of course that $$X$$ is nonnegative.

<div style="text-align: right">QED</div>

It's worth noting that while Markov's Inequality is usually a loose upper bound, no stronger bound exists in general for all nonnegative inequalities. By this, we simply mean that for every $$a$$, we can construct a random variable $$X$$ for which Markov's Inequality binds.

As an example, we can achieve this simply by having $$X$$ take the value $$a$$ with probability one. From the proof, we can derive some necessary conditions to achieve $$\mathbb{P}(X \geq a) = \mathbb{E}(X) / a$$. Namely, this can be the case only when we have both of the following:

  $$\int_0^a \frac{x f(x)} {a} dx = 0 \qquad \qquad \int_a^\infty \frac{x f(x)} {a} dx = \int_a^\infty x f(x) dx$$

As such, any random variable $$X$$ on which Marov's inequality binds can only take the values $$a$$ and $$0$$.

# Chebyshev's Inequality

**Theorem (Chebyshev's Inequality):** For any $$k  > 0$$ and random variable $$X$$ with mean $$\mu$$ and variance $$\sigma^2$$, we have:

  $$\mathbb{P}(\vertX - \mu\vert \geq k \sigma) \leq \frac{1}{k^2}$$

*Proof:* Chebyshev's inequality follows directly from Markov's inequality:

  $$\mathbb{P}(\vertX - \mu\vert \geq k\sigma) = \mathbb{P}((X - \mu)^2 \geq k^2 \sigma^2) \leq \frac{\mathbb{E}[(X - \mu)^2]}{k^2 \sigma^2} = \frac{1}{k^2}$$

<div style="text-align: right">QED</div>

Chebyshev's inequality is tight in the same sense as Markov's Inequality. To derive a binding example, we must construct $$X$$ so that Markov's inequality binds on $$(X - \mu)^2$$ with $$a = k^2 \sigma^2$$. The necessary condition we discussed earlier for Markov's inequality to bind are helpful here.

One solution is to take $$X$$ to be the discrete random variable on $$\{-1, 0, 1\}$$ with the following probability distribution:

  $$p_X(x) = \begin{cases}
    \frac{1}{2k^2} &: x = \pm 1 \\
    1 - \frac{1}{k^2} &: x = 0
  \end{cases}$$

Checking that $$X$$ binds Chebyshev's Inequality is simple and we'll leave that to you.
# Borel-Cantelli Lemmas

**Theorem (Borel-Cantelli):** Suppose that we have a sequence $$\mathcal{A} = A_1, A_2, A_3, \ldots$$ of events on some probability space. If the series $$\sum_k \mathbb{P}(A_k)$$ converges, then with probability one, only a finite number of the events from $$\mathcal{A}$$ occur.

*Proof:* For each $$r = 0, 1, 2, \ldots$$, let $$B_r$$ be the indicator for the case in which $$\geq r$$ events from $$\mathcal{A}$$ are true. As such, it suffices to prove the following:

  $$\lim_{r \rightarrow \infty} \mathbb{P} (B_r) = 0$$

Taking arbitrary $$\epsilon > 0$$, since $$\sum_k \mathbb{P}(A_k)$$ converges, there exists $$K$$ sufficient large such that:

  $$\sum_{k = K}^\infty \mathbb{P}(A_k) < \epsilon$$

$$B_K$$ requires at least one event from the tail $$A_K, A_{K + 1}, \ldots$$ to be true and as such, we have:

  $$\mathbb{P}(B_K) \leq \mathbb{P} \left( \bigcup_{k=K}^\infty A_k\right) \leq \sum_{k=K}^\infty \mathbb{P} (A_k) < \epsilon$$

Since the sequence $$\{\mathbb{P} (B_r)\}_r$$ is descending, this suffices to prove the desired limit above.

<div style="text-align: right">QED</div>

# The Strong Law of Large Numbers

We can use Borel-Cantelli and Chebyshev's inequality to give us a short proof of the strong law of large numbers which is a major cornerstone in both classical statistics as well as machine-learning.

**Theorem (Strong Law of Large Numbers):** Suppose that we have a sequence of iid. random variables $$X_1, X_2, X_3, \ldots$$ which each have mean $$\mu$$ and variance $$\sigma^2$$. Let $$\bar X_n$$ be the sample mean of the first $$n$$ terms of our sequence:

  $$\bar X_n = \frac{1}{n} \sum_{k=1}^n X_k$$

Then, with probability one, the sequence $$\bar X_n \rightarrow \mu$$ as $$n \rightarrow \infty$$.

*Proof:* The sequence $$\bar X_n$$ fails to converge to $$\mu$$ if and only if there exists $$\delta > 0$$ such that $$\vert\bar X_n - \mu\vert > \delta$$. As such, taking arbitrary $$\delta > 0$$, it suffices to prove that, with probability one, there is a largest $$n$$ for which the inequality $$\vert\bar X_n - \mu\vert > \delta$$ is true. The Borel-Cantelli Lemma give this to us if we can show the follow series converges:

  $$\sum_{n=1}^\infty \mathbb{P}(\vert\bar X_n - \mu\vert > \delta) < \infty$$

It is evident that $$\bar X_n$$ has mean $$\mu$$. Since our sequence is independent, its variance can be derived to be $$(\sigma / n)^2$$. As such, Chebyshev's inequality gives us the following inequality:

  $$\mathbb{P}(\vert\bar X_n - \mu\vert > \delta) \leq \frac{\sigma^2}{n^2 \delta^2}$$

This bounds our series by a converging arithmetic series which proves our desired result:

  $$\sum_{n=1}^\infty \mathbb{P}(\vert\bar X_n - \mu\vert > \delta) \leq \sum_{n=1}^\infty \frac{\sigma^2}{n^2 \delta^2} < \infty$$

<div style="text-align: right">QED</div>

Notice the independence assumption is used in deriving the variance of the sample mean and is necessary despite the absence of such an asusmption for Borel-Cantelli.
