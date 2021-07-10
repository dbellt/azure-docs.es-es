---
title: Extracción de frases clave con Text Analytics REST API
titleSuffix: Azure Cognitive Services
description: Cómo extraer frases clave con Text Analytics REST API desde Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 03/29/2021
ms.author: aahi
ms.openlocfilehash: e25851bc9d3cba6e71056a6fb6a24ff9a0cee6ce
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968058"
---
# <a name="example-how-to-extract-key-phrases-using-text-analytics"></a>Ejemplo: Extracción de frases clave con Text Analytics

[Key Phrase Extraction API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) evalúa el texto no estructurado y, en cada documento JSON, devuelve una lista de frases clave.

Esta capacidad es útil si necesita identificar rápidamente los principales puntos en una colección de documentos. Por ejemplo, si el texto de entrada es "La comida estaba deliciosa y el personal era maravilloso", el servicio devuelve los principales puntos de conversación: "comida" y "personal maravilloso".

Para más información, consulte [Lenguajes admitidos](../language-support.md).

> [!TIP]
> * Text Analytics proporciona también una imagen de contenedor de Docker basada en Linux para la extracción de frases clave, por lo que puede [instalar y ejecutar el contenedor de Text Analytics](text-analytics-how-to-install-containers.md) cerca de los datos.
> * Esta característica también se puede usar de forma [asincrónica](text-analytics-how-to-call-api.md) mediante el punto de conexión `/analyze`.

## <a name="preparation"></a>Preparación

La extracción de frases clave funciona mejor cuando se proporcionan cantidades más grandes de texto en los que trabajar. Es lo contrario del análisis de opiniones, que funciona mejor con cantidades de texto más pequeñas. Para obtener los mejores resultados de ambas operaciones, considere la posibilidad de reestructurar las entradas en consecuencia.

Debe tener documentos JSON en este formato: identificador, texto, idioma

El tamaño del documento debe ser de 5120 caracteres o menos por documento y puede tener hasta 1000 elementos (identificadores) por colección. La colección se envía en el cuerpo de la solicitud. El ejemplo siguiente es una ilustración del contenido que podría enviar a la extracción de frases clave. 

Para obtener más información sobre los objetos de solicitud y respuesta, consulte [Cómo llamar a la API de Text Analytics](text-analytics-how-to-call-api.md).  

### <a name="example-synchronous-request-object"></a>Ejemplo de objeto de solicitud sincrónica


```json
    {
        "documents": [
            {
                "language": "en",
                "id": "1",
                "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!"
            },
            {
                "language": "en",
                "id": "2",
                "text": "Poorly marked trails! I thought we were goners. Worst hike ever."
            },
            {
                "language": "en",
                "id": "3",
                "text": "Everyone in my family liked the trail but thought it was too challenging for the less athletic among us. Not necessarily recommended for small children."
            },
            {
                "language": "en",
                "id": "4",
                "text": "It was foggy so we missed the spectacular views, but the trail was ok. Worth checking out if you are in the area."
            },
            {
                "language": "en",
                "id": "5",
                "text": "This is my favorite trail. It has beautiful views and many places to stop and rest"
            }
        ]
    }
```

### <a name="example-asynchronous-request-object"></a>Ejemplo de objeto de solicitud asincrónica

A partir de `v3.1-preview.3`, puede enviar solicitudes de NER de forma asincrónica mediante el punto de conexión `/analyze`.


```json
{
    "displayName": "My Job",
    "analysisInput": {
        "documents": [
            {
                "id": "doc1",
                "text": "It's incredibly sunny outside! I'm so happy"
            },
            {
                "id": "doc2",
                "text": "Pike place market is my favorite Seattle attraction."
            }
        ]
    },
    "tasks": {
        "keyPhraseExtractionTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }],
    }
}
```

## <a name="step-1-structure-the-request"></a>Paso 1: Estructurar la solicitud

Para más información sobre la definición de la solicitud, consulte [Cómo llamar a la API REST de Text Analytics](text-analytics-how-to-call-api.md). Recapitulamos los siguientes puntos para su comodidad:

+ Cree una solicitud **POST**. Revise la documentación de la API para esta solicitud: [Key Phrases API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases).

+ Establezca el punto de conexión de HTTP para la extracción de frases clave mediante un recurso de Text Analytics en Azure o un [contenedor de Text Analytics](text-analytics-how-to-install-containers.md) con instancias. Si utiliza la API sincrónicamente, debe incluir `/text/analytics/v3.0/keyPhrases` en la dirección URL. Por ejemplo: `https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`.

+ Establezca un encabezado de solicitud para incluir la [clave de acceso](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) para las operaciones de Text Analytics.

+ En el cuerpo de la solicitud, proporcione la colección de documentos JSON que preparó para este análisis.

> [!Tip]
> Use [Postman](text-analytics-how-to-call-api.md) o abra la **consola de prueba de la API** en la [documentación](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) para estructurar y enviar una solicitud POST al servicio.

## <a name="step-2-post-the-request"></a>Paso 2: Publicar la solicitud

El análisis se realiza tras la recepción de la solicitud. Para obtener información sobre el tamaño y el número de solicitudes que puede enviar por minuto o por segundo, consulte el artículo sobre [límites de datos](../concepts/data-limits.md).

Recuerde que el servicio no tiene estado. No se almacena ningún dato en su cuenta. Los resultados se devuelven inmediatamente en la respuesta.

## <a name="step-3-view-results"></a>Paso 3: Vista de resultados

Todas las solicitudes POST devolverán una respuesta con formato JSON con los identificadores y las propiedades detectadas. El orden de las frases clave devueltas lo determina internamente el modelo.

La salida se devuelve inmediatamente. Puede transmitir los resultados a una aplicación que acepte JSON o guardar la salida en un archivo en el sistema local y, a continuación, importarlo en una aplicación que permita ordenar, buscar y manipular los datos.

Aquí se muestra un ejemplo de la salida de la extracción de frases clave del punto de conexión de v3.1-preview:

### <a name="synchronous-result"></a>Resultado sincrónico

```json
    {
       "documents":[
          {
             "id":"1",
             "keyPhrases":[
                "year",
                "trail",
                "trip",
                "views",
                "hike"
             ],
             "warnings":[]
          },
          {
             "id":"2",
             "keyPhrases":[
                "marked trails",
                "Worst hike",
                "goners"
             ],
             "warnings":[]
          },
          {
             "id":"3",
             "keyPhrases":[
                "trail",
                "small children",
                "family"
             ],
             "warnings":[]
          },
          {
             "id":"4",
             "keyPhrases":[
                "spectacular views",
                "trail",
                "Worth",
                "area"
             ],
             "warnings":[]
          },
          {
             "id":"5",
             "keyPhrases":[
                "places",
                "beautiful views",
                "favorite trail",
                "rest"
             ],
             "warnings":[]
          }
       ],
       "errors":[],
       "modelVersion":"2020-07-01"
    }

```
Como se indicó, el analizador busca y descarta las palabras que no son esenciales y conserva solo los términos o frases que parecen ser el asunto o el objeto de una frase.

### <a name="asynchronous-result"></a>Resultado asincrónico

Si usa el punto de conexión `/analyze` en la operación asincrónica, recibirá una respuesta que contiene las tareas que envió a la API.

```json
{
  "displayName": "My Analyze Job",
  "jobId": "dbec96a8-ea22-4ad1-8c99-280b211eb59e_637408224000000000",
  "lastUpdateDateTime": "2020-11-13T04:01:14Z",
  "createdDateTime": "2020-11-13T04:01:13Z",
  "expirationDateTime": "2020-11-14T04:01:13Z",
  "status": "running",
  "errors": [],
  "tasks": {
      "details": {
          "name": "My Analyze Job",
          "lastUpdateDateTime": "2020-11-13T04:01:14Z"
      },
      "completed": 1,
      "failed": 0,
      "inProgress": 2,
      "total": 3,
      "keyPhraseExtractionTasks": [
          {
              "name": "My Analyze Job",
              "lastUpdateDateTime": "2020-11-13T04:01:14.3763516Z",
              "results": {
                  "inTerminalState": true,
                  "documents": [
                      {
                          "id": "doc1",
                          "keyPhrases": [
                              "sunny outside"
                          ],
                          "warnings": []
                      },
                      {
                          "id": "doc2",
                          "keyPhrases": [
                              "favorite Seattle attraction",
                              "Pike place market"
                          ],
                          "warnings": []
                      }
                  ],
                  "errors": [],
                  "modelVersion": "2020-07-01"
              }
          }
      ]
  }
}
```


## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo de la extracción de frases clave mediante Text Analytics de Cognitive Services. En resumen:

+ [Key phrase extraction API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) está disponible para los idiomas seleccionados.
+ Los documentos JSON del cuerpo de la solicitud incluyen un identificador, un texto y el código de idioma.
+ La solicitud POST se realiza a un punto de conexión `/keyphrases` o `/analyze`, con una [clave de acceso y un punto de conexión](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personalizados, que son válidos para la suscripción.
+ La salida de la respuesta, que consta de palabras clave y frases para cada id. de documento, se puede transmitir a cualquier aplicación que acepte JSON, incluidas Microsoft Office Excel y Power BI, por nombrar algunas.

## <a name="see-also"></a>Consulte también

 [Introducción a Text Analytics](../overview.md)[Preguntas más frecuentes (FAQ)](../text-analytics-resource-faq.md)</br>
 [Página del producto de Text Analytics](//go.microsoft.com/fwlink/?LinkID=759712)

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Text Analytics](../overview.md)
* [Uso de la biblioteca cliente de Text Analytics](../quickstarts/client-libraries-rest-api.md)
* [Novedades](../whats-new.md)
* [Versiones del modelo](../concepts/model-versioning.md)
