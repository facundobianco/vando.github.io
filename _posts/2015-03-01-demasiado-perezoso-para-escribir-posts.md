---
layout:   post
title:    Demasiado perezoso para escribir un nuevo post
date:     2015-03-01
summary:  >
    Un script para crear nuevas entradas en el blog y que luego es 
    abierto en nuestro editor preferido.
tags: cli script
---

Te resulta cómodo utilizar
[markdown](http://daringfireball.net/projects/markdown/) para crear
nuevos posts en tu blog pero encontrás que se podría automatizar parte
de ello? Bueno, es lo que acabo de hacer con un simple script.

Primero, hay que estar dentro del directorio de nuestro sitio y luego
alcanza con elegir el nombre del *:title* ([no
confundir](http://jekyllrb.com/docs/permalinks/) con el título que
aparecerá en el documento)

```sh
newpost demasiado perezoso para escribir post
```

Crea, en este ejemplo, el archivo
`_posts/2015-03-01-demasiado-perezoso-para-escribir-posts.md`
con el contenido

```
---
layout:   post
title:    
date:     2015-03-01
summary:  
tags: 
---
```

y lo abre con el editor que tengamos defindo en la variable de entorno
`EDITOR`.

El script

```sh
#!/bin/bash

if [ ! -d _posts ]
then
    echo "Not the repository of your site"  
    exit 1
fi

IFS='-'
DATE=`date +%F`
FILE="_posts/$DATE-$*.md"

echo "---
layout:   post
title:    
date:     $DATE
summary:  
tags: 
---
" > "$FILE"

"$EDITOR" "$FILE"
```