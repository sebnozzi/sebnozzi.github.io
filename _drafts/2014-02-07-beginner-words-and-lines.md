---
id: 244
title: 'Scala Beginners: counting words and lines'
date: 2014-02-07T19:14:14+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=244
permalink: /244/beginner-words-and-lines/
dsq_thread_id:
  - 2234542079
categories:
  - Scala
tags:
  - exercise
  - howto
  - scala
  - screencast
  - tutorial
---
## A simple command-line tool

At work, I&#8217;m teaching one intern and one colleague how to program.

I gave them the following exercise:

> Write an utility that takes 3 command-line parameters P1, P2 and P3. P3 is OPTIONAL (see below) P1 is always a file path/name. P2 can take the values:
> 
>   * &#8220;lines&#8221;
>   * &#8220;words&#8221;
>   * &#8220;find&#8221; 
> 
> Only P2 is &#8220;find&#8221;, then P3 is relevant/needed, otherwise it is not.
> 
> So, the utility does the following:
> 
>   * If P2 is &#8220;rows&#8221; it says how many lines it has
>   * If P2 is &#8220;words&#8221; it says how many words it has (the complete file)
>   * If P2 is &#8220;find&#8221; it prints out the lines where P3 is present

<!--more-->

Examples:

> Given a file myfile.txt with this content (without the line-numbers):
> 
>     01: Scala is a statically typed 
>     02: language released in 2004. 
>     04: It incorporates language 
>     05: features found in Ruby, 
>     06: Haskell and Java, among 
>     07: others.
>     
> 
> Then, if the utility is called &#8220;mytool&#8221;, then:
> 
>     mytool myfile.txt lines 
>     7
>     
>     mytool myfile.txt words 
>     21
>     
>     mytool myfile.txt find language 
>     language released in 2004. 
>     It incorporates language
>     

I asked them to solve it in Ruby, since I think it might be easier for them to start.

How would you solve this in **Scala**?

If you&#8217;re learning Scala, I encourage you to **try for yourself** first instead of reading my solution. It&#8217;s simply the best way to learn.

Feel free to ask questions in the comments section.

If you already tried, are stuck or are just outright impatient, read on&#8230;

## Solution in Scala

Here I present you my solution:



### Some highlights of the video:

#### Matching the command-line arguments

The naive approach:

<pre class="brush: scala; notranslate">if(args.size == 2) {
  if(args(1) == "lines") {
    countLines(args(0))
  } else if(args(1) == "words") {
    countWords(args(0))
  } else {
    printUsage()
  }
} else {
  if(args(1) == "find") {
    findWord(args(0), args(2))
  } else {
    printUsage()
  }
}
</pre>

The better approach:

<pre class="brush: scala; notranslate">args match {
  case Array(file, "words") =&gt; countWords(file)
  case Array(file, "lines") =&gt; countLines(file)
  case Array(file, "find", wordToFind) =&gt; findWord(file, wordToFind)
  case _ =&gt; printUsage()
}
</pre>

Things to notice:

  * How much simpler and elegant it is
  * How we match and bind variables
  * How we match against string-literals

#### Counting lines

<pre class="brush: scala; notranslate">def countLines(file: String) {
  val src = Source.fromFile(file)
  val count = src.getLines.size
  println(count)
}
</pre>

Things to notice:

  * The use of `Source` and `getLines` to read the lines of a file

#### Counting words

<pre class="brush: scala; notranslate">def countWords(file: String) {
  val src = Source.fromFile(file)
  val count =
    (for {
      line &lt;- src.getLines
    } yield {
      val words = line.split("\\s+")
      words.size
    }).sum
  println(count)
}
</pre>

Things to notice:

  * How we yield the amount of words per line
  * How we sum all word-amounts with `sum`
  * How we split words by one or more whitespace characters (space, tab, etc.)

#### Finding words

<pre class="brush: scala; notranslate">def findWord(file: String, wordToFind: String) {
  val src = Source.fromFile(file)
  for {
    (line, idx) &lt;- src.getLines.zipWithIndex
    if line.contains(wordToFind)
  } {
    println(f"$idx%02d: " + line)
  }
}
</pre>

Things to notice:

  * The use of `zipWithIndex` to have the index of the current element in a loop
  * The guard/condition in the for loop, which filters the lines we want
  * The use of the formatting string-interpolator

Drop me a line and tell me what you think! :-)