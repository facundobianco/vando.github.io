---
layout:   post
title:    IPtv Simple for Kodi 15.2 in Debian Jessie
date:     2016-05-25
summary:  Create a PVR IPtv Simple deb file for Debian 8 
tags:     misc, mediacenter
---

I've been using [OpenELEC for Raspberry Pi v2](http://wiki.openelec.tv/index.php/Raspberry_Pi)
but it buffered all the time.

So I moved to my desktop and I installed Kodi 15.2 [from Jessie Backports](https://packages.debian.org/jessie-backports/kodi)
but I can't find the package PVR IPtv Simple. For some reason there isn't a 
package for Jessie Backports (in the official repository, it's for [kodi 16.1 (stretch)](https://packages.debian.org/stretch/kodi-pvr-iptvsimple)
and on [Debian Multimedia](https://www.deb-multimedia.org/dists/stable/main/binary-amd64/package/kodi) it's for Kodi 14.2).

I made a PVR IPtv Simple deb file for Debian 8.

First, install this packages

```
apt-get install -y clang cmake libc-dev-bin libc6 libc6-dev libp8-platform2v4 gcc
```

Then run

```
BUILD_DIR=`pwd`
git clone --depth=50 --branch=Isengard https://github.com/kodi-pvr/pvr.iptvsimple.git
git clone --depth=1  --branch=Isengard https://github.com/xbmc/xbmc.git
mkdir -p pkg/DEBIAN pkg/usr/share/kodi/addons pkg/usr/share/doc/kodi-pvr-iptvsimple \
      pkg/usr/lib/x86_64-linux-gnu/kodi/addons/pvr.iptvsimple
cd pvr.iptvsimple ; mkdir build ; cd build
cmake -DADDONS_TO_BUILD=pvr.iptvsimple -DADDON_SRC_PREFIX=$BUILD_DIR \
      -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=$BUILD_DIR/pkg/usr/share/kodi/addons \
      -DPACKAGE_ZIP=1 $BUILD_DIR/xbmc/project/cmake/addons && \
make || exit 1
# This creates your deb file
cd $BUILD_DIR/pkg
mv usr/share/kodi/addons/pvr.iptvsimple/pvr.iptvsimple.so* \
   usr/lib/x86_64-linux-gnu/kodi/addons/pvr.iptvsimple/
mv $BUILD_DIR/debian/copyright usr/share/doc/kodi-pvr-iptvsimple/
find -type f ! -regex '.*?DEBIAN.*' -printf '%P ' | xargs md5sum > DEBIAN/md5sums
wget -qO DEBIAN/control https://gist.github.com/vando/3fb5f8324dc58347738378494af7044f/raw
echo "pvr.iptvsimple 15.2 kodi-pvr-iptvsimple" > DEBIAN/shlibs
sudo chown -R root:root *
dpkg -b . ../kodi-pvr-iptvsimple_2.4.0+git20160518-1_amd64.deb && \
     rm -rf pkg pvr.iptvsimple xbmc
```

And you'll get this file:

```
kodi-pvr-iptvsimple_2.4.0+git20160518-1_amd64.deb
```

Otherwise, you can [download it](http://devio.us/~vando/pkg/kodi-pvr-iptvsimple_2.4.0+git20160518-1_amd64.deb) 
from my `public_html` in Devio.us and its MD5 is

```
37816afa3ea1cde414f79b24eb03f353  kodi-pvr-iptvsimple_2.4.0+git20160518-1_amd64.deb
```
