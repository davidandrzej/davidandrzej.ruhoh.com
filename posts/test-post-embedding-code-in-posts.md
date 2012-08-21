---
title: TEST POST - embedding code with gists and prettify
date: '2012-08-21'
description: Testing gist and prettify
categories: test
tags: [software, math]
---

It is also useful to have a simple way to display nicely formatted
code.  Our example snippet is a [Clojure](http://clojure.org/)
function for constructing a machine learning feature encoder, and we
display it by 

* embedding a [GitHub gist](https://gist.github.com/)
* using [Google Prettify](http://code.google.com/p/google-code-prettify/)

### GitHub gist embedding
<script src="https://gist.github.com/970144.js"> </script>

### Google Prettify
<pre>
;; Assumes examples are Clojure maps, 
;; for example from clojure.contrib.sql
;;
;; {:eyecolor "brown" :heightmeters 1.7 ...}
;; {:eyecolor "blue" :heightmeters 1.5 ...}
;;
;; (def eye-encoder (discrete-feature-encoder :eyecolor myexamples))
;;
(defn discrete-feature
  "Create binary 1-of-N encoding of a discrete feature"
  [featurekey allexamples]
  (let [uniqvals (->> allexamples (map featurekey) set vec)]
    {:values (map #(format "%s:%s" (str featurekey) %1) uniqvals)
     :encoder (fn [example]
                (map #(if (= %1 (featurekey example)) 1 0) uniqvals))}))
</pre>
