---
layout: post
title: "Introducing the vrtk package"
date: 2013-06-05 08:29
comments: true
categories: intern
---

As part of my internship project here, at System Insights, Chennai, I've been involved in developing an R-package called the 'vrtk', essentially a set of tools for reading and working with data from the vimana platform.

'vrtk' is short for _'Vimana Research ToolKit'_ or _'Vimana R ToolKit'_. It is intended to be an open-source set of some of the most basic tools needed to read and work with data.

The library is very simple to use. Visit [the github page](https://www.github.com/systeminsights/vrtk) for the download links, the source code, and to find the example files.

An Example
----------

The rest of this blogpost will be a simple demonstration -- A rudimentary look at path-feedrate data from an unknown machine.

The full example script can be found at [this link](https://www.github.com/systeminsights/vrtk/tree/master/example), and it doubles as a tutorial. This post provides just a concise overview.

#### Installation and Loading:

Follow the instructions on the [same github page](https://www.github.com/systeminsights/vrtk) to download and install the package. Next, open the example.R script, and follow along with the R-console open.

The example file sets up a demo dataset, which has feedrate data and tool position data from an unknown machine, monitored for around 30 minutes.

#### Reading the Data:

We use some of the core vrtk functions: `createDeviceGroupfromGhetto`, `summary`, `getDevice`, `getData` and `merge` to get the data into the right form.

Use the createDeviceGroupfromGhetto method to initialize a device group. Let's call it devGrp.

    devGrp <- createDeviceGroupfromGhetto("./402.json",data_folderpath="./402",createRData=TRUE)

Now, we can store the device itself in a variable.

    dev <- getDevice(devGrp)

The `vimana::summary` function provides a quick summary of the information present.

    summary(devGrp)
    summary(dev)

The summary tells us that the json is associated with a total of 1 device, with 11 DataItems associated.

A DataItem is vrtk's representation of the type of data measured. Examples -- `Xload`, `Yload`, `Zload`, `path_feedrate`, `Xact` etc.

The vimana::merge function to merge the DataItems we want, and reads them into a dataframe with the appropriate timestamps.

For this example, we want the pathfeedrate data and the position data. So we use merge as follows.

    allDataFrame <- merge(dev,"FEEDRATE-ACTUAL|POS")

After some column-renaming, the data is in a data.frame and ready for use. We can plot the pathfeedrate data with the following command. The resultant plot is shown below.

    plot(allDataFrame$timestamp,allDataFrame$reported.path.feedrate,type="s")

![path.feedrate-plot](/images/2013-06-05-pathfeedrateplot.png)

#### The Example Analysis

For now, we'll simply compute a feedrate from the position data and see how and when it differs from the pathFeedrate data from the same machine during the same time-duration.

First, we'd need the distance travelled given position data. Write a small function to compute it.

    dist <- function(xVec,yVec,zVec) { 
      # Euclidean dist
      xDiff = diff(xVec)
      yDiff = diff(yVec)
      zDiff = diff(zVec)
      return(c(NA,sqrt(xDiff^2 + yDiff^2 + zDiff^2)))
    }
    allDataFrame$prev.dist <- dist(allDataFrame$xPos,allDataFrame$yPos,allDataFrame$zPos)
    
Now we just need to compute the distances, and divide by the time difference to get the computed feedrate.

    timeDifsVector <- c(NA,diff(allDataFrame$timestamp))
    allDataFrame$computed.path.feedrate <- allDataFrame$prev.dist/timeDifsVector

Try to derive the Pearson correlation coefficient.

    cor(allDataFrame$reported.path.feedrate,allDataFrame$computed.path.feedrate,use="na.or.complete")

This yields a value of ~0.78, which is decent but not as high as expected. What would account for the difference, though?

Let's conclude this example by looking at the difference between the reported and computed feedrates. While the difference does not 'mean anything' in physical terms, it provides a clear picture of where the reported feedrate is differing from the computed one.

    difference <- allDataFrame$reported.path.feedrate - allDataFrame$computed.path.feedrate
    plot(difference,type="s")
    
![diffs-plot](/images/2013-06-05-diffsplot.png)

The difference seems to spike deterministically. We surmise that the spikes correspond to tool changes. This can actually be checked -- we just need to check the toolID data, if it's available.

For now, we'll conclude this example by taking a much smaller sampling period, where there aren't any spikes, and check the correlation.

    sample.reported <- allDataFrame$reported.path.feedrate[675:740]
    sample.computed <- allDataFrame$computed.path.feedrate[675:740]
    plot(sample.reported,type="s",ylim=c(-500,2000))
    lines(sample.computed,type="s",col="red")
    cor(sample.computed,sample.reported)

![closeup-plot](/images/2013-06-05-closeupplot.png)

97% in the regions between spikes. Not too shabby.

We see several interesting things happening here:
* In regions between spikes, the reported and computed feedrates correlate well.
* While the reported feedrate maintains a particular plateau value for a while, the computed feedrate shows minor fluctuations. Interestingly, using the same position data as in this example, it is possible to see that feedrate fluctuates (usually, the reported is higher), just before the tool takes a sharp turn.
* Also, when there's a change in the plateau value, the computed feedrate always lags for a datapoint or two before reaching the reported value, as one would expect from a real world machine.

That's about it, I suppose! This concludes our trivial example using the vrtk package. Be sure to check out the github and the documentation for further details and the source-code.
