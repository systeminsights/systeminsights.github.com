---
layout: post
title: MongoDB and Replica Sets
date: 2012-05-08 10:50
comments: true
categories: mongodb 
author: deepak
---

## Why do we care

We use mongoDB to persist data from the vimana tenants. All historical data from our customers is stored in mongo, and its very important that we can continuously persist into mongo without losing data. We are using replica sets to make sure that our data is redundantly stored and to ensure that we have a failover mechanism when one of the mongoDBs lose connection.

## Introduction to Replica Sets

Replica sets are a form of asynchronous master/slave replication, adding automatic failover and automatic recovery of member nodes. A replica set consists of two or more nodes that are copies of each other. (i.e.: replicas). The replica set automatically elects a primary (master). Drivers (and mongos) can automatically detect when a replica set primary changes and will begin sending writes to the new primary.

## Those GOTCHAs

In order enable replica sets, you need to pass the "replSet" parameter while starting the mongod processes.
Replica sets cannot be initiated on those mongod instance which were already started with out "replSet" parameter.  Stop and restart the mongod processes with replSet parameter.
When replica sets are configured, all the writes will only go to the primary!!! Mongo has its own algorithms for syncing the data across the nodes.

## That setup

Starting multiple mongo instances.

	$ mongod --dbpath mongo_rpl/data1 --replSet set1 --port 27018
	$ mongod --dbpath mongo_rpl/data2 --replSet set1 --port 27019
	$ mongod --dbpath mongo_rpl/data3 --replSet set1 --port 27020

This starts 3 different instances of mongo running on different ports.

## Setting up the replica config

	➜  ~  mongo localhost:27018
	MongoDB shell version: 1.8.2
	Mon Apr 30 11:13:59 *** warning: spider monkey build without utf8 support.  consider rebuilding with utf8 support
	connecting to: localhost:27018/test
	> config = {_id: "set1", members: [{_id: 0, host: "localhost:27018"}, {_id: 1, host: "localhost:27019"},{_id: 2, host:"localhost:27020"}]}
	{
 	   "_id" : "set1",
 	  "members" : [
 	 {
 		"_id" : 0, 
 	   "host" : "localhost:27018"
 	},
 	{
 	   "_id" : 1,
 	  "host" : "localhost:27019"
 	},
 	{
 	   "_id" : 2,
 	  "host" : "localhost:27020"
 	}
 	]
	}
	> rs.initiate(config)
	{
 	   "info" : "Config now saved locally.  Should come online in about a minute.",
 	  "ok" : 1
	}
	>

This config enables the replicas to talk with each other. The talking would involve the replicas communicating who is the primary and who are all the secondaries.

## That app

We wrote a small ruby script which would tail the oplog of our test servers and insert some interesting records into the replicas. The oplog watcher is available (here)[https://github.com/deepakprasanna/mongo_oplog_watcher].

## Those observations

### Writing scenarios

Secondary goes down: There is no problem at all. If the secondary comes up again, mongo will take care of replicating the records which were lost by the time when it was down. Perfect!
Primary goes down: The ruby mongo driver raises "Mongo::ConnectionFailure". Checkout the oplog watcher, we have caught this exception and we are doing a puts that the connection is lost. But how ever the after a few seconds, when one of the other secondaries get elected as the primary the writes become successful. The interesting fact is that all the writes which failed during this recovery process is lost. Since we are able to catch "Mongo::ConnectionFailure", it is up-to the client to pull the sleeves and persist the data somewhere else until another secondary becomes a primary.While testing we lost about 20-30 records when the primary was down. I guess this number would be more or less depending the latency which we would face in the realtime.
Last mongod instance is not becoming primary: As you can see from the config above, we have 3 replicas. So we would have one primary(27018) and two secondaries(27019 and 27020). We are stopping the primary(27018). Now one of the secondaries becomes a primary(say 28019). Now we would have one primary(27019) and one secondary(27020). Again we are stopping the primary(27019). But the left over secondary(27020) will not become the primary!!!!!!! This causes all the writes to fail. But if we bring another dead mongod instance(27018) up, the leftover secondary(27020) becomes the primary and from then on the writes become successful.  This was found while digging into the mongo logs.
[rs Manager] replSet can't see a majority, will not try to elect self: From what I understand , Replica sets will do the election process only if there are 2 or more replicas available. If at all we have only one replica alive, the election will not happen and all the writes will fail because there will be no primary. We need have 2 replicas alive at anytime for the writes to become successful. This is interesting.

### Reading Scenarios

MongoDB doesnot by default support serving reads from the replicas. MongoDB will serve all reads from primary by default. Reads from secondaries can be configured, which is actually a 2 step process.  The first step is to configure "slaveOK" in the mongo console. This will tell mongo, it is okay to serve the reads from the secondaries. The second step is to instantiate the MongoReplConnection with :read => :secondary option.
This will tell the driver that it is okay to send the reads to the secondaries. Mongo driver will randomly select one of the secondaries to serve the reads. The distribution of the reads across the secondaries is handled by the driver.

Secondary goes down:
slaveOk -> Mongo::ConnectionFailure will be raised when there is a failure. Mongo driver is intelligent, when it sees a Mongo::ConnectionFailure it prevents the next reads from going to that dead secondary.
The driver has its own algorithm to find out if the dead secondary is back alive or not. As far a read is concerned we need to catch Mongo::ConnectionFailure and make the read once more(Assuming that another secondary will be up).If the secondaries are configured to serve the reads, then the primary is not touched at all until all other secondaries are dead. But there is no real way to find out which mongod instance served the read.

Without SlaveOk ->  Rest of the world goes as usual.
Primary goes down:
slaveOk -> Rest of the world goes as usual.

without SlaveOk -> All the reads are going to fail since primary can only serve the reads. There are 2 ways to solve this problem, catch the exception and throw an error message. Or keep polling the server until one of the other secondaries becomes a primary and read becomes successful. If the client decides to retry, it's not guaranteed that another member of the replica set will have been promoted to primary right away, so it's still possible that the driver will raise another Mongo::ConnectionFailure.

Happy hacking,
Deepak.