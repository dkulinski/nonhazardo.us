---
layout: post
title: Erasure Coding
date: 2016-03-08 08:06:00
categories: storage linux file-system erasure
---
##Overview
This is a quick overview of how I understand erasure coding.

##Summary
Erasure encoding was released in the latest version of Gluster, v3.7.  Erasure coding is a method used to scale redundancy
but at a cheaper cost than pure replication.  

###Replication in Gluster
Replication is easily supported in Gluster and you can specify an amount of copies of the data.  More copies across
servers guarantee safety of the data.

###Erasure Coding
With Erasure Coding each file is split into separate parts with a parity algorith being applied.  The specification
generally looks like m+n which stands for M chunks per file and N pieces of redundancy.  If you specify 5+3 a file 
would be split into a total of 8 chunks and any 5 of those chunks can be used to reassemble the whole.  This would
allow for 3 brick failures in the Gluster file system and the data would still be accessible.

####Upside
With erasure coding you are not storing full replicas.  In the above scenario the total overhead is 38% vs 300%
for storing 3 replicas of the original data.

####Downside
The down side to erasure coding is that it is CPU intensive.  This will create latency to your data so it shouldn't
be used where performance is concerned.  It is a mechanism designed to allow higher redundancy at a lower total disk
cost.  
