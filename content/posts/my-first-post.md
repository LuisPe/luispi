---
draft: false
title: "De los errores se aprende, si se comparten, se crece"
description: "Compartir y adquirir conocimiento debería ser la guía."
date: 2022-06-05T21:53:34-03:00
tags: ["reflexion"]
---
En nuestra sociedad en general y en tecnología en particular tenemos la extraña costumbre de publicar los aciertos
(opinión de quien redacta, mal llamado **caso de éxito**) y pocas, por no decir nulas, las ocasiones donde se comparten los 
errores (opinión de quien redacta, mal llamado **fracaso**).

Como estarás sospechando a continuación voy a compartir algunos errores, que aprendí de ellos y más aún cuanto crecí y 
entendí de la industria al compartirlos.

## Enseñanza 1:

>Escribir un paso a paso, compartirlo, validarlo y re-validarlo antes de ejecutar nada

Mi primer desarrollo productivo fue en 2012 (?) aproximadamente, se trató de una web para una empresa que necesitaba
publicar su catálogo de productos, enumero tecnologías para tener un poco de contexto:
- digitalocean, recién salido, además de los droplets no se cuantos productos más tenía en cartera
- angular js (angular 1)
- creo que nodeJS para el backend

Este desarrollo lo hice con quien es una de las personas que más respeto y admiro, Víctor De Grandis –Vitor–. 

Mi experiencia configurando servidores era nula, tuve la delirante idea de decirle a Vitor:

_che amigo, ¿puedo configurar el servidor y después intentas romperlo?_

Acto seguido me pongo a realizar varias configuraciones entre ellas me puse a jugar con los puertos, hasta que muy 
orgulloso le digo:

_Vitor, probá a ver qué onda_

Después de un rato me llega un mensaje,

_amigo ¿puede ser que cerraste el puerto 22?_

Hermoso, salí de la casa, cerré la puerta y tiré la llave a una laguna.

Como la gente de digitalocean está preparada para personas como quien redacta, tiene una funcionalidad que si se accede 
desde la web (no tengo mucha idea como se resuelve por detrás) podes ejecutar una terminal en el navegador y acceder al droplet.

Configure de nuevo los puertos, y el resto medio que es historia.

## Enseñanza 2:

> Documentar el conocimiento por más trivial que lo consideremo e invitemos a que el resto lo haga. Además de evitar 
> varias reuniones de “transferencia de conocimiento” nos vamos a ahorrar varios arrobas, ¿te recuerda algo?
> 
>@fulanito puede darte una mano

En otro equipo que tuve la fortuna de trabajar, pequeño en cantidad de personas pero de las cuales aprendí mucho mucho 
de cada una de ellas, lamentablemente la aventura fue muy corta en tiempo.

Lo que sí me extrañó mucho a medida que uno a uno se iba yendo gente del equipo es que perdíamos mucho conocimiento a 
punto tal de parar motores (literal) y estar semanas sin entregar funcionalidades a producción porque tuvimos que sentarnos 
a entender cómo dejar esa línea de código local y exponerla a internet.

¿Qué errores detecte y que aprendí?

Documentar el conocimiento, por más trivial que lo consideremos, documentarlo.

O acaso, y te invito a pensar durante unos segundos. ¿te encontraste una o más veces haciendo reuniones de 
“knowledge transfer” porque alguna persona estaba dejando el equipo?

¿o acaso eras vos quien lo estaba dejando?

## Enseñanza 3:

>Ascender no debería ser la única o principal meta, compartir y adquirir conocimiento debería ser la guía.
>
>Enseñanza 3 bis:
> 
>Pensar y realizar pruebas antes de implementar cualquier cosa es una guía fundamental para entender el comportamiento de 
> nuestros sistemas o del conocimiento del dominio que tenemos hasta el momento. Siempre y repitamos, **siempre**, 
> es una ganancia en el tiempo.

Por último y en otro lugar que trabajé, conocí a una de las personas que mejor resuelve problemas, Juan Moreno,
¿qué es mejor para mí?, soluciones simples a problemas complejos.

Después de este pequeño paréntesis y dedicatoria a quien hoy es una de mis fuentes de consulta, voy a intentar explicar 
la enseñanza número 3 y 3 bis.

En los tiempos pre pandemia, solía –costumbre que aun conservo desde lo remoto– arrancar temprano, tener un par de horas 
“a solas” me permite focalizar y priorizar en la medida de lo posible las cosas que quiero y me comprometí a realizar.

Una de esas mañanas y en consecuencia a una funcionalidad que estábamos desarrollando en el equipo el líder llegó un 
poco enojado (por ser amable) y agarró a los primeros que encontró del equipo, pudiendo así descargar su enojo porque 
había “quedado mal” ante sus superiores porque de todas las casuísticas había un caso que rompía.

Famoso _Take a breath_ y al cabo de unos minutos me junte con él a solas, le hice una pregunta:

_¿cuántas personas están involucradas en esta funcionalidad?_

Entendió con la pregunta que debía haber esperado y comentarle a todo el equipo responsable lo que había pasado y 
no agarrar a los primero que se cruzó en el camino.

Por último le dije que comprendía pero no compartía su enojo ni como lo había manejado.

Quiero aclarar que mi intención no es juzgar, sino compartir que aprendí de ese error, de nuevo:

>Ascender no debería ser la única o principal meta, se corre el riesgo de estar bajo una enorme presión por no estar a 
> la altura de las circunstancias. Compartir y adquirir conocimiento debería ser la guía.
> 
> Pensar y realizar test antes de implementar cualquier cosa es la guía fundamental para entender el comportamiento 
> de nuestros sistemas o del conocimiento del dominio que tenemos hasta el momento. Siempre y repitamos, **siempre**,
> es una ganancia en el tiempo.

Para finalizar entiendo que voy a seguir cometiendo errores y compartiendo la experiencia que me dejaron, 
eso no me hace ni peor ni mejor que nadie.

Estoy convencido que es una sana manera de transitar la vida donde lo “normal” es mostrarnos “exitosos” o 
como me gusta hacer paralelismos con el fútbol, “jugar para la tribuna”, pero a fin de cuenta es solo una apariencia.
