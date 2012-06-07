---           
layout: post
title: Setting up a MTConnect Agent on a Linux (Ubuntu) machine
date: 2012-03-16 10:40:00 UTC
comments: true
categories: mtconnect interns
author: prince
---

This was blogged first at my personal blog [here](http://princearora.wordpress.com/2012/02/24/setting-up-a-mtconnect-agent-on-a-unixubuntu-machine/).

While working on my internship project, I got a chance to test out an MTConnect agent built in C++ for Linux (ubuntu). I was surprised to find that absolutely no documentation existed for setting up the Agent in a Linux environment. Although it didn't turn out to be a big hassle in the end, I thought that it would be a good idea to list down the process of setting up a MTConnect agent on Linux. So, here you go:

*Step 1*: Download the zip archive of the latest version of MTConnect C++ Agent SDK from MTConnect Github and extract its contents onto your local disk.


*Step 2*: Download & Install libxml-2.0 and libxml-dev packages from the apt repository.

	apt-get install libxml2 
	apt-get install libxml2-dev 


*Step 3*: Now you need to prepare a Makefile in order to compile the agent. This can be done using the Cmake package.
Download & install cmake if you don't already have it.

	apt-get install cmake 
Open the 'agent' folder in the terminal and run cmake and make.

	cd agent/
	cmake .
	make 

*Step 4*: If everything went right, your agent would have been build. You can now start it off as a service.

	./agent daemonize 

If you are unsure whether the process is running, you can check out the process status:

	ps aux | grep agent 

The agent service should be up and running. You may change the agent.cfg file in any text editor based on the instructions here.

Have fun! 