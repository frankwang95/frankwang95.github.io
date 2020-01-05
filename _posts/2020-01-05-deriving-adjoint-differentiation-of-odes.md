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

*Proof:* To derive this theorem, we make use of the method of Lagrangians.  We start by defining two constraint functions:

$$g(x) = x - x_0 \qquad h(x, \dot x) = \dot x - f(x)$$

Notice that these functions satisfy the zero-identity where $$g(x(0)) = 0$$ and $$h(x(t), x_p(t)) = 0$$ for all $$t$$ and $$p$$. As such, if we define a Lagrangian $$L$$ as follows, regardless of our choice of function for $$\lambda$$ and $$\mu$$, the terms they reside in revert to zero, so that $$L(t) = x(t)$$. We note that it will suffice to set $$\mu$$ to be a constant to get out desired result.

$$L(T) = \int_0^T f(x(\tau)) + \lambda(\tau) h(x(\tau), \dot x (t)) \, d\tau + \mu g(x(0))$$

Since $$L(t) = x(t)$$ for all $$t$$ and $$p$$, it follows that $$L_p(T) = x_p(T)$$ everywhere. As such, to computing $$x_p$$ can be done by differentiating $$L$$ with respect to $$p$$ where we can make use of freedom we have in choosing $$\lambda$$ and $$\mu$$ to eliminate difficult to compute terms during the differentiation. To see how this can be done, we can begin by naive differentiating $$L$$ with respect to $$p$$ at $$T, p$$ as follows:

$$L_p(T) = \int_0^T f_x x_p + f_p + \lambda_p h(x, x_t) + \lambda [h_x x_p + h_{\dot x} x_{t,p} + h_p] \, d\tau+ \mu g_x x_p(0)$$

<div style="text-align: right">QED</div>
