---
layout: post
title: "Existence and Uniqueness of Solutions to ODEs"
date: 2020-02-15
tags: ODEs analysis Picard-Lindelof
---

**Theorem (Picard-Lindelof):** Suppose we have a smooth function $$f: \mathbb{R} \rightarrow \mathbb{R}$$ which is Lipschitz with constant $$L$$ and some constant points $$t_0, x_0 \in \mathbb{R}$$. Then, for any $$q \in (0, 1)$$, there is an unique local solution $$x$$ to the initial value problem in the interval $$I_{t_0} = [t_0, t_0 + q/L]$$:

$$x'(t) = f(x(t)) \qquad x(t_0) = x_0$$

*Proof:* The general proof of the Picard-Lindelof theorem leans on the Banach-Caccioppoli fixed point theorem. In particular, if we consider the space of continuous functions $$\mathcal{C}(I_{t_0})$$ with domain $$I_{t_0}$$, we note that any solution to the initial value problem is a fixed point of the function $$T: \mathcal{C}(I_{t_0}) \rightarrow \mathcal{C}(I_{t_0})$$  given by:

$$Tg(t) = x_0 + \int_{t_0}^t f(g(\tau)) \, d\tau$$

It turns out that $$T$$ is a contraction mapping under the Chebyshev metric $$d$$ on $$I_{t_0}$$ for any $$q \in (0, 1)$$. The metric space defined by $$d$$ on $$\mathcal{C}(I_{t_0})$$ happens to be complete so the fixed point theorem implies that such a solution exists and is unique. To make this rigorous, we must prove that $$T$$ is well defined as stated, that $$d$$ is in fact a complete metric on $$\mathcal{C}(I_{t_0})$$, and that $$T$$ is a contraction mapping.

Justifying $$T$$ is well defined involves simply showing that $$f \circ g$$ is integrable and that $$Tg$$ is continuous. The first fact is elementary and the second follows immediately from the first Fundamental Theorem of Calculus.

To show that $$d$$ defines a valid metric on $$I_{t_0}$$, the only nontrivial fact to be shown is that $$d$$ adheres to the triangle inequality. Taking arbitrary $$g_1, g_2, g_3 \in \mathcal{C}(I_{t_0})$$, we can choose $$t_1, t_2, t_3 \in I_{t_0}$$ where these functions achieve their maximal difference in $$I_{t_0}$$ (note this makes use of the fact that $$I_{t_0}$$ is compact). Then, we have:

$$\begin{align}
d(g_1, g_3) &= \vert g_1(t_1) - g_3(t_1) \vert \\
&\leq \vert g_1(t_1) - g_2(t_1) \vert + \vert g_2(t_1) - g_3(t_1) \vert \\
&\leq \vert g_1(t_2) - g_2(t_2) \vert + \vert g_2(t_3) - g_3(t_3) \vert \\
&= d(g_1, g_2) + d(g_2, g_3)
\end{align}$$

The completeness of $$d$$ on the space of $$\mathcal{C}(I_{t_0})$$ follows simply from the uniform convergence theorem. Therefore, $$g$$ is continuous and thus lies in the space $$\mathcal{C}(I_{t_0})$$.

Finally, what remains to be shown is that $$T$$ is a contraction map. This requires use of the Lipschitz regularity of $$f$$ and the restricted length of $$I_{t_0}$$ as we see in the following:

$$\begin{align}
d(T g_1, T g_2) &= \max_{t \in I_{t_0}} \left\vert \int_{t_0}^t f(g_1(\tau)) \, d\tau - \int_{t_0}^t f(g_2(\tau)) \, d\tau \right\vert \\
&= \max_{t \in I_{t_0}} \left\vert \int_0^t f(g_1(\tau)) - f(g_2(\tau)) \, d\tau \right\vert \\
&\leq \max_{t \in I_{t_0}} \int_{t_0}^t \vert f(g_1(\tau)) - f(g_2(\tau)) \vert \, d\tau \\
&\leq \max_{t \in I_{t_0}} \int_{t_0}^t L \vert g_1(\tau) - g_2(\tau) \vert \, d\tau \\
&\leq \frac{q}{L} L \max_{\tau \in I_{t_0}}\vert g_1(\tau) - g_2(\tau) \vert = q d(g_1, g_2)
\end{align}$$

<div style="text-align: right">QED</div>

We note that the length of the interval $$I_{t_0}$$ provided by Picard-Lindelof is independent of our explicit choice of $$t_0$$ and $$x_0$$ and depends only on the local Lipschitz constant around $$t_0$$. Thus, if $$f$$ has a global Lipschitz constant as we describe above, we can cover $$\mathbb{R}$$ with countably many Picard-Lindelof intervals to obtain a proof for global uniqueness as well.
