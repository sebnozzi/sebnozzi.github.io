---
id: 84
title: 'Scala to JavaScript &#8211; Comparing Scala.js and JScala'
date: 2013-09-14T14:07:16+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=84
permalink: /84/scala-to-javascript-comparison/
dsq_thread_id:
  - 1759163998
categories:
  - Scala
tags:
  - javscript
  - jscala
  - scala
  - scala.js
---
 <img class=" wp-image-101 alignnone" style="font-family: 'Open Sans', sans-serif; font-size: 15px; line-height: 24.296875px; font-style: normal; font-variant: normal;" alt="scala-to-js" src="/assets/2013/09/scala-to-js.png" width="762" height="386" />Whenever coding JavaScript, these are some of the things I miss from Scala:

  * Type-checking 
  * Type declarations 
  * Classes / Traits / Case-Classes 
  * Code modularization 
  * IDE support 

In this post I want to compare two very promising frameworks that generate JavaScript code from Scala.

<!--more-->

# Scala.js

Uses the **full Scala compiler** back-end to generate JavaScript from SBT projects.

Github repo: <a href="https://github.com/lampepfl/scala-js" target="_blank">https://github.com/lampepfl/scala-js</a>.

And more info <a href="http://lampwww.epfl.ch/~doeraene/scala-js/" target="_blank">here</a>.

## Pros

  * Covers the whole Scala language
  * Covers a good part of the Scala/Java library
  * Can re-use existing Scala SBT projects
  * Generates <a title="About source maps" href="http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/" target="_blank">source-maps</a> for Scala
  * Optimizes generated JavaScript
  * jQuery and DOM (statically-typed) wrappers available
  * It&#8217;s a <a href="http://lamp.epfl.ch/" target="_blank">LAMP EPFL</a> initiative, endorsed by M. Odersky

## Cons

  * <del><span style="line-height: 15px;">Long compile times</span></del> <span style="line-height: 15px;">Only when optimizing code (for production). Otherwise very fast incremental compiling.</span>
  * <span style="color: #000000;"><del>Big generated file </del>Only in development mode. Nevertheless Firefox loads it in under 1 second.</span>
  * <del>Calling Scala.js from existing JavaScript not trivial </del>Calling conventions have been improved.
  * <del>IDE integration issues </del>IDE Integration working without issues now.

<del>Note that all of these &#8220;negative&#8221; points are in the process to be fixed or improved upon</del> (FIXED!). Also, longer compile times and relatively big file sizes are an inescapable consequence of having the whole Scala language and library available. Perhaps one remaining &#8220;issue&#8221; is that it&#8217;s not trivial to integrate to existing frameworks (like Play). However, in my opinion this is outside of the scope of this project.

# JScala Uses

**macro-expansion** to generate JavaScript code in runtime.

Github repo: <a href="https://github.com/nau/jscala" target="_blank">https://github.com/nau/jscala</a> .

## Pros

  * Trivial to set up
  * **Generates code quickly on the fly**
  * Generated JavaScript is clean and readable
  * Calling generated code from JavaScript easy
  * Easily integrated in existing Web Frameworks (e.g. Play)

## Cons

  * **Covers only a subset of Scala</span>**
  * Covers only basic parts of the library (relies on JS for the rest)
  * Not trivial to modularize/share code
  * Cryptic macro-expansion error messages 

Note that although listed as a disadvantage, the fact that it only covers a subset of Scala and its library might be one of the reasons why code generation is so fast and clean. Being lightweight _has_ its advantages.

# Why can&#8217;t they collaborate?

This is one obvious question to arise when reading about these two frameworks. It would seem that the project leads are duplicating effort&#8230; But in the <a href="https://groups.google.com/forum/#!msg/scala-user/PbRQs6sl2eM/GTZqkCL51PEJ" target="_blank">words of Sèbastien Doeraene</a>, creator of Scala.js:

<div>
  <blockquote>
    <p>
      Sharing code between the Scala.js backend and JavaScript macros is indeed difficult.
    </p>
  </blockquote>
</div>

<div>
  <blockquote>
    <p>
      The translation from Scala ASTs to JS ASTs simply cannot be shared, because we do not do the same thing: the ASTs reaching the macros are much more complex than those reaching the backend, since those have already been simplified by the compiler transformation phases. Besides, as was mentioned, for macros you need to able to splice values defined in the surrounding Scala code (JsLazy?). Whereas the backend has other requirements: it must encode classes, interfaces, and overloaded methods into a specific scheme in JavaScript.
    </p>
  </blockquote>
</div>

<div>
  <blockquote>
    <p>
      So all in all, the job of our respective translation phase is actually very different: not the same input, not the same output!
    </p>
  </blockquote>
</div>

<div>
  <blockquote>
    <p>
      [&#8230;] So, I&#8217;m afraid we live in worlds that are too different, and that, despite the seemingly related job, we cannot share code effectively.
    </p>
  </blockquote>
</div>

# Which one to use?

This is something very subjective. The first thing I would say is: use whichever approach you feel more **comfortable** with, and what **best suits** your needs. My personal opinion would be to use:

  * **JScala** where you would write small-/medium-sized JavaScript code to really &#8220;script&#8221; some aspects of your web-page or application. It&#8217;s also best suited if your app is mostly server-based, but have some client-based aspects and don&#8217;t need advanced Scala features in your client-side-logic. It&#8217;s like Coffeescript but with Scala syntax. 
  * **Scala.js** if you have a medium-/big-sized codebase, which deserves a project on its own. Something like a one-page JavaScript application (fat-client), a demo, or a game. If you need the full spectrum of the Scala language and its library, and want to share code between server and client, this might be your safest (and only?) bet. So choose wisely. Or combine both.

# <span style="font-family: Oswald, sans-serif; font-size: 36px; line-height: 1.62em;">Conclusion</span>

For those of use that love the power and elegance of the Scala language and miss its features when coding in JavaScript, these are good news. No matter which approach you use, soon we might find ourselves doing web-based development exclusively in Scala. Sharing code between server and browser, benefiting from compile type-checking and IDE support, and leveraging existing JavaScript libraries. For the ones of us unhappy with JavaScript (and dialects) finally front-end web-development might become **fun** again!