---
id: 188
title: 'P03 - Find the nᵗʰ Element of a List'
date: 2014-01-16T01:40:01+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=188
permalink: /188/p03-find-nth-of-list/
dsq_thread_id:
  - 2131146488
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - scala
---
This problem is very nice because the Clojure and Scala implementations are practically the same. It allows to appreciate the different syntaxes better.

Below is [Peter&#8217;s solution](http://pbrc.blogspot.co.at/2014/01/99-clojure-problems-3-find-kth-element.html) to the problem, in Clojure:

<pre class="brush: clojure; notranslate">(defn kth [n xs]  
  {:doc "P03 (*) Find the Kth element of a list." 
   :pre [(seq? xs)]}
  (if (= 0 n)
    (first xs)
    (recur (- n 1) (next xs)))
  )
</pre>

Here is my Scala:

<pre class="brush: scala; notranslate">def nth[T](n: Int, list: List[T]): T =
  if (n == 0)
    list.head
  else
    nth(n - 1, list.tail)
</pre>

In this case one can see a small but significant difference where Clojure sticks to functional-programming and Scala favours object-oriented: while in Clojure `first` and `next` are functions that operate on collections, in Scala `head` and `tail` are methods to be called upon a collection instance.