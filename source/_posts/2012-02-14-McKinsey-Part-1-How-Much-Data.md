---           
layout: post
title: McKinsey on (Manufacturing) Big Data - Part 1 - How Much Data?
date: 2012-02-14 06:50:00 UTC
comments: true
categories: big-data
author: athulan
---

McKinsey recently published a report about Big Data, going into considerable detail about its impact on different fields, including manufacturing. In the next few posts, I will be digging into this report looking at the specific impacts of big data on machining-related manufacturing, focusing on ways that improve productivity and efficiency.
First, lets start with how the "Big Data" is defined:
>“Big data” refers to datasets whose size is beyond the ability of typical database software tools to capture, store, manage, and analyze. This definition is intentionally subjective and incorporates a moving definition of how big a dataset needs to be in order to be considered big data—i.e., we don’t define big data in terms of being larger than a certain number of terabytes (thousands of gigabytes).

This definition applies remarkably well to Manufacturing Big Data. The "bigness" of the data is not necessarily its absolute size, because process data from a machine tool might be in the tens of gigabytes, which is paltry compared to Internet data like website click-throughs or ad impressions. The bigness comes from the fact that traditional manufacturing decision making systems (spreadsheets, sticky-notes-on-whiteboards, MES systems) deal with very small sets of data – perhaps in the megabytes – and we are now looking at harnessing data that is several orders of magnitude larger than that.

So how much data are we exactly talking about? Lets take the case of collecting MTConnect-based data streams from manufacturing equipment. With Basic monitoring (looking at production efficiency, part count, alarms, messages, and overrides), we can estimate a data rate of about 10 samples a second, with each sample consisting of 10 data items. This generates over 400MB of data daily, or more than 150GB annually per device. With Advanced monitoring (which will complement Basic monitoring with data from embedded and external sensors), this grows to over 21 GB a day, or close to 8 TB a year. With these estimates, a small manufacturing shop with about 10 devices, wil generate over 2 TB of data a day with Basic monitoring, or close to 80 TB with Advanced monitoring. Moving up to a multi-facility enterprise with about 500 devices, we are looking at about 80 TB a year with Basic monitoring, and close to 4 PB (Petabytes) with Advanced monitoring. The table below gives a few more examples.

<iframe frameborder="0" height="360" src="https://docs.google.com/spreadsheet/pub?key=0AnJkF2eeISMAdG5yTjZ4TkgyVEVEWE5rM2E5bmE1eUE&amp;single=true&amp;gid=0&amp;output=html&amp;widget=true" width="500"></iframe>

If we extrapolated this further to look at the total number of machine tools currently installed in the United States today (approximately 1.2 Million, based on estimates from AMT), Basic monitoring will generate over 189 PB of data, while Advanced monitoring will generate over 9,400 PB of data (over 9.4 Exabytes). Of course, this does overstate the total data load since not all of these machines can be readily addressed, but it gives a sense of the scale of manufacturing data, and -- more importantly -- the opportunity to harness it.

{% img center /images/2012-02-14-McKinsey-Part-1-How-Much-Data-Sectoral-Data-Storage.png %}

McKinsey estimates about 966 PB of stored data in the Discrete Manufacturing sector (see above). Add Process data to that, we are looking at greatly increasing the total storage requirements of the sector. Manufacturing has the largest storage needs of all the surveyed sectors, and the potential of Big Data analytics on process data will further increase them.