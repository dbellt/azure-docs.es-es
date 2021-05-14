---
title: Modificación de la topología de Azure File Sync | Microsoft Docs
description: Instrucciones sobre cómo modificar la topología de sincronización de Azure File Sync
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 4/23/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 4977b56265f542f8f8ff13693abb468bc695a384
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992776"
---
# <a name="modify-your-azure-file-sync-topology"></a>Modificación de la topología de Azure File Sync

En este artículo se tratan las formas más comunes en las que a los clientes les gustaría modificar la topología de Azure File Sync y nuestras recomendaciones sobre cómo hacerlo. Si desea modificar la topología de Azure File Sync, consulte los procedimientos recomendados siguientes para evitar errores o posibles pérdidas de datos.

## <a name="migrate-a-server-endpoint-to-a-different-azure-file-sync-storage-sync-service"></a>Migración de un punto de conexión de servidor a un servicio de sincronización de almacenamiento de Azure File Sync diferente

Una vez que se haya asegurado de que los datos están actualizados en el servidor local, desaprovisione el punto de conexión de servidor. Para obtener instrucciones sobre cómo hacerlo, consulte [Desaprovisionamiento del punto de conexión de servidor de Azure File Sync](./file-sync-deprovision-server-endpoint.md#scenario-2-you-intend-to-delete-your-server-endpoint-and-stop-using-this-specific-azure-file-share). A continuación, vuelva a aprovisionar en el grupo de sincronización deseado y el servicio de sincronización de almacenamiento.

Si desea migrar todos los puntos de conexión de servidor asociados a un servidor a otro grupo de sincronización o servicio de sincronización de almacenamiento, consulte [Desaprovisionamiento de todos los puntos de conexión de servidor asociados a un servidor registrado](#deprovision-all-server-endpoints-associated-with-a-registered-server).

## <a name="change-the-granularity-of-a-server-endpoint"></a>Cambio de la granularidad de un punto de conexión de servidor

Después de confirmar que los datos están actualizados en el servidor local (consulte [Desaprovisionamiento del punto de conexión de servidor de Azure File Sync](./file-sync-deprovision-server-endpoint.md#scenario-2-you-intend-to-delete-your-server-endpoint-and-stop-using-this-specific-azure-file-share)), desaprovisione el punto de conexión de servidor. A continuación, vuelva a aprovisionarlo con la granularidad deseada.

## <a name="deprovision-azure-file-sync-topology"></a>Desaprovisionamiento de la topología de Azure File Sync

Los recursos de Azure File Sync deben desaprovisionarse en un orden específico: puntos de conexión de servidor, grupo de sincronización y, a continuación, servicio de almacenamiento. Aunque todo el flujo se documenta a continuación, puede detenerse en cualquier nivel que desee. 

En primer lugar, vaya al recurso de servicio de sincronización de almacenamiento en Azure Portal y seleccione un grupo de sincronización en el servicio de sincronización de almacenamiento. Siga los pasos descritos en [Desaprovisionamiento del punto de conexión de servidor de Azure File Sync](./file-sync-deprovision-server-endpoint.md) para garantizar la integridad y disponibilidad de los datos al eliminar los puntos de conexión de servidor. Para desaprovisionar el grupo de sincronización o el servicio de sincronización de almacenamiento, se deben eliminar todos los puntos de conexión de servidor. Si solo tiene como objetivo eliminar puntos de conexión de servidor específicos, puede detenerse aquí. 

Una vez que elimine todos los puntos de conexión de servidor del grupo de sincronización, elimine el punto de conexión de nube. 

A continuación, elimine el grupo de sincronización. 

Repita estos pasos para todos los grupos de sincronización del servicio de sincronización de almacenamiento que desea eliminar. Una vez eliminados todos los grupos de sincronización de ese servicio de sincronización de almacenamiento, elimine el recurso de servicio de sincronización de almacenamiento.

## <a name="rename-a-server-endpoint-path-or-sync-group"></a>Cambio de nombre de una ruta de acceso de punto de conexión de servidor o un grupo de sincronización

En la actualidad, esto no se admite. 

Si actualmente usa la unidad D y planea migrar a la nube, consulte [Conversión de la unidad D: de una máquina virtual en un disco de datos: Azure Virtual Machines](https://docs.microsoft.com/azure/virtual-machines/windows/change-drive-letter).

## <a name="deprovision-all-server-endpoints-associated-with-a-registered-server"></a>Desaprovisionamiento de todos los puntos de conexión de servidor asociados a un servidor registrado

Para asegurarse de que los datos están seguros y totalmente actualizados antes de desaprovisionar, consulte [Desaprovisionamiento del punto de conexión de servidor de Azure File Sync](./file-sync-deprovision-server-endpoint.md).

Vaya al recurso de servicio de sincronización de almacenamiento y vaya a la pestaña Servidores registrados. Seleccione el servidor del que desea cancelar el registro y seleccione **Cancelar registro de servidor**. Con esta acción, se desaprovisionarán rápidamente todos los puntos de conexión de servidor asociados a ese servidor.

## <a name="next-steps"></a>Pasos siguientes
* [Desaprovisionamiento de un punto de conexión de servidor de Azure File Sync](./file-sync-deprovision-server-endpoint.md)



