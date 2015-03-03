---
layout:   post
title:    "rTorrent: configuración y tips"
date:     2015-02-28
summary:  Uno de los mejores clientes en la terminal para torrents.
category: cli
---

[rTorrent][] es uno de los mejores clientes en la terminal para
manejar tus torrents. Lo utilizo hace muchos años y hace un tiempo
incorporó el manejo de magnet links.

Si nunca lo utilizaste te recomiendo leer la [rTorrent User
Guide](https://github.com/rakshasa/rtorrent/wiki/User-Guide).

![rTorrent in action](/images/rTorrent.png)

Mi archivo de configuración

```sh
min_peers          = 20
max_peers          = 200
max_uploads        = 50
max_uploads_global = 200

# Global upload and download rate in KiB. 
# "0" for unlimited.
download_rate      = 700
upload_rate        = 45

# Default directory to save the downloaded torrents.
directory          = /mnt/torrents

# Default session directory.
session            = /home/vando/.rtorrent
log.execute        = /home/vando/.rtorrent/rtorrent.log

# Close torrents when diskspace is low.
schedule = low_diskspace,5,60,close_low_diskspace=100M

# Ports and connections
port_random        = no
port_range         = 6890-6890
dht                = on
dht_port           = 6895
use_udp_trackers   = yes
peer_exchange      = yes
encryption         = allow_incoming,try_outgoing,enable_retry,prefer_plaintext

# Check hash for finished torrents. Might be usefull until the bug is
# fixed that causes lack of diskspace not to be properly reported.
check_hash         = yes

```

El mismo se encuentra en [mi repositorio de archivos de
configuración](https://github.com/vando/dotfiles) dentro de *ncurses/rtorrent*.

### Tips

#### Agregar torrents desde un directorio

Una manera fácil de controlar qué torrents estamos bajando y en qué
directorios deben ir los mismos

```sh
# Watch a directory for new torrents, and stop those 
# that have been deleted.
schedule           = watch_directory_1,5,5,"load=/home/vando/trwatch/*.torrent"
schedule           = watch_directory_2,5,5,"load=/home/vando/pr0nwatch/*.torrent,d.set_directory=/mnt/pr0n"
schedule           = untied_directory,5,5,stop_untied=
```

#### Ratio según el horario

Esto es ideal si utilizamos trackers privados para abrir el ancho de
subida cuando estamos durmiendo, esto es, cuando seguramente no
estemos teniendo acceso a nuestro home server.

Entre las 01:00hs y las 09:00hs el ratio de subida será de 100KiB pero
en otro horario será de 45KiB (esto se debe basar en las *prestaciones*
de tu ISP)

```sh
# Scheduling upload rate (for private trackers)
schedule           = throttle_high,01:01:00,09:00:00,upload_rate=100
schedule           = throttle_low,09:01:00,01:00:00,upload_rate=45
````

#### Nuevas vistas

Se puede cambiar la vista octava (*seeding*) ordenando por ratio de
subida

```sh
# Sort the seeding view by the upload rate and only show torrents with peers
view.sort_current = seeding,greater=d.get_up_rate=
view.filter = seeding,"and=d.get_complete=,d.get_peers_connected="
view.sort_new = seeding,less=d.get_up_rate=
view.sort = seeding
```

Y se pueden agregar dos nuevas vistas, novena y décima, una para
listar los torrents que estemos descargando y otra para listar
los activos

```sh
# Sort the leeching view by name
view.sort_current = leeching,greater=d.get_name=
view.sort_new = leeching,greater=d.get_name=
view.sort = leeching

# Filter the active view by connected peers
view.sort_current = active,less=d.get_name=
view.sort_new = leeching,less=d.get_name=
view.filter = active,d.get_peers_connected=
view.sort = active
```

#### Canvas patch

Es el único patch con colores que quedó para rTorrent, ideal para
diferenciar los torrents. (No agrego el link al mismo porque depende
de la versión de rTorrent que utilices.)

Para aplicar el patch

```sh
patch -uNp1 -i /usr/ports/network/rtorrent/rtorrent-0.9.4_color.patch
```

Y luego agregar al archivo de config

```sh
# canvas patch
color_dead_fg     = 1
color_inactive_fg = 7
color_active_fg   = 3
color_finished_fg = 5
```

#### Notificación de torrents completos

Con la siguiente línea le decimos que al finalizar un torrent ejecute
un comando y nos envíe el nombre y ruta al mismo

```sh
# Send an email for completed downloads
system.method.set_key = event.download.finished,notify_me,"execute=/home/vando/bin/torrent_notify,$d.get_name=,$d.get_directory="
```

Depende de donde tengas tu rTorrent corriendo puedes enviar un email o
un SMS o avisar con libnotify; en mi caso, que tengo rTorrent en un
server dentro de la LAN con otro server que utilizo para leer los
emails con mutt, uso el siguiente script

```sh
#!/bin/bash

echo "From: rtorrent-srvr <rtorrent@darkstar.lan>
Content-Type: text/plain; charset=utf-8
Subject: $1
Date: `date -R`

Path to torrent: $2" | \
ssh mac "cat > /home/vando/spool/mail/torrents/new/`date +%s.%y%m%d_1.mac`"
```

#### rTorrent remoto

Tengo un home server (`srvr0`) al cual accedo desde Internet por SSH y
dentro de él utilizo mutt e irssi pero rTorrent esta en otro equipo
(`srvr1`) que solo permite conecciones SSH por LAN.

Para ello configuré que cuando lanzo rTorrent en el server `srvr0`
este se contecte por SSH a `srvr1` y se inicie rTorrent o haga un
*reattached* del mismo.

Dentro de `srvr0` esta el alias

```sh
alias rtorrent='RTORRENT=1 ssh -o SendEnv=RTORRENT rtorrent'
```

Y dentro de `srvr1` hay que agregar al archivo `/etc/ssh/sshd_config`

```sh
PermitUserEnvironment yes
AcceptEnv RTORRENT
```

Y también agregar al archivo `.profile` de la cuenta que corra
rTorrent, si utiliza tmux

```sh
rtorrent(){ tmux new -s rtorrent -A "/usr/bin/rtorrent -o http_capath=/etc/ssl/certs" ; }
[[ "$RTORRENT" ]] && rtorrent

```

o si utiliza screen

```sh
rtorrent(){ screen -rmS rtorrent 2> /dev/null ; [[ "$?" -gt 0 ]] && screen -mS rtorrent /usr/bin/rtorrent -o http_capath=/etc/ssl/certs ; }
[[ "$RTORRENT" ]] && rtorrent
```
[rTorrent]: http://rakshasa.github.io/rtorrent/