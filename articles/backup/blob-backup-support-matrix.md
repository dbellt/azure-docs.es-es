---
title: Matriz de compatibilidad para la copia de seguridad de blobs de Azure
description: Proporciona un resumen de opciones y limitaciones de compatibilidad a la hora de realizar copias de seguridad en blobs de Azure.
ms.topic: conceptual
ms.date: 05/05/2021
ms.custom: references_regions
ms.openlocfilehash: eb923cb38a76e56c1ba60b3863b111aac83eeb7e
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517683"
---
# <a name="support-matrix-for-azure-blobs-backup"></a>Matriz de compatibilidad para la copia de seguridad de blobs de Azure

En este artículo se resume la disponibilidad regional, los escenarios admitidos y las limitaciones de la copia de seguridad operativa de blobs.

## <a name="supported-regions"></a>Regiones admitidas

La copia de seguridad operativa para blobs está disponible actualmente en las siguientes regiones: Centro de Australia, Este de Australia, Sur de Brasil, Centro de Canadá, Centro de la India, Centro de EE. UU., Asia Pacífico, Este de EE. UU., Este de EE. UU. 2, Centro-oeste de Alemania, Japón Oriental, Japón Occidental, Centro de Corea del Sur, Sur de Corea del Sur, Norte de Europa, Centro-sur de EE. UU., Sudeste Asiático, Norte de Suiza, Norte de Emiratos Árabes, Sur de Reino Unido, Oeste de Reino Unido, Centro-oeste de EE. UU., Oeste de Europa, Oeste de EE. UU., Oeste de EE. UU. 2

## <a name="limitations"></a>Limitaciones

La copia de seguridad operativa de blobs usa la restauración a un momento dado de los blobs, el control de versiones de blobs, la eliminación temporal de blobs, la fuente de cambios para blobs y el bloqueo de eliminación para proporcionar una solución de copia de seguridad local. Por lo tanto, las limitaciones que se aplican a estas funcionalidades también se aplican a la copia de seguridad operativa.

**Escenarios admitidos:** La copia de seguridad operativa solo admite blobs en bloques en las cuentas de almacenamiento v2 estándar de uso general. Por lo tanto, no se admiten las cuentas ADLS Gen2. Además, los blobs en páginas, los blobs en anexos y los blobs prémium de la cuenta de almacenamiento no se restaurarán y solo se restaurarán los blobs en bloques.

**Otras limitaciones:**

- Si ha eliminado un contenedor durante el período de retención, dicho contenedor no se restaurará con la operación de restauración a un momento dado. Si intenta restaurar un intervalo de blobs que incluye blobs en un contenedor eliminado, se producirá un error en la operación de restauración a un momento dado. Para más información sobre cómo proteger los contenedores contra la eliminación, consulte [Eliminación temporal de contenedores (versión preliminar)](../storage/blobs/soft-delete-container-overview.md).
- Si un blob se ha desplazado entre los niveles de acceso frecuente y esporádico en el período comprendido entre el momento actual y el punto de restauración, el blob se restaura a su nivel anterior. No se admite la restauración de blobs en blob en bloques en el nivel de archivo. Por ejemplo, si un blob en el nivel de acceso frecuente se movió al nivel de archivo hace dos días y se realiza una operación de restauración a un momento de hace tres días, el blob no se restaura en el nivel de acceso frecuente. Para restaurar un blob archivado, sáquelo antes del nivel de archivo. Para más información, consulte [Rehidratación de los datos de blob desde el nivel de archivo](../storage/blobs/storage-blob-rehydration.md).
- Un bloque que se ha cargado mediante la operación [Put Block](/rest/api/storageservices/put-block) o [Put Block from URL](/rest/api/storageservices/put-block-from-url), pero que no se ha confirmado a través de [Put Block List](/rest/api/storageservices/put-block-list), no es parte de un blob y, por tanto, no se restaura como parte de una operación de restauración.
- No se puede restaurar un blob con una concesión activa. Si se incluye un blob con una concesión activa en el intervalo de blobs para restaurar, la operación de restauración producirá un error automáticamente. Interrumpa las concesiones activas antes de iniciar la operación de restauración.
- Las instantáneas no se crean ni se eliminan como parte de una operación de restauración. Solo el blob base se restaura a su estado anterior.

## <a name="next-steps"></a>Pasos siguientes

[Introducción a la copia de seguridad operativa para blobs de Azure](blob-backup-overview.md)