---
layout: post
title: Storage Tiering
date: 2016-03-08 13:18:00
categories: storage file-system tiering
---
##Overview
Storage tiering is the practice of using fast (IE expensive) disk for files that are in use or 
often read/modified and slow disk or possibly even tape for files that are rarely accessed. Using
such architecture allows for costs to be controlled.

##Storage Tiering in Concept
There are several pieces to a tiered storage.  There are the tiers, generally composed of a fast
SSD/SAS shelf or shelves on the first tier, enterprise SATA drives on the second tier and if 
there is a third tier it can be super large SATA/SAS hard drives or tape.  There is a manager
that generally has rules defined about how and when to move files between tiers.  Finally there
is the presentation layer which generally exports all tiers in a single namespace to hide the 
details behind the actual location of the files.

##Storage Tiering in Practice

###Implementations
There are several implementations of storage tiering.  On the commercial side there is Quantum 
StorNext and IBM Tivoli as two large examples.  On the open source front Lustre has implemented
a hierarchial storage manager (HSM) to their file system.  

###Lustre HSM
Lustre uses an open source product called RobinHood which is Lustre aware and can take advantage
of certain features to minimize impact to the live file system.  It has the ability to move data
between Lustre OSD pools and even external programs, such as a tape system.  

###Policies
Generally policy can target specific metadata of a file.  This may include the last modified date,
the last access date, the size and the file name itself.  Certain file or directories, such as
a database directory, may be imperative to keep in the top tier.  Other files, such as log files,
may be more quickly moved to a tape tier.


