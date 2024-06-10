---
title: 'WiFi Controlled RC Car'
author: '**Alexander Kogan and Jonathan Vazquez** (website template by Ryan Tsang)'
date: '*EEC172 WQ24*'

subtitle: '<blockquote><b>EEC172 Final Project Webpage Example</b><br/>
Note to current students: this is an <i>example</i> webpage and
may not fulfill all stated requirements of the current quarter''s 
assignment.<br/>The website source is hosted 
<a href="https://github.com/ucd-eec172/project-website-example">on github</a>.
</blockquote>'

toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'Our project is a remote controlled RC car that communicates over TCP via a WiFi hotspot. Instead of a typical remote that uses levers or buttons to control the direction of the car, the remote instead is the accelerometer on a Ti CC3200 board. In other words, in order to move the car, you would have to tilt the Ti CC3200 board in the direction you want it to move. The actual car is a kit from DIYables that comes with 2 DC motors, a chassis with wheels, and a battery pack to power the motors. In addition, a L298N driver module needs to be acquired in order to control the motors. Typically the car and the remote would communicate via IR or bluetooth but we have instead opted to communicate over TCP because the CC3200 board does not support bluetooth or IR communication. Because of this, there is one CC3200 board that is physically controlling the car and another CC3200 board that is acting as the controller in order to be able to control the RC car without wires needing to be attached. The L298N driver is also able to power the CC3200 board that it is attached to because the battery pack has a total capacity of 6V. The motors and the CC3200 board that controls them are both being powered by the same source which lowers how many power sources we need on the car chassis. In order for the CC3200 boards to communicate over TCP, they needed to know the IP address of the other. The C3200 that controls the car posts its IP address to AWS IoT so that the controller CC3200 can retrieve it and establish a connection. The controller board also has an OLED screen that displays the accelerometer data of the board on the RC car. Information being displayed means that both boards are successfully sharing information with each other. If switch 3 on the controller board is pressed then it stops sending data to the RC board, acting as if the car has stopped.
<br/><br/>

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;padding-top:20px">
  <div style="display: inline-block; vertical-align: bottom;">
    <img src="./media/image.jpg" style="width:auto;height:4in"/>
    <!-- <span class="caption"> </span> -->
  </div>
</div>

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe style="left:0;top:0;width:100%;height:100%;position:absolute;" width="560" height="315" src="https://www.youtube.com/embed/wSRtnAEZhmc?si=3vQXNj4h0WkW-F-q" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
'
---

<!-- EDIT METADATA ABOVE FOR CONTENTS TO APPEAR ABOVE THE TABLE OF CONTENTS -->
<!-- ALL CONTENT THAT FOLLWOWS WILL APPEAR IN AND AFTER THE TABLE OF CONTENTS -->

# Design

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
    Both boards send their own IP address to the IOT shadow, then read the other’s IP address from the shadow, and finally connect to each other over TCP. The controller board then sends its accelerometer data over to the RC board and awaits accelerometer data from the RC board. The RC board is waiting for accelerometer data from the controller board and once it receives it, the RC board sets the motors in the direction the accelerometer data tilted in. Once that is done, the RC board sends its own accelerometer data to the controller board and then awaits for more accelerometer data from the controller board. Once the controller board receives accelerometer data from the RC board, it displays that data on the OLED. Then it sends its own accelerometer data back to the RC board in a never ending loop. If the switch on the controller board is pressed then it gets out of the loop so that the car stops and if it is pressed again then the loop continues.
  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 400px;">
    <div class="fig">
      <img src="./media/block.png" style="width:90%;height:auto;" />
      <span class="caption">System Flowchart</span>
    </div>
  </div>
</div>

## System Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 300px;">
    There are 3 main components to the project, the network, controller, and RC car component. The controller component is in charge of sending its own accelerometer data to the RC car component and displaying the accelerometer data of the RC car component. The RC car component is in charge of setting the motors in the direction according to the accelerometer data of the controller component and sending its own accelerometer data to the controller component. The networking component has two main parts. The first part is the IOT shadow which stores the IP addresses of the controller component and RC car component so they can each read each other’s IP addresses. While both the controller and RC car components incorporate the networking component, they each are more focused with the physical aspects of each component. 
  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 500px">
    <div class="fig">
      <img src="./media/state.png" style="width:90%;height:auto;" />
      <span class="caption">State Diagram</span>
    </div>
  </div>
</div>

# Implementation

### Building

The car first has to be put together, according to the kit provided. Each motor needs to be connected to the L298N driver module in the output pins. The battery pack requires 4 AA batteries and needs to be connected to the L298N driver module in the power pins. Four GPIO pins on the CC3200 board will need to be connected to the direction control pins of the L298N driver module. Once the software is flashed onto the RC car board, the board power input pins will need to be connected to the L298N driver module in order to power the CC3200 board. Both boards need the I2C jumpers set so they can receive accelerometer readings. SPI also needs to be set up to communicate with the Adafruit OLED display.

### Networking

This component first requires setting up an Amazon IOT shadow that both boards will connect to. Almost as if the shadow represents two real world objects. After connecting to a WiFi hotspot named “rccontrol”, the board controlling the car needs to post its IP address on the shadow so that the controller knows the car’s IP address. After the car board posts its IP address, it simply listens for a TCP socket connection. Meanwhile, the controller retrieves the IP address of the car and then 

### Controller

Once the controller has the connection to the RC car, it sends its own accelerometer data to the RC car and then awaits the RC car to send the RC car accelerometer data. Once it receives the RC car accelerometer data, it should then be displayed on the OLED. The data can be displayed on the OLED however one wants. This can be by just displaying the data raw as a string or can be more creative such as drawing a car on the OLED in the direction the car is moving. Once the data is displayed, the controller can do two things. One option is to repeat the previous parts in a loop, starting when it sends its own accelerometer data to the car. The option is when switch 3 is pressed down the controller should loop forever doing nothing so that it acts as if the car is “off” until the switch is pressed again which restarts the main loop.

### RC Car

Once the RC car has the connection to the Controller, it awaits accelerometer data from the controller. Once it has received it, it needs to set the GPIO pins to the necessary settings in order to move the motors in the direction the controller wants to. The direction can be decided however one wants forward to be. There are 4 control pins on the L298N driver module. Two pins control one motor and the other two pins control the other motor. In order to move them, one of the pins needs to be set to high and the other needs to be set to low, changing direction depending on which one is high. To stop the motors, both pins need to be high or both need to be low. Once the motors are set to the desired setting, the RC car board then sends its own accelerometer data to the controller board and restarts the loop by awaiting for new accelerometer data from the controller board. The intensity of voltages supplied to each motor can also be optionally modulated by setting the GPIO pins to high for a portion of a small duty cycle, where the larger the portion the greater the average power supplied to the motor will be. We opted for a 1ms duty cycle where after every 10 ns, a systick handler sets the GPIO pins to high or low depending on the X and Y inputs from the controller such that the car can be controlled arbitrarily.

# Challenges

## Car Construction

One of the first challenges we encountered was difficulties with building our car. We encountered problems using the battery pack that came with the kit, which seemed to be mysteriously broken the first time we used it and then seemed to somehow work after testing it thoroughly. We also broke the wiring on one of the motors which required us to borrow a motor from the lab room, which fortunately had the exact same models available.

## TCP Networking/Establishing Connections

Another difficulty was in configuring the TCP connection process. Even though an example project exists already in the CC3200 SDK, it still required some careful thought and workarounds for ensuring that connections are eventually established, including mechanisms for waiting for the other board to connect and gracefully handling bad initial connections. In fact, we went through several different implementations for the actual synchronization process, but eventually settled on the one we use with AWS IoT and the car as the TCP host so to speak.

# Future Work

The main future component that needs to be implemented is a way to have the two boards reconnect if they ever lose connection. Right now, if either board loses connection completely then the entire system just stops working. Some more fine tuning of the voltage management would also be nice, as the car feels a bit too sensitive to certain motions of the controller. Finally, a mechanism that uses the IR remote and receiver provided in class would provide an interesting means for connecting to any WiFi that’s available, instead of hardcoding the AP name into the networking code.

# Finalized BOM

<!-- you can convert google sheet cells to html for free using a converter
  like https://tabletomarkdown.com/convert-spreadsheet-to-html/ -->

<table style="border-collapse:collapse;">
<thead>
  <tr>
    <th><p>No.</p></th>
    <th><p>PART NAME</p></th>
    <th><p>DESCRIPTION</p></th>
    <th><p>Qty</p></th>
    <th><p>SUPPLIER / MANUFACTURER</p></th>
    <th><p>UNIT COST</p></th>
    <th><p>TOTAL PART COST</p></th>
    <th><p>Purpose</p></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><p>1</p></td>
    <td><p>CC3200-LAUNCHXL</p></td>
    <td><p>MCU Evaluation Board</p></td>
    <td><p>2</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$66.00</p></td>
    <td><p>$132.00</p></td>
    <td><p>RC Car and Controller</p></td>
  </tr>
  <tr>
    <td><p>2</p></td>
    <td><p>Adafruit 1431 OLED</p></td>
    <td><p>128x128 RGB OLED Display. SPI protocol</p></td>
    <td><p>1</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$39.95</p></td>
    <td><p>$39.95</p></td>
    <td><p>Display accelerometer readings from car</p></td>
  </tr>
  <tr>
    <td><p>3</p></td>
    <td><p>DIYables RC 2WD Car Chassis Kit</p></td>
    <td><p>RC Car works with Arduino, ESP32, ESP8266, Raspberry Pi, or any 5V or 3.3V microcontroller</p></td>
    <td><p>1</p></td>
    <td><p>Purchased from Amazon</p></td>
    <td><p>$13.99</p></td>
    <td><p>$13.99</p></td>
    <td><p>The physical chasis, motors, wheels, and batteries for the RC car</p></td>
  </tr>
  <tr>
    <td><p>4</p></td>
    <td><p>L298N Motor Driver Controller</p></td>
    <td><p>ShillehTek L298N Motor Driver Controller Board Module for Stepper Motor & DC Dual H-Bridge</p></td>
    <td><p>1</p></td>
    <td><p>Purchased from Amazon</p></td>
    <td><p>$6.89</p></td>
    <td><p>$6.89</p></td>
    <td><p>Sends power to the left and right motors and the RC car's CC3200</p></td>
  </tr>
  <tr>
    <td colspan="3">
      <p>TOTAL PARTS</p></td>
    <td><p>5</p></td>
    <td colspan="2">
      <p>TOTAL</p></td>
    <td><p>$192.38</p></td>
    <td></td>
  </tr>
  <tr>
    <td colspan="3">
      <p>TOTAL PARTS (Excluding Provided)</p></td>
    <td><p>2</p></td>
    <td colspan="2">
      <p>TOTAL (Exluding Provided)</p></td>
    <td><p>$20.88</p></td>
    <td></td>
  </tr>
</tbody>
</table>