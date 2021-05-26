---
title: 'Esquemas de eventos: Azure Event Grid en IoT Edge | Microsoft Docs'
description: Esquemas de eventos en Event Grid en IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: de328fb70f8abc32a89b8d5b54b0cb40a12f9356
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370384"
---
# <a name="event-schemas"></a>Esquemas de eventos

El módulo Event Grid acepta y entrega eventos en formato JSON. Actualmente, hay tres esquemas compatibles con Event Grid:

* **EventGridSchema**
* **CustomSchema**
* **CloudEventSchema**

Puede configurar el esquema con el que debe cumplir un publicador durante la creación del tema. Si no se especifica, el valor predeterminado es **EventGridSchema**. Se rechazarán los eventos que no cumplan con el esquema esperado.

Los suscriptores también pueden configurar el esquema en el que quieren que se entreguen los eventos. Si no se especifica, el valor predeterminado es el esquema del tema.
Actualmente, el esquema de entrega del suscriptor debe coincidir con el esquema de entrada de su tema. 

## <a name="eventgrid-schema"></a>Esquema EventGrid

El esquema EventGrid consta de un conjunto de propiedades necesarias con la que debe cumplir una entidad de publicación. Cada publicador tiene que rellenar los campos de nivel superior.

```json
[
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
]
```

### <a name="eventgrid-schema-properties"></a>Propiedades del esquema EventGrid

Todos los eventos tienen los datos de nivel superior siguientes:

| Propiedad | Tipo | Obligatorio | Descripción |
| -------- | ---- | ----------- |-----------
| topic | string | No | Debe coincidir con el tema en el que se publica. Si no se especifica, Event Grid lo rellena con el nombre del tema en el que se publica. |
| subject | string | Sí | Ruta al asunto del evento definida por el anunciante. |
| eventType | string | Sí | El tipo de evento de este origen del evento, por ejemplo, BlobCreated. |
| eventTime | string | Sí | La hora de generación del evento en función de la hora UTC del proveedor. |
| id | string | No | Identificador único para el evento |
| datos | object | No | Se usa para capturar datos de eventos específicos para la entidad de publicación. |
| dataVersion | string | Sí | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| metadataVersion | string | No | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

### <a name="example--eventgrid-schema-event"></a>Ejemplo: evento del esquema EventGrid

```json
[
  {
    "id": "1807",
    "eventType": "recordInserted",
    "subject": "myapp/vehicles/motorcycles",
    "eventTime": "2017-08-10T21:03:07+00:00",
    "data": {
      "make": "Ducati",
      "model": "Monster"
    },
    "dataVersion": "1.0"
  }
]
```

## <a name="customevent-schema"></a>Esquema CustomEvent

En el esquema personalizado, no hay propiedades obligatorias que se apliquen como en el esquema EventGrid. La entidad de publicación puede controlar el esquema de eventos por completo. Proporciona la máxima flexibilidad y permite escenarios en los que ya se ha implementado un sistema basado en eventos y donde quisiera reutilizar los eventos existentes o no quisiera estar vinculado a un esquema específico.

### <a name="custom-schema-properties"></a>Propiedades del esquema personalizado

No hay propiedades obligatorias. Depende de la entidad de publicación determinar la carga útil.

### <a name="example--custom-schema-event"></a>Ejemplo: evento del esquema personalizado

```json
[
  {
    "eventdata": {
      "make": "Ducati",
      "model": "Monster"
    }
  }
]
```

## <a name="cloudevent-schema"></a>Esquema CloudEvent

Además de los esquemas anteriores, Event Grid admite de manera nativa eventos en el [esquema JSON CloudEvents](https://github.com/cloudevents/spec/blob/master/json-format.md). CloudEvents es una especificación abierta para describir datos de eventos. Simplifica la interoperabilidad al proporcionar un esquema de eventos común para la publicación y el consumo de eventos. Forma parte de [CNCF](https://www.cncf.io/) y la versión disponible actual es 1.0-rc1.

### <a name="cloudevent-schema-properties"></a>Propiedades del esquema CloudEvent

Consulte la [especificación de CloudEvents](https://github.com/cloudevents/spec/blob/master/json-format.md#3-envelope) sobre las propiedades de sobre obligatorias.

### <a name="example--cloud-event"></a>Ejemplo: evento en la nube
```json
[{
       "id": "1807",
       "type": "recordInserted",
       "source": "myapp/vehicles/motorcycles",
       "time": "2017-08-10T21:03:07+00:00",
       "datacontenttype": "application/json",
       "data": {
            "make": "Ducati",
            "model": "Monster"
        },
        "dataVersion": "1.0",
        "specVersion": "1.0-rc1"
}]
```
