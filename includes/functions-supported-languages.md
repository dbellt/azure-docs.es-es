---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 06/01/2021
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 3edb1db5fd714e8ff9b21a0617ac5053e77a0e20
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111905641"
---
|Idioma                                 |1.x         |2.x| 3.x |
|-----------------------------------------|------------|---| --- |
|[C#](../articles/azure-functions/functions-dotnet-class-library.md)<sup>1</sup>|Disponibilidad general (.NET Framework 4.8)|Disponibilidad general (.NET Core 2.1<sup>2</sup>)| Disponibilidad general (.NET Core 3.1)<br/>[Disponibilidad general (.NET 5.0)](../articles/azure-functions/dotnet-isolated-process-guide.md) |
|[JavaScript](../articles/azure-functions/functions-reference-node.md#node-version)|Disponibilidad general (nodo 6)|Disponibilidad general (nodos 10 y 8)| Disponibilidad general (nodos 14, 12 y 10) |
|[F#](../articles/azure-functions/functions-reference-fsharp.md)|Disponibilidad general (.NET Framework 4.8)|Disponibilidad general (.NET Core 2.1<sup>2</sup>)| Disponibilidad general (.NET Core 3.1) |
|[Java](../articles/azure-functions/functions-reference-java.md)|N/D|Disponibilidad general (Java 8)| Disponibilidad general (Java 11 y 8)|
|[PowerShell](../articles/azure-functions/functions-reference-powershell.md) |N/D|Disponibilidad general (PowerShell Core 6)| Disponibilidad general (PowerShell 7 y Core 6)|
|[Python](../articles/azure-functions/functions-reference-python.md#python-version)|N/D|Disponibilidad general (Python 3.7 y 3.6)| Disponibilidad general (Python 3.8, 3.7 y 3.6) <br/> Versión preliminar (Python 3.9)|
|[TypeScript](../articles/azure-functions/functions-reference-node.md#typescript) |N/D|Disponibilidad general<sup>3</sup>| Disponibilidad general<sup>3</sup> |

<sup>1</sup> Hay disponible una versión experimental de Azure Functions que le permite usar la versión preliminar de .NET 6.0. Para más información, consulte la página de la [primera versión preliminar de Azure Functions v4](https://aka.ms/functions-dotnet6earlypreview-wiki).
<sup>2</sup> Las aplicaciones de biblioteca de clases de .NET destinadas a la versión en tiempo de ejecución 2.x se ejecutan en .NET Core 3.1 en el modo de compatibilidad de .NET Core 2.x. Para más información, consulte [Consideraciones sobre Functions v2.x](../articles/azure-functions/functions-dotnet-class-library.md#functions-v2x-considerations).  
<sup>3</sup> Se admite mediante la transpilación de JavaScript.

Para más información sobre las versiones del lenguaje que se admiten, consulte el artículo de la guía para desarrolladores específica del lenguaje.   
Para información sobre los cambios planeados en la compatibilidad de lenguaje, consulte [Azure roadmap](https://azure.microsoft.com/roadmap/?tag=functions).
