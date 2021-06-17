---
title: 'Creación de revisiones de moderación con la consola de API REST: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Use las API de revisión de Azure Content Moderator para crear revisiones de imagen o texto para la moderación humana.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: pafarley
ms.openlocfilehash: d697cf4c04753427d161196b3efb7e5e055e32ac
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112033732"
---
# <a name="create-human-reviews-api-console"></a>Creación de revisiones humanas (consola de API)

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Las [revisiones](./review-api.md#reviews) almacenan y muestran el contenido que habrán de evaluar moderadores humanos. Cuando un usuario completa una revisión, los resultados se envían a un punto de conexión de devolución de llamada especificado. En esta guía, aprenderá a configurar las revisiones mediante API REST de revisión a través de la consola de API. Una vez que comprenda la estructura de las API, podrá migrar fácilmente estas llamadas a cualquier plataforma compatible con REST.

## <a name="prerequisites"></a>Prerrequisitos

- Inicie sesión o cree una cuenta en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="create-a-review"></a>Creación de una revisión

Para crear una revisión, vaya a la página de referencia de API **[Review - Create](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4)** (Revisar: crear) y seleccione el botón para su región clave (lo encontrará en la dirección URL de punto de conexión en la página **Credenciales** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/)). Esto inicia la consola de API, donde puede crear y ejecutar llamadas API REST con facilidad.

![Selección de región para Revisar: obtener](images/test-drive-region.png)

### <a name="enter-rest-call-parameters"></a>Especificación de parámetros de llamada de REST

Especifique los valores para **teamName** y **Ocp-Apim-Subscription-Key**:

- **teamName**: identificador de equipo que creó al configurar la cuenta de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) (se encuentra en el campo **Id.** en la pantalla de credenciales de la herramienta de revisión).
- **Ocp-Apim-Subscription-Key**: su clave de Content Moderator. Puede encontrarla en la pestaña **Configuración** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com).

### <a name="enter-a-review-definition"></a>Especificación de una definición de revisión

Edite el cuadro **Cuerpo de la solicitud** para especificar la solicitud JSON con los siguientes campos:

- **Metadatos**: pares de clave-valor personalizados devueltos al punto de conexión de la devolución de llamada. Si la clave es un código corto definido en la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com), aparece como una etiqueta.
- **Content**: en el caso de contenido de imagen y vídeo, se trata de una cadena de dirección URL que apunta al contenido. Para el contenido de texto, esta es la cadena de texto real.
- **ContentId**: cadena de identificador personalizada. Esta cadena se pasa a la API y se devuelve a través de la devolución de llamada. Es útil para asociar los identificadores internos o metadatos con los resultados de un trabajo de moderación.
- **CallbackEndpoint**: (opcional) dirección URL para recibir información de devolución de llamada cuando la revisión se completa.

El cuerpo de la solicitud predeterminado muestra ejemplos de los diferentes tipos de revisiones que se puede crear:

```json
[Image]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Image",
    "Content": "<Content Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Text]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Text",
    "Content": "<Your Text Content>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Video]
[
  {
    "VideoFrames":[
      {
          "Id": "<Frame Id>",
          "Timestamp": "<Frame Timestamp",
          "FrameImage":"<Frame Image URL",
          "Metadata": [
            {
              "Key": "<Key>",
              "Value": "<Value"
            }
          ],
          "ReviewerResultTags": [
          ]
    ], 
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      },
      //For encrypted Videos
        {
          "Key": "protectedType",
          "Value": "AES or FairPlay or Widevine or Playready"
        },
        {
          "Key": "authenticationToken",
          "Value": "your viewtoken(In case of Video Indexer AES encryption type, this value is viewtoken from breakdown json)"
        },
      //For FairPlay encrypted type video include certificateUrl as well
        {
          "Key": "certificateUrl",
          "Value": "your certificate url"
        }
    ],
    "Type": "Video",
    "Content": "<Stream Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>",
    [Optional]
    "Timescale": "<Timescale of the video>
  }
]
```

### <a name="submit-your-request"></a>Enviar la solicitud
  
Seleccione **Enviar**. Si la operación se realiza correctamente, **Estado de respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra un identificador para la revisión. Copie esta id. para usarla en los pasos siguientes.

![Cuadro de contenido de la repuesta de la consola Review - Create (Revisar: crear) muestra la id. de revisión](images/test-drive-review-2.PNG)

### <a name="examine-the-new-review"></a>Examen de la nueva revisión

En el [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com), seleccione **Revisar** > **Imagen**/**Texto**/**Vídeo** (según el contenido que use). Debe aparecer el contenido que ha cargado, listo para revisión humana.

![Imagen de un balón de fútbol de la herramienta de revisión](images/test-drive-review-5.PNG)

## <a name="get-review-details"></a>Obtención de detalles de la revisión

Para recuperar detalles sobre una revisión existente, vaya a la página de referencia de la API [Review - Get](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2) (Revisar: obtener) y seleccione el botón para su región (la región en la que se administra su clave).

![Selección de región para Workflow - Get (Flujo de trabajo: Obtener)](images/test-drive-region.png)

Escriba los parámetros de llamada de REST como en la sección anterior. Para este paso, **reviewId** es la cadena de identificador único que recibió cuando creó la revisión.

![Resultados de Obtener de la consola Review - Create (Revisar: crear)](images/test-drive-review-3.PNG)
  
Seleccione **Enviar**. Si la operación se realiza correctamente, el **estado de la respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra los detalles de la revisión en formato JSON, como el siguiente:

```json
{  
  "reviewId":"201712i46950138c61a4740b118a43cac33f434",
  "subTeam":"public",
  "status":"Complete",
  "reviewerResultTags":[  
    {  
      "key":"a",
      "value":"False"
    },
    {  
      "key":"r",
      "value":"True"
    },
    {  
      "key":"sc",
      "value":"True"
    }
  ],
  "createdBy":"<teamname>",
  "metadata":[  
    {  
      "key":"sc",
      "value":"true"
    }
  ],
  "type":"Image",
  "content":"https://reviewcontentprod.blob.core.windows.net/<teamname>/IMG_201712i46950138c61a4740b118a43cac33f434",
  "contentId":"0",
  "callbackEndpoint":"<callbackUrl>"
}
```

Tome nota de los siguientes campos en la respuesta:

- **status**
- **reviewerResultTags**: esto aparece si el equipo de revisión humana ha agregado manualmente las etiquetas (se muestra en el campo **createdBy**).
- **metadata**: esto muestra las etiquetas que se agregaron inicialmente en la revisión, antes de que el equipo de revisión humana realizara cambios.

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a crear las revisiones de moderación de contenido mediante la API REST. A continuación, integre las revisiones en un escenario de moderación de un extremo a otro, como el tutorial de [moderación de comercio electrónico](./ecommerce-retail-catalog-moderation.md).