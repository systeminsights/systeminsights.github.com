---
layout: post
title: "Monitoring MTC Streams: MTConnect Graphr"
date: 2012-07-02 11:15
comments: true
categories: mtconnect interns
author: prince 
---

A few months back I took up the fun task of exploring MTConnect streams and the amazing possibilities which it presented to a developer. That culminated into a web base monitoring app, MTConnect Graphr, which can now be downloaded from [github](http://github.com/princearora/mtconnect-graphr). In this post I'll run down through the development process of the same.

The web scene has changed remarkably in a past few years. The web applications are expected to be compatible to smartphones and tablets. They are supposed to be clean and responsive. This requirement was enough to persuade me to use [bootstrap by twitter](http://twitter.github.com/bootstrap/) as the basic framework for the app. It is tiny, expandable and has got a good documentation to get you started.

Coming to the implementation, the first task was to dynamically connect to the xml stream provided by the agent. Though it sounds pretty easy, the task becomes a bit tricky because it requires a recurring connection with external urls. The easiest way out of this situation is to implement a php proxy. For this we write a php loader script, and then use it every time we need to connect to an external host.

	header('Content-type: application/xml'); //specififying the return content type
	$q = $_GET['url'];
	  handle = fopen($q, "r");              //connecting to the url
	if ($handle) {                          
	    while (!feof($handle)) {
	        $buffer = fgets($handle, 4096);
	        echo $buffer;                  //reading and returning content
	    }
	    fclose($handle);
	}


That sums up our loader.php. Next all we need to do is to write a simple function to load the xml file via the proxy.

	function getCurrentXML(conn_url) {              //function to retrieve a xml file 
	    var n = "";
	    return $.ajax({
	        url: "loader.php?url=http://"+conn_url, //using the proxy
	        cache: !1,
	        async: !1,
	        dataType: "txt",
	        success: function (t) {
	            n = t
	        }
	    }), n
	}

Once we have access to the xml stream, there are a plethora of tools available to parse and get data out of it. I chose to use a combination of jQuery and JSON for the job. The xml2json plugin available [here](http://www.fyneworks.com/jquery/xml-to-json/) provided an easy conversion to JSON. 

	var xmldata = getCurrentXML(conn_url),
	    i = $.xml2json(xmldata);

With JSON, life is easy. It can't be any simpler to parse data than it is with JSON. All I did was to write regular funtions to parse conditions and device parameters. But there is a catch. Not all xml tags will be meaningful and to make them appear right, we need to write individual functions for each of them. Here I will explain the working of the function used to parse the conditions for all parameters.

	this.getCondition = function (n) {
	  var r=new Object();
	  r.type= new Array(),r.value=new Array();
	  var count=0;
	  for (var t = 0; t < n.ComponentStream.length; t++) {
	    var i = n.ComponentStream[t],
            u = i.name;
	    if (i.Condition)
	    {
	      var v = i.Condition;
	      if(v.Normal){ if(v.Normal.length>1){
	        for(var f =0; f < v.Normal.length; f++)
	        {
	          r.type[count] =  u+' '+v.Normal[f].type;
	          r.value[count++] = "Normal"
	        }}
	      else{
	          r.type[count] =  u+' '+v.Normal.type;
	          r.value[count++] = "Normal"}
	        }
	      ..........
	      //similarly for other conditions 
	     }r.len=count;
	   }
           return r 
	},

Though this takes away the reusability of the script, the data displayed turns out to be easier to comprehend for all.

With all these pieces in place, a simple requirement is to refresh the input from the MTC stream every few milliseconds. A recursive function with delay takes care of that

	var updateFromMTC = function(){
	.....
	  setTimeout('updateFromMTC("'+conn_url+'")' , 1000);
	.....
	}

The task we confront next is to display it elegantly. That is taken care by using poweress of HTML5. The devices in the stream are populated at the top of the page with the color of each dependent on the availability of the device. To display all the parameters we use two empty divs, in which the parameters and the conditions are populated using a javascript user function.

	<div class="container" id='SelStats' position='absolute'">
	  <div id='conditions' position='absolute'></div>
	</div>

	var updateSelected = function(){
	 if(thePage.ActiveShape){
	   var selShp = thePage.ActiveShape;
		var SelDisplay = document.getElementById('SelStats');
		if(SelDisplay && selShp){
		 if(selShp.deviceName != ''){
			$(SelDisplay).empty();
			$(SelDisplay).append('<b>Machine:</b>' + selShp.text);
			......

So, this completes the basic task of monitoring an mtconnect stream. Next we need to plot it. There are some really advanced open source scripts out there to assist plotting data, but for this particular task [smoothie charts](http://smoothiecharts.org/) seemed perfect fit to me. It is a really small charting library designed for live streaming data. Integration with the existing code was easy. A few more lines to the code, and it plots like a charm.

	var smoothie = new SmoothieChart();
	  smoothie.streamTo(document.getElementById("mycanvas") 3000 /*delay*/);
	var line1 = new TimeSeries();
	  setInterval(function() {line1.append(new Date().getTime(), math.random());}, 3000 /*delay*/);

Finally, we need to add an emergency alarm light for the parameter being monitored. A slick form to enter the maximum/minimum value, and a basic function to compare instantaneous values are enough to pull it off. With the div being populated dynamically every few seconds, we need to save some info in a cookie which is made easy by the [jQuery-cookie](https://github.com/carhartl/jquery-cookie/) plugin.
<div style="text-align: center;">
<img src="/images/graphr-1.jpg" width=360 height=600 /> <img src="/images/graphr-2.jpg" width=360 height=600 /> </div>
I guess that's it. The app is ready to roll. I checked it out locally on a PC, iPOD touch and an android device. Seems to be working fine for me. Let me know if any of you notice anything off about it. 

PS: Please ensure that the application is run on a php server. Otherwise the application will fail to connect to the stream and all you will see is a white blank page. I'd recommend WAMP/LAMP for users trying it on their personal PCs.	