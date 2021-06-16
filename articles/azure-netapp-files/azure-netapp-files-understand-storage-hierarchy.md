---
title: Jerarquía de almacenamiento de Azure NetApp Files | Microsoft Docs
description: Describe la jerarquía del almacenamiento, incluidas las cuentas, grupos de capacidad y volúmenes de Azure NetApp Files.
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
ms.topic: overview
ms.date: 06/14/2021
ms.author: b-juche
ms.openlocfilehash: fd96a8c2002bc22a0e4ef2c5a19fa06bf10b5565
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071900"
---
# <a name="storage-hierarchy-of-azure-netapp-files"></a>Jerarquía de almacenamiento de Azure NetApp Files

Antes de crear un volumen en Azure NetApp Files, debe adquirir y configurar un grupo para la capacidad aprovisionada.  Para configurar un grupo de capacidad, debe tener una cuenta de NetApp. Conocer la jerarquía de almacenamiento le ayuda a configurar y administrar los recursos de Azure NetApp Files.

> [!IMPORTANT] 
> Actualmente, Azure NetApp Files no admite la migración de recursos entre suscripciones.

## <a name="netapp-accounts"></a><a name="azure_netapp_files_account"></a>Cuentas de NetApp

- Una cuenta de NetApp actúa como una agrupación administrativa de los grupos de capacidad constituyentes.  
- No es lo mismo que la cuenta de almacenamiento general de Azure. 
- Una cuenta de NetApp es regional en el ámbito.   
- Puede tener varias cuentas de NetApp en una región, pero cada una está asociada solo a una única región.

## <a name="capacity-pools"></a><a name="capacity_pools"></a>Grupos de capacidad

Comprender cómo funcionan los grupos de capacidad ayuda a seleccionar los tipos de grupo de capacidad adecuados para las necesidades de almacenamiento. 

### <a name="general-rules-of-capacity-pools"></a>Reglas generales de grupos de capacidad

- Un grupo de capacidad se mide por su capacidad aprovisionada.   
    Consulte [Tipos de calidad de servicio](#qos_types) para más información.  
- La capacidad se aprovisiona con las SKU fijas que adquirió (por ejemplo, una capacidad de 4 TiB).
- Un grupo de capacidad puede tener solo un nivel de servicio.  
- Cada grupo de capacidad puede pertenecer a una sola cuenta de NetApp. Sin embargo, puede tener varios grupos de capacidad dentro de una cuenta de NetApp.  
- Un grupo de capacidad no se puede mover entre las cuentas de NetApp.   
  Por ejemplo, en el [Diagrama conceptual de la jerarquía de almacenamiento](#conceptual_diagram_of_storage_hierarchy) a continuación, el grupo de capacidad 1 no se puede mover de la cuenta de NetApp Este de EE. UU. a la cuenta de NetApp Oeste de EE. UU. 2.  
- No se puede eliminar un grupo de capacidad hasta que todos los volúmenes que contiene se hayan eliminado.

### <a name="quality-of-service-qos-types-for-capacity-pools"></a><a name="qos_types"></a>Tipos de calidad de servicio (QoS) de los grupos de capacidad

El tipo de QoS es un atributo de un grupo de capacidad. Azure NetApp Files proporciona dos tipos de QoS de grupos de capacidad. 

- Tipo de QoS *automático* (o auto)  

    Cuando se crea un grupo de capacidad, el tipo de QoS predeterminado es automático.

    En un grupo de capacidad de QoS automático, el rendimiento se asigna automáticamente a los volúmenes del grupo, de forma proporcional a la cuota de tamaño asignada a los volúmenes. 

    El rendimiento máximo asignado a un volumen depende del nivel de servicio del grupo de capacidad y de la cuota de tamaño del volumen. Para ver un ejemplo de cálculo, consulte [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md).

- Tipo de QoS <a name="manual_qos_type"></a>*manual*  

    Tiene la opción de usar el tipo de QoS manual para un grupo de capacidad.

    En un grupo de capacidad de QoS manual, puede asignar la capacidad y el rendimiento de un volumen de forma independiente. El rendimiento total de todos los volúmenes creados con un grupo de capacidad de QoS manual está limitado por el rendimiento total del grupo.  Viene determinado por la combinación del tamaño del grupo y el rendimiento del nivel de servicio. 

    Por ejemplo, un grupo de capacidad de 4 TiB con el nivel de servicio Ultra tiene una capacidad de rendimiento total de 512 MiB/s (4 TiB x 128 MiB/s/TiB) disponible para los volúmenes.


## <a name="volumes"></a><a name="volumes"></a>Volúmenes

- Un volumen se mide según el consumo de capacidad lógica y es escalable. 
- El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo.
- El consumo de rendimiento de un volumen se calcula contra el rendimiento disponible de su grupo. Consulte [Tipo de QoS manual](#manual_qos_type).
- Cada volumen pertenece a un solo grupo, pero un grupo puede contener varios volúmenes. 

## <a name="conceptual-diagram-of-storage-hierarchy"></a><a name="conceptual_diagram_of_storage_hierarchy"></a>Diagrama conceptual de la jerarquía de almacenamiento 
El ejemplo siguiente muestra las relaciones de la suscripción de Azure, las cuentas de NetApp, los grupos de capacidad y los volúmenes.   

![Diagrama conceptual de la jerarquía de almacenamiento](../media/azure-netapp-files/azure-netapp-files-storage-hierarchy.png)

## <a name="next-steps"></a>Pasos siguientes

- [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
- [Registro de Azure NetApp Files](azure-netapp-files-register.md)
- [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Consideraciones sobre el rendimiento de Azure NetApp Files](azure-netapp-files-performance-considerations.md)
- [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
- [Administración de un grupo de capacidad de QoS manual](manage-manual-qos-capacity-pool.md)
