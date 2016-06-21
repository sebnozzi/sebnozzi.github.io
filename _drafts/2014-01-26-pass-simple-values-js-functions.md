---
id: 229
title: 'Scala.js &#8211; Passing simple values to JavaScript functions'
date: 2014-01-26T18:24:23+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=229
permalink: /229/pass-simple-values-js-functions/
dsq_thread_id:
  - 2178727661
categories:
  - Scala
tags:
  - coderetreat
  - cookbook
  - howto
  - javascript
  - recipe
  - scala
  - scala.js
  - scalajs
---
## Problem

You need to pass simple values (String, Int, Boolean) to JavaScript functions.

<!--more-->

## Solution

Let us illustrate the case by writing some functions on the host-page.

Edit `index-dev.html` and add the following:

<pre class="brush: html; notranslate">&lt;script type="text/javascript"&gt;
function happyNew(name, year) {
  alert("Hey " + name + ", happy new " + year + "!");
}
&lt;/script&gt;
</pre>

As we saw in the recipe of [accessing the global scope](/220), we need to make the following import:

<pre class="brush: scala; notranslate">import scala.scalajs.js
import js.Dynamic.{ global =&gt; g }
</pre>

Now on the Scala side you can call it like this:

<pre class="brush: scala; highlight: [2]; notranslate">def main(): Unit = {
  g.happyNew("Martin", 2014)
}
</pre>

Which will result in the values being used and the corresponding alert being shown.

### How does this work?

If your IDE shows the use of implicits, you&#8217;ll note that two implicit conversions are in play here:

  * fromString
  * fromInt

These are defined inside of Scala.js&#8217;s `js.Any`. This is an excerpt of that:

<pre class="brush: scala; notranslate">package scala.scalajs.js

...

/** Provides implicit conversions from Scala values to JavaScript values. */
object Any {
  ...
  implicit def fromInt(value: scala.Int): Number = sys.error("stub")
  implicit def fromLong(value: scala.Long): Number = value.toDouble
  implicit def fromFloat(value: scala.Float): Number = sys.error("stub")
  implicit def fromDouble(value: scala.Double): Number = sys.error("stub")
  ...
  implicit def fromString(s: java.lang.String): String = sys.error("stub")
  ...
}
</pre>

### Important

Note that `Number` and `String` you see here are not the ones of the Scala type hierarchy! Though not seen in the code, in reality they are `js.Number`, `js.String`, `js.Boolean`, etc. which map to the **real JavaScript types** (String, Number, Boolean, etc.)

You **NEVER** pass Scala values or object instances to JavaScript (except some cases). Instead, these **implicit** functions **convert** from the **Scala type** to the **JavaScript type** seamlessly.

The fact that you don&#8217;t see it (thanks to implicit conversions) does not mean it&#8217;s not there. It _is_ there and _needs_ to be there.

### Other types

In other recipes we&#8217;ll cover cases like passing collections or anonymous functions.