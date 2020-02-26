+++ 
date = "2020-02-25"
title = "The Pi-plate: An Interactive Raspberry Pi Driven Nameplate"
tags = ["networking", "pi-hole", "dns", "raspberry-pi" ]
categories = ["blog"]
+++

I've recently been thinking of a few different projects around my home to build some sort of "kiosk" to display information.  Am example of this is a display for some SmartThings sensors I have around the house, another is calendar information by the door instead of the traditional paper calendar we have.  Before I go all in and try to accomplish any of these, I wanted to build a basic proof of concept.

So,  I decided to make a Raspberry PI driven kiosk.  This PoC is a nameplate for my desk, which I call a pi-plate.  It's a 7" display driven by a Raspberry Pi.  In addition to being a nameplate for my desk, it will be digital signage at my work desk testifying to my undeniable nerdiness.  I also wanted it to be fun. The screen is a capacitive touch screen, so I added a simple polling application that asks folks random questions.  For example... Who is your favorite disney princess?  Would you rather drink Pepsi or Coke?  But the serious goal of this PoC is to get people talking about technology-- and my currently low tech non-digital nameplate is out of date.

I'll be writing a number of posts over the coming weeks about how this was built.  Some design decisions that I had to make to get it working, and a few the rough spots I had to overcome.

Overall it was a fun little project, and most of the code was done mostly over some downtime during a weekend and a few weeknights.  Physical assembly of the it was completed over the weekend. I added a case in the back to make sure no one gets shocked, and it came Tuesday, so I put that on Tuesday night.

For this first post, I just wanted to introduce the project, and share a few screenshots of what I had built.  If you work at BSE and this interests you, feel free to stop on over sometime and have a chat, or reach out on LinkedIn if you aren't at BSE!

The source code is on [GitHub](https://github.com/jerhon/pi-plate).

I'm working on the instructions for how it's all put together, which will be put on GitHub as well.

Here are a few pictures of the current state of the project.  Currently, just plug it into the wall, and up comes my information, and the fun polling questions.  Put it on my desk, and people will know where I sit.

## The Nameplate

![The nameplate](/images/pi-plate-nameplate.jpg)

## The Disney Princess Poll

![A fun poll](/images/pi-plate-question.jpg)

