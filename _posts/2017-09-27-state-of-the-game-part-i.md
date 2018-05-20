---
title: State of the Game Part I
date: 2017-09-27T20:55:31+00:00
categories:
  - DKP
---
I&#8217;ve mainly spent time implementing core mechanics so what you&#8217;re about to see isn&#8217;t flashy but considering I&#8217;ve started with an empty scene and no Unit experience (besides the official tutorials) I&#8217;m happy with the current state of the game. If you don&#8217;t know what DKP is about, check out [Fucking 50 DKP Minus: Raiders of the Lost Content.](http://manuel-huber.de/2017/09/25/fucking-50-dkp-minus-raiders-of-the-lost-content/) Here&#8217;s what I got:

#### Selection & Movement

You can select unity via mouse clicking, selection dragging and hotkeys. The movement system is based on the unity navMesh, so I didn&#8217;t need to implement any pathfinding myself. I spent a lot amount of time on the hierarchy. Who checks for click actions, who manages movement and such. It&#8217;s all inner-workings software engineering stuff that you won&#8217;t notice in game but that I will notice when I fix / change something. Basically there is a MainControl singleton that is responsible for managing the current unit selection, checks for mouse and keyboard input in the Update function and then hands those events off to selected units. Then the Hero can decide what he does with the click event (is it for movement or am I casting a spell etc.). I&#8217;ve made most of my stuff generic. I don&#8217;t work directly with my PlayerCharacterController but with a MouseControllable that receive generic click events. Maybe someday I want to make something mouse controllable that&#8217;s not a character&#8230; or maybe I&#8217;m spending too much time pointlessly making things generic &#8211; who knows? So far I&#8217;m in love with C#. I come from a Java and Typescript background and C# feels like a safe, strongly typed language that offers very elegant functional aspects. It&#8217;s fun!

<div style="width: 640px;" class="wp-video">
  <!--[if lt IE 9]><![endif]--><video class="wp-video-shortcode" id="video-110-1" width="640" height="360" loop="1" autoplay="1" preload="metadata" controls="controls"><source type="video/mp4" src="http://manuel-huber.de/wp-content/uploads/2017/09/2017-09-25-20-20-34.mp4?_=1" />
  
  <a href="http://manuel-huber.de/wp-content/uploads/2017/09/2017-09-25-20-20-34.mp4">http://manuel-huber.de/wp-content/uploads/2017/09/2017-09-25-20-20-34.mp4</a></video>
</div>

&nbsp;

I&#8217;ve decided to split this into 3 parts since after half a day of working on the game and then working a full day I can&#8217;t muster up the strength to write more. And I hope shorter articles are more likely to actually get read. Next up tomorrow: Attack & Health
