---
layout: post
title: "Deriving Adjoint Differentiation of ODEs"
date: 2020-01-05
tags: ODE numerical analysis
---

Consider a smooth function $$x(t, p): \mathbb{R} \times \mathbb{R}^{n_p} \rightarrow \mathbb{R}$$ given by the standard ordinary differential equation problem:

$$ x_t(t, p) = f(x(t), p) \qquad x(0, p) = x_0$$

where $$f$$ is assumed to be smooth. In this problem, $$p \in \mathbb{R}^{n_p}$$ can be considered to be a set of parameters for a temporal model $$x$$.  It's often useful but difficult to be able to compute the derivative of $$x$$ with respect to $$p$$ in order to use gradient based optimization methods.

In this post, we'll derive one way to efficiently compute $$x_p$$ called the adjoint method. In a future post, we will discuss some less-efficient alternatives to this method and compare the performance of these options both theoretically and experimentally.

**Theorem:** Given a functions $$x$$ and $$f$$ as described above, then a fixed value of $$T$$ and $$p$$, if we define an adjoint function $$\lambda: \mathbb{R} \times \mathbb{R}^{n_p} \rightarrow \mathbb{R}$$ as the solution to the following ODE with an initial value of $$\lambda(T) = 0$$: 

$$\lambda_t(t, p) = f_x(t, p)[1 - \lambda(t, p)]$$

Then, it follows that the derivative $$x_p$$ at $$T, p$$ can be written as:

$$x_p(T, p) = \int_0^T f_p(x(\tau, p), p) - \lambda(\tau, p) f_p(x(\tau), p) \, d\tau$$
