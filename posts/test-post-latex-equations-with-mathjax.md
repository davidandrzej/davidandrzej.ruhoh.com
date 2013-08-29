---
title: TEST POST - LaTeX equations with MathJax
date: '2012-08-19'
description: Testing MathJax
categories: test
tags: [software, math]
---
{{{ widgets.mathjax.mathjax }}}

It is very useful to have nicely formatted equations in blog posts.
The test example below uses the [MathJax](http://www.mathjax.org/)
JavaScript library, with a ruhoh configuration based on a
[GitHub comment](https://github.com/ruhoh/ruhoh.rb/issues/46#issuecomment-6988308)
from [Ramnath Vaidyanathan](https://github.com/ramnathv).

### Loss functions and empirical risk

<div> 

Given a dataset of observed input-output pairs $(x_1, y_1), \ldots,
(x_N, y_N)$, we can estimate a function $f(x)$ to predict future
values of $y$ given $x$.  We measure the difference between the true
$y$ and our prediction $f(x)$ with a loss function, such as $\ell(y,
f(x)) = (y-f(x))^2$.  We can then compute the empirical risk $R$ of
our function $f$ with respect to loss function $\ell$ over a dataset as:

$$ R(\ell,f) = \frac{1}{N} \sum_{i=1}^{N} (y_i - f(x_i))^2 $$

</div>

