---
title: 'Notes from BayLearn 2012'
date: '2012-09-08'
tagline: 'Bay Area Machine Learning Symposium'
description: ''
categories: 'machine learning'
tags: [conferences, industrial, research]
---

One of the benefits of
[working in the San Francisco Bay Area](http://www.sumologic.com/company/careers/)
is access to tons of interesting tech talks and other related
events. Last week I attended the
[Bay Area Machine Learning Symposium](http://www.baylearn.org/), which
was graciously hosted by Google.  Below are some brief notes on some
of the talks (any errors or misunderstandings are of course mine).
From my perspective, an interesting theme that emerged was the
"asynchronous update" design strategy for large-scale distributed
machine learning.

## Keynotes

The [keynote talks](http://www.baylearn.org/keynotes) were excellent,
and represented very different branches of work in modern machine
learning.

### Convex Optimization: From embedded real-time to large-scale distributed (Stephen Boyd, Stanford)

**Big idea:** Convex optimization is a general and powerful tool: many
real-world applications admit formulation as convex optimization
problems, and the theory/practice/technology for solving these is
quite well-developed.

**Current research:** This talk suggested a loose categorization for
thinking about the scale of different convex optimization problems:

* real-time embedded - we want to solve a small-ish problem many times
  per second using very few computational resources.  This might come
  up in the context of industrial or mechanical control (eg, the head
  of a disk drive).
 
* medium-scale - we want to solve a medium-ish problem on our desktop
  or laptop (Prof. Boyd contends that this is more or less a solved
  problem).
  
* large-scale distributed - we want to solve some massive problem
  across a huge network of machines (eg, Google).
  
Research directions included adapting to the "high" and "low" ends of
the scale via distributed algorithms and code generation tools,
respectively.

**Other resources:** Prof. Boyd is the co-author (along with
[Lieven Vandenberghe](http://www.ee.ucla.edu/~vandenbe/)) of the
definitive textbook _Convex Optimization_, which is
[available as a free PDF download](http://www.stanford.edu/~boyd/cvxbook/).

Convex optimization also plays a key role in different research areas
such as
[Compressive Sensing](http://people.ee.duke.edu/~willett/SSP//Tutorials/ssp07-cs-tutorial.pdf)
(CS), which allows you to "beat" the
[Nyquist-Shannon sampling theorem](http://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem),
and
[low-rank matrix completion](http://www-stat.stanford.edu/~candes/papers/MatrixCompletion.pdf),
which is one way to mathematically formulate the well-known
[Netflix Challenge](http://www.netflixprize.com/). The
[repository](http://dsp.rice.edu/cs) maintained by Rice University and
the
[Nuit Blanche](http://nuit-blanche.blogspot.com/p/teaching-compressed-sensing.html)
blog are great resources for learning more about these fields.

### Machine learning and AI via large scale brain simulations (Andrew Ng, Stanford)

**Big idea:** For many applications of machine learning, the main
bottleneck is the human expertise and effort required to craft
suitable features for use as inputs to the algorithm.  _Deep learning_
techniques aim to automatically learn useful representations from the
data.  For example, deep learning algorithms applied to images often
"discover" a "vocabulary" of edge detectors and simple shapes.  The
use of these building blocks (versus "raw" pixel data) can then lead
to improved performance on vision tasks like object recognition.

**Current research:** A joint Stanford-Google team recently published
a
[paper](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/unsupervised_icml2012.pdf)
at [ICML 2012](http://icml.cc/2012/) where an unsupervised feature
learning system was trained on a large corpus of YouTube videos using
some of Google's considerable computational resources.  The system
learned to recognize human and (more amusingly) cat faces, a result
which garnered a good deal of
[press coverage](http://www.nytimes.com/2012/06/26/technology/in-a-big-network-of-computers-evidence-of-machine-learning.html?pagewanted=all).
One technical challenge involved scaling up their approach to a
large-scale distributed setting, which was accomplished using
asynchronous updates against a central "parameter server".

**Other resources:** The Stanford
[deep learning page](http://deeplearning.stanford.edu/) has links to
papers and an interesting
[deep learning tutorial](http://deeplearning.stanford.edu/wiki/index.php/UFLDL_Tutorial).
Prof. Ng had also given a talk on this subject at an ICML 2011
[workshop](http://icml2011speechvision.wordpress.com/program/), and
the
[slides](http://icml2011speechvision.files.wordpress.com/2011/06/visionaudio.pdf)
give a great background and context for the ICML 2012 paper.

Finally, if you are at all interested in this area of research, I
would _strongly_ recommend enrolling in the (free!) online
[Coursera](https://www.coursera.org/) class
[Neural Networks for Machine Learning](https://www.coursera.org/course/neuralnets),
taught by [Geoffrey Hinton](http://www.cs.toronto.edu/~hinton/).


### Modeling User Interests (Alex Smola, Google)

**Big idea:** [Probabilistic modeling](http://www.cs.princeton.edu/courses/archive/fall10/cos513/)
provides a useful framework for developing rich models of user
interests and behavior.  Here "rich" means models that attempt to
capture more realistic details, or equivalently, to make fewer
simplifying assumptions. For example, we can explicitly model the fact
that a user's interests are not static, but rather evolve over time.
Given enough data, these higher-fidelity models can yield significant
gains in prediction and recommendation performance.

**Current research:** These models often include many _latent variables_
representing unobserved quantities such as the underlying "topics" of
a given news article, or the mix of "interests" associated with a
particular user.  This usually leaves us with two related challenges
when doing inference:

1. searching the large state space requires lots of computational resources
2. latent variables introduce "coupling" over the dataset, complicating distributed approaches to problem #1

Besides this, there are very interesting design questions about which
aspects of reality to capture in the model, and which to ignore.

**Other resources:** In the talk Prof. Smola alluded to a
[scalable distributed implementation](http://blog.smola.org/post/6359713161/speeding-up-latent-dirichlet-allocation),
of the
[Latent Dirichlet Allocation (LDA)](http://jmlr.csail.mit.edu/papers/volume3/blei03a/blei03a.pdf)
topic model that was developed while he was at Yahoo.  Interestingly,
the organization of this system was similar to the one later used by
Prof Ng. and colleagues, using asynchronous updates against a global
"parameter server".

As it turns out, Coursera is _also_ offering a free online course in
[Probabilistic Graphical Models](https://www.coursera.org/course/pgm),
taught by [Daphne Koller](http://ai.stanford.edu/~koller/).

### Other highlights

* [Guaranteed Sparse Optimization on the Probability Simplex via Convex Programming](https://docs.google.com/file/d/0Bxf-khCt_eknNjZxRi11NktQcnc/edit?pli=1)
(Laurent El Ghaoui, UC Berkeley; Mert Pilanci, UC Berkeley; Venkat
Chandrasekaran, Caltech) - This paper addresses the problem of
_cardinality_-penalized optimization of a convex function (ie, the L0
norm) over the probability simplex. While penalizing the L1 norm can
often encourage sparsity, this doesn't work here because the L1 norm
is 1 for all feasible solutions (by the definition of the simplex).

* [Fixed-Form Variational Posterior Approximation Through Stochastic Linear Regression](https://docs.google.com/file/d/0Bxf-khCt_eknM1ZmMzJlUHhZR3M/edit)
(Tim Salimans, Erasmus U. Rotterdam; David Knowles, Stanford
University) - This paper presents an interesting general technique
for variational inference. It is also available as a (much)
[longer arXiv version](http://arxiv.org/pdf/1206.6679v2.pdf).
