---
id: 204
title: 'P07 &#8211; Flatten a nested list structure'
date: 2014-01-19T11:53:12+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=204
permalink: /204/p07-flatten-list/
dsq_thread_id:
  - 2143526032
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - recursion
  - scala
---
Peter not only offers you one but [three solutions](http://pbrc.blogspot.co.at/2014/01/99-clojure-problems-7-flatten-nested.html) in Clojure.

Clojure recursive:

<pre class="brush: clojure; notranslate">(defn flatten-recur
  "P07 (**) Flatten a nested list structure."
  ([xs] (flatten-recur [] xs))
  ([acc xs]
     (if-not
      (seq? xs) (conj acc xs)
      (if (empty? xs)
        acc
        (-&gt; (flatten-recur acc (first xs))
            (flatten-recur (rest xs)))))))
</pre>

Clojure de-structured:

<!--more-->

<pre class="brush: clojure; notranslate">(defn flatten-destructured
  "P07 solution I saw on cchandler's github repo"
  [x & tail]
  (concat (if (seq? x)
            (apply flatten-destructured x)
            [x])
          (if (nil? tail)
            nil
            (apply flatten-destructured tail))))
</pre>

Clojure with reduce:

<pre class="brush: clojure; notranslate">(defn flatten-reduce
  "P07 solution I saw on rodnaph's github repo "
  [xs]
  (reduce #(if (seq? %2)
             (concat %1 (flatten-reduce %2))
             (concat %1 (list %2)))
          '() xs))
</pre>

I just came up with one for Scala:

<pre class="brush: scala; notranslate">def flatten(list: List[_]): List[_] = {
  def helper(remainder: List[_], result: List[_]): List[_] =
    remainder match {
      case Nil =&gt; result
      case (head: List[_]) :: tail =&gt; helper(tail, result ++ flatten(head))
      case head :: tail =&gt; helper(tail, result :+ head)
    }
  helper(list, Nil)
}
</pre>

For the first time I am forces to depart from lists of one type of elements and resort to `List[_]`, which would translate to &#8220;list of I-don&#8217;t-care&#8221;. This is of course so because due to the nature of nested lists, one cannot effectively make predictions about the type. It would result in an infinite recursion of declarations: &#8220;list of elements of type T OR of (lists of element of type T or of (lists of&#8230;&#8221;.

It is also unfortunate that the type declaration makes the logic of the function a little bit more difficult to read (though one can get used to it). Of course a type-aliasing could solve the problem like this:

<pre class="brush: scala; notranslate">type L = List[_]
def flatten(list: L): L = {
  def helper(remainder: L, result: L): L =
    remainder match {
      case Nil =&gt; result
      case (head: L) :: tail =&gt; helper(tail, result ++ flatten(head))
      case head :: tail =&gt; helper(tail, result :+ head)
    }
  helper(list, Nil)
}
</pre>

We can see that this solution is, in spirit, not much different than the first Clojure one. It just does not explicitly ask &#8220;is this a collection?&#8221;, &#8220;is it empty?&#8221; through functions like in Clojure but leaves those decisions to the Scala pattern-matching mechanism. The benefit of this is the compiler warning if the pattern matching is not exhaustive.

The Scala solution uses, once again, a helper function which does the real recursion and accumulation of results. The recursion ends when there isn&#8217;t anything else to process.

The interesting matching is done on the head element of the incoming list (we are always within a list). The head element might itself be a list or not. If it is a list, then it is further flattened and appended to the current results. If it is not, then it&#8217;s appended to the results without flattening.

Note the difference between `++` and `:+` in (immutable) collections. The former serves to obtain the concatenation of two collections, while the latter serves to obtain the concatenation of one **element** to an existing collection.