# Analizar la experiencia de carga de nuestras páginas
> Por desgracia no existe una métrica `perception.ready()`que nos suministren los navegadores, por lo que tendremos que decidir en qué
métricas nos fijamos para analizar la experiencia de un usuario respecto a los tiempos de carga de nuestras páginas.


## window-onload no puede ser la referencia
Cuando se pide un análisis de la experiencia de carga de una página, la métrica que con más frecuencia se emplea es el evento `window-onload` (evento `load` del window).

Sin embargo, es una métrica que no está en la ruta crítica para el pintado de la página (critical rendering path). El navegador puede comenzar el pintado de las página sin que se haya "lanzado" el evento `load`. 

Un buen ejemplo de esto es la [página de producto de Amazon] (https://www.amazon.com/gp/product/B008R5259Y/). 

![Amazon-initialview](https://cloud.githubusercontent.com/assets/22852880/22625483/74fafdb2-eb98-11e6-9d3e-5dfb68668f33.png)

![Amazon-times] (https://cloud.githubusercontent.com/assets/22852880/22625487/9a24645c-eb98-11e6-9c34-7c797714032f.png)

A lo 3sg al usuario se le muestra practicamente completa la primera vista de la página (primer pantallazo, o `above the fold`), mientras que el evento `load` se produce a los 28sg.


## Critical Rendering Path
Este concepto hace referencia a la serie de eventos que se deben producir para obtener la visualialización de la primera vista de la página, que representa mejor la experiencia de carga que percibe un usuario.

Cuando Google se refiere a "papespeed", no está hablando del tiempo que necesita el navegador para descargar todos los elementos de la página, sino a cómo de rápido el usuario comienza a ver el contenido de la página (`initial view`). A lo que también le añade el factor de cómo de rápido puede interactuar con dicho contenido.

> Our main concern when we talk about webpage speed is to get content to the user as soon as possible in the initial view

### Los conceptos

* **critical**: totalmente necesario
* **rendering**: hace referencia a que la página se nuestra en pantalla 
* **path**: ruta de eventos que llevan al navegador a mostrar el contenido de la página en pantalla
* **initial view**: la parte de la página visible al usuario sin que haya realizado ningún scroll. Se suele referir a ella como `above the fold`.

### El path

![CRP image](http://www.qingpingshan.com/uploads/allimg/161025/1T0316046-4.jpg)

#### La secuencia más sencilla de pasos hasta mostrar la página en pantalla es:

![CRP-noCSS-noJS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom.png)

1. el navegador solicita el documento html
2. una vez el navegador ha recibido el documento html, comienza a procesarlo para generar el árbol DOM que describe la estructura y el contenido de la página.
3. la construcción del DOM es un proceso que se ejecuta de modo gradual. El navegador va recibiendo el html en paquetes (TCP responses), y a medida que los recibe, los va parseando para ir generando el árbol DOM.
4. Una vez se ha completado el árbol DOM, ya se pueden ejecutar los procesos del render de la página: creación del rendering tree (se integra la información del DOM con la de estilos de los elementos del CSSOM (que en este caso no hay), el layout (calcular disposición y tamaño de los elementos en la pantalla), y el paint (covertirlo a pixels).

#### En el caso de que intervenga una hoja de estilos (imagines en fichero css externo), la secuencia de eventos es:

![CRP-noJS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css.png)

1. el navegador solicita el documento html
2. ha medida que recibe el documento, comienza la construcción del DOM
3. cuando detecta que se necesita un fichero css, lo solicita al servidor
4. recibido el fichero css, lo procesa para construir el CSSOM
5. a diferencia del flujo anterior, en esta ocasión el navegador deberá esperar a disponer del CSSOM para pasar a la etapa de renderizado de la página. 

Es lo que se denomina "render blocking CSS"

#### Por último, en el caso en que tb intervenga código javascript, la secuencia es más compleja:

![CRP](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css-js.png)

1. el navegador solicita el documento html
2. ha medida que recibe el documento, comienza la construcción del DOM
3. cuando detecta la necesidad de css y js, solicita estos recursos al servidor.
4. adicionalmente el navegador, al detectar código javascript detiene la construcción del DOM, que no se reanuda hasta que se haya ejecutado el script.
5. a su vez, la ejecución del script requiere de que se haya completado el CSSOM, por lo que queda bloqueado hasta que descarguen y procesen las hojas de estilo.

Este impacto en el critical rendering path se denomina "render blocking javascript"

#### Otros recursos, como imágenes
Aunque están fuera del CTR, cuando durante el parseo del HTML para la construción del DOM, el navegador detecta que se necesitan recursos adicionales, como las imágenes, éste las solicita, si bien, los procesos del pintado de los contenidos pueden comenzar sin esperar a que lleguen al navegador.

No bloquean los procesos de renderering, si bien sí pueden afectar a los tiempos de descarga de recursos `críticos`(css, javascripts), ya que sustraen ancho de banda, y consumen conexiones http (un navegador trabajando con el protocolo HTTP1 solo puede gesitonar 6 conexiones de modo simultáneo).


#### Render blocking CSS
Como se ha visto en la descripción de la secuencia del CRP, sin que se hayan descargado todas las hojas de estilo y se haya generado el CSSOM, el navegador NO puede pintar contenido en pantalla.

El número y tamaño de los ficheros css afectará por tanto al tiempo que necesita el navegador para mostrarle el primer pantallazo al usuario.

La excepción son las hojas de estilo cargadas a través de media queries. El navegador solo descargará y procesará aquellas que cumplan con la especificación del medio y las características que se hayan indicado en las reglas, ej.-
``` html
<link href="style.css" rel="stylesheet">
<link href="print.css" rel="stylesheet" media="print">
<link href="other.css" rel="stylesheet" media="(min-width: 1024px)">
```
En este ejemplo, la hoja de estilo `style.css` bloqueará el `CRP` siempre, mientras que `print.css` no intervendrá cuando se está visualizando la página en la pantalla, lo mismo con `other.css` que no lo hará en el renderizado para móviles.


#### Render blocking Javascript
Como se ha visto en la de la secuancia del CRP, el efecto del javascript puede ser peor, ya que bloquea el proceso de construción del DOM.

La excepción son los javascript de ejecución diferida, o a los que se aplica técnicas para su carga y ejecución diferida.

Lo más común es emplear el atributo en la etiqueta `async` o `defer`en la llamada a un fichero js.

En ambos casos le estaríamos indicando al navegador que no bloquee el procesado del DOM.

Adicionalmente, en el caso del atributo `async` estamos indicando al navegador que la ejecución del script no tiene que esperar al CSSOM. De ese modo, en caso de que se descargara el javascript antes de que se hubiera completado el CSSOM éste podría ejecutarse. Por supuesto tb antes de que se completase el DOM.

Por su parte, el atributo `defer` provoca que el javascript, aunque esté descargado, no se ejecute hasta después de que el DOM se ha completado.

Otras técnicas, que requieren cierta programación, persiguen diferir no solo la ejecución, sino la descarga del javascript haciendo que ésta no se produzca hasta que se ha completado la descarga completa de los elementos de la página, evento load de la página. Esta técnica podrá aplicarse solo a ciertos javascript, a aquellos que puedan ejecutarse de forma diferida sin afectar a la usabilidad de la página, ejemplo, ciertas analíticas, botones de redes sociales, etc. 

#### Javascript bloqueado por el CSSOM
Como hemos visto, el código javascript requiere del CSSOM para ejecutarse. Por un lado será importante asegurar una descarga lo más rápido posible de las hojas de estilo (pocos ficheros y de tamaño razonable), por otro será importante el orden en el que en el documento están las llamadas a los ficheros css y ficheros javascript, siendo la práctica general recomendada que estén las llamadas a ficheros css por encima de las llamadas a ficheros javascript.

## Métricas

### Eventos del navegador
A través de la API javascript: Navegation Timming API, es posible recoger datos sobre el performance de una página web.
Entre los eventos que recoge, podemos destacar:
* **domLoading**: indica el momento en el que comienza el parser a procesar los primeros bytes del HTML que ya ha recibido. Nos da por tanto información de tiempos de red hasta primeros bytes.
* **domInteractive**: indica el momento en el que el parser del navegador ha generado el DOM completo de la página. No significa que tb esté disponible el CSSOM, por lo que no indica que pueda comenzar el renderizado. Sí indica, que de existir, se han ejecutado los scripts síncronos, ya que estos interrumpen la generación del DOM. En este escenario, tb significará que se ha completado el CSSOM, ya que como sabemos la ejecución de estos scripts "síncronos" requieren que el CSSOM esté disponible.
* **domContentLoadedEventEnd**: este evento del Timming API, coincide con el evento del `DomContentLoaded` del DOM. Indica que además de que el DOM se haya generado, también se han ejecutado todos los javascripts que se habían encolado para ejecutarse una vez el parser haya terminado la construcción del DOM (los javascript con el atributo `defer`). El método `.ready()` the `JQuery` permite ejecutar código cuando se ha alcanzado este punto en el que el DOM se considera disponible.

> the page's Document Object Model (DOM) becomes safe to manipulate. This will often be a good time to perform tasks that are needed before the user views or interacts with the page, for example to add event handlers and initialize plugins.

Indicar que los ficheros javascripts con atributo `async` no quedan encolados como los `defer`para ser ejecutados cuando el parser haya terminado el DOM, sino que se ejecutan tan pronto están disponibles (se han descargado). Si el parser no ha completado el DOM, lo interrumpirán mientras se ejecutan (recordar que tampoco tienen que esperar al CSSOM), y si están disponibles una vez el parse ha terminado el DOM se ejecutarán en ese momento.

* **domComplete**: indica que el proceso con el documento principal a terminado, y todos los recursos (imágenes, etc) se han descargado. Este evento del `Timming API` si todo va bien coincide practicamente con el evento `load` del window (`onload`).

> El Navigation Timming API no tiene ningún evento relacionado con el render de la página, siendo el `domInteractive` el más cercano a indicar cuando el navegador puede comenzar a renderizar contenido.
Tan solo no se cumplirá en páginas en las que no haya scripts sincronos, algo realmente dificil de imaginar.
Como hemos visto, en este escenario podríamos tener el `evento domInteractive` sin que se haya completado el CSSOM.


> Es más habitual en las herramientas de medición, tener acceso al `evento DomContentLoaded`. Es un evento importante porque representa el momento en el que el documento es accesible y puede responder a la interactividad.
Sin embargo no necesariamente coincide con el momento en que el navegador puede comenzar las tareas de renderizado.
Como hemos visto, en caso de que haya una carga importante de javascripts con el `atributo defer`se puede dar la situación de que el pintado de los elementos de la página se inicien antes de que se ejecute el `evento DomContentLoaded`. 

### Otros indicadores
Pensando en un cuadro de mando automático, del apartado anterior recogemos como indicadores principales:
* domInteractive time
* domContentLoaded time
* load time

Y a estos indicadores deberíamos incorporar:
* first paint time, que no se obtiene del Navigation API Timming, y que requeriría trabajar con alguna librería javascript específica.
* first meaningful paint, :) Herramientas como GTMetrix, WebPageTest, y tb la propia devtool de Chrome.
* Main html size
* Total page size, que afectará al load time
* Requests, que también afectará al load time.

### Herramientas para análisis del rendimiento de los tiempos de carga
Obtenidos los tiempos, si lo que se quiere es realizar un análisis más profundo, las herramientas más empleadas son:
* Chrome devtools
* Google Pagespeed, https://developers.google.com/speed/pagespeed/ 
* GTMetrics, https://gtmetrix.com/
* Varvy tools, https://varvy.com/pagespeed/
* Webpagetest, https://www.webpagetest.org/ 
etc.


## Técnicas de optimización del CRP
A falta de desarrollarlo en mayor profundidad listamos algunas de las prácticas a analizar.

### Tiempos de red
* Activar la compresión de ficheros.
* gestionar el caché del navegador
* minificar los ficheros: html, javascript y css
* uso de CSS sprints para ciertas imágenes

### Priorizar la parte visible de la página
* UX de modo que el contenido visible en primera pantalla resulte relevante al usuario
* estructurar el HTML priorizando el contenido `above the fold`: ej1.- <content> por delante de <sidebar>, ej.- partir la estructura en una terraza que agrupe el contenido de la parte visible.
* reducir la cantidad de datos requeridos en este primer pantallazo

### Render blocking CSS
* evitar uso de @import para llamar a las hojas de estilo
* combinar ficheros de hojas de estilo, reduciendo el nº de ficheros css (mejor a uno)
* evitar el overhead de estilos, haciendo uso de media queries, y gestionando convenientemente el overhead que puedan introducir ciertos frameworks.
* inline CSS para el contenido "above the fold" 
* optimizar tamaño y cacheo de fuentes 

### Render blocking JS
* evitar o diferir los Javascripts :) que vayan a impactar en el CRP: async, defer o técnicas de carga 
* combinar ficheros Javascript

### Diferir carga de imágenes y videos
* Uso de lazy load para carga de imágenes
* Técnica similar para los videos, retrasando su carga tras el evento onload.

### Publicidad
* Uff!


## Hay más aspectos que intervienen en los tiempos de carga de las páginas
Nos hemos centrado en el análisis del CRP, es decir, los eventos que se producen entre que el navegador comienza a recibir el html del documento principal y se pinta el pantallazo inicial. 

Sin embargo, en los tiempos de carga de la página intervienen más aspectos:
* **los tiempos de renderizado**, aunque dentro del CRP, en este documento no hemos profundizado en los procesos de layout y paint de los contenidos
* **los tiempos de construcción del CSSOM**, que dependerá del tamaño y estructura de las hojas de estilo
* **los tiempos de contrucción del html en servidor**, que dependerá de la estructura de la página, y la programación de backend.
* **los tiempos de red**, que dependerá de la configuración de servidores, uso de CDNs, estrategias de minificación, cacheado en navegador, etc.

## Referencias
http://www.stevesouders.com/blog/2013/05/13/moving-beyond-window-onload/
https://developers.google.com/web/fundamentals/performance/critical-rendering-path/analyzing-crp
https://developers.google.com/web/fundamentals/performance/critical-rendering-path/measure-crp?hl=es-419
https://developer.mozilla.org/en-US/docs/Web/API/Navigation_timing_API
https://developer.mozilla.org/en-US/docs/Web/API/PerformanceTiming
http://stackoverflow.com/questions/3665561/document-readystate-of-interactive-vs-ondomcontentloaded
http://stackoverflow.com/questions/8996852/load-and-execute-order-of-scripts


