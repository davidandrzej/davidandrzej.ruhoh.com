---
title: 'Word count is a monoid homomorphism'
date: '2013-04-30'
tagline: 'and, who cares?'
categories: 'data'
tags: [algorithms, theory]

type: draft
---

### Why does Map-Reduce work?

Counting word frequencies in a collection of documents is the
["Hello World" of Hadoop](http://hadoop.apache.org/docs/r1.0.4/mapred_tutorial.html#Example%3A+WordCount+v1.0),
with good reason.  It is a not-too-contrived task whose **underlying
structure** is a natural fit for distributed computation.  In this
post we focus on better understanding that underlying structure using
some tools from abstract algebra. This approach has useful practical
consequences - I was originally motivated to further explore these
concepts by a discussion of the theoretical foundations of the
[Algebird](https://github.com/twitter/algebird) data processing
library during an interesting
[meetup talk](http://www.meetup.com/Bay-Area-Scala-Enthusiasts/events/105409962/)
on scalable and flexible machine learning.

Code fragments in this post use Scala, but hopefully they are short
and simple enough for non-Scala programmers to understand easily.

### String monoid under concatenation

Loosely speaking, a [monoid](http://en.wikipedia.org/wiki/Monoid) consists of 

* a _collection of objects_ (eg, strings)
* a _binary operation_ for combining these objects to yield a new object of the same type (eg, string concatenation)
* an _identity object_ whose combination leaves an object unchanged (eg, the empty string)

For the word count example, we can consider each document to be a
single string, and the complete corpus to be the (space-separated)
concatenation of all documents.

### Map[String,Int] monoid under key-wise addition

The output of word count is a mapping from words (strings) to their
frequencies in the corpus (integers). These maps naturally form
another monoid, where the binary operation is key-wise addition using
0 as the value for missing keys.  That is, we combine two maps by
summing the values for each word as one would naturally do when
combining counts:

<script src="https://gist.github.com/davidandrzej/5413350.js"></script>

It should be fairly clear that the identity element here is simply the
empty map. Note that it was straightforward to form this construction
in part because the integers are _themselves_ a monoid under addition
with identity element 0.

### wordCount: String => Map\[String,Int\] monoid homomorphism 

A
[monoid homomorphism](http://www.math.cornell.edu/~mec/2008-2009/Victor/part6.htm)
is a **structure-preserving** function from one monoid to another. To
state this more clearly, let's introduce a small amount of notation.

<div>

Let $s \in S$ be a string, with $s_1 + s_2$ representing string
concatenation. Then let $m \in M$ be a count map, with $m_1 \oplus m_2$
representing key-wise addition.

We then define our wordCount() function $f: S \rightarrow M$.  The
monoid homomorphism property is then given by

$$ f(s_1 + s_2) = f(s_1) \oplus f(s_2) $$

</div>

That is, in order for wordCount() to be a monoid homomorphism from
strings to count maps, we **must** get the same result from
concatenating the strings and counting their words or counting the
words of the individual strings and summing the count maps.  It is
obvious that a simple token-counting implementation of wordCount()
satisfies this property, assuming that our string concatenation does
not affect tokenization (eg, we add whitespace between pairs of
concatenated strings).

So there you have it - the innumerable "how to count words in Hadoop"
tutorials available on the web are in fact well-disguised
introductions to monoid homomorphisms. The homomorphism property is
the "secret sauce" that ensures a Map-Reduce style computation of word
frequencies gives the same result, no matter how we split up the
document corpus.

### So what?

While this all may seem a bit abstract (indeed, that is the point), I
would argue that there are practical benefits to thinking about data
processing in this way. We now have a nice characterization of the
**underlying structure** that makes a given aggregation well-suited
for distributed computation in a Map-Reduce (or similar) framework:

* original **source** data (eg, documents) is a monoid - we can sensibly break it up into pieces
* **target** aggregate (eg, count) is a monoid - we can sensibly combine its values
* aggregation **function** (eg, word counting) is a monoid
  homomorphism - it can be applied separately to pieces of the dataset
  and then combined without changing the final result

The generality of this perspective can be quite powerful. As mentioned
earlier, the [Algebird](https://github.com/twitter/algebird) library
is built around algebraic abstractions such as monoids. These
abstractions can be used to cleanly separate concerns in data
processing infrastructure: "plumbing" code can be written against an
abstract monoid "interface".

Another interesting observation is that a variety of
[probabilistic data structures](http://highlyscalable.wordpress.com/2012/05/01/probabilistic-structures-web-analytics-data-mining/)
can be defined as monoids, and in fact Algebird contains
implementations for several of them.  For example, this could allow
you to trade exactness for scalability by (very) easily substituting
an _approximate_ version of wordCount() into your data pipeline.

Basic familiarity with the vocabulary of abstract algebra can also be
helpful to take full advantage of libraries which leverage these
concepts, like Algebird and
[scalaz](https://github.com/scalaz/scalaz). 

Finally, I have always found it intrinsically beneficial to try to
understand things from multiple angles.  The algebraic formulation of
distributed data processing provides a slightly different and valuable
way of thinking about problems, solutions, and their properties in
this domain.
  
#### More resources

* "Scalable and flexible machine learning with Scala" - meetup talk
  ([slides](http://www.slideshare.net/VitalyGordon/scalable-and-flexible-machine-learning-with-scala-linkedin),
  [video](http://www.youtube.com/watch?v=VTn2-VZYpoQ))
* scala.collection.approximate - nescala 2013 talk
  ([video](http://www.youtube.com/watch?v=yzitqjUI6ok))
* Life After Monoids - nescala 2013 talk
  ([video](http://www.youtube.com/watch?v=xO9AoZNSOH4))
* Metrics on Words - blog post on string mathematics
  ([link](http://jeremykun.com/2011/12/19/metrics-on-words/))
* Monoids: Theme and Variations - Haskell example
  ([pdf](http://www.cis.upenn.edu/~byorgey/pub/monoid-pearl.pdf))
* MATH 240H Notes: Binary Operations, Monoids, Groups and Examples -
  abstract algebra notes
  ([pdf](http://www.math.rochester.edu/courses/240H/home/Groups.pdf))
