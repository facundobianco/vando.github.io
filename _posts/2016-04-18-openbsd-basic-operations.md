---
layout:   post
title:    "OpenBSD: Basic operations"
date:     2016-04-18
summary:  Are you comming from Linux? There are a list of basic commands.
tags: cli
---

### Where is `/proc/cpuinfo` ?

For list all information avalaible 

```
sysctl hw
```

And for the micro vendor and their cores

```
sysctl hw.vendor hw.product hw.ncpu
```

### Display real and available memory

```
grep ' mem =' /var/run/dmesg.boot
```

### Display amount of free and used memory

It's the most similar to `free -h`

```
top -d1 | grep Memory
```

### Which HDD are connected?

List the device names

```
sysctl hw.disknames
```

### List HDD partitions

The `c` partition is the entire disk, from the first sector to the last. 

```
disklabel /dev/wd0c
```

### Mount a FAT pendrive

By default, pendrives with FAT filesystem have the partition `i` for the files

```
mkdir /mnt/usb0
mount /dev/sd0i /mnt/usb0
```
