---
layout: post
autor: Mstaaravin
title: Laboratorio iPXE Flisol 2014
published: true
tags: Flisol iPXE dnsmasq Libvirt
categories: Flisol KVM Tips Virtualization
---

Este Sábado 26 de Abril se realiza en Buenos Aires el “Festival Latinoamericano de instalación de Software Libre” o Flisol para simplificar.

Como en muchas ocasiones en ediciones anteriores me he encargado de la configuración y administración de red, mirror, repositorios, etc (con mayor o menor éxito) aunque puedo decir que cada año he utilizado una configuración cada vez mas sofisticada.

Para este año he decidido publicar con tiempo toda la configuración que usará el server el día del evento en un repositorio de Github con la idea de que cualquiera pueda armar algo similar y con la remota esperanza de que también sea de utilidad a otras sedes, pueden verlo en el siguiente link:
<https://github.com/Mstaaravin/flisol-ipxe>

Pero he aqui que no todo el mundo tiene la posibilidad de disponer de hardware dedicado (switches, PCs, etc) a armar un laboratorio para probar que el booteo por PXE funcione para todas las distribuciones por lo que decidí simplificar utilizando KVM + Libvirt mas un pequeño hack en la configuración de red que utiliza libvirt para simular una red real.

Utilizo KVM para virtualizar y libvirt + virt-manager para administrar todas las VMs en mi desktop/server personal.

Por default cuando utilizamos KVM tenemos disponible una red “default”

{% gist 786020e4e35b89f25a58 %}

Que podemos ver con mas detalles en virt-manager

![](/public/img/lhome-Connection-Details_001-500x390.png){: .img-center .responsive-image width="500px"}

El objetivo de este post no es explicar todo lo relacionado a la configuración de red que utiliza libvirt, sino modificar la red x default para que nos permita tener booteo PXE en esa red virtual.

Lo primero que debemos hacer es detener la red y su bridge asociado (virbr0)

{% gist 5ee8293afe87818b48dc %}

Vemos la configuración que está usando por default

{% gist 7105597b251b88f97cbd %}

Y le agregamos soporte para PXE con 2 nuevos parámetros
"tftp root=" y "bootp file="

Hay 2 formas de editar la red (siempre que esté apagada), con:

{% gist 869b7c90730f1fe5bbae %}

O manualmente en _/etc/libvirt/qemu/networks/default.xml_

Y debería quedar así:

{% gist e54666edeff65f3f4789 %}

_tftp root='/home/tftp'_ Por supuesto es el root path donde tenemos todos los archivos necesarios para que PXE funcione, hay muchos tutoriales en internet que explicar cómo utilizar dnsmasq (es lo que utiliza libvirt para sus redes) también pueden descargarse toda la configuración desde mi repositorio github mencionado al inicio del post.

Probamos que funcione (no olvidar seleccionar PXE como primera opción de boot en virt-manager)

<iframe width="420" height="315" src="https://www.youtube.com/embed/v3MiUHa1Fzw" frameborder="0" allowfullscreen></iframe>
