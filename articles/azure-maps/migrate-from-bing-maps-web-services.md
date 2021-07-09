---
title: 'Tutorial: Migración de servicios web desde Mapas de Bing | Microsoft Azure Maps'
description: Tutorial sobre cómo migrar servicios web desde Mapas de Bing a Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 04/26/2021
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 07fbe0cff104c25eca6db2750c2db692429ada65
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786521"
---
# <a name="tutorial-migrate-web-service-from-bing-maps"></a>Tutorial: Migración de servicios web desde Mapas de Bing

Tanto Azure Maps como Mapas de Bing proporcionan acceso a API espaciales mediante servicios web REST. Las interfaces de API de estas plataformas realizan funcionalidades similares, pero emplean convenciones de nomenclatura y objetos de respuesta diferentes. En este tutorial, aprenderá a:

> * Geocodificación directa e inversa
> * Búsqueda de puntos de interés
> * Cálculo de rutas y direcciones
> * Recuperación de una imagen de mapa
> * Cálculo de una matriz de distancia
> * Obtención de detalles de la zona horaria

En la siguiente tabla se indican las API de servicio de Azure Maps que proporcionan una funcionalidad similar a la de las API de servicio de Mapas de Bing enumeradas.

| API de servicio de Mapas de Bing                 | API de servicio de Azure Maps      |
|---------------------------------------|-----------------------------|
| Autosuggest                           | [Búsqueda](/rest/api/maps/search)     |
| Direcciones (incluido el camión)          | [Direcciones de ruta](/rest/api/maps/route/getroutedirections)                          |
| Matriz de distancia                       | [Matriz de ruta](/rest/api/maps/route/postroutematrixpreview)                          |
| Imágenes: mapa estático                  | [Representar](/rest/api/maps/render/getmapimage)                                   |
| Isócronas                            | [Rango de rutas](/rest/api/maps/route/getrouterange)                                    |
| Local Insights                        | [Búsqueda](/rest/api/maps/search) + [Rango de rutas](/rest/api/maps/route/getrouterange)    |
| Búsqueda local                          | [Búsqueda](/rest/api/maps/search)     |
| Reconocimiento de ubicación (POI)           | [Búsqueda](/rest/api/maps/search)     |
| Ubicaciones (codificación geográfica directa/inversa) | [Búsqueda](/rest/api/maps/search)                                               |
| Ajustar a la carretera                          | [Direcciones de ruta POST](/rest/api/maps/route/postroutedirections)                         |
| Servicios de datos espaciales (SDS)           | [Search](/rest/api/maps/search) + [Route](/rest/api/maps/route) + otros servicios de Azure |
| Zona horaria                             | [Zona horaria](/rest/api/maps/timezone)  |
| Incidentes de tráfico                     | [Detalles del incidente de tráfico](/rest/api/maps/traffic/gettrafficincidentdetail)                     |
| Elevation                             | [Elevación](/rest/api/maps/elevation)

Las siguientes API de servicio no están disponibles actualmente en Azure Maps:

-   Rutas de itinerario optimizadas: planificadas Route API de Azure Maps admite la optimización para vendedores ambulantes en un solo vehículo.
-   Metadatos de imágenes: se usa principalmente para obtener direcciones URL de mosaicos en Mapas de Bing. Azure Maps dispone de un servicio independiente para el acceso directo a los mosaicos de mapa.

Azure Maps tiene varios servicios web REST adicionales que pueden ser de interés:

-   [Azure Maps Creator](./creator-indoor-maps.md): cree un gemelo digital privado personalizado de edificios y espacios.
-   [Operaciones espaciales](/rest/api/maps/spatial): descargue cálculos y operaciones espaciales complejos, como geovallas, en un servicio.
-   [Mosaicos de mapa](/rest/api/maps/render/getmaptile): acceda a mosaicos de carreteras e imágenes desde Azure Maps en forma de mosaicos de trama y vectoriales.
-   [Rutas por lotes](/rest/api/maps/route/postroutedirectionsbatchpreview): permite realizar hasta 1000 solicitudes de ruta en un único lote durante un período. Las rutas se calculan en paralelo en el servidor para acelerar el procesamiento.
-   [Flujo de tráfico](/rest/api/maps/traffic): acceda a los datos de flujo de tráfico en tiempo real en forma de mosaicos de trama y vectoriales.
-   [Geolocation API (versión preliminar)](/rest/api/maps/geolocation/getiptolocationpreview): obtenga la ubicación de una dirección IP.
-   [Servicios meteorológicos](/rest/api/maps/weather): obtenga acceso a datos meteorológicos de previsión y en tiempo real.

Asegúrese también de revisar las siguientes guías de procedimientos recomendados:

-   [Procedimientos recomendados de búsqueda](./how-to-use-best-practices-for-search.md)
-   [Procedimientos recomendados de cálculo de ruta](./how-to-use-best-practices-for-routing.md)

## <a name="prerequisites"></a>Requisitos previos

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
2. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
3. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

## <a name="geocoding-addresses"></a>Geocodificación de direcciones

La geocodificación es el proceso de convertir una dirección (por ejemplo, `"1 Microsoft way, Redmond, WA"`) en una coordenada (como longitud: -122.1298, latitud: 47,64005). Las coordenadas se suelen usar para centrar un mapa o colocar en él un marcador.

Azure Maps proporciona varios métodos de geocodificación de direcciones:

-   [Geocodificación de direcciones en formato libre](/rest/api/maps/search/getsearchaddress): especifique una única cadena de dirección (como `"1 Microsoft way, Redmond, WA"`) y procese la solicitud inmediatamente. Este es el servicio recomendado si hay que geocodificar direcciones de forma individual rápidamente.
-   [Geocodificación de direcciones estructurada](/rest/api/maps/search/getsearchaddressstructured): especifique las partes de una dirección (como el nombre de la calle, la ciudad, el país y el código postal) y procese la solicitud inmediatamente. Este es el servicio recomendado si hay que geocodificar direcciones de forma individual rápidamente y los datos ya se han separado en sus partes individuales.
-   [Geocodificación de direcciones por lotes](/rest/api/maps/search/postsearchaddressbatchpreview): cree una solicitud que contenga hasta 10 000 direcciones y procéselas durante un período de tiempo determinado. Todas las direcciones se geocodificarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo. Este es el servicio recomendable para geocodificar grandes conjuntos de datos.
-   [Búsqueda aproximada](/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. La API toma una cadena de forma libre (que puede ser una dirección, un lugar, un punto de referencia, un punto de interés o una categoría de punto de interés) y procesa la solicitud inmediatamente. Se recomienda usar esta API en situaciones en las que los usuarios pueden buscar direcciones o puntos de interés desde el mismo cuadro de texto.
-   [Búsqueda parcial por lotes](/rest/api/maps/search/postsearchfuzzybatchpreview): cree una solicitud que contenga hasta 10 000 direcciones, lugares, puntos de referencia o puntos de interés y procéselos durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.

En las siguientes tablas se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps para la geocodificación de direcciones estructuradas y en formato libre.

**Ubicación por dirección (dirección estructurada)**

| Parámetro de API de Mapas de Bing              | Parámetro de API de Azure Maps similar                 |
|--------------------------------------|-----------------------------------------------------|
| `addressLine`                      | `streetNumber`, `streetName` o `crossStreet` |
| `adminDistrict`                    | `countrySubdivision`                              |
| `countryRegion`                    | `country` y `countryCode`                     |
| `locality`                         | `municipality` o `municipalitySubdivision`     |
| `postalCode`                       | `postalCode`                                      |
| `maxResults` (`maxRes`)          | `limit`                                           |
| `includeNeighborhood` (`inclnb`) | N/D: Azure Maps siempre lo devuelve si está disponible.   |
| `include` (`incl`)               | N/D: código de país ISO2 que siempre devuelve Azure Maps. |
| `key`                              | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)                  | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md)) |
| `userRegion` (`ur`)              | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

Azure Maps también admite:

* `countrySecondarySubdivision`: municipios, distritos
* `countryTertiarySubdivision`: áreas designadas, barrios, cantones, comunidades
* `ofs`: desplácese por los resultados en combinación con el parámetro `maxResults`.

**Ubicación por consulta (cadena de dirección en formato libre)**

| Parámetro de API de Mapas de Bing              | Parámetro de API de Azure Maps similar      |
|--------------------------------------|------------------------------------------|
| `query`                            | `query`                                |
| `maxResults` (`maxRes`)          | `limit`                                |
| `includeNeighborhood` (`inclnb`) | N/D: Azure Maps siempre lo devuelve si está disponible.  |
| `include` (`incl`)               | N/D: código de país ISO2 que siempre devuelve Azure Maps.  |
| `key`                              | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)                  | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))  |
| `userRegion` (`ur`)              | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

Azure Maps también admite:

* `typeahead`: especifica si la consulta se interpretará como una entrada parcial y la búsqueda pasará al modo predictivo (sugerencias automáticas o autocompletar).
* `countrySet`: una lista separada por comas de códigos de países ISO2 a los que se va a limitar la búsqueda.
* `lat`/`lon`, `topLeft`/`btmRight`, `radius`: especifique la ubicación y el área de usuario para que los resultados sean más pertinentes localmente.
* `ofs`: desplácese por los resultados en combinación con el parámetro `maxResults`.

[Aquí](./how-to-search-for-address.md) se documenta un ejemplo sobre cómo usar el servicio de búsqueda. Asegúrese de revisar la documentación de [procedimientos recomendados de búsqueda](./how-to-use-best-practices-for-search.md).

## <a name="reverse-geocode-a-coordinate-find-a-location-by-point"></a>Inversión del código geográfico en una coordenada (búsqueda de una ubicación por punto)

La geocodificación inversa es el proceso por el cual unas coordenadas geográficas (como, por ejemplo, una longitud de -122,1298 y una latitud de 47,64005) en su dirección aproximada (como `"1 Microsoft way, Redmond, WA"`).

Azure Maps proporciona diversos métodos de geocodificación inversa:

-   [Geocodificador inverso de direcciones](/rest/api/maps/search/getsearchaddressreverse): especifique una única coordenada geográfica para obtener la dirección aproximada correspondiente y procese la solicitud inmediatamente.
-   [Geocodificador inverso de cruces de direcciones](/rest/api/maps/search/getsearchaddressreversecrossstreet): especifique una única coordenada geográfica para obtener información sobre un cruce de direcciones (por ejemplo, "Primera" con "Central") y procese la solicitud inmediatamente.
-   [Geocodificador inverso de direcciones por lotes](/rest/api/maps/search/postsearchaddressreversebatchpreview): cree una solicitud que contenga hasta 10 000 coordenadas y procéselas durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing              | Parámetro de API de Azure Maps similar       |
|--------------------------------------|-------------------------------------------|
| `point`                              | `query`                                   |
| `includeEntityTypes`                 | `entityType`: consulte a continuación la tabla de comparación de tipos de entidad.    |
| `includeNeighborhood` (`inclnb`)     | N/D: Azure Maps siempre lo devuelve si está disponible.         |
| `include` (`incl`)                   | N/D: código de país ISO2 que siempre devuelve Azure Maps.    |
| `key`                                | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)                      | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))   |
| `userRegion` (`ur`)                  | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

Asegúrese de revisar la documentación de [procedimientos recomendados de búsqueda](./how-to-use-best-practices-for-search.md).

Reverse Geocoding API de Azure Maps tiene algunas características adicionales que no están disponibles en Mapas de Bing que podrían ser útiles de cara a la integración al migrar la aplicación:

* Obtención de datos de límite de velocidad
* Recupere información de uso de carreteras: comarcales, arterias, accesos limitados, rampas, etc.
* Lado de la calle en el que está la coordenada

**Tabla de comparación de tipos de entidad**

En la tabla siguiente se contrastan los valores de tipo de entidad de Mapas de Bing con los nombres de propiedad equivalentes de Azure Maps.

| Tipo de entidad de Mapas de Bing | Tipo de entidad de Azure Maps comparable               | Descripción                                |
|-----------------------|-------------------------------------------------|--------------------------------------------|
| `Address`             |                                                 | *Dirección*                                  |
| `Neighborhood`        | `Neighbourhood`                                 | *Vecindario*                             |
| `PopulatedPlace`      | `Municipality` o `MunicipalitySubdivision`     | *Ciudad*, *población o periferia* o *megalópolis*     |
| `Postcode1`           | `PostalCodeArea`                                | *Código postal*                |
| `AdminDivision1`      | `CountrySubdivision`                            | *Estado* o *provincia*                      |
| `AdminDivision2`      | `CountrySecondarySubdivison`                    | *Municipio* o *distritos*                    |
| `CountryRegion`       | `Country`                                       | *Nombre del país*                             |
|                       | `CountryTertiarySubdivision`                    | *Barrios*, *cantones*, *comunidades*          |

## <a name="get-location-suggestions-autosuggest"></a>Obtención de sugerencias de ubicación (sugerencia automática)

Varias de las API de búsqueda de Azure Maps admiten el modo predictivo, que se puede usar para escenarios de sugerencia automática. [Fuzzy Search API](/rest/api/maps/search/getsearchfuzzy) de Azure Maps es muy parecida a Autosuggest API de Mapas de Bing. Las siguientes API también admiten el modo predictivo; agregue `&typeahead=true` a la consulta:

-   [Geocodificación de direcciones en formato libre](/rest/api/maps/search/getsearchaddress): especifique una única cadena de dirección (como `"1 Microsoft way, Redmond, WA"`) y procese la solicitud inmediatamente. Este es el servicio recomendado si hay que geocodificar direcciones de forma individual rápidamente.
-   [Búsqueda aproximada](/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. La API toma una cadena de forma libre (que puede ser una dirección, un lugar, un punto de referencia, un punto de interés o una categoría de punto de interés) y procesa la solicitud inmediatamente. Se recomienda usar esta API en situaciones en las que los usuarios pueden buscar direcciones o puntos de interés desde el mismo cuadro de texto.
-   [Búsqueda de puntos de interés](/rest/api/maps/search/getsearchpoi): se buscan puntos de interés por su nombre Por ejemplo, `"starbucks"`.
-   [Búsqueda de categorías de puntos de interés](/rest/api/maps/search/getsearchpoicategory): se buscan puntos de interés por su categoría Por ejemplo, "restaurante".

## <a name="calculate-routes-and-directions"></a>Cálculo de rutas y direcciones

Azure Maps se puede usar para calcular rutas y direcciones. Azure Maps tiene muchas funcionalidades similares a las del servicio de enrutamiento de Mapas de Bing, como, por ejemplo:

* Horas de salida y de llegada.
* Rutas de tráfico predictivas y en tiempo real.
* Distintos modos de desplazamiento (coche, a pie, camión)
* Optimización del orden de los puntos de referencia (vendedores ambulantes)

> [!NOTE]
> Azure Maps requiere que todos los puntos de referencia sean coordenadas y que las direcciones estén geocodificadas.

El servicio de enrutamiento de Azure Maps proporciona las siguientes API para el cálculo de las rutas:

-   [Calcular ruta](/rest/api/maps/route/getroutedirections): calcula una ruta y la solicitud se procesa inmediatamente. Esta API admite solicitudes tanto GET como POST. Se recomienda usar solicitudes POST cuando se especifique un gran número de puntos de referencia o al usar muchas opciones de ruta para asegurarse de que la solicitud de dirección URL no se eternice y cause problemas.
-   [Ruta por lotes](/rest/api/maps/route/postroutedirectionsbatchpreview): cree una solicitud que contenga hasta 1000 solicitudes de ruta y procéselas durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.
-   [Servicio Mobility (versión preliminar)](/rest/api/maps/mobility): calcula rutas y direcciones según el transporte público.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing                                    | Parámetro de API de Azure Maps similar               |
|------------------------------------------------------------|---------------------------------------------------|
| `avoid`                                                    | `avoid`                                           |
| `dateTime` (`dt`)                                          | `departAt` o `arriveAt`                          |
| `distanceBeforeFirstTurn` (`dbft`)                         | N/D                                               |
| `distanceUnit` (`du`)                                      | No procede (Azure Maps usa exclusivamente el sistema métrico)     |
| `heading` (`hd`)                                           | `vehicleHeading`                                  |
| `maxSolutions` (`maxSolns`)                                | `maxAlternatives`, `alternativeType`, `minDeviationDistance` y `minDeviationTime`.  |
| `optimize` (`optwz`)                                       | `routeType` y `traffic`                         |
| `optimizeWaypoints` (`optWp`)                              | `computeBestOrder`                                |
| `routeAttributes` (`ra`)                                   | `instructionsType`                                |
| `routePathOutput` (`rpo`)                                  | `routeRepresentation`                             |
| `timeType` (`tt`)                                          | `departAt` o `arriveAt`                          |
| `tolerances` (`tl`)                                        | N/D                                               |
| `travelMode`                                               | `travelMode`                                      |
| `waypoint.n` (`wp.n`) o `viaWaypoint.n` (`vwp.n`)         | `query` (se coordina con el formato `lat0,lon0:lat1,lon1….`)   |
| `key`                                                      | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)                                            | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md)) |
| `userRegion` (`ur`)                                        | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

La API de enrutamiento de Azure Maps también admite enrutamiento de camiones dentro de la misma API. En la siguiente tabla se contrastan los parámetros de enrutamiento de camiones de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing                  | Parámetro de API de Azure Maps similar        |
|------------------------------------------|--------------------------------------------|
| `dimensionUnit` (`dims`)                 | N/D: solo se admiten dimensiones en metros. |
| `weightUnit` (`wu`)                      | N/D: solo se admiten pesos en kilogramos. |
| `vehicleHeight` (`height`)               | `vehicleHeight`                            |
| `vehicleWidth` (`width`)                 | `vehicleWidth`                             |
| `vehicleLength` (`vl`)                   | `vehicleLength`                            |
| `vehicleWeight` (`weight`)               | `vehicleWeight`                            |
| `vehicleAxles` (`axles`)                 | `vehicleAxelWeight`                        |
| `vehicleTrailers` (`vt`)                 | **N/D**                                    |
| `vehicleSemi` (`semi`)                   | `vehicleCommercial`                        |
| `vehicleMaxGradient` (`vmg`)             | **N/D**                                    |
| `vehicleMinTurnRadius` (`vmtr`)          | **N/D**                                    |
| `vehicleAvoidCrossWind` (`vacw`)         | **N/D**                                    |
| `vehicleAvoidGroundingRisk` (`vagr`)     | **N/D**                                    |
| `vehicleHazardousMaterials` (`vhm`)      | `vehicleLoadType`                          |
| `vehicleHazardousPermits` (`vhp`)        | `vehicleLoadType`                          |

> [!TIP]
> La API de ruta de Azure Maps solo devuelve de forma predeterminada un resumen (distancia y lapsos de tiempo) y las coordenadas del trayecto de ruta. Use el parámetro `instructionsType` para obtener instrucciones paso a paso. El parámetro `routeRepresentation` se puede usar para descartar el resumen y el trayecto de ruta.

Asegúrese de revisar también la documentación sobre [procedimientos recomendados de enrutamiento](./how-to-use-best-practices-for-routing.md).

La API de enrutamiento de Azure Maps presenta numerosas características adicionales que no están disponibles en Mapas de Bing que pueden ser útiles de cara a la integración al migrar la aplicación:

* Compatibilidad con el tipo de ruta: la más corta, la más rápida y la de mayor ahorro de combustible.
* Compatibilidad con más modos de desplazamiento: bicicleta, autobús, motocicleta, taxi, camión y furgoneta.
* Compatibilidad con 150 puntos de referencia.
* Cálculo de varias duraciones de recorrido en una misma solicitud; tráfico histórico, tráfico en directo, sin tráfico.
* Posibilidad de descartar otros tipos de carreteras: transporte compartido, carreteras sin asfaltar, caminos ya usados.
* Enrutamiento basado en la especificación del motor. Cálculo de rutas para vehículos eléctricos o de combustión en función de la carga o el combustible que queda y de las especificaciones del motor.
* Especificación de la velocidad máxima del vehículo.

## <a name="snap-coordinates-to-road"></a>Ajuste de las coordenadas a la carretera

Hay varias maneras de ajustar las coordenadas a las carreteras en Azure Maps.

* Use Route Directions API para ajustar las coordenadas a una ruta lógica a lo largo de la red de carreteras.
* Use el SDK web de Azure Maps para ajustar coordenadas individuales a la carretera más cercana en los mosaicos vectoriales.
* Use directamente los mosaicos vectoriales de Azure Maps para ajustar coordenadas individuales.

**Uso de Route Directions API para ajustar las coordenadas**

Azure Maps puede ajustar las coordenadas a las carreteras mediante [Route Directions API](/rest/api/maps/route/postroutedirections). Este servicio se puede usar para reconstruir una ruta lógica entre un conjunto de coordenadas y es comparable a Snap to Road API de Mapas de Bing.

Hay dos maneras diferentes de usar Route Directions API para ajustar las coordenadas a las carreteras.

* Si hay 150 coordenadas o menos, se pueden pasar como puntos de referencia en GET Route Directions API. Con este enfoque se pueden recuperar dos tipos diferentes de datos ajustados; las instrucciones de ruta contendrán los puntos de referencia ajustados individuales, mientras que el trazado de ruta tendrá un conjunto interpolado de coordenadas que rellenan el trazado completo entre ellas.
* Si hay más de 150 coordenadas, se puede usar POST Route Directions API. Las coordenadas iniciales y finales tienen que pasarse al parámetro de consulta, pero todas las coordenadas se pueden pasar al parámetro `supportingPoints` en el cuerpo de la solicitud POST y formatearse como una colección de puntos de geometría GeoJSON. Los únicos datos ajustados disponibles con este enfoque serán el trazado de ruta, que es un conjunto interpolado de coordenadas que rellenan el trazado completo entre ellas. [Este es un ejemplo](https://azuremapscodesamples.azurewebsites.net/?sample=Snap%20points%20to%20logical%20route%20path) de este enfoque mediante el módulo de servicios del SDK web de Azure Maps.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing    | Parámetro de API de Azure Maps similar                                 |
|----------------------------|---------------------------------------------------------------------|
| `points`                   | `supportingPoints`: pase estos puntos al cuerpo de la solicitud POST.  |
| `interpolate`              | N/D                                                                 |
| `includeSpeedLimit`        | N/D                                                                 |
| `includeTruckSpeedLimit`   | N/D                                                                 |
| `speedUnit`                | N/D                                                                 |
| `travelMode`               | `travelMode`                                                        |
| `key`                      | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)            | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))   |
| `userRegion` (`ur`)        | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views).   |

La API de enrutamiento de Azure Maps también admite el parámetro de enrutamiento de camiones dentro de la misma API para garantizar que se calculan los trazados lógicos. En la siguiente tabla se contrastan los parámetros de enrutamiento de camiones de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing                 | Parámetro de API de Azure Maps similar        |
|-----------------------------------------|--------------------------------------------|
| `dimensionUnit` (`dims`)                | N/D: solo se admiten dimensiones en metros. |
| `weightUnit` (`wu`)                     | N/D: solo se admiten pesos en kilogramos. |
| `vehicleHeight` (`height`)              | `vehicleHeight`                            |
| `vehicleWidth` (`width`)                | `vehicleWidth`                             |
| `vehicleLength` (`vl`)                  | `vehicleLength`                            |
| `vehicleWeight` (`weight`)              | `vehicleWeight`                            |
| `vehicleAxles` (`axles`)                | `vehicleAxelWeight`                        |
| `vehicleTrailers` (`vt`)                | **N/D**                                    |
| `vehicleSemi` (`semi`)                  | `vehicleCommercial`                        |
| `vehicleMaxGradient` (`vmg`)            | **N/D**                                    |
| `vehicleMinTurnRadius` (`vmtr`)         | **N/D**                                    |
| `vehicleAvoidCrossWind` (`vacw`)        | **N/D**                                    |
| `vehicleAvoidGroundingRisk` (`vagr`)    | **N/D**                                    |
| `vehicleHazardousMaterials` (`vhm`)     | `vehicleLoadType`                          |
| `vehicleHazardousPermits` (`vhp`)       | `vehicleLoadType`                          |

Dado que en este enfoque se usa Route Directions API, se puede utilizar el conjunto completo de opciones de ese servicio para personalizar la lógica utilizada para ajustar la coordenada a las carreteras. Por ejemplo, si se especifica una hora de salida, se tendrán en cuenta los datos de tráfico históricos.

Route Directions API de Azure Maps no devuelve actualmente datos de límite de velocidad, pero se pueden recuperar mediante Reverse Geocoding API de Azure Maps.

**Uso del SDK web para ajustar coordenadas**

El SDK web de Azure Maps usa mosaicos vectoriales para representar los mapas. Estos mosaicos contienen la información de geometría de carreteras sin procesar y se pueden usar para calcular la carretera más cercana a una coordenada a fin de ajustar de forma sencilla las coordenadas individuales. Este enfoque resulta útil si quiere que las coordenadas aparezcan visualmente sobre las carreteras y ya usa el SDK web de Azure Maps para visualizar los datos.

Sin embargo, con él el ajuste solo se realizará a los segmentos de carretera ubicados dentro de la vista del mapa. Al hacer zoom para alejar en el nivel de país, es posible que no haya datos de carretera, por lo que no se podrá realizar el ajuste; sin embargo, en ese nivel de zoom, un solo píxel puede representar el área de varios bloques de ciudad, por lo que no es necesario el ajuste. Para solucionarlo, se puede aplicar la lógica de ajuste cada vez que se haya terminado de mover el mapa. [Este es un ejemplo de código](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Basic%20snap%20to%20road%20logic) donde se muestra esto.

**Uso de los mosaicos vectoriales de Azure Maps directamente para ajustar las coordenadas**

Los mosaicos vectoriales de Azure Maps contienen los datos de geometría de carreteras sin procesar que se pueden usar para calcular el punto más cercano en una carretera hasta una coordenada a fin de ajustar de forma básica las coordenadas individuales. Todos los segmentos de carretera aparecen en los sectores con el nivel de zoom 15, así que querrá recuperar los mosaicos desde ahí. Después, puede usar la [matemática de pirámides de mosaicos de quadtree](./zoom-levels-and-tile-grid.md) para determinar la necesidad de mosaicos y convertir estos en geometrías. A partir de ahí, se puede usar una biblioteca de matemática espacial, como [turf js](http://turfjs.org/) o [NetTopologySuite](https://github.com/NetTopologySuite/NetTopologySuite) para calcular los segmentos de línea más próximos.

## <a name="retrieve-a-map-image-static-map"></a>Recuperación de una imagen de mapa (Static Map)

Azure Maps proporciona una API para representar imágenes de mapa estático con datos superpuestos. [Map Image Render API](/rest/api/maps/render/getmapimagerytile) de Azure Maps es comparable a Static Map API de Mapas de Bing.

> [!NOTE]
> En Azure Maps, es necesario que el centro, todos los marcadores y las ubicaciones de trazado sean coordenadas en formato `longitude,latitude`, mientras que en Mapas de Bing se usa el formato `latitude,longitude`.</p>
<p>y que las direcciones estén geocodificadas.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing  | Parámetro de API de Azure Maps similar            |
|--------------------------|------------------------------------------------|
| `centerPoint`            | `center`                                       |
| `format`                 | `format` (se especifica como parte de la ruta de dirección URL). Actualmente, solo se admite el formato PNG.  |
| `heading`                | N/D: no se admite Streetside.                |
| `imagerySet`             | `layer` y `style` (vea la documentación de [estilos de mapa compatibles](./supported-map-styles.md))   |
| `mapArea` (`ma`)         | `bbox`                                         |
| `mapLayer` (`ml`)        | N/D                                            |
| `mapSize` (`ms`)         | `width` y `height` (puede tener un tamaño máximo de 8192 x 8192) |
| `declutterPins` (`dcl`)  | N/D                                            |
| `dpi`                    | N/D                                            |
| `drawCurve`              | `path`                                         |
| `mapMetadata`            | N/D                                            |
| `pitch`                  | N/D: no se admite Streetside.                |
| `pushpin` (`pp`)         | `pins`                                         |
| `zoomLevel`              | `zoom`                                         |
| `query`                  | N/D: se debe usar el centro o el rectángulo de selección.     |
| `highlightEntity` (`he`) | N/D                                            |
| `style`                  | N/D                                            |
| parámetros de ruta         | N/D                                            |
| `key`                    | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)          | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))   |
| `userRegion` (`ur`)      | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

> [!NOTE]
> Azure Maps usa un sistema de mosaicos que tienen el doble del tamaño de los mosaicos de mapa usados en Mapas de Bing, con lo cual el valor de nivel de zoom en Azure Maps será un nivel de zoom más cercano en comparación con Mapas de Bing. Para compensarlo, reduzca a uno menos el nivel de zoom en las solicitudes que vaya a migrar.

Para más información, consulte la [Guía de procedimientos de Map Image render API](./how-to-render-custom-data.md).

Además de poder generar una imagen de mapa estático, el servicio de representación de Azure Maps también ofrece la posibilidad de acceder directamente a los mosaicos de mapa en formato de trama (PNG) y vectorial:

-   [Mosaico de mapa](/rest/api/maps/render/getmaptile): recupere mosaicos de trama (PNG) y vectoriales de los mapas base (carreteras, fronteras, entorno).
-   [Mosaico de imágenes de mapa](/rest/api/maps/render/getmapimagerytile): recupere mosaicos de imágenes aéreas y por satélite.

### <a name="pushpin-url-parameter-format-comparison"></a>Parámetro pushpin en la dirección URL: comparación de formatos

**Antes: Mapas de Bing**

En Mapas de Bing, se pueden agregar marcadores a una imagen de mapa estático usando el parámetro `pushpin` en la dirección URL. El parámetro `pushpin` toma una ubicación con el formato `latitude,longitude`, un estilo de icono y una etiqueta de texto (hasta tres caracteres), como se muestra a continuación:

> `&pushpin=latitude,longitude;iconStyle;label`

Se pueden agregar marcadores adicionales agregando otros parámetros `pushpin` a la dirección URL con un conjunto diferente de valores. Los estilos de icono de marcador se limitan a uno de los estilos predefinidos disponibles en la API de Mapas de Bing.

Por ejemplo, en Mapas de Bing, se puede agregar un marcador rojo con la etiqueta "AB" al mapa en las coordenadas (longitud: -110, latitud: 45) con el siguiente parámetro de dirección URL:

> `&pushpin=45,-110;7;AB`

![Chincheta de mapa estático de Mapas de Bing](media/migrate-bing-maps-web-service/bing-maps-static-map-pin.jpg)

**Después: Azure Maps**

En Azure Maps, también se pueden agregar marcadores a una imagen de mapa estático especificando el parámetro `pins` en la dirección URL. Los marcadores de Azure Maps se definen mediante la especificación de un estilo de icono y una lista de ubicaciones que lo usan. A continuación, esta información se pasa al parámetro `pins`, que se puede especificar varias veces para permitir marcadores con distintos estilos.

> `&pins=iconType|pinStyles||pinLocation1|pinLocation2|...`

Para usar más estilos, hay que agregar más parámetros `pins` a la dirección URL con un estilo y un conjunto de ubicaciones diferentes.

En lo referente a las ubicaciones de las chinchetas, en Azure Maps es necesario que las coordenadas tengan el formato `longitude latitude`, mientras que en Mapas de Bing se usa el formato `latitude,longitude`. Fíjese también en que **hay un espacio (y no una coma)** entre la longitud y la latitud en Azure Maps.

El valor `iconType` especifica el tipo de anclaje que se va a crear y puede tener los valores siguientes:

* `default`: icono de anclaje predeterminado.
* `none`: no se muestra ningún icono, solo se representarán etiquetas.
* `custom`: especifica que se va a usar un icono personalizado. Se puede agregar una dirección URL que apunte a la imagen del icono al final del parámetro `pins`, después de la información de ubicación del anclaje.
* `{udid}`: identificador de datos único (UDID) de un icono almacenado en la plataforma de almacenamiento de datos de Azure Maps.

Los estilos de anclaje en Azure Maps se agregan con el formato `optionNameValue`, con cada estilo separado por un carácter de barra vertical (`|`), así: `iconType|optionName1Value1|optionName2Value2`. Fíjese en que los nombres de opción y los valores no se separan. Se pueden usar los siguientes nombres de opciones de estilo para aplicar estilo a los marcadores de Azure Maps:

* `al`: especifica la opacidad (valor alfanumérico) de los marcadores. Puede ser un número entre 0 y 1.
* `an`: especifica el delimitador del anclaje. Los valores de píxel x e y se especifican con el formato `x y`.
* `co`: color del anclaje. Debe ser un color hexadecimal de 24 bits: de `000000` a `FFFFFF`.
* `la`: especifica el delimitador de la etiqueta. Los valores de píxel x e y se especifican con el formato `x y`.
* `lc`: color de la etiqueta. Debe ser un color hexadecimal de 24 bits: de `000000` a `FFFFFF`.
* `ls`: tamaño de la etiqueta en píxeles. Puede ser un número mayor que 0.
* `ro`: valor en grados para girar el icono. Puede ser un número entre -360 y 360.
* `sc`: valor de escala del icono de anclaje. Puede ser un número mayor que 0.

Se especifican valores de etiqueta para la ubicación de cada chincheta, en lugar de tener un valor de etiqueta único que se aplique a todos los marcadores de la lista de ubicaciones. El valor de etiqueta puede ser una cadena de varios caracteres y especificarse entre comillas simples para no confundirlo con un valor de estilo o de ubicación.

Por ejemplo, en Azure Maps, se puede agregar un icono predeterminado rojo (`FF0000`) con la etiqueta "Space Needle" situada debajo (15 50) del icono en las coordenadas (longitud: -122,349300, latitud: 47,620180) con el siguiente parámetro de dirección URL:

> `&pins=default|coFF0000|la15 50||'Space Needle'-122.349300 47.620180`

![Chincheta de mapa estático de Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-pin.jpg)

En el siguiente ejemplo se agregan tres anclajes con los valores de etiqueta "1", "2" y "3":

> `&pins=default||'1'-122 45|'2'-119.5 43.2|'3'-121.67 47.12`

![Varias chinchetas de mapa estático de Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-multiple-pins.jpg)

### <a name="draw-curve-url-parameter-format-comparison"></a>Parámetro drawCurve en la dirección URL: comparación de formatos

**Antes: Mapas de Bing**

En Mapas de Bing, se pueden agregar líneas y polígonos a una imagen de mapa estático usando el parámetro `drawCurve` en la dirección URL. El parámetro `drawCurve` toma un estilo de forma, un tipo de estilo y una lista de ubicaciones que se van a representar en el mapa, como se muestra a continuación:

> `&drawCurve=shapeType,styleType,location1,location2...`

Para usar más estilos, hay que agregar más parámetros `drawCurve` a la dirección URL con un estilo y un conjunto de ubicaciones diferentes.

Las ubicaciones en Mapas de Bing se especifican con el formato `latitude1,longitude1_latitude2,longitude2_…`. También se pueden codificar las ubicaciones.

Los tipos de formas de Mapas de Bing incluyen líneas, polígonos, círculos y curvas. Los tipos de estilo incluyen color de línea, grosor de línea, color de contorno, color de relleno, grosor de contorno y radio circular.

Por ejemplo, en Mapas de Bing, se puede agregar una línea azul con una opacidad del 50 % y un grosor de cuatro píxeles al mapa entre las coordenadas (longitud: -110, latitud: 45 y longitud: -100, latitud: 50) con el siguiente parámetro de dirección URL:

`&drawCurve=l,FF000088,4;45,-110_50,-100`

![Línea de mapa estático de Mapas de Bing](media/migrate-bing-maps-web-service/bing-maps-static-map-line.jpg)

**Después: Azure Maps**

En Azure Maps, también se pueden agregar líneas y polígonos a una imagen de mapa estático especificando el parámetro *path* en la dirección URL. Al igual que Mapas de Bing, se puede especificar un estilo y una lista de ubicaciones en este parámetro, y el parámetro *path* se puede especificar varias veces para representar varios círculos, líneas y polígonos con distintos estilos.

> `&path=pathStyles||pathLocation1|pathLocation2|...`

En lo referente a las ubicaciones de los trazados, en Azure Maps es necesario que las coordenadas tengan el formato `longitude latitude`, mientras que en Mapas de Bing se usa el formato `latitude,longitude`. Fíjese también en que hay un **espacio (y no una coma)** entre la longitud y la latitud en Azure Maps. En la actualidad, Azure Maps no admite trazados codificados. Se pueden cargar conjuntos de datos más grandes como un objeto GeoJSON en la API de almacenamiento de datos de Azure Maps, como se documenta [aquí](./how-to-render-custom-data.md#get-data-from-azure-maps-data-storage).

Los estilos de ruta en Azure Maps se agregan con el formato `optionNameValue`, con cada estilo separado por un carácter de barra vertical (`|`), así: `optionName1Value1|optionName2Value2`. Fíjese en que los nombres de opción y los valores no se separan. Se pueden usar los siguientes nombres de opción de estilo para aplicar estilo a las rutas de Azure Maps:

* `fa`: opacidad del color de relleno (alfanumérico) usada al representar polígonos. Puede ser un número entre 0 y 1.
* `fc`: color de relleno usado para representar el área de un polígono.
* `la`: opacidad del color de línea (alfanumérico) usada al representar líneas y el contorno de polígonos. Puede ser un número entre 0 y 1.
* `lc`: color de línea usado para representar líneas y el contorno de polígonos.
* `lw`: ancho de la línea en píxeles.
* `ra`: especifica el radio de un círculo en metros.

Por ejemplo, en Azure Maps, se puede agregar una línea azul con una opacidad del 50 % y un grosor de cuatro píxeles al mapa entre las coordenadas (longitud: -110, latitud: 45 y longitud: -100, latitud: 50) con el siguiente parámetro de dirección URL:

> `&path=lc0000FF|la.5|lw4||-110 45|-100 50`

![Línea de mapa estático de Azure Maps](media/migrate-bing-maps-web-service/azure-maps-static-map-line.jpg)

## <a name="calculate-a-distance-matrix"></a>Cálculo de una matriz de distancia

Azure Maps proporciona una API para calcular los tiempos y las distancias de desplazamiento entre un conjunto de ubicaciones como una matriz de distancia. La API de matriz de distancia de Azure Maps es similar a la API con el mismo nombre de Mapas de Bing:

-   [Matriz de ruta](/rest/api/maps/route/postroutematrixpreview): calcula asincrónicamente los tiempos y las distancias de desplazamiento de un conjunto de orígenes y destinos. Se admite un máximo de 700 celdas por solicitud (número de orígenes multiplicado por el número de destinos). Teniendo en cuenta esa restricción, estos son algunos ejemplos de posibles dimensiones de la matriz: `700x1`, `50x10`, `10x10`, `28x25`, `10x70`.

> [!NOTE]
> Las solicitudes a la API de matriz de distancia solo se pueden realizar mediante una solicitud POST, con la información de origen y de destino en el cuerpo de la solicitud.</p>
<p>Además, Azure Maps requiere que todos los orígenes y destinos sean coordenadas y que las direcciones estén geocodificadas.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing | Parámetro de API de Azure Maps similar                         |
|-------------------------|-------------------------------------------------------------|
| `origins`               | `origins`: especifíquelo como GeoJSON en el cuerpo de la solicitud POST.    |
| `destinations`          | `destination`: especifíquelo como GeoJSON en el cuerpo de la solicitud POST. |
| `endTime`               | `arriveAt`                                                  |
| `startTime`             | `departAt`                                                  |
| `travelMode`            | `travelMode`                                                |
| `resolution`            | N/D                                                         |
| `distanceUnit`          | N/D: todas las distancias en metros.                              |
| `timeUnit`              | N/D: todas las horas en segundos.                                 |
| `key`                   | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)         | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))  |
| `userRegion` (`ur`)     | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views).     |

> [!TIP]
> Todas las opciones de enrutamiento avanzadas disponibles en la API de enrutamiento de Azure Maps (enrutamiento de camiones, especificaciones del motor, elementos que pueden evitarse...) son compatibles con la API de matriz de distancia de Azure Maps.

## <a name="calculate-an-isochrone"></a>Cálculo de una isócrona

Azure Maps proporciona una API para calcular una isócrona, un polígono que cubre un área que se puede recorrer en cualquier dirección desde un punto de origen en un período especificado o según una cantidad de combustible o carga definida. Route Range API de Azure Mapas es comparable a Isochrone API de Mapas de Bing;

-   [Route](/rest/api/maps/route/getrouterange) Range**: calcula un polígono que abarca un área que se puede recorrer en cualquier dirección desde un punto de origen en un periodo especificado, según una distancia definida o la cantidad de combustible o carga disponible.

> [!NOTE]
> En Azure Maps, es necesario que el origen de la consulta sea una coordenada y que las direcciones estén geocodificadas.</p>
<p>Además, Mapas de Bing puede calcular isócronas en función de la hora o la distancia, mientras que Azure Maps puede calcularlas según el tiempo, la distancia o la cantidad de combustible o carga disponible.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing      | Parámetro de API de Azure Maps similar            |
|------------------------------|------------------------------------------------|
| `waypoint` (`wp`)            | `query`                                        |
| `dateTime` (`dt`)            | `departAt`                                     |
| `maxTime`                    | `timeBudgetInSec`                              |
| `timeUnit` (`tu`)            | N/D: todas las horas en segundos.                    |
| `travelMode` (`mode`)        | `travelMode`                                   |
| `maxDistance` (`maxDis`)     | `distanceBudgetInMeters`                       |
| `distanceUnit` (`du`)        | N/D: todas las distancias en metros.                 |
| `optimize` (`optmz`)         | `routeType`                                    |
| `key`                        | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)              | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))  |
| `userRegion` (`ur`)          | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

> [!TIP]
> Todas las opciones de enrutamiento avanzadas disponibles en la API de enrutamiento de Azure Maps (enrutamiento de camiones, especificaciones del motor, elementos que pueden evitarse...) son compatibles con Isochrone API de Azure Maps.

## <a name="search-for-points-of-interest"></a>Búsqueda de puntos de interés

En Mapas de Bing, los datos de punto de interés se pueden buscar mediante las siguientes API:

-   **Local Search:** busca puntos de interés cercanos (búsqueda radial), por nombre o por tipo de entidad (categoría). [POI Search API](/rest/api/maps/search/getsearchpoi) y [POI Category Search API](/rest/api/maps/search/getsearchpoicategory) de Azure Maps son muy parecidas a esta API.
-   **Reconocimiento de ubicación**: se buscan los puntos de interés que están a una determinada distancia de una ubicación. [Nearby Search API](/rest/api/maps/search/getsearchnearby) de Azure Maps es muy parecida a esta API.
-   **Información local:** busca los puntos de interés que se encuentran dentro del tiempo de conducción o la distancia máximos especificados desde una coordenada concreta. Esto se puede hacer con Azure Maps, para lo cual primero se calcula una isócrona y, luego, se pasa a [Search Within Geometry API](/rest/api/maps/search/postsearchinsidegeometry).

Azure Maps proporciona varias API de búsqueda de puntos de interés:

-   [Búsqueda de puntos de interés](/rest/api/maps/search/getsearchpoi): se buscan puntos de interés por su nombre Por ejemplo, `"starbucks"`.
-   [Búsqueda de categorías de puntos de interés](/rest/api/maps/search/getsearchpoicategory): se buscan puntos de interés por su categoría Por ejemplo, "restaurante".
-   [Búsqueda de elementos cercanos](/rest/api/maps/search/getsearchnearby): se buscan los puntos de interés que están a una determinada distancia de una ubicación.
-   [Búsqueda aproximada](/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. La API toma una cadena de forma libre (que puede ser una dirección, un lugar, un punto de referencia, un punto de interés o una categoría de punto de interés) y procesa la solicitud inmediatamente. Se recomienda usar esta API en situaciones en las que los usuarios pueden buscar direcciones o puntos de interés desde el mismo cuadro de texto.
-   [Search Within Geometry](/rest/api/maps/search/postsearchinsidegeometry): se buscan los puntos de interés que se encuentran dentro de una geometría específica (polígono).
-   [Búsqueda a lo largo de una ruta](/rest/api/maps/search/postsearchalongroute): se buscan los puntos de interés que hay a lo largo del trayecto de una ruta específica.
-   [Búsqueda parcial por lotes](/rest/api/maps/search/postsearchfuzzybatchpreview): cree una solicitud que contenga hasta 10 000 direcciones, lugares, puntos de referencia o puntos de interés y procéselos durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.

Asegúrese de revisar la documentación de [procedimientos recomendados de búsqueda](./how-to-use-best-practices-for-search.md).

## <a name="get-traffic-incidents"></a>Obtención de los incidentes de tráfico

Azure Maps proporciona varias API para recuperar los datos de tráfico. Hay dos tipos de datos de tráfico disponibles:

* **Datos de flujo**: proporcionan métricas sobre el flujo de tráfico en secciones de carreteras. Se suele usar para codificar por colores las carreteras. Estos datos se actualizan cada dos minutos.
* **Datos de incidentes**: proporcionan información sobre construcción, cierres de carreteras, accidentes y otros incidentes que pueden afectar al tráfico. Estos datos se actualizan cada minuto.

Mapas de Bing proporciona datos de los incidentes y del flujo de tráfico en sus controles de mapa interactivos, y también permite que los datos de incidentes estén disponibles como servicio.

Los datos de tráfico también se integran en los controles de mapa interactivos de Azure Maps. Azure Maps también proporciona las siguientes API de servicios de tráfico:

-   [Traffic Flow Segments](/rest/api/maps/traffic/gettrafficflowsegment): proporciona información sobre las velocidades y los tiempos de desplazamiento del fragmento de carretera más próximo a las coordenadas especificadas.
-   [Traffic Flow Tiles](/rest/api/maps/traffic/gettrafficflowtile): proporciona mosaicos de trama y vectoriales que contienen datos del flujo de tráfico. Se pueden usar con los controles de Azure Maps o en controles de mapas de terceros como Leaflet. Los mosaicos vectoriales también se pueden usar para análisis de datos avanzado.
-   [Traffic Incident Details](/rest/api/maps/traffic/gettrafficincidentdetail): proporciona detalles de los incidentes de tráfico que se encuentran dentro de un rectángulo de selección, un nivel de zoom y un modelo de tráfico.
-   [Traffic Incident Tiles](/rest/api/maps/traffic/gettrafficincidenttile): proporciona mosaicos de trama y vectoriales que contienen datos de incidentes de tráfico.
-   [Traffic Incident Viewport](/rest/api/maps/traffic/gettrafficincidentviewport): recupera la información legal y técnica de la ventanilla descrita en la solicitud, como el identificador del modelo de tráfico.

En la tabla siguiente se contrastan los parámetros de la API de tráfico de Mapas de Bing con los parámetros de la API de detalles de incidentes de tráfico comparable de Azure Maps.

| Parámetro de API de Mapas de Bing  | Parámetro de API de Azure Maps similar   |
|--------------------------|---------------------------------------|
| `mapArea`                | `boundingBox` y `boundingZoom`      |
| `includeLocationCodes`   | N/D                                   |
| `severity` (`s`)         | N/D: todos los datos devueltos.               |
| `type` (`t`)             | N/D: todos los datos devueltos.               |
| `key`                    | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)          | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md)) |
| `userRegion` (`ur`)      | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views). |

## <a name="get-a-time-zone"></a>Obtención de una zona horaria

Azure Maps proporciona una API para obtener la zona horaria en la que una coordenada se encuentra. Time Zone API de Azure Maps es comparable a la API con el mismo nombre de Mapas de Bing:

-   [Time Zone By Coordinate](/rest/api/maps/timezone/gettimezonebycoordinates): especifique una coordenada y obtenga los detalles de la zona horaria en la que se encuentra.

En la siguiente tabla se contrastan los parámetros de API de Mapas de Bing con los parámetros de API comparables de Azure Maps.

| Parámetro de API de Mapas de Bing | Parámetro de API de Azure Maps similar          |
|-------------------------|----------------------------------------------|
| `point`                 | `query`                                      |
| `query`                 | N/D: primero se deben geocodificar las ubicaciones.      |
| `dateTime`              | `timeStamp`                                  |
| `includeDstRules`       | N/D: siempre se incluye en la respuesta de Azure Maps. |
| `key`                   | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](./azure-maps-authentication.md)) |
| `culture` (`c`)         | `language` (vea la documentación de [compatibilidad de idiomas](./supported-languages.md))  |
| `userRegion` (`ur`)     | `view`: consulte la documentación de [vistas admitidas](./supported-languages.md#azure-maps-supported-views).  |

Además de esto, la plataforma de Azure Maps también proporciona otras API de zona horaria que ayudan en las conversiones de identificadores y nombres de zona horaria:

-   [Zona horaria por identificador](/rest/api/maps/timezone/gettimezonebyid): devuelve la información de zona horaria actual, histórica y futura relativa al identificador de zona horaria de IANA especificado.
-   [Enumeración de zonas horarias, IANA](/rest/api/maps/timezone/gettimezoneenumiana): devuelve una lista completa de los identificadores de zona horaria de IANA. Las actualizaciones del servicio IANA se reflejan en el sistema en el plazo de un día. 
-   [Enumeración de zonas horarias, Windows](/rest/api/maps/timezone/gettimezoneenumwindows): devuelve una lista completa de los identificadores de zona horaria de Windows.
-   [Versión de IANA de zona horaria](/rest/api/maps/timezone/gettimezoneianaversion): devuelve el número de versión de IANA actual utilizado por Azure Maps. 
-   [Zona horaria de Windows a IANA](/rest/api/maps/timezone/gettimezonewindowstoiana): devuelve el identificador de IANA correspondiente a un identificador de zona horaria de Windows válido especificado. Se pueden obtener varios identificadores de IANA de un mismo identificador de Windows.

## <a name="spatial-data-services-sds"></a>Servicios de datos espaciales (SDS)

Los servicios de datos espaciales de Mapas de Bing proporcionan tres funcionalidades clave:

* Geocodificación por lotes: procese un gran lote de códigos geográficos de direcciones con una sola solicitud.
* Recuperación de datos de límites administrativos: use una coordenada y obtenga un límite de intersección para un tipo de entidad especificado.
* Hospedaje y consulta de datos empresariales espaciales: cargue una tabla simple 2D de datos y acceda a ella con unas cuantas consultas espaciales sencillas.

### <a name="batch-geocode-data"></a>Datos de geocodificación geográfica por lotes

La geocodificación por lotes es el proceso de tomar un gran número de direcciones o lugares, pasarlos todos en una única solicitud a un servicio y hacer que todos ellos se geocodifiquen en paralelo y que los resultados se devuelvan en una sola respuesta.

Mapas de Bing permite pasar hasta 200 000 direcciones en una única solicitud de geocodificación por lotes. Esta solicitud entra en una cola y se procesa normalmente durante un período que va de unos minutos a unas horas, según el tamaño del conjunto de datos y la carga en el servicio. Cada dirección de la solicitud genera una transacción.

Azure Maps dispone de un servicio de geocodificación por lotes, pero permite pasar hasta 10 000 direcciones en una única solicitud, que se procesa en un intervalo de unos segundos a varios minutos, según el tamaño del conjunto de datos y la carga en el servicio. Cada dirección de la solicitud genera una transacción. En Azure Maps, el servicio de geocodificación por lotes solo está disponible en los planes de tarifa Gen 2 o S1. Para más información sobre los planes de tarifa, consulte [Elección del plan de tarifa adecuado de Azure Maps](choose-pricing-tier.md).

Otra opción para geocodificar una gran cantidad de direcciones con Azure Maps es realizar solicitudes paralelas a las API de búsqueda estándar. Estos servicios solo aceptan una dirección por solicitud, pero se pueden usar con el nivel S0 que también proporciona límites de uso gratis. El nivel S0 permite hasta 50 solicitudes por segundo a la plataforma de Azure Maps desde una sola cuenta. Por lo tanto, si el límite de procesamiento permanece dentro de ese valor, es posible geocodificar en sentido ascendente 180 000 direcciones por hora. Los planes de tarifa Gen 2 o S1 no tienen un límite documentado sobre el número de consultas por segundo que se pueden realizar desde una cuenta, por lo que se pueden procesar muchos más datos más rápido cuando se usan esos planes de tarifa; sin embargo, el uso del servicio de geocodificación por lotes ayudará a disminuir la cantidad total de datos transferidos y reducirá drásticamente el tráfico de red.

-   [Geocodificación de direcciones en formato libre](/rest/api/maps/search/getsearchaddress): especifique una única cadena de dirección (como `"1 Microsoft way, Redmond, WA"`) y procese la solicitud inmediatamente. Este es el servicio recomendado si hay que geocodificar direcciones de forma individual rápidamente.
-   [Geocodificación de direcciones estructurada](/rest/api/maps/search/getsearchaddressstructured): especifique las partes de una dirección (como el nombre de la calle, la ciudad, el país y el código postal) y procese la solicitud inmediatamente. Este es el servicio recomendado si hay que geocodificar direcciones de forma individual rápidamente y los datos ya se han separado en sus partes individuales.
-   [Geocodificación de direcciones por lotes](/rest/api/maps/search/postsearchaddressbatchpreview): cree una solicitud que contenga hasta 10 000 direcciones y procéselas durante un período de tiempo determinado. Todas las direcciones se geocodificarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo. Este es el servicio recomendable para geocodificar grandes conjuntos de datos.
-   [Búsqueda aproximada](/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. La API toma una cadena de forma libre (que puede ser una dirección, un lugar, un punto de referencia, un punto de interés o una categoría de punto de interés) y procesa la solicitud inmediatamente. Se recomienda usar esta API en situaciones en las que los usuarios pueden buscar direcciones o puntos de interés desde el mismo cuadro de texto.
-   [Búsqueda parcial por lotes](/rest/api/maps/search/postsearchfuzzybatchpreview): cree una solicitud que contenga hasta 10 000 direcciones, lugares, puntos de referencia o puntos de interés y procéselos durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.

### <a name="get-administrative-boundary-data"></a>Obtención de los datos de límite administrativo

En Mapas de Bing, los límites administrativos de países, estados/provincias, municipios, ciudades y códigos postales están disponibles a través de Geodata API. Esta API toma una coordenada o una consulta para geocodificar. Si se pasa una consulta, se geocodifica y se usan las coordenadas del primer resultado. Esta API toma las coordenadas y recupera el límite del tipo de entidad especificado que forma una intersección con la coordenada. Tenga en cuenta que esta API no devuelve necesariamente el límite de la consulta que se pasa. Si se pasa una consulta para `"Seattle, WA"`, pero el valor del tipo de entidad se establece en país o región, se devolverá el límite de Estados Unidos.

Azure Maps también proporciona acceso a límites administrativos (países, estados/provincias, municipios, ciudades y códigos postales). Para recuperar un límite, debe consultar una de las API de búsqueda del límite que elija (por ejemplo, `Seattle, WA`). Si el resultado de la búsqueda tiene un límite asociado, se proporcionará un identificador de geometría en la respuesta del resultado. Search Polygon API se puede usar para recuperar los límites exactos de uno o más identificadores de geometría. Esto es un poco diferente de Mapas de Bing, dado que Azure Maps devuelve el límite de lo que se ha buscado, mientras que Mapas de Bing devuelve un límite para un tipo de entidad especificado en una coordenada especificada. Además, los datos de límite devueltos por Azure Maps están en formato GeoJSON.

En resumen:

1.  Pase una consulta para el límite que quiera recibir en una de las siguientes API de búsqueda.
    -   [Free-Form Address Geocoding](/rest/api/maps/search/getsearchaddress)
    -   [Structured Address Geocoding](/rest/api/maps/search/getsearchaddressstructured)
    -   [Batch Address Geocoding](/rest/api/maps/search/postsearchaddressbatchpreview)
    -   [Búsqueda aproximada](/rest/api/maps/search/getsearchfuzzy)
    -   [Fuzzy Batch Search](/rest/api/maps/search/postsearchfuzzybatchpreview)
2.  Si los resultados deseados tienen un identificador de geometría, páselo a [Search Polygon API](/rest/api/maps/search/getsearchpolygon).

### <a name="host-and-query-spatial-business-data"></a>Hospedaje y consulta de datos empresariales espaciales

Los servicios de datos espaciales de Mapas de Bing proporcionan una sencilla solución de almacenamiento de datos espaciales para hospedar datos empresariales y exponerlos como un servicio REST espacial. Este servicio proporciona cuatro consultas principales: búsqueda por propiedad, búsqueda cercana, búsqueda en el rectángulo de selección y búsqueda con 1 kilómetro de una ruta. Con frecuencia, muchas empresas que utilizan este servicio ya tienen sus datos empresariales almacenados en una base de datos en algún lugar y han cargado un pequeño subconjunto de ellos en este servicio para aplicaciones avanzadas, como los localizadores de tiendas. Dado que la autenticación basada en claves proporciona seguridad básica, se recomienda usar este servicio solo con datos orientados al público.

La mayoría de los datos de ubicaciones empresariales comienzan en una base de datos. En ese sentido, se recomienda usar soluciones de Azure Storage existentes, como Azure SQL o Azure PostgreSQL (con el complemento PostGIS). Ambas soluciones de almacenamiento admiten datos espaciales y proporcionan un amplio conjunto de funcionalidades de consulta espacial. Una vez que los datos están en una solución de almacenamiento adecuada, se pueden integrar en la aplicación mediante la creación de un servicio web personalizado o con un marco, como ASP.NET o Entity Framework. El uso de este enfoque proporciona más funcionalidades de consulta, así como opciones de seguridad mucho más elevada.

Azure Cosmos DB también proporciona un conjunto limitado de funcionalidades espaciales que, según su escenario, pueden ser suficientes.

Estos son algunos recursos útiles sobre el hospedaje y la consulta de datos espaciales en Azure.

-   [Introducción a los tipos de datos espaciales de Azure SQL](/sql/relational-databases/spatial/spatial-data-types-overview)
-   [Consultar datos espaciales para el vecino más próximo](/sql/relational-databases/spatial/query-spatial-data-for-nearest-neighbor)
-   [Introducción a las funcionalidades geoespaciales de Azure Cosmos DB](../cosmos-db/sql-query-geospatial-intro.md)

## <a name="client-libraries"></a>Bibliotecas de clientes

Azure Maps proporciona bibliotecas cliente para los siguientes lenguajes de programación:

-   JavaScript, TypeScript, Node.js ([documentación](./how-to-use-services-module.md) \| [paquete de NPM](https://www.npmjs.com/package/azure-maps-rest))

Bibliotecas cliente de código abierto para otros lenguajes de programación:

* .NET Standard 2.0 ([proyecto de GitHub](https://github.com/perfahlen/AzureMapsRestServices) \| [paquete NuGet](https://www.nuget.org/packages/AzureMapsRestToolkit/)

## <a name="clean-up-resources"></a>Limpieza de recursos

No hay recursos para limpiar.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los servicios REST de Azure Maps.

> [!div class="nextstepaction"]
> [Procedimientos recomendados de uso del servicio de búsqueda](how-to-use-best-practices-for-search.md)
