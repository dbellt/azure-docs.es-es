---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 06/01/2021
ms.author: glenga
ms.openlocfilehash: e65a628f846892294fdd9be693b89a535ac694a0
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111901454"
---
## <a name="supported-versions"></a>Versiones compatibles

Las versiones del entorno en tiempo de ejecución de Functions funcionan con versiones específicas de .NET. Para obtener más información sobre las versiones de Functions, consulte [Introducción a las versiones de tiempo de ejecución de Azure Functions](../articles/azure-functions/functions-versions.md). La compatibilidad con versiones depende de si las funciones se ejecutan en proceso o fuera de proceso (aislado). 

En la tabla siguiente se muestra el nivel más alto de .NET Core o .NET Framework que se puede usar con una versión específica de Functions. 

| Versiones del entorno en tiempo de ejecución de Functions | En proceso<br/>([Biblioteca de clases .NET](../articles/azure-functions/functions-dotnet-class-library.md)) | Fuera de proceso<br/>([.NET aislado](../articles/azure-functions/dotnet-isolated-process-guide.md)) |
| ---- | ---- | --- |
| Functions 4.x<sup>1</sup> | .NET 6.0 (versión preliminar)| .NET 6.0 (versión preliminar) |
| Functions 3.x | .NET Core 3.1 | .NET 5.0 |
| Functions 2.x | .NET Core 2.1<sup>2</sup> | N/D |
| Functions 1.x | .NET Framework 4.8 | N/D |

<sup>1</sup>Azure Functions proporciona compatibilidad experimental que le permite probar las funciones que se ejecutan en la versión preliminar de .NET 6.0. Esta versión previa no se admite oficialmente. Para más información, consulte la página de la [primera versión preliminar de Azure Functions v4](https://aka.ms/functions-dotnet6earlypreview-wiki).  
<sup>2</sup> Para más información, consulte [Consideraciones sobre Functions v2.x](../articles/azure-functions/functions-dotnet-class-library.md#functions-v2x-considerations).   

Para obtener las últimas noticias sobre las versiones de Azure Functions, incluida la eliminación de versiones secundarias específicas anteriores, revise los [anuncios de Azure App Service](https://github.com/Azure/app-service-announcements/issues).
