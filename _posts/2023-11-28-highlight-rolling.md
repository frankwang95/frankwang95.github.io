---
layout: post
title: "Studies on Highlight Rolling"
date: 2023-11-28
tags: snippits
---

My typical approach to tone mapping can be roughly be summarized as adding contrast via an S-curve while pushing shadows and managing oversaturation. I frequently make use of multiple exposures to capture the dynamic range I want with the aim being to compress as much dynamic information into the photograph as possible and using the entirety of the tonal range of the digital medium to represent that dynamic range.

I think this is typically considered the technical approach and it seems like the computational photography algorithms common in smartphones mpwadays are making use of tone maps optimizing for this. Recently, this style of image gives me a bit of an artifitial feeling which is why I've been carrying a point-and-shoot to use day-to-day instead of my phone.

In post, I've also playing with the idea of applying the opposite principal as before. Namely:


* Use tone curves that are not S shaped with the goal of allocating tonal range to bands that benefit from more contrast and taking range away from bands that don't.
* Unused tonal range should be removed with non-zero blackpoint or non-one white points.
* Roll shadows and highlights.
* Start with flat ICC profiles and linear tone maps
* Some images are good (or even better) out of focus or blurred.

The philosophy is that parts of a photograph (and in-particular textures) might simply be visually unimportant and thus distracting. A picture may benefit from having this contrast thrown away via tonal compression and in fact, we may not really need to use the whole tonal range at all. Blacks and whites can maybe be safely mapped to shades of grey. This produces a sort of muted, lo-fi style that emphasizes color and form more than texture.

This style of editing seems to have been gaining popularity alongside the analog trend for quite some time now. I'm a little bit late to this party having been stuck with a habit that I felt like was working well for me.