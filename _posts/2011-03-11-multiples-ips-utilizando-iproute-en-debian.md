---
layout: post
autor: Mstaaravin
title: Múltiples IPs utilizando iproute en Debian
published: true
tags: tips interfaces 
categories: Debian Networking
---

En el ámbito de administración de servidores GNU/Linux, cuando se necesita especificar más de una IP en una interface se recurre generalmente los alias de interface,
los conocidos ethx:x\\
Aunque funciona perfectamente bien y la administración de las reglas iptables asociadas es relativamente fácil estamos dejando de lado una suma de funciones
poderosas a utilizar en un servidor GNU/Linux. 

Nunca me gustó utilizar alias, pocas veces lo he necesitado; Ahora justamente tengo en mi empresa un server con Debian “Squeeze” cumpliendo las funciones de gateway,
tenemos un enlace dedicado con un pool de 14 IPs públicas y necesito dejar todo listo para un próximo enlace dedicado a la par del actual,
utilizar alias realmente me limita en comparación con los beneficios a futuro si utilizo [iproute](http://lartc.org).  

Tengo hasta el momento 4 interfaces físicas en mi server 

 * eth0=ISP1
 * eth1=ISP2
 * eth2=lan
 * eth3=dmz

Como por el momento tengo solamente 1 ISP, y estoy utilizando al menos 3 IPs públicas del pool disponible y en el futuro inmediato necesito:

 * balanceo de carga
 * Wan failover
 * traffic shapping
 * múltilples rutas entre la (dmz, openvpn, lan, internet, etc)

La solución es integrar iproute con la estructura de networking en Debian.  

Debian utiliza el archivo /etc/network/interfaces para definir el estado de las interfaces y sus IPs, en este caso en particular definimos las IPs utilizando iproute,
debemos cambiar entonces el comportamiento modificando algunos valores.  

He aqui un ejemplo típico (**las IPs son ficticias**)  

{% highlight console %}
auto eth0
iface eth0 inet static
        address 190.139.257.1
        netmask 255.255.255.240
        network 190.139.257.0
        broadcast 190.139.257.15
        gateway 190.139.257.14
{% endhighlight %}

Utilizando iproute cambiamos el valor inet a “manual” quedando como el ejemplo que sigue.

{% gist c02f75f0367efa0406bf %}

Para confirmar que las IPs esta correctamente configuradas, iproute nos provee con el comando “ip” para verificarlo

{% gist e77ebdd7829108a24c3e %}


Eso es todo, posteriormente se puede agregar a las lineas de /etc/network/interfaces todo lo que necesitemos aplicar con iproute (rutas extras, traffic shapping, etc)

