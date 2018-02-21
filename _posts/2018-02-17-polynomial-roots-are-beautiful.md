---
layout: post
title: "Beautiful Fractal Patterns in the Roots of Polynomial Roots"
date: 2018-02-17
tags: polynomials root-solving fractals math
comments: true
---

Polynomials are a pretty common mathematical construct. On a field $$R$$, the polynomial ring $$R[x]$$ in invariant $$x$$ is the space of functions of the form:

  $$x \longmapsto \sum_{k=0}^n a_{k} x^k$$

for arbitrary integer $$n$$ and field elements $$a_k \in R$$. Besides being an algebraic ring, $$R[x]$$ has all sorts of interesting properties in abstract algebra. It is, for instance, commutative if $$R$$ is commutative and forms an integral domain if and only if $$R$$ has a field structure. None of that, however, will be the topic of this post. Today, we will focus on the roots of complex polynomials and more specifically, on how pretty they can look:

<br>
<p align="center" class="no-hov">
  <a href="https://frankwang95.github.io/assets/polynomial_roots_full.jpg">
  <img src="https://frankwang95.github.io/assets/polynomial_roots.jpg">
  </a>
</p>
<br>

This image is a density map of all the complex polynomial roots of all the polynomials of degree up to 15 with coefficients in $$\{0, 1, -1\}$$. It is super high resolution so explore it at your leisure.

The roots are generated using a Mathematica notebook and the image was generated as a PPM (forgive me) with Haskell. Both of these scripts are enormously inefficient, especially when if comes to memory usage. In part this is because they were written many years ago when I was still learning to code, but in addition to that, I have never quite gotten the hang of efficiently using memory with functional programing languages. Those who are interested can find the code on [github](https://github.com/frankwang95/polynomials).
