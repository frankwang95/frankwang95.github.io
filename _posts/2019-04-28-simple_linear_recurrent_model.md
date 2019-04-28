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

One can describe our model to be a linear, recurrent model with a simple auto-regressive state. Alternatively, for those who have studied time series methods, our model is identical to an ARMA model in which the moving average noise pattern takes the form of non-independent Bernoulli events instead of i.i.d. gaussians.

We start by taking a fixed finite alphabet $$\mathcal{A}$$ of characters of size $$N_\mathcal{A}$$. A string of text takes the form of a sequence of characters $$s = c_1, c_2, \ldots$$ drawn from $$\mathcal{A}$$. We pose a binary classification model $$\sigma(s, i)$$ which predicts whether to consider the character at some index $$i$$ to be a part of a payment amount.

To impose structure onto the parameterization of $$\sigma$$, we summarize the string $$s$$ locally about $$i$$ using a fixed size vector $$v(s, i) \in \mathbb{R}^D$$ which contains enough information for us to do successful classification. Depending on how much information you wish to include and how you choose to encode it, the size of $$D$$ can vary. However, it is important to keep in mind over-parameterization problems, especially if you are working with modest amounts of training data.


In our model, we construct $$v(s, i)$$ to include two principal signals. First, we include the character information in a relatively small window about $$i$$. Each character can be represented as a one-hot encoded vector in $$\mathbb{R}^{N_\mathcal{A}}$$ and we can simply concatenate these vectors for all $$W$$ characters in our window.

In addition to character data, we have chosen to include a single bit indicating the label of the previous character. This bit is what gives this model its recurrent/auto-regressive property. In our training data, this bit is generated from the ground truth as a `0` or a `1` but when computing inference it is populated by the model's output on the previous index, $$\sigma(s, i - 1)$$.

With these features, $$\sigma$$ is nothing more than a binary classification problem on $$N_\mathcal{A} W + 1$$ dimensional data. The diagram below summarizes what we've described here:

<img src="https://raw.githubusercontent.com/borrowbot/simple_state_recurrent_model/master/readme_resources/model_diagram.png">

<!-- ## Expressed as an ARIMA(1, $WN_{\mathcal{A}}$) model

## Training


# Performance

For our use case, this model was trained on a dataset of 80 labeled strings (though each string generates more than one training example). The model weights were zero-initialized (randomness was not needed for weight initialization in this class of model as discussed above) and regular, old stochastic gradient descent was used to train the model.

Using sensible defaults, training the model is very fast - we were able to


# Interpretation
-->
