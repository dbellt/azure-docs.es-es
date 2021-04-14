---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: cedfd719a5f0aeed6fc2e932d3aa5189b83c9796
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026481"
---
La [biblioteca del Administrador de configuración de Microsoft Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/) proporciona una clase para analizar una cadena de conexión desde un archivo de configuración. La clase [CloudConfigurationManager](/previous-versions/azure/reference/mt634650(v=azure.100)) analiza los valores de configuración. Analiza la configuración de las aplicaciones cliente que se ejecutan en el escritorio, en un dispositivo móvil, en una máquina virtual de Azure o en un servicio en la nube de Azure.

Para hacer referencia al paquete `CloudConfigurationManager`, agregue las siguientes directivas `using`:

```csharp
using Microsoft.Azure; //Namespace for CloudConfigurationManager
using Microsoft.Azure.Storage;
```

Este es un ejemplo que muestra cómo recuperar una cadena de conexión de un archivo de configuración:

```csharp
// Parse the connection string and return a reference to the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));
```

El uso del Administrador de configuración de Azure es opcional. También se puede utilizar una API, como la [clase ConfigurationManager](/dotnet/api/system.configuration.configurationmanager)de .NET Framework.