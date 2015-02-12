---
layout: post
autor: Mstaaravin
title: Cybermonday Argentina análisis del fiasco
published: true
category: Webservers
tags: Cybermonday Nginx Varnish Apache
---

Este es mi análisis netamente técnico de lo acontecido hoy durante el llamado Cybermonday anunciado en todas partes, pero lo mas destacado en mi opinión fueron 3 cosas.

En primer lugar lo notoriamente mal que ha funcionado la infraestructura de muchos sino la mayoría de los sitios.

En segundo lugar los paupérrimos descuentos, creo que la gente que trabaja en Marketing (he trabajado con ellos y los detesto) tienen lo que literalmente denomino “un moco en la cabeza”, no pueden ser tan pero tan pelotudos de pensar que la gente va a comprar algo a precios inflados cuando en USA o Europa aprovechan fechas como estas para sacarse de encima todo el stock viejo a como dé lugar pero aquí no, aquí intentan maximizar las ganancias como sea.

<!-- more -->

Y en tercer lugar las expresiones de quejas de la gente, se puede usar twitter para seguir minuto a minuto el desencanto y disconformidad del público como por ejemplo siguiendo este link: [cybermonday argentina](https://twitter.com/search?src=typd&q=cybermonday%20argentina)

Esta mañana apenas comenzó el horario laboral y en vistas de que prácticamente todo el mundo estaría accediendo a los distintos sitios publicados desde [http://cybermondayarg.com.ar/](http://cybermondayarg.com.ar/) me dediqué a hacer un rápido análisis de la configuración de esos sitios con un simple [curl](http://es.wikipedia.org/wiki/CURL) para obtener los headers de las páginas principales.

A medida que lo iba haciendo lo iba publicando en mi cuenta de [twitter](https://twitter.com/Mstaaravin) con un escueto screenshot del resultado pero sin explicar qué era cada cosa, porque básicamente el resultado de lo que mostraba al ejecutar


{% gist ec1b067043af47e5e85e %}

Sólo lo iban a entender este tipo de especímenes: sysadmins, diseñadores (de los buenos), programadores web y pará de contar.

Porqué me interesé en los headers…?
Bueno, veamos.

## Headers
Sólo una pequeña introducción a los headers que pueden leerla en wikipedia: [http://es.wikipedia.org/wiki/Hypertext_Transfer_Protocol](http://es.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
Otro link/imagen interesante que explica qué sucede cada vez que hacemos una petición http a un webserver es el siguiente:

  ![http headers]({{ site.eurl }}/public/img/6dwzl1.jpg){: .img-center .responsive-image }

Para los sysadmins o personas que trabajamos en infraestructura web, los headers de un webserver nos dan información muy valiosa sobre la configuración y comportamiento de estos servidores porque tenemos acceso a información que nos permite interpretar muy bien dónde hacer mejoras, encontrar fallas, etc en la configuración de estos mismos webservers.

La información que podemos obtener nos permite tomar deciciones correctas en el caso de proyectar la perfomance y desempeño de un webserver como veremos mas adelante.

## Webservers

Todo el que trabaja relaciondo con sitios de “alto” tráfico y me refiero por alto a mantener servidores con unas cuantas centenas de miles de visitas diarias conoce de términos como Proxys reversos, cache web, memcache, apc, etc. Todas estas son palabras de tecnologías relacionadas a nuestro trabajo y que nos facilitan el día a día.
Tambien sabemos todos los que trabajamos y mantenemos sitios de alta perfomance que Apache (el webserver mas utilizado) realmente no es muy eficiente que digamos en manejar muchas peticiones simultáneas, en otras palabras explota por todos lados. Es casi 100% compatible con casi todo lo publicable vía web (excepto .NET) pero como decía no es el Oooooh de los webserver en lo que a perfomance refiere.

Y cuando de perfomance pura estamos hablando es donde entran en juego otras palabras mágicas tales como nginx, Varnish, squid, etc. Los cuales son ampliamente utilizados como [proxies reversos](http://en.wikipedia.org/wiki/Reverse_proxy) que a grandes rasgos es como esto:

![Proxy reverso]({{ site.eurl }}/public/img/2juzbc.jpg){: .img-center .responsive-image }

Pero vayamos al quid de la cuestión que es la configuración.

Si, en el caso de hoy donde se sabía que la afluencia de visitas iba a ser bastante alta uno como sysadmin debe prepararse para eso y buscar obtener el máximo provecho que las distinas herramientas mencionadas ponen a nuestra disposición.

Varnish se utiliza para cachear en memoria contenido estático (imágenes, css, js, html, etc) para que sea más rápido “servir” ese contenido desde la ram y evitar sobrecargar al propio Apache procesando todas las peticiones dejando a este último el contenido dinámico.
Varnish es un excelente software pero para funcionar correctamente necesita de una gran cantidad de ram porque NO lee desde el disco todo el contenido.
Pero el contenido cacheado “desde” el webserver no es lo único importante como veremos.

Veamos algunos casos:

## caso Fravega
El sitio de Fravega fue uno de los que peor funcionó, en ningún momento del dia de las varias veces que intenté entrar estaba funcional.
Analizemos el resultado de los headers de la página principal.

![Fravega]({{ site.eurl }}/public/img/fravega.png){: .img-center .responsive-image }

O con Google Chrome por ejemplo al html principal.

![Fravega haders]({{ site.eurl }}/public/img/2088l5s.jpg){: .img-center .responsive-image }

Lo que realmente nos interesa de la información que vemos alli es que sí, estan utilizando Varnish y consideran que eso es mas que suficiente para mantener el sitio online con la avalancha de usuarios que ha habido hoy cuando han ignorado olímpicamente lo que he remarcado en rojo que es el Cache-Control de http.
Porqué digo que han desaprovechado el uso de Cache-Control…?
Bueno, [aquí](http://www.mobify.com/blog/beginners-guide-to-http-cache-headers/) [aquí](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) y [aquí](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields) tienen todos los detalles técnicos en profundidad.
Pero puedo decirles que habilitando la cache del contenido estático uno puede forzar a los browsers de los clientes y a todo proxy de contenidos que haya entre el webserver y el browser del cliente a mantener una copia en cada uno de los intermediarios.

Ahondemos un poco mas.
Todo contenido estático se puede cachear en 2 lugares distintos.

* “EN” el webserver
- “EN” el browser del cliente
- “EN/ENTRE” el medio de ambos

Cómo… uhh me perdí, no eran 2…?

En el primer caso que mencioné a Varnish, es un ejemplo de contenido estático cacheado en webserver. No es por supuesto el único que puede hacer esto yo particularmente tengo predilección por nginx (mi favorito) que tiene un método de cache que usa el filesystem en vez de ram como Varnish (tiene sus pros y sus contras) pero que usando cierto artilugios que todo Linux/Unix nos permite también podemos cachear contenido en la ram.

En el segundo caso que es cachear el contenido “en” el browser del cliente, es allí donde entran en juego los headers http que mencioné mas arriba, porque manipulando cómo se debe/permite cachear este contenido estático (.jpg, .png, .gif, .js, .css, .fonts, etc) **uno puede hacer que la infraestructura de los clientes y de muchas empresa trabaje para nosotros SIN sobrecargar nuestro webserver + cache server**

![Qué, cómo...?]({{ site.eurl }}/public/img/imgur_9CUtI.gif){: .img-center .responsive-image }

Veamos un ejemplo práctico con una imagen cualquiera alojada en este blog:

{% gist 75acbd31c135fc7a4ab7 %}

* **curl -I http://blog.ngen.com.ar/wp-content/uploads/2013/09/virsh_list.png** es el comando que ejecuté en mi Linux Desktop (by Debian) para obtener información sobre esa imagen.
- **Expires:** Me indica cuándo va a expirar esa imagen en la cache del cliente, en este caso el 10 de diciembre del corriente año. Esto significa que si el mismo usuario con el mismo browser ingresa mañana, pasado mañana o el 9 de Diciembre (día de mi cumpleaños dicho sea de paso) a mi blog y visualiza esa imagen, esta se mostrará desde la caché de su propio browser en vez de verla desde mi webserver y/o cache server (nginx, Varnish, squid, etc)
- **Cache-Control:** Justamente la duración en segundos del objeto en la cache del browser del cliente, en este caso corresponde a una semana.
- **Pragma:** public Directamente relacionado al siguiente ítem
- **Cache-Control:** public Este parámetro junto max-age es lo que produce la magia, hacer que el contenido estático pueda ser públicamente cacheado significa que cualquier instancia entre el webserver y el browser del cliente que sea capaz de cachear contenido pueda hacerlo, ejemplo: El proxy de una empresa, o el proxy de un ISP. Uno de los usos mas comunes y erróneos en mi opinión es definir este parámetro como “private” que significa que sólo la sesión (identificada con Etag + session id (en algunos casos) puede cachear el contenido o sea sólo el browser del usuario que pidió ese objeto, algo poco práctico en sitios con contenido estático de acceso público.

Con básicamente esos parámetros el 90% de los browsers y web proxies en el medio van a ser capaces de cachear contenido ahorrando tráfico y peticiones a nuestro webserver.
Hay por supuesto algunos parámetros específicos para controlar el comportamiento de los proxies como por ejemplo “Max-Forwards” que indica la cantidad de veces que el mismo objeto puede pasar de proxy a proxy antes de expirar y solicitarlo directamente al host que lo aloja.

Así como analizamos sólo un caso, pueden extrapolar este comportamiento al resto de todos los sitios que se subieron a la ola del cybermonday, algunos increíblemente ni siquiera disponían de un cache server, y como seguramente sucedió por incompetencia de mandos medios algún gerente o jefe de turno decidió que la mejor forma de prepararse para un dia de alto tráfico era aumentar la ram de/los webservers.

Utilizar todas estas herramientas que estan literalmente al alcance de los dedos de cualquier sysadmin no garantizan una perfecta perfomance de un website con alto tráfico pero ayudan bastante.

Puedo decirles por experiencia que trabajando a la par de un equipo de buenos desarrolladores (escasean bastante) he sido capaz de publicar un sitio hecho en WordPress con picos de 90.000 visitas diarias con un server en Rackspace Cloud con 1gb de ram consumiendo un máximo de 700mb.
Puede sonar increíble pero les aseguro que es perfectamente posible cuando lo que publicamos no tiene errores de de código y tenemos un memcache + APC trabajando a la par de una buena configuración de nginx.

![Shut UP and take my money]({{ site.eurl }}/public/img/43477830.jpg){: .img-center .responsive-image }

## Nginx

Aunque Varnish tiene su propio método para sobrescribir los headers de un webserver en backend aqui les muestro cómo hago esto con nginx.
Sólo tienen que editar la configuración de su VirtualHost y agregar algo como esto:

{% gist ca14c6d60cb8bf0d3473 %}


## Apache
Un buen sitio donde aprender a configurar correctamente todos los intringulis de Apache es este: [http://www.askapache.com/htaccess/htaccess.html](http://www.askapache.com/htaccess/htaccess.html)

Espero que este corto artículo les haya sido de ayuda, porque como dice la frase “El diablo está en los detalles”
