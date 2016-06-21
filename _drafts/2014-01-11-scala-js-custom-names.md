---
id: 142
title: Scala.js Project with Custom Names
date: 2014-01-11T23:43:08+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=142
permalink: /142/scala-js-custom-names/
dsq_thread_id:
  - 2110051368
categories:
  - Scala
tags:
  - howto
  - recipe
  - sbt
  - scalajs
---
In this recipe you&#8217;ll learn how to:

  * Create a basic Scala.js project
  * … with a **custom name**
  * … and **package structure**

As an example, we&#8217;ll set-up a project for a “countdown” application.

<!--more-->

## Cloning the sample application

We&#8217;ll start by cloning the **sample-application** provided by Sébastien Doeraene.

Assuming “git” is installed in your machine, you need to clone this project:

<p style="padding-left: 30px;">
  <a href="https://github.com/sjrd/scala-js-example-app">https://github.com/sjrd/scala-js-example-app</a>
</p>

Rename the resulting **directory** “scala-js-example” to “countdown-example”.

## Changing the project name

Look at the directory structure. Edit ”**buid.sbt**”, change the name to “Countdown Example”.

<pre>name := "Countdown Example"</pre>

I am using a string with a space on purpose to handle this more “tricky” case. Normally, just “Countdown” would suffice.

## Changing the Packages

Renaming the packages might be easier to accomplish from within an IDE than directly manipulating directories.

In this case you should **generate the IDE project** first, and then import it (using the SBT plugins for either Eclipse or IntelliJ).

After having imported the project to your IDE, do the following:

  * rename the package “example” (in “**main**”) to “com.sebnozzi.countdown”
  * rename the package “example.test” (in “**test**”) also to “com.sebnozzi.countdown”

<p style="padding-left: 30px;">
  For Eclipse, watch out for <strong>Scala-IDE bugs</strong>. The package names, or the imports, in the Scala files might be scrambled or wrong :-(
</p>

At the end, the **directory structure** should match this:

  * src/**main**/scala/com/sebnozzi/countdown/**ScalaJSExample.scala**
  * src/**test**/scala/com/sebnozzi/countdown/**ScalaJSExampleTest.scala**

And the the **Scala classes** should refer to the new package:

<pre>package com.sebnozzi.countdown</pre>

## Renaming the classes

Rename **both** file-names and classes&#8230;

  * from “ScalaJSExample” to “**CountdownExample**[.scala]”.
  * from “ScalaJSExampleTest” to “**CountdownExampleTest**[.scala]”.

Make sure that you rename **both files _and_ classes**, as in Scala they don&#8217;t need to match.

Also watch out for the **test class**, since it references (imports) the main class. If your IDE didn&#8217;t pick up the rename, you&#8217;ll have to fix it yourself:

<pre>it("should implement square()") {
 import CountdownExample._</pre>

Lastly, though not critical, it would also be correct to change this line in the **test class**:

<pre>describe(“ScalaJSExample”) {</pre>

to:

<pre>describe(“CountdownExample”) {</pre>

## Fixing the JavaScript file-references

The **HTML files** “index-dev.html” and “index.html” are used to &#8220;host&#8221; the Scala.js application. They load the generated JavaScript files, which the SBT plugin generates somewhere under &#8220;target&#8221;.

Since the generated file-names correspond to the SBT-project&#8217;s name (which we renamed), the HTML files need to be updated.

In &#8220;**index-dev.html**&#8221; you need to change:

  * “example-extdeps.js” to “countdown-example-extdeps.js”
  * “example-intdeps.js” to “countdown-example-intdeps.js”
  * “example.js” to “countdown-example.js”

And in &#8220;**index.html**&#8220;:

  * “example-opt.js” to “countdown-example-opt.js”

## Fixing the bootstrap file (startup.js)

There is one last very special file that needs to be updated. Under the directory &#8220;js&#8221; you&#8217;ll find &#8220;**startup.js**&#8220;.

This JavaScript file kick-starts the Scala.js application by calling the &#8220;main()&#8221; method of the main class.

<p style="padding-left: 30px;">
  As you might have notices, nowhere in the HTML files is &#8220;startup.js&#8221; referenced. Instead, it is referenced in &#8220;build.sbt&#8221; <strong>build-file</strong>. When generating code, Scala.js <strong>includes</strong> the contents of &#8220;startup.js&#8221; in the <strong>resulting JavaScript files</strong>.
</p>

We need to change both the references to the **package** and the **main class** (the method name did not change). The **convention** to use for the package name is to replace the dots “.” with **underscores** “_”.

The end-result should look like:

<pre>ScalaJS.modules.com_sebnozzi_countdown_CountdownExample().main();</pre>

## <span style="font-family: Oswald, sans-serif; font-size: 32px; line-height: 1.62em;">Making sure it works</span>

Generate the **development (**<span style="font-weight: 600;">unoptimized</span>**)** version with “packageJS” from the SBT-console:

<pre>sbt&gt; packageJS</pre>

Once finished, load the “**index-dev.html**” file in your browser.

<p style="padding-left: 30px;">
  Loading the development version takes from less than 1 second to more than 5, depending on your setup and browser. Experiment with different browsers, as some are much faster than others (in turn, depending on your operating system).
</p>

You should read “It works!”.

Just to be sure, try out the **optimized** version as well. Again, on the SBT-console run:

<pre>sbt&gt; optimizeJS</pre>

<p style="padding-left: 30px;">
  WARNING: while the optimization process produces a very small and fast file, it takes some time to do so. On my setup it&#8217;s around 2 minutes. This is mode is intended for &#8220;deployment&#8221; situations.
</p>

Once done, open “**index.html**”. It should load immediately and again you should read “It works!”.

Last but not least, let&#8217;s check that the **tests** work. On the SBT console, run:

<pre>sbt&gt; test</pre>

The tests should all pass.

## Wrap-up

In this recipe you learned how to set-up a basic Scala.js with custom class names and package.

This should give you what you need to start your own Scala.js projects.