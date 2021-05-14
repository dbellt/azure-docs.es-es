---
title: Preguntas frecuentes sobre los servicios Weather de Microsoft Azure Maps
description: Encuentre respuestas a preguntas comunes sobre los datos y las características de los servicios Weather de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 440ff94f4d5f3514d97d3e76299b60e462cebff3
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257558"
---
# <a name="azure-maps-weather-services-frequently-asked-questions-faq"></a>Preguntas frecuentes sobre los servicios Weather de Azure Maps

En este artículo se responde a preguntas comunes sobre los datos y las características de los [servicios de Weather](/rest/api/maps/weather) de Azure Maps. Se tratan los siguientes temas:

* Orígenes de datos y modelos de datos
* Cobertura y disponibilidad de los servicios de Weather
* Frecuencia de actualización de los datos
* Desarrollo con los SDK de Azure Maps
* Opciones para visualizar datos meteorológicos, incluida la integración con Microsoft Power BI

## <a name="data-sources-and-data-models"></a>Orígenes de datos y modelos de datos

**¿Cómo obtiene Azure Maps los datos meteorológicos?**

Azure Maps se ha creado con la colaboración de asociados de tecnología de movilidad y localización de talla mundial, como AccuWeather, que proporciona los datos meteorológicos subyacentes. Para leer el anuncio de la colaboración de Azure Maps con AccuWeather, consulte [Lluvia o sol: Los servicios de Weather de Azure Maps aportarán más información a su empresa](https://azure.microsoft.com/blog/rain-or-shine-azure-maps-weather-services-will-bring-insights-to-your-enterprise/).

AccuWeather cuenta con información meteorológica y medioambiental en tiempo real disponible en cualquier parte del mundo, en gran medida debido a sus asociaciones con numerosas agencias de meteorología nacionales gubernamentales y otros acuerdos privados. A continuación, se proporciona una lista de esta información fundamental.

* Observaciones de la superficie terrestre que publican los organismos gubernamentales
* Conjuntos de datos privados de observación de la superficie terrestre procedentes de gobiernos y empresas privadas
* Datos de radar de alta resolución de más de 40 países o regiones
* Los mejores datos mundiales de relámpagos en tiempo real
* Avisos meteorológicos emitidos por el gobierno de más de 60 países, regiones y territorios
* Datos satelitales procedentes de satélites meteorológicos geoestacionarios que abarcan todo el mundo
* Más de 150 modelos numéricos de previsión, como el modelado interno privado, modelos gubernamentales como el Global Forecast System (GFS) y modelos a escala reducida proporcionados por empresas privadas
* Observaciones de la calidad del aire
* Observaciones de los departamentos de transporte

Se incorporan decenas de miles de observaciones de la superficie, junto con otros datos, para crear las condiciones actuales puestas a disposición de los usuarios e influir en ellas. Aquí se incluyen no solo los conjuntos de datos estándar disponibles gratuitamente, sino también las observaciones únicas obtenidas de los servicios meteorológicos nacionales de muchos países o regiones, como la India, Brasil y Canadá y otras aportaciones privadas. Estos conjuntos de datos únicos aumentan la resolución espacial y temporal de los datos de las condiciones actuales para nuestros usuarios. 

La precisión de estos conjuntos de datos se revisa en tiempo real para el sistema de previsión digital, que emplea algoritmos de inteligencia artificial propiedad de AccuWeather para modificar continuamente las previsiones, de forma que se disponen de los datos más actuales y precisos.

**¿Qué modelos crean datos de previsión meteorológica?**

Para formular previsiones en todo el mundo, se usan numerosos sistemas de guía de previsiones meteorológicas. Cada día se emplean más de 150, que generan tanto conjuntos de datos externos como internos. Esto incluye modelos gubernamentales como el Centro Europeo de Previsiones Meteorológicas (CEPMPM) y el Global Forecast System (GFS). Además, AccuWeather incorpora modelos de alta resolución privados que reducen la escala de las previsiones a ubicaciones específicas y dominios regionales estratégicos a fin de predecir el tiempo con una mayor exactitud. En las últimas décadas, se han desarrollado algoritmos exclusivos de fusión y ponderación de AccuWeather. Estos algoritmos aprovechan de forma óptima las numerosas entradas de previsión para proporcionar información muy precisa.

## <a name="weather-services-coverage-and-availability"></a>Cobertura y disponibilidad de los servicios de Weather

**¿Qué tipo de cobertura puedo esperar en distintos países o regiones?**

La cobertura del servicio Weather varía según el país o la región. Todas las características no están disponibles en todos los países o regiones. Para más información, consulte la [documentación sobre cobertura](./weather-coverage.md).

## <a name="data-update-frequency"></a>Frecuencia de actualización de los datos

**¿Con qué frecuencia se actualizan los datos de las condiciones actuales?**

Los datos de las condiciones actuales se actualizan aproximadamente al menos una vez por hora, pero se pueden actualizar con más frecuencia cuando las condiciones cambian rápidamente, como bruscos cambios de temperatura, cambios en las condiciones del cielo, cambios en las precipitaciones, etc. La mayoría de las estaciones de observación de todo el mundo informan muchas veces por hora a medida que cambian las condiciones. Sin embargo, algunas áreas se seguirán actualizando solo una, dos o cuatro veces por hora a intervalos programados.  

Azure Maps almacena en caché los datos de las condiciones actuales durante un período de hasta 10 minutos para ayudar a capturar la frecuencia de actualización casi en tiempo real más cercana de los datos a medida que se generan. Para ver cuándo expira la respuesta almacenada en caché y evitar la visualización de datos obsoletos, puede aprovechar la información del encabezado Expires en el encabezado HTTP de la respuesta de API de Azure Maps.

**¿Con qué frecuencia se actualizan los datos de previsión diaria y por hora?**

Los datos de previsión diaria y por hora se actualizan varias veces al día, a medida que se reciben las observaciones actualizadas.  Por ejemplo, si se supera la temperatura alta o baja prevista, nuestros datos de previsión se ajustarán en el siguiente ciclo de actualización. Esto puede ocurrir a intervalos diferentes, pero normalmente sucede en un plazo de una hora. Muchas condiciones meteorológicas repentinas pueden provocar un cambio en los datos de previsión. Por ejemplo, en una tarde de verano calurosa, puede estallar de repente una tormenta aislada que dé lugar a cielos cubiertos y lluvia. La tormenta aislada puede hacer descender la temperatura hasta 10 grados. Este nuevo valor de temperatura afectará a las previsiones diarias y por hora durante el resto del día y, como tal, se actualizará en nuestros conjuntos de datos.

Las API de previsión de Azure Maps se almacenan en caché durante un período de hasta 30 minutos. Para ver cuándo expira la respuesta almacenada en caché y evitar la visualización de datos obsoletos, puede aprovechar la información del encabezado Expires en el encabezado HTTP de la respuesta de API de Azure Maps. Se recomienda actualizar cuando sea necesario en función del caso de uso de un producto específico y de la interfaz de usuario (IU).

## <a name="developing-with-azure-maps-sdks"></a>Desarrollo con los SDK de Azure Maps

**¿Admite el SDK de web de Azure Maps de forma nativa la integración de los servicios Weather?**

El SDK web de Azure Maps proporciona un módulo de servicios. Este módulo es una biblioteca auxiliar que facilita el uso de los servicios de REST de Azure Maps en aplicaciones web o de Node.js mediante JavaScript o TypeScript. Para empezar, consulte nuestra [documentación](./how-to-use-services-module.md).

**¿Admite Android SDK de Azure Maps de forma nativa la integración de los servicios Weather?**

Las instancias de Android SDK de Azure Maps admiten capas de mosaicos de Mercator, que pueden tener notación x/y/zoom, notación de clave cuádruple o notación de cuadro de límite EPSG 3857.

Está previsto crear un módulo de servicios para Java/Android similar al módulo del SDK web. El módulo de servicios de Android facilitará el acceso a todos los servicios de Azure Maps en una aplicación Java o Android.  

## <a name="data-visualizations"></a>Visualizaciones de datos  

**¿Admite el objeto visual de Power BI de Azure Maps los mosaicos meteorológicos?**

Sí. Para información sobre cómo migrar mosaicos de satélite radar y de infrarrojos al objeto visual de Microsoft Power BI, consulte [Adición de una capa de mosaicos al objeto visual de Power BI](./power-bi-visual-add-tile-layer.md). 

**¿Cómo se interpretan los colores usados para los mosaicos de radar y satélite?**

El [artículo sobre conceptos de Weather](./weather-services-concepts.md#radar-and-satellite-imagery-color-scale) incluye una guía para ayudar a interpretar los colores usados en los mosaicos de radar y satélite. En el artículo se tratan las muestras de color y los códigos de color hexadecimales.
 
**¿Puedo crear animaciones de mosaicos de radar y satélite?**

Sí. Además de los mosaicos de radar y satélite en tiempo real, los clientes de Azure Maps pueden solicitar mosaicos pasados y futuros para mejorar las visualizaciones de datos con las superposiciones de mapas. Para ello, se puede llamar directamente a [Get Map Tile v2 API](/rest/api/maps/renderv2/getmaptilepreview) o solicitar mosaicos mediante el SDK web de Azure Maps. Se proporcionan mosaicos de radar para 1,5 horas en el pasado y 2 horas en el futuro, como máximo. Los mosaicos están disponibles en intervalos de 5 minutos. Se proporcionan mosaicos de infrarrojos para 3 horas en el pasado como máximo y están disponibles en intervalos de 10 minutos. Para más información, consulte el [ejemplo de código](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Animated%20tile%20layer) de la animación de mosaicos meteorológicos de código abierto.  

**¿Se ofrecen iconos para distintas condiciones meteorológicas?**

Sí. Puede buscar iconos y sus códigos respectivos [aquí](./weather-services-concepts.md#weather-icons). Tenga en cuenta que solo algunas de las API del servicio Weather (versión preliminar), como [Get Current Conditions API](/rest/api/maps/weather/getcurrentconditions), devuelven el valor de *iconCode* en la respuesta. Para más información, consulte el [ejemplo de código](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Get%20current%20weather%20at%20a%20location) de Current WeatherConditions.

## <a name="next-steps"></a>Pasos siguientes

Si no encuentra una respuesta a su pregunta, póngase en contacto con nosotros mediante los siguientes canales (en orden incremental):

* La sección Comentarios de este artículo.
* [Página de preguntas y respuestas de MSFT para Azure Maps](/answers/topics/azure-maps.html).
* Soporte técnico de Microsoft. Para crear una solicitud de soporte técnico, en [Azure Portal](https://portal.azure.com/), vaya a la pestaña Ayuda, seleccione el botón **Ayuda y soporte técnico** y elija **Nueva solicitud de soporte técnico**.
* [UserVoice de Azure Maps](https://feedback.azure.com/forums/909172-azure-maps) para enviar solicitudes de características.

Aprenda a solicitar datos meteorológicos en tiempo real y previsiones mediante los servicios Weather de Azure Maps:
> [!div class="nextstepaction"]
> [Solicitud de datos meteorológicos en tiempo real](how-to-request-weather-data.md)

Artículo sobre los conceptos de los servicios Weather de Azure Maps:
> [!div class="nextstepaction"]
> [Conceptos de servicios Weather](weather-coverage.md)

Explore la documentación de API de los servicios Weather de Azure Maps:

> [!div class="nextstepaction"]
> [Servicios de Weather de Azure Maps](/rest/api/maps/weather)