---
id: 185
title: 'P02 - Find the Penultimate Element of a List'
date: 2014-01-16T01:35:39+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=185
permalink: /185/p02-find-the-penultimate-element-of-a-list/
dsq_thread_id:
  - 2143602634
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - recursion
  - scala
---
Peter [brings us](http://pbrc.blogspot.co.at/2014/01/99-clojure-problems-2-find-penultimate.html) this solution:

<pre class="brush: clojure; notranslate">(defn penultimate [xs]
  {:doc  "P02 (*) Find the last but one element of a list."
   :pre [(seq? xs)]}
  (loop [ret (first xs) xs xs]
    (if (next xs)
      (recur (first xs) (next xs))
      ret)))
</pre>

I can not comment much, since I&#8217;m no Clojure developer. In particular I would like to know what `recur` does? A generic macro to do recursion on a function? Seems interesting&#8230;

This is my solution in Scala:

<!--more-->

<pre class="brush: scala; notranslate">def penultimate[T](list: List[T]): T =
  list match {
    case List(first, last) =&gt; first
    case first :: second :: rest =&gt; penultimate(second :: rest)
    case _ =&gt; error("Can not find penultimate of this list: " + list)
  }
</pre>

Using pattern matching, it does the following:

  * if it matches a list consisting of only two elements (first and last) return the first one
  * if it matches a list consisting of a least two elements and a rest, do recursion using the second and the rest
  * in any other case the list has an unacceptable for, raise an error

Note the two matching styles for a list. The first one using the de-structuring of the List (`unapply` behind the scenes) and the second using the &#8220;consing&#8221; of list-elements.

This implementation either returns the sought element, or it raises an error. It might be a cleaner, more idiomatic Scala implementation to return an `Option[T]` instead. Let&#8217;s explore that possibility:

<pre class="brush: scala; notranslate">def penultimate[T](list: List[T]): Option[T] =
  list match {
    case List(first, last) =&gt; Some(first)
    case first :: second :: rest =&gt; penultimate(second :: rest)
    case _ =&gt; None
  }
</pre>

It can now be used like this:

<pre class="brush: scala; notranslate">penultimate(myList) map { element =&gt; /* do something with element */ }
</pre>

Or, if one wants to handle the case where `None` is found:

<pre class="brush: scala; notranslate">penultimate(myList) getOrElse { /* no penultimate found, deal with this */}
</pre>

Or, to handle both cases:

<pre class="brush: scala; notranslate">penultimate(myList) map { element =&gt; 
  /* do something with element */ 
} getOrElse { 
  /* no penultimate found, deal with this */
}
</pre>