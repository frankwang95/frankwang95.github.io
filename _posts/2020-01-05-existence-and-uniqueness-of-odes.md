---
layout: post
title: "Existence and Uniqueness of Solutions to ODEs"
date: 2020-02-15
tags: ODEs analysis Picard-Lindelof
---

**Theorem (Picard-Lindelof):** Suppose we have a smooth function $$f: \mathbb{R} \rightarrow \mathbb{R}$$ which is Lipschitz with constant L and some constant points $$t_0, x_0 \in \mathbb{R}$$. Then, for any $$q \in (0, 1)$$, there is an unique local solution $$x$$ to the initial value problem in the interval $$I_{t_0} = [t_0, t_0 + q/L]$$:

$$x'(t) = f(x(t)) \qquad x(t_0) = x_0$$

*Proof:* The general proof of the Picard-Lindelof theorem leans on the Banach-Caccioppoli fixed point theorem. In particular, if we consider the space of continuous functions $$\mathcal{C}(I_{t_0})$$ with domain $$I_{t_0}$$, we note that any solution to the initial value problem is a fixed point of the function $$T: \mathcal{C}(I_{t_0}) \rightarrow \mathcal{C}(I_{t_0})$$  given by:

$$Tg(t) = x_0 + \int_{t_0}^t f(g(\tau)) \, d\tau$$

It turns out that $$T$$ is a contraction mapping under the Chebyshev metric $$d$$ on $$I_{t_0} $$ for any $q \in (0, 1)$. The metric space defined by $$d$$ on $$\mathcal{C}(I_{t_0})$$ happens to be complete so the fixed point theorem implies that such a solution exists and is unique. To make this rigorous, we must prove that $$T$$ is well defined as stated, that $$d$$ is in fact a complete metric on $$\mathcal{C}(I_{t_0})$$, and that $$T$$ is a contraction mapping.

<div style="text-align: right">QED</div>

We note that the length of the interval $I_{t_0}$ provided by Picard-Lindelof is independent of our explicit choice of $$t_0$$ and $$x_0$$ and depends only on the local Lipschitz constant around $$t_0$$. Thus, if $$f$$ has a global Lipschitz constant as we describe above, we can cover $$\mathbb{R}$$ with countably many Picard-Lindelof intervals to obtain a proof for global uniqueness as well.
