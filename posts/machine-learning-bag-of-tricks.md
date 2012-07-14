---
title: 'Production machine learning tricks'
date: '2012-07-09'
description:
categories:
tags: []

type: draft
---

By design, an academic machine learning paper tends to present a
proposed method or algorithm in isolation.  Problem context, data
preparation, and feature engineering are (hopefully) discussed to the
extent required for scientific reproducibility, but are rightly not
the primary focus.  Logistical and implementation details that might
be relevant to the use of the proposed technique in a live production
system are typically not mentioned whatsoever.

These aspects are typically left as machine learning hacker "folk
wisdom" to be picked up from colleagues,
[blog](http://techblog.netflix.com/2012/04/netflix-recommendations-beyond-5-stars.html)
[posts](http://techblog.netflix.com/2012/06/netflix-recommendations-beyond-5-stars.html),
[discussion boards](http://metaoptimize.com/qa/), [snarky
tweets](https://twitter.com/jaykreps/status/219977241839411200),
[open-source libraries] (http://scikit-learn.org/), or
unfortunately more often than not the hard school of experience.

One notable exception is an intersting KDD 2011 paper from Google on
detecting "malicious" (ie, scam/spam) advertisements:

> Detecting Adversarial Advertisements in the Wild
>
> D. Sculley, Matthew Otey, Michael Pohl, Bridget Spitznagel, 
>
> John Hainsworth, Yunkai Zhou
>
> http://research.google.com/pubs/archive/37195.pdf

On the face of it, one might think of this as a "Hello-World" machine
learning problem right out of a textbook: train a Naive Bayes
classifier on a bunch of bad ads and a bunch of legit ones and you're
done.  Interestingly, this is apparently *not* the case.  While the
paper is understandably shy about hard numbers, this is a
business-critical problem for Google and they mention several
issues which make this an especially challenging setting.

The paper goes on to describe their approach to this problem which,
rather than being a single unified model or algorithm, consists of a
pragmatic blend of different techniques and tricks including bringing
humans into the loop.  These tricks include previous research paper
results, software engineering and operations best practices, and
domain-specific customizations.  While recently revisiting this paper,
I took notes on some of these tricks from the paper and I thought I'd
share some of them here.


### Classification

The core machine learning technique is (unsurprisingly)
classification; here are a few techniques used.

#### Only auto-block or auto-allow on high-confidence predictions,

While there is some additional work needed to properly calibrate and
quantify prediction uncertainty, in this application it is worthwhile
to have the automated system simply say "I don't know" when
appropriate and escalate to a human.

#### Throw a ton of features at the model and let L1 sparsity figure it out 

Feature representation is a crucial machine learning design
decision.  Here they cast a very wide net in terms of representing
an ad including words and topics used in the ad, links to and from
the ad landing page, information about the advertiser, and more.
Ultimately they rely on strong L1 regularization to enforce
sparsity and uncover a limited number of truly relevant features.

#### Use the "hashing trick" on the features

This is a clever and very practical trick for handling
high-dimensional feature spaces -
[this](http://metaoptimize.com/qa/questions/6943/what-is-the-hashing-trick)
is a nice writeup.

#### Use ranking SVM instead of binary SVM to cope with class imbalance

While the pairwise ranking SVM might seem intractable, they use a
[sampling technique](http://www.eecs.tufts.edu/~dsculley/papers/large-scale-rank.pdf)
to get around this.

#### Use a _cascade_ of classifiers

This trick is somewhat tailored to their specific problem, but the
general design pattern is interesting.  They address the class
imbalance issue and the different "kinds" of bad ads with two stages
of classification.  First, is the ad good or bad?  If bad, is the ad
Bad Type A or not, is it Bad Type B or not, and so on.

### Scalability, Engineering, and Operations

A production machine learning system exists within an engineering and
business context that may not apply to experimental software written
solely for research papers, putting increased importance on
validation, reliability, and maintainability.

#### Map example pre-processing, reduce SGD training

Somewhat surprisingly, they found the scalability bottleneck to be
loading examples from disk and extracting features.  Therefore, they
performed that work in parallel Map jobs, and used a single Reduce job
to do Stochastic Gradient Descent classifier training.  It is worth
noting that there is a fair amount of recent work on
[parallelizing SGD](http://research.cs.wisc.edu/hazy/papers/hogwild-nips.pdf)
which might be relevant.

#### Monitor all the things

To make sure the system "still works" as its inputs evolve over time,
they do extensive monitoring:
* precision/recall on held-aside test data
* input feature distributions
* output score distributions
* output classification distributions
* human-judged system quality

#### Rich model objects

In machine learning papers, a linear model often consists of nothing
more than its weight vector.  However, they found it useful to extend
the concept of a _model_ to include feature transformation,
probability calibration, training hyperparameters, as well as other
information.


### Human-in-the-loop

#### Make efficient use of expert effort 

This included using active learning -esque strategies for identifying
high-value examples for humans to classify, as well as a powerful UI
that allowed experts to interact with and guide the system.

#### Allow humans to hard-code rules 


