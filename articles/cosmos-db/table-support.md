---
title: Compatibilidad de Azure Table Storage con Azure Cosmos DB
description: Obtenga información acerca de la forma en que Table API de Azure Cosmos DB y las tablas de Azure Storage funcionan conjuntamente compartiendo el mismo modelo de datos de tabla y las mismas operaciones
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 01/08/2021
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.openlocfilehash: 6c38ed9767f92eaaa53b7b95eba9e455130606b3
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063959"
---
# <a name="developing-with-azure-cosmos-db-table-api-and-azure-table-storage"></a>Implementación con Table API de Azure Cosmos DB y Azure Table Storage
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Table API de Azure Cosmos DB y Azure Table Storage comparten el mismo modelo de datos de tablas y exponen las mismas operaciones de creación, eliminación, actualización y consulta a través de sus SDK.

> [!NOTE]
> El [modo de capacidad sin servidor](serverless.md) ahora está disponible en Table API de Azure Cosmos DB.

[!INCLUDE [storage-table-cosmos-comparison](../../includes/storage-table-cosmos-comparison.md)]

## <a name="developing-with-the-azure-cosmos-db-table-api"></a>Desarrollo con Table API de Azure Cosmos DB

En este momento, [Table API de Azure Cosmos DB](table-introduction.md) tiene cuatro SDK disponibles para el desarrollo: 

* [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table): .NET SDK. Esta biblioteca va destinada a .NET Standard y tiene las mismas clases y firmas de método que el [SDK de Azure Storage de Windows](https://www.nuget.org/packages/WindowsAzure.Storage) público, pero también se puede conectar a cuentas de Azure Cosmos DB mediante Table API. Se recomienda que los usuarios de la biblioteca de .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/) actualicen a [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), ya que está en modo de mantenimiento y pronto quedará en desuso.

* [SDK de Python](table-sdk-python.md): El nuevo SDK de Python de Azure Cosmos DB es el único SDK que admite Azure Table Storage en Python. Este SDK se conecta con Azure Table Storage y Table API de Azure Cosmos DB.

* [SDK de Java](table-sdk-java.md): Este SDK de Azure Storage tiene la capacidad de conectarse a las cuentas de Azure Cosmos DB mediante Table API.

* [SDK de Node.js](table-sdk-nodejs.md): Este SDK de Azure Storage tiene la capacidad de conectarse a las cuentas de Azure Cosmos DB mediante Table API.


Puede encontrar información adicional sobre el funcionamiento con Table API en el artículo [Preguntas más frecuentes: desarrollo con Table API en .NET](table-api-faq.yml).

## <a name="developing-with-azure-table-storage"></a>Desarrollo con Azure Table Storage

Azure Table Storage tiene las siguientes SDK disponibles para el desarrollo:

- Las bibliotecas [Microsoft.Azure.Storage.Blob](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/), [Microsoft.Azure.Storage.File](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/), [Microsoft.Azure.Storage.Queue](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/) y [Microsoft.Azure.Storage.Common](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) le permiten trabajar con el servicio de almacenamiento de tablas de Azure. Si usa Table API en Azure Cosmos DB, puede usar entonces la biblioteca [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/).
- [SDK de Python](https://github.com/Azure/azure-cosmos-table-python). El SDK Table de Azure Cosmos DB para Python es compatible con el servicio Table Storage (porque las Table API de Azure Table Storage y Cosmos DB comparten las mismas características y funcionalidades, y, a fin de factorizar nuestros esfuerzos de desarrollo del SDK, se recomienda usar este SDK).
- [SDK de Azure Storage para Java](https://github.com/azure/azure-storage-java). Este SDK de Azure Storage proporciona una biblioteca cliente en Java que consume Azure Table Storage.
- [SDK de Node.js](https://github.com/Azure/azure-storage-node). Este SDK proporciona un paquete Node.js y una biblioteca cliente de JavaScript compatible con el explorador que consume Storage Table service.
- [Módulo de PowerShell AzureRmStorageTable](https://www.powershellgallery.com/packages/AzureRmStorageTable). Este módulo de PowerShell contiene cmdlets para trabajar con tablas de almacenamiento.
- [Biblioteca cliente de Azure Storage para C++](https://github.com/Azure/azure-storage-cpp/). Esta biblioteca le permite compilar aplicaciones en Azure Storage.
- [Biblioteca cliente de Table de Azure Storage Table para Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table). Este proyecto proporciona un paquete Ruby que facilita el acceso a instancias de Azure Storage Table service.
- [Biblioteca cliente de PHP de Table de Azure Storage Table](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table). Este proyecto proporciona una biblioteca cliente de PHP que facilita el acceso a instancias de Azure Storage Table service.


   





