---
id: 192
title: 'P04 - Find the Number of Elements of a List'
date: 2014-01-16T01:48:07+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=192
permalink: /192/p04-find-the-number-of-elements-of-a-list/
dsq_thread_id:
  - 2127775265
categories:
  - Scala
tags:
  - 99-problems
  - clojure
  - scala
---
Clojure recursive:

```clojure
(defn my-count [xs]
  "P04 (*) Find the number of elements of a list."
  (loop [num 0 xs xs]
    (if (next xs)
      (recur (+ num 1) (next xs))
      (+ num 1))))
```

Clojure with reduce:

```clojure
(defn my-count-reduce [xs]
  "P04 (*) Find the number of elements of a list."
  (reduce (fn [[c xs]] (inc c)) 0 xs))
```

Scala recursive:

```scala
def length[T](list: List[T]): Int =
  list match {
    case Nil => 0
    case x :: rest => 1 + length(rest)
  }
```

Scala foldLeft:

```scala
def length[T](list: List[T]): Int =
  list.foldLeft(0)((count, _) => 1 + count)
```

This last can probably be improved.
