---
title: Administración de un grupo de capacidad de QoS manual de Azure NetApp Files | Microsoft Docs
description: Se describe cómo administrar un grupo de capacidad que usa el tipo de QoS manual, incluida la configuración de un grupo de capacidad de QoS manual y el cambio de un grupo de capacidad para usar QoS manual.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/14/2021
ms.author: b-juche
ms.openlocfilehash: 18333b7c9fded73d330cacd6322b0869bf65f9d9
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071090"
---
# <a name="manage-a-manual-qos-capacity-pool"></a>Administración de un grupo de capacidad de QoS manual

En este artículo se describe cómo administrar un grupo de capacidad que usa el tipo de QoS manual.  

Vea [Jerarquía de almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md) y [Consideraciones sobre el rendimiento de Azure NetApp Files](azure-netapp-files-performance-considerations.md) para comprender las consideraciones sobre los tipos de QoS.  

## <a name="set-up-a-new-manual-qos-capacity-pool"></a>Configuración de un grupo de capacidad de QoS manual 

Para crear un nuevo grupo de capacidad mediante el tipo de QoS manual:

1. Siga los pasos que se indican en [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).  

2. En la ventana Nuevo grupo de capacidad, seleccione el tipo **QoS manual**.  

## <a name="change-a-capacity-pool-to-use-manual-qos"></a><a name="change-to-qos"></a>Cambio de un grupo de capacidad para usar QoS manual

Puede cambiar un grupo de capacidad que actualmente usa el tipo de QoS automático para usar el tipo de QoS manual.  

> [!IMPORTANT]
> Establecer el tipo de capacidad en QoS manual es un cambio permanente. No se puede convertir un grupo de capacidad de QoS manual en uno automático.  
> En el momento de la conversión, los niveles de rendimiento se pueden limitar para ajustarse a los de los volúmenes del tipo de QoS manual. Vea [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md#resource-limits).

1. En la hoja de administración de la cuenta de NetApp, haga clic en **Grupos de capacidad** para mostrar los grupos de capacidad existentes.   
 
2.  Haga clic en el grupo de capacidad que desea cambiar para usar QoS manual.

3.  Haga clic en **Cambiar el tipo de QoS**. A continuación, establezca **Nuevo tipo de QoS** en **Manual**. Haga clic en **OK**. 

![Cambiar el tipo de QoS](../media/azure-netapp-files/change-qos-type.png)


## <a name="monitor-the-throughput-of-a-manual-qos-capacity-pool"></a>Supervisión del rendimiento de un grupo de capacidad de QoS manual  

Hay disponibles métricas para ayudarle a supervisar el rendimiento de lectura y escritura de un volumen.  Consulte [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md).  

## <a name="modify-the-allotted-throughput-of-a-manual-qos-volume"></a>Modificación del rendimiento asignado de un volumen de QoS manual 

Si un volumen se incluye en un grupo de capacidad de QoS manual, puede modificar el rendimiento del volumen asignado según sea necesario.

1. En la página **Volúmenes**, seleccione el volumen cuyo rendimiento quiere modificar.   

2. Haga clic en **Cambiar rendimiento**. Especifique el **rendimiento (MiB/S)** que quiere. Haga clic en **OK**. 

    ![Cambiar el rendimiento de QoS](../media/azure-netapp-files/change-qos-throughput.png)

## <a name="next-steps"></a>Pasos siguientes  

* [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
* [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md)
* [Consideraciones sobre el rendimiento de Azure NetApp Files](azure-netapp-files-performance-considerations.md)
* [Solución de problemas de grupos de capacidad](troubleshoot-capacity-pools.md)
* [Jerarquía de almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
* [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Creación de un volumen NFS](azure-netapp-files-create-volumes.md)
* [Creación de un volumen SMB](azure-netapp-files-create-volumes-smb.md)
* [Creación de un volumen de protocolo dual](create-volumes-dual-protocol.md)
