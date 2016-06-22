---
id: 163
title: 'P01 - Find the last element of a list'
date: 2014-01-16T01:07:27+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=163
permalink: /163/p01-find-the-last-element-of-a-list/
dsq_thread_id:
  - 2127686976
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - scala
---
The first problem is to find the last element of a list.

This is Peter&#8217;s [Clojure version](http://pbrc.blogspot.co.at/2014/01/99-clojure-problems-1-find-last-element.html):

<pre class="brush: clojure; notranslate">(defn my-last [input]
  "P01 (*) Find the last element of a list."
  (if (next input)
    (recur (next input))
    (first input)))
</pre>

And here is my Scala version:

<!--more-->

<pre class="brush: scala; notranslate">def last[T](list: List[T]): T =
  list match {
    case List(only) =&gt; only
    case first :: rest =&gt; last(rest)
    case Nil =&gt; error("An empty list has no last element")
  }
</pre>

Some things to note about the Scala version:

  * It is, of course, type-safe. It won&#8217;t accept anything but a list.
  * It is generic: it will work on lists of anything.
  * Because it&#8217;s recursive, the return type has to be declared.
  * You get a runtime-error if the list is empty

Because it&#8217;s generic and due to Scala&#8217;s type inference this will also work:

<pre class="brush: scala; gutter: false; notranslate">scala&gt; last(List(1, 2, true, "five", 8))
res0: Any = 8
</pre>

Scala will just infer to the most immediate matching super-class. In this case: `Any`.

But there is something disturbing in this implementation, however: raising an error if the list is empty. The Clojure version simply returns `nil` instead. Both solutions are not quite acceptable to me.

Having to deal with runtime-errors or magic &#8220;nulls&#8221; is something that as a Scala developer you are inclined to avoid. Fortunately Scala makes it very easy to do so, by switching to `Option[T]`:

<pre class="brush: scala; notranslate">def last[T](list: List[T]): Option[T] =
  list match {
    case List(only) =&gt; Some(only)
    case first :: rest =&gt; last(rest)
    case Nil =&gt; None
  }
</pre>

Now, the function might return `Some[T]` or `None` (it reads like English, doesn&#8217;t it?).