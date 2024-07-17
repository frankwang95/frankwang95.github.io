---
layout: post
title: "Deriving the Volumetric Rendering Function"
date: 2024-07-16
tags: NeRF
---

A Neural Radiance Field (NeRF) use a volumetric rendering function to render the color for a camera ray:

$$\int_0^\infty T(t) \sigma(t) c(t) \,dt \qquad \qquad T(t) = \exp\left(-\int_0^t \sigma(s) \,ds\right)$$

Here, $$\sigma(t)$$ and $$c(t)$$ are volume density and RGB color properties of the point $$t$$ distance into the ray. $$T(t)$$ is the accumulated transmittance and can be interpreted intuitively as the amount of light from point $$t$$ which will reach the camera through the volume that exists in between.

Note that taking this integral numerically involves making calculations at discrete points, say $$t_1, t_2 \ldots t_n$$. However, if we treat the transmittance as a discrete phenomena along our ray, we might expect something like the following as our accumulated transmittance:

$$T(t_i) = \prod_{j = 1}^{i - 1} 1 - \sigma(t_j)$$

This expression does not have the exponential and negative terms in the continuous version of $$T$$. Where do these come from?

For me, it is intuitive to start with the accumulated radiance $$T$$, reparameterized as $$T(t_1, t_2)$$, representing the probability that a particle of light passes through the space between $$t_1$$ and $$t_2$$. This should satisfy $$T(t_1, t_2) T(t_2, t_3) = T(t_1, t_3)$$ for any $$t_1, t_2, t_3$$ as the conjunction of two independent probabilities. We also adopt $$T(t) = T(0, t)$$ as is consistent with our earlier usage of $$T$$. Then, the derivative of this function can be written as follows:

$$\begin{align}
    dT(t) &= \lim_{\epsilon \rightarrow 0} \frac {T(0, t + \epsilon) - T(0, t)} {\epsilon} \\
    &= \lim_{\epsilon \rightarrow 0} \frac {T(0, t) T(t, t + \epsilon) - T(0, t)} {\epsilon}\\
    &= - T(0, t) \lim_{\epsilon \rightarrow 0} \frac {1 - T(t, t + \epsilon)} {\epsilon}
\end{align}$$

Here we define $$\sigma(t) = \lim_{\epsilon \rightarrow 0}(1 - T(t, t + \epsilon)) / \epsilon$$. This gives us the following initial value problem for $$T$$:

$$dT(t) = - \sigma(t) T(t) \qquad T(0) = 1$$

This differential equation can be solved algebraically:

$$\begin{align}
    dT(t) &= -\sigma(t) T(t) \\
    \frac {dT(t)} {T(t)} &= -\sigma(t) \\
    \int_0^\tau\frac {dT(t)} {T(t)} \,dt &= -\int_0^\tau \sigma(t) \,dt \\
    \log(T(\tau)) - \log(T(0)) &= - \int_0^\tau \sigma(t) \,dt \\
    T(\tau) - \log(1) = T(\tau) &= \exp\left(-\int_0^\tau -\sigma(t) \,dt\right)
\end{align}$$

This derives the relationship between $$T$$ and $$\sigma$$ and in particular defines $$\sigma$$ in relation to $$T$$ as $$\lim_{\epsilon \rightarrow 0} T(t, t + \epsilon) / \epsilon$$. We note in particular that this limit is not bounded so $$\sigma(t)$$ takes values in $$\mathbb{R}^+$$ and should not be interpreted as a probability or proportion.
