---
layout: post
title: "Deriving Adjoint Differentiation of ODEs"
date: 2020-01-05
tags: ODE numerical analysis
---

Consider a smooth function $$x(t, p): \mathbb{R} \times \mathbb{R}^{n_p} \rightarrow \mathbb{R}$$ given by the standard ordinary differential equation problem:

$$ x_t(t, p) = f(x(t, p), p) \qquad x(0, p) = x_0$$

where $$f$$ is assumed to be smooth. In this problem, $$p \in \mathbb{R}^{n_p}$$ can be considered to be a set of parameters for a temporal model $$x$$.  It's often useful but difficult to be able to compute the derivative of $$x$$ with respect to $$p$$ in order to use gradient based optimization methods.

In this post, we'll derive one way to efficiently compute $$x_p$$ called the adjoint method. In a future post, we will discuss some less-efficient alternatives to this method and compare the performance of these options both theoretically and experimentally.

**Theorem:** Given a functions $$x$$ and $$f$$ as described above, then a fixed value of $$T$$ and $$p$$, if we define an adjoint function $$\lambda: \mathbb{R} \times \mathbb{R}^{n_p} \rightarrow \mathbb{R}$$ as the solution to the following ODE with an initial value of $$\lambda(T) = 0$$: 

$$\lambda_t(t, p) = f_x(x(t, p), p)[1 - \lambda(t, p)]$$

Then, it follows that the derivative $$x_p$$ at $$T, p$$ can be written as:

$$x_p(T, p) = \int_0^T f_p(x(\tau, p), p) - \lambda(\tau, p) f_p(x(\tau, p), p) \, d\tau$$

*Proof:* To derive this theorem, we make use of the method of Lagrangians.  We start by defining two constraint functions:

$$g(x) = x - x_0 \qquad h(x, \dot x) = \dot x - f(x)$$

Notice that these functions satisfy the zero-identity where $$g(x(0)) = 0$$ and $$h(x(t), x_p(t)) = 0$$ for all $$t$$ and $$p$$. As such, if we define a Lagrangian $$L$$ as follows, regardless of our choice of function for $$\lambda$$ and $$\mu$$, the terms they reside in revert to zero, so that $$L(t) = x(t)$$. We note that it will suffice to set $$\mu$$ to be a constant to get out desired result.

$$L(T) = \int_0^T f(x(\tau)) + \lambda(\tau) h(x(\tau), x_t (t)) \, d\tau + \mu g(x(0))$$

Since $$L(t) = x(t)$$ for all $$t$$ and $$p$$, it follows that $$L_p(T) = x_p(T)$$ everywhere. As such, to computing $$x_p$$ can be done by differentiating $$L$$ with respect to $$p$$ where we can make use of freedom we have in choosing $$\lambda$$ and $$\mu$$ to eliminate difficult to compute terms during the differentiation. To see how this can be done, we can begin by naive differentiating $$L$$ with respect to $$p$$ at $$T, p$$ as follows:

$$L_p(T) = \int_0^T f_x x_p + f_p + \lambda_p h(x, x_t) + \lambda [h_x x_p + h_{\dot x} x_{t,p} + h_p] \, d\tau+ \mu g_x x_p \vert_0$$

The above expression follows from extensive application of the multivariate chain rule and product rule. It also requires us to commute the indefinite integral with a partial differentiation operation which is valid when the integrand is differentiable with respect to $$p$$ and has a derivative with respect to $$\tau$$ that is integrable over $$\tau$$. It's worth noting that checking that the integrand meets this condition requires is independent of our choice fo $$\lambda$$ due to the zero-identity property of $$h$$ and thus is true given only the good behavior of $$f$$.

Noting that $$h_{\dot x} = 1$$ is constant, we can reduce one of the terms in the above integral by integrating by parts to get:

$$\int_0^T \lambda h_{\dot x} x_{t,p} \, d\tau = \lambda  x_p \rvert^T_0 - \int_0^T \lambda_t x_p\, d\tau$$

Taking this, we can further expand $$L_p(T)$$ to the following form:

$$\begin{array}{lll}
L_p(T) & \!\!=\!\! & \displaystyle \int_0^T f_x x_p + f_p + \lambda_p h + \lambda [h_x x_p + h_p ] - \lambda_t  x_p \, d\tau + \\
       &           & \displaystyle \mu g_x x_p \vert_0 + \lambda x_p \vert_T - \lambda  x_p \vert_0 \\
       & \!\!=\!\! & \displaystyle \int_0^T f_p + \lambda_p h + \lambda h_p + x_p[f_x + \lambda h_x - \lambda_t] \, d\tau + \\
       &           & \displaystyle x_p (\mu g_x - \lambda) \vert_0+ x_p \lambda \vert_T
\end{array}$$

This expression reveals to us everything we need to choose $$\lambda$$ and $$\mu$$. In the final form of the expression above, we'd like ideally to eliminate any of the terms containing $$x_p$$ in some form or another.

1. The $$x_p$$ term in the integral can be dropped by solving $$f_x + \lambda h_x - \lambda_t = 0$$. Noting that $$h_x = -f_x$$, this gives us the differential equation $$\lambda_t = f_x - \lambda f_x$$.
2. The $$x_p\lambda \vert_T$$ term at the end of the equation can eliminated with an initial value condition $$\lambda(T) = 0$$.
3. Finally, the $$x_p(\mu g_x - \lambda) \vert_0$$ term disappears by setting $$\mu = g_x^{-1}\lambda = \lambda$$ though it is worth noting that applying this theorem requires no explicit consideration of the $$\mu$$ term of the Lagrangian.

Finally, we can clean up all these equations by explicitly computing the terms containing $$h$$ and $$g$$ and dropping terms that resolve to zero. Our strategic choices for $$\lambda$$ and $$\mu$$ lets us drop all the terms containing $$x_p$$ and furthermore $$\lambda h$$ can be dropped due to the zero-identity property of $$h$$. Finally, since we have $$\lambda h_p = \lambda f_p$$, we are left with our final formula which matches the one in the theorem statement:

$$x_p(T)  = L_p(T) = \int_0^T f_p + \lambda f_p \, d\tau$$ 

<div style="text-align: right">QED</div>

# Easy Closed Form Example

To check our math, it's illuminating to work through a simple example in which all both $$x$$ and $$\lambda$$ have closed form expressions. The simple linear ODE works perfectly here:

$$x(0) = 1 \quad x_t(t, p) = f(x(t, p), p) = p x(t, p)$$

The closed form of this ODE has the solution $$x(t, p) = \exp(pt)$$. In general, deriving $$x_p(t, p) = t \exp(pt)$$ is easy as well.

After fixing the points $$T$$ and $$p$$ at which we'd like to evaluate $$x_p$$, the theorem dictates that $$\lambda$$ be the solution to the following initial value problem:

$$\lambda(T) = 0 \qquad \lambda_t(t, p) = p - \lambda(t, p) p$$

This is conveniently solvable in closed form to give us $$\lambda(t, p) = 1 - \exp(p(T - t))$$.

The equality in the presented theorem them reduces to the following computation:

$$\begin{align}
  &\int_0^T f_p(x(\tau, p), p) - \lambda(\tau, p) f_p(x(\tau, p), p) \, d\tau \\
=\quad &\int_0^T x(t, p) - (1 - \exp(p (T - t)) x(t, p) \, d\tau \\
=\quad &\int_0^T \exp(p (T - t)) x(t, p) \, d\tau = \int_0^T \exp(pT) d\tau\\
=\quad &T \exp(T, p) = x_p(T, p)
\end{align}$$

# Extensions

We'd like to  briefly note some simple extensions to the adjoint method we derived above. Firstly, it's very simple to extend this method to cases in which the initial value of $$x$$ depends on $$p$$ such that $$x(0, p) = x_0(p)$$ for some smooth function $$x_0$$. This is done simply by expanding the constraint function $$g$$ to depend also on $$p$$ so that $$g(x, p) = x - x_0(p)$$. Along a similar vein, this method can be easily extended to slightly nonstandard differential equations like for instance if $$f(x, t, p)$$ also contained explicit dependence on the the timing parameter. Both these extensions make use of the exact same methods as the derivation we provide above taking care to account for the additional terms from the chain rule that would consequently arise.

Expanding this method to the multivariate case is a bit more tricky as some special care is needed with regards to commutativity, and parameterization of the line integral. Elementary integration by parts needs also to be generalized, I think, to Green's theorem. I have not taken the time to go through this in detail though I would like to do so someday.
