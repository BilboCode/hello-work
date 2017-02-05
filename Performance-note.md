# Analizar la experiencia de carga de nuestras páginas
> Por desgracia no existe una métrica `perception.ready()`que nos suministren los navegadores, por lo que tendremos que decidir en qué
única métrica nos fijamos para analizar la experiencia de un usuario respecto a los tiempos de carga de nuestras páginas.


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

La secuencia más sencilla de pasos hasta mostrar la página en pantalla es:

![CRP-noCSS-noJS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom.png)

1. el navegador solicita el documento html
2. una vez el navegador ha recibido el documento html, comienza a procesarlo para generar el árbol DOM que describe la estructura y el contenido de la página.
3. la construcción del DOM es un proceso que se ejecuta de modo gradual. El navegador va recibiendo el html en paquetes (TCP responses), y a medida que los recibe, los va parseando para ir generando el árbol DOM.
4. Una vez se ha completado el árbol DOM, ya se pueden ejecutar los procesos del render de la página: creación del rendering tree (se integra la información del DOM con la de estilos de los elementos del CSSOM (que en este caso no hay), el layout (calcular disposición y tamaño de los elementos en la pantalla), y el paint (covertirlo a pixels).

En el caso de que intervenga una hoja de estilos (imagines en fichero css externo), la secuencia de eventos es:

![CRP-noJS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css.png)

1. el navegador solicita el documento html
2. ha medida que recibe el documento, comienza la construcción del DOM
3. cuando detecta que se necesita un fichero css, lo solicita al servidor
4. recibido el fichero css, lo procesa para construir el CSSOM
5. a diferencia del flujo anterior, en esta ocasión el navegador deberá esperar a disponer del CSSOM para pasar a la etapa de renderizado de la página. 

Es lo que se denomina "render blocking CSS"

Por último, en el caso en que tb intervenga código javascript, la secuencia es más compleja:

![CRP](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css-js.png)

1. el navegador solicita el documento html
2. ha medida que recibe el documento, comienza la construcción del DOM
3. cuando detecta la necesidad de css y js, solicita estos recursos al servidor.
4. adicionalmente el navegador, al detectar código javascript detiene la construcción del DOM, que no se reanuda hasta que se haya ejecutado el script.
5. a su vez, la ejecución del script requiere de que se haya completado el CSSOM, por lo que queda bloqueado hasta que descarguen y procesen las hojas de estilo.

Este impacto en el critical rendering path se denomina "render blocking javascript"

Aunque están fuera del CTR, cuando durante el parseo del HTML para la construción del DOM, el navegador detecta que se necesitan recursos adicionales, como las imágenes, éste las solicita, si bien, los procesos del pintado de los contenidos pueden comenzar sin esperar a que lleguen al navegador.

#### Render blocking CSS
Como se ha visto en la descripción de la secuancia del CRP, sin que se hayan descargado todas las hojas de estilo y se haya generado el CSSOM, el navegador NO puede pintar contenido en pantalla.

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

Adicionalmente, en el caso del atributo `async` estamos indicando al navegador que la ejecución del script no tiene que esperar al CSSOM. De ese, en caso de que se descargara el javascript antes de que se hubiera completado el CSSOM éste podría ejecutarse. Por supuesto tb antes de que se completase el DOM.

Por su parte, el atributo `defer` provoca que el javascript, aunque esté descargado, no se ejecute hasta después de que el DOM se ha completado.

También en ambos casos hay que indicar que su ejecución no retrasa el evento domContentLoaded, que se producirá inmediatamente después de que se haya completado el DOM (aunque no se haya completado el CSSOM). Aunque el rendering de la página deberá esperar al CSSOM, dar un buen tiempo para el evento domContentLoaded es importante, porque representa el que el objeto DOM está disponible, lo que por ejemplo puede ser aprovechado para que se vinculen lo antes posible eventos javascript al documento.

Otras técnicas, que requieren cierta programación, persiguen diferir no solo la ejecución, sino la descarga del javascript haciendo que ésta no se produzca hasta que se ha completado la descarga completa de los elementos de la página, evento load de la página. Esta técnica podrá aplicarse solo a ciertos javascript, a aquellos que puedan ejecutarse de forma diferida, sin afectar a la usabilidad de la página, ejemplo, ciertas analíticas, botones de redes sociales, etc. 

#### Javascript bloqueado por el CSSOM
Como hemos visto, el código javascript requiere del CSSOM para ejecutarse. Por un lado será importante asegurar una descarga lo más rápido posible de las hojas de estilo (pocos ficheros y de tamaño razonable), por otro será importante el orden en el que en el documento están las llamadas a los ficheros css y ficheros javascript, siendo la práctica general recomendada que estén las llamadas a ficheros css por encima de las llamadas a ficheros javascript.


Es por ello, que resulta importante asegurar una descarga rápida de las hojas de estilo, así como la posición del código javascript 

Sin embargo, su ejecución deberá esperar a que el CSSOM esté construido.



que deberá ser ejecutado antes de que se produzca el render de la página.

Sin embargo, el javascript no puede ser ejecutado sin que antes se haya generado el CSSOM  


#### las imágenes y otros recursos (subframes: video, ....)

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
