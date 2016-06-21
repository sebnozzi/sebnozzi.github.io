---
id: 282
title: Scaladin chat demo in a Vaadin/Eclipse project
date: 2014-04-15T21:33:15+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=282
permalink: /282/scaladin-chat-demo-eclipse/
dsq_thread_id:
  - 2685517854
categories:
  - Scala
tags:
  - eclipse
  - scala
  - vaadin
---
In this post I&#8217;m going to show you how to integrate a [Scaladin](https://github.com/henrikerola/scaladin) project into an Eclipse-generated [Vaadin](https://vaadin.com/home) setup. The [demo](https://github.com/henrikerola/scaladin-chat) I&#8217;ll be using is a small but impressive one: it uses Vaadin&#8217;s [Atmosphere-powered](https://github.com/Atmosphere/atmosphere) &#8220;**server-push**&#8221; and **Akka-Actors** (disclaimer: I don&#8217;t take credit, since I did not write the demo).

I like Scala, [Vaadin](https://vaadin.com/home), and find the idea of [Scaladin](https://github.com/henrikerola/scaladin) excellent (to be able to code for Vaadin using idiomatic Scala). There is a [SBT plugin for Scaladin](https://github.com/henrikerola/sbt-vaadin-plugin), which integrates very well with this build tool. On the other side, the benefits of the Vaadin plugin and IDE integration are lost: like debugging in Eclipse, for example.

What I set up to do was modify the IDE-generated Vaadin project until:

  * Scala was supported
  * [Scaladin](https://github.com/henrikerola/scaladin) was supported

As an excercise I &#8220;ported&#8221; (actually more like &#8220;copy-pasting&#8221;) the existing &#8220;[Scaladin Chat demo](https://github.com/henrikerola/scaladin-chat)&#8221; to the IDE-generated default setup.

[<img src="http://www.sebnozzi.com/wp-content/uploads/2014/04/scaladin-chat-eclipse.png" alt="scaladin-chat-eclipse" width="1325" height="929" class="aligncenter size-full wp-image-290" />](http://www.sebnozzi.com/wp-content/uploads/2014/04/scaladin-chat-eclipse.png)

Here&#8217;s what I did&#8230;

<!--more-->

First, I followed the [official instructions on how to get started with a Scala Vaadin project](https://vaadin.com/wiki/-/wiki/Main/Scala+and+Vaadin+HOWTO) (this includes adding the &#8220;Scala nature&#8221; to the project!).

However, I did not add the `scala-library.jar` as instructed there. Instead, I added it as an Ivy dependency to the `ivy.xml` file:

    <!-- Scala Runtime Library -->      
    <dependency org="org.scala-lang" name="scala-library" rev="2.10.4"/>
    

At that point I had a Scala-capable Vaadin project. Then I added [Scaladin](https://github.com/henrikerola/scaladin):

    <!-- Scaladin -->
    <dependency org="org.vaadin.addons" name="scaladin" rev="3.0.0" />
    

Since I didn&#8217;t know how to bootstrap Scaladin from a Scala class, I left the Java bootstraping class for the moment. Instead, I wrote a Scala class that would provide the main component:

    package com.example.vaadindemo
    
    import vaadin.scala.VerticalLayout
    import vaadin.scala.Button
    import vaadin.scala.Notification
    
    class SomeScalaClass {
    
      def content: com.vaadin.ui.Component = {
        val scaladinComponent = new VerticalLayout {
          margin = true
          components += Button(caption = "Click Button!", {
            println("Button clicked (Server)")
            Notification.show("Hello via Scaladin!")
          })
        }
    
        // return the "real" wrapped Vaadin component
        scaladinComponent.p
      }
    
    }
    

Here you can see that I&#8217;m returning a normal Vaadin UI component, not a Scaladin one. Although I am using Scaladin&#8217;s &#8220;magic&#8221; (the nice DSL which wraps over Vaadin) at the end of the day I return the wrapped Vaadin component. Which is the one &#8220;consumed&#8221; by the Java bootstrapping class:

    package com.example.vaadindemo;
    
    import javax.servlet.annotation.WebServlet;
    
    import com.vaadin.annotations.Theme;
    import com.vaadin.annotations.VaadinServletConfiguration;
    import com.vaadin.server.VaadinRequest;
    import com.vaadin.server.VaadinServlet;
    import com.vaadin.ui.UI;
    
    @SuppressWarnings("serial")
    @Theme("vaadindemo")
    public class VaadindemoUI extends UI {
    
        @WebServlet(value = "/*", asyncSupported = true)
        @VaadinServletConfiguration(productionMode = false, ui = VaadindemoUI.class, widgetset = "com.example.vaadindemo.widgetset.VaadindemoWidgetset")
        public static class Servlet extends VaadinServlet {
        }
    
        @Override
        protected void init(VaadinRequest request) {
        SomeScalaClass scalaInstance = new SomeScalaClass();
            setContent(scalaInstance.content());
        }
    
    }
    

Note how the Java part consumes the component instantiated on the Scala part. This is a very nice setup to profit from Vaadin&#8217;s annotations, which makes using a `web.xml` file unnecessary. More on that in a moment.

Debugging works as expected, which I tested by placing a breakpoint on Scala&#8217;s `println()`.

But of course having Java code lying around didn&#8217;t make me happy&#8230; I wanted to see if I could integrate the &#8220;[Scaladin Chat demo](https://github.com/henrikerola/scaladin-chat)&#8221; into this project setup. In other words: if I could also bootstrap the Vaadin application from Scala.

First thing I did was to check if I satisfied the dependencies&#8230; These are the dependencies listed in the project&#8217;s `build.sbt`:

    libraryDependencies ++= Seq(
      "com.vaadin" % "vaadin-server" % "7.1.3",
      "com.vaadin" % "vaadin-themes" % "7.1.3" % "container",
      "com.vaadin" % "vaadin-client-compiled" % "7.1.3" % "container",
      "com.vaadin" % "vaadin-push" % "7.1.3",,
      "vaadin.scala" %% "scaladin" % "3.0-SNAPSHOT",
      "com.typesafe.akka" %% "akka-actor" % "2.2.1",
      "org.eclipse.jetty" % "jetty-webapp" % "8.1.12.v20130726" % "container",
      "org.eclipse.jetty" % "jetty-websocket" % "8.1.12.v20130726" % "container"
    )
    

What&#8217;s obviously missing is `akka-actor`. I assumed that the jetty components were used by the sbt-plugin and not needed in my case (I was using Tomcat). So I added the missing dependency to my Ivy file:

    <!-- Akka Actors -->
    <dependency org="com.typesafe.akka" name="akka-actor_2.10" rev="2.2.1"/>
    

After that I copy-pasted the Scala classes / files directly under &#8220;src&#8221;:

  * `com/example/scaladinchat/ScaladinChatUI.scala`
  * `com/example/scaladinchat/chat.scala`

The IDE-generated Vaadin project has only &#8220;src&#8221;; it does not have &#8220;main/scala&#8221; or &#8220;main/java&#8221;. For sake of brevity I will not reproduce the contents of these Scala files, which can be found on the [original project](https://github.com/henrikerola/scaladin-chat).

Now comes a very important point. The Scaladin project makes use of (needs?) `web.xml`. On the original project this is found under `src/main/webapp/WEB-INF`. This file, on the IDE-generated project is to be put under `WebContent/WEB-INF`.

I really don&#8217;t know why Scaladin&#8217;s version needs the `web.xml` whereas the official Vaadin code does not seem to. If any reader can enlighten me, I will be very grateful.

Part of what the annotations used to cover in the original Java code is then found on the `web.xml` file, like:

  * the servlet path
  * the servlet class (auto-discovered)
  * the production mode

Other info covered by the annotations can be passed as initialization parameters to the main Scaladin UI component. I modified the `ScaladinChatUI` construction to look like this:

    class ScaladinChatUI extends UI(
      theme = "vaadindemo",
      widgetset = "com.example.scaladinchat.widgetset.VaadindemoWidgetset")
        with ChatClient { app =>
    

I had changed the path of the widgetset file. As a last step, I deleted my two previous files `VaadindemoUI.java` and `SomeScalaClass.scala`.

Finally, after all these changes, I had very nice Vaadin demo, using server-push (powered by [Atmosphere](https://github.com/Atmosphere/atmosphere)) and Akka-Actors, written exclusively in Scala!

## Credits

Many thanks go to [Henri Kerola](https://github.com/henrikerola?tab=repositories) for his contributions! Henri works at [Vaadin](https://vaadin.com/home) and created practically everything Scala+Vaadin related: from the chat-demo used as illustration here, to the SBT-plugin, to Scaladin itself. Be sure to check out his [github repositories](https://github.com/henrikerola?tab=repositories), where you can find other Scaladin demos, integration of Vaadin into Play, Akka-related projects, etc.