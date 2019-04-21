---
layout: post
title: "Interpreting Cross Entropy as a Loss Function"
date: 2018-03-06
tags: information-theory machine-learning shannon-entropy cross-entropy kullbeck-leibler-divergence
comments: true
---

$\DeclareMathOperator*{\argmin}{\arg\!\min}$
$\DeclareMathOperator*{\argmax}{\arg\!\max}$

# Introduction

We [previously](https://frankwang95.github.io/2018/02/shannon-entropy-as-expected-message-length) wrote about the Shannon entropy of a discrete, finite probability distribution $$(\mathcal{X}, p)$$ and how Shannon entropy can be interpreted as the expected code length when we send encoded messages describing independent draws from $$(\mathcal{X}, p)$$. Today we will look into a closely related subject, cross entropy, which is frequently used as a loss function for loss-based classification methods such as neural networks. In particular, we will discuss why cross entropy is the natural choice for this class of problem from an interpretive point of view.

Cross entropy is a non-commutative similarity measure between two probability distributions $$p$$ and $$q$$ on the same outcome space $$\mathcal{X}$$, which we will assume to be discrete and finite with with size $$n$$. Representing $$p$$ and $$q$$ as vectors in $$[0, 1]^n$$, cross entropy is defined by the formula:

  $$\mathcal{L}_{\text{ce}}(p, q) = - \sum_{i=1}^n p_i \log_2 (q_i)$$

We see that $$\mathcal{L}_{\text{ce}}$$ is well defined only when values in $$q$$ are nonzero though no such requirement is needed for $$p$$. These constraints typically must be respected by models using cross entropy. For instance, training targets will be given as $$p$$ because they are typically one-hot encoded and in unrestrictive model classes like neural-networks, softmax is usually applied to prevent 0-probability predictions.

As we expect, $$\mathcal{L}_{\text{ce}}(p, q)$$ is minimized when $$p$$ and $$q$$ are the same and in fact, we notice that in this case, cross entropy is exactly equal to Shannon entropy. This observation gives us a hint as to how we cross entropy can be interpreted.

# Cross Entropy as Message Lengths Under Nonoptimal Encoding

We have shown [previously](https://frankwang95.github.io/2018/02/shannon-entropy-as-expected-message-length) that to define an optimally bit efficient, prefix-free, binary encoding on the distribution $$q$$, an outcome occurring with probability $$q_i$$ should be assigned a code length of $$-\log_2(q_i)$$. With this in mind, $$\mathcal{L}_{\text{ce}}(p, q)$$ can be interpreted as the expected code length for us to communicate independent draws from $$(\mathcal{X}, p)$$ when we encode observations optimally for the probability distribution $$(\mathcal{X}, q)$$.

This interpretation is closely related to Kullbeck-Leibler divergence (which notably is an equivalent loss function to cross-entropy). Recall that the KL-divergence between $$p$$ and $$q$$ is given by:

  $$\mathcal{L}_{\text{kl}}(p, q) = \left[\sum_{i=1}^n p_i \log_2(p_i) - p_i\log_2(q_i)\right]$$

This is then the the cost incurred in expected code length when we mistakenly encode draws from $$(\mathcal{X}, p)$$ under the optimal encoding for $$(\mathcal{X}, q)$$ instead.

# Cross Entropy as Maximum Likelihood Estimation

Though the information-theoretic interpretation of cross entropy is neat, it does not provide a particularly satisfying reason for why it should be used as the canonical loss-function for classification. For a more generally appealing interpretation of cross entropy, we can show that minimizing cross entropy loss is equivalent to finding hyper parameters to our model using maximum likelihood estimation. We will derive this the same way we show this property for simple linear models and in fact, our claim is true for any regression outputting probability predictions.

To describe a classification problem mapping elements of $$\mathbb{R}^m$$ into the $$n$$ categories of $$\mathcal{X}$$, we represent our model by the a function $$\sigma(x, \theta)$$ taking a input vector $$x \in \mathbb{R}^m$$ and trainable parameters $$\theta$$ into a probability distribution $$[0, 1]^n$$. The likelihood given fixed training data $$(x_1, y_1), (x_2, y_2), \ldots, (x_N, y_N)$$ and parameters $$\theta$$ is given by:

  $$L(\theta) = \prod_{i=1}^N \sigma(x_i, \theta) \cdot y_i$$

As such, our problem is to show the following equivalence:

  $$\argmax_\theta \, L(\theta) = \arg\min_\theta \, \mathcal{L}_\text{ce}(y_i, \sigma(x_i, \theta))$$

We must take advantage of the fact that each $$y_i$$ comes in the form of a one-hot encoded vector in order to commute the dot-product with the logarithm, though this does not generally hold. Using this, our derivation is simple:

  $$\begin{align}
    \arg\min_\theta \, \mathcal{L}_\text{ce}(y_i, \sigma(x_i, \theta)) &= \arg\min_\theta \, \sum_{i=1}^N - y_i \cdot \log_2(\sigma(x_i, \theta)) \\
    &= \argmax_\theta \, \sum_{i=1}^N y_i \cdot \log_2(\sigma(x_i, \theta)) \\
    &= \argmax_\theta \, \sum_{i=1}^N \log_2(y_i \cdot \sigma(x_i, \theta)) \\
    &= \argmax_\theta \, \log_2\left[\prod_{i=1}^N y_i \cdot \sigma(x_i, \theta) \right] \\
    &= \argmax_\theta \, \log_2(L(\theta))) = \argmax_\theta \, L(\theta))
  \end{align}$$
