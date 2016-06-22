---
id: 82
title: Integrating Cucumber and Play
date: 2014-05-13T22:39:01+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=82
permalink: /82/cucumber-and-play/
dsq_thread_id:
  - 2682993638
categories:
  - Scala
tags:
  - cucumber
  - play
  - sbt
  - scala
---
In this post I want to show you how to integrate [Cucumber](http://cukes.info/) into a [Play](http://www.playframework.com/) (2.2.3) application.

It assumes some previous Play experience and that you know what Cucumber is about.

The example is minimal, but hopefully it is enough to get you started for bigger projects.

# Setting up the SBT Plugin

Let&#8217;s start by setting up the SBT plugin. Add to &#8220;project/plugins.sbt&#8221; the following lines:

    resolvers += "Templemore Repository" at "http://templemore.co.uk/repo"
    
    addSbtPlugin("templemore" % "sbt-cucumber-plugin" % "0.8.0")
    

This will make it possible for SBT to find and load the plugin. However, your project still lacks the corresponding _settings_ (refer to the [official page](https://github.com/skipoleschris/xsbt-cucumber-plugin) for a complete list). For this tutorial, we&#8217;ll be overriding some defaults. Add these lines to your &#8220;build.sbt&#8221;:

    cucumberSettings
    
    cucumberFeaturesLocation := "./test/features"
    
    cucumberStepsBasePackage := "features.steps"
    

<!--more-->

## Note

If you ever need to clone the [github project](https://github.com/skipoleschris/xsbt-cucumber-plugin) and publish locally it could be that Play does not find your local Ivy2 (as it was in my case). The solution is to add this resolver to &#8220;project/plugins.sbt&#8221;:

    resolvers += Resolver.file("Local repo", file(System.getProperty("user.home") + "/.ivy2/local"))(Resolver.ivyStylePatterns)
    

# Setting up Cucumber-Scala

Until now we just added the ability to SBT to find and deal with cucumber files. In order to _implement_ cucumber steps (and have good IDE integration) we need to add cucumber as a dependency.

We&#8217;ll be using the snapshot versions, so add:

    resolvers += "Sonatype-Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
    

And finally add &#8220;[cucumber-scala](https://github.com/cucumber/cucumber-jvm/tree/master/scala)&#8221; as a dependency:

    "info.cukes" % "cucumber-scala_2.10" % "1.1.5" % "test"
    

My complete &#8220;build.sbt&#8221; file looks like this:

    name := "PlayProjectName" // replace with your own
    
    version := "1.0-SNAPSHOT"
    
    scalaVersion := "2.10.3"
    
    resolvers += "Sonatype-Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
    
    libraryDependencies ++= Seq(
      jdbc,
      anorm,
      cache,
      "org.scalatest" % "scalatest_2.10" % "2.1.6" % "test",
      "info.cukes" % "cucumber-scala_2.10" % "1.1.5" % "test"  
    )     
    
    play.Project.playScalaSettings
    
    cucumberSettings
    
    cucumberFeaturesLocation := "./test/features"
    
    cucumberStepsBasePackage := "features.steps"
    

Note the [ScalaTest](http://www.scalatest.org/) dependency, which we&#8217;ll use later.

# Features and Step-Definitions

Under the &#8220;test&#8221; directory, create the package &#8220;features.steps&#8221;. This will result in 2 directories being created (&#8220;features&#8221; and &#8220;steps&#8221;). We&#8217;ll put the &#8220;.feature&#8221; files directly under &#8220;features&#8221; and the Scala step-definitions under &#8220;steps&#8221;.

Note that we already wrote the corresponding settings for SBT to find both features and step-definitions in these locations. The reason I like to specify the locations instead of letting the SBT plugin scan the whole classpath is of course because it&#8217;s _faster_ (it just looks on one place instead of everywhere).

When running Cucumber on the SBT console, a message should come that no features were found. Fine.

## Feature

Create a file called &#8220;up-and-running.feature&#8221; under &#8220;features&#8221; (the name does not really matter, but of course you want to choose a good name).

In the file, write the following content:

    Feature: Application up and running
    
      As a Play developer I want to see that I integrated Cucumber correctly
    
      Scenario: Seeing that the application is up and running
        Given my application is running
        When I go to the "start" page
        Then I should see "Hello world!"
    

## Definitions

Let&#8217;s create a file which will contain the step definitions, but let&#8217;s leave it empty for now. Create a class &#8220;StepDefinitions&#8221; in the package &#8220;features.steps&#8221; like this:

    package features.steps
    
    import org.scalatest.Matchers
    import cucumber.api.scala.{ ScalaDsl, EN }
    import cucumber.api.DataTable
    import cucumber.api.PendingException
    
    class StepDefinitions extends ScalaDsl with EN with Matchers {
    
      // step definitions will come here...
    }
    

Now if you run the &#8220;cucumber&#8221; on SBT you&#8217;ll get some messages telling you about unimplemented steps. You are given snippets which you can copy-paste into the Scala class and continue from there:

    Given("""^my application is running$""") { () =>
      //// Express the Regexp above with the code you wish you had
      throw new PendingException()
    }
    
    When("""^I go to the "([^"]*)" page$""") { (pageName: String) =>
      //// Express the Regexp above with the code you wish you had
      throw new PendingException()
    }
    
    Then("""^I should see "([^"]*)"$""") { (expectedText: String) =>
      //// Express the Regexp above with the code you wish you had
      throw new PendingException()
    }
    

Save the file and re-run the task. Now we get a &#8220;cucumber.api.PendingException&#8221;. Good.

# Play Integration

If you look at Play&#8217;s automatically generated `IntegrationSpec`, you&#8217;ll see that it looks very clean and lean. That&#8217;s because of facilities provided by Play&#8217;s &#8220;WithBrowser&#8221; class.

We want to re-produce the same goodness for our cucumber-based acceptance testing!

Inside of our test-definitions, we&#8217;ll not only start and stop our Play application, but also a server to host that application and a browser to inspect / interact with the UI.

Add this import line:

    import play.api.test._
    

Add these lines to your step-definitions class, at the beginning:

    val webDriverClass = Helpers.HTMLUNIT
    
    val app = FakeApplication()
    val port = 3333 // or whatever you want
    
    lazy val browser: TestBrowser = TestBrowser.of(webDriverClass, Some("http://localhost:" + port))
    lazy val server = TestServer(port, app)
    def driver = browser.getDriver()
    
    Before() { s =>
      // initialize play-cucumber
      server.start()
    }
    
    After() { s =>
      // shut down play-cucumber
      server.stop()
      browser.quit()
    }
    

# Implementing the Steps

## Navigating to type-safe URLs

Now we are ready to really implement the steps. Let&#8217;s start with the navigation.

Add these imports:

    import play.api._
    import play.api.mvc._
    

Let&#8217;s implement this step like this:

    When("""^I go to the "([^"]*)" page$""") { (pageName: String) =>
      val pageUrl = pageName match {
        case "start" => controllers.routes.Application.index.url
        case _ => throw new RuntimeException(s"unsupported page: $pageName")
      }
      browser.goTo(pageUrl)
    }
    

Note the beauty of using type-safe URLs here. Even if our case is trivial, and indeed `controllers.routes.Application.index.url` is much longer than writing `"/"`, it is a best practice that can pay-off later in a real and complex project.

Let&#8217;s leave this step empty:

    Given("""^my application is running$""") { () =>
      Logger.debug("Yeah, application IS running")
    }
    

Just to make sure the tests will fail let&#8217;s modify &#8220;index.scala.html&#8221; like this:

    @(message: String)
    
    @main("Welcome to Play") {
    
        Not what you are expecting...
    
    }
    

And now let&#8217;s run the cucumber task. The steps pass because we are not done yet&#8230;

## Using Fluentlenium

For the last step implementation, we are using [Fluentlenium](https://github.com/FluentLenium/FluentLenium), which is a Selenium wrapper already included in Play. Add these import statements:

    import org.openqa.selenium._
    import org.fluentlenium.core.filter.FilterConstructor._
    

(the first one is not strictly necessary in this case, but you might find it useful in your projects)

And let&#8217;s implement the step like this:

    Then("""^I should see "([^"]*)"$""") { (expectedText: String) =>
      val element = browser.find("body", withText().contains(expectedText))
      withClue("Expected text not found in body: " + expectedText) {
        element shouldNot be(empty)
      }
    }
    

(Note that using `browser.find("body", withText(expectedText))` would be incorrect since it would mean &#8220;the body tag which contains **exactly** the expected text&#8221;. It would pass in our case but only as long as the body&#8217;s content didn&#8217;t include anything else)

Let&#8217;s run the cucumber task&#8230; it should fail. Excellent.

Now that we are sure, let&#8217;s change our content of &#8220;index.scala.html&#8221; to the correct one:

    @(message: String)
    
    @main("Welcome to Play") {
    
        Hello world!
    
    }
    

Running the cucumber task should show all green.

# Recap

In this post I showed you:

  * how to setup your project to use the cucumber SBT-plugin
  * how to get started with cucumber
  * how to implement some basic Play integration
  * how to navigate to an URL type-safely
  * how to make basic usage of Fluentlenium

I hope you found it useful! Happy cucking!
