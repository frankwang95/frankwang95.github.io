---
layout: post
title: "Deriving Shannon Entropy Using Constrained Optimization"
date: 2018-02-14
tags: information-theory machine-learning shannon-entropy cross-entropy kullbeck-leibler-divergence
comments: true
---

# Introduction

Shannon entropy is a significant topic in information theory and makes interesting appearances in machine learning, especially in the form of cross-entropy loss and Kullbeck-Leibler divergence. One of the many interpretations of Shannon entropy is as a theoretical lower bound on lossless message encoding. In this post we will derive this interpretation of Shannon entropy for a special class of message encodings called prefix encodings.

Given a finite, discrete, probability space $$\mathcal{X}$$ of size $$n$$, a probability distribution $$p$$ on $$\mathcal{X}$$ can be represented by a vector in $$[0, 1]^n$$ with components summing to one. The Shannon entropy of $$p$$ is then given by the formula:

  $$\mathcal{E}(p) = -\sum_{i=1}^n p_i \log_2(p_i)$$

Note that to ensure that the logarithm is well defined, we must assume without a loss of generality every element of $$\mathcal{X}$$ is assigned nonzero probability.

At face value, Shannon entropy measures uncertainty in the probability distribution. It is maximized when each outcome in $$\mathcal{X}$$ has equal probability of occurring. In this case, the probability distribution is "unpredictable" in the sense that our best guess of a draw from $$\mathcal{X}$$ is to arbitrary choose. As the probability of different outcomes become imbalanced and our ability to guess draws from $$\mathcal{X}$$ improves, we see a corresponding decrease in entropy.

That being said, it is not immediately clear where this formula comes from, especially since we can think of many formulas for measuring "uncertainty" in the sense we described above. However, Shannon entropy actually has a remarkably elegant interpretation in information theory related to the amount of bandwidth needed to send an encoded message.

Suppose that we would like to communicate, using binary strings, a sequence of observations $$x_1, x_2, x_3, \ldots$$ all drawn independently from the probability distribution $$(\mathcal{X}, p)$$. One way we can do this is to give each code a length of $$\log_2(n)$$. Then we have enough bits to encode each element of $$\mathcal{X}$$ as a unique text string to send. If we send messages in this naive way, we have an expected code length of:

  $$\sum_{i=1}^n p_i \log_2(n)$$

But what if we want to communicate our observations more efficiently using less bits? It turns out that in some cases, by encoding our outcomes in clever ways, we can reduce their expected length. Consider if one outcome occurs with much greater frequency than the others. In this case, we might expect to benefit by making the code for this outcome much shorter, even if it comes at the expense of making the other codes longer. We can formalize this idea and eventually show that the optimal code length for an outcome with probability $$p_i$$ is $$-\log_2(p_i)$$ regardless of the probability of the other possible events. From this fact, it becomes clear that the expression for Shannon entropy above is exactly the expected message length under our optimal encoding.


# Prefix-Free Encodings and Uniquely Decodable Encodings

Lets start by considering what we must costs we incur to shorten the length of a code. The primary concern is whether our sequence of codes can be uniquely interpreted by the receiver. If, for example, we had three possible outcomes $$1$$, $$2$$, or $$3$$, which we encode by `010`, `01`, and `0` respectively, then if we receive the string `0100`, there is no way for us to know whether the message indicates the outcome sequence $$(1, 3)$$ or $$(2, 3, 3)$$.

The mathematical term we use to describe a valid code is an uniquely decodable encoding. That is, given a string in the form of many codes conjoined to each other, there is only one possible sequence of outcomes that corresponds to that string. Notice that one rule we can use to give us an uniquely decodable encoding is that no code should be the prefix of any other. The class of encodings which satisfy this property are called prefix-free encodings (or sometimes prefix encodings). Though it is clear that prefix encodings are uniquely decodable, it is noteworthy there are uniquely decodable encodings which are not prefix-free.

Since we are concerned principally with optimizing our encodings so as to minimize length, the exact way we encode each outcome does not matter and instead we only need to consider the length of each code. Thus, each possible encoding for our probability distribution $$(\mathcal{X}, p)$$ can be represented by an integer vector $$m \in \mathbb{Z}_+^{n}$$.

In general, there exists a prefix-free encoding with code lengths consistent with $$m$$ if and only if $$m$$ satisfies the constraint:

  $$\sum_{i=1}^n \frac{1}{2^{m_i}} \leq 1$$

An intuitive explanation of this is that in binary encodings, given a code of length $$m_i$$, our code is the prefix of exactly $$1/2^{m_i}$$ of all possible binary encodings in some sense. For instance, half of all codes begin with a `1` and the other half of codes begin with a `0`. Similarly, every possible code of length greater than or equal to two have one of the four length two codes (`00`, `01`, `10`, and `11`) as a prefix. From here, we notice that choosing a length one code for one of the possible observations 'disqualifies', half of all possible codes from being chosen for other observations just like choosing a length two code for an observation 'disqualifies' a quarter of them.

Despite my hand-wavy explanation here, this statement turns out not to be too hard to show formally and perhaps one day I will return to this post and fill it in.

# Optimal Encodings as a Constrained Optimization

Having established the fundamental constraint which characterizes prefix encodings, we move on to state the main result - to show that Shannon entropy $$\mathcal{E}(p)$$ is the expected message length when we encode observations from $$(\mathcal{X}, p)$$ with the optimal  prefix-free encoding in the sense of minimizing expected bits. Using the notation $$m(x)$$ for the length of the code assigned to a random observation $$x$$ under the encoding $$m$$, we show the following:

  $$\mathcal{E}(p) = \left[ \min_{m} \, \mathbb{E} [m(x)] \quad \text{s.t.} \quad \sum_{i=1}^n \frac{1}{2^{m_i}}  \leq 1\right]$$

This problem can be solved using the method of Lagrangian Multipliers.

Firstly, to show that our constraint is binding, we can use the Karush-Kuhn-Tucker Conditions though I will omit this due to lack of personal interest. Instead, I will vaguely support this claim by saying that when the constraint is slack, we are wasting valid codes whose inclusion would not result in ambiguous decoding. With that, we go about solving the optimization above by following finding critical points to the Lagrangian over $$m$$ and $$\lambda$$:

  $$\begin{align}
  L(m, \lambda) &= \mathbb{E}[m(x)] - \lambda \left(1 - \sum_{i=1}^n \frac{1}{2^{m_i}}\right) \\
  &= \sum_{i=1}^n p_i m_i - \lambda \left(1 - \sum_{i=1}^n \frac{1}{2^{m_i}}\right)
  \end{align}$$

The first order conditions are pretty simple to compute for each $$m_i$$ and $$\lambda$$:

  $$\begin{align}
  \frac{\partial L}{\partial m_i} &= p_i - \lambda \log(1/2)\frac{1}{2^{m_i}} &&\qquad (1)\\
  \frac{\partial L}{\partial \lambda} &= 1 - \sum_{i=1}^n \frac{1}{2^{m_i}} &&\qquad (2)
  \end{align}$$

Setting each of the above equal to 0, we solve for our critical points. Equation $$(1)$$ gives us each $$m_i$$ written as a function of $$\lambda$$:

  $$m_i = \log_2\left( \frac{\lambda \log(1/2)}{p_i} \right) \qquad \qquad \quad (3)$$

Then substituting the above equation for each $$m_i$$ into equation $$(2)$$, yields a unique critical point for $$\lambda$$, making use of the fact that $$\sum_i p_i = 1$$:

  $$\lambda = \frac{1}{\log(1/2)}$$

Substituting this back into equation $$(3)$$ gives us the critical values for each $$m_i$$:

  $$m_i = \log_2 (1 / p_i) = -\log_2(p_i)$$

Since the system of equations to solve have a unique solution, there is no question that the optimal values for $$m$$ and $$\lambda$$ are not local optimums. Furthermore, the Hessian matrix for our Lagrangian can be easily checked to be positive-definite, implying that our critical points are global minima.

This proves that to choose a prefix-free encoding which minimizes expected message length, an outcome with probability $$p_i$$ should be encoded with a binary string of length $$-\log_2 (p_i)$$ as we claimed. The expected message length over the whole distribution $$(\mathcal{X}, p)$$ is then $$- \sum_i p_i \log_2(p_i)$$ which is exactly the formula for Shannon entropy.
