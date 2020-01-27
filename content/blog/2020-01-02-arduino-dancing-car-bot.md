+++ 
date = "2020-01-02"
title = "Arduino Dancing Car Bot"
tags = ["arduino", "robotics"]
categories = ["blog"]
+++

My kids received a robotics kit for Christmas.  [Here's an Amazon link to a similar car](https://www.amazon.com/LAFVIN-Include-Ultrasonic-Bluetooth-Tutorial/dp/B07JN46YSW).  It was a fun project to put together to explain some of the inner workings of robotics.  My oldest has been working with scratch, so he really was interested in what 'real code' looks like.

It also gave me an excellent way to revisit C/C++ programming as it had been a while since I had done any projects in it.

With the project, we did a dance-off.  Each member of our family came up with a robot dance. I had given them 5 different instructions they could give the robot.

* Move Forward
* Move Back
* Turn Left
* Turn Right
* Wait

I also told them they could loop any of those instructions.
The dance in total had to take less than 30 seconds. 

I helped by translating their sequence of instructions into code. Here's the sample code for a dance.  It reminded me a lot of writing LOGO when I was a kid.

```cpp
_car->setSpeed(100);

for (int i = 0; i < 10; i++)
{
    _car->moveForward();
    delay(500);
    _car->turnLeft();
    delay(500);
    _car->turnRight();
    delay(500);
    _car->turnLeft();
    delay(500);
}

_car->stop();
```

The project code is availble on GitHub - [arduino-bot](https://github.com/jerhon/arduino-bot) take a look at the Danceoff.cpp file.  I also made some helper classes around the motor controller.
