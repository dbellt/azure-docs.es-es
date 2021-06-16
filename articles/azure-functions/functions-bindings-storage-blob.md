---
title: Desencadenador y enlaces de Azure Blob Storage para Azure Functions
description: Obtenga información sobre cómo usar el desencadenador y los enlaces de Azure Blob Storage en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: 30bd445940187d3514dc4b7024d3dbb8f7d920b2
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536731"
---
# <a name="azure-blob-storage-bindings-for-azure-functions-overview"></a>Introducción a los enlaces de Azure Blob Storage para Azure Functions

Azure Functions se integra con [Azure Storage](../storage/index.yml) mediante [desencadenadores y enlaces](./functions-triggers-bindings.md). La integración con Blob Storage permite crear funciones que reaccionan a los cambios en los datos del blob, así como a los valores de lectura y escritura.

| Acción | Tipo |
|---------|---------|
| Ejecución de una función cuando cambian los datos de Blob Storage | [Desencadenador](./functions-bindings-storage-blob-trigger.md) |
| Lectura de datos de Blob Storage en una función | [Enlace de entrada](./functions-bindings-storage-blob-input.md) |
| Escritura de datos de almacenamiento de blobs desde una función |[Enlace de salida](./functions-bindings-storage-blob-output.md) |

## <a name="add-to-your-functions-app"></a>Adición a la aplicación de Functions

### <a name="functions-2x-and-higher"></a>Functions 2.x y superiores

Para trabajar con el desencadenador y los enlaces, es necesario hacer referencia al paquete adecuado. En las bibliotecas de clases de .NET se usa el paquete NuGet, mientras que en los demás tipos de aplicaciones se emplea el conjunto de extensiones.

| Idioma                                        | Agregar mediante...                                   | Observaciones 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Instalación del [paquete NuGet], versión 3.x | |
| Script de C#, Java, JavaScript, Python, PowerShell | Registro de [conjunto de extensiones]          | Se recomienda usar la [extensión Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) con Visual Studio Code. |
| Script de C# (solo en línea en Azure Portal)         | Adición de un enlace                            | Para actualizar extensiones de enlace existentes sin tener que volver a publicar la aplicación de funciones, vea [Actualización de las extensiones]. |

#### <a name="storage-extension-5x-and-higher"></a>Extensión de Storage 5.x y versiones posteriores

Hay disponible una nueva versión de la extensión de enlaces de Storage en un [paquete NuGet en versión preliminar](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.3). Esta versión preliminar presenta la posibilidad de [conectarse con una identidad en lugar de un secreto](./functions-reference.md#configure-an-identity-based-connection). En el caso de las aplicaciones .NET, también cambian los tipos con los que se puede enlazar; así, los tipos `WindowsAzure.Storage` y `Microsoft.Azure.Storage` se reemplazan por otros más recientes de [Azure.Storage.Blobs](/dotnet/api/azure.storage.blobs).

> [!NOTE]
> El paquete en versión preliminar no se incluye en un conjunto de extensiones y debe instalarse manualmente. En el caso de las aplicaciones .NET, agregue una referencia al paquete. Con el resto de tipos de aplicaciones, consulte [Actualización de las extensiones].

[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Actualización de las extensiones]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Las aplicaciones de Functions 1.x tienen automáticamente una referencia al paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), versión 2.x.

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="hostjson-settings"></a>configuración de host.json

> [!NOTE]
> Esta sección no se aplica cuando se usan versiones de extensión anteriores a 5.0.0. Para esas versiones, no hay ningún valor de configuración global para los blobs.

En esta sección se describen las opciones de configuración global disponibles para este enlace cuando se usa la [versión de extensión 5.0.0 y posteriores](#storage-extension-5x-and-higher). El siguiente archivo *host.json* de ejemplo contiene solo la configuración de la versión 2.x+ de este enlace. Para más información acerca de las opciones de configuración globales de la versión 2.x y posteriores de Functions, consulte [Referencia de host.json para Azure Functions](functions-host-json.md).

```json
{
    "version": "2.0",
    "extensions": {
        "blobs": {
            "maxDegreeOfParallelism": "4"
        }
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|maxDegreeOfParallelism|8* (el número de núcleos disponibles)|Número entero de invocaciones simultáneas permitidas para cada función desencadenada por un blob. El valor mínimo permitido es 1.|

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando cambian los datos de Blob Storage](./functions-bindings-storage-blob-trigger.md)
- [Lectura de datos de Blob Storage cuando se ejecuta una función](./functions-bindings-storage-blob-input.md)
- [Escritura de datos de Blob Storage desde una función](./functions-bindings-storage-blob-output.md)
