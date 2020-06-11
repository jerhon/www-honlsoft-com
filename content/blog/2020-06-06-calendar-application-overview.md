+++
date = "2020-06-06"
title = "Calendar Application - Another Raspberry Pi Project"
tags = ["pi-calendar", "raspberry-pi", "web-development"]
categories = ["blog"]
+++

A few months ago, I built a Raspberry Pi Kiosk that acted as a nameplate.  Since then, a lot has changed in the world.  I've been working from home full time, so COVID-19 has rendered my nameplate useless.  Even when work returns to normal, the interactive portions of it don't seem appropriate anymore. 

[You can read more about the Raspberry Pi based nameplate I built here.](/projects/pi-plate/)

With the pandemic, many developer conferences have gone online, providing a wealth of new technology related content.  Recently Microsoft had it's Build 2020 conference all online.  Docker also had it's Dockercon, again all online.  While it's tough to spend an entire day just viewing materials on these, having them online allows most of the events to be available to watch online after the conference.  Microsoft had already been doing this for years, but it's great to see others do it as well.

With all that said, I've been watching a few of these sessions sessions from these conferences, and one of the great ones I watched was around the Microsoft Identity Platform in Azure.  Part of the video introduced me to the MSAL (the Microsoft Authentication Library).  MSAL greatly simplifies building applications that need to authenticate to Azure Active Directory and accessing data from the Microsoft Graph APIs.  MSAL also has implementations for many of the major application development frameworks.

Using what I've learned about Microsoft's MSAL library, I've decided to take my Raspberry Pi kiosk and  re-purpose it with another proof of concept. I'm going to build a calendar application to keep up on appointments-- something I can just put on the counter and see what is upcoming for the day. 

![Calendar](/images/raspberry-pi-calendar.jpg)

I've been writing a few articles about the project and will start posting them over the next few weeks.