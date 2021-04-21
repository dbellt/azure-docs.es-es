---
title: Obtención de notificaciones de mantenimiento con la CLI
description: Use la CLI de Azure para ver las notificaciones de mantenimiento de máquinas virtuales que se ejecutan en Azure e iniciar el mantenimiento de autoservicio.
author: shants123
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: d8a9b7ec6425a3cd32b597c3f14f8227fde67064
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777876"
---
# <a name="handling-planned-maintenance-notifications-using-the-azure-cli"></a>Control de las notificaciones de mantenimiento planeado con la CLI de Azure

**Este artículo es válido para máquinas virtuales que se ejecutan tanto en Linux como en Windows.**

La CLI se puede usar para ver cuándo está programado el [mantenimiento](maintenance-notifications.md) de las máquinas virtuales. La información de mantenimiento planeado está disponible en [azure vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view).
 
La información de mantenimiento se devuelve solo si hay mantenimiento planeado. 

```azurecli-interactive
az vm get-instance-view -n myVM -g myResourceGroup --query instanceView.maintenanceRedeployStatus
```

Resultados
```
      "maintenanceRedeployStatus": {
      "additionalProperties": {},
      "isCustomerInitiatedMaintenanceAllowed": true,
      "lastOperationMessage": null,
      "lastOperationResultCode": "None",
      "maintenanceWindowEndTime": "2018-06-04T16:30:00+00:00",
      "maintenanceWindowStartTime": "2018-05-21T16:30:00+00:00",
      "preMaintenanceWindowEndTime": "2018-05-19T12:30:00+00:00",
      "preMaintenanceWindowStartTime": "2018-05-14T12:30:00+00:00"
```

## <a name="start-maintenance"></a>Iniciar mantenimiento

La siguiente llamada iniciará el mantenimiento en una máquina virtual si `IsCustomerInitiatedMaintenanceAllowed` está establecido en true.

```azurecli-interactive
az vm perform-maintenance -g myResourceGroup -n myVM 
```

## <a name="classic-deployments"></a>Implementaciones clásicas

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Si todavía tiene máquinas virtuales heredadas que se implementaron según el modelo de implementación clásica, puede usar la CLI de Azure clásica para consultar las máquinas virtuales e iniciar el mantenimiento.

Asegúrese de que se encuentra en el modo correcto para trabajar con máquinas virtuales clásicas. Para ello, escriba lo siguiente:

```
azure config mode asm
```

Para obtener el estado de mantenimiento de una máquina virtual denominada *myVM*, escriba lo siguiente:

```
azure vm show myVM 
``` 

Para iniciar el mantenimiento en una máquina virtual clásica denominada *myVM* en el servicio *myService* y la implementación *myDeployment*, escriba lo siguiente:

```
azure compute virtual-machine initiate-maintenance --service-name myService --name myDeployment --virtual-machine-name myVM
```

## <a name="next-steps"></a>Pasos siguientes

El mantenimiento planeado también se puede controlar con [Azure PowerShell](maintenance-notifications-powershell.md) o con [Portal](maintenance-notifications-portal.md).
