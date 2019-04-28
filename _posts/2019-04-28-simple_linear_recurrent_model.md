---
layout: post
title: "A Auto-Regressive Language Model for Recurrent Logistic Regression"
date: 2019-04-28
tags: recurrent NLP ARMA auto-regressive
---

One nice application of machine learning is in the use of exceptionally simple models in the place of complex, hard-coded heuristics. In natural language applications for instance, parsing semantic data is made difficult because expressions of the data often are nonstandard and contain errors. The rules needed to parse human generated text with high recall are oftentimes non-commutative and consequently difficult to reason about. Adding logic to cover missed cases often leads to unanticipated false positives.

In this post, I will discuss a character model that I have been using to extract references to payment amounts in natural text. The intention here is to give an example application of small-scale machine learning as an alternative to hard-coded logic. An example application of this model is seen below:

<img src="https://github.com/borrowbot/simple_state_recurrent_model/raw/master/readme_resources/example_inference.png">

We face a number of challenges in identifying payment in natural text. The formatting of the amount can take many different forms. For instance, the amounts may or may not include a currency symbol or abbreviation (`$400`, `400`, `400€`, or `400cad`) and some people include comma (or sometimes period) separators (`1650`, `$1,650`, or `£1.650,00`) into their numbers. Furthermore, the problem is made more difficult because numerical characters often refer to things other than payment amounts such as dates, which themselves can be expressed in a variety of ways.

Our model handles these cases with almost no explicit design. It was trained using less than 100 hand labelled examples in about a minute and contains less than 500 parameters, consuming 3.7kB when serialized. Inference is incredibly fast and parsing performance is exceptional, achieving an accuracy rate in excess of 95%.

For those only interested in making use of this, the source code for this project can be found [here](https://github.com/borrowbot/simple_state_recurrent_model) along with pre-trained models for parsing payment amounts (models for parsing date entities are coming soon). It is worth noting that the data used to train these models follows a distribution that may not match the data for your use case exactly and so model performance for you may be lower than the numbers reported here. I will be releasing the training data for this project in the near future and will be researching improvements to the generality of these models. Check back here later for more updates or feel free to reach out to me if you are interested in contributing yourself.

The remaining sections below are for those interested in a more detailed discussion of our work. We give a mathematical formulation of the model along with some of its expected mathematical properties before discussing its evaluation and performance. Finally, we close with a discussion on the interpretability of the learned weights in a trained model.


# Mathematical Description

One can describe our model to be a linear, recurrent model with a simple auto-regressive state. Alternatively, for those who have studied time series methods, our model can be thought of as a $$(0,1)$$ normalized ARMA model in which the moving average noise pattern takes the form of non-independent bernoulli events instead of i.i.d. gaussians.

We start by taking a fixed, finite alphabet $$\mathcal{A}$$ of characters of size $$N_\mathcal{A}$$. A string of text takes the form of a sequence of characters $$s = c_1, c_2, \ldots$$ drawn from $$\mathcal{A}$$. We pose a binary classification model $$\sigma(s, i)$$ which predicts whether to consider the character at some index $$i$$ to be a part of a payment amount.

The mathematical expression for our model is given by:

$$
\sigma(s, i) = \text{logit}^{-1}\left(\zeta \sigma(s, i - 1) + \sum_{i=0}^{W N_\mathcal{A}} \phi_i v_i + b\right)
$$

where $$\sigma(s, i - 1)$$ is the output of our model on the previous character, $$v \in \mathbb{R}^{W N_\mathcal{A}}$$ is a vector encoding of the characters in a size $$W$$ window about character $$i$$,and $$\zeta$$, $$b$$, and $$\phi$$ are learnable parameters. The inverse logit function is given by the formula seen below, and is applied to normalize the model's output to the range $$(0, 1)$$.

$$
\text{logit}^{-1}(x) = \frac{\exp(x)}{1 + \exp(x)}
$$

The diagram below summarizes what we've described here:

<img style="max-width: 1200px; margin: 0 0 0 -250px;" src="https://raw.githubusercontent.com/borrowbot/simple_state_recurrent_model/master/readme_resources/model_diagram.png">

It is worth noting that the model described here has considerable similarities to an ARMA($$1$$, $$WN_\mathcal{A}$$) time series model. This similarity suggests to us some interesting training strategies based on the Yule-Walker equations though we will save that discussion for another day.


## Training

Our model is trained by maximizing a naive form of likelihood using stochastic gradient ascent. This claim is based on a derivation similar to the one found [here](https://frankwang95.github.io/2018/03/interpreting-cross-entropy) for cross entropy loss. This is equivalent to using SGD to minimize the following loss:

$$
\mathcal{L}(\sigma, \mathcal{D}) = -\sum_{s, l \in \mathcal{D}} \sum_{i=1}^{|s|} l_i \sigma(s, i) + (1 - l_i) (1 - \sigma(s, i))
$$

where $$\mathcal{D}$$ is a set of training data consisting of strings $$s$$ and character-level labels $$l$$. We have found in practice that training in this way leads to working models. However, this training method lacks theoretical elegance because its derivation makes use of two fallacious, simplifying assumptions.

Firstly, this loss-function cannot be said to be strictly derived from the principal of likelihood maximization because the expression for likelihood depends on an assumption of independence between every sample. Though it may be reasonable to make independence assumptions about distinct strings, it is clear that two training samples derived from the same string (and in-particular if they contain overlapping character windows) are highly mutually dependent.

Furthermore, we make a further major simplification in training by removing the recurrent dependency of $$\sigma(s, i)$$ on $$\sigma(s, i - 1)$$. More specifically, during training time, we replace the $$\sigma(s, i - 1)$$ term in $$\sigma(s, i)$$ by a simple `1` or `0` taken from the ground truth. This helps us avoid the recursive dependence of $$\mathcal{L}$$ on the parameters of $$\sigma$$ and makes computing the loss-gradient significantly simpler. Furthermore, this simplification reduces our problem to fitting a linear logistic regression which is a convex optimization problem. We are therefore endowed with theoretical guarantees that gradient based methods will find the unique global minima of $$\mathcal{L}$$, if it exists (this depends on the rank of the data matrix).

That being said, These issues, I feel are tractable and I hope to have a more elegant expression for model training to share in the near future. It seems plausible to me that our training expression converges to a true likelihood maximization loss as the number of distinct strings becomes large relative to the average length of each string. Given a language model to express the dependence between characters in a string, it may also be possible to compute an expression for likelihood over an entire string in unison.




<!-- # Performance

We are working on a careful evaluation of this model - please check back  -->

<!--
# Performance

For our use case, this model was trained on a dataset of 80 labeled strings (though each string generates more than one training example). The model weights were zero-initialized (randomness was not needed for weight initialization in this class of model as discussed above) and regular, old stochastic gradient descent was used to train the model.

Using sensible defaults, training the model is very fast - we were able to


# Interpretation
-->
