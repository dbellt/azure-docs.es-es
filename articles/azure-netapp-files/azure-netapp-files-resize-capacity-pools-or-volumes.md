---
title: Cambiar el tamaño de un grupo de capacidad o de un volumen de Azure NetApp Files  | Microsoft Docs
description: Aprenda a cambiar el tamaño de un grupo de capacidad o de un volumen. Al cambiar el tamaño del grupo de capacidad, cambia la capacidad adquirida de Azure NetApp Files.
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
ms.date: 04/30/2021
ms.author: b-juche
ms.openlocfilehash: b57997408112466c45d6ce1364e1ac0a2c358cd1
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321040"
---
# <a name="resize-a-capacity-pool-or-a-volume"></a>Cambiar el tamaño de un grupo de capacidad o de un volumen
Puede cambiar el tamaño de un grupo de capacidad o un volumen según sea necesario, por ejemplo, cuando se llena un volumen o un grupo de capacidad. 

Para obtener información sobre cómo supervisar la capacidad de un volumen, vea [Supervisión de la capacidad de un volumen](monitor-volume-capacity.md).

## <a name="resize-the-capacity-pool-using-the-azure-portal"></a>Cambio del tamaño del grupo de capacidad mediante Azure Portal 

Puede cambiar el tamaño del grupo de capacidad en incrementos o decrementos de 1 TiB. Aun así, el tamaño del grupo de capacidad no puede ser inferior a 4 TiB. Al cambiar el tamaño del grupo de capacidad, cambia la capacidad adquirida de Azure NetApp Files.

1. En la vista Cuenta de NetApp, vaya a **Grupos de capacidad** y haga clic en el grupo de capacidad al que desea cambiar el tamaño.
2. Haga clic con el botón derecho en el nombre del grupo de capacidad o haga clic en el icono "..." al final de la fila del grupo de capacidad para mostrar el menú contextual. Haga clic en **Cambiar tamaño**. 

    ![Captura de pantalla que muestra el menú contextual del grupo.](../media/azure-netapp-files/resize-pool-context-menu.png)  

3. En la ventana Cambiar el tamaño del grupo, especifique el tamaño del grupo.  Haga clic en **Aceptar**.

    ![Captura de pantalla que muestra la ventana para cambiar el tamaño del grupo.](../media/azure-netapp-files/resize-pool-window.png) 

## <a name="resize-a-volume-using-the-azure-portal"></a>Cambio del tamaño de un volumen mediante Azure Portal

Puede cambiar el tamaño de un volumen según sea necesario. El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo.

1. En la vista Cuenta de NetApp, vaya a **Volúmenes** y haga clic en el volumen al que desea cambiar el tamaño.
2. Haga clic con el botón derecho en el nombre del volumen o haga clic en el icono "..." al final de la fila del volumen para mostrar el menú contextual. Haga clic en **Cambiar tamaño**.

    ![Captura de pantalla que muestra el menú contextual del volumen.](../media/azure-netapp-files/resize-volume-context-menu.png) 
    
3. En la ventana Actualizar cuota de volumen, especifique la cuota del volumen. Haga clic en **Aceptar**.   

    ![Captura de pantalla que muestra la ventana para actualizar la cuota de volumen.](../media/azure-netapp-files/resize-volume-quota-window.png) 

## <a name="resizing-the-capacity-pool-or-a-volume-using-azure-cli"></a>Cambio del tamaño del grupo de capacidad o un volumen mediante la CLI de Azure  

Puede usar los siguientes comandos de las [herramientas de la línea de comandos (CLI) de Azure](azure-netapp-files-sdk-cli.md) para cambiar el tamaño de un grupo de capacidad o un volumen:

* [`az netappfiles pool`](/cli/azure/netappfiles/pool?preserve-view=true&view=azure-cli-latest)
* [`az netappfiles volume`](/cli/azure/netappfiles/volume?preserve-view=true&view=azure-cli-latest)

## <a name="resizing-the-capacity-pool-or-a-volume-using-rest-api"></a>Cambio del tamaño del grupo de capacidad o un volumen mediante la API REST

Puede crear una automatización para controlar el cambio del tamaño del grupo de capacidad y el volumen.   

Consulte [API REST para Azure NetApp Files](azure-netapp-files-develop-with-rest-api.md) y [API REST con PowerShell para Azure NetApp Files](develop-rest-api-powershell.md). 

La especificación de la API REST y el código de ejemplo de Azure NetApp Files están disponibles a través del [directorio de GitHub del administrador de recursos](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager/Microsoft.NetApp/stable). 

## <a name="resize-a-cross-region-replication-destination-volume"></a>Cambio de tamaño de un volumen de destino de replicación entre regiones 

En una relación de [replicación entre regiones](cross-region-replication-introduction.md), el tamaño de un volumen de destino se cambia automáticamente en función del tamaño del volumen de origen. De este modo, no es necesario cambiar el tamaño del volumen de destino por separado. Este cambio de tamaño automático es aplicable cuando los volúmenes establecen una relación de replicación activa, o cuando el emparejamiento de replicación se interrumpe con la [operación de resincronización](cross-region-replication-manage-disaster-recovery.md#resync-replication). 

En la tabla siguiente se describe el comportamiento de cambio de tamaño del volumen de destino en función del [estado del reflejo](cross-region-replication-display-health-status.md):

|  Estado del reflejo  | Comportamiento de cambio de tamaño del volumen de destino |
|-|-|
| *Reflejado* | Cuando el volumen de destino se ha inicializado y está listo para recibir actualizaciones de creación de reflejo, el cambio de tamaño del volumen de origen cambia automáticamente el tamaño de los volúmenes de destino. |
| *Interrumpido* | Cuando se cambia el tamaño del volumen de origen y el estado del reflejo se *interrumpe*, el tamaño del volumen de destino se cambia automáticamente con la [operación de resincronización](cross-region-replication-manage-disaster-recovery.md#resync-replication).  |
| *Sin inicializar* | Al cambiar el tamaño del volumen de origen y el estado del reflejo es aún *inicializar*, el cambio de tamaño del volumen de destino debe realizarse manualmente. Como tal, se recomienda esperar a que finalice la inicialización (es decir, cuando el estado del reflejo sea *reflejado*) para cambiar el tamaño del volumen de origen. | 

> [!IMPORTANT]
> Asegúrese de que dispone de suficiente espacio en los grupos de capacidad para los volúmenes de origen y de destino de la replicación entre regiones. Al cambiar el tamaño del volumen de origen, se cambia automáticamente el tamaño del volumen de destino. Sin embargo, si el grupo de capacidad que hospeda el volumen de destino no tiene suficiente espacio, se producirá un error al cambiar de tamaño los volúmenes de origen y de destino.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
- [Administración de un grupo de capacidad de QoS manual](manage-manual-qos-capacity-pool.md)
- [Cambio dinámico del nivel de servicio de un volumen](dynamic-change-volume-service-level.md) 
- [Descripción de la cuota de volumen](volume-quota-introduction.md)
- [Supervisión de la capacidad de un volumen](monitor-volume-capacity.md)
- [Preguntas más frecuentes sobre la administración de la capacidad](azure-netapp-files-faqs.md#capacity-management-faqs)
