---
layout:   post
title:    Debian: A lightweight APT
date:     2016-05-17
summary:  A few tips for APT
category: cli
---

## Do not consider recommended packages as a dependency for installing

For me, one of the most annoying thing in Debian is the APT "I'll-install-all-packages" manager.
I just want one specific package and (of course) their required packages.

Create a file called `/etc/apt/apt.conf.d/01norecommend` whit the content

```
APT::Install-Recommends "0";
APT::Install-Suggests "0";
```

## Avoid downloading translation packages

Are you tired about downloading various packages for differents
languages? 

Create a file called `/etc/apt/apt.conf.d/99translations` with the line

```
Acquire::Languages "none";
```

## Plus: force APT to use IPv4 instead IPv6

Create a file called `/etc/apt/apt.conf.d/99force-ipv4` with the line

```
Acquire::ForceIPv4 "true";
```
