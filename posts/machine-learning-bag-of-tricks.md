---
title: 'Practical machine learning tricks from the KDD 2011 best industry paper'
date: '2012-07-26'
tagline: '"Detecting Adversarial Advertisements in the Wild" by D. Sculley et al'
categories: 'machine learning'
tags: [industrial, algorithms, engineering]
---

A machine learning research paper tends to present a newly proposed
method or algorithm in relative isolation.  Problem context, data
preparation, and feature engineering are hopefully discussed to the
extent required for reader understanding and scientific
reproducibility, but are usually not the primary focus.  Given the
goals and constraints of the format, this can be seen as a reasonable
trade-off: the authors opt to spend scarce "ink" on only the most
essential (often abstract) ideas.

As a consequence, implementation details relevant to the use of the
proposed technique in an actual production system are often not
mentioned whatsoever. This aspect of machine learning is often left as
"folk wisdom" to be picked up from colleagues,
[blog](http://techblog.netflix.com/2012/04/netflix-recommendations-beyond-5-stars.html)
[posts](http://techblog.netflix.com/2012/06/netflix-recommendations-beyond-5-stars.html),
[discussion boards](http://metaoptimize.com/qa/),
[snarky tweets](https://twitter.com/jaykreps/status/219977241839411200),
[open-source libraries](http://scikit-learn.org/), or more often than
not, first-hand experience.

Papers from conference "industry tracks" often deviate from this
template, yielding valuable insights about what it takes to make
machine learning effective in practice.  This paper from Google on
detecting "malicious" (ie, scam/spam) advertisements won
[best industry paper](https://twitter.com/googleresearch/status/106105833393369088)
at [KDD 2011](http://www.sigkdd.org/kdd2011/) and is a particularly
interesting example.

> Detecting Adversarial Advertisements in the Wild
>
> D. Sculley, Matthew Otey, Michael Pohl, Bridget Spitznagel, 
>
> John Hainsworth, Yunkai Zhou
>
> http://research.google.com/pubs/archive/37195.pdf

At first glance, this might appear to be a "Hello-World" machine
learning problem straight out of a textbook or tutorial: we simply
train a Naive Bayes on a set of bad ads versus a set of good ones.
However this is apparently **far** from being the case - while Google
is understandably shy about hard numbers, the paper mentions several
issues which make this especially challenging and notes that this is a
business-critical problem for Google.

The paper describes an impressive and pragmatic blend of different
techniques and tricks. I've briefly described some of the highlights,
but I would certainly encourage the interested reader to check out the
original [paper](http://research.google.com/pubs/archive/37195.pdf)
and
[presentation slides](http://www.eecs.tufts.edu/~dsculley/papers/Detecting_Adversarial_Advertisements.pdf).

## 1) Classification

The core machine learning technique is (unsurprisingly)
classification: is this ad OK to show to the user or not?
[Code](http://code.google.com/p/sofia-ml/) for the some of the core
machine learning algorithms involved is available.

#### ABE: Always Be Ensemble-ing

Like the
[winning submission to the Netflix Prize](http://www.netflixprize.com/assets/GrandPrize2009_BPC_BellKor.pdf),
[Microsoft Kinect](http://research.microsoft.com/pubs/145347/BodyPartRecognition.pdf),
and [IBM Watson](http://www.aaai.org/Magazine/Watson/watson.php), the
proposed system uses an **ensemble** approach where the outputs of
many different models are combined to yield a final prediction.  This
technique constitutes the closest thing to a "free lunch" in modern
machine learning; if raw prediction accuracy is the goal, the use of
ensembles should at least be considered.

#### Only auto-block or auto-allow on high-confidence predictions,

While there is additional work needed to properly calibrate and
quantify prediction uncertainty, in this application it is worthwhile
to enable the automated system to simply say "I don't know" when
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
spaces by hashing features down to a lower dimensional space -
[this answer](http://metaoptimize.com/qa/questions/6943/what-is-the-hashing-trick)
on the MetaOptimize discussion board gives a nice explanation with
references.

#### Handle the class imbalance problem with _ranking_

Highly imbalanced classes (here there are far more legitimate ads than
scam ones) are a classic supervised classification "gotcha".  There
are different ways to handle this, but here they achieve improved
performance by transforming the problem to _ranking_: all malicious
ads should be "ranked higher" than legitimate ones.

#### Use a _cascade_ of classifiers

Besides class imbalance, the task is also complicated by the existence
of different "kinds" of bad ads (ads that direct you to malware, ads
for counterfeit goods, etc). They simultaneously address both issues
by using two-stage classification.  First, is the ad _Good_ or _Bad_?
Second, if the ad is _Bad_, is it _BadTypeA_ or not, is it _BadTypeB_
or not, and so on.

## 2) Scalability, engineering, and operations

Unlike experimental software written primarily for a research paper, a
production machine learning system exists within an engineering and
business context.  This puts increased importance on scalability,
validation, reliability, and maintainability.

#### MapReduce: pre-processing (map), algorithm training (reduce)

Somewhat surprisingly, they find the scalability bottleneck to be
loading examples from disk and extracting features.  Therefore, they
perform that work in parallel Map jobs, and used a single Reduce job
to do Stochastic Gradient Descent (SGD) classifier training.

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

In machine learning papers, a predictive model is often boiled down to
its mathematical essence - nothing more than a vector of learned
weights.  However, in software engineering practice the authors find
it useful to extend the "model object" to include feature
transformation, probability calibration, training hyperparameters, as
well as other information.

## 3) Human-in-the-loop

The business importance and general trickiness of the problem
necessitates the use of human experts as an integral part of the
overall solution.

#### Make efficient use of expert effort 

They use
[active learning](http://www.cs.cmu.edu/~bsettles/pub/settles.activelearning.pdf)-esque
strategies to identify the highest "value" examples for humans experts
to manually label (eg, ambiguous or particularly difficult cases).
They also provide an information retrieval-based user interface to
help experts in searching for new emerging threats.

#### Allow humans to hard-code rules 

Sometimes "the human knows best" - they are not dogmatic about doing
absolutely everything with fully automated machine learning methods.
Instead, they allow the experts to craft simple hard-coded rules when
appropriate.

#### Human evaluation 

Even expert judgments cannot be interpreted as the Fixed and Absolute
Truth.  Expert-provided labels may differ due to human error, varying
interpretations of label categories, or simple differences of opinion.
To adjust for this uncertainty, they use multiple expert judgments _on
the same ads_ to calibrate confidences in both annotations and
annotators.  There is a sizable
[body](http://www.stanford.edu/~jurafsky/amt.pdf) of
[research](http://stat.wharton.upenn.edu/~lzhao/papers/MyPublication/MultiExpert_ICML_2009.pdf)
on how to do this (eg, for
[Amazon Mechanical Turk](https://www.mturk.com/mturk/welcome) tasks).
See the
[LingPipe blog](http://lingpipe-blog.com/category/data-annotation/)
for more references and discussion.

Finally, they also periodically use non-expert evaluations to make
sure the system is working well in the eyes of regular users.  Since
end-user satisfaction is (presumably!) the ultimate objective,
actually measuring this is probably a very good idea.

