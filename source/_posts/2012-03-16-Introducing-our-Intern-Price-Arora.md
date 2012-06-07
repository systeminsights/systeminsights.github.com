---           
layout: post
title: Introducing our Intern - Prince Arora
date: 2012-03-16 04:00:00 UTC
comments: true
categories: mtconnect interns
author: athulan
---

I want to introduce Prince Arora, a student from IIT Madras, who is interning with us at our Chennai office. Prince is developing a suite of MTConnect-based tools to track the maintenance status of machine tools. He is blogging about his experiences at System Insights at his blog, [The 20four hour log](http://princearora.in).

Prince started out by building a simple webapp to display various maintenance-related parameters from an MTConnect data stream. Specifically [he developed an app](http://princearora.wordpress.com/2012/02/14/mtconnect-the-problem-statement/) to:

* Connect to any MTConnect Stream specified by the user
* Recognize all the devices within the stream
* Continuously read & display multiple parameters for each of the device
* Display condition of all components
* Plot a curve of the variation of a parameter over time
* Allow user to monitor a parameter and alarm if it moves outside the range of values entered

Here is a screenshot of viewing realtime data from an MTConnect Agent. The app plots the value of an MTConnect Sample DataItem in realtime. The screen below shows the app plotting the X position, but this can also be used to plot maintenance-related data like a vibration or temperature.

<!-- {% img 2012-03-16-Introducing-our-Intern-Price-Arora-pic-2.jpeg %} -->

You can also load up the status of various conditions active in the machine tool, and see if the parameter that is being plotted is within some user-determined bounds. The screen below shows bounds set for the DataItem Commanded Y Position between 3 and -2. A green indication is shown because the data item is operating within bounds.

<!-- {% img http://placekitten.com/890/280 %} -->

Prince will be posting more about his internship in these pages. Stay tuned!