---
title: Enlaces de Azure Event Grid para Azure Functions
description: Comprenda cómo se pueden controlar eventos de Event Grid en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: fasttrack-edit
ms.openlocfilehash: f346995e47c0156f546722031ebaa8734239549c
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536714"
---
# <a name="azure-event-grid-bindings-for-azure-functions"></a>Enlaces de Azure Event Grid para Azure Functions

En esta referencia se explica cómo controlar eventos de [Event Grid](../event-grid/overview.md) en Azure Functions. Para obtener detalles sobre cómo controlar mensajes de Event Grid en un punto de conexión HTTP, vea [Recepción de eventos en un punto de conexión de HTTP](../event-grid/receive-events.md).

Event Grid es un servicio de Azure que envía solicitudes HTTP para notificarle acerca de eventos que ocurre en los *publicadores*. Un publicador es el servicio o recurso que origina el evento. Por ejemplo, una cuenta de Azure Blob Storage es un publicador y [una carga o eliminación de blobs es un evento](../storage/blobs/storage-blob-event-overview.md). Algunos [servicios de Azure tienen compatibilidad integrada para publicar eventos en Event Grid](../event-grid/overview.md#event-sources).

Los *controladores* de eventos reciben y procesan eventos. Azure Functions es uno de los [servicios de Azure con compatibilidad integrada para controlar eventos de Event Grid](../event-grid/overview.md#event-handlers). En esta referencia se aprende a usar un desencadenador de Event Grid para invocar a una función cuando se recibe un evento de Event Grid y a usar el enlace de salida para enviar eventos a una [Publicación en un tema personalizado de Azure Event Grid](../event-grid/post-to-custom-topic.md).

Si lo prefiere, puede usar un desencadenador HTTP para controlar los eventos de Event Grid; consulte [Recepción de eventos en un punto de conexión HTTP](../event-grid/receive-events.md). Actualmente, no puede usar un desencadenador de Event Grid para una aplicación de Azure Functions cuando el evento se entrega en el esquema de[ CloudEvents](../event-grid/cloudevents-schema.md#azure-functions). En cambio, debe usar un desencadenador HTTP.

| Acción | Tipo |
|---------|---------|
| Ejecuta una función cuando se envía un evento de Event Grid | [Desencadenador](./functions-bindings-event-grid-trigger.md) |
| Envía un evento de Event Grid |[Enlace de salida](./functions-bindings-event-grid-output.md) |

El código de esta referencia tiene como valor predeterminado la sintaxis de .NET Core, que se usa en Functions versión 2.x y superiores. Para obtener información sobre la sintaxis de 1.x, consulte las [plantillas de Functions 1.x](https://github.com/Azure/azure-functions-templates/tree/v1.x/Functions.Templates/Templates).

## <a name="add-to-your-functions-app"></a>Adición a la aplicación de Functions

### <a name="functions-2x-and-higher"></a>Functions 2.x y superiores

Para trabajar con el desencadenador y los enlaces, es necesario hacer referencia al paquete adecuado. En las bibliotecas de clases de .NET se usa el paquete NuGet, mientras que en los demás tipos de aplicaciones se emplea el conjunto de extensiones.

| Idioma                                        | Agregar mediante...                                   | Observaciones 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Instalación del [paquete NuGet], versión 2.x | |
| Script de C#, Java, JavaScript, Python, PowerShell | Registro de [conjunto de extensiones]          | Se recomienda usar la [extensión Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) con Visual Studio Code. |
| Script de C# (solo en línea en Azure Portal)         | Adición de un enlace                            | Para actualizar extensiones de enlace existentes sin tener que volver a publicar la aplicación de funciones, vea [Actualización de las extensiones]. |

[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid
[Actualización de las extensiones]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

#### <a name="event-grid-extension-3x-and-higher"></a>Extensión 3.x y superior de Event Grid

Hay disponible una nueva versión de la extensión de enlaces de Event Grid en un [paquete NuGet en versión preliminar](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid/3.0.0-beta.2). En el caso de las aplicaciones .NET, cambian los tipos con los que se puede enlazar; así, los tipos `Microsoft.Azure.EventGrid.Models` se reemplazan por otros tipos más recientes de [Azure.Messaging.EventGrid](/dotnet/api/azure.messaging.eventgrid). Los [eventos en la nube](/dotnet/api/azure.messaging.cloudevent) también se admiten en la nueva extensión de Event Grid.

> [!NOTE]
> El paquete en versión preliminar no se incluye en un conjunto de extensiones y debe instalarse manualmente. En el caso de las aplicaciones .NET, agregue una referencia al paquete. Con el resto de tipos de aplicaciones, consulte [Actualización de las extensiones].

[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Actualización de las extensiones]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Las aplicaciones de Functions 1.x tienen automáticamente una referencia al paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), versión 2.x.

## <a name="next-steps"></a>Pasos siguientes
* [Ejecución de una función cuando se envía un evento de Event Grid](./functions-bindings-event-grid-trigger.md)
* [Envío de un evento de Event Grid](./functions-bindings-event-grid-trigger.md)
