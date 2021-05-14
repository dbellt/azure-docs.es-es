---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/04/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 6e819e1078ac90ef16070702e7961122b06c1d6f
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107879404"
---
Mientras se encuentra en versión preliminar, NFS tiene las siguientes limitaciones:

- NFS 4.1 actualmente solo admite la mayoría de las características de la [especificación del protocolo](https://tools.ietf.org/html/rfc5661). No se admiten algunas características, como delegaciones y devoluciones de llamada de todo tipo, actualizaciones de bloqueos y degradados, y el cifrado y la autenticación Kerberos.
- Si la mayoría de las solicitudes se centran en metadatos, la latencia será peor en comparación con las operaciones de lectura, escritura y actualización.
- Los recursos compartidos de NFS solo se pueden habilitar o crear en nuevas cuentas de almacenamiento y no en las existentes.
- Solo se admiten las API de REST del plano de administración. Las API REST del plano de datos no están disponibles, lo que significa que las herramientas como el Explorador de Storage no funcionarán con los recursos compartidos de NFS ni podrán examinar los datos de recursos compartidos de NFS en Azure Portal.
- AzCopy no se admite actualmente.
- Solo están disponibles para el nivel prémium.
- Los recursos compartidos NFS solo aceptan UID/GID numéricos. Para evitar que los clientes envíen UID/GID alfanuméricos, debe deshabilitar la asignación de identificadores.
- Los recursos compartidos solo se pueden montar desde una cuenta de almacenamiento en una máquina virtual individual cuando se usan vínculos privados. Se producirá un error al intentar montar recursos compartidos de otras cuentas de almacenamiento.
- Es mejor opción depender en los permisos asignados al grupo principal. A veces, los permisos asignados al grupo no principal del usuario pueden dar lugar a que se deniegue el acceso debido a un error conocido.

### <a name="azure-storage-features-not-yet-supported"></a>Características de Azure Storage que aún no son compatibles

Además, las siguientes características de Azure Files no están disponibles con los recursos compartidos de NFS:

- Autenticación basada en identidad
- Soporte técnico de Azure Backup
- Instantáneas
- Eliminación temporal
- Compatibilidad total con el cifrado en tránsito (para más información, vea [seguridad de NFS](../articles/storage/files/storage-files-compare-protocols.md#security))
- Azure File Sync (solo disponible para los clientes de Windows, que son incompatibles con NFS 4.1)
