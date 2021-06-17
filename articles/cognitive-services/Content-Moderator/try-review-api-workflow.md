---
title: 'Definición de flujos de trabajo de moderación con la consola de API de REST: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Puede usar las API de revisión de Azure Content Moderator para definir flujos de trabajo y umbrales personalizados en función de las directivas de contenido.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/14/2019
ms.author: pafarley
ms.openlocfilehash: 210ed6e3a73721a7b4b4508071dc98e0bfd27498
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031662"
---
# <a name="define-and-use-moderation-workflows-api-console"></a>Definición y uso de los flujos de trabajo de moderación (consola de API)

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Los flujos de trabajo son filtros personalizados basados en la nube que puede usar para controlar el contenido de forma más eficaz. Los flujos de trabajo pueden conectarse a diversos servicios para filtrar el contenido de maneras diferentes y luego realizar la acción apropiada. En esta guía se le muestra cómo usar las API de REST de flujo de trabajo, a través de la consola de API, para crear y usar flujos de trabajo. Una vez que comprenda la estructura de las API, podrá migrar fácilmente estas llamadas a cualquier plataforma compatible con REST.

## <a name="prerequisites"></a>Prerrequisitos

- Inicie sesión o cree una cuenta en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="create-a-workflow"></a>Creación de un flujo de trabajo

Para crear o actualizar un flujo de trabajo, vaya a la página de referencia de API **[Flujo de trabajo: Crear o actualizar](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59)** y seleccione el botón para su región clave. Puede encontrar la región en la dirección URL de punto de conexión en la página **Credenciales** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/). Esto inicia la consola de API, donde puede crear y ejecutar llamadas API REST con facilidad.

![Selección de región para Workflow - Create Or Update (Flujo de trabajo: Crear o actualizar)](images/test-drive-region.png)

### <a name="enter-rest-call-parameters"></a>Especificación de parámetros de llamada de REST

Especifique los valores para **team**, **workflowName** y **Ocp-Apim-Subscription-Key**:

- **team**: identificador de equipo que creó al configurar la cuenta de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) (se encuentra en el campo **Id.** en la pantalla de credenciales de la herramienta de revisión).
- **workflowname**: nombre de un nuevo flujo de trabajo que se va a agregar (o un nombre existente, si desea actualizar un flujo de trabajo existente).
- **Ocp-Apim-Subscription-Key**: su clave de Content Moderator. Puede encontrarla en la pestaña **Configuración** de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com).

![Parámetros y encabezados de consulta de la consola de Workflow - Create Or Update (Flujo de trabajo: Crear o actualizar)](images/workflow-console-parameters.PNG)

### <a name="enter-a-workflow-definition"></a>Especificación de una definición del flujo de trabajo

1. Edite el cuadro de diálogo **Cuerpo de la solicitud** para especificar la solicitud JSON con la información para **Descripción** y **Tipo** (`Image` o `Text`).
2. En **Expresión**, copie la expresión JSON del flujo de trabajo predeterminada. La cadena JSON final debe tener este aspecto:

```json
{
  "Description":"<A description for the Workflow>",
  "Type":"Text",
  "Expression":{
    "Type":"Logic",
    "If":{
      "ConnectorName":"moderator",
      "OutputName":"isAdult",
      "Operator":"eq",
      "Value":"true",
      "Type":"Condition"
    },
    "Then":{
      "Perform":[
        {
          "Name":"createreview",
          "CallbackEndpoint":null,
          "Tags":[

          ]
        }
      ],
      "Type":"Actions"
    }
  }
}
```

> [!NOTE]
> Puede definir expresiones simples, complejas e incluso anidadas para los flujos de trabajo mediante esta API. La documentación [Workflow - Create Or Update](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59) (Flujo de trabajo: Crear o actualizar) tiene ejemplos de lógica más compleja.

### <a name="submit-your-request"></a>Enviar la solicitud
  
Seleccione **Enviar**. Si la operación se realiza correctamente, el **estado de la respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra `true`.

### <a name="examine-the-new-workflow"></a>Examen del nuevo flujo de trabajo

En la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/), seleccione **Configuración** > **Flujos de trabajo**. Su nuevo flujo de trabajo debe aparecer en la lista.

![Lista de flujos de trabajo de la herramienta de revisión](images/workflow-console-new-workflow.PNG)

Seleccione la opción **Editar** del flujo de trabajo y vaya a la pestaña **Diseñador**. Aquí puede ver una representación intuitiva de la lógica JSON.

![Pestaña Diseñador para un flujo de trabajo seleccionado](images/workflow-console-new-workflow-designer.PNG)

## <a name="get-workflow-details"></a>Obtención de detalles del flujo de trabajo

Para recuperar detalles sobre un flujo de trabajo existente, vaya a la página de referencia de API **[Workflow - Get](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b44b3f9b0711b43c4c58)** (Flujo de trabajo: Obtener) y seleccione el botón para su región (la región en la que se administra su clave).

![Selección de región para Workflow - Get (Flujo de trabajo: Obtener)](images/test-drive-region.png)

Escriba los parámetros de llamada de REST como en la sección anterior. Asegúrese de que esta vez, **workflowname** es el nombre de un flujo de trabajo existente.

![Obtención de parámetros y encabezados de consulta](images/workflow-get-default.PNG)

Seleccione **Enviar**. Si la operación se realiza correctamente, el **estado de la respuesta** es `200 OK` y el cuadro **Contenido de la respuesta** muestra el flujo de trabajo en formato JSON, como el siguiente ejemplo:

```json
{
  "Name":"default",
  "Description":"Default",
  "Type":"Image",
  "Expression":{
    "If":{
      "ConnectorName":"moderator",
      "OutputName":"isadult",
      "Operator":"eq",
      "Value":"true",
      "AlternateInput":null,
      "Type":"Condition"
    },
    "Then":{
      "Perform":[
        {
          "Name":"createreview",
          "Subteam":null,
          "CallbackEndpoint":null,
          "Tags":[

          ]
        }
      ],
      "Type":"Actions"
    },
    "Else":null,
    "Type":"Logic"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a usar flujos de trabajo con [trabajos de moderación de contenido](try-review-api-job.md).