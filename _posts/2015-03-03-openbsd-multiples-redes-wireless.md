---
layout:     post
title:      "OpenBSD: manejo de diferentes redes wireless"
date:       2015-03-03
summary:    Un script para manejar de forma inteligente las diferentes redes wireless con <code>/etc/netstart</code>.
categories: openbsd wireless script
---

A diferencia de Linux donde wireless-tools únicamente soporta WEP (y
por ello para conectar a APs con cifrados mas seguros requiere el uso
de [wpa_supplicant](http://w1.fi/wpa_supplicant)), OpenBSD [sí
soporta](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ifconfig.8?query=ifconfig)
todos los cifrados wireless.

El unico inconveniente es que no maneja diferentes *profiles* de redes
wireless como sí hace wpa_supplicant. Pero encontré una solución en
[OpenBSD
Journal](http://undeadly.org/cgi?action=article&sid=20071224164233&mode=flat)
a la que agregué algunas modificaciones propias: soporte para redes
abiertas o WPA y opciones de dirección IP y para la placa wireless.

Primero, hay que agregar las siguientes líneas al archivo
`/etc/netstart`

```sh
# Create hostname.if(5) file for wireless device.
if [ -f /etc/rc.wireless ]
then
       . /etc/rc.wireless
fi
```

El archivo `rc.wireless`

```sh
#!/bin/sh

CONF="/etc/rc.wireless.conf"
IFACE=`sed -n 's/IFACE="\(.*\)"/\1/p' "$CONF"`
HFILE="/etc/hostname.$IFACE"

ifconfig "$IFACE" scan | sed -n '/  nwid/s/.*bssid \([^ ]*\) .*$/\1/p' | \
while read MAC
do
    . "$CONF"
    
    if [ "$?" = 0 ]
    then
        OUTPUT="nwid $NWID"
        [ -n "$NWKEY" ]  && OUTPUT="$OUTPUT nwkey $NWKEY"   || OUTPUT="$OUTPUT -nwkey"
        [ -n "$WPAKEY" ] && OUTPUT="$OUTPUT wpakey $WPAKEY" || OUTPUT="$OUTPUT -wpakey"
        echo -e "$OUTPUT $OPTS\n$ADDR" > "$HFILE"
        break
    fi
done
```

Y el archivo `rc.wireless.conf` donde se configuran los diferentes
*profiles* (es a modo de ejemplo)

```sh
#!/bin/sh
# wireless configuration

IFACE="iwn0"
OPTS="-powersave"
ADDR="dhcp"

case "$MAC" in
    00:00:00:A0:A0:A0)
        NWID="hotspot"
        ;;
    00:00:00:B0:B0:B0)
        NWID="ap_wep"
        NWKEY="wepkey"
        ;;
    00:00:00:C0:C0:C0)
        NWID="ap_wpa"
        WPAKEY="wpakey"
        ;;
    00:00:00:D0:D0:D0)
        NWID="ap2_wpa"
        NWKEY="wpakey"
        OPTS="bssid 00:00:00:D0:D0:D0 chan 12 $OPTS"
        ADDR="inet 10.0.0.38 255.255.255.0 NONE"
        ;;
    *)
        return 1
esac
```