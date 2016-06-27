---
layout: post
title: Diving Into Arduino
categories: arduino
---
Last month I came a cross a video of a small dancing robot called [OTTO](https://create.arduino.cc/projecthub/cparrapa/otto-build-you-own-robot-in-two-hours-5f2a1c). What makes OTTO special for me is that it is completely OpenSource. I am a professional software developer and I work with OpenSource software for years. But this little robot reminded me that **hardware** can also be open sourced. You can build a clone of this cute robot at home in a day : 3D print the moving parts, buy the required electronics, download the software and build the robot by reading the construction manual.

<iframe width="420" height="315" src="https://www.youtube.com/embed/ZcGgz6v1efg" frameborder="0" allowfullscreen></iframe>

The brain of this robot is powered by an [Arduino Nano](https://www.arduino.cc/en/Main/ArduinoBoardNano) board. Arduino is an open source prototyping platform. An Arduino board can read inputs and turn these inputs into  outputs. For example it can read commands over bluetooth and turn these commands to outputs that activate a motor. **Arduino** is a trade mark name but the hardware is open source. Anyone can freely copy the design and sell their own clones. 

Last month I ordered a [Starter Kit For Arduino](http://www.aliexpress.com/item/UNO-KIT-Upgraded-version-of-the-For-Starter-Kit-the-RFID-learn-Suite-Stepper-Motor-ULN2003/1082495572.html) from [AliExpress](http://www.aliexpress.com). My 28$ starter kit from China has a UNO board which is a clone of the original [Arduino UNO](https://www.arduino.cc/en/Main/ArduinoBoardUno).

![arduino starter kit](/assets/diving_into_arduino/arduino_starter_kit.jpg){: .center-image }

After reading the documentation and some sample projects I am able to turn on and off LEDs attached to breadboard. I think blinking a LED is the **Hello World** of programming an Arduino board. I am a software developer, the only thing I remember about electronics is the [Ohm's Law](https://en.wikipedia.org/wiki/Ohm%27s_law). **V = I * R**. If you are also new to electronics like me you can virtually design your circuits and test your code by simulation on **[circuits.io](http://circuits.io)**. Below is an Arduino "Hello World" example:

<iframe frameborder='0' height='448' marginheight='0' marginwidth='0' scrolling='no' src='https://circuits.io/circuits/2294317-led-blink/embed#breadboard' width='650'></iframe>

An Arduino project has mainly two parts:

1. Hardware and electronics (which I have a little understanding)
2. Software (this is where I am good at.)

Here is a simple Arduino program that blinks a LED attached to input pin 9 of an Arduino Uno board:

{% highlight cpp %}
// Pin 9 has an LED connected
int led = 9;

// the setup routine runs once when you press reset:
void setup() {
  // initialize the digital pin as an output.
  pinMode(led, OUTPUT);
}

// the loop routine runs over and over again forever:
void loop() {
  digitalWrite(led, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);               // wait for a second
  digitalWrite(led, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);               // wait for a second
}
{% endhighlight %}

An Arduino program has two main functions:

1. void **setup**()
2. void **loop**()

The **loop function** is very similar to [game loop](http://gameprogrammingpatterns.com/game-loop.html). Which is the heart of every digital game on this planet. A game loop:

* runs continuously
* processes user input (each turn)
* updates the state of the game
* renders the game

If we apply the game loop concepts to an Arduiono program the loop function can be writen as a game loop:

* Arduino program runs continouosly, until we press the reset button or the cut the power off.
* On each turn we can process user input. For example read the input pins and detect if the user pressed a button. Read an analog signal from an analog input pin to which a sensor is attached.
* Update global state of the program. For example update the counters
* Write to digital or analog output pins, so we can control the attached peripherals. (Turn LEDS on, write text to LCD screen, activate motors)

I am trying to write a C++ library that will simplify the loop function. The library will:

* act as an event handler. I will be able to attach functions to specific input events.
* simplify the implementation of timed actions. For this I am planning to implement something similar to [Libgdx Scene2d Actions](https://github.com/libgdx/libgdx/wiki/Scene2d#actions) Here is what I mean:

{% highlight cpp %}
void setup() {
  looper.addAction(parallel(foo(), bar()));
  looper.addAction(forever(sequence(foo(), bar(), delay(1000))));	
}

void loop() {
  looper.loop();
}
{% endhighlight %}

I am going to share a more detailed example when this library called "Looper" is finished. 
