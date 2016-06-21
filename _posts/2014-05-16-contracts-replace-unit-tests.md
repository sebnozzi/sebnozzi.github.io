---
id: 362
title: Can Contracts replace Unit-Tests?
date: 2014-05-16T16:03:10+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=362
permalink: /362/contracts-replace-unit-tests/
dsq_thread_id:
  - 2690184895
categories:
  - Scala
  - Software Development
tags:
  - contracts
  - dbc
  - eiffel
  - scala
  - tdd
  - tests
  - unit-tests
---
# Introduction

What are &#8220;contracts&#8221;? Contracts are the key element in the [design-by-contract](https://en.wikipedia.org/wiki/Design_by_contract) technique, natively supported by the [Eiffel](https://en.wikipedia.org/wiki/Eiffel_%28programming_language%29) programming language.

They allow operate at many levels:

  1. methods: as pre- and post-conditions
  2. classes: as class invariants
  3. loops: as loop variants/invariants

In this article we shall only consider method pre- and post-conditions. That means, before method execution the pre-condition is checked and after the method is finished (upon completion) the post-condition is checked.

It&#8217;s an automatic runtime-enforced form of defensive programming. Recently I started debating with an Eiffel expert about unit-tests vs. contracts, his points being that contracts offer a superior mechanism over unit-tests. That discussion lead me to analyse his claims and write this article.

<img src="/assets/2014/05/agreement-hands-300x198.jpg" alt="agreement-hands" width="300" height="198" class="aligncenter size-medium wp-image-391" />

So the question is: **can contracts replace unit-tests?**

<!--more-->

And I am tempted to answer &#8220;no&#8221;. Why? Contracts are a wonderful thing, but it&#8217;s deceiving to think that they can be used in the same way, and for the same purposes, as unit-tests. Let&#8217;s look at some examples to back up my claim&#8230;

> (Disclaimer: I am not an Eiffel expert and part, or the whole, of what I say here regarding that language can be inaccurate or wrong. If this is the case, feel free to correct me.)

# Naive Example

Naive comparison, consider this Scala code:

    def getFirstElement(nonEmptyList: List[T]) {


Some unit-tests we would like to write are, in ScalaTest:

    it should "throw exception if list not empty" in { ...

    it should "return a non-null element for non-empty list" in { ...


The contracts proponent can argue that unit-testing is not necessary, that it can be covered by contracts.

Consider this pseudo-Eiffel code:

    first_element(non_empty_list: List) is
    require
      non_empty_list.is_empty == false
    do
      -- implementation here
    ensure
      element != null
    end


Contracts seem to be better here. We are checking the same, but with contracts we have the checks visible as part of the API. Better for self-documenting code.

One big difference, however, is that of timing. When are contracts checked and when are unit-tests run?

Contracts are checked, and fail, at runtime. If one wants to provoke a contract to fail, then we must do so either by the normal flow of the program (start the program, do some manual testing and see what happens) or explicitly as part of a test-run (as in EiffelStudio&#8217;s AutoTest).

In the first case it provides a helpful source of _debugging information_, but nothing we would like to happen on production. Clearly we want to catch those errors _before_ deploying!

And provoking the failing as part of an automated test-run seems to be similar with unit-testing. But contracts still have the added benefits of providing self-documenting code, also propagated through the hierarchy chain.

Contracts **are** a valuable tool. But can they _replace_ unit-testing?

# Simple Counter-Example

Consider this Scala function:

    def max(a: Int, b: Int): Int = ???


It&#8217;s clear: it should return the maximum between 2 numbers. The implementation is trivial, it would go something along the lines of:

    if(a > b) a else b


Now&#8230; it is always important to test that the implementation is correct. Some unit-tests would check that:

  * given 2 and 3, then 3 should be the result
  * given 32 and -10, then 32 should be the result
  * etc.

Indeed we could go on as far as testing this function using _[property testing](http://scalacheck.org/)_.

What would the contract for a function like this look like?

Would be re-stating in the contract what the implementation already states? That if a is greater than b, then the result should have been a, otherwise b? Or would we just state that the result is either a or b?

None of the options seem satisfactory. In the first one we are repeating ourselves, in the second one, the check is not enough.

No matter which route we go, we would still want to test the function against a pre-defined set of data, and checking that it works as expected. Clearly contracts are not adding much what normal unit-tests would not cover. Indeed, contracts seem to fall short here.

# A more complex example

Now consider a more complex example: a class has as a dependency a password-list (or password-list provider), and a function has the task to see if a pair of credentials is valid.

In Scala:

    def isValidPair(user: String, password: String) : Boolean = ???


The implementation would look at the list and would evaluate as follows:

  * if the user was not found => FALSE
  * if the user was found but the password does not match => FALSE
  * if the user was found and the passwords match => TRUE

What could contracts to this function be?

That user and password may not be null? OK&#8230; That the user has to be in the list? That the password is valid? Clearly not: these are things the function has to, and be given the chance to, check.

Again, should the &#8220;ensure&#8221; part re-state what the implementation does? Generally: should we make sure that the implementation works by generally re-stating the implementation in the ensure part? The idea seems ridiculous to me.

Clearly the only way to see if this function behaves as it should is through unit-testing: Instantiate a class with a fake collaborator, a list of users and passwords known beforehand, and test the function with known users and passwords (and users or passwords known to be wrong) and see what happens.

I don&#8217;t see contracts solving this problem as unit-tests do.

# Conclusion

To summarize:

Contracts _are_ a very handy and valuable tool. Contracts are in fact better than defensive asserts in the implementation, because they become part of the API interface and serve as executable documentation (when assertions are turned on).

But contracts are not a replacement for unit-tests. There are cases where contracts can not cover what unit-tests do.

Rather than one being superior to the other, contracts and unit-tests _complement_ each other.
