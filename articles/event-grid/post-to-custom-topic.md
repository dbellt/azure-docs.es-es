---
title: Publicación de eventos en un tema personalizado de Azure Event Grid
description: En este artículo se describe cómo publicar un evento en un tema personalizado. Muestra el formato de los datos de publicación y eventos.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b662cd2034fc1035a5da64c19d9aba87b180af8f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701146"
---
# <a name="post-to-custom-topic-for-azure-event-grid"></a>Publicación en un tema personalizado de Azure Event Grid

En este artículo se describe cómo publicar un evento en un tema personalizado. Muestra el formato de los datos de publicación y eventos. El [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/event-grid/v1_0/) solo se aplica a las publicaciones que coinciden con el formato esperado.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="endpoint"></a>Punto de conexión

Al enviar el método HTTP POST a un tema personalizado, use el formato del URI: `https://<topic-endpoint>?api-version=2018-01-01`.

Por ejemplo, un URI válido es: `https://exampletopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01`.

Para obtener el punto de conexión de un tema personalizado con la CLI de Azure, use:

```azurecli-interactive
az eventgrid topic show --name <topic-name> -g <topic-resource-group> --query "endpoint"
```

Para obtener el punto de conexión de un tema personalizado con Azure PowerShell, use:

```powershell
(Get-AzEventGridTopic -ResourceGroupName <topic-resource-group> -Name <topic-name>).Endpoint
```

## <a name="header"></a>Encabezado

En la solicitud, incluya un valor de encabezado denominado `aeg-sas-key` que contenga una clave para la autenticación.

Por ejemplo, un valor de encabezado válido es `aeg-sas-key: VXbGWce53249Mt8wuotr0GPmyJ/nDT4hgdEj9DpBeRr38arnnm5OFg==`.

Para obtener la clave de un tema personalizado con la CLI de Azure, use:

```azurecli
az eventgrid topic key list --name <topic-name> -g <topic-resource-group> --query "key1"
```

Para obtener la clave de un tema personalizado con PowerShell, use:

```powershell
(Get-AzEventGridTopicKey -ResourceGroupName <topic-resource-group> -Name <topic-name>).Key1
```

## <a name="event-data"></a>Datos de evento

Para los temas personalizados, los datos de nivel superior contienen los mismos campos que los eventos estándar definidos por los recursos. Una de esas propiedades es una propiedad de datos que contiene propiedades únicas del tema personalizado. Como publicador de eventos, debe determinar las propiedades del objeto de datos. Use el esquema siguiente:

```json
[
  {
    "id": string,    
    "eventType": string,
    "subject": string,
    "eventTime": string-in-date-time-format,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string
  }
]
```

Para obtener una descripción de estas propiedades, vea [Esquema de eventos de Azure Event Grid](event-schema.md). Al publicar eventos en un tema de Event Grid, la matriz puede tener un tamaño total de hasta 1 MB. El tamaño máximo permitido para un evento es también de 1 MB. Los eventos de más de 64 KB se cobran en incrementos de 64 KB. 

Por ejemplo, un esquema de datos de evento válido es:

```json
[{
  "id": "1807",
  "eventType": "recordInserted",
  "subject": "myapp/vehicles/motorcycles",
  "eventTime": "2017-08-10T21:03:07+00:00",
  "data": {
    "make": "Ducati",
    "model": "Monster"
  },
  "dataVersion": "1.0"
}]
```

## <a name="response"></a>Response

Después de publicar en el punto de conexión del tema, recibirá una respuesta. La respuesta es un código de respuesta HTTP estándar. Algunas respuestas comunes son:

|Resultado  |Response  |
|---------|---------|
|Correcto  | 200 OK  |
|Los datos del evento tienen un formato incorrecto | 400 - Solicitud incorrecta |
|Clave de acceso no válida | 401 No autorizado |
|Punto de conexión incorrecto | 404 No encontrado |
|La matriz o el evento superan los límites de tamaño | 413 Carga demasiado grande |

Si hay errores, el cuerpo del mensaje tiene el formato siguiente:

```json
{
    "error": {
        "code": "<HTTP status code>",
        "message": "<description>",
        "details": [{
            "code": "<HTTP status code>",
            "message": "<description>"
    }]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para más información sobre la clave de autenticación, vea [Seguridad y autenticación de Event Grid](security-authentication.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
