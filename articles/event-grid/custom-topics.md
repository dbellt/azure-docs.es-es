---
title: Temas personalizados en Azure Event Grid
description: Describe los temas personalizados en Azure Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: c211e529e5382fd655fca6d78ab32409cbff4453
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736653"
---
# <a name="custom-topics-in-azure-event-grid"></a>Temas personalizados en Azure Event Grid
En un tema de Event Grid se proporciona un punto de conexión al que el origen envía los eventos. El publicador crea el tema de Event Grid y decide si un origen de evento necesita un tema o más de un tema. Un tema se usa para una colección de eventos relacionados. Para responder a determinados tipos de eventos, los suscriptores deciden los temas a los que se suscriben.

Los **temas personalizados** son temas de terceros y de aplicación. Cuando cree un tema personalizado, o se le asigne acceso al mismo, verá ese tema personalizado en su suscripción. 

Cuando diseñe la aplicación, tiene flexibilidad al decidir cuántos temas se crean. Para soluciones grandes, cree un **tema personalizado** para **cada categoría de eventos relacionados**. Por ejemplo, considere una aplicación que envía eventos relacionados con la modificación de las cuentas de usuario y el procesamiento de pedidos. Es poco probable que algún controlador de eventos quiera ambas categorías de eventos. Cree dos temas personalizados y deje que los controladores de eventos se suscriban a uno que les interese. Para soluciones pequeñas, puede que prefiera enviar todos los eventos a un solo tema. Los suscriptores de eventos se pueden filtrar por los tipos de evento que desean.

## <a name="event-schema"></a>Esquema del evento
Para información detallada sobre el esquema de eventos, consulte [Esquema de eventos de Azure Event Grid](event-schema.md). En el caso de los temas personalizados, el publicador de eventos determina el objeto **data**. Los datos de nivel superior deben tener los mismos campos que los eventos estándar definidos por recursos.

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

En las secciones siguientes se proporcionan vínculos a tutoriales para crear temas personalizados mediante Azure Portal, la CLI, PowerShell y plantillas de Azure Resource Manager (ARM). 


## <a name="azure-portal-tutorials"></a>Tutoriales de Azure Portal
|Título  |Descripción  |
|---------|---------|
| [Guía de inicio rápido: creación y enrutamiento de eventos personalizados con Azure Portal](custom-event-quickstart-portal.md) | Muestra cómo utilizar Azure Portal para enviar eventos personalizados. |
| [Guía de inicio rápido: enrutamiento de eventos personalizados a Azure Queue Storage](custom-event-to-queue-storage.md) | Describe cómo enviar eventos personalizados a una instancia de Queue Storage. |
| [Publicación en tema personalizado](post-to-custom-topic.md) | Se describe cómo publicar un evento en un tema personalizado. |


## <a name="azure-cli-tutorials"></a>Tutoriales de la CLI de Azure
|Título  |Descripción  |
|---------|---------|
| [Guía de inicio rápido: creación y enrutamiento de eventos personalizados con la CLI de Azure](custom-event-quickstart.md) | Muestra cómo utilizar la CLI de Azure para enviar eventos personalizados. |
| [CLI de Azure: creación de un tema personalizado de Event Grid](./scripts/event-grid-cli-create-custom-topic.md)|Script de ejemplo que crea un tema personalizado. El script recupera el punto de conexión y una clave.|
| [CLI de Azure: suscripción a eventos de un tema personalizado](./scripts/event-grid-cli-subscribe-custom-topic.md)|Script de ejemplo que crea una suscripción a un tema personalizado. Envía eventos a un webhook.|

## <a name="azure-powershell-tutorials"></a>Tutoriales de Azure PowerShell
|Título  |Descripción  |
|---------|---------|
| [Guía de inicio rápido: creación y enrutamiento de eventos personalizados con Azure PowerShell](custom-event-quickstart-powershell.md) | Muestra cómo utilizar Azure PowerShell para enviar eventos personalizados. |
| [PowerShell: Creación de un tema personalizado de Event Grid](./scripts/event-grid-powershell-create-custom-topic.md)|Script de ejemplo que crea un tema personalizado. El script recupera el punto de conexión y una clave.|
| [PowerShell: suscripción a eventos de un tema personalizado](./scripts/event-grid-powershell-subscribe-custom-topic.md)|Script de ejemplo que crea una suscripción a un tema personalizado. Envía eventos a un webhook.|

## <a name="arm-template-tutorials"></a>Tutoriales de plantillas de ARM
|Título  |Descripción  |
|---------|---------|
| [Plantilla de Resource Manager: tema personalizado y punto de conexión del webhook](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.eventgrid/event-grid) | Plantilla de Resource Manager que crea un tema personalizado y una suscripción a ese tema personalizado. Envía eventos a un webhook. |
| [Plantilla de Resource Manager: tema personalizado y punto de conexión a Event Hubs](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.eventgrid/event-grid-event-hubs-handler)| Plantilla de Resource Manager que crea una suscripción a un tema personalizado. Envía eventos a una instancia de Azure Event Hubs. |

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 

- [Temas del sistema](system-topics.md)
- [Dominios](event-domains.md)