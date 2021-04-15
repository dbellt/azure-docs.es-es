---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: twooley
ms.service: storage
ms.topic: include
ms.date: 09/30/2020
ms.author: twooley
ms.custom: include file
ms.openlocfilehash: 7098f23e9b5b6f56fbbe761335afc65375aea680
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96509512"
---
**Azure Data Lake Storage Gen2** no es un servicio dedicado ni un tipo de cuenta de almacenamiento. Es la versión más reciente de las funcionalidades dedicadas al análisis de macrodatos.  Estas funcionalidades están disponibles en una cuenta de almacenamiento de uso general v2 o de BlockBlobStorage, y para obtenerlas se debe habilitar la característica **Espacio de nombres jerárquico** de la cuenta. Para conocer los objetivos la escala, consulte estos artículos. 

- [Objetivos de escala de Blob Storage](../articles/storage/blobs/scalability-targets.md#scale-targets-for-blob-storage).
- [Objetivos de escala de las cuentas de almacenamiento estándar](../articles/storage/common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#scale-targets-for-standard-storage-accounts).

**Azure Data Lake Storage Gen1** es un servicio dedicado. Es un repositorio de gran escala en toda la empresa para cargas de trabajo de análisis de macrodatos. Puede usar Data Lake Storage Gen1 para capturar datos de cualquier tamaño, tipo y velocidad de ingesta en un único lugar para realizar análisis exploratorios y operativos. No hay ningún límite en cuanto a la cantidad de datos que puede almacenar en una cuenta de Data Lake Storage Gen1.

| **Recurso** | **LIMIT** | **Comentarios** |
| --- | --- | --- |
| Número máximo de cuentas de Data Lake Store Gen1, por suscripción y por región |10 | Para solicitar un aumento de este límite, póngase en contacto con el soporte técnico. |
| Número máximo de ACL de acceso, por archivo o carpeta |32 | Se trata de un límite máximo. Use grupos para administrar el acceso con menos entradas. |
| Número máximo de ACL predeterminadas, por archivo o carpeta |32 | Se trata de un límite máximo. Use grupos para administrar el acceso con menos entradas. |