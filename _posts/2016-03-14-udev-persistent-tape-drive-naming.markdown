---
layout: post
title: Persistent Tape Drive Naming in Linux with udev
date: 2016-03-14 11:33:00
categories: tape storage udev
---
##Overview
When Linux boots it will enumerate devices on the SCSI bus as they are presented to the kernel.  This
ordering can change due to several factors including device order initialization or the timing of
the devices appearing after a LIP on a the SAN.  Using the udev facility you can create symlinks to
give a persistent name to these devices.

##udev and its Role in the Linux Boot Process
udev is the daemon that receives information from the kernel about a device when added to the kernl.
Devices can be added at boot time and dynamically after boot, such as USB devices or removable hard
drives.  For instance, on most machines you can look in /sys/block and see your block devices, hard
drives most likely.  SCSI tape drives appear in /sys/class/scsi\_tape. Some information about each 
device is also passed to the udev system.  Here is how to interrogate a device node to find out 
more information (commands run on a CentOS 6.5 machine):

```
[root@deer ~]# udevadm info --name=/dev/st0 --query=all
P: /devices/pci0000:00/0000:00:02.0/0000:09:00.0/0000:0a:00.0/0000:0b:00.1/host3/rport-3:0-0/target3:0:0/3:0:0:0/scsi_tape/st0
N: st0
S: char/9:0
S: tape/by-id/scsi-3500110a0008e226f
S: tape/by-path/pci-0000:0b:00.1-fc-0x500110a0008e2270-lun-0
S: tapes/hp-lto4-2
E: UDEV_LOG=3
E: DEVPATH=/devices/pci0000:00/0000:00:02.0/0000:09:00.0/0000:0a:00.0/0000:0b:00.1/host3/rport-3:0-0/target3:0:0/3:0:0:0/scsi_tape/st0
E: MAJOR=9
E: MINOR=0
E: DEVNAME=/dev/st0
E: SUBSYSTEM=scsi_tape
E: BSG_DEV=/dev/bsg/3:0:0:0
E: ID_SCSI=1
E: ID_VENDOR=HP
E: ID_VENDOR_ENC=HP\x20\x20\x20\x20\x20\x20
E: ID_MODEL=Ultrium_4-SCSI
E: ID_MODEL_ENC=Ultrium\x204-SCSI\x20\x20
E: ID_REVISION=H6FW
E: ID_TYPE=tape
E: ID_SERIAL_RAW=3500110a0008e226f
E: ID_SERIAL=3500110a0008e226f
E: ID_SERIAL_SHORT=500110a0008e226f
E: ID_WWN=0x500110a0008e226f
E: ID_WWN_WITH_EXTENSION=0x500110a0008e226f
E: ID_SCSI_SERIAL=HUE11519M5
E: ID_BUS=scsi
E: ID_PATH=pci-0000:0b:00.1-fc-0x500110a0008e2270-lun-0
E: DEVLINKS=/dev/char/9:0 /dev/tape/by-id/scsi-3500110a0008e226f /dev/tape/by-path/pci-0000:0b:00.1-fc-0x500110a0008e2270-lun-0 /dev/tapes/hp-lto4-2
```

Each line starts with a letter.  N is the kernel name, S are symlinks, E are environmental variables and P
are the PCI paths.  Record the ID\_SERIAL\_SHORT  output for use in the rules.

###Adding udev Rules
On CentOS (and RHEL) 6.5 the rules are located in /etc/udev/rules.d.  The rules are numbered and each file
is read in order starting with the smallest.  72-tape-persistent-names.rules was created for the tape drives.
In this file the following lines were added:

```
KERNEL=="st*", ENV(ID_SERIAL_SHORT}=="3500110a0008e226f", SYMLINK+="tapes/hp-lto4-2", MODE="0666"
```

The ID\_SERIAL\_SHORT was used from the udevadm output and a symlink will be created in /dev/tapes.  
This is the second LTO4 drive in a library which is why it was named hp-lto4-2.  Additionally the
permissions were opened up so any user on the system can write to the tape drive which may not be
necessary depending on use case.  
