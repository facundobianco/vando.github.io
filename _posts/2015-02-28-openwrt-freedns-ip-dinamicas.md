---
layout:   post
title:    "OpenWRT + FreeDNS: como lidiar con las IP dinámicas"
date:     2015-02-28
summary: > 
    Un script sencillo dentro de OpenWRT para actualizar nuestra 
    IP dinámica con FreeDNS. A diferencia de otros scripts yo utilizo 
    la herramienta uci.
category: net, cli
---

Los mejores router hogareños que he conocido son los Linksys
[WRT54G][] (inclusive los que son versión >= 5.0 con la mitad de RAM y
ROM) siempre y cuando corran [OpenWRT][].

Si bien es cierto que [OpenWRT][] no ofrece en su configuración alguna
forma de actualizar nuestra IP externa -como sí lo hace
[DD-WRT][]- esto se puede solucionar con un sencillo script.

A diferencia de otros scripts que hay en la red yo utilizo la
herramienta [`uci`](http://wiki.openwrt.org/doc/uci) para consultar
y/o guardar la nueva IP. Y me sirvo de [FreeDNS][] como servidor DNS
(funciona excelente y ofrece gratis la administración de hasta cinco
domain names).

El script es 

```sh
#!/bin/sh

IP=`uci -P /var/state get network.wan.ipaddr`

if [ "$IP" != `uci -P /var/state get network.wan.lastipaddr` ]
then
   wget -qO /dev/null http://freedns.afraid.org/dynamic/update.php?YOUR_TOKEN

   # Got freeDNS my new IP? Ask to openDNS!
   sleep 10
   if [ `nslookup your_domain.com 208.67.222.222 | sed -n '${s/.*: \(.*\) .*/\1/p}'` == "$IP" ]
   then
      date "+%Y%m%d %R  $IP" >> /var/log/ip.log
      uci -P /var/state set network.wan.lastipaddr="$IP"
   fi
fi
```

Nótese que antes de actualizar la variable `network.wan.lastipaddr` se
guarda en un *log* la fecha y hora y la IP que ha sido asignada; esto
se puede quitar.

Por último agregamos al *crond* de [OpenWRT][]

```sh
*/30 * * * * /usr/local/bin/freedns
```

### Tip

A veces me encontré que la interfaz WAN estaba caída. Para ello
agregué las siguientes líneas al inicio del script

```sh
if [ `uci -P /var/state get network.wan.up` != 1 ]
then
    /etc/init.d/network restart
    exit 1
fi
```

El script se encuentra en [mi repositorio de
scripts](https://github.com/vando/scripts) con el nombre de
*freedns.sh* (dentro del directorio *misc*).


[WRT54G]: https://en.wikipedia.org/wiki/Linksys_WRT54G_series#WRT54G
[OpenWRT]: http://wiki.openwrt.org/about/start
[DD-WRT]: http://www.dd-wrt.com/site/index
[FreeDNS]: http://freedns.afraid.org
