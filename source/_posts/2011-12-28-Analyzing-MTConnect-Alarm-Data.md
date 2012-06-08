---           
layout: post
title: Analyzing MTConnect Alarm Data
date: 2011-12-28 12:23:00 UTC
comments: true
categories: mtconnect
author: athulan
---

Machine Tools produce enormous quantities of Alarm data, but analyzing this data can be a challenge. We are primarily interested in finding out how Alarms can help us understand Production disruptions and downtimes. While Machine Tools tend to be chatty with alarms, the alarms contain only limited information of value. Part of the problem is the lack of descriptive alarm text. Take a look at some examples from a modern, multi-axis CNC-controlled machine tool:

- + OVERTRAVEL ( SOFT 1 )
- 1-ROT MOTOR SENSOR ERROR 81
- HYDRAULIC PRESSURE DOWN
- HIGH PRESSURE COOLANT CLEAN TANK OIL LOW LEVEL
- OIL-MATIC TEMP./FILTER ALARM
- Y AXIS HOME POSITION RETURN REQUEST

While some of these alarms are self-explanatory (like "`HYDRAULIC PRESSURE DOWN`"), other alarms can be a little harder to understand without any context. One way of adding context to Alarms is to look at the ControllerMode and the ExecutionStatus when the Alarm fired, and how they changed across the duration the alarm was active. 

We took several months of data from a multi-axis CNC-controlled Lathe, and we looked at the different Alarms that occurred in this period. To keep the list manageable, we only looked at Alarms which had the severity level "Fault". 
Here are the alarms that occurred during this period:

_Alarm Count_

_Alarm Duration_

We can see that the Chuck Barrier alarm occurred the most number of of times, but was not active the longest (that distinction went to the two Spindle-related alarms). To place these alarms in better light, we can look at the ControllerMode and ExecutionStatus when these alarms were active:

_Alarm Duration by Execution Status_

_Alarm Duration by Controller Mode_

We are most interested in Alarms that have stopped part production, so we can focus on those have have occurred when the ExecutionStatus was `STOPPED` or `INTERRUPTED`. Similarly, Alarms that occur when the Controller was in `Manual` or `MDI` mode are probably those which were a by-product of something a user was manually doing on the machine, so we can ignore those to focus on those that occurred when the mode was `AUTOMATIC`. With this filter, we can see for example that the "CHUCK BARRIER" alarm which occurred the most number of times, seems to have exclusively occurred when the ExecutionStatus was `READY` (implying that it was not interrupting program execution) and when the ControllerMode was `MANUAL` (implying that a user was manually operating the machine when the alarm fired). The two Spindle Alarms now look more interesting, since they occurred when the ExecutionStatus was `STOPPED`, and the ControllerMode was both `AUTOMATIC` and `MANUAL`. This can imply that this alarm did interrupt production, and that it led the the Mode being changed from `AUTOMATIC` to `MANUAL`. 

We can dig one level deeper, and look at the combined state mapped by the `ControllerMode` and `ExecutionStatus`:

_Alarm Duration by MetaState_

This gives us even more clarity: we can see that two Spindle Alarms switched two states – between `STOPPED/AUTOMATIC` and `STOPPED/MANUAL`, further confirming that this Alarm did stop program execution, and that the mode changed from `AUTOMATIC` to `MANUAL`.

Given the lack of clarity in alarm data, looking at the ControllerMode and ExecutionStatus gives us a better understanding of Alarms that can have an impact on Production. In vimana, we filter Alarms based on the ControllerMode and ExecutionStatus so that we can filter and look at only those that interrupt Production. We also look at temporal patterns between alarms, so that the ordering of alarms can be studied to better understand the phenomena they are describing. Downtimes are classified based on these characteristics of alarms. These will be discussed in this blog in an upcoming post. 