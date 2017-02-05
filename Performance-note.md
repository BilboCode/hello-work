# Analizar la experiencia de carga de nuestras páginas
> Por desgracia no existe una métrica `perception.ready()`que nos suministren los navegadores, por lo que tendremos que decidir en qué
métricas nos fijamos para analizar la experiencia de un usuario respecto a los tiempos de carga de nuestras páginas.


## window-onload no puede ser la referencia
Cuando se pide un análisis de la experiencia de carga de una página, la métrica que con más frecuencia se emplea es el evento `window-onload` (evento `load` del window).

Sin embargo, es una métrica que no está en la ruta crítica para el pintado de la página (critical rendering path). El navegador puede comenzar el pintado de las página sin que se haya "lanzado" el evento `load`. 

Un buen ejemplo de esto es la [página de producto de Amazon] (https://www.amazon.com/gp/product/B008R5259Y/). 

imagen Amazon 1

imagen Amazon 2

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

La secuencia más sencilla de pasos hasta mostrar la página en pantalla es:

1. el navegador solicita el documento html
2. una vez el navegador ha recibido el documento html, comienza a procesarlo para generar el árbol DOM que describe la estructura y el contenido de la página.
3. el navegador descarga y procesa todas las hojas de estilo que se encuenta al procesar el html, y generar a partir de ellas el árbol CSSOM que describe los estilos que se aplicará a cada nodo del DOM.
4. de igual modo, el navegador también iniciará la descarga de cualquier imagen o subframe que encuentre en el documento html, si bien no estarán en el camino 
4. una vez que están generados el DOM y CSSOM el navegador consrtuye el render tree, que combina el contenido de la página con la información de los estilos que habrá que aplicarles.
5. el layout consiste en calcular a partir del render tree y el tamaño de pantalla la información de ubicación y tamaño de los contenidos en pantalla.
6. el último paso es el de convertir esa información en pixels en el monitor.

Sin embargo, debido a las dependencias que se producen para la construcción del DOM y del CSSOM el proceso es algo más complejo, y son estas dependencias las suelen tener un mayor impacto en el performance del critital rendering path.






> las imágenes

Aun cuando no forman parte del rendering critical path, para tener una visión completa del proceso que ejecuta el navegador, hay que indicar que durante el paso 2, el navegador iniciará la descarga de las imágenes y subframes que encuentre en el documento html. 

No bloquean los procesos de renderering, si bien sí pueden afectar a los tiempos de descarga de recursos `críticos`(css, javascripts), ya que sustraen ancho de banda, y consumen conexiones http (un navegador en http1 solo puede gesitonar 6 conexiones de modo simultáneo).




Before loading the CSS and constructing the full CSSOM nothing can be displayed to the client. Therefore CSS is called render blocking.
JavaScript (JS) is even worse, because it can access and change both the DOM and CSSOM. This means that once a script tag in the HTML is discovered, the DOM construction is paused and the script is requested from the server. Once the script is loaded it cannot be executed before all CSS is fetched and the CSSOM is constructed. After CSSOM construction JS is executed, which as in the example below may access and alter the DOM as well as the CSSOM. Only then the construction of the DOM can proceed and the page can be displayed to the client. Therefore JavaScript is called parser blocking.

Reduce critical ressources: Critical ressources are those needed for the initial rendering of the page (HTML, CSS, JS files). These can be highly reduced by inlining CSS and JS needed for rendering the portion of the website visible without scrolling (called above the fold). Further JS and CSS should be loaded asynchronously . Files that cannot be loaded asynchronously can be concatenated into one file.
Minimize bytes: The number of bytes loaded in the CRP can be highly reduced by minifying and compressing CSS, JS and images.
Shorten CRP length: The CRP length is the maximum number of consecutive roundtrips to the server needed to fetch all critical resources. It is shortened both by reducing critical resources and minimizing their size (large files need multiple roundtrips to fetch). What further helps is to include CSS at the top of the HTML and JS at the bottom , because JS execution would block on fetching CSS and building the CSSOM and DOM anyway.



## Métricas









Esto es más evidente en páginas de gran tamaño, con muchos elementos





sino que informa del tiempo que necesita el navegador para descargarse todos 
Este evento lo dispara el navegador cuando se ha completado la descarga de todos los elementos de la página (hojas de estilo, scripts, imágenes, y subframes: iframes, embed, objects). *tb img lazyload, y js asynch?*

Sin embargo, el navegador no requiere haber descargado todos los elementos de la página para comenzar el pintado. Así por ejemplo,
las imágenes no bloquean el que se pueda mostrar el contenido de una página, mientras que sí bloquean al evento `onload`.
Es decir, este evento no está en la ruta crítica para el pintado de la página.

Existen multitud de ejemplos en los que se muestra un primer contenido al usuario, mientras que el evento onload da tiempos muy superiores.

* página de producto de Amazon
https://www.amazon.com/gp/product/B008R5259Y/

* página de 

Ojo el above de fold is just for lab, with limitations.


el contenido rápido  en las que 

http://yellowlab.tools/result/emtze8kszd
https://www.keycdn.com/blog/website-speed-test-tools/


![warwefall-dom](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/waterfall-dom.png)

En este ejemplo se puede ver que mientras el evento domContentLoaded se produce sin esperar a la descarga de la imagen, el evento load sí espera a que la imagen se ha descargado.

Es el evento domContentLoaded el que indica que el navegador ha generado los árboles DOM y el CSSOM del documento, y que por tanto ya puede crear el árbol de visualización (render tree), paso previo a comenzar con el pintado de la página. 


Resulta que podemos crear el árbol de publicación e incluso dibujar la página sin tener que esperar a cada elemento de la página
no todos los recursos son esenciales para proporcionar un primer esbozo rápido



Después, 

Es una métrica que excluye lo que sucede con el render (pintado) de la página

sobre los que no se
haya aplicado alguna técnica de "carga diferida", ej.- lazyload para imágenes, asynch para los javascripts....


## Ruta crítica de publicación (critical rendering path)



## Otras métricas

* document.readyState: loading

* document.readyState: interactive

* document.readyState: complete

### document.readyState: loading
El navegador tiene el `html` (se ) y lo está procesando.
Aún no está descargando, ni procesando `css`, `javascript`, `imágenes`, y `subframes`.
El `Navigation Timing API`  lanza un evento `domLoading` tan pronto el `readyState` del documento pasa a `loading` 
Resulta útil como métrica que indica que el navegador ya tiene el documento `HTML` y va a comenzar a procesarlo.

### document.readyState: interactive
Este estado se alcanza cuando el navegador ya ha procesado completamente el `html` y se ha construido el `DOM` resultante de
dicho html.
Es el punto en que se inicia 


ya tiene el `HTML` y lo está procesando.



Buscamos medir cómo de rápido cargan nuestra páginas para un usuario, y eso a medida que las páginas incorporan más scripts


## Referencias
http://www.stevesouders.com/blog/2013/05/13/moving-beyond-window-onload/


 use the browser’s Navigation Timing API to collect data from most points in the page load process
 
 

window.onload is not the best metric for measuring website speed
