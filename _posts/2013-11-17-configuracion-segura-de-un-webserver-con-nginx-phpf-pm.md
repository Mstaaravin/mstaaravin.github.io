---
layout: post
autor: Mstaaravin
title: Configuración segura de un webserver con nginx + php-fpm
published: true
tags: nginx php-fpm
category: [Webservers]
---

Desde la semana pasada estoy como sysadmin freelance de una serie de webservers de una empresa de diseño que hostea los sitios webs de sus propios clientes.

El objetivo es actualizar, optimizar y asegurar esos pocos webservers (menos de 10) y justo cuando tomé el control de los mismos me estreno con un deface a TODOS los sitios de varios de ellos apareciendo una portada como la siguiente:

![](/public/img/defaced1.jpg){: .img-center .responsive-image }

Por supuesto, se fue al garete todo el plan de upgrades que tenia pensado y tuve que ponerme a trabajar contrareloj para solventar este inconveniente y permitir que los sitios esten nuevamente online.
En un principio sólo restauré archivos borrados y databases y los sitios volvieron a estar online pero a las pocas horas nuevamente volvieron a comprometer los sitios asi que había una cuestión de fondo que resolver.

> Para el ejemplo sólo usaré dominios bajo mi control para mantener en privado los reales sobre los que realizé el trabajo

En la GRAN mayoría de los casos cuando nos encontramos con sitios defaced, la causa es una combinación de frameworks sin actualizar y por consiguiente vulnerables (WordPress, Joomla, Drupal, etc) + algo típico en sysadmins sin mucha experiencia: servers mal configurados.

Este caso no fue la excepción por lo que nada podía hacer si la propia empresa no actualizaba esos WordPress & Joomla (la mayoría) que estaban siendo vulnerados a cada momento.
Por lo que tuve que implementar una solución para al menos mitigar estas vulnerabilidades y en el caso de que comprometan un sitio, no puedan acceder a los demás.

Esto último lo explico de esta forma:
Por defecto (incluso yo mismo he hecho esto en el pasado) la mayoría de los webservers con apache+php o nginx+php tienen definido en el php.ini una variable open_basedir que en el caso de los servidores Debian es /var/www esto hace que si logran comprometer un sitio y NO estan deshabilitadas las funciones exec,passthru,shell_exec,eval,system los defacers pueden ganar acceso a otros directorios donde se alojen otros sitios y poder comprometerlos sin necesidad de que esos otros sitios tengan vulnerabilidades php, sql injections, etc.
Se entiende entonces la importancia de definir correctamente variables y funciones en la configuración de PHP para aumentar las chances en la seguridad de un sitio.

Lo primero fue dar de baja Apache+PHP e instalar nginx+php-fpm, los servidores son Debian Squeeze por lo que tuve que instalar todos los paquetes desde los repositorios de 
[dotdeb](http://www.dotdeb.org/instructions/)

## Usuarios
Una vez instalado nginx+php-fpm quería ejecutar un pool por cada site y con usuarios (**sin privilegios**) y paths distintos, para lo cual tuve que crear usuarios nuevos iguales a nobody (por ejemplo) comenzando con el usuario y grupo.

{% gist a40204d5aea4cd795811 %}

> Nota importante, el usuario que acabamos de crear tiene el mismo perfil que nobody:nogroup

## Permisos
Si el sitio se publica desde /var/www/site1.ngen.com.ar debemos cambiar los permisos y owner de todos los archivos y directorios.

{% gist 8cf76a608c026f30e9d1 %}

## Pools
Llegó el momento de configurar un pool, cuando se instala php-fpm por defecto se crea un pool **"www"** en **/etc/php5/fpm/pool.d/www.conf**, podemos mantenerlo, modificarlo o crear uno nuevo y tantos como necesitemos y/o soporte nuestro server.

Si decidimos crear un nuevo pool con hacer una copia de ese archivo (/etc/php5/fpm/pool.d/site1.conf) y editarlo es suficiente, los valores a editar van a ser:

{% gist ec610d96e71e27bef91b %}

> listen = /var/run/php-fpm/site1.sock es para diferenciar los distintos sockets de cada pool.

Una vez que tienen aceitado el procedimiento para crear distintos pooles separados con sus usuarios pueden ver los permisos de los directorios de cada sitio y los procesos (con su owner).

{% gist 32d9b9403a9c376f0a7a %}

## php.ini
Una de las ventajas que nos da utilizar php-fpm es que podemos ser lo más restrictivos posibles en la configuración usada en **/etc/php5/fpm/php.ini** y a medida que necesitemos determinada configuración mas permisiva podemos habilitarla en forma individual en cada pool.

Por lo que en este caso reemplazo la configuración x default de estos valores.

{% gist 42273e7d44fccc1507ad %}

## pool.d/blog.conf
Esta es la configuración que estoy utilizando en este blog a modo de ejemplo, donde muestro qué valores utilizo y cómo aplicar cambios para sobreescribir el php.ini el cual tiene una configuración MUY restrictiva.

{% gist 032ff7dc7fd0bca01f1b %}

Es importante definir en los pools valores de php distintos a los que hay en **/etc/php5/fpm/php.ini** sino muchas aplicaciones no funcionarán correctamente, tal es el caso del valor “sql.safe_mode” con WordPress que debe estar en off para que funcione.

> Tambien es **MUY** importante el valor “open_basedir” porque en combinación con el “chroot” del propio php-fpm aísla el acceso de la aplicación php a solamente el path que definamos, en el ejemplo /var/www/blog.ngen.com.ar y no se permite que los scripts php puedan escalar directorios.

Con estos simples pasos pude aislar exitosamente los distintos sitios de los servers, en todos los casos excepto en 1 pude evitar que los sitios se vuelvan a ser comprometidos, ademas tuve que probar y flexibilizar bastante los valores definidos en “disabled_functions” para que otros sitios funcionaran correctamente, eso depende de cada caso en particular.
Por lo menos en el caso del único sitio que quedó vulnerable (Joomla 1.1) y volvía a ser comprometido una y otra vez no afectó el normal funcionamiento de los demás, y en cuestión de dias fue actualizado y no presentó mas problemas.

Hay formas de aplicar el mismo tipo de seguridad con Apache+php-fpm pero prácticamente no utilizo Apache asi que no sé cómo hacerlo.

Algo que también se puede implementar para incrementar la seguridad de nginx y de las aplicaciones es utilizar naxsi un WAF (Web Application Firewall) similar a mod-security para Apache, algo que estoy implementando en este blog y testeando antes de ponerlo en producción en otros servers.

## Links de interés
<http://php.net/manual/en/install.fpm.configuration.php>
<http://www.cyberciti.biz/tips/php-security-best-practices-tutorial.html>
<http://myjeeva.com/php-fpm-configuration-101.html>
<https://docs.newrelic.com/docs/php/per-directory-settings>