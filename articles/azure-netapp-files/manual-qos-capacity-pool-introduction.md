---
title: Grupo de capacidad de QoS manual de Azure NetApp Files | Microsoft Docs
description: Proporciona una introducción al grupo de capacidad de QoS manual y referencias para obtener información adicional.
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
ms.topic: conceptual
ms.date: 06/14/2021
ms.author: b-juche
ms.openlocfilehash: 49cc885976e022e8b7c4c44c3546febc217e2d7c
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071828"
---
# <a name="manual-qos-capacity-pool"></a>Grupo de capacidad de QoS manual

En este artículo se proporciona una introducción a la función de grupo de capacidad de Calidad de servicio (QoS) manual.

## <a name="how-manual-qos-differs-from-auto-qos"></a>Diferencias entre QoS manual y QoS automático

El [tipo de QoS](azure-netapp-files-understand-storage-hierarchy.md#qos_types) es un atributo de un grupo de capacidad. Azure NetApp Files proporciona dos tipos de QoS de grupos de capacidad: automático (predeterminado) y manual.  

En un grupo de capacidad de QoS *manual*, puede asignar la capacidad y el rendimiento de un volumen de forma independiente. Para conocer los niveles de rendimiento mínimo y máximo, consulte [Límites de recursos de Azure NetApp Files](azure-netapp-files-resource-limits.md#resource-limits). El rendimiento total de todos los volúmenes creados con un grupo de capacidad de QoS manual está limitado por el rendimiento total del grupo. Viene determinado por la combinación del tamaño del grupo y el rendimiento del nivel de servicio. 

En un grupo de capacidad de QoS *automático*, el rendimiento se asigna automáticamente a los volúmenes del grupo, de forma proporcional a la cuota de tamaño asignada a los volúmenes.  

Vea [Jerarquía de almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md) y [Consideraciones sobre el rendimiento de Azure NetApp Files](azure-netapp-files-performance-considerations.md) para obtener consideraciones sobre los tipos de QoS.

## <a name="example-of-using-manual-qos"></a>Ejemplo de usar QoS manual

Cuando se usa un grupo de capacidad de QoS manual con, por ejemplo, un sistema SAP HANA, una base de datos de Oracle u otras cargas de trabajo que requieren varios volúmenes, se puede usar el grupo de capacidad para crear estos volúmenes de aplicaciones.  Cada volumen puede proporcionar el tamaño y rendimiento individuales para satisfacer los requisitos de la aplicación.  Para obtener información detallada sobre las ventajas, vea [Ejemplos de límite de rendimiento de los volúmenes en un grupo de capacidad de QoS manual](azure-netapp-files-service-levels.md#throughput-limit-examples-of-volumes-in-a-manual-qos-capacity-pool).  

## <a name="how-to-specify-the-manual-qos-type"></a>Procedimientos para especificar el tipo de QoS manual

Al [crear un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md), puede especificar que el grupo de capacidad use el tipo de QoS manual.  También puede [cambiar un grupo de capacidad existente](manage-manual-qos-capacity-pool.md#change-to-qos) para que use el tipo de QoS manual. 

Establecer el tipo de capacidad en QoS manual es un cambio permanente. No se puede convertir un grupo de capacidad de QoS manual en uno automático. 

## <a name="next-steps"></a>Pasos siguientes

* [Administración de un grupo de capacidad de QoS manual](manage-manual-qos-capacity-pool.md)
* [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
* [Jerarquía de almacenamiento](azure-netapp-files-understand-storage-hierarchy.md) 
* [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Consideraciones sobre el rendimiento de Azure NetApp Files](azure-netapp-files-performance-considerations.md)
* [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Creación de un volumen NFS](azure-netapp-files-create-volumes.md)
* [Creación de un volumen SMB](azure-netapp-files-create-volumes-smb.md)
* [Creación de un volumen de protocolo dual](create-volumes-dual-protocol.md)
* [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md)
* [Solución de problemas de grupos de capacidad](troubleshoot-capacity-pools.md)
