+++ 
date = "2020-01-26"
title = "Arduino Obstacle Avoiding Car Bot"
tags = ["arduino", "robotics", "threading", "protothreads"]
categories = ["blog"]
+++

A few weeks ago I described my families new years celebration, building a arduino car bot that we used to have a family dance-off.  We've been playing around with the arduino bot, and I decided to make it  drive and avoid obstacles autonomously.

The bot has a servo at the front, with a ultrasonic distance sensor attached to it.  The servo turns the sensor in 180 degrees to effectively from side to side.  This gives the ability to roughly judge distance objects by the robot.

The ultrasonic sensor is an [HC-SR04](https://www.digikey.com/catalog/en/partgroup/hc-sr04-ultrasonic-sonar-distance-sensors/82205?utm_adgroup=xGeneral&utm_source=google&utm_medium=cpc&utm_campaign=Dynamic%20Search&utm_term=&utm_content=xGeneral&gclid=EAIaIQobChMIypK7sdai5wIVh7zACh2brAr2EAAYASAAEgKY3fD_BwE)

The servo is an [SG90](https://www.amazon.com/Smraza-Helicopter-Airplane-Control-Arduino/dp/B07L2SF3R4/ref=sr_1_1_sspa?keywords=sg90&qid=1580104198&sr=8-1-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUE3WEVOQzIwUTNUR0QmZW5jcnlwdGVkSWQ9QTA4MjM1NTlKSUFFS1ZRQTYxTkUmZW5jcnlwdGVkQWRJZD1BMDIwNzMwN0dRSDFXTkhNVURFNCZ3aWRnZXROYW1lPXNwX2F0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=).

The algorithm to make the robot work was roughly this.

1. Move forward.
2. Check for obstacles from 30-150 degrees, where 90 degrees is pointing forward.
3. If an obstacle is within 30cm, stop.
4. Take distance readings in the area from 0-180 degrees, and look for the closest point under 50 cm.
5. If one is found, turn away from that obstacle and then go back to step 1.
    a. Where turning away means, if the obstacle is on the right side, turn left, and vice versa.

The algorithm took some tweeks especially around the angles to scan, and the distances.  Overall it works.  It will get stuck from time to time on small objects.  It would likely require a higher resolution distance sensor, or a contact sensor to avoid that.  It also has a problem in confined spaces, but lowering some of the distance thresholds, and speed would help in that case. 

There were a few challenges implementing this.

I had a few problems with the ultrasonic sensor.  After online research and review of the library's code I was using, I found the measurements weren't working or implemented as well as I had hoped.  I found an alternate library named NewPing, and it does an excellent job and the distance measurements were much more reliable.  There are a number of libraries available for arduino for the HC-SR04, all with varying levels of quality, just another reminder to be careful of dependencies.

Debugging was a challenge at times.  Implementing a physical indicator of when the robot detects something really aided the development.  For example, when the bot is trying to look for the closest thing to it, after it's done the servo will turn to point the sensor directly at that closest object.

I've been using the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) for development on my laptop.  I really enjoy being able to use the linux command line and tooling, and it also feels like I have a separated area where I can code.  Key to this has been VS Code and it's ability to remote into WSL to develop.  It is really slick, and with WSL2's immanent release, it's only going to get better.

The only downside to using WSL, is I've noticed odd behavior on the serial ports.  WSL does map the arduino connected to USB to one of it's tty devices.  However, I have issues connecting straight to the arduino through WSL.  First in a windows arduino IDE (not in WSL), I need to open Serial Monitor to establish a question.  After that has been done, then I can upload the program from the arduino extension from VS code living in WSL.  I don't know why it's a problem yet, but am hoping to figure it out sometime.

