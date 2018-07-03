---
layout:   post
title:    EmulationStation's latest release for Debian Jessie
date:     2016-05-25
summary:  EmulationStation was builed for Boost 1.54 and two years ago Debian moved to 1.55.
tags:     misc, mediacenter
---

[EmulationStation](http://www.emulationstation.org) was builed for Boost 1.54 and two years ago
Debian [moved to 1.55](http://metadata.ftp-master.debian.org/changelogs//main/b/boost1.55/boost1.55_1.55.0+dfsg-3_changelog).

You can rebuild it inside a Docker container (because it requires lot of dependencies)

```
FROM debian:8.3
MAINTAINER @vando

RUN apt-get update && apt-get install -y --no-install-recommends \
    libsdl2-dev libboost-system-dev libboost-filesystem-dev libboost-date-time-dev \
    libboost-locale-dev libfreeimage-dev libfreetype6-dev libeigen3-dev \
    libcurl4-openssl-dev libasound2-dev libgl1-mesa-dev build-essential cmake \
    fonts-droid git openssl ca-certificates
    
RUN git clone --depth=1 --branch=master https://github.com/Aloshi/EmulationStation.git
RUN cd EmulationStation && cmake -DCMAKE_INSTALL_PREFIX=. && make 
```

And then you can copy the binary `emulationstation` from container to host

```
docker cp container-name:EmulationStation/emulationstation .
```

Final step is build a deb package

```
ES="/path/to/rebuiled/emulationstation"

wget -qO - http://emulationstation.org/downloads/releases/emulationstation_amd64_latest.deb | ar x -
mkdir DEBIAN tar -zxC DEBIAN -f control.tar.gz
sed -e '/^Depends/s/1.54.0/1.55.0/g' -e 's/$/, libboost-date-time1.55.0/' -i DEBIAN/control
tar Jxf data.tar.xz && rm control.tar.gz data.tar.xz  debian-binary
mv ${ES} usr/bin
find usr/ -type f -printf '%P ' | xargs md5sum > DEBIAN/md5sums
sudo chown -R root:root *
dpkg -b . emulationstation_2.0.1a-1_amd64.deb && rm -r DEBIAN etc usr
```

And you'll get this file:

```
emulationstation_2.0.1a-1_amd64.deb
```

If you have a Kodi mediacenter I recommend you to install the 
[plugin that launches EmulationStation](https://github.com/sdt/kodi-emulationstation-launcher).
