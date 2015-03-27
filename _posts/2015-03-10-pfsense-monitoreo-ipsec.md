---
layout:   post
title:    "pfSense: monitoreo de conexiones IPSec"
date:     2015-03-10
summary:  >
    Un script en PHP para pfSense que monitorea los enlaces IPSec y 
    notifica si los mismos están caídos.
tags: net script
---

El inconveniente con los enlaces IPsec en
[pfSense](https://doc.pfsense.org/index.php/IPsec_Status) es que nos
entereamos están caídos recién cuando dejan de funcionar; esto es,
cuando algún usuario nos reporta problemas de conectividad.

Desde los [docs](https://doc.pfsense.org/index.php/IPsec_Status) la
única forma de monitoreo que ofrecen es observando la pestaña de
*IPSec Status* pero encontré en el
[foro](https://forum.pfsense.org/index.php/topic,42025.0.html) un
script para monitorear los enlaces y modificar su *status* según
sea la configuración.

Yo cambié ese script por uno más sencillo que únicamente notifica por
email si el enlace esta caído. La configuración de email que utiliza
es la definida en *SMTP E-Mail* dentro de *System* -> *Advanced* ->
*Notifications*.

El script

```php
<?php
require_once("util.inc");
require_once("functions.inc");
require_once("globals.inc");
require_once("config.inc");
require_once("notices.inc");

$host=array_pop($argv);
if (! is_ipaddr($host)){
        print "invalid ip address!\n";
        exit(1);
}
array_shift($argv);
$args=implode(" ", $argv);

exec("/sbin/ping -c 4 -t 1 $args $host",$ret,$exit);
if ($exit != 0){
        $message = "Visit https://pfsense.urdomain.com/diag_ipsec.php";
        $subject = "IPSec: Link offline $host\n";
        send_smtp_message($message, $subject);
}
?>
```

Se utiliza desde la terminal 

```sh
checklink.php -S ip_src ip_dst
```

Para varios enlaces se puede llamar desde un script en bash

```sh
#!/bin/sh

for HOST in ip_dst0 ip_dst1 ip_dst2 ip_dst3
do
        /usr/local/bin/php -q /root/checklink.php -S ip_src "$HOST"
done
```

Y lo utilizamos dentro de `/etc/crontab`

```sh
*/15    *       *       *       *       root    /bin/sh /root/checklink.sh
```
