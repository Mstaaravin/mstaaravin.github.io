---
layout: post
autor: Mstaaravin
title: KVM Full backups de virtual disks (qcow2) con virsh + rsync
published: true
tags: libvirt rsync backup qcow2
category: [Virtualization]
---

Cuando utilizamos KVM (qemu + libvirt) para virtualizar podemos utilizar diferentes métodos para los discos virtuales de las VMs (LVM, Raw, QCow2, etc) mi preferencia pasa por utilizar qcow2 como formato de disco virtual.

Por defecto, todos los archivos de discos virtuales se guardan en **/var/lib/libvirt/images**

En el ejemplo vemos una infraestructura de 3 servidores (Ardenas, Siracusa, Zama) que virtualizan con KVM y tienen N cantidad de VMs.

<!-- more -->

![](/public/img/bkpserver.png){: .img-center .responsive-image }

BKPServer es un servidor dedicado exclusivamente a tareas de backups el cual tiene (vamos a suponer) un partición con espacio suficiente para alojar todos los archivos de los discos virtuales que existen en Ardenas, Siracusa & Zama. El objetivo obviamente es disponer de un backup full de todas las VMs una vez a la semana y hacerlo de forma centralizada.
Para ello BKPServer debe conectarse a cada uno de los servidores, suspender las VMs , transferir desde los servidores a BKPServer los archivos *.img, conectarse a los servidores y hacer un virsh resume a las VMs.

## Requisitos

* red gigabit
* rsync en modo daemon en cada server KVM
* acceso mediante ssh-key desde bkpserver a cada server KVM
* crontab en bkpserver

## /etc/rsyncd.conf

Este es el archivo de configuración /etc/rsyncd.conf a modo de ejemplo que vamos a usar en los servidores KVM.
Les recuerdo que en Debian para iniciar rsync en modo daemon además de instalarlo deben editar el archivo /etc/default/rsync

No pretendo explicar cómo se configura rsync, solamente aclaro que el share “libvirt” permite sólo lectura y conexión desde la IP 172.16.10.234 que en este caso es el server BKPServer.

{% gist 1b899e3b68b978b9cbd3 %}

## SSH keys
Para que desde BKPServer se pueda hacer suspend & resume remotamente de las VMs vamos a generar un juego de ssh keys y agregarlas al ~/authorized_keys del usuario root en Ardenas, Siracusa & Zama.
Tampoco voy a explicar cómo generar un juego de llaves ssh, aunque pueden ver cómo hacerlo en este link: how generate ssh-keys

Dado que vamos a usar el usuario root para conectarnos, como medida de seguridad extra haremos 2 cambios.

Editaremos el archivo /root/.ssh/authorized_keys y agregaremos al comienzo de la key ssh un from=”172.16.10.234″ tal como se muestra a continuación para limitar el acceso con esa ssh key solamente desde BKPServer.

{% gist ce5f72db721dd2b22bcf %}

También pueden editar el archivo /etc/ssh/sshd_config y cambiar a “no” la opción PasswordAuthentication, pero mucho cuidado que no podrán acceder con usuario y password, si o si deberán hacerlo con ssh-keys y obviamente deben estar agregadas (previamente) a los ~/.ssh/authorized_keys de los usuarios con los que necesiten hacer login.

{% gist a13efb3afac89e0ebbb4 %}

![](/public/img/virsh_list.png){: .img-center .responsive-image }

## /usr/local/bin/virsh-snapshot.sh

Este es el script que realiza los backups y debe estar ubicado en BKPServer.
Lo que hace es conectarse mediante con virsh -c qemu+ssh, suspende cada una de las VMs, hace un rsync desde alguno de los servidores KVM a (en este caso) /var/lib/bulk/libvirt/public/img/, se vuelve a conectar con virsh -c qemu+ssh y hace un resume de la VM, dependiendo del tamaño que ocupe el *.img puede demorar unos pocos minutos a un par de horas.

En el script se considera que los archivos *.img se llaman exactamente igual que el nombre de la VM, que podemos ver con virsh -c qemu+ssh://172.16.10.200/system list –all

{% gist 45c2e1dcb15c7e9b687f %}

## crontab
Y por último necesitamos automatizar todo este proceso en BKPServer, que por supuesto lo haremos en el cron de root.

Tal como se muestra a continuación, se hacen 3 cosas.

1. Se borran de /var/lib/bulk/libvirt/public/img/ todos los *.img copiados previamente
2. A pesar de que usamos ssh-keys, cron no toma el path /root/.ssh/ por lo que debemos agregar la ssh private key con ssh-agent & ssh-add
3. Ejecutamos /usr/local/bin/virsh-snapshot.sh propiamente dicho que realiza toda la tarea

{% gist bc4bf671052ffbb4fdad %}

>Se entiende que si en los servidores KVM agregan/quitan VMs, deben editar el script /usr/local/bin/virsh-snapshot.sh para asegurarse de que backupean correctamente todas sus VMs.
>Tambien se aclara que este script sólo copia los *.img, no los *.xml que contienen la configuración de cada VM aunque fácilmente pueden agregar los cambios necesarios para guardar también los *.xml.

Para una completa referencia sobre el comando virsh, pueden consultar la documentación online: [http://libvirt.org/sources/virshcmdref/html/](http://libvirt.org/sources/virshcmdref/html/)


