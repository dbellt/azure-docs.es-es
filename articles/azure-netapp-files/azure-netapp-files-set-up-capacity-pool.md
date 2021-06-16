---
title: Configuración de un grupo de capacidad de Azure NetApp Files | Microsoft Docs
description: Describe cómo configurar un grupo de capacidad para poder crear volúmenes en él.
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
ms.openlocfilehash: e49a68ee4c4e4f60d8af6919e4eaec33ac0086aa
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071918"
---
# <a name="set-up-a-capacity-pool"></a>Configuración de un grupo de capacidad

Configurar un grupo de capacidad le permite crear volúmenes en él.  

## <a name="before-you-begin"></a>Antes de empezar 

Debe haber creado ya una cuenta de NetApp.   

[Creación de una cuenta de NetApp](azure-netapp-files-create-netapp-account.md)

## <a name="steps"></a>Pasos 

1. Vaya a la hoja de administración de su cuenta de NetApp y, luego, en el panel de navegación, seleccione **Capacity pools** (Grupos de capacidad).  
    
    ![Ir al grupo de capacidad](../media/azure-netapp-files/azure-netapp-files-navigate-to-capacity-pool.png)

2. Haga clic en **+ Agregar grupos** para crear un nuevo grupo de capacidad.   
    Aparecerá el cuadro de diálogo Nuevo grupo de capacidad.

3. Proporcione la siguiente información para el nuevo grupo de capacidad:  
   * **Nombre**  
     Especifique el nombre para el grupo de capacidad.  
     El nombre del grupo de capacidad debe ser único para cada cuenta de NetApp.

   * **Nivel de servicio**   
     Este campo muestra el rendimiento de destino para el grupo de capacidad.  
     Especifique el nivel de servicio del grupo de capacidad: [**Ultra**](azure-netapp-files-service-levels.md#Ultra), [**Premium**](azure-netapp-files-service-levels.md#Premium) o [**Estándar**](azure-netapp-files-service-levels.md#Standard).

    * **Tamaño**     
     Especifique el tamaño del grupo de capacidad que va a adquirir.        
     El tamaño mínimo del grupo de capacidad es de 4 TiB. Puede crear un grupo con un tamaño que sea múltiplo de 4 TiB.   

   * **QoS**   
     Especifique si el grupo de capacidad debe usar el tipo de QoS **manual** o **automático**.  

     Consulte [Jerarquía de almacenamiento](azure-netapp-files-understand-storage-hierarchy.md) y [Consideraciones de rendimiento](azure-netapp-files-performance-considerations.md) para comprender los tipos de QoS.  

     > [!IMPORTANT] 
     > Establezca **Tipo de QoS** en **Manual** si es permanente. No se puede convertir un grupo de capacidad de QoS manual para que use el QoS automático. Sin embargo, puede convertir un grupo de capacidad de QoS automático para que utilice QoS manual. Vea [Cambio de un grupo de capacidad para usar QoS manual](manage-manual-qos-capacity-pool.md#change-to-qos).   

    ![Nuevo grupo de capacidad](../media/azure-netapp-files/azure-netapp-files-new-capacity-pool.png)

4. Haga clic en **Crear**.

## <a name="next-steps"></a>Pasos siguientes 

- [Jerarquía de almacenamiento](azure-netapp-files-understand-storage-hierarchy.md) 
- [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Página de precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
- [Administración de un grupo de capacidad de QoS manual](manage-manual-qos-capacity-pool.md)
- [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md)
