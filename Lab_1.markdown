---
layout: page
title: Lab 1
permalink: /Lab_1/
---
Lab 1 was an introduction to using the artemis board with the arduino ide and showing off various capabilities of the board

## LED
The first example was to use the simple blink example used in most microcontrollers. In this video, the LED can be seen blinking.
<video width="320" height="240" controls>
  <source src="../videos/Lab_1_blink.webm" type="video/webm">
  Your browser does not support the video tag.
</video>

## Temperature sensor
The next example showed off the temperature sensor on the artemis board. In the video, the second collumn of the serial monitor can be seen displaying a higher temperature value when the artemis board is pressed up against the warm surface of the laptop. The temperature slightly decreases when blowing on it.
<video width="320" height="240" controls>
  <source src="../videos/Lab_1_Temp.webm" type="video/webm">
</video>

## Serial communication
The next example showed off the uart communication between the artemis board and the host computer. The serial shows the program having the artemis board send an initial message. Then, when the user inputs a number or string into the serial monitor, the artemis board echos the sent message back through the serial monitor.
<video width="320" height="240" controls>
  <source src="../videos/Lab_1_Serial.webm" type="video/webm">
</video>

## Microphone
The next example shows off the ability to use the artemis board's microphone as well as the ability to use an FFT to do a frequency analysis on the microphone data. Through the serial monitor, the artemis board can be seen displaying the maximum amplitude frequency in the data received from the microphone. When I induce a high-frequency whistle. The maximum frequency raises on the serial monitor in response to the whistle
<video width="320" height="240" controls>
  <source src="../videos/Lab_1_Microphone.webm" type="video/webm">
</video>

This is info on lab 1
