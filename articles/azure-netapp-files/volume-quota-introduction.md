---
title: Descripción de la cuota de volumen para Azure NetApp Files | Microsoft Docs
description: Se proporciona información general sobre la cuota de volumen. También se proporcionan referencias sobre la supervisión y la administración de la capacidad del volumen y del grupo.
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
ms.openlocfilehash: d648630e02cbf8c1e7c771f77a3d2342fd9731f7
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294188"
---
# <a name="understand-volume-quota"></a>Descripción de la cuota de volumen

En este artículo se proporciona información general sobre la cuota de volumen para Azure NetApp Files. También se proporcionan referencias a detalles que pueden ayudarle a supervisar y administrar la capacidad de un grupo de capacidad o de un volumen.  

## <a name="behaviors-of-volume-quota"></a>Comportamientos de la cuota de volumen 

* La capacidad de almacenamiento de un volumen de Azure NetApp Files está limitada al tamaño establecido (cuota) del volumen. 

* Cuando el consumo de volumen alcanza el máximo, ni el volumen ni el grupo de capacidad subyacente crecen automáticamente. En su lugar, el volumen recibirá una condición de "fuera de espacio". Sin embargo, puede [cambiar el tamaño del grupo de capacidad o un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md) según sea necesario. Debe [supervisar activamente la capacidad de un volumen](monitor-volume-capacity.md) y del grupo de capacidad subyacente.

* Según el tipo de grupo de capacidad, el tamaño (cuota) de un volumen de Azure NetApp Files afecta al rendimiento de su ancho de banda y a la capacidad aprovisionada.  Para más información, consulte el [tipo de grupo de QoS automático](azure-netapp-files-understand-storage-hierarchy.md#qos_types). 

* La capacidad consumida por las [instantáneas](snapshots-introduction.md) cuenta para el espacio aprovisionado del volumen. 

* La cuota de volumen no se aplica a un [volumen de destino de replicación](cross-region-replication-introduction.md).

* Consulte [Modelo de costos de Azure NetApp Files](azure-netapp-files-cost-model.md) para información sobre el cálculo del consumo de capacidad y el uso por encima del límite.

## <a name="next-steps"></a>Pasos siguientes

* [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Supervisión de la capacidad de un volumen](monitor-volume-capacity.md)
* [Cambio de tamaño de un grupo de capacidad o de un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md)
* [Preguntas más frecuentes sobre la administración de la capacidad](azure-netapp-files-faqs.md#capacity-management-faqs)