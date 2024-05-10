---
layout: post
title: "An Analogy to the Adjoint Method for Invertible Discrete NNs"
date: 2024-02-19
tags: autodiff gradients machine-learning
---

We've discussed Neural ODEs in some depth in the past. One of the benefits of NODEs is constant memory scaling as a function of network depth which is made possible by calculating the gradient via the [adjoint method](). This comes with a limitation that the width of the network must remain constant.

We also spent some time discussing the tradeoffs of different forms of [jacobian accumulation](https://frankwang95.github.io/2020/05/fwd-vs-reverse-mode-autodiff). One thing that stands out from our discussion is that when taking the jacobian of a sequence of functions, if all our functions are consistent in the dimensionality of their inputs and outputs, then all jacobian accumulations require the same number of FLOPs to compute. Thus we can take Jacobians with constant memory without needing to make computational tradeoffs. 

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

It's important to remark that this methodology does not make much practical sense in many contexts. If we focus our attention on fully connected MLPs with single sample mini-batches and no bias terms, the cache size may be negligible relative to the size of the model because the cached intermediate values of the model are a quadratic factor of the parameter count. For instance, a fully connected layer of width 64 has 4096 parameters so avoiding caching for backpropagation represents only a 1.6% memory savings. This is reduced to 0.4% when we consider layers with width 256. 

Naturally, this improvement scales linearly with batch sizes so avoiding the backpropagation cache may allow us to use larger batch sizes without the need for gradient accumulation.

This becomes substantially more pronounced inside of other neural network architectures. Convolutional layers typically have much fewer parameters than the input/output sizes. A 1D convolutional layer with kernel shape 8x64x64 on a 256x64 sized input would allow us to train a 50% larger model on the same hardware even with single sample mini-batches. If we are using 2D convolutional layers with 8x8x64x64 kernels and 256x256x64 inputs would allow us to train models that are 1700% larger.

We can also estimate the computational cost of these memory savings. For a model consisting of $$d$$ fully connected layers of width $$w$$, the relevant FLOP costs can be roughly broken into 3 parts:

1. $$2w^3 d$$ FLOPS for matrix multiplication in the forwards pass.
2. $$2 (w^2 d)^2 d = 2 w^4 d^3$$ FLOPS for the backwards jacobiaan accumulations. The $$w^2 d$$ term represents the total number of parameters in our model.
3. $$\frac{8}{3} n^3 d$$ FLOPS for matrix inversions with an additional $$2 w^3 d$$ flops to apply the invertions to activations in the backwards pass if we choose to recover activations by inverting our model versus with caching.

These calculations ignore the final loss layer, activations, and biases. We also note that compared to our theoretical analysis above, we are differentiate with respect to the model parameters as opposed to its inputs. The former is typically much larger and this has the effect that (2) comes to dominate the FLOP cost in the decompositions above. Therefore, the compuational overhead of invertible networks becomes neglegable for very large models. The equivalent calculation for convolutional models I think would be similar but smaller. I have not taken the time to work out the precise cost to invert convolutional layers but if they are similar to the inverse fully connected layers, we would expect the computation in (3) to be less dominiant for convolutional layers because of the relatively smaller number of parameters.

Finally, its worth remarking that (3) can likely be optimized slightly by solving for $$x_{n-1}$$ in $$x_n = W x_{n-1}$$ directly using LU decomposition as argued [here](https://gregorygundersen.com/blog/2020/12/09/matrix-inversion). I am also not totally certain that the jacobian accumulation described in (2) cannot be optimized by taking advantage of the fact that most of our model parameters are not used directly in any given layer.

# Next Steps

We will be putting forwards an implementation that validates some of the back-of-the-envelope caclulations we make here. We will write also about inverting convolutional models where we expect to see the greatest memory savings.

Update (May 10, 2024): It appears that most of these ideas are reflected already in [Gomez et al, 2017](https://arxiv.org/abs/1707.04585).
