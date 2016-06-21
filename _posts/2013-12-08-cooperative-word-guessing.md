---
id: 104
title: Cooperative word-guessing in Scala
date: 2013-12-08T14:48:07+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=104
permalink: /104/cooperative-word-guessing/
dsq_thread_id:
  - 2035287216
categories:
  - Scala
tags:
  - akka
  - dojo
  - play
  - scala
  - svug
---
In our <a title="Scala-Vienna-User-Group Dojo" href="http://www.meetup.com/scala-vienna/events/122006312/" target="_blank">last Scala coding-dojo</a>, we had some fun with word-guessing (think &#8220;<a title="Hangman (game)" href="http://en.wikipedia.org/wiki/Hangman_(game)" target="_blank">hangman</a>&#8220;, but without hanging anybody ;-).

The initial problem was like this: I brought game server (running on my laptop) and the task was to implement an interactive console-based client. A simple overview of the client logic can be illustrated like this:

[<img class="wp-image-107 aligncenter" alt="wordguess-workflow" src="/assets/2013/12/wordguess-workflow.png" width="216" height="364" />](/assets/2013/12/wordguess-workflow.png)

Actually it was an <a title="Word-guess-server project" href="https://github.com/scala-vienna/wordguess-server" target="_blank">Akka-based word-guessing server</a>, for which I provided the <a title="Word-guess-client project" href="https://github.com/scala-vienna/wordguess-client" target="_blank">bare-bones client project</a>, which already came with the Akka-messages, the setup, and the message processing loop.

<!--more-->

For those new to Akka, it translates to this: there is a game-server and all the clients connect to it. Client and server send each other “messages” over the network **asynchronously**. The messages are implemented in the form of simple case-classes:

<pre class="brush: scala; notranslate">// Request to participate in a game 
case class RequestGame(playerName: String)

// Sent to the server to make a guess in the current game 
case class MakeGuess(letter: Char)

// Reflects the (partial) status of a game. Unsolved letters are "None". 
case class GameStatus(gameId: Int, letters: Seq[Option[Char]], remainingTries: Int)

abstract class GameOver
case class GameWon(finalStatus: GameStatus) extends GameOver
case class GameLost(finalStatus: GameStatus) extends GameOver // Sent when there are no more available games. Client should quit. 

case class NoAvailableGames()
</pre>

Given those messages, we can illustrate the message passing:

[<img class="wp-image-116 aligncenter" alt="wordguess-messaging" src="/assets/2013/12/wordguess-messaging1.png" width="211" height="228" />](/assets/2013/12/wordguess-messaging1.png)

As I would have expected, the interactive client was done in no time. After 10 minutes or so all the pairs had finished their implementations and were already trying to guess some words.

This is where the surprise came in, as I connected my laptop to the beamer and showed this:

<div style="width: 943px" class="wp-caption alignnone">
  </p> 
  
  <p>
    <a href="http://www.meetup.com/scala-vienna/events/122006312/"><img title="word-guess live-feed" alt="" src="http://photos2.meetupstatic.com/photos/event/2/5/f/a/highres_312669722.jpeg" width="933" height="697" /></a>
  </p>
  
  <p>
    <p class="wp-caption-text">
      Real-time updates of the word-guessing games
    </p></div> 
    
    <p>
      That&#8217;s right! The words for the individual games were actually coming from a larger text. I then revealed what the <em>real</em> goal of the dojo was, to:
    </p>
    
    <blockquote>
      <p>
        &#8230;try to unfold the whole text, programmatically and <strong>together</strong>.
      </p>
    </blockquote>
    
    <p>
      How so? There were two additional messages I previously didn&#8217;t focus on:
    </p>
    
    <pre class="brush: scala; notranslate">// Used to send a message to all other players 
case class SendToAll(msg: String) 
// Players get this message when talking to each other 
case class MsgToAll(msg:String)
</pre>
    
    <p>
      Using these two messages the clients could broadcast a String to all other clients.
    </p>
    
    <p>
      Also something which I minimized at the beginning was the significance of the <strong>game-id</strong>, and its relation to the whole text. As you might have guessed the game-id was nothing more than the <strong>word-index within the text</strong>.
    </p>
    
    <p>
      So, how to use this information and message-passing to solve the problem? The idea was to “share knowledge”. Clients would be communicating with each other about their successes and failures. Given a word that one client could not solve, the knowledge would be communicated about which letters worked and which didn&#8217;t. If another client got that word in the future, it would have much better chances of solving that word by <strong>relying on previous knowledge</strong>.
    </p>
    
    <p>
      In short, a client would would ask itself: Do I have knowledge about the word? If so:
    </p>
    
    <ul>
      <li>
        Try the successful letters right away
      </li>
      <li>
        Avoid the letters known to fail
      </li>
      <li>
        Try letters still untried letters
      </li>
      <li>
        If the world is solved, bingo
      </li>
      <li>
        If it&#8217;s not, <strong>communicate what worked and what didn&#8217;t</strong> The results were pretty good: the pairs came up with a communication protocol and ultimately the problem was solved. Even by doing clever tricks like taking into account the <strong>probabilities</strong> of the letters given a word, or given the English language in general.
      </li>
    </ul>
    
    <div style="width: 943px" class="wp-caption alignnone">
      </p> 
      
      <p>
        <a href="http://www.meetup.com/scala-vienna/events/122006312/"><img title="Text almost solved" alt="" src="http://photos1.meetupstatic.com/photos/event/2/6/4/0/highres_312669792.jpeg" width="933" height="697" /></a>
      </p>
      
      <p>
        <p class="wp-caption-text">
          Christian's and Nikolay's implementations in action. Green words are solved. Red words are being played.
        </p></div> 
        
        <p>
          And while the code produced in such a rush would not have won a beauty contest (next dojo will focus on clean and idiomatic Scala), <strong>we had a lot of fun together</strong>. I certainly also had fun implementing the server, and learnt a lot (Akka, Play web-sockets) from the <a title="Backend for the Akka-worshop of the SVUG" href="https://github.com/clashcode/clashcode" target="_blank">code it was based on</a>.
        </p>
        
        <p>
          If you are part of a Scala user-group, or just want to teach Scala, feel free to use/clone these projects (server/client word-guessing) if you also want to have fun word-guessing:
        </p>
        
        <blockquote>
          <p>
            <a href="https://github.com/scala-vienna/wordguess-server" target="_blank">https://github.com/scala-vienna/wordguess-server</a>
          </p>
          
          <p>
            <a href="https://github.com/scala-vienna/wordguess-client" target="_blank">https://github.com/scala-vienna/wordguess-client</a>
          </p>
        </blockquote>