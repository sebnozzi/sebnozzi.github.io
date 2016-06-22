---
id: 220
title: 'Scala.js &#8211; Accessing the JavaScript global scope'
date: 2014-01-26T17:42:49+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=220
permalink: /220/accessing-js-global/
dsq_thread_id:
  - 2178630366
categories:
  - Scala
tags:
  - cookbook
  - howto
  - javascript
  - recipe
  - scala
  - scala.js
  - scalajs
---
## Problem

You need to access the JavaScript global scope from Scala.js, in order to call functions like `alert`, `prompt`, `console`, etc.

<!--more-->

## Solution

What we need is the `global` value, which is a &#8220;view&#8221; of the JavaScript global scope.

It can be imported like this:

```scala
import scala.scalajs.js.Dynamic.global
```

Alternatively, you can import it like this:

```scala
import scala.scalajs.js
import js.Dynamic.{ global => g }
```

The latter might be more convenient way, since it imports the whole `js` package and renames `global` to something more easy to use (just `g`).

Now let&#8217;s issue an alert from Scala. Import the global scope and add this to your main class:

```scala
def main(): Unit = {
  g.alert("Hello from Scala")
}
```

After re-compiling and re-loading the host page, you should see the corresponding browser alert.

Let&#8217;s write something to the JavaScript console. Change your main class to look like this:

```scala
def main(): Unit = {
  g.console.log("Logged a message from Scala")
}
```

Re-compile and re-load. If you open your JavaScript console, you should see the message.
