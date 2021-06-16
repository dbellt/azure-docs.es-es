---
title: 'Introducción a Table Storage: almacenamiento de objetos en Azure | Microsoft Docs'
description: Almacene datos estructurados en la nube con el Almacenamiento de tablas de Azure, un almacén de datos NoSQL.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.devlang: dotnet
ms.topic: overview
ms.date: 05/27/2021
ms.subservice: tables
ms.openlocfilehash: 690afd54cef64f893bf5e68d537de4a47f1a93db
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110654968"
---
# <a name="what-is-azure-table-storage-"></a>¿Qué es Azure Table Storage? 

[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-tip-include.md)]

Azure Table Storage es un servicio que almacena datos estructurados no relacionales (también conocidos como datos NoSQL estructurados) en la nube, lo que proporciona un almacén de claves y atributos con un diseño sin esquema. Como Almacenamiento de tablas carece de esquema, es fácil adaptar los datos a medida que evolucionan las necesidades de la aplicación. El acceso a los datos de Table Storage es rápido y rentable para muchos tipos de aplicaciones y, por lo general, el costo es normalmente menor que con el SQL tradicional para volúmenes parecidos de datos.

Table Storage se puede usar para almacenar conjuntos de datos flexibles, como datos de usuarios para aplicaciones web, libretas de direcciones, información de dispositivos u otros tipos de metadatos que el servicio requiera. Una tabla puede almacenar un número cualquiera de entidades y una cuenta de almacenamiento puede incluir un número cualquiera de tablas, hasta alcanzar el límite de capacidad de este tipo de cuenta.

[!INCLUDE [storage-table-concepts-include](../../../includes/storage-table-concepts-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* El [Explorador de Microsoft Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.

* [Get started with Azure Table Storage in .NET](../../cosmos-db/tutorial-develop-table-dotnet.md) (Introducción a Azure Table Storage en .NET)

* Consulte la documentación de referencia de Table service para obtener información detallada acerca de las API disponibles:

    * [Referencia de la biblioteca de clientes de almacenamiento para .NET](/dotnet/api/overview/azure/storage)

    * [Referencia de API de REST](/rest/api/storageservices/)