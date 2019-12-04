---
layout: post
title: "Two Small Datasets for Semantic Entity Segmentation in Natural Language"
date: 2019-06-01
tags: data NLP
---

I recently wrote about a lightweight model that I used for segmentation of semantic entities in text. In our discussion, we described an application of this model to extracting references to monetary values. The data used to train our model I recently uploaded to [GitHub](https://github.com/borrowbot/data_model_repo) along with another dataset containing data for segmenting out references to dates. This post will serve as a reference to the interpreting these tiny ($$O(100)$$) datasets as provided.


# General Schema

Both datasets covered here consist of scraped submission titles from the [r/borrow](https://www.reddit.com/r/borrow/) subreddit, which is a community which enables the coordination of personal loans between redditors. Each of the scraped submission titles comes with a label marking the segments in the strings which contain the semantic entities of interest. We serialize our input/labels label pairs using [YAML](https://yaml.org/) which trade off storage efficiency for human readability and are delivered as a series of relatively small files. In all, each file might look something like this:

```yaml

-
    input: '[PAID] (u/verydisappointing) ($180 GBP + Int.) (EARLY)'
    labels:
        -
            - 31
            - 34
-
    input: '[REQ] Need $25 for food in Manchester, CT. Can pay back $30 in a week'
    labels:
        -
            - 12
            - 14
        -
            - 57
            - 59

```

As you can see each of the labels is given as a list of pairs of integers which mark the beginning and end of each of the semantic entities which we can extract as follows:

```python

>>> s = '[REQ] Need $25 for food in Manchester, CT. Can pay back $30 in a week.'
>>> labels = [[12, 14], [57, 59]]
>>>
>>> s[labels[0][0]:labels[0][1]]
'25'
>>> s[labels[0][0]:labels[0][1]]
'30'

```

It is worth noting that the scraped input titles in these datasets are not sampled uniformly from the pool of titles. Each input was chosen by hand from the complete set with the goal of selecting titles where the semantic entities which are somewhat non-typical. In the two sections below where we discuss details of each dataset, we discuss some examples of what sorts of expressions we consider non-typical alongside guidelines we use for defining ground truth labels in obscure cases.


# Monetary Reference Data

Our initial release of monetary reference labels consists of 80 labeled strings. References to money are labeled as sequences of consecutive numerical characters along with possibly decimal points (`.`) and commas (`,`). Detections do not include currency symbols that may accompany the amounts. Below, you can see a selection of example strings with the labels in bold:

<pre><code>

    [REQ] ($<b>150</b>) (#Akron, OH, USA)(payback $<b>60</b> on 1/4, 1/18, and 2/1)

    [REQ] (<b>200</b>) - (Dallas, TX, USA), (01/15/2018), (PayPal)

    [REQ] (<b>80</b>) - (Largo, FL, USA), (Repay $<b>100</b> on 10/12/2018), (PayPal)

    [REQ] (<b>1000.00</b>) - (#Saco, Maine, USA), (<b>1250.00</b> by 9/30/17), (Paypal)

    [REQ] (<b>3,000</b>) - (#Fort Dodge, Iowa, U.S.), (03/01/18), (PayPal)

</code></pre>

In selecting titles for this dataset, we looked for examples with a couple of particular consideration which seem to summarize most atypical cases.

1. Monetary references with and without currency markers, with markers of different kinds, and with markers in different positions.
2. Titles with references to different dates.
3. Titles with and without the presence of `.` and `,` characters.
4. A diverse selection of monetary amounts.

In practice we have actually found that our data is somewhat insufficient with respect to attributions (1) and (4) though the details of such a question is more appropriate in its own discussion.


# Date Reference Data

In addition to our monetary reference dataset, we are also releasing 151 labeled strings segmenting out date references. Each segment is intended to references a specific day of a calendar year (with or without the year specified) including any user included punctuation. References to days of the week (ie. `Monday`, `Tuesday` etc.) are not included but we do include labels of generic months without a day specified (ie. `Will repay in June`). Some examples follow:

<pre><code>

    (£350) - (#Runcorn, Cheshire, UK), (£600 returned in two parts <b>26th October</b> & <b>26th Novmeber</b>), (Paypal)

    [REQ] ($210) - (#santa clara, CA, USA), (<b>3/17</b>), (paypal repay $235)

    [REQ] ($400) - (#Hyderabad, Telangana, India), (<b>15 January</b>), (PayPal)

    REQ] ($500 ) - (#Manassas, Virginia, USA), (<b>Jan 1-2</b>), (Paypal/will pay back in crypto)

    [REQ] ($150) - (#Myrtle beach, Sc, USA), ($170 By<b>June 6th</b>), (Paypal/Pre-Arranged

    [REQ] ($120) - (#Little Rock, Ar, US), (Repayment Date <b>7-14-17</b> ), ( payback total $130)

    [REQ] (€200) - (#Dublin, Ireland), (<b>19th of May</b>), (PayPal)

</code></pre>

Data is divided into 12 files, each containing 10-15 labeled strings corresponding to months. Strings containing dates from multiple months are included in the file of the earliest relevant month referenced. In addition to finding examples from a broad collection of months as samples, we also made efforts to include a strong mixture of spelled out dates, numerical dates (of different formats), and abbreviations and alternative spellings of month names.
