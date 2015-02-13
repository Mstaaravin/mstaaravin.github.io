---
layout: post
autor: Mstaaravin
title: Configurar apt.conf para no instalar paquetes recomendados/sugeridos en Debian
published: true
tags: apt.conf
category: [Debian, Tips]
---

Despues de tantos años de utilizar Debian en servidores y Desktops he llegado a tener un checklist de tareas que cumplo a rajatabla cada vez que necesito hacer una nueva instalación.

Este post tiene como finalidad compartir uno de estos tips:

## Administración de paquetes 

Amo Debian… 

Hago alarde de un verdadero derroche de satisfacción al utilizar Debian dia a dia pero no todo es color de rosas y como sabemos la perfección no existe.
Y es verdad que tengo que darle la razón a amigos que cordial y salvajemente me recuerdan a la mínima posibilidad de lo poco optimizado que está implementado el sistema de dependencias en Debian (y derivados), no es ningún secreto que una instalación promedio instala muchas mas cosas de las necesarias y eso hasta a mí me molesta porque con el tiempo la cantidad de librerias innecesarias crece incesantemente y los updates/upgrades bajan cada vez mayor cantidad de paquetes.

Por ese motivo incluso cuando instalo Debian desde cero en una notebook/desktop y siempre con el netinstaller, nunca jamás selecciono la opción “Desktop Environment”, sino que sólo dejo tildada la opción “Standard System Utilities”

![](/public/img/Wheezy-Virtual-Machine_501-500x294.png){: .img-center .responsive-image }

Siempre! hago un párate y termino la instalación con lo mínimo e indispensable, cuando reinicio por primera vez lo primero que hago es editar/crear el archivo /etc/apt/apt.conf y le agrego estas opciones:

{% gist 1359fff51993bf88c58e %}

En qué cambia todo…?
Es mucho mejor explicarlo con un ejemplo, algo que automáticamente hago apenas tengo shell post-instalación disponible es instalar esta lista de paquetes:

{% gist 0d18fa4ebec2074cbf3c %}

14.3 MB con un apt.conf vacío, veamos cuántos MBs bajaría indicándole al sistema que no instale recomendados ni sugeridos.

{% gist a9a178298b6148196157 %}

12.1 MB! sólo es una pequeña diferencia con tan poco paquetes, pero veamos hasta qué punto podemos optimizar esto instalando un entorno completo de escritorio como XFCE.

![](/public/img/Selection_006-500x200.png){: .img-center .responsive-image }

![](/public/img/Selection_007-500x209.png){: .img-center .responsive-image }

Sin la optimización en /etc/apt/apt.conf toda la descarga de XFCE serían 562MB contra los 324MB si no instalamos paquetes recomendados ni sugeridos, creo que está mas que clara la ventaja de utilizar esta configuración en un servidor para no solamente ocupar menos espacio sino que al tener menos paquetes y librerias disminuimos las posibilidades de sufrir vulnerabilidades en nuestro sistema.

