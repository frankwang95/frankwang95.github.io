---
layout: post
title: "Bounds on the NeRF Integral"
date: 2025-12-10
tags: NeRF
---

Recall the two integrals used to calculate volume along a ray in a NeRF:

$$c = \int_0^\infty T(t) \sigma(t) c(t) \,dt \qquad \qquad T(t) = \exp\left(-\int_0^t \sigma(s) \,ds\right)$$

In the first integral, $T(t)\sigma(t)$ are continuous weights along the ray representing how much of the color $c(t)$ will reach the camera through the volume that exists in between. It would make sense for these weights to satisfy the following relation for any $\tau$:

$$\int_0^\tau T(t) \sigma(t) \,dt \in [0, 1]$$

Namely with the integral taking the value of 0 if the volume between $0$ and $\tau$ is totally transparent and 1 if the volume between $0$ and $\tau$ is totally opaque. In our [previous derivations](https://frankwang95.github.io/2024/07/deriving-volumetric-rendering) we can see that $T, \sigma \geq 0$ by definition. This gives us our lower bound trivially. Furthermore, we can see that $T \in [0, 1]$ trivially from definition. In our previous derivations, we show that $dT(\tau) = -\sigma(\tau) T(\tau)$ and therefore our upper bound arises as a consequence of the fundamental theorem of calculus:

$$\int_0^\tau T(t) \sigma(t) d,dt = T(0) - T(\tau) = 1 - T(\tau) \leq 1$$