---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 814bf085ca910ef07a13df779eef25e0e6510e57
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96011776"
---
Todos los datos almacenados en Azure Files se cifran en reposo mediante el cifrado de servicio de almacenamiento (SSE) de Azure. El cifrado del servicio de almacenamiento funciona de forma similar a BitLocker en Windows: los datos se cifran bajo el nivel del sistema de archivos. Dado que los datos se cifran bajo el sistema de archivos del recurso compartido de archivos de Azure, ya que se codifican en el disco, no es necesario tener acceso a la clave subyacente en el cliente para leer o escribir en dicho recurso compartido. El cifrado en reposo se aplica a los protocolos SMB y NFS.

De manera predeterminada, los datos almacenados en Azure Files se cifran con claves administradas por Microsoft. Con las claves administradas por Microsoft, Microsoft mantiene las claves para cifrar o descifrar los datos y es responsable de rotarlas de forma periódica. También puede elegir administrar sus propias claves, lo que le permitirá controlar el proceso de rotación. Si decide cifrar los recursos compartidos de archivos con claves administradas por el cliente, Azure Files está autorizado para tener acceso a las claves con el fin de satisfacer las solicitudes de lectura y escritura de los clientes. Con las claves administradas por el cliente, puede revocar esta autorización en cualquier momento, pero esto significa que el recurso compartido de archivos de Azure ya no será accesible a través de SMB o la API FileREST.

Azure Files usa el mismo esquema de cifrado que los otros servicios de almacenamiento de Azure, como Azure Blob Storage. Para aprender más sobre el cifrado del servicio de almacenamiento (SSE) de Azure, consulte [Cifrado de Azure Storage para datos en reposo](../articles/storage/common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).