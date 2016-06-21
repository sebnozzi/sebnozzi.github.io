---
id: 128
title: Sliding-Puzzle with Scala.js
date: 2013-12-29T18:33:35+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=128
permalink: /128/sliding-puzzle-js/
dsq_thread_id:
  - 2080154092
categories:
  - Scala
tags:
  - javafx
  - javascript
  - scala
  - scalajs
---
## JavaFX Version

This **sliding-puzzle** started being an experiment to learn the basics of **JavaFX**. So the first version was **desktop-only**:

[<img class="aligncenter size-full wp-image-129" alt="screenshot-javafx" src="http://www.sebnozzi.com/wp-content/uploads/2013/12/screenshot-javafx.jpg" width="694" height="504" />](http://www.sebnozzi.com/wp-content/uploads/2013/12/screenshot-javafx.jpg)

But at the same time I already had heard of **[Scala.js](http://www.scala-js.org/)**, so I took care to make the code modular, separating **game-logic** from the **UI** as much as possible.

## JavaScript UI

During the last [DevFest](http://www.devfest.at/agenda) I _tried_ to implement the **Scala.js frontend** in the course of one afternoon. I made some progress, but was not nearly finished.

<!--more-->

My initial approach was to start from a\*\* JavaScript prototype of the UI\*\*, and try to glue it with the **existing Scala game-logic** later. This approach proved out to be very **productive**: I could try things, reload and see changes instantly. Soon I had the **basic functionality** which the game would need:

  * <span style="line-height: 15px;">separating the image into <strong>tiles</strong></span>
  * **event-handling** of mouse-clicks
  * **moving tiles** with animation
  * **hiding/showing** tiles with animation
  * **re-calculating** the tiles upon puzzle-**size change**

But due to &#8220;lack of time&#8221; (actually, because of [other projects](http://www.sebnozzi.com/104/cooperative-word-guessing/ "Cooperative word-guessing in Scala")) I left this for a while&#8230;

## Connecting with Scala.js

A couple of days ago, I continued my journey and started to **glue the existing Scala.js code with the UI prototype** written in JavaScript.

I learned some **valuable lessons** along the way, like:

  * how to **interact with JavaScript** code from Scala.js
  * how to write JavaScript calls to **3rd party libraries** that can survive [Google Closure](https://developers.google.com/closure/)&#8216;s optimizations
  * about the [jQuery](https://github.com/scala-js/scala-js-jquery) and [DOM](https://github.com/scala-js/scala-js-dom) **wrappers** for Scala.js

Once I got it working, I began to **migrate** what I had written in JavaScript more and more to the Scala side, until the original **JavaScript code completely disappeared** ;-)

After some days of hard-work I finally arrived at a version I&#8217;m comfortable with:

[<img class="aligncenter size-full wp-image-130" alt="screenshot-browser" src="http://www.sebnozzi.com/wp-content/uploads/2013/12/screenshot-browser.jpeg" width="672" height="545" />](http://www.sebnozzi.com/wp-content/uploads/2013/12/screenshot-browser.jpeg)

## Results

You can **try it out** without downloading or installing anything right now, since I made an **online demo** available here:

<http://www.sebnozzi.com/demos/sliding-puzzle/>

And if you are curious about how it was done and how you can use Scala.js to build multi-platform code, or simply develop a browser-based application, you can take a look at the **source-code** here:

<https://github.com/sebnozzi/sliding-puzzle>

The resulting project compiles both to **JavaScript** (and uses jQuery under the hood) and as a **native JVM** application (requires support for JavaFX).

I hope this simple demo gives an idea of what is possible today by writing **pure Scala**.