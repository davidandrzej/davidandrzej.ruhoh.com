---
title: 'Machine learning in practice'
date: '2012-07-20'
tagline: 'D. Sculley et al (KDD 2011)'
categories: 'machine learning'
tags: [industrial, algorithms, engineering]

type: draft
---

By design, an academic machine learning paper tends to present a
proposed method or algorithm in isolation.  Problem context, data
preparation, and feature engineering are hopefully discussed to the
extent required for scientific reproducibility, but are rightly not
the primary focus.  Logistical and implementation details relevant to
the use of the proposed technique in an actual production system are
typically not mentioned whatsoever.

These aspects are typically left as machine learning hacker "folk
wisdom" to be picked up from colleagues,
[blog](http://techblog.netflix.com/2012/04/netflix-recommendations-beyond-5-stars.html)
[posts](http://techblog.netflix.com/2012/06/netflix-recommendations-beyond-5-stars.html),
[discussion boards](http://metaoptimize.com/qa/),
[snarky tweets](https://twitter.com/jaykreps/status/219977241839411200),
[open-source libraries](http://scikit-learn.org/), or more often than
not, first-hand experience.

A notable exception is an interesting paper from Google on detecting
"malicious" (ie, scam/spam) advertisements which won
[best industry paper](https://twitter.com/googleresearch/status/106105833393369088)
at KDD 2011:

> Detecting Adversarial Advertisements in the Wild
>
> D. Sculley, Matthew Otey, Michael Pohl, Bridget Spitznagel, 
>
> John Hainsworth, Yunkai Zhou
>
> http://research.google.com/pubs/archive/37195.pdf

On the face of it, this might appear to be a "Hello-World" machine
learning problem right out of a textbook: simply train a Naive Bayes
classifier on a set of bad ads versus a set of good ones.  This is
apparently far from being the case.  While the paper is understandably
shy about hard numbers, this is a business-critical problem for Google
and they mention several issues which make this an especially
challenging setting.

The paper describes an approach to this problem which consists of a
pragmatic blend of different techniques and tricks. Here I briefly
discuss some of the highlights, although I would encourage the reader
to check out the original
[paper](http://research.google.com/pubs/archive/37195.pdf) and
[presentation slides](http://www.eecs.tufts.edu/~dsculley/papers/Detecting_Adversarial_Advertisements.pdf).

## 1) Classification

The core machine learning technique is (unsurprisingly)
classification: is this ad OK to show to the user or not?  The
[first author](http://www.eecs.tufts.edu/~dsculley/) has released code
for the
[core machine learning algorithms](http://code.google.com/p/sofia-ml/)
involved.

#### Only auto-block or auto-allow on high-confidence predictions,

While there is additional work needed to properly calibrate and
quantify prediction uncertainty, in this application it is worthwhile
to have the automated system simply say "I don't know" when
appropriate and escalate the decision to a human.

#### Throw a ton of features at the model and let L1 sparsity figure it out 

Feature representation is a crucial machine learning design decision.
They cast a very wide net in terms of representing an ad including
words and topics used in the ad, links to and from the ad landing
page, information about the advertiser, and more.  Ultimately they
rely on strong
[L1 regularization](http://www.di.ens.fr/~fbach/cvpr_tutorial_2010_sparse_methods_ML.pdf)
to enforce sparsity and uncover a limited number of truly relevant
features.

#### Map features with the "hashing trick"

This is a very practical trick for handling high-dimensional feature
spaces by hashing features down to a lower dimensional space - see
[this](http://metaoptimize.com/qa/questions/6943/what-is-the-hashing-trick)
discussion board answer.

#### Handle class imbalance with ranking SVM instead of binary SVM

Highly imbalanced classes (here far more legit ads than scam ones) are
a classic supervised classification "gotcha".  There are different
ways to handle this, but here they achieve improved performance by
transforming the problem to _ranking_: all malicious ads should be
"ranked higher" than legitimate ones. While this pairwise ranking
constraint might seem intractable, they use a
[sampling technique](http://www.eecs.tufts.edu/~dsculley/papers/large-scale-rank.pdf)
to get around this.

#### Use a _cascade_ of classifiers

This trick is somewhat tailored to their specific problem, although
the design pattern may be generally applicable.  They address the
class imbalance issue and the different "kinds" of bad ads with two
stages of classification.  First, is the ad _Good_ or _Bad_?  If
_Bad_, is it _BadTypeA_ or not, is it _BadTypeB_ or not, and so on.

## 2) Scalability, engineering, and operations

A production machine learning system exists within an engineering and
business context that may not apply to experimental software written
solely for research papers.  This puts increased importance on
scalability, validation, reliability, and maintainability.

#### MapReduce: example pre-processing (map), SGD training (reduce)

Somewhat surprisingly, they find the scalability bottleneck to be
loading examples from disk and extracting features.  Therefore, they
perform that work in parallel Map jobs, and used a single Reduce job
to do Stochastic Gradient Descent (SGD) classifier training.  There
has been a fair amount of recent work on
[parallelizing SGD itself](http://research.cs.wisc.edu/hazy/papers/hogwild-nips.pdf).

#### Monitor all the things

To make sure the system "still works" as its inputs evolve over time,
they do extensive monitoring of key quantities and investigate further
if large changes are observed:

* precision/recall on held-aside test data
* input feature distributions
* output score distributions
* output classification distributions
* human-judged system quality

#### Rich model objects

In machine learning papers, a linear model often consists of nothing
more than its weight vector.  However, in practice they found it
useful from a software engineering standpoint to extend the "model
object" to include feature transformation, probability calibration,
training hyperparameters, as well as other information.

## 3) Human-in-the-loop

The business importance and general trickiness of the problem
necessitates the use of human judgment.

#### Make efficient use of expert effort 

They use
[active learning](http://www.cs.cmu.edu/~bsettles/pub/settles.activelearning.pdf)-esque
strategies to identify the highest "value" examples for humans experts
to manually.  This might include particularly difficult cases, or
novel examples representing a new trend. They also provide an
information retrieval-oriented user interface to help experts in
searching for new emerging threats.

#### Allow humans to hard-code rules 

Sometimes the Human Knows Best - they are not dogmatic about doing
absolutely everything in a fully automated machine learning setting,
instead allowing the experts to craft simple hard-coded rules when
appropriate.

#### Human evaluation 

They use multiple expert judgments on the same ads to calibrate their
confidence in both annotations and annotators.  There is a sizable
[body](http://www.stanford.edu/~jurafsky/amt.pdf) of
[research](http://stat.wharton.upenn.edu/~lzhao/papers/MyPublication/MultiExpert_ICML_2009.pdf)
on doing this kind of thing (eg, for
[Amazon Mechanical Turk](https://www.mturk.com/mturk/welcome)), see
the
[LingPipe blog](http://lingpipe-blog.com/category/data-annotation/)
for more references and discussion.

Finally, they also periodically use non-expert evaluations to make
sure the system is working well in the eyes of regular users.  Since
end-user satisfaction is the ultimate objective, actually measuring
this is a very sensible thing to do.
