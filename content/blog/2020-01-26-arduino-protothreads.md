+++ 
date = "2020-01-02"
title = "Arduino Dancing Car Bot"
tags = ["arduino", "robotics", "threading", "protothreads"]
categories = ["blog"]
+++

One major challenge in developing for small microprocessors is the lack of conveniences.  Particularly in the area of threading, it can become challenging.

For example, I'm making a small arduino car bot, I want it to drive two sides of a motor controller, turn a servo, and detect distance.  Some of these processes can run in sequence, but as I'm doing this, there may be potentially other things I want to add while the bot is doing these things.

Since microprocessors generally lack threading support, using a state machine in a processing loop is an architectual paradigm that can be used.

1. Having a loop in the application from which all code is run.
2. Seperating processes into seperate logical pieces.
3. Changing each piece of code to run quickly, and keep track of it's current state.

Building state machines is not the most exciting development.  However, some simple macros in C/C++ have made this code managable.  One such pattern that has been well developed is protothreading. 