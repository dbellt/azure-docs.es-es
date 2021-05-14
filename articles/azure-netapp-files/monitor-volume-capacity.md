---
title: Supervisión de la capacidad de un volumen de Azure NetApp Files | Microsoft Docs
description: Describe maneras de supervisar el uso de la capacidad de un volumen de Azure NetApp Files.
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
ms.openlocfilehash: 43605f8dfdcac8fe545e5464c13d9a50ae8ed82c
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294260"
---
# <a name="monitor-the-capacity-of-a-volume"></a>Supervisión de la capacidad de un volumen  

En este artículo se describen maneras de supervisar el uso de la capacidad de un volumen de Azure NetApp Files.  

## <a name="using-windows-or-linux-clients"></a>Uso de clientes Windows o Linux

En esta sección se muestra cómo usar un cliente Windows o Linux para supervisar la capacidad del volumen. En los escenarios descritos en esta sección se supone que hay un volumen configurado con un tamaño de 1 TiB (cuota) en un grupo de capacidad de nivel de servicio Ultra de 4 TiB. 

### <a name="windows-smb-clients"></a>Clientes de Windows (SMB)

Puede usar clientes de Windows para comprobar la capacidad usada y disponible de un volumen mediante las propiedades de la unidad asignada de la red. Puede utilizar uno de los dos siguientes métodos: 

* Vaya a Explorador de archivos, haga clic con el botón derecho en la unidad asignada y seleccione **Propiedades** para mostrar la capacidad.  

    [ ![Captura de pantalla que muestra las propiedades de la unidad del Explorador y las propiedades del volumen. ](../media/azure-netapp-files/monitor-explorer-drive-properties.png) ](../media/azure-netapp-files/monitor-explorer-drive-properties.png#lightbox)

* Ejecute el comando `dir` en el símbolo del sistema: 

    ![Captura de pantalla que muestra el uso del comando dir para mostrar la capacidad.](../media/azure-netapp-files/monitor-volume-properties-dir-command.png) 

El *espacio disponible* es preciso mediante el Explorador de archivos o el comando `dir`. Sin embargo, el *espacio consumido o usado* será una estimación cuando se generen instantáneas en el volumen. La [capacidad de instantánea consumida](azure-netapp-files-cost-model.md#capacity-consumption-of-snapshots) cuenta para el espacio total consumido en el volumen. Para obtener el consumo de volumen absoluto, incluida la capacidad que usan las instantáneas, use las [métricas de Azure NetApp](azure-netapp-files-metrics.md#volumes) en Azure Portal. 

### <a name="linux-nfs-clients"></a>Clientes de Linux (NFS) 

Los clientes de Linux pueden comprobar la capacidad usada y disponible de un volumen mediante el [comando df](https://linux.die.net/man/1/df).  

La opción `-h` muestra el tamaño, incluido el espacio usado y el espacio disponible en formato legible (con tamaños de unidad M, G y T).

En la siguiente instantánea se muestra la información sobre capacidad del volumen en Linux:

![Captura de pantalla que muestra los informes de capacidad de volumen en Linux.](../media/azure-netapp-files/monitor-volume-properties-linux-command.png) 

El *espacio disponible* es preciso mediante el comando `df`. Sin embargo, el *espacio consumido o usado* será una estimación cuando se generen instantáneas en el volumen. La [capacidad de instantánea consumida](azure-netapp-files-cost-model.md#capacity-consumption-of-snapshots) cuenta para el espacio total consumido en el volumen. Para obtener el consumo de volumen absoluto, incluida la capacidad que usan las instantáneas, use las [métricas de Azure NetApp](azure-netapp-files-metrics.md#volumes) en Azure Portal. 

## <a name="using-azure-portal"></a>Uso de Azure Portal
Azure NetApp Files aprovecha la funcionalidad de [Azure Monitor](/azure/azure-monitor/overview) estándar. Por lo tanto, puede usar Azure Monitor para supervisar volúmenes de Azure NetApp Files.  

## <a name="using-azure-cli"></a>Uso de la CLI de Azure  

Puede usar los comandos [`az netappfiles volume`](/cli/azure/netappfiles/volume?view=azure-cli-latest&preserve-view=true) de las [herramientas de la línea de comandos (CLI) de Azure](azure-netapp-files-sdk-cli.md) para supervisar un volumen.
 
## <a name="using-rest-api"></a>Uso de la API de REST  

Consulte [API REST para Azure NetApp Files](azure-netapp-files-develop-with-rest-api.md) y [API REST con PowerShell para Azure NetApp Files](develop-rest-api-powershell.md). 

La especificación de la API REST y el código de ejemplo de Azure NetApp Files están disponibles a través del [directorio de GitHub del administrador de recursos](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager/Microsoft.NetApp/stable). 

## <a name="next-steps"></a>Pasos siguientes

* [Descripción de la cuota de volumen](volume-quota-introduction.md)
* [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Cambio de tamaño de un grupo de capacidad o de un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md)
* [Preguntas más frecuentes sobre la administración de la capacidad](azure-netapp-files-faqs.md#capacity-management-faqs)