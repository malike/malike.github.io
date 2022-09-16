---
layout: post
comments: true
title: Design Thinking Approach For Hobby Robotics Project - Part 2
categories: [electronics]
author: malike
tags: [documentation,sample]
image:
  path: /posts/the_three_laws_of_robotics.png
  width: 800
  height: 500
alt: .
redirect_from: "/Design-Thinking-Approach-For-Robotic-Hobbyist-2/"
---


Ok, this is the long overdue part two of where we left [off](http://malike.github.io/Design-Thinking-Approach-For-Hobby-Robotic-Project). What are we doing today? <br/>

    1. Give it a name

    2. Components from prototype

    3. Program 

    4. Simulation


### 1. Give it a name

Because its movement is modeled after a bug we *(I)* should call it **Bug1**. 

### 2. Check prototype and pick components.

If this is not your first electronic project you probably would have identified some components in the prototype. Like the Ultrasonic Sensor and the servos. Two servos one for the front and rear legs for 
movement.

*New to the hobby [servo](https://www.princeton.edu/~mae412/TEXT/NTRAK2002/292-302.pdf)? What about the [HC-SR04 ultrasonic sensor](http://www.elecfreaks.com/store/download/product/Sensor/HC-SR04/HC-SR04_Ultrasonic_Module_User_Guide.pdf)?*


**i. Front Legs with a Servo**

![_config.yml](/posts/front_legs.png) 

Bug1 uses two hobby servos for movement. With the arduino as the command. But the position of the two servos 
to give the required movement is not that straight forward. 

Honestly [this](http://www.eucognition.org/index.php?page=leg-coordination-in-insect-walking) was what I wanted but the point of this post is to focus on the methodology not the actual project. I had to create a constant reminder of this because I kept drifting away.

From the diagram above, if the part labelled **2** (front right leg) moves forward that is 120 degrees **3** (front left leg)would be 60 degrees. The back and forth of this movement would push *Bug1* forward.

This servo can be controlled to steer *Bug1* as well. The front servo is mainly responsible for how Bug1 moves and where it turns. <br/>


{::nomarkdown}

<div>
<div style="width:30%;float:left;">

<img src="/posts/servo.jpg" alt="servo"/>

</div>
<div style="width:68%;float:right;">

<blockquote> <b>Specifications</b><p>&nbsp;</p>

	Operating Speed: 0.17sec / 60 degrees (4.8V no load)<br/> 
Operating Speed: 0.13sec / 60 degrees (6.0V no load)<br/> 
Stall torque:12kg/cm(6V)<br/>
Operation Voltage : 4.8 - 7.2 V <br/>
Temperature range: 0°C to 55°C<br/>
Power Supply: Through External Adapter<br/>
Length 300mm<br/>

</blockquote>

</div>
</div>

<p style="clear:both;">&nbsp;</p>
{:/}


**ii. Rear Legs with a Servo**


![_config.yml](/posts/rear_legs.png) 

The servo controlling the rear legs is responsible for supporting Bug1. This enables it to have good composure
when moving or not. Think of it as the second pair of legs designed to make Bug1 *stand* like a table. Its placed such that its movement is restricted to just an up and down movement.


**iii. Eyes with the HC-SR04 UltraSonic Sensor**

![_config.yml](/posts/ultrasonic_sensor.png) 

The UltraSonic sensor helps *Bug1 see* and *sense* the environment to help in the way it reacts to the environment.

{::nomarkdown}

<div>
<div style="width:35%;float:left;">

<img src="/posts/ultra.png" alt="HC-SR04"/>

</div>
<div style="width:63%;float:right;">

<blockquote> <b>Specifications</b><p>&nbsp;</p>
  Power Supply :+5V DC<br/>
  Working Current: 15mA<br/>
 Ranging Distance : 2cm – 400 cm/1″ – 13ft<br/>
Measuring Angle: 30 degree<br/>
Trigger Input Pulse width: 10uS<br/>
Dimension: 45mm x 20mm x 15mm<br/>	
</blockquote>

</div>
</div>

<p style="clear:both;">&nbsp;</p>
{:/}


It works simply by sending an ultrasound of about 40Khz and then listens for the pulse to echo back in time,t. 
We can calculate the distance of the object from the sensor (*Bug1*) as


>Distance from object (m) = 340 (m/s) * t(s) / 2 

*Because the speed of light is approx 340 m/s at room temperature*

**iv. [Bluetooth Receiver](http://cdn.sparkfun.com/datasheets/Wireless/Bluetooth/Bluetooth-RN-42-DS.pdf)**

Although not shown in our prototype this is the connection between our *command center* and the *Bug1*. We can send and recieve data from *Bug1*.

{::nomarkdown}

<div>
<div style="width:30%;float:left;">

<img src="/posts/12577-01.jpg" alt="servo"/>

</div>
<div style="width:68%;float:right;">

<blockquote> <b>Specifications</b><p>&nbsp;</p>

Extremely small radio - 0.15x0.6x1.9"<br/>
Very robust link both in integrity and transmission distance (18m)<br/>
Hardy frequency hopping scheme - operates in harsh RF environments like WiFi, 802.11g, and Zigbee<br/>
Encrypted connection<br/>
Frequency: 2.402~2.480 GHz
Operating Voltage: 3.3V-6V<br/>
Serial communications: 2400-115200bps<br/>
Operating Temperature: -40 ~ +70C<br/>
Built-in antenna<br/>
Dimensions: 45mm x 16.6mm x 3.9mm<br/>

</blockquote>

</div>
</div>

<p style="clear:both;">&nbsp;</p>
{:/}



**v. *"Sticky"* Legs**

Well *Bug1* has to move right?. With the help of metal bended legs we can make this happen.

**vi. [Arduino Uno](https://www.arduino.cc/en/Main/ArduinoBoardUno)**


*Bug1* needs to process.

**vi. Batteries**

Since we using an Arduino any 5V battery would do.

### 3. Program

*Bug1* is programmed using **C**.The codes and other files for the project can be found [here](https://github.com/malike/Bug1). But note that changes can be made to when we start building the project.

<script src="https://gist.github.com/malike/c0ca72deab3c71e479244d9a69995841.js"></script>


### 4. Simulation.

Before we jump in soldering and building *Bug1* lets first run a simulation to see how our code would run. Fortunately for us there are tons of tools to run this simulation which you can find [here](http://smashingrobotics.com/arduino-simulators-lineup-start-developing-without-real-board/). I chose Protues.

If you decide to also use Protues check [this](https://github.com/malike/Bug1/tree/master/Proteus)  or just use the [codes](https://github.com/malike/Bug1/tree/master/Bug1) with any simulation tool.

We can safely test our codes and wiring with ease.
[![_config.yml](/posts/Bug1.PNG)](https://xkcd.com/1613/) 


If you've  ever used Protues you'd know that some components do not come out of the box. But you can get them online. 

I'm using this [Arduino - Protues](http://blogembarcado.blogspot.com/) and [HC-SR04 Ultrasonic Sensor - Protues](http://blogembarcado.blogspot.com/). 


> Next up we get to build *Bug1*. <br/> Hopefully the components I ordered would arrive by then
  Simulate the serial communication  and automated movement of *Bug1*
  <br/>Check out proejct files on [GitHub](https://github.com/malike/Bug1). 
