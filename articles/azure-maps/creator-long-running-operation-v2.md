---
title: API V2 de operaciones de larga duración de Azure Maps
description: Aprenda sobre el procesamiento en segundo plano asincrónico de larga duración V2 en Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 05/18/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 454c34ce5cf45c293e62464dc67b8bb5576752be
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110192361"
---
# <a name="creator-long-running-operation-api-v2"></a>API V2 de operaciones de larga duración de Creator

Algunas API de Azure Maps usan un [patrón de respuesta a solicitudes asincrónico](/azure/architecture/patterns/async-request-reply). Este patrón permite que Azure Maps proporcione servicios de gran disponibilidad y capacidad de respuesta. En este artículo se explica la implementación específica de Azure Maps de la operación de procesamiento en segundo plano asincrónico de larga duración.

## <a name="submit-a-request"></a>Envío de una solicitud

Una aplicación de cliente inicia una operación de larga duración a través de una llamada sincrónica a una API de HTTP. Normalmente, esta llamada tiene el formato de una solicitud HTTP de POST. Cuando una carga de trabajo asincrónica se crea correctamente, la API devolverá un código de estado HTTP `202`, que indica que se ha aceptado la solicitud. Esta respuesta contiene un encabezado `Location` que apunta a un punto de conexión que el cliente puede sondear para comprobar el estado de la operación de larga duración.

### <a name="example-of-a-success-response"></a>Ejemplo de una respuesta correcta

```HTTP
Status: 202 Accepted
Operation-Location: https://atlas.microsoft.com/service/operations/{operationId} 

```

Si la llamada no pasa la validación, la API devolverá en su lugar una respuesta HTTP `400` por una solicitud incorrecta. Asimismo, el cuerpo de la respuesta proporcionará al cliente más información sobre por qué la solicitud no era válida.

### <a name="monitor-the-operation-status"></a>Supervisión del estado de la operación

Se puede sondear el punto de conexión de ubicación proporcionado en los encabezados de respuesta aceptados para comprobar el estado de la operación de larga duración. El cuerpo de respuesta de la solicitud de estado de la operación siempre contendrá las propiedades `status` y `created`. La propiedad `status` muestra el estado actual de la operación de larga duración. Los posibles estados incluyen `"NotStarted"`, `"Running"`, `"Succeeded"` y `"Failed"`. La propiedad `created` muestra la hora en que se realizó la solicitud inicial para comenzar la operación de ejecución de larga duración. Cuando el estado es `"NotStarted"` o `"Running"`, también se proporcionará un encabezado `Retry-After` con la respuesta. El encabezado `Retry-After`, el cual se mide en segundos, se puede usar para determinar cuándo se debe realizar la siguiente llamada de sondeo a la API de estado de la operación.

### <a name="example-of-running-a-status-response"></a>Ejemplo de ejecución de una respuesta de estado

```HTTP
Status: 200 OK
Retry-After: 30
{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "created": "3/11/2020 8:45:13 PM +00:00",
    "status": "Running"
}
```

## <a name="handle-operation-completion"></a>Control de la integridad de la operación

Al completar la operación de larga duración, el estado de la respuesta será `"Succeeded"` o `"Failed"`. Todas las respuestas devolverán un código HTTP 200 Correcto. Cuando se ha creado un recurso a partir de una operación de larga duración, la respuesta también contendrá un encabezado `Resource-Location` que apunta a los metadatos sobre el recurso. Tras un error, la respuesta tendrá una propiedad `error` en el cuerpo. Los datos de error se adhieren a la especificación de un error de OData.

### <a name="example-of-success-response"></a>Ejemplo de una respuesta correcta

```HTTP
Status: 200 OK
Resource-Location: "https://atlas.microsoft.com/tileset/{tileset-id}"
 {
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "created": "2021-05-06T07:55:19.5256829+00:00",
    "status": "Succeeded"
}
```

### <a name="example-of-failure-response"></a>Ejemplo de respuesta de error

```HTTP
Status: 200 OK

{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "created": "3/11/2020 8:45:13 PM +00:00",
    "status": "Failed",
    "error": {
        "code": "InvalidFeature",
        "message": "The provided feature is invalid.",
        "details": {
            "code": "NoGeometry",
            "message": "No geometry was provided with the feature."
        }
    }
}
```