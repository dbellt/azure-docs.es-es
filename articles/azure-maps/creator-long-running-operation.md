---
title: API de operaciones de larga duración de Azure Maps
description: Obtenga información sobre el procesamiento en segundo plano asincrónico de larga duración en Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 956c1969afa4f25fa4f431cca31caa410607777f
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110787152"
---
# <a name="creator-long-running-operation-api"></a>API de operaciones de larga duración de Creator

Algunas API de Azure Maps usan un [patrón de respuesta a solicitudes asincrónico](/azure/architecture/patterns/async-request-reply). Este patrón permite que Azure Maps proporcione servicios de gran disponibilidad y capacidad de respuesta. En este artículo se explica la implementación específica de Azure Maps de la operación de procesamiento en segundo plano asincrónico de larga duración.

## <a name="submitting-a-request"></a>Envío de una solicitud

Una aplicación de cliente inicia una operación de larga duración a través de una llamada sincrónica a una API de HTTP. Normalmente, esta llamada tiene el formato de una solicitud HTTP de POST. Cuando una carga de trabajo asincrónica se crea correctamente, la API devolverá un código de estado HTTP `202`, que indica que se ha aceptado la solicitud. Esta respuesta contiene un encabezado `Location` que apunta a un punto de conexión que el cliente puede sondear para comprobar el estado de la operación de larga duración.

### <a name="example-of-a-success-response"></a>Ejemplo de una respuesta correcta

```HTTP
Status: 202 Accepted
Location: https://atlas.microsoft.com/service/operations/{operationId}

```

Si la llamada no pasa la validación, la API devolverá en su lugar una respuesta HTTP `400` por una solicitud incorrecta. Asimismo, el cuerpo de la respuesta proporcionará al cliente más información sobre por qué la solicitud no era válida.

### <a name="monitoring-the-operation-status"></a>Supervisión del estado de la operación

Se puede sondear el punto de conexión de ubicación proporcionado en los encabezados de respuesta aceptados para comprobar el estado de la operación de larga duración. El cuerpo de respuesta de la solicitud de estado de la operación siempre contendrá las propiedades `status` y `createdDateTime`. La propiedad `status` muestra el estado actual de la operación de larga duración. Los posibles estados incluyen `"NotStarted"`, `"Running"`, `"Succeeded"` y `"Failed"`. La propiedad `createdDateTime` muestra la hora en que se realizó la solicitud inicial para comenzar la operación de ejecución de larga duración. Cuando el estado es `"NotStarted"` o `"Running"`, también se proporcionará un encabezado `Retry-After` con la respuesta. El encabezado `Retry-After`, el cual se mide en segundos, se puede usar para determinar cuándo se debe realizar la siguiente llamada de sondeo a la API de estado de la operación.

### <a name="example-of-running-a-status-response"></a>Ejemplo de ejecución de una respuesta de estado

```HTTP
Status: 200 OK
Retry-After: 30
{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Running"
}
```

## <a name="handling-operation-completion"></a>Gestión de la final de la operación

Al completar la operación de larga duración, el estado de la respuesta será `"Succeeded"` o `"Failed"`. Cuando se crea un recurso nuevo a partir de una operación de larga duración, la respuesta correcta devolverá un código de estado HTTP `201 Created`. La respuesta también contendrá un encabezado `Location` que apunta a los metadatos del recurso. Si no se ha creado ningún recurso nuevo, la respuesta correcta devolverá un código de Estado HTTP `200 OK`. Igualmente, cuando se produce un error, el código de estado de la respuesta también será el código `200 OK`. Sin embargo, la respuesta tendrá una propiedad `error` en el cuerpo. Los datos de error se adhieren a la especificación de un error de OData.

### <a name="example-of-success-response"></a>Ejemplo de una respuesta correcta

```HTTP
Status: 201 Created
Location: "https://atlas.microsoft.com/tileset/{tileset-id}"
 {
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Succeeded",
    "resourceLocation": "https://atlas.microsoft.com/tileset/{tileset-id}"
}
```

### <a name="example-of-failure-response"></a>Ejemplo de respuesta de error

```HTTP
Status: 200 OK

{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
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