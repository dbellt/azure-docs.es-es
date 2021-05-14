---
title: Procedimientos recomendados del SDK de Azure Maps Web | Microsoft Azure Maps
description: Aprenda consejos y trucos para optimizar el uso del SDK web de Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 3/22/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 31db8c4d8a371aaad939e3a85972958befb2206e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137570"
---
# <a name="azure-maps-web-sdk-best-practices"></a>Procedimientos recomendados del SDK de Azure Maps Web

Este documento se centra en los procedimientos recomendados para el SDK Web de Azure Maps, sin embargo, muchos de los procedimientos recomendados y las optimizaciones que se describen pueden aplicarse a todos los demás SDK de Azure Maps.

El SDK de Azure Maps web proporciona un lienzo eficaz para representar grandes conjuntos de datos espaciales de muchas maneras diferentes. En algunos casos, hay varias maneras de representar los datos de la misma manera, pero en función del tamaño del conjunto de datos y de la funcionalidad deseada, un método puede funcionar mejor que otros. En este artículo se resaltan los procedimientos recomendados y sugerencias y trucos para maximizar el rendimiento y crear una experiencia de usuario fluida.

Por lo general, al considerar mejorar el rendimiento del mapa, busque formas de reducir el número de capas y orígenes, así como la complejidad de los conjuntos de datos y los estilos de representación que se usan.

## <a name="security-basics"></a>Aspectos básicos de la seguridad

La parte más importante de la aplicación es su seguridad. Si su aplicación no es segura, un pirata informático puede estropear cualquier aplicación, con independencia de la calidad de la experiencia del usuario. A continuación se muestran algunas sugerencias para proteger la aplicación Azure Maps. Al usar Azure, asegúrese de familiarizarse con las herramientas de seguridad disponibles. Consulte este documento para obtener una [Introducción a la seguridad de Azure](../security/fundamentals/overview.md).

> [!IMPORTANT]
> Azure Maps proporciona dos métodos de autenticación.
> * Autenticación basada en claves de suscripción
> * La autenticación de Azure Active Directory usa Azure Active Directory en todas las aplicaciones de producción.
> La autenticación basada en claves de suscripción es sencilla y lo que la mayoría de las plataformas de asignación usan como método claro para medir el uso de la plataforma con fines de facturación. Sin embargo, esta no es una forma segura de autenticación y solo debe usarse localmente al desarrollar aplicaciones. Algunas plataformas proporcionan la capacidad de restringir las direcciones IP o el remitente HTTP que se encuentra en las solicitudes; sin embargo, esta información se puede suplantar fácilmente. Si usa claves de suscripción, asegúrese de [girarlas con regularidad](how-to-manage-authentication.md#manage-and-rotate-shared-keys).
> Azure Active Directory es un servicio de identidad empresarial que tiene una gran variedad de opciones de seguridad y configuración para todo tipo de escenarios de aplicaciones. Microsoft recomienda que todas las aplicaciones de producción que utilicen Azure Maps utilicen Azure Active Directory para la autenticación.
> Obtenga más información sobre la [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md) en este documento.

### <a name="secure-your-private-data"></a>Asegure sus datos privados

Cuando los datos se agregan a los SDK de mapas interactivos Azure Maps, se representan localmente en el dispositivo del usuario final y nunca se envían de vuelta a Internet por cualquier motivo.

Si la aplicación está cargando datos que no deben ser accesibles públicamente, asegúrese de que los datos se almacenan en una ubicación segura, se tiene acceso a ellos de forma segura y que la propia aplicación está bloqueada y solo está disponible para los usuarios deseados. Si se omite alguno de estos pasos, una persona no autorizada tiene la posibilidad de tener acceso a estos datos. Azure Active Directory puede ayudarle a bloquear esto.

Vea este tutorial en [Incorporación de la autenticación a una aplicación web en Azure App Service](../app-service/scenario-secure-app-authentication-app-service.md)

### <a name="use-the-latest-versions-of-azure-maps"></a>Usar las versiones más recientes de Azure Maps

Los SDK de Azure Maps pasan por las pruebas de seguridad periódicas junto con cualquier biblioteca de dependencias externa que puedan usar los SDK. Cualquier problema de seguridad conocido se corrige oportunamente y se lanza a producción. Si la aplicación apunta a la versión principal más reciente de la versión hospedada del SDK Web de Azure Maps, recibirá automáticamente todas las actualizaciones de versión secundaria que incluirán correcciones relacionadas con la seguridad.

Si hospeda automáticamente el SDK Web de Azure Maps a través del módulo NPM, asegúrese de usar el símbolo de intercalación (^) en junto con el número de versión del paquete de Azure Maps NPM en el archivo `package.json` para que siempre apunte a la última versión secundaria.

```json
"dependencies": {
  "azure-maps-control": "^2.0.30"
}
```

## <a name="optimize-initial-map-load"></a>Optimizar la carga inicial del mapa

Cuando se carga una página web, una de las primeras cosas que desea hacer es empezar a representar algo lo antes posible para que el usuario no se reproduzca en una pantalla en blanco.

### <a name="watch-the-maps-ready-event"></a>Ver el evento de asignación lista

Del mismo modo, cuando la asignación se carga inicialmente, a menudo se desea cargar datos en ella lo más rápido posible, por lo que el usuario no está examinando una asignación vacía. Dado que el mapa carga los recursos de forma asincrónica, tiene que esperar hasta que la asignación esté lista para interactuar antes de intentar representar sus propios datos. Hay dos eventos que puede esperar, un evento `load` y un evento `ready`. El evento de carga se activará después de que la asignación haya terminado de cargar completamente la vista de asignación inicial y se hayan cargado todos los mosaicos de asignación. El evento listo se activará cuando tenga los recursos de asignación que se necesitan como mínimo para empezar a interactuar con la asignación. El evento listo se puede activar a menudo a la mitad del tiempo del evento cargar y, por lo tanto, le permite empezar a cargar los datos en la asignación antes.

### <a name="lazy-load-the-azure-maps-web-sdk"></a>Carga lenta del SDK web de Azure Maps

Si el mapa no es necesario inmediatamente, cargue de forma lenta el SDK Web de Azure Maps hasta que sea necesario. Esto retrasará la carga de los archivos JavaScript y CSS utilizados por el SDK Web de Azure Maps hasta que sea necesario. Un escenario común en el que esto ocurre es cuando el mapa se carga en un panel de tabulación o de flotante que no se muestra en la carga de la página.
En el ejemplo de código siguiente se muestra cómo retrasar la carga del SDK web Azure Maps hasta que se presiona un botón.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Carga diferida de la asignación" src="https://codepen.io/azuremaps/embed/vYEeyOv?height=500&theme-id=default&default-tab=js,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Vea el lápiz de <a href='https://codepen.io/azuremaps/pen/vYEeyOv'>carga de KML en el mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="add-a-placeholder-for-the-map"></a>Agregar un marcador de posición para el mapa

Si el mapa tarda un tiempo en cargarse debido a limitaciones de red u otras prioridades dentro de la aplicación, considere la posibilidad de agregar una pequeña imagen de fondo a la asignación `div` como un marcador de posición para la asignación. Esto rellenará el vacío de la asignación `div` mientras se está cargando.

### <a name="set-initial-map-style-and-camera-options-on-initialization"></a>Establecer opciones de cámara y estilo de mapa iniciales en la inicialización

A menudo, las aplicaciones quieren cargar el mapa en una ubicación o un estilo específicos. A veces, los desarrolladores esperan hasta que se carga la asignación (o esperan el `ready` evento) y, a continuación, usan las funciones `setCemer` o `setStyle` del mapa. A menudo, esto tardará más tiempo en llegar a la vista de mapa inicial deseada, ya que muchos recursos acaban de cargarse de forma predeterminada antes de que se carguen los recursos necesarios para la vista de mapa deseada. Un mejor enfoque consiste en pasar las opciones de estilo y la cámara de mapa deseada en el mapa al inicializarla.

## <a name="optimize-data-sources"></a>Optimización del origen de datos

El SDK Web tiene dos orígenes de datos,

* **Origen de GeoJSON**: conocido como la `DataSource` clase, administra los datos de ubicación sin procesar en formato GeoJSON localmente. Adecuado para conjuntos de datos de tamaño pequeño a medio (con cientos de miles de características).
* **Origen de mosaico vectorial**: conocido como la clase `VectorTileSource`, carga los datos con formato de mosaico vectorial para la vista del mapa actual, en función del sistema de mosaico de mapas. Ideal para conjuntos de datos grandes o masivos (millones o miles de millones de características).

### <a name="use-tile-based-solutions-for-large-datasets"></a>Usar soluciones basadas en mosaicos para grandes conjuntos de valores

Si trabaja con conjuntos de datos más grandes que contienen millones de características, la manera recomendada de lograr un rendimiento óptimo es exponer los datos mediante una solución de servidor como vector o el servicio de mosaico de imagen de trama.  
Los mosaicos vectoriales están optimizados para cargar solo los datos que están en la vista con las geometrías recortadas en el área de enfoque del icono y generalizadas para que coincidan con la resolución del mapa para el nivel de zoom del icono.

La [plataforma Azure Maps Creator](creator-indoor-maps.md) proporciona la capacidad de recuperar datos en formato de mosaico vectorial. Otros formatos de datos pueden usar herramientas como [Tippecanoe](https://github.com/mapbox/tippecanoe) o una de la lista de muchos [recursos de esta página](https://github.com/mapbox/awesome-vector-tiles).

También es posible crear un servicio personalizado que represente los conjuntos de datos como mosaicos de imagen rasterizada en el lado servidor y cargar los datos mediante la clase TileLayer en el SDK de MAP. Esto proporciona un rendimiento excepcional, ya que el mapa solo necesita cargar y administrar unas docenas de imágenes como máximo. Sin embargo, hay algunas limitaciones en el uso de mosaicos de tramas, ya que los datos sin procesar no están disponibles localmente. A menudo es necesario un servicio secundario para potenciar cualquier tipo de experiencia de interacción, por ejemplo, averiguar en qué forma un usuario hizo clic. Además, el tamaño de archivo de un mosaico de tramas suele ser mayor que un icono de vector comprimido que contiene geometrías generalizadas y optimizadas de nivel de zoom.

Obtenga más información sobre los orígenes de datos en [crear un documento de origen de datos](create-data-source-web-sdk.md).

### <a name="combine-multiple-datasets-into-a-single-vector-tile-source"></a>Combinar varios conjuntos de valores en un único origen de mosaico de vector

Los orígenes de datos menos que el mapa tiene que administrar, cuanto más rápido puede procesar todas las características que se van a mostrar. En concreto, cuando se trata de orígenes de mosaicos, la combinación de dos orígenes de mosaicos vectoriales reduce el número de solicitudes HTTP para recuperar los mosaicos en la mitad y la cantidad total de datos sería ligeramente menor, ya que solo hay un encabezado de archivo.

La combinación de varios conjuntos de datos en un único origen de mosaico de vector se puede lograr mediante una herramienta como [Tippecanoe](https://github.com/mapbox/tippecanoe). Los conjuntos de datos se pueden combinar en una sola colección de características o se pueden separar en capas independientes dentro del mosaico de vector conocido como capas de origen. Al conectar un origen de mosaico vectorial a una capa de representación, debe especificar la capa de origen que contiene los datos que desea representar con la capa.

### <a name="reduce-the-number-of-canvas-refreshes-due-to-data-updates"></a>Reducir el número de actualizaciones de lienzo debido a las actualizaciones de datos

Hay varias maneras de agregar o actualizar los datos de una clase `DataSource`. A continuación se muestran los distintos métodos y algunas consideraciones para garantizar un buen rendimiento.

* La función de orígenes de datos `add` se puede usar para agregar una o más características a un origen de datos. Cada vez que se llama a esta función, se desencadena una actualización del lienzo del mapa. Si va a agregar muchas características, combínelos en una matriz o colección de características y pasándolos a esta función una vez, en lugar de recorrer en bucle un conjunto de datos y llamar a esta función para cada característica.
* La función de orígenes de datos `setShapes` se puede utilizar para sobrescribir todas las formas de un origen de datos. En el capó, combina los orígenes de datos `clear` y `add` las funciones y realiza una única actualización del lienzo de mapa en lugar de dos, que es mucho más rápida. Asegúrese de usarlo cuando desee actualizar todos los datos de un origen de datos.
* La función de orígenes de datos `importDataFromUrl` se puede usar para cargar un archivo GEOjson a través de una dirección URL en un origen de datos. Una vez que se han descargado los datos, se pasan a la función de orígenes de datos `add`. Si el archivo GeoJSON está hospedado en un dominio diferente, asegúrese de que el otro dominio admite solicitudes entre dominios (CORs). Si no tiene en cuenta la posibilidad de copiar los datos en un archivo local del dominio o de crear un servicio de proxy con CORs habilitado. Si el archivo es grande, considere la posibilidad de convertirlo en un origen de mosaico vectorial.
* Si las características se ajustan a la `Shape` clase, las `addProperty` `setCoordinates` funciones, y `setProperties` de la forma desencadenarán una actualización en el origen de datos y una actualización del lienzo del mapa. Todas las características devueltas por los orígenes de datos `getShapes` y `getShapeById` las funciones se ajustan automáticamente con la clase `Shape`. Si desea actualizar varias formas, es más rápido convertirlas en JSON mediante la función de orígenes de datos `toJson`, editando GeoJSON y, a continuación, pasando estos datos a la función de orígenes de datos `setShapes`.

### <a name="avoid-calling-the-data-sources-clear-function-unnecessarily"></a>Evitar llamar a la función de borrado de orígenes de datos innecesariamente

La llamada a la función Clear de la clase `DataSource` produce una actualización del lienzo de mapa. Si la función `clear` se llama varias veces en una fila, se puede producir un retraso mientras el mapa espera a que se produzca cada actualización.

Un escenario común en el que suele aparecer en las aplicaciones es cuando una aplicación borra el origen de datos, descarga nuevos datos, vuelve a borrar el origen de datos y, a continuación, agrega los nuevos datos al origen de datos. En función de la experiencia de usuario deseada, las siguientes alternativas serían mejores.

* Borre los datos antes de descargar los nuevos datos y, a continuación, pase los nuevos datos a los orígenes de datos a la función `add` o `setShapes`. Si este es el único conjunto de datos en el mapa, el mapa estará vacío mientras se descargan los datos nuevos.
* Descargue los nuevos datos y, a continuación, páselo a la función de orígenes de datos `setShapes`. Esto reemplazará todos los datos del mapa.

### <a name="remove-unused-features-and-properties"></a>Quitar las características y propiedades no utilizadas

Si el conjunto de elementos contiene características que no se van a usar en la aplicación, quítelas. Del mismo modo, quite cualquier propiedad de las características que no sean necesarias. Esto presenta varias ventajas:

* Reduce la cantidad de datos que se deben descargar.
* Reduce el número de características que es necesario recorrer en bucle al representar los datos.
* A veces puede ayudar a simplificar o quitar expresiones y filtros controlados por datos, lo que significa menos procesamiento necesario en el momento de la representación.

Cuando las características tienen muchas propiedades o contenido, es mucho más eficaz limitar lo que se agrega al origen de datos a solo los necesarios para la representación y tener un método o servicio independiente para recuperar la propiedad o el contenido adicional cuando sea necesario. Por ejemplo, si tiene un mapa sencillo que muestra las ubicaciones de un mapa cuando se hace clic en una serie de contenido detallado, se muestra. Si desea usar el estilo controlado por datos para personalizar cómo se representan las ubicaciones en el mapa, solo debe cargar las propiedades necesarias en el origen de datos. Si desea mostrar el contenido detallado, utilice el identificador de la característica para recuperar el contenido adicional por separado. Si el contenido se almacena en el lado servidor, se puede usar un servicio para recuperarlo de forma asincrónica, lo que reduciría drásticamente la cantidad de datos que es necesario descargar cuando se carga inicialmente la asignación.

Además, reducir el número de dígitos significativos en las coordenadas de las características también puede reducir significativamente el tamaño de los datos. No es raro que las coordenadas contengan 12 o más posiciones decimales. sin embargo, seis posiciones decimales tienen una precisión de aproximadamente 0,1 medidor, que suele ser más precisa que la ubicación que representa la coordenada (se recomiendan seis posiciones decimales al trabajar con datos de ubicación pequeños, como los diseños de creación interiores). Si tiene más de seis posiciones decimales, es probable que no se produzca ninguna diferencia en cómo se representan los datos y solo se requerirá que el usuario descargue más datos para obtener una ventaja adicional.

Esta es una lista de [herramientas útiles para trabajar con datos GeoJSON](https://github.com/tmcw/awesome-geojson).

### <a name="use-a-separate-data-source-for-rapidly-changing-data"></a>Usar un origen de datos independiente para cambiar rápidamente los datos

A veces, es necesario actualizar rápidamente los datos en el mapa para cosas como Mostrar actualizaciones dinámicas de datos de streaming o animar características. Cuando se actualiza un origen de datos, el motor de representación recorrerá y representará todas las características del origen de datos. Separar los datos estáticos de los datos que cambian rápidamente en distintos orígenes de datos puede reducir significativamente el número de características que se vuelven a representar en cada actualización del origen de datos y mejorar el rendimiento general.

Si usa mosaicos vectoriales con datos en directo, una manera fácil de admitir las actualizaciones es usar el encabezado de respuesta `expires`. De forma predeterminada, cualquier capa de mosaico vectorial o de mosaico de tramas volverá a cargar automáticamente los mosaicos cuando la fecha `expires`. Los mosaicos de flujo de tráfico e incidente en la asignación usan esta característica para garantizar que se muestren datos de tráfico nuevos en tiempo real en el mapa. Esta característica se puede deshabilitar si se establece la opción de servicio de asignaciones `refreshExpiredTiles` en `false`.

### <a name="adjust-the-buffer-and-tolerance-options-in-geojson-data-sources"></a>Ajuste del búfer y las opciones de tolerancia en orígenes de datos GeoJSON

La clase `DataSource` convierte los datos de ubicación sin procesar en mosaicos vectoriales locales para la representación en la marcha. Estos mosaicos de vectores locales recortan los datos sin procesar en los límites del área de mosaico con un poco de búfer para garantizar una representación fluida entre los mosaicos. Cuanto menor sea la opción `buffer`, menos datos superpuestos se almacenarán en los mosaicos de vectores locales y mejor rendimiento, sin embargo, mayor será el cambio de los artefactos de representación que se producen. Intente modificar esta opción para obtener la combinación correcta de rendimiento con los artefactos de representación mínimos.

La clase `DataSource` también tiene una opción `tolerance` que se utiliza con el algoritmo de simplificación de Douglas-Peucker cuando se reduce la resolución de las geometrías con fines de representación. Al aumentar este valor de tolerancia se reducirá la resolución de las geometrías y, a su vez, se mejorará el rendimiento. Retoque esta opción para obtener la combinación correcta de rendimiento y resolución de geometría del conjunto de datos.

### <a name="set-the-max-zoom-option-of-geojson-data-sources"></a>Establecer la opción de zoom máximo de orígenes de datos GeoJSON

La clase `DataSource` convierte los datos de ubicación sin procesar en mosaicos vectoriales locales para la representación en la marcha. De forma predeterminada, lo hará hasta el nivel de zoom 18, momento en el que, cuando se acerque más hacia abajo, realizará un muestreo de los datos de los mosaicos generados para el nivel de zoom 18. Esto funciona bien con la mayoría de los conjuntos de datos que necesitan tener una alta resolución cuando se amplía en estos niveles. Sin embargo, cuando se trabaja con conjuntos de datos que es más probable que se vean cuando se aleja más, por ejemplo, al ver polígonos de estado o provincia, si se establece la opción `minZoom` del origen de datos en un valor más pequeño como, `12` se reducirá la cantidad de cálculos, la generación de mosaicos local que se produce y la memoria que usa el origen de datos y el rendimiento.

### <a name="minimize-geojson-response"></a>Minimizar la respuesta GeoJSON

Al cargar datos de GeoJSON desde un servidor a través de un servicio o mediante la carga de un archivo plano, asegúrese de que los datos se han reducido para quitar los caracteres de espacio innecesario que hacen que el tamaño de la descarga sea mayor que el necesario.

### <a name="access-raw-geojson-using-a-url"></a>Acceso a GeoJSON sin procesar mediante una dirección URL

Es posible almacenar objetos GeoJSON en línea dentro de JavaScript; sin embargo, esto usará una gran cantidad de memoria, ya que las copias de la misma se almacenarán en la variable que creó para este objeto y en la instancia de origen de datos, que la administra en un trabajador web independiente. Exponga el archivo GeoJSON a la aplicación con una dirección URL en su lugar y el origen de datos cargará una única copia de datos directamente en el trabajo Web de orígenes de datos.  

## <a name="optimize-rendering-layers"></a>Optimizar las capas de representación

Azure Maps proporciona varios niveles diferentes para representar datos en un mapa. Hay muchas optimizaciones que puede aprovechar para adaptar estas capas a su escenario, lo que aumenta el rendimiento y la experiencia de usuario global.

### <a name="create-layers-once-and-reuse-them"></a>Crear capas una vez y reutilizarlas

El SDK Web de Azure Maps se decide como controlado por datos. Los datos entran en orígenes de datos, que después se conectan a capas de representación. Si desea cambiar los datos del mapa, actualice los datos en el origen de datos o cambie las opciones de estilo de una capa. Esto suele ser mucho más rápido que quitar y volver a crear capas cada vez que se produce un cambio.

### <a name="consider-bubble-layer-over-symbol-layer"></a>Considerar capa de burbuja sobre la capa de símbolos

La capa de burbuja representa los puntos como círculos en el mapa y puede tener fácilmente su radio y estilo de color usando una expresión controlada por datos. Puesto que el círculo es una forma simple de WebGL para dibujar, el motor de representación podrá representarlas mucho más rápido que una capa de símbolos, que tiene que cargar y representar una imagen. La diferencia de rendimiento de estas dos capas de representación es apreciable al representar decenas de miles de puntos.

### <a name="use-html-markers-and-popups-sparingly"></a>Usar marcadores HTML y ventanas emergentes con moderación

A diferencia de la mayoría de las capas del control web de Azure Maps que usan WebGL para la representación, los marcadores HTML y las ventanas emergentes usan elementos DOM tradicionales para la representación. Por tanto, mientras más marcadores HTML y ventanas emergentes se agreguen a una página, más elementos DOM habrá. Se puede degradar el rendimiento después de agregar cientos de marcadores HTML o ventanas emergentes. Para conjuntos de datos grandes, considere la posibilidad de agrupar los datos en clústeres o de usar una capa de símbolo o burbuja. En el caso de los elementos emergentes, una estrategia habitual es crear una única ventana emergente y volver a usarla actualizando su contenido y su posición, tal como se muestra en el ejemplo siguiente:

<br/>

<iframe height='500' scrolling='no' title='Reutilizar un elemento emergente con varias marcas' src='//codepen.io/azuremaps/embed/rQbjvK/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen sobre <a href='https://codepen.io/azuremaps/pen/rQbjvK/'>reutilización de elementos emergentes con varias marcas</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Dicho esto, si solo tiene unos cuantos puntos para representar en el mapa, es posible que se prefiera la simplicidad de los marcadores HTML. Además, los marcadores HTML pueden ser fácilmente arrastrables si es necesario.

### <a name="combine-layers"></a>Combinar capas

El mapa es capaz de representar cientos de capas; sin embargo, cuanto más capas hay, más tiempo se tarda en representar una escena. Una estrategia para reducir el número de capas es combinar las capas que tienen estilos similares o se pueden aplicar estilos mediante [estilos controlados por datos](data-driven-style-expressions-web-sdk.md).

Por ejemplo, considere un conjunto de datos en el que todas las características tienen una `isHealthy` propiedad que puede tener un valor de `true` o `false`. Si crea una capa de burbujas que represente burbujas de color diferentes basadas en esta propiedad, hay varias maneras de hacerlo, tal y como se muestra a continuación, con un rendimiento mínimo de mayor rendimiento.

* Divida los datos en dos orígenes de datos en función del valor `isHealthy` y adjunte una capa de burbuja con una opción de color codificado de forma rígida a cada origen de datos.
* Coloque todos los datos en un único origen de datos y cree dos capas de burbujas con una opción de color codificada de forma rígida y un filtro basado en la `isHealthy` propiedad. 
* Coloque todos los datos en un único origen de datos, cree una sola capa de burbujas con una expresión de estilo `case` para la opción de color basada en la propiedad `isHealthy`. Este es un ejemplo de código donde se muestra esto.

```javascript
var layer = new atlas.layer.BubbleLayer(source, null, {
    color: [
        'case',

        //Get the 'isHealthy' property from the feature.
        ['get', 'isHealthy'],

        //If true, make the color 'green'. 
        'green',

        //If false, make the color red.
        'red'
    ]
});
```

### <a name="create-smooth-symbol-layer-animations"></a>Crear animaciones de capas de símbolos suaves

Las capas de símbolos tienen habilitada la detección de colisiones de forma predeterminada. Esta detección de colisiones pretende asegurarse de que no se superpongan dos símbolos. Las opciones de icono y texto de una capa de símbolos tienen dos opciones,

* `allowOverlap`: especifica si el símbolo será visible si entra en conflicto con otros símbolos.
* `ignorePlacement`: especifica si los otros símbolos pueden entrar en conflicto con el símbolo.

Ambas opciones se establecen en `false` de forma predeterminada. Al animar un símbolo, los cálculos de detección de colisiones se ejecutarán en cada fotograma de la animación, lo que puede ralentizar la animación y hacer que parezca menos fluido. Para suavizar la animación, establezca estas opciones en `true`.

En el ejemplo de código siguiente se muestra una forma sencilla de animar una capa de símbolos.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Animación de capas de símbolos" src="https://codepen.io/azuremaps/embed/oNgGzRd?height=500&theme-id=default&default-tab=js,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el fragmento de código <a href='https://codepen.io/azuremaps/pen/oNgGzRd'>Animación de capa de símbolos</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="specify-zoom-level-range"></a>Especificar el intervalo de nivel de zoom

Si los datos cumplen uno de los siguientes criterios, asegúrese de especificar el nivel de zoom mínimo y máximo de la capa para que el motor de representación pueda omitirla cuando esté fuera del intervalo de nivel de zoom.

* Si los datos proceden de un origen de mosaicos vectoriales, a menudo las capas de origen para tipos de datos diferentes solo están disponibles a través de un intervalo de niveles de zoom.
* Si usa una capa de mosaico que no tiene mosaicos para todos los niveles de zoom de 0 a 24 y desea que solo se representen en los niveles que tiene mosaicos, y no intente rellenar los mosaicos que faltan con iconos de otros niveles de zoom.
* Si solo desea representar una capa en determinados niveles de zoom.
Todas las capas tienen opción `minZoom` y `maxZoom` en la que la capa se representará cuando se entre estos niveles de zoom en función de esta lógica ` maxZoom > zoom >= minZoom`.

**Ejemplo**

```javascript
//Only render this layer between zoom levels 1 and 9. 
var layer = new atlas.layer.BubbleLayer(dataSource, null, {
    minZoom: 1,
    maxZoom: 10
});
```

### <a name="specify-tile-layer-bounds-and-source-zoom-range"></a>Especificar los límites de la capa de mosaico y el intervalo de zoom de origen

De forma predeterminada, las capas de mosaico cargarán mosaicos en todo el globo. Sin embargo, si el servicio de mosaico solo tiene iconos para un área determinada, el mapa intentará cargar los mosaicos cuando se encuentre fuera de esta área. Cuando esto sucede, se realizará una solicitud para cada mosaico y se esperará una respuesta que pueda bloquear otras solicitudes realizadas por la asignación y, por lo tanto, ralentizar la representación de otras capas. Al especificar los límites de una capa de mosaico, el mapa solo solicitará mosaicos que estén dentro de ese cuadro de límite. Además, si la capa de mosaico solo está disponible entre determinados niveles de zoom, especifique el zoom de origen mínimo y máximo por la misma razón.

**Ejemplo**

```javascript
var tileLayer = new atlas.layer.TileLayer({
    tileUrl: 'myTileServer/{z}/{y}/{x}.png',
    bounds: [-101.065, 14.01, -80.538, 35.176],
    minSourceZoom: 1,
    maxSourceZoom: 10
});
```

### <a name="use-a-blank-map-style-when-base-map-not-visible"></a>Usar un estilo de mapa en blanco cuando el mapa base no esté visible

Si una capa se está superpuesta en el mapa que abarcará por completo el mapa base, considere la posibilidad de establecer el estilo de mapa en `blank` o `blank_accessible` para que no se represente la asignación base. Un escenario común para hacerlo es cuando la superposición de un mosaico de globo terráqueo completo en no tiene ninguna opacidad ni área transparente sobre el mapa base.

### <a name="smoothly-animate-image-or-tile-layers"></a>Animar las capas de imagen o de mosaico sin problemas

Si desea animar a través de una serie de capas de imagen o de mosaico en el mapa. A menudo, es más rápido crear una capa para cada capa de imagen o de mosaico y cambiar la opacidad que para actualizar el origen de una sola capa en cada fotograma de animación. Ocultar una capa estableciendo la opacidad en cero y mostrar una nueva capa estableciendo su opacidad en un valor mayor que cero es mucho más rápido que actualizar el origen de la capa. Como alternativa, se puede alternar la visibilidad de las capas, pero asegúrese de establecer la duración de transición de la capa en cero; de lo contrario, se animará la capa al mostrarla, lo que provocará un efecto de parpadeo, ya que la capa anterior se habría ocultado antes de que la capa nueva estuviera visible.

### <a name="tweak-symbol-layer-collision-detection-logic"></a>Ajustar la lógica de detección de colisiones de capas de símbolos

La capa de símbolos tiene dos opciones que existen para el icono y el texto denominado `allowOverlap` y `ignorePlacement`. Estas dos opciones especifican si el icono o el texto de un símbolo pueden superponerse o superponerse. Cuando se establecen en `false`, la capa de símbolos realizará cálculos al representar cada punto para ver si entra en conflicto con cualquier otro símbolo ya representado en la capa y, si lo hace, no representará el símbolo en conflicto. Esto es bueno a la vez que se reduce la aglomeración en el mapa y se reduce el número de objetos que se representan. Al establecer estas opciones en `false`, se omitirá esta lógica de detección de colisiones y todos los símbolos se representarán en el mapa. Retoque esta opción para obtener la mejor combinación de rendimiento y experiencia del usuario.

### <a name="cluster-large-point-data-sets"></a>Conjuntos de datos de punto grande del clúster

Cuando se trabaja con conjuntos grandes de puntos de datos, es posible que cuando se representan en determinados niveles de zoom, muchos de los puntos se superpongan y solo sean visibles parcialmente, si es que están en absoluto. La agrupación en clústeres es un proceso que consiste en agrupar puntos cercanos y representarlos como un único punto agrupado. A medida que el usuario amplía el mapa en, los clústeres se dividen en sus puntos individuales. Esto puede reducir de forma significativa la cantidad de datos que se deben representar, hacer que el mapa se sienta menos saturado y mejorar el rendimiento. La clase `DataSource` tiene opciones para la agrupación en clústeres de datos localmente. Además, muchas herramientas que generan mosaicos vectoriales también tienen opciones de agrupación en clústeres.

Además, aumente el tamaño del radio del clúster para mejorar el rendimiento. Cuanto mayor sea el radio del clúster, menos puntos en clúster habrá que realizar el seguimiento y la representación.
Más información en el [documento de datos de punto de agrupación en clústeres](clustering-point-data-web-sdk.md)

### <a name="use-weighted-clustered-heat-maps"></a>Usar mapas térmicos en clúster ponderados

La capa de mapa térmico puede representar decenas de miles de puntos de datos fácilmente. Para conjuntos de datos más grandes, considere la posibilidad de habilitar la agrupación en clústeres en el origen de datos y el uso de un radio de clúster pequeño y usar la propiedad `point_count` de clústeres como ponderación de la asignación de alto. Cuando el radio del clúster tiene solo unos pocos píxeles de tamaño, habrá poca diferencia visual en el mapa térmico representado. El uso de un radio de clúster mayor mejorará el rendimiento, pero puede reducir la resolución del mapa térmico representado.

```javascript
var layer = new atlas.layer.HeatMapLayer(source, null, {
   weight: ['get', 'point_count']
});
```

Obtenga más información en la [agrupación en clústeres y las asignaciones térmicas en este documento](clustering-point-data-web-sdk.md #clustering-and-the-heat-maps-layer)

### <a name="keep-image-resources-small"></a>Mantener los recursos de imagen pequeños

Se pueden agregar imágenes al sprite de imagen de los mapas para representar iconos en una capa de símbolos o patrones en una capa de polígonos. Mantenga estas imágenes pequeñas para minimizar la cantidad de datos que se deben descargar y la cantidad de espacio que ocupan en el sprite de imagen de mapas. Cuando use una capa de símbolos que escale el icono mediante la opción `size`, use una imagen que tenga el tamaño máximo que debe mostrar el plan en el mapa y que no sea más grande. Esto garantizará que el icono se represente con alta resolución, al tiempo que se minimizan los recursos que utiliza. Además, también se puede usar SVG como un formato de archivo más pequeño para imágenes de iconos simples.

## <a name="optimize-expressions"></a>Optimizar expresiones

Las [expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md) proporcionan una gran flexibilidad y eficacia para filtrar y aplicar estilos a los datos del mapa. Hay muchas maneras en las que se pueden optimizar expresiones. Aquí encontrará algunas sugerencias.

### <a name="reduce-the-complexity-of-filters"></a>Reduzca la complejidad del método

Filtra todos los datos de un origen de datos y comprueba si cada filtro coincide con la lógica del filtro. Si los filtros son complejos, esto puede causar problemas de rendimiento. A continuación se indican algunas estrategias posibles para solucionarlo.

* Si usa mosaicos vectoriales, divida los datos en diferentes capas de origen.
* Si usa la clase `DataSource`, divida los datos en orígenes de datos independientes. Intente equilibrar el número de orígenes de datos con la complejidad del filtro. Un número excesivo de orígenes de datos puede causar problemas de rendimiento, por lo que es posible que tenga que realizar algunas pruebas para averiguar qué funciona mejor para su escenario.
* Al usar un filtro complejo en una capa, considere la posibilidad de usar varias capas con expresiones de estilo para reducir la complejidad del filtro. Evite crear un grupo de capas con estilos codificados cuando se pueden utilizar expresiones de estilo, ya que un gran número de capas también puede causar problemas de rendimiento.

### <a name="make-sure-expressions-dont-produce-errors"></a>Asegúrese de que las expresiones no producen errores

Las expresiones se suelen usar para generar código para realizar cálculos o operaciones lógicas en el momento de la representación. Al igual que el código en el resto de la aplicación, asegúrese de que los cálculos y el sentido lógico tienen sentido y no son propensos a errores. Los errores en las expresiones provocarán problemas al evaluar la expresión, lo que puede provocar problemas de representación y rendimiento reducidos.

Un error común que se debe tener en cuenta es tener una expresión que se basa en una propiedad de característica que podría no existir en todas las características. Por ejemplo, el código siguiente usa una expresión para establecer la propiedad color de una capa de burbuja en la propiedad `myColor` de una característica.

```javascript
var layer = new atlas.layer.BubbleLayer(source, null, {
    color: ['get', 'myColor']
});
```

El código anterior funcionará correctamente si todas las características del origen de datos tienen una propiedad `myColor` y el valor de esa propiedad es un color. Esto puede no ser un problema si tiene un control completo de los datos en el origen de datos y sabe que todas las características tendrán un color válido en una propiedad `myColor`. Dicho esto, para que este código sea seguro frente a errores, `case` se puede usar una expresión con la `has` expresión para comprobar que la característica tiene la propiedad `myColor`. Si es así, `to-color` se puede usar la expresión de tipo para intentar convertir el valor de esa propiedad en un color. Si el color no es válido, se puede usar un color de reserva. En el código siguiente se muestra cómo hacerlo y se establece el color de reserva en verde.

```javascript
var layer = new atlas.layer.BubbleLayer(source, null, {
    color: [
        'case',

        //Check to see if the feature has a 'myColor' property.
        ['has', 'myColor'],

        //If true, try validating that 'myColor' value is a color, or fallback to 'green'. 
        ['to-color', ['get', 'myColor'], 'green'],

        //If false, return a fallback value.
        'green'
    ]
});
```

### <a name="order-boolean-expressions-from-most-specific-to-least-specific"></a>Ordenar las expresiones booleanas de más a menos específicas

Cuando utilice expresiones booleanas que contengan múltiples pruebas condicionales, ordene las pruebas condicionales de la más específica a la menos específica. Al hacerlo, la primera condición debe reducir la cantidad de datos con los que se debe probar la segunda condición, lo que reduce el número total de pruebas condicionales que se deben realizar.

### <a name="simplify-expressions"></a>Simplifica expresiones

Las expresiones pueden ser eficaces y a veces complejas. Cuanto más sencillo sea una expresión, más rápida se evaluará. Por ejemplo, si se necesita una comparación simple, una expresión como `['==', ['get', 'category'], 'restaurant']` sería mejor que usar una expresión de coincidencia como `['match', ['get', 'category'], 'restaurant', true, false]`. En este caso, si la propiedad que se está comprobando es un valor booleano, una `get` expresión sería incluso más sencilla `['get','isRestaurant']`.

## <a name="web-sdk-troubleshooting"></a>Solución de problemas del SDK web

A continuación se muestran algunas sugerencias para depurar algunos de los problemas comunes que se producen al desarrollar con el SDK de Azure Maps Web.

**¿Por qué no se muestra el mapa al cargar el control Web?**

Haga lo siguiente:

* Asegúrese de que ha agregado las opciones de autenticación agregadas a la asignación. Si no se agrega, la asignación se cargará con un lienzo en blanco, ya que no puede acceder a los datos del mapa base sin autenticación y los errores 401 aparecerán en la pestaña red de las herramientas de desarrollo del explorador.
* Asegúrese de que dispone de una conexión a Internet.
* Compruebe si hay errores de las herramientas de desarrollo del explorador en la consola. Algunos errores pueden provocar que la asignación no se represente. Depure la aplicación.
* Asegúrese de que usa un [explorador compatible](supported-browsers.md).

**Todos los datos se muestran en el otro lado del mundo, ¿qué está ocurriendo?**
Las coordenadas, también denominadas posiciones, en los SDK de Azure Maps se alinean con el formato estándar del sector geoespacial de `[longitude, latitude]`. El mismo formato también es cómo se definen las coordenadas en el esquema GeoJSON; los datos principales con formato usados dentro de los SDK de Azure Maps. Si los datos aparecen en el lado opuesto del mundo, lo más probable es que se deba a que los valores de longitud y latitud se invierten en la información de coordenadas o posiciones.

**¿Por qué aparecen marcadores HTML en el lugar equivocado del control Web?**

Cosas que puede comprobar:

* Si usa el contenido personalizado para el marcador, asegúrese de que las opciones `anchor` y `pixelOffset` son correctas. De forma predeterminada, el centro inferior del contenido se alinea con la posición en el mapa.
* Asegúrese de que se ha cargado el archivo CSS para Azure Maps.
* Inspeccione el elemento DOM del marcador HTML para ver si cualquier CSS de la aplicación se ha anexado al marcador y está afectando a su posición.

**¿Por qué los iconos o el texto de la capa de símbolos aparecen en el lugar equivocado?**
Compruebe que las opciones `anchor` y `offset` están configuradas correctamente para alinearse con la parte de la imagen o el texto que desea que se alinee con la coordenada del mapa.
Si el símbolo solo está fuera del sitio cuando se gira el mapa, active la opción `rotationAlignment`. De forma predeterminada, los símbolos girarán con la ventanilla de mapas para que aparezcan verticalmente para el usuario. Sin embargo, en función del escenario, puede ser conveniente bloquear el símbolo a la orientación del mapa. Establezca la opción `rotationAlignment` en `’map’` para hacerlo.
Si el símbolo solo está fuera de lugar cuando el mapa está inclinado, marque la opción `pitchAlignment`. De forma predeterminada, los símbolos se mantendrán en posición vertical con la ventana de visualización de los mapas, ya que el mapa se inclina. Sin embargo, en función del escenario, puede ser conveniente bloquear el símbolo en el paso del mapa. Establezca la opción `pitchAlignment` en `’map’` para hacerlo.

**¿Por qué no aparece ninguno de mis datos en el mapa?**

Cosas que puede comprobar:

* Compruebe la consola para ver si hay errores en las herramientas de desarrollo del explorador.
* Asegúrese de que se ha creado un origen de datos y que se ha agregado a la asignación, y que el origen de datos se ha conectado a una capa de representación que también se ha agregado a la asignación.
* Agregue puntos de interrupción en el código y recorra el paso para asegurarse de que los datos se agregan al origen de datos y que el origen de datos y las capas se agregan a la asignación sin que se produzcan errores.
* Intente quitar las expresiones controladas por datos de la capa de representación. Es posible que una de ellas tenga un error en el que está causando el problema.

**¿Puedo usar el SDK Web de Azure Maps en un iframe de espacio aislado?**

Sí. Tenga en cuenta que [Safari tiene un error](https://bugs.webkit.org/show_bug.cgi?id=170075) que impide que los iframes en espacio aislado ejecuten los trabajos Web, que es el requisito del SDK web de Azure Maps. La solución consiste en agregar la etiqueta `"allow-same-origin"` a la propiedad Sandbox del iframe.

## <a name="get-support"></a>Obtención de soporte técnico

A continuación se muestran las distintas formas de obtener soporte técnico para Azure Maps en función del problema.

**Cómo notificar un problema de datos o un problema con una dirección?**

Azure Maps tiene una herramienta de comentarios de datos donde se pueden informar y realizar un seguimiento de los problemas de datos. [https://feedback.azuremaps.com/](https://feedback.azuremaps.com/) Cada problema enviado genera una dirección URL única que puede usar para realizar el seguimiento del progreso del problema de los datos. El tiempo que se tarda en resolver un problema de datos varía en función del tipo de problema y de lo fácil que es comprobar que el cambio es correcto. Una vez corregido, el servicio de representación verá la actualización en la actualización semanal, mientras que otros servicios como la geocodificación y el enrutamiento verán la actualización en la actualización mensual. En este [documento](how-to-use-feedback-tool.md)se proporcionan instrucciones detalladas sobre cómo notificar un problema de datos.

**Cómo notificar un error en un servicio o API?**

https://azure.com/support

**¿Dónde obtengo ayuda técnica para Azure Maps?**

Si está relacionado con el Azure Maps visual en Power BI: https://powerbi.microsoft.com/support/ para todos los demás servicios de Azure Maps: https://azure.com/support o los foros para desarrolladores: https://docs.microsoft.com/answers/topics/azure-maps.html

**¿Cómo se puede realizar una solicitud de características?**

Realice una solicitud de características en nuestro sitio de voz de usuario: https://feedback.azure.com/forums/909172-azure-maps

## <a name="next-steps"></a>Pasos siguientes

Consulte los artículos siguientes para obtener más sugerencias sobre cómo mejorar la experiencia del usuario en su aplicación.

> [!div class="nextstepaction"]
> [Hacer que la aplicación sea accesible](map-accessibility.md)

Obtenga más información sobre la terminología usada por Azure Maps y el sector geoespacial.

> [!div class="nextstepaction"]
> [Glosario de Azure Maps](glossary.md)