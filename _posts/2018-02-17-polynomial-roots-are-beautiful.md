---
layout: post
title: "Beautiful Fractal Patterns in the Roots of Polynomials"
date: 2018-02-17
tags: polynomials root-solving fractals math
---

Polynomials are a pretty common mathematical construct. On a ring $$R$$, the polynomial ring $$R[x]$$ in invariant $$x$$ is the space of functions of the form:

  $$x \longmapsto \sum_{k=0}^n a_{k} x^k$$

for arbitrary integer $$n$$ and field elements $$a_k \in R$$. Besides being an ring, $$R[x]$$ has all sorts of interesting properties in abstract algebra. It is, for instance, commutative if $$R$$ is commutative and forms an integral domain if and only if $$R$$ has a field structure. None of that, however, will be the topic of this post. Today, we will focus on the roots of complex polynomials and more specifically, on how pretty they can look:

<br>
<p align="center">
  <a href="https://frankwang95.github.io/assets/polynomial_roots_full.jpg" class="no-hov">
  <img style="max-width: 1024px; margin: 0 0 0 -162px;" src="https://frankwang95.github.io/assets/polynomial_roots.jpg">
  </a>
</p>
<br>

This image is a density map of all the complex polynomial roots of four billion polynomials with coefficients in $$\{0, 1, -1\}$$. Roots which appear with multiplicity greater than one are plotted only once. It is super high resolution so explore it at your leisure.

The roots are generated using a small python script which was run on a Ryzen 1700 machine for four days. Source code can be found on [github](https://github.com/frankwang95/polynomials).
