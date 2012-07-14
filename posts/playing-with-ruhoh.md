---
title: 'Static blogging with ruhoh'
date: '2012-07-13'
description: 
categories: writing
tags: [software, organization, productivity]
---

This is an early experiment using [ruhoh](http://ruhoh.com) for
blogging.  Ruhoh is a **static** blog generation tool: given properly
formatted input content, ruhoh simply **generates** the fixed code for a
blog-style website.  This generated code **is** the website - you can
just serve it up, for example from
[Amazon S3](http://aws.typepad.com/aws/2011/02/host-your-static-website-on-amazon-s3.html).
This style of blog has somewhat recently become a flourishing
technological niche; probably the most famous specimen is
[Jekyll](https://github.com/mojombo/jekyll/) which is used to drive
[GitHub Pages](http://pages.github.com/).  I'll briefly describe the
appeal of this approach for my needs.

### Plaintext

The blog content and styling consists of nothing more than a specially
structured directory of plaintext files.  This means you can easily
write posts in the text editor of your choice, version control
everything, and publish from the command-line.  Pretty sweet, if
you're into that sort of thing.

### Simplicity

By definition, a static site has fewer "moving parts" than a
fully-loaded dynamic one.  This sacrifices some flexibility in terms
of commenting, plugins, and other nifty dynamic content.  However if
you simply want to write things down and put them on The Internet, the
loss of these features can be outweighed by the gain of not having to
deal with their complexity.

### Security 

Not to tempt fate, but there is less surface area for security
disaster when simply serving static files.  More feature-rich blogging
platforms require hyper-vigilance around patching and operations,
otherwise you can easily find yourself unwittingly advertising
counterfeit Russian pharmaceuticals.

### Why ruhoh and not X?

A
[chance tweet](http://twitter.com/chlalanne/status/220442853241925632)
pointed me at [Jekyll Bootstrap](http://jekyllbootstrap.com/).  It
looked interesting but its author made a compelling case for his new
project ruhoh.  In the end I opted to just go for it and try ruhoh
instead of spending too much time/effort exploring the vast space
options in this space - so far so good.
