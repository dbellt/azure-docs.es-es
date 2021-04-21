---
title: Protección de las instancias del conjunto de escalado de máquinas virtuales de Azure
description: Obtenga información sobre cómo proteger instancias del conjunto de escalado de máquinas virtuales de Azure contra operaciones de reducción horizontal y conjunto de escalado.
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: instance-protection
ms.date: 02/26/2020
ms.reviewer: jushiman
ms.custom: avverma
ms.openlocfilehash: 292abce3361c000eeeef2c399d5ffa2d2c4852e1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762864"
---
# <a name="instance-protection-for-azure-virtual-machine-scale-set-instances"></a>Protección de las instancias del conjunto de escalado de máquinas virtuales de Azure

Los conjuntos de escalado de máquinas virtuales de Azure permiten una mejor elasticidad para las cargas de trabajo mediante [Escalabilidad automática](virtual-machine-scale-sets-autoscale-overview.md), de manera que puede realizar configuraciones cuando la infraestructura se escala horizontalmente y cuando se reduce horizontalmente. Los conjuntos de escalado permiten administrar, configurar y actualizar de manera central una gran cantidad de máquinas virtuales mediante distintas configuraciones de [directiva de actualización](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model). Puede configurar una actualización en el modelo del conjunto de escalado y la nueva configuración se aplica automáticamente a cada instancia del conjunto de escalado si se ha configurado la directiva de actualización en Automática o Revirtiendo.

Como la aplicación procesa el tráfico, puede haber situaciones en las que quiera que instancias específicas se tratan de forma distinta al resto de la instancia del conjunto de escalado. Por ejemplo, puede que determinadas instancias del conjunto de escalado realicen operaciones de larga duración y que no quiera que esas instancias se reduzcan horizontalmente hasta que finalicen las operaciones. Asimismo, puede que tenga unas cuantas instancias especializadas en el conjunto de escalado para realizar tareas adicionales o diferentes que otros miembros del conjunto de escalado. Necesita que esas máquinas virtuales "especiales" no se modifiquen con las demás instancias del conjunto de escalado. La protección de instancias proporciona los controles adicionales que permiten estos y otros escenarios para la aplicación.

En este artículo se describe cómo se pueden aplicar y usar las diferentes capacidades de protección de instancias con instancias del conjunto de escalado.

## <a name="types-of-instance-protection"></a>Tipos de protección de instancias
Los conjuntos de escalado proporcionan dos tipos de capacidades de protección de instancias:

-   **Protección contra la reducción horizontal**
    - Habilitada a través de la propiedad **protectFromScaleIn** en la instancia del conjunto de escalado.
    - Protege la instancia contra la reducción horizontal iniciada por la escalabilidad automática.
    - Las operaciones de instancia iniciadas por el usuario (incluida la eliminación de la instancia) **no están bloqueadas**.
    - Las operaciones iniciadas en el conjunto de escalado (actualizar, restablecer imagen inicial, desasignar, etc.) no **están bloqueadas**.

-   **Protección contra las acciones del conjunto de escalado**
    - Habilitada a través de la propiedad **protectFromScaleSetActions** en la instancia del conjunto de escalado.
    - Protege la instancia contra la reducción horizontal iniciada por la escalabilidad automática.
    - Protege la instancia contra las operaciones iniciadas en el conjunto de escalado, como actualizar, restablecer imagen inicial, desasignar, etc.
    - Las operaciones de instancia iniciadas por el usuario (incluida la eliminación de la instancia) **no están bloqueadas**.
    - La eliminación del conjunto de escalado completo **no está bloqueada**.

## <a name="protect-from-scale-in"></a>Protección contra la reducción horizontal
La protección de instancias se puede aplicar a instancias del conjunto de escalado después de crear las instancias. La protección se aplica y modifica solo en el [modelo de la instancia](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-vm-model-view), no en el [modelo del conjunto de escalado](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-model).

Hay varias maneras de aplicar la protección contra la reducción horizontal en las instancias del conjunto de escalado, tal como se describe en los siguientes ejemplos.

### <a name="azure-portal"></a>Azure portal

Puede aplicar la protección de la reducción horizontal mediante Azure Portal a una instancia del conjunto de escalado. No se puede ajustar más de una instancia a la vez. Repita los pasos para cada instancia que quiera proteger.
 
1. Vaya a un conjunto de escalado de máquinas virtuales existente.
1. Seleccione **Instancias** en el menú de la izquierda, en **Configuración**.
1. Seleccione el nombre de la instancia que desea proteger.
1. Seleccione la pestaña **Directiva de protección**.
1. En la hoja **Directiva de protección**, seleccione la opción **Proteger de la reducción horizontal**.
1. Seleccione **Guardar**. 

### <a name="rest-api"></a>API DE REST

En el siguiente ejemplo se aplica la protección contra la reducción horizontal a una instancia del conjunto de escalado.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualMachines/{instance-id}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "protectionPolicy": {
      "protectFromScaleIn": true
    }
  }        
}

```

> [!NOTE]
>La protección de instancias solo se admite con la versión de API 2019-03-01 y superiores.

### <a name="azure-powershell"></a>Azure PowerShell

Use el cmdlet [Update-AzVmssVM](/powershell/module/az.compute/update-azvmssvm) para aplicar la protección contra la reducción horizontal en su instancia del conjunto de escalado.

En el siguiente ejemplo se aplica la protección contra la reducción horizontal a una instancia del conjunto de escalado con Id. de instancia 0.

```azurepowershell-interactive
Update-AzVmssVM `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myVMScaleSet" `
  -InstanceId 0 `
  -ProtectFromScaleIn $true
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0

Use [az vmss update](/cli/azure/vmss#az_vmss_update) para aplicar la protección de reducción horizontal en la instancia del conjunto de escalado.

En el siguiente ejemplo se aplica la protección contra la reducción horizontal a una instancia del conjunto de escalado con Id. de instancia 0.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --instance-id 0 \
  --protect-from-scale-in true
```

## <a name="protect-from-scale-set-actions"></a>Protección contra las acciones del conjunto de escalado
La protección de instancias se puede aplicar a instancias del conjunto de escalado después de crear las instancias. La protección se aplica y modifica solo en el [modelo de la instancia](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-vm-model-view), no en el [modelo del conjunto de escalado](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-model).

La protección de una instancia contra las acciones del conjunto de escalado también protege la instancia contra la reducción horizontal iniciada por la escalabilidad automática.

Hay varias maneras de aplicar la protección contra las acciones del conjunto de escalado en las instancias del conjunto de escalado, tal como se describe en los siguientes ejemplos.

### <a name="azure-portal"></a>Azure portal

Puede aplicar la protección contra las acciones del conjunto de escalado mediante Azure Portal a una instancia del conjunto de escalado. No se puede ajustar más de una instancia a la vez. Repita los pasos para cada instancia que quiera proteger.
 
1. Vaya a un conjunto de escalado de máquinas virtuales existente.
1. Seleccione **Instancias** en el menú de la izquierda, en **Configuración**.
1. Seleccione el nombre de la instancia que desea proteger.
1. Seleccione la pestaña **Directiva de protección**.
1. En la hoja **Directiva de protección**, seleccione la opción **Proteger frente a acciones del conjunto de escalado**.
1. Seleccione **Guardar**. 

### <a name="rest-api"></a>API DE REST

En el siguiente ejemplo se aplica la protección contra las acciones del conjunto de escalado a una instancia del conjunto de escalado.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vMScaleSetName}/virtualMachines/{instance-id}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "protectionPolicy": {
      "protectFromScaleIn": true,
      "protectFromScaleSetActions": true
    }
  }        
}

```

> [!NOTE]
>La protección de instancias solo se admite con la versión de API 2019-03-01 y superiores.</br>
La protección de una instancia contra las acciones del conjunto de escalado también protege la instancia contra la reducción horizontal iniciada por la escalabilidad automática. No se puede especificar "protectFromScaleIn": false al configurar "protectFromScaleSetActions": true

### <a name="azure-powershell"></a>Azure PowerShell

Use el cmdlet [Update-AzVmssVM](/powershell/module/az.compute/update-azvmssvm) para aplicar la protección contra las acciones del conjunto de escalado a su instancia del conjunto de escalado.

En el siguiente ejemplo se aplica la protección contra las acciones del conjunto de escalado a una instancia del conjunto de escalado con Id. de instancia 0.

```azurepowershell-interactive
Update-AzVmssVM `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myVMScaleSet" `
  -InstanceId 0 `
  -ProtectFromScaleIn $true `
  -ProtectFromScaleSetAction $true
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0

Use [az vmss update](/cli/azure/vmss#az_vmss_update) para aplicar la protección contra las acciones del conjunto de escalado a su instancia del conjunto de escalado.

En el siguiente ejemplo se aplica la protección contra las acciones del conjunto de escalado a una instancia del conjunto de escalado con Id. de instancia 0.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --instance-id 0 \
  --protect-from-scale-in true \
  --protect-from-scale-set-actions true
```

## <a name="troubleshoot"></a>Solución de problemas
### <a name="no-protectionpolicy-on-scale-set-model"></a>No se aplica protectionPolicy en el modelo del conjunto de escalado
La protección de instancias solo se aplica en instancias del conjunto de escalado y no en el modelo del conjunto de escalado.

### <a name="no-protectionpolicy-on-scale-set-instance-model"></a>No se aplica protectionPolicy en el modelo de la instancia del conjunto de escalado.
De manera predeterminada, la directiva de protección no se aplica a una instancia cuando esta se crea.

Puede aplicar la protección de instancias a instancias del conjunto de escalado después de crearlas.

### <a name="not-able-to-apply-instance-protection"></a>Not se puede aplicar la protección de instancias
La protección de instancias solo se admite con la versión de API 2019-03-01 y superiores. Compruebe la versión de API que está usando y actualícela según sea necesario. Puede que también deba actualizar PowerShell o CLI a la versión más reciente.

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [implementar la aplicación](virtual-machine-scale-sets-deploy-app.md) en conjuntos de escalado de máquinas virtuales.
