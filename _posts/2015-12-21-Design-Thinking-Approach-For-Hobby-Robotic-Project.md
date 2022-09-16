---
layout: post
comments: true
title: Design Thinking Approach For Hobby Robotics Project
author: malike
categories: [electronics]
tags: [documentation,sample]
image:
 path: /posts/new_pet.png
 width: 800
 height: 500
alt: .
redirect_from: "/Design-Thinking-Approach-For-Hobby-Robotic-Project/"
---

I work at [DreamOval](http://dreamoval.com) and we were privileged to be part of the [Stanford Seed](https://seed.stanford.edu/)
training this year. One of the highlights of the training was **Design Thinking**.

What is  [design thinking](https://dschool.stanford.edu/sandbox/groups/designresources/wiki/36873/attachments/74b3d/ModeGuideBOOTCAMP2010L.pdf)?.
*(follow link)*

I wanted to apply design thinking to a hobby project. A hardware one. It shouldn't be too simple. This automatically takes out the *hello world* of hardware projects,blinking LED, out of this. 
Definitely not a *mars rover* too. So this is an *in-between kinda project*. But closer to the blinking
LED.

We are going to design,code,build a simple robot. But let me add that the real focus of this is not the project itself. But ***the processes involved in getting this done***.

It would be a 4-part *(maybe 5)* series. This is the first part of the series.

Personally when it comes to my robotic projects. I like to see what I plan to have in the end before 
I start. Some people prefer to design their circuits before knowing where to place each component that
makes their build. But having a design in mind helps me know what components to choose. 
An example is, if after design I realise I need to build a robot where its size matter during the 
design process I'll rather I pick ATmega328 mcu compared to an arduino.


The design process is the most difficult part for me. Why? because this is the part you see the end 
from the beginning. Most hobby project tend to be open ended because we get carried away and run wild with features and upgrades. Using design thinking approach helps in solving this. 

![_config.yml]({/posts/design-thinking.png) 

***1. Empathy***

Empathy involves us sharing the feeling of another.This makes this part a bit tricky because we are
about to share our feeling with ourselves. If you are working in groups you can pick partners.

After interviewing myself. This is what I gathered. Its ok if your results vary.
 
    * This is a hobby. 
	* Budget is very small. 
	* Robot should be able to move and see its environment. 
	* User (I) should be able to see what the robot sees.
	* I should be able to control the robot.
	* Robot should be able to move on its own.
	* Again, budget is really small.
	* Robot should move around easily on its own in 
	  a known or unknown environment.

Thats about it. Lets move on to the next step.	

***2. Define***

Now using our interview results we can define what the user wants. We should be able to see the user's
real problem. Defining it brings us a step closer to solving the real problem for the user.This can be iterative process.


	* Robot should use relatively cheap components.
	* Robot should be able to move on its own.
	* Robot should move by acting on commands from user.
	* User should "see" what robot "sees".
	* Robot should not be  huge.
	* Robot should be able to "think" on its own. 
	

***3. Ideate***

This is where we find possible solutions to the problem. Building up on what we gathered from defining the 
problems/solutions. 

Based on what we defined I was able to create 5 categories. The one section I didn't gather from interviewing myself is power source,I added that. Its not always necessary to list out categories.
List out all the ideas that come to mind here for creating the solution. Then strike out the ones that 
won't work. 

{::nomarkdown}

<div class="stickynote">

<ul style="width:150%;margin-left:-150px;">
    <li>
      <span>
        <h2>Power Source</h2>
        <p>5V battery that can be changed.</p>
        <p class="strike-through">Solar ?</p>        
      </span>
    </li> 
    <li>
      <span>
        <h2>Sense & Sight</h2>
        <p class="strike-through">Camera for sight</p>
        <p>UltraSonic Sensor would be used (just like bats)</p>
        <p class="strike-through">Send realtime stream from camera over Wi-Fi to user</p>
        <p class="strike-through">Identify objects by "remembering" them</p>
        <p class="strike-through">Tag objects by shape and/or color</p>
      </a>
    </li>      
    <li>
      <span>
        <h2>Movement</h2>
        <p class="strike-through">Wheels for movement</p>
        <p class="strike-through">Tank treads for movement</p>        
        <p>Metal bended as legs</p>
        <p class="strike-through">Fly-- mini drone/quad or tri -copter</p>        
      </span>
    </li>
    <li>
      <span>
        <h2>Interaction</h2>
        <p>Ultrasonic sensor to measure distance from objects</p>
        <p class="strike-through">Can move objects in its path</p>
        <p>Send relative position of objects to be mapped</p>
        <p class="strike-through">Should be able to jump or fly over objects in its path</p>
      </span>
    </li>
    <li>
      <span>
        <h2>Intelligence</h2>
        <p class="strike-through">Should be able to remember paths already taken  to pick best 
          in terms of distance on another occasion</p>
        <p class="strike-through">Identify objects by "remembering" them</p>
      </span>
    </li>
    <li>
      <span>
        <h2>Control</h2>
        <p class="strike-through">Send and receive signals via Wi-Fi</p>
        <p> - via Bluetooth</p>
        <p> - via ?</p>        
      </span>
    </li>
   </ul>
  </div> 
{:/}



After striking out the ones not necessary based on requirements gathered
under the sections, ***Empathy*** and ***Define***. 

I had to remove most of the ideas because I wanted to keep the budget really small.
The best approach however is to prototype each of the ideas preferably in groups.  

***4. Prototype***

Using generated ideas we can build examples of the robots product and explain how it works.
I use [SketchUp](http://www.sketchup.com/) for building my prototypes. Use whatever you are comfortable with. If you have a 3D printer use it. Unfortunately I don't.


<iframe width="560" height="315" src="https://www.youtube.com/embed/mt5zsMHGz4k" frameborder="0" allowfullscreen></iframe>

<br/>

> Next up we'll continue working on the ***prototype*** section.<br/> 
 Perfect design in SketchUp.<br/>
 Pick  electronic components,design and program test robot.<br/> 
 Simulate build and test robot in an environment.<br/>
 Putting it together,build and ***test***.<br/><br/>
 [Part 2 ](http://malike.github.io/Design-Thinking-Approach-For-Robotic-Hobbyist-2)


