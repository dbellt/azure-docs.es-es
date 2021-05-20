---
title: Modo de costo de Azure NetApp Files | Microsoft Docs
description: Describe el modelo de costo de Azure NetApp Files para administrar los gastos del servicio.
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
ms.date: 04/30/2021
ms.author: b-juche
ms.openlocfilehash: 2df332450c996c1a7b1b9b5e35b06d4fb226ed93
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108794473"
---
# <a name="cost-model-for-azure-netapp-files"></a>Modelo de costo de Azure NetApp Files 

Entender el modelo de costo de Azure NetApp Files ayuda a administrar los gastos del servicio. 

Para el modelo de costos específico de la replicación entre regiones, consulte [Modelo de costos de la replicación entre regiones](cross-region-replication-introduction.md#cost-model-for-cross-region-replication).

## <a name="calculation-of-capacity-consumption"></a>Cálculo del consumo de la capacidad

Azure NetApp Files se factura en función de la capacidad de almacenamiento aprovisionada.  La capacidad aprovisionada se asigna mediante la creación de grupos de capacidad.  Los grupos de capacidad se facturan en función del importe en dólares, la cantidad aprovisionada de GiB y el mes en incrementos de una hora. El tamaño mínimo de un grupo de capacidad es de 4 TiB, y los grupos de capacidad se pueden expandir posteriormente en incrementos de 1 TiB. Los volúmenes se crean dentro de grupos de capacidad.  A cada volumen se le asigna una cuota que disminuye a partir de la capacidad aprovisionada de los grupos. La cuota que se puede asignar a los volúmenes oscila entre un mínimo de 100 GiB y un máximo de 100 TiB.  

Para un volumen activo, el consumo de capacidad frente a la cuota se basa en la capacidad lógica (efectiva), ya sea en el sistema de archivos activo o en los datos de instantáneas. Un volumen solo puede contener tantos datos como indique el tamaño establecido (cuota).

La capacidad total utilizada en un grupo de capacidad con respecto a la cantidad aprovisionada es la suma del consumo real de todos los volúmenes del grupo: 

   ![Expresión que muestra el cálculo de la capacidad total utilizada.](../media/azure-netapp-files/azure-netapp-files-total-used-capacity.png)

## <a name="capacity-consumption-of-snapshots"></a>Consumo de capacidad de instantáneas 

El consumo de la capacidad de instantáneas de Azure NetApp Files se cobra en función de la cuota del volumen primario.  Como resultado, comparte la misma tarifa de facturación que el grupo de capacidad al que pertenece el volumen.  Pero, a diferencia del volumen activo, el consumo de instantáneas se mide según la capacidad incremental consumida.  Las instantáneas de Azure NetApp Files son diferenciales por naturaleza. Según la velocidad de cambio de los datos, las instantáneas a menudo consumen mucha menos capacidad que la capacidad lógica del volumen activo. Por ejemplo, imagine que tiene una instantánea de un volumen de 500 GiB que solo contiene 10 GiB de datos diferenciales. El consumo de capacidad que se cuenta para la cuota de volumen del sistema de archivos activo y la instantánea sería de 510 GiB, no de 1000 GiB. Como norma general, se puede suponer que una capacidad recomendada del 20 % conserva los datos de instantáneas durante una semana (en función de la frecuencia de las instantáneas y las frecuencias de cambio diarias en el nivel de bloque de la aplicación). 

En el siguiente diagrama, se ilustran estos conceptos. 

* Supongamos que hay un grupo de capacidad con 40 TiB de capacidad aprovisionada. El grupo contiene tres volúmenes:    
    * Al volumen 1 se le asigna una cuota de 20 TiB y tiene 13 TiB de consumo (12 TiB activos, 1 TiB de instantáneas).
    * Al volumen 2 se le asigna una cuota de 1 TiB y tiene 450 GiB de consumo.
    * Al volumen 3 se le asigna una cuota de 14 TiB, pero tiene 8,8 TiB de consumo (8 TiB activos, 800 GiB de instantáneas).   
* El grupo de capacidad se mide para 40 TiB de capacidad (la cantidad aprovisionada). Se consumen 22,25 TiB de capacidad (13 TiB, 450 GiB y 8,8 TiB de la cuota de los volúmenes 1, 2 y 3). El grupo de capacidad tiene 17,75 TiB de capacidad restante.   

![Diagrama que muestra el grupo de capacidad con tres volúmenes.](../media/azure-netapp-files/azure-netapp-files-capacity-pool-with-three-vols.png)

## <a name="next-steps"></a>Pasos siguientes

* [Página de precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
* [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Modelo de costos de la replicación entre regiones](cross-region-replication-introduction.md#cost-model-for-cross-region-replication)
* [Descripción de la cuota de volumen](volume-quota-introduction.md)
* [Supervisión de la capacidad de un volumen](monitor-volume-capacity.md)
* [Cambio de tamaño de un grupo de capacidad o de un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md)
* [Administración de facturación mediante etiquetas](manage-billing-tags.md)
* [Preguntas más frecuentes sobre la administración de la capacidad](azure-netapp-files-faqs.md#capacity-management-faqs)
