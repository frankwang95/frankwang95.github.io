---
layout: post
title: "An Analogy to the Adjoint Method for Invertible Discrete NNs"
date: 2024-02-19
tags: autodiff gradients machine-learning
---

We've discussed Neural ODEs in some depth in the past. One of the benefits of NODEs is constant memory scaling as a function of network depth which is made possible by calculating the gradient via the [adjoint method](). This comes with a limitation that the width of the network must remain constant.

We also spent some time discussing the tradeoffs of different forms of [jacobian accumulation](). One thing that stands out from our discussion is that when taking the jacobian of a sequence of functions, if all our functions are consistent in the dimensionality of their inputs and outputs, then all jacobian accumulations require the same number of FLOPs to compute. Thus we can take Jacobians with constant memory without needing to make computational tradeoffs. 

I thought there was some potential that these concepts were related. In particular, under the constraint that each layer of a network preserved dimensionality, we could find an accumulation that allows us to take the gradient with memory constant in the network depth and at the same computational cost as backpropagation. Any cursory analysis of a small example shows this not to be the case. Imposing the constraints on the intermediate dimensionality of a neural network does not enable better scaling of memory through different Jacobian accumulations.

# Fixed Widths Enable Invertibility

That said, in a neural network of fixed width, using an invertible activation makes the entire network invertible. This admits an algorithm for computing a scalar gradient that uses constant memory using roughly double the compute of backpropagation. That is, we can scale memory as a constant instead of a linear factor of network depth while retaining linear computational complexity.

To continue to use the notation from past work, consider a sequence of functions $$f_k: \mathbb{R}^n \rightarrow \mathbb{R}^n$$ for $$k = 1, 2, \ldots D$$ which we compose cumulatively to obtain another sequence $$F_k:\mathbb{R}^n \rightarrow \mathbb{R}^n$$ as follows:

$$F_k(x) = f_{k} \circ \ldots f_2 \circ f_1 (x)$$

Given a scalar-valued loss function $$L$$, we can compute the gradient $$D(L \circ F_D) \bigg\vert_x$$ by a reverse-mode derivative without caching, efficiently recovering each intermediate $$F_k(x)$$ as $$f_{k + 1}^{-1} \circ F_{k + 1} (x)$$:

```
Inputs:
  composed functions : fs = [f1, ..., fk] (each invertible with a function inv)
  derivatives        : dfs = [Df1, ..., Df_k]
  loss function      : L
  loss derivitive    : dL
  input point        : x

aFi = x
for i in range(k):
  aFi = fs[i](aFi)

dFi = dL(aFi)
for i in range(k-1, -1, -1):
    aFi = inv(fs[i])(x)
    dFi = dfs[i](aFi) * dFi

return dFi
```


Note that this performs the same jacobian accumulation as backpropagation and so we retain its inexpensive computation cost. However, since we can recover each activation in the reverse pass, we don't incur the linear memory cost of caching results of the forward pass. With typical neural network layers, it's worth noting that we do need to invert a matrix to compute each layer's inverse. This has a cost comparable (though slightly higher) to the forward pass of the network. Both are cubic so our asymptotic performance remains unchanged.

# Comparison to Adjoint Differentiation of ODEs (WIP)

# Back of the Envelope Calculations

We can do a very quick back-of-the envelope calculation to show two main things. First, that there is only an incremental increase in the number of FLOPS needed compute a gradient and second, that the memory saved represents a meaningful amount in the context of the larger model.

To do this, we use the model we describe above treating each $$f_k$$ as a fully connected layer ignoring bias terms and activations. $$L$$ we treat as a simple $$n \times 1$$ matrix used to reduce the scalarize the intermediate layers. Note that previously, we don't give separate treatment to parameters and input dimensions but this is very important for our analysis as the parameter space is typically at least a quadratic factor larger than the input space. If we need only to calculate jacobians with respect to inputs, even forwards pass differentiation has only a factor of 2 FLOP cost over backpropogation, but when taking jacobians wrt. parameters, this difference expands to several order of magnitudes. With this in mind, we assign the parameter count to be $$p$$ distinguished from the size of the input space $$n$$.

Forwards mode differentiation of our model takes 

Using naive methods, multiplication of $$n \times n$$ matrices costs roughly $$2n^3 - n^2$$ FLOPS. Inverting it costs roughly $$$$ 
