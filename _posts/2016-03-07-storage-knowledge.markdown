---
layout: post
title: Storage knowledge
date: 2016-03-07 11:03:00
categories: storage linux file-system
---
##Overview
This is an article to get the information from my head on to a written form.  I will briefly explain some topics below.
##Storage
Today there are four types of storage:

* Network Attached Storage (NAS)
* Storage Area Network (SAN)
* Direct Attached Storage (DAS)
* Parallel File Systems 

###Network Attached Storage
Network Attached Storage (NAS) are file systems that are network aware.  They use a specific API to
allow connections.  They come in many different flavors with the two most popular being NFS and Samba/CIFS.

####NFS
NFS is the Network File System.  It was been implemented by UNIX and Windows for many years.  Access control
is done based on IP address, or more recently starting in NFSv4, through Kerberos. Starting with NFSv4.1 
parallel NFS has become a reality but requires additional setup and multiple servers which makes is a hybrid
of a NAS and a Parallel File System.
 
####Samba/CIFS
CIFS is a file system made popular by Windows and is the default shared file system enabled in Windows.
The common implementation of CIFS on UNIX/Linux operating systems is Samba.  CIFS shares can be 
authenticated by a simple username/password hash or can be joined to a Kerberos domain on Windows
Active Directory (AD) architectures.

###Storage Area Network (SAN)
Storage Area Networks (SAN) are ways to expose block devices over a network.  The most common SANs today are 
Fiber Channel and iSCSI.  There are more esoteric networks such as Fiber Channel over Ethernet (FCoE) and ATA Over 
Ethernet (AOE) which use Ethernet as a base protocol.  Fiber Channel has the advantage of lower latency times vs
an Ethernet based architecture.  This comes at a cost of running a separate network and the associated costs
for host bus adapters and switching infrastructure.  Access control on these networks are typically done by zones
in which either physical ports are access controlled or World Wide Node Names (WWNN) are access controlled.  iSCSI 
is SCSI implemented over IP.  This has gained a lot of traction over the past few years.  Each iSCSI client and server
is assinged a unique identifier.  The unique identifier is used to control access at the iSCSI server.  A lot of 
network adapters these days include acceleration for iSCSI storage and includes the ability to present such storage 
during the boot of a server.

###Direct Attached Storage
Direct Attached Storage (DAS) is where a block device is located directly on a server.  This includes the 
Serial Advanced Technology Attachment (SATA), Serial Attached SCSI (SAS) and Small Computer System Interface (SCSI) 
busses.  SATA is most commmonly found on all systems and associated with cheaper disks.  SCSI is outdated and
has been replaced by SAS.  SAS drives are generally higher performance and consequently higher priced.

###Parallel File Systems
Parallel File Systems are a relative newcomer in the storage arena.  Rather than offering just vertical growth 
it promises to offer both vertical and horizontal growth under a single namespace.  There are two defining 
characteristics of a Parallel File System architecture.  These are the Meta Data Servers (MDS) and the Object
Store Servers (OSS).  The MDS is tasked with keeping a record of the files in the file system and what objects
compose that file.  The OSS are responsible for storing the data of the objects.  Lustre, GPFS, Ceph and Panasas all
follow this hierarchy.  Gluster uses hashing from the client to facilitate the storage of data and keeps the 
metadata local to the OSS.  This reduces a potential bottleneck of performance in theory but in reality it rarely
does.  

##Linux File Systems
Linux supports several file systems.  These include ext4, XFS, ZFS and BTRFS.  Each represent and handle data
differently.  

###ext4
ext4 is the Extended File System version 4.  It has many features to improve performance and integrity.  Journaling
offers the ability to quickly recover from an unexpected power outage and leave the file system in tact.  Extents
allows planning and laying down correlated data (data in the same file) in the same proximity to boost sequestial
performance.

###XFS
XFS is the file system used by IRIX and ported to Linux.  It has many features to improve performance and integrity.  Journaling
offers the ability to quickly recover from an unexpected power outage and leave the file system in tact.  Extents
allows planning and laying down correlated data (data in the same file) in the same proximity to boost sequestial
performance.  XFS is also 64 bit allowing a maximum size of 8 exabytes for a single volume.

###ZFS/BTRFS
ZFS and BTRFS are two new file systems which incorporate volume management as well.  ZFS was pioneered by Sun 
Microsystems while BTRFS is an effort from Oracle.  These two are lumped together because they share many 
similarities.  Both are Copy-on-Write (COW) file systems where new data for existing files is copied and 
written into a new area.  The file systems use a B-Tree, or in the case of ZFS B-Tree-like, structure
for metadata.  Both include checksums for the data on disk that is rechecked at time of read and recalculated on writes.
Checksums allow for end to end data security and can expose hardware problems such as faulty RAM, bad disks and even
bad cables.  Due to the structure of the metadata and COW this allows for snapshots without overhead and the ability
to send and receive snapshots as a block stream for easy migration or redundant copies of data and only sends the 
changed blocks.  ZFS has some advanced caching features such as the ZFS Intent Log (ZIL) which records data operations
before they are committed to disk and the Level 2 Adjustable Replacement Cache which is where hot files are kept in
for a read cache. Because ZFS also manages volumes separate classes of disks can be assigned to the storage pool,
the ZIL and the L2ARC to allow for performance tuning.  Both ZFS and BTRFS offer "RAID" levels comparable to 
RAID 5 and RAID 6.  ZFS further allows triple redundancy RAID.  Avoiding a hardware controller allows for certain 
performance enhancements over a hardware RAID5/6, mainly skipping the "write hole".  The write hole is a term used
to describe the performance penalty when writing data that is less than the full stripe size of the RAID controller.  
If the stripe size is 1MB and only 4kb is written the hardware RAID controller will still need to write 1MB of data.  
ZFS avoids the write hole by using variable sized stripes down to the block level of the disk drives.  ZFS also offers 
data deduplication, which is very RAM intensive, and both ZFS and BTRFS allow compression.  
