---
title: Novedades de Text Analytics API
titleSuffix: Text Analytics - Azure Cognitive Services
description: En este artículo se proporciona información sobre las versiones y características nuevas de Text Analytics de Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 06/03/2021
ms.author: aahi
ms.custom: references_regions
ms.openlocfilehash: 63063ef205b456c368cec2d5c1a8ce1f456aaa59
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111756480"
---
# <a name="whats-new-in-the-text-analytics-api"></a>Novedades de Text Analytics API

Text Analytics API se actualiza constantemente. Para mantenerse al día con los avances recientes, en este artículo se proporciona información sobre las nuevas versiones y características.

## <a name="june-2021"></a>Junio de 2021

### <a name="general-api-updates"></a>Actualizaciones de API generales

* Nueva versión del modelo `2021-06-01` para la extracción de frases clave, que agrega compatibilidad con chino simplificado.
* La versión del modelo de `2021-06-01` para [Reconocimiento de entidades con nombre](how-tos/text-analytics-how-to-entity-linking.md) v3.x, que proporciona: 
  * Se ha mejorado la calidad de la inteligencia artificial y se ha ampliado la compatibilidad con los idiomas en la categoría de entidad *Aptitud*. 
  * Se ha agregado compatibilidad con los idiomas español, francés, alemán, italiano y portugués en la categoría de entidad *Aptitud*.
* Tanto las operaciones asincrónicas como Text Analytics for Health están disponibles en todas las regiones. 

### <a name="text-analytics-for-health-updates"></a>Actualizaciones de Text Analytics for Health

* Ya no es necesario solicitar acceso para obtener una versión preliminar de Text Analytics for Health.
* Nueva versión del modelo `2021-05-15` para el punto de conexión `/health` y el contenedor local que proporciona
    * Cinco nuevos tipos de entidad: `ALLERGEN`, `CONDITION_SCALE`, `COURSE`, `EXPRESSION` y `MUTATION_TYPE`.
    * Catorce nuevos tipos de relación.
    * Detección de aserciones expandida para nuevos tipos de entidad.
    * Compatibilidad de vinculación con el tipo de entidad ALLERGEN.

 
## <a name="may-2021"></a>Mayo de 2021

* Ya se puede acceder a [Respuesta personalizada a preguntas](../qnamaker/custom-question-answering.md) la (anteriormente QnA Maker) mediante un recurso de Text Analytics. 

### <a name="general-api-updates"></a>Actualizaciones de API generales

* Versión de la nueva API v3.1-preview.5, que incluye: 
  * La [API de análisis](how-tos/text-analytics-how-to-call-api.md?tabs=asynchronous) asincrónica ahora admite Análisis de sentimiento (SA) y Minería de opiniones (OM).
  * Ahora, los clientes que opten por no registrar el texto de entrada de informes de incidentes disponen de un nuevo parámetro de consulta `LoggingOptOut`.  Para obtener más información sobre este parámetro, consulte el artículo sobre [privacidad de datos](/legal/cognitive-services/text-analytics/data-privacy?context=/azure/cognitive-services/text-analytics/context/context).
* Text Analytics for Health y las operaciones asincrónicas de análisis ya están disponibles en todas las regiones.

## <a name="march-2021"></a>Marzo de 2021

### <a name="general-api-updates"></a>Actualizaciones de API generales
* Versión de la nueva API v3.1-preview.4 que incluye 
   * Cambios en el cuerpo de respuesta JSON de minería de opiniones: 
      * `aspects` ahora es `targets` y `opinions` ahora es `assessments`. 
   * Cambios en el cuerpo de respuesta JSON de la API web hospedada de Text Analytics for Health: 
      * El nombre booleano `isNegated` de un objeto de entidad detectado para la negación está en desuso y se ha reemplazado por la detección de aserciones.
      * Una nueva propiedad denominada `role` ahora forma parte de la relación extraída entre un atributo y una entidad, así como de la relación entre las entidades.  Agrega especificidad al tipo de relación detectado.
   * La vinculación de entidades ahora está disponible como tarea asincrónica en el punto de conexión `/analyze`.
   * Un nuevo parámetro `pii-categories` ahora está disponible en el punto de conexión `/pii`.
      * Este parámetro le permite especificar determinadas entidades de información de identificación personal, así como las que no se admiten de forma predeterminada para el idioma de entrada.
* Bibliotecas cliente actualizadas, que incluye operaciones asíncrónicas de Analyze y Text Analytics for Health. Puede encontrar ejemplos en GitHub:

    * [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
    * [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
    * [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)
    * [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/v5/javascript)
    
> [!div class="nextstepaction"]
> [Más información sobre Text Analytics API v3.1-Preview.4](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-4/operations/Languages)

### <a name="text-analytics-for-health-updates"></a>Actualizaciones de Text Analytics for Health

* Nueva versión del modelo `2021-03-01` para el punto de conexión `/health` y el contenedor local que proporciona
    * Cambio de nombre del tipo de entidad `Gene` a `GeneOrProtein`.
    * Nuevo tipo de entidad `Date`.
    * Detección de aserciones que reemplaza la detección de negaciones (solo disponible en la API v3.1-preview.4).
    * Nueva propiedad `name` preferida para las entidades vinculadas que se ha normalizado a partir de varias ontologías y varios sistemas de codificación (solo disponible en la API v3.1-preview.4). 
* Se ha publicado una nueva imagen de contenedor con la etiqueta `3.0.015490002-onprem-amd64` y la nueva versión de modelo `2021-03-01` en el repositorio de vista previa de contenedores. 
    * A partir del 26 de abril de 2021 la imagen de este contenedor no volverá a estar disponible para la descarga desde `containerpreview.azurecr.io`.
* Ahora hay disponible una nueva instancia de Text Analytics para la imagen de contenedor de estado con estos mismos modelo-versión en `mcr.microsoft.com/azure-cognitive-services/textanalytics/healthcare`. A partir del 26 de abril, el contenedor solo se podrá descargar desde este repositorio.

> [!div class="nextstepaction"]
> [Más información sobre Text Analytics para el estado](how-tos/text-analytics-for-health.md)

### <a name="text-analytics-resource-portal-update"></a>Actualización del portal de recursos de Text Analytics
* Los **registros de texto procesados** ahora están disponibles como una métrica en la sección **Supervisión** del recurso de Text Analytics en Azure Portal.  

## <a name="february-2021"></a>Febrero de 2021

* La versión del modelo `2021-01-15` del punto de conexión PII en [Reconocimiento de entidades con nombre](how-tos/text-analytics-how-to-entity-linking.md) v3.1-preview.x, que proporciona: 
  * Compatibilidad ampliada con 9 idiomas nuevos
  * Calidad de inteligencia artificial mejorada de las categorías de entidades con nombre para todos los idiomas admitidos.
* Los planes de tarifa de S0 a S4 se retiran el 8 de marzo de 2021. Si tiene un recurso de Text Analytics existente que usa el plan de tarifa S0 a S4, debe actualizarlo para que use el [plan de tarifa](how-tos/text-analytics-how-to-call-api.md#change-your-pricing-tier) Estándar.
* El [contenedor de Detección de idioma](how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment) tiene ya disponibilidad general.
* Se está retirando la versión 2.1 de la API. 

## <a name="january-2021"></a>Enero de 2021

* La versión del modelo de `2021-01-15` para [Reconocimiento de entidades con nombre](how-tos/text-analytics-how-to-entity-linking.md) v3.x, que proporciona: 
  * Compatibilidad de lenguaje expandida para [varias categorías de entidad generales](named-entity-types.md). 
  * Calidad de IA mejorada de las categorías de entidades generales para todos los lenguajes de v3 admitidos. 

* La versión del modelo de `2021-01-05` para [detección de idioma](how-tos/text-analytics-how-to-language-detection.md), que proporciona [compatibilidad con el idioma](language-support.md?tabs=language-detection) adicional.

Estas versiones del modelo no están disponibles actualmente en la región Este de EE. UU. 

> [!div class="nextstepaction"]
> [Más información sobre el nuevo modelo de NER](https://azure.microsoft.com/updates/text-analytics-ner-improved-ai-quality)

## <a name="december-2020"></a>Diciembre de 2020

* Se han [actualizado los precios](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) de Text Analytics API.

## <a name="november-2020"></a>Noviembre de 2020

* Un 4[nuevo punto de conexión](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Analyze) con API Text Analytics v3.1-preview.3 para la nueva [Analyze API](how-tos/text-analytics-how-to-call-api.md?tabs=analyze) asincrónica, que admite el procesamiento por lotes para NER, PII y operaciones de extracción de frases clave.
* Un [nuevo punto de conexión](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Health) con Text Analytics API v 3.1-preview. 3 para la nueva API hospedada de [Text Analytics for Health](how-tos/text-analytics-for-health.md) asincrónica compatible con el procesamiento por lotes.
* Las dos características nuevas indicadas anteriormente solo está disponible en las siguientes regiones: `West US 2`, `East US 2`, `Central US`, `North Europe` y `West Europe`.
* El idioma Portugués (Brasil) `pt-BR`ahora se admite en [Análisis de sentimiento](how-tos/text-analytics-how-to-sentiment-analysis.md) v3.x, a partir de la versión del modelo`2020-04-01`. Se agrega a la compatibilidad `pt-PT` existente con Portugués.
* Bibliotecas cliente actualizadas, que incluye operaciones asíncrónicas de Analyze y Text Analytics for Health. Puede encontrar ejemplos en GitHub:

    * [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
    * [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
    * [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)
    * 
> [!div class="nextstepaction"]
> [Más información sobre Text Analytics API v3.1-Preview.3](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Languages)

## <a name="october-2020"></a>Octubre de 2020

* Compatibilidad del hindi para Análisis de sentimiento v3.x, a partir de la versión del modelo `2020-04-01`. 
* Versión de modelo `2020-09-01` para el punto de conexión /languages v3, que agrega mejoras en la precisión y detección de más idiomas.
* Disponibilidad de v3 en Centro de la India y Norte de Emiratos Árabes Unidos.

## <a name="september-2020"></a>Septiembre de 2020

### <a name="general-api-updates"></a>Actualizaciones de API generales

* Publicación de una nueva dirección URL para la versión preliminar pública de Text Analytics v3.1 para admitir las actualizaciones de los siguientes puntos de conexión de Reconocimiento de entidades con nombre v3: 
    * El punto de conexión `/pii` ahora incluye la nueva propiedad `redactedText` en el archivo JSON de respuesta, donde las entidades PII detectadas en el texto de entrada se reemplazan por un `*` por cada carácter de esas entidades.
    * El punto de conexión `/linking` ahora incluye la propiedad `bingID` en el archivo JSON de respuesta para las entidades vinculadas.
* Los siguientes puntos de conexión de Text Analytics API en versión preliminar se retiraron el 4 de septiembre de 2020:
    * v2.1-preview
    * v3.0-preview
    * v3.0-preview.1
    
> [!div class="nextstepaction"]
> [Más información sobre de Text Analytics API v3.1-preview.2](quickstarts/client-libraries-rest-api.md)

### <a name="text-analytics-for-health-container-updates"></a>Actualizaciones de Text Analytics para el contenedor de estado

Las siguientes actualizaciones son específicas de la versión de septiembre de Text Analytics para el contenedor de estado únicamente.
* Se ha publicado una nueva imagen de contenedor con la etiqueta `1.1.013530001-amd64-preview` con la nueva versión de modelo `2020-09-03` en el repositorio de vista previa de contenedores. 
* Esta versión del modelo proporciona mejoras en el reconocimiento de entidades, detección de abreviaturas y mejoras de latencia.

> [!div class="nextstepaction"]
> [Más información sobre Text Analytics para el estado](how-tos/text-analytics-for-health.md)

## <a name="august-2020"></a>Agosto de 2020

### <a name="general-api-updates"></a>Actualizaciones de API generales

* Versión del modelo `2020-07-01` para los puntos de conexión `/keyphrases`, `/pii` y `/languages` v3, que agrega:
    * [Categorías de entidades](named-entity-types.md?tabs=personal) gubernamentales y nacionales adicionales para el reconocimiento de entidades con nombre.
    * Compatibilidad con noruego y turco en Análisis de sentimiento v3.
* Ahora se devolverá un error HTTP 400 para las solicitudes de API v3 que superen los [límites de datos](concepts/data-limits.md) publicados. 
* Los puntos de conexión que devuelven un desplazamiento ya admiten el parámetro opcional `stringIndexType`, que ajusta los valores devueltos `offset` y `length` para que coincidan con un [esquema de índice de cadena](concepts/text-offsets.md) compatible.

### <a name="text-analytics-for-health-container-updates"></a>Actualizaciones de Text Analytics para el contenedor de estado

Las siguientes actualizaciones son específicas de la versión de agosto de Text Analytics para el contenedor de estado únicamente.

* Nueva versión de modelo de Text Analytics para estado: `2020-07-24`
* Nueva dirección URL para enviar solicitudes de estado de Text Analytics: `http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health` (Tenga en cuenta que es necesario borrar la caché del explorador para poder usar la aplicación web de demostración incluida en esta nueva imagen de contenedor)

Se han cambiado las siguientes propiedades en la respuesta JSON:

* El nombre de `type` ha cambiado a `category` 
* El nombre de `score` ha cambiado a `confidenceScore`
* Las entidades del campo `category` de la salida JSON están ahora en formato Pascal. Se ha cambiado el nombre de las siguientes entidades:
    * Se ha cambiado el nombre de `EXAMINATION_RELATION` a `RelationalOperator`.
    * Se ha cambiado el nombre de `EXAMINATION_UNIT` a `MeasurementUnit`.
    * Se ha cambiado el nombre de `EXAMINATION_VALUE` a `MeasurementValue`.
    * Se ha cambiado el nombre de `ROUTE_OR_MODE` a `MedicationRoute`.
    * Se ha cambiado el nombre de la entidad relacional `ROUTE_OR_MODE_OF_MEDICATION` a `RouteOfMedication`.

Se han agregado las entidades siguientes:

* NER
    * `AdministrativeEvent`
    * `CareEnvironment`
    * `HealthcareProfession`
    * `MedicationForm` 

* Extracción de relaciones
    * `DirectionOfCondition`
    * `DirectionOfExamination`
    * `DirectionOfTreatment`

> [!div class="nextstepaction"]
> [Más información sobre la versión preliminar de Text Analytics para el contenedor de estado](how-tos/text-analytics-for-health.md)

## <a name="july-2020"></a>Julio de 2020 

### <a name="text-analytics-for-health-container---public-gated-preview"></a>Text Analytics para la versión preliminar pública de contenedor de estado

Text Analytics para el contenedor de estado está ahora en versión preliminar pública, lo que le permite extraer información de texto en inglés no estructurado en documentos clínicos como los formularios de ingesta de pacientes, las notas del doctor, los documentos de investigación y los resúmenes de descarga. Actualmente, no se le facturará el uso de Text Analytics para el contenedor de estado.

El contenedor ofrece las siguientes características:

* Reconocimiento de entidades con nombre
* Extracción de relaciones
* Vinculación de entidad
* Negación

## <a name="may-2020"></a>Mayo de 2020

### <a name="text-analytics-api-v3-general-availability"></a>Disponibilidad general de Text Analytics API v3

Text Analytics API v3 ahora está disponible con carácter general con las siguientes actualizaciones:

* Versión del modelo `2020-04-01`
* Nuevos [límites de datos](concepts/data-limits.md) para cada característica
* [Compatibilidad de idiomas](language-support.md) actualizada para [Análisis de sentimiento (SA) v3](how-tos/text-analytics-how-to-sentiment-analysis.md)
* Punto de conexión independiente para la vinculación de entidad 
* Nueva categoría de entidad "Dirección" en [Reconocimiento de entidades con nombre (NER) v3](how-tos/text-analytics-how-to-entity-linking.md).
* Nuevas subcategorías en NER v3:
   * Ubicación: geográfica
   * Ubicación: estructural
   * Organización: bolsa
   * Organización: médica
   * Organización: deportes
   * Evento: cultural
   * Evento: natural
   * Evento: deportes

Se han agregado las siguientes propiedades en la respuesta JSON:
   * `SentenceText` en Análisis de sentimiento
   * `Warnings` para cada documento 

Se han cambiado los nombres de las siguientes propiedades en la respuesta JSON, donde procedía:

* El nombre de `score` ha cambiado a `confidenceScore`
    * `confidenceScore` tiene dos espacios decimales de precisión. 
* El nombre de `type` ha cambiado a `category`
* El nombre de `subtype` ha cambiado a `subcategory`

> [!div class="nextstepaction"]
> [Más información sobre Text Analytics API v3](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages)

### <a name="text-analytics-api-v31-public-preview"></a>Versión preliminar pública de Text Analytics API v3.1
   * Nueva característica de Análisis de sentimiento: [minería de opiniones](how-tos/text-analytics-how-to-sentiment-analysis.md#opinion-mining)
   * Nuevo filtro de dominio personal (`PII`) para obtener información de salud protegida (`PHI`).

> [!div class="nextstepaction"]
> [Más información sobre la versión preliminar de Text Analytics API v3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/Languages)

## <a name="february-2020"></a>Febrero de 2020

### <a name="sdk-support-for-text-analytics-api-v3-public-preview"></a>Compatibilidad del SDK con la versión preliminar pública de Text Analytics API v3

Como parte de la [versión unificada de Azure SDK](https://techcommunity.microsoft.com/t5/azure-sdk/january-2020-unified-azure-sdk-release/ba-p/1097290), el SDK de Text Analytics API v3 ahora está disponible como versión preliminar pública para los siguientes lenguajes de programación:
   * [C#](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-csharp&tabs=version-3)
   * [Python](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-python&tabs=version-3)
   * [JavaScript (Node.js)](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-javascript&tabs=version-3)
   * [Java](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-java&tabs=version-3)
   
> [!div class="nextstepaction"]
> [Más información sobre el SDK de Text Analytics API v3](./quickstarts/client-libraries-rest-api.md?tabs=version-3)

### <a name="named-entity-recognition-v3-public-preview"></a>Versión preliminar pública de reconocimiento de entidades con nombre v3

Los tipos de entidad adicionales ahora están disponibles en el servicio de versión preliminar pública de Reconocimiento de entidades con nombre (NER) v3 a medida que se expande la detección de entidades de información general y personal detectada en el texto. Esta actualización presenta [versión de modelo](concepts/model-versioning.md) `2020-02-01`, que incluye:

* Reconocimiento de los siguientes tipos de entidad generales (solo inglés):
    * PersonType
    * Producto
    * Evento
    * Entidad geopolítica (GPE) como subtipo de Ubicación
    * Habilidad

* Reconocimiento de los tipos de entidad de información personal siguientes (solo inglés):
    * Person
    * Organización
    * Edad como subtipo de Cantidad
    * Fecha como subtipo de DateTime
    * Email 
    * Número de teléfono (solo EE. UU.)
    * URL
    * Dirección IP

### <a name="october-2019"></a>Octubre de 2019

#### <a name="named-entity-recognition-ner"></a>Reconocimiento de entidades con nombre (NER)

* Un [punto de conexión nuevo](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesRecognitionPii) para reconocer los tipos de entidad de información personal (solo en inglés).

* Puntos de conexión independientes para el [reconocimiento de entidades](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesRecognitionGeneral) y la [vinculación de entidades](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesLinking).

* [Versión de modelo](concepts/model-versioning.md) `2019-10-01`, que incluye:
    * Detección expandida y categorización de entidades detectadas en el texto. 
    * Reconocimiento de los siguientes tipos de entidad nuevos:
        * Número de teléfono
        * Dirección IP

La vinculación de entidades admite el inglés y el español. La compatibilidad de idiomas de NER varía según el tipo de entidad.

#### <a name="sentiment-analysis-v3-public-preview"></a>Versión preliminar pública de la versión 3 de Análisis de sentimiento

* Un [nuevo punto de conexión](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/Sentiment) para analizar la opinión.
* [Versión de modelo](concepts/model-versioning.md) `2019-10-01`, que incluye:

    * Importantes mejoras respecto a la precisión y el detalle de la categorización del texto y la puntuación de la API.
    * Etiquetado automático para los distintos sentimientos en el texto.
    * Análisis de sentimiento y salida a nivel de documento y oración. 

Admite inglés (`en`), japonés (`ja`), chino simplificado (`zh-Hans`), chino tradicional (`zh-Hant`), francés (`fr`), italiano (`it`), español (`es`), neerlandés (`nl`), portugués (`pt`) y alemán (`de`) y está disponible en las siguientes regiones: `Australia East`, `Central Canada`, `Central US`, `East Asia`, `East US`, `East US 2`, `North Europe`, `Southeast Asia`, `South Central US`, `UK South`, `West Europe` y `West US 2`. 

> [!div class="nextstepaction"]
> [Más información sobre Análisis de sentimiento v3](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features)

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Text Analytics API?](overview.md)  
* [Ejemplos de escenarios de usuario](text-analytics-user-scenarios.md)
* [Análisis de opiniones](how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Detección de idioma](how-tos/text-analytics-how-to-language-detection.md)
* [Reconocimiento de entidades](how-tos/text-analytics-how-to-entity-linking.md)
* [Extracción de frases clave](how-tos/text-analytics-how-to-keyword-extraction.md)
