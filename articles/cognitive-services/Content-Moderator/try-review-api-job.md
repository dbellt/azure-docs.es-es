---
title: 'Uso de trabajos de moderación de contenido con la consola de API REST: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Use las operaciones de trabajo de la API de revisión para iniciar trabajos de moderación de contenido de un extremo a otro para el contenido de imágenes o texto en Azure Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.openlocfilehash: 62a6cd9e6302e66277b67ca7d37dd1ef33f96f1a
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031698"
---
# <a name="define-and-use-moderation-jobs-api-console"></a>Definición y uso de trabajos de moderación (consola de API)

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Un trabajo de moderación actúa como un tipo de contenedor para la funcionalidad de moderación de contenido, flujos de trabajo y revisiones. Esta guía muestra cómo usar las API REST de trabajos para iniciar y comprobar trabajos de moderación de contenido. Una vez que comprenda la estructura de las API, podrá migrar fácilmente estas llamadas a cualquier plataforma compatible con REST.

## <a name="prerequisites"></a>Prerrequisitos

- Inicie sesión o cree una cuenta en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.
- (Opcional) [Defina un flujo de trabajo personalizado](./Review-Tool-User-Guide/Workflows.md) para utilizarlo con el trabajo; también puede usar el flujo de trabajo predeterminado.

## <a name="create-a-job"></a>Creación de un trabajo

Para crear un trabajo de moderación, vaya a la página de referencia de API [Trabajo: crear](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5) y seleccione el botón correspondiente a su región de suscripción. Puede encontrar la región en la dirección URL de punto de conexión en la página **Credenciales** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/). Esto inicia la consola de API, donde puede crear y ejecutar llamadas API REST con facilidad.

![Selección de la región en la página Job - Create (Trabajo: crear)](images/test-drive-job-1.png)

### <a name="enter-rest-call-parameters"></a>Especificación de parámetros de llamada de REST

Escriba los valores siguientes para construir la llamada REST:

- **teamName**: identificador de equipo que creó al configurar la cuenta de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) (se encuentra en el campo **Id.** en la pantalla de credenciales de la herramienta de revisión).
- **ContentType**: puede ser "Imagen", "Texto" o "Vídeo".
- **ContentId**: cadena de identificador personalizada. Esta cadena se pasa a la API y se devuelve a través de la devolución de llamada. Es útil para asociar los identificadores internos o metadatos con los resultados de un trabajo de moderación.
- **Workflowname**: nombre del flujo de trabajo que creó anteriormente (o "predeterminado" para el flujo de trabajo predeterminado).
- **CallbackEndpoint**: (opcional) dirección URL para recibir información de devolución de llamada cuando la revisión se completa.
- **Ocp-Apim-Subscription-Key**: su clave de Content Moderator. Puede encontrarla en la pestaña **Configuración** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com).

### <a name="fill-in-the-request-body"></a>Cumplimentación del cuerpo de la solicitud

El cuerpo de la llamada REST contiene un solo campo, **ContentValue**. Pegue el contenido de texto sin formato si modera texto o especifique una dirección URL de imagen o vídeo si modera imagen o vídeo. Puede utilizar al siguiente dirección URL de imagen de ejemplo: [https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg)

![Parámetros de consulta de la consola Job - Create (Trabajo: crear), encabezados y cuadro Cuerpo de la solicitud](images/job-api-console-inputs.PNG)

### <a name="submit-your-request"></a>Enviar la solicitud

Seleccione **Enviar**. Si la operación se realiza correctamente, el valor de **Estado de respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra un identificador para el trabajo. Copie esta id. para usarla en los pasos siguientes.

![Cuadro de contenido de la repuesta de la consola Review - Create (Revisar: crear) muestra la id. de revisión](images/test-drive-job-3.PNG)

## <a name="get-job-status"></a>Obtención de estado del trabajo

Para obtener el estado y los detalles de un trabajo en ejecución o completado, vaya a la página de referencia de API [Job - Get](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c3) (Trabajo: obtener) y seleccione el botón correspondiente a su región (la región en la que se administra su clave).

![Selección de región para Job - Get (Trabajo: obtener)](images/test-drive-region.png)

Escriba los parámetros de llamada de REST como en la sección anterior. Para este paso, **JobId** es la cadena de identificador único que recibió cuando creó el trabajo. Seleccione **Enviar**. Si la operación se realiza correctamente, el valor de **Estado de respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra el trabajo en formato JSON, como el siguiente:

```json
{  
  "Id":"2018014caceddebfe9446fab29056fd8d31ffe",
  "TeamName":"some team name",
  "Status":"Complete",
  "WorkflowId":"OCR",
  "Type":"Image",
  "CallBackEndpoint":"",
  "ReviewId":"201801i28fc0f7cbf424447846e509af853ea54",
  "ResultMetaData":[  
    {  
      "Key":"hasText",
      "Value":"True"
    },
    {  
      "Key":"ocrText",
      "Value":"IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n"
    }
  ],
  "JobExecutionReport":[  
    {  
      "Ts":"2018-01-07T00:38:29.3238715",
      "Msg":"Posted results to the Callbackendpoint: https://requestb.in/vxke1mvx"
    },
    {  
      "Ts":"2018-01-07T00:38:29.2928416",
      "Msg":"Job marked completed and job content has been removed"
    },
    {  
      "Ts":"2018-01-07T00:38:29.0856472",
      "Msg":"Execution Complete"
    },
    {  
      "Ts":"2018-01-07T00:38:26.7714671",
      "Msg":"Successfully got hasText response from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:26.4181346",
      "Msg":"Getting hasText from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:25.5122828",
      "Msg":"Starting Execution - Try 1"
    }
  ]
}
```

![Respuesta de llamada REST para Job - Get (Trabajo: obtener)](images/test-drive-job-5.png)

### <a name="examine-the-new-reviews"></a>Examen de las nuevas revisiones

Si el trabajo de contenido dio como resultado la creación de una revisión, puede verla en el [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com). Seleccione **Revisar** > **Imagen**/**Texto**/**Vídeo** (en función del contenido que haya usado). Debe aparecer el contenido, listo para revisión humana. Después de que un moderador humano revise las etiquetas asignadas automáticamente y los datos de predicción y envíe una decisión de moderación final, la API de trabajos envía toda esta información al punto de conexión de devolución de llamada designado.

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a crear y consultar trabajos de moderación de contenido con la API REST. A continuación, integre los trabajos en un escenario de moderación de un extremo a otro, como el tutorial de [moderación de comercio electrónico](./ecommerce-retail-catalog-moderation.md).