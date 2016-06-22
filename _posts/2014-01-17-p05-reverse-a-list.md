---
id: 197
title: 'P05 - Reverse a List'
date: 2014-01-17T23:50:20+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=197
permalink: /197/p05-reverse-a-list/
dsq_thread_id:
  - 2138361673
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - recursion
  - scala
---
Clojure &#8211; recursive:

<pre class="brush: clojure; notranslate">(defn my-reverse [xs]
  "P05 (*) Reverse a list"
  (loop [ret [] xs xs]
    (if (empty? xs)
      ret
      (recur (cons (first xs) ret) (next xs)))))
</pre>

Clojure &#8211; with reduce:

<pre class="brush: clojure; notranslate">(defn my-reverse-reduce [xs]
  "P05 (*) Reverse a list"
  (reduce #(cons %2 %1) '() xs))
</pre>

Scala &#8211; recursive:

<pre class="brush: scala    ; notranslate">def reverse[T](list: List[T]): List[T] = {
  def helper(remaining: List[T], reversed: List[T]): List[T] =
    remaining match {
      case Nil =&gt; reversed
      case first :: rest =&gt; helper(rest, first :: reversed)
    }
  helper(list, Nil)
}
</pre>

Scala &#8211; with folding:

<pre class="brush: scala    ; notranslate">def reverseFolding[T](list: List[T]): List[T] =
  list.foldLeft(List.empty[T])((result, element) =&gt; element :: result)
</pre>