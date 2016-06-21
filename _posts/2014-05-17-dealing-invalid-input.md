---
id: 370
title: Dealing with invalid Input
date: 2014-05-17T12:10:49+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=370
permalink: /370/dealing-invalid-input/
dsq_thread_id:
  - 2692219089
categories:
  - Scala
  - Software Development
tags:
  - compiler
  - eiffel
  - option
  - scala
  - type-safety
  - validation
---
Recently I&#8217;ve been discussing about [unit-tests and contracts](/362/contracts-replace-unit-tests/), and the question arose if mechanism existed that would check **at compile-time** whether a value satisfied a certain condition or not.

This is a difficult question and no doubt a hot topic in computer-science research.

My intuitive answer would be: no. There can&#8217;t be such a mechanism, especially if those values are coming from sources we can not control. For example: user input, reading from a file, from a database, from a web-service, etc.

Since those values will be known at _run-time_, we are forced to deal with them at that stage. However, we can use our faithful assistance the compiler to remind us (even force us) to deal with these cases.

What do I mean? If we can model the situation where value is either valid or not _with the type-system_ then we&#8217;ll have all the compiler assistance we want.

<img src="/assets/2014/05/Choice-sign-300x180.jpg" alt="Choice-sign" width="300" height="180" class="aligncenter size-medium wp-image-388" />

In Scala this is easily doable and in fact already part of the standard library. But nothing prevents it from being implemented in other languages as well (how easy or difficult this would be, is another question).

In this post I want to show you how to use Scala&#8217;s mechanisms to **safely parse an Int**, and how to guarantee at compile-time that it will be a **positive Int**.

<!--more-->

> Note: in Scala there are even more advanced solutions than what I&#8217;ll present here. The libraries scalaz and shapeless provide types which can model more closely cases like this. I&#8217;ll leave them out of the discussion because I want to explain the underlying concepts in a simple manner for non-Scala developers.

# Enter the Option

The `Option[T]` class in Scala is a type-safe wrapper around a _possible_ value of type T.

The &#8220;possible&#8221; in the previous statement is important. At runtime, concrete instances of Option[T] will either be:

  1. `Some[T]`, being Some[T] a subclass of Option[T]
  2. `None`, being it a singleton instance of Option[T]

Both Some and None implement the &#8220;protocol&#8221; of superclass `Option[T]`, but obviously do so _differently_.

For example, let&#8217;s say we have a variable `optName` of type `Option[String]`.

    optName foreach { name => println("Hello, " + name ) }
    

The statement above will only print a greeting IF `optName` has a value (or: is an instance of `Some[T]`). Otherwise nothing will happen. Note `name` parameter inside of the anonymous function / closure / block (however you want to call it). Inside of the block, the `name` parameter **is** of type String. We are safe there.

This form allows us to deal with both cases (when optName is Some, or is None) separately:

    optName map { name =>
      println("Hello, " + name)
    } getOrElse {
      println("Hello, World")
    }
    

And there are much more things we can do with Options, but we&#8217;ll leave it there. Just note that everything we are doing is type-safe, which means the compiler will remind (force) us if something is not quite right.

For all these kind of problems of questionable input the approach can be reduced to this:

  * either the input _was_ valid, and we&#8217;ll have a Some[T] to deal with
  * or the input wasn&#8217;t, and we&#8217;ll have a None to deal with

The relevant part is that **we&#8217;ll have to** deal with them in a type-safe way. Although the check will be done at _runtime_, at compile-time we&#8217;ll have to specify how to deal with theses cases **in advance**.

# Parsing Safely

Armed with this knowledge, we are ready to tackle the first problem: how to parse input and deal with it in a type-safe way.

We could model a parsing function like this:

    def safelyParse(nrStr: String): Option[Int]
    

The input is an arbitrary String, but the output is not directly an Int but an Option[Int]. Thus, we&#8217;ll have to respectfully handle the _possible_ case that the input was correct. For example like this:

    val userInput = Console.readLine()
    val optInt = safelyParse(userInput)
    
    optInt foreach { intValue =>
      println("You entered an Int: " + intValue)
    }
    

The above print statement will only execute IF the input was valid and the parsing succeeded. The variable `intValue` exists in a safe block which is only invoked in case of correct input. Otherwise, outside of that scope it is not clear what the Int value is, or if there is one at all.

A short implementation of the `safelyParse` could be:

    def safelyParse(nrStr: String): Option[Int] = {
      try {
        Some(nrStr.toInt)
      } catch {
        case e: NumberFormatException => None
      }
    }
    

# Positive Integers

Now it gets more interesting. Can we use the same approach to guarantee, at compile time, that we&#8217;ll be dealing with _positive integers_?

If we had a type that can be instantiated in a controlled way, I think we could. Imagine a &#8220;factory&#8221; of PositiveInt that gave you:

  1. either a Some[PositiveInt], in case you provided valid input, or
  2. a None, if the input provided was not valid

If you could guarantee that the **only** way to create a PositiveInt was through that factory, then the problem would be solved. After that, we have the situation of above: the developer is forces to deal with Option[T] accordingly.

## An unusual class declaration

A possible solution would look like this:

    class PositiveInt private (val value: Int) extends AnyVal {}
    

Whoa! A lot of things going on here. Allow me to explain:

  * It is a class declaration. So much should be clear.
  * The `(val value: Int)` declares a constructor parameter called &#8220;value&#8221; of type Int
  * Because of the `val` before it, the parameter `value` will be publicly accessible but read-only
  * The `private` keyword in front of the parameter list states that the constructor is private
  * Extending `AnyVal` tells the compiler to treat this class as a wrapper of a &#8220;primitive&#8221; type. Although Scala-the-language knows nothing about primitive types, Scala-the-compiler does. This means that our wrapper class will be as fast, and consume as few memory, as a real Int.

For our discussion the important part is that the constructor is private. That means: not just anyone can instantiate a PrivateInt.

So, who can? Read on&#8230;

## The companion object

In Scala it&#8217;s possible to declare _object instances_ by using the `object` keyword. A very special kind of object instances are the &#8220;companion objects&#8221; of classes. These are instances that can do things with a class that normal objects (or users of the class) can not. The only conditions are that:

  1. There can be only one companion object of a class
  2. It has to have the same name of the class
  3. It has to be on the same file as the class implementation

Let&#8217;s return at our PositiveInt class and remember that the constructor is private. Something like this is normally not possible and prevented by the compiler:

    new PositiveInt(4)
    

However, instance-creation of a PositiveInt **is** possible from within the companion object!

Thus, we can solve our problem like this:

    object PositiveInt {
      def create(nr: Int): Option[PositiveInt] = {
        if (nr > 0)
          // Note: the companion object CAN access private constructors
          Some(new PositiveInt(nr))
        else
          None
      }
    }
    

It could then used like:

    val optPositiveInt = PositiveInt.create(someIntValue)
    

Note that the creation **attempt** returns not a PositiveInt directly but an Option[PositiveInt].

## Controlled Instance Creation

Building on our previous example, we can now attempt to construct a PositiveInt like this:

    val userInput = Console.readLine()
    val optInt = safelyParse(userInput)
    
    optInt foreach { intValue =>
      val optPositiveInt = PositiveInt.create(intValue)
    
      optPositiveInt foreach { posInt =>
        println("Thanks! You gave me a positive Int: " + posInt.value)
      }
    }
    

The explanation given previously in the case of `safelyParse` also applies here: there are different ways to deal with the possible outcomes (Some/None).

I posted a [complete example, full with comments and unit-tests](https://gist.github.com/sebnozzi/c4824a1fb8bcc8a57aac) as a gist.

# Conclusion

While the compiler can not anticipate the validity of a value, and we are forced to check for this at run-time, we can make use of the type-system to isolate this problem as much as possible and let us deal with the possible outcomes in a type-safe way.

The mechanism we used to achieve this were:

  * The Option[T] wrapper class
  * Private/controlled instance creation

Although the examples were shown in Scala, where this is particularly easy thanks to built-in features suited for this purpose, nothing prevents implementing similar solutions in other languages with similar facilities.