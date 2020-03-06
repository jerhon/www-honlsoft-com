+++ 
date = "2020-02-29"
title = "The Pi-plate: Setting up a Raspberry Pi Kiosk"
tags = ["pi-plate", "raspberry-pi", "linux"]
categories = ["blog"]
+++

In my first post, I gave a brief introduction to the pi-plate. In this post, I want to talk about the work it took to get my Raspberry Pi to run as a kiosk.

When I reference ‘running my Raspberry Pi as a kiosk’, I generally mean:
* The pi displays a window or application without any of the window manager chrome.
* A user cannot exit or switch to another application.
* The application running in the kiosk starts up automatically on boot.

I had to read a lot of blogs to put this all together. Many described the process in part but left out other parts such as user management. Here is an overview of what I had to do to my Raspberry Pi.

Overall the work required:
* Setting up a non-administrator user to run the kiosk.
* Having the Raspberry Pi to auto-login as the user, and start an X session.
* Creating a shell script to set up the environment, and run Chromium in kiosk mode.
* Creating a service on the Linux box to run the shell script to start Chromium once a graphical session has started.

## Prerequisites

I started with the graphical Raspbian distribution without extras, and updated the packages through apt.  I explicitly installed the dependency  ```unclutter```.  This can be installed through apt: ```sudo apt install unclutter```

## Setting up a New User for Auto-login

Whenever I'm working with a Raspberry Pi, I change the default password for the pi user.  The default password is well known, and if it's ever connected to an untrusted or semi-public network with a default password, it's as good as hacked.

For a service such as this, having a separate user to login is essential.  If someone could break out of the kiosk, I wouldn't want that person to have an account with potential administrative access. Setting up a new user in the standard Linux way, ```sudo useradd kiosk``` and follow the prompts through.

Getting the user to auto-login was slightly more complex.  Raspberry Pi comes with a configuration tool ```sudo raspi-config``` for configuring common setup options.  One of the options is to set up the default 'pi' user to auto-login to a graphical session.   However, it's not possible to specify which user should be the auto-login user.  Some manual configuration tweaking is required after selecting and saving the option in ```raspi-config```.  Under the ```/etc/lightdm/lightdm.conf``` there is a line ```autologin-user=pi```, change that pi user to 'kiosk' (or whatever the user added for this purpose), and it will be the auto-login user.

It's very important to know, each time ```raspi-config``` runs, it will overwrite the ```lightdm.conf``` completely, which requires setting the auto-login user again.

## X Settings

There are a variety of settings related to X to keep the display active.

```
xset s off
xset s noblank
xset -dpms

unclutter -idle 0.01 -root &
```
The first three lines 
1. Disable the screensaver
2. Disable 'blanking' the screen
3. Disables power management

The unclutter command is used to hide the mouse cursor on the screen.  If it's idle for 0.01 seconds with no movement, the cursor will be hidden.

## Chromium Based Kiosk Mode

The application for the pi-plate was written as a web application.  Chromium gives the ability to run in 'kiosk' mode which provides a sort of jail that a user cannot get out of.  Much of the configuration for this can be accomplished through command line switches. It takes some trial and error to figure out all the right options to set.

Here are the applicable switches I found:

* ```--kiosk``` to enables kiosk mode  
* ```--disable-pinch``` and ```--overscroll-history-navigation=0``` disable certain touch-based events on the screen
* ```--noerrdialogs``` and ```--disable-infobars``` disable various 'popups' that can display in the browser
* ```--check-for-update-interval=604800``` causes the browser not to show the update prompt
* http://localhost:80 - I'm serving up my application on localhost, but I'll have another post about the actual application.

I find it bizarre that all these things need to be set along with the kiosk option, but it's what's worked best for me.  The following command line shows all the options I'm using.

```
/usr/bin/chromium-browser --incognito --disable-pinch --overscroll-history-navigation=0 --noerrdialogs --disable-infobars --check-for-update-interval=604800 --kiosk http://localhost:80
```

There were additional commands I saw on a few blogs to reset the browser state if it crashed or didn't exit cleanly.  I don't know how well they worked as I never witnessed a crash.  These should be run each time before the Chromium browser is started.

```
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/kiosk/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/kiosk/.config/chromium/Default/Preferences
```

## Linux Service and the Kiosk Startup

The last portion is installing a service so that the Chromium browser will start on the X session starting.  I combined all the commands from the previous steps to setup the X session and start the kiosk and place them in a shell file.

After this, I copy out my ```pi-plate.service``` and ```pi-plate.sh``` files to a ```/srv/pi-plate``` directory and enable the service via ```sudo systemctl enable pi-plate.service``` in that directory.

Reboot, and the service should work.

Here's the service file I used.

```toml
[Unit]
Description=pi-plate
Wants=graphical.target
After=graphical.target

[Service]
Environment=DISPLAY=:0.0
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=simple
ExecStart=/bin/bash /srv/pi-plate/pi-plate.sh
Restart=on-abort
User=kiosk
Group=kiosk

[Install]
WantedBy=graphical.target
```

## Wrapping Up

Overall, there's nothing overly complex to get this done.  There are just a lot of steps- and it's a matter of piecing them all together to get a kiosk set up.

My next post in this series will be on the Angular application I wrote for the nameplate, and how it's run on the Raspberry Pi.