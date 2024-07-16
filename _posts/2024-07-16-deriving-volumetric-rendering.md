---
layout: post
title: "Deriving the Volumetric Rendering Function"
date: 2024-07-16
tags: snippits
---

A Neural Radiance Field (NeRF) use a volumetric rendering function to render the color for a camera ray:

$$\int_0^\infty T(t) \sigma(t) c(t) \text{dt} \qquad \text{where} \qquad T(t) = \exp\left(-\int_0^t \sigma(s) \text{ds}\right)$$

Here, $$\sigma(t)$$ and $$c(t)$$ are volume density and RGB color properties of the point $$t$$ distance into the ray. $$T(t)$$ is the accumulated transmittance and can be interpreted intuitively as the amount of light from point $$t$$ which will reach the camera through the volume between.

Note that calculating this integral numerical involves calculated at discrete points, say $$t_1, t_2 \ldots t_n$$. However, if we treat the transmittance as a discrete phenomena along our ray, we might expect something like the following as our accumulated transmittance:
