---
title: Reparaciones automáticas de instancias con conjuntos de escalado de máquinas virtuales de Azure
description: Aprenda a configurar la directiva de reparaciones automáticas para instancias de máquinas virtuales en un conjunto de escalado.
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: instance-protection
ms.date: 02/28/2020
ms.reviewer: jushiman
ms.custom: avverma, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: af63d27f60c3e3fa8f6aa95cd30d1f93114d4078
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031176"
---
# <a name="automatic-instance-repairs-for-azure-virtual-machine-scale-sets"></a>Reparaciones automáticas de instancias para conjuntos de escalado de máquinas virtuales de Azure

La habilitación de las reparaciones automáticas de instancias para conjuntos de escalado de máquinas virtuales de Azure ayuda a lograr una alta disponibilidad para las aplicaciones al mantener un conjunto de instancias correctas. Si se detecta que una instancia del conjunto de escalado tiene un estado incorrecto según lo indicado por la [extensión de estado de la aplicación](./virtual-machine-scale-sets-health-extension.md) o por los [sondeos de estado del equilibrador de carga](../load-balancer/load-balancer-custom-probe-overview.md), esta característica realiza automáticamente la reparación de la instancia mediante la eliminación de la instancia incorrecta y la creación de una nueva para reemplazarla.

## <a name="requirements-for-using-automatic-instance-repairs"></a>Requisitos para el uso de reparaciones automáticas de instancias

**Habilitación de la supervisión del estado de la aplicación para el conjunto de escalado**

El conjunto de escalado debe tener la supervisión del estado de la aplicación para las instancias habilitadas. Esto puede hacerse mediante la [extensión de estado de la aplicación](./virtual-machine-scale-sets-health-extension.md) o los [sondeos de estado del equilibrador de carga](../load-balancer/load-balancer-custom-probe-overview.md). Solo se puede habilitar uno de ellos a la vez. La extensión de mantenimiento de la aplicación o el equilibrador de carga sondea el punto de conexión de la aplicación configurado en las instancias de máquina virtual para determinar el estado de mantenimiento de la aplicación. El orquestador del conjunto de escalado usa este estado de mantenimiento para supervisar el estado de las instancias y realizar reparaciones cuando sea necesario.

**Configuración del punto de conexión para proporcionar el estado de mantenimiento**

Antes de habilitar la directiva de reparaciones automáticas de instancias, asegúrese de que las instancias del conjunto de escalado tienen configurado el punto de conexión de aplicación para emitir el estado de mantenimiento de la aplicación. Cuando una instancia devuelve el estado 200 (correcto) en este punto de conexión de la aplicación, la instancia se marca con estado "Correcto". En todos los demás casos, la instancia se marca con estado "Incorrecto", incluidos los escenarios siguientes:

- Cuando no hay ningún punto de conexión de aplicación configurado dentro de las instancias de máquina virtual para proporcionar el estado de mantenimiento de la aplicación
- Cuando el punto de conexión de la aplicación no está configurado correctamente
- Cuando no se puede tener acceso al punto de conexión de la aplicación

En el caso de las instancias marcadas con el estado "Incorrecto", el conjunto de escalado desencadena las reparaciones automáticas. Asegúrese de que el punto de conexión de la aplicación está configurado correctamente antes de habilitar la directiva de reparaciones automáticas para evitar reparaciones de instancias no intencionadas mientras se configura el punto de conexión.

**Número máximo de instancias del conjunto de escalado**

Esta característica solo está disponible para los conjuntos de escalado que tienen 500 instancias como máximo. El conjunto de escalado se puede implementar como un solo grupo de selección de ubicación o un grupo de varias ubicaciones; sin embargo, el recuento de instancias no puede ser superior a 500 si está habilitada la reparación automática de instancias para el conjunto de escalado.

**Versión de API**

La directiva de reparaciones automáticas es compatible con Compute API versión 2018-10-01 o superior.

**Restricciones en los movimientos de recursos o suscripciones**

Actualmente no se admiten los movimientos de recursos o suscripciones para los conjuntos de escalado cuando la característica de reparaciones automáticas está habilitada.

**Restricción para los conjuntos de escalado de Service Fabric**

Esta característica no se admite actualmente para los conjuntos de escalado de Service Fabric.

## <a name="how-do-automatic-instance-repairs-work"></a>¿Cómo funciona la reparación automática de instancias?

La característica de reparación automática de instancias se basa en la supervisión del estado de las instancias individuales de un conjunto de escalado. Las instancias de máquina virtual de un conjunto de escalado se pueden configurar para emitir el estado de mantenimiento de la aplicación mediante la [extensión de estado de aplicación](./virtual-machine-scale-sets-health-extension.md) o los [sondeos de estado del equilibrador de carga](../load-balancer/load-balancer-custom-probe-overview.md). Si una instancia se encuentra en un estado incorrecto, el conjunto de escalado realiza la acción de reparación eliminando la instancia incorrecta y creando una nueva para reemplazarla. El modelo de conjunto de escalado de máquinas virtuales más reciente se usa para crear la nueva instancia. Esta característica se puede habilitar en el modelo de conjunto de escalado de máquinas virtuales mediante el objeto *automaticRepairsPolicy*.

### <a name="batching"></a>Lotes

Las operaciones de reparaciones automáticas de instancias se realizan en lotes. En un momento dado, no se reparan más del 5 % de las instancias del conjunto de escalado mediante la directiva de reparaciones automáticas. Esto ayuda a evitar la eliminación simultánea y la nueva creación de un gran número de instancias si se encuentran en estado incorrecta al mismo tiempo.

### <a name="grace-period"></a>Período de gracia

Cuando una instancia pasa por una operación de cambio de estado debido a una acción PUT, PATCH o POST realizada en el conjunto de escalado (por ejemplo, restablecimiento de la imagen, reimplementación, actualización, etc.), se realiza cualquier acción de reparación en esa instancia solo después de esperar el período de gracia. El período de gracia es la cantidad de tiempo que se permite que la instancia vuelva al estado correcto. El período de gracia se inicia después de que se haya completado el cambio de estado. Esto ayuda a evitar cualquier operación de reparación prematura o accidental. El período de gracia se respeta para cualquier instancia recién creada en el conjunto de escalado (incluido el creado como resultado de la operación de reparación). El período de gracia se especifica en minutos en formato ISO 8601 y se puede establecer mediante la propiedad *automaticRepairsPolicy.gracePeriod*. El período de gracia puede oscilar entre 10 minutos y 90 minutos, y tiene un valor predeterminado de 30 minutos.

### <a name="suspension-of-repairs"></a>Suspensión de reparaciones

Los conjuntos de escalado de máquinas virtuales proporcionan la capacidad de suspender temporalmente la reparación automática de instancias si es necesario. El parámetro *serviceState* para las reparaciones automáticas en la propiedad *orchestrationServices* en la vista de instancias del conjunto de escalado de máquinas virtuales muestra el estado actual de las reparaciones automáticas. Cuando un conjunto de escalado participa en reparaciones automáticas, el valor del parámetro *serviceState* se establece en *Running*. Cuando se suspenden las reparaciones automáticas para un conjunto de escalado, el parámetro *serviceState* se establece en *Suspended*. Si se define *automaticRepairsPolicy* en un conjunto de escalado, pero la característica de reparaciones automática no está habilitada, el parámetro *serviceState* se establece en *Not Running*.

Si las instancias que se acaban de crear para reemplazar a las que tienen un estado incorrecto en un conjunto de escalado siguen siendo incorrectas incluso después de realizar varias veces las operaciones de reparación, como medida de seguridad, la plataforma actualiza el parámetro *serviceState* para las reparaciones automáticas a *Suspended*. Puede reanudar de nuevo las reparaciones automáticas si establece el valor de *serviceState* para las reparaciones automáticas en *Running*. En la sección sobre [visualización y actualización del estado del servicio de la directiva de reparaciones automáticas](#viewing-and-updating-the-service-state-of-automatic-instance-repairs-policy) encontrará instrucciones detalladas para su conjunto de escalado.

El proceso de reparaciones automáticas de instancias funciona de la siguiente manera:

1. La [extensión de estado de aplicación](./virtual-machine-scale-sets-health-extension.md) o los [sondeos de estado del equilibrador de carga](../load-balancer/load-balancer-custom-probe-overview.md) hacen ping en el punto de conexión de la aplicación dentro de cada máquina virtual del conjunto de escalado para obtener el estado de mantenimiento de la aplicación para cada instancia.
2. Si el punto de conexión responde con un estado 200 (correcto), la instancia se marca con estado "Correcto". En todos los demás casos (incluido si no se puede tener acceso al punto de conexión), la instancia se marca con estado "Incorrecto".
3. Cuando una instancia se encuentra en un estado incorrecto, el conjunto de escalado desencadena una acción de reparación al eliminar la instancia incorrecta y crear una nueva para reemplazarla.
4. Las reparaciones de instancias se realizan en lotes. En un momento dado, no se reparan más del 5 % del total de instancias del conjunto de escalado. Si un conjunto de escalado tiene menos de 20 instancias, las reparaciones se realizan para una instancia incorrecta a la vez.
5. El proceso anterior continúa hasta que se reparen todas las instancias incorrectas del conjunto de escalado.

## <a name="instance-protection-and-automatic-repairs"></a>Protección de instancias y reparaciones automáticas

Si una instancia de un conjunto de escalado está protegida por la aplicación de una de las [directivas de protección](./virtual-machine-scale-sets-instance-protection.md), las reparaciones automáticas no se realizan en esa instancia. Esto se aplica a ambas directivas de protección: *Proteger de la reducción horizontal* y *Proteger frente a acciones del conjunto de escalado*.

## <a name="terminatenotificationandautomaticrepairs"></a>Notificación de finalización y reparaciones automáticas

Si la característica de [notificación de finalización](./virtual-machine-scale-sets-terminate-notification.md) está habilitada en un conjunto de escalado, durante la operación de reparación automática, la eliminación de una instancia incorrecta sigue a la configuración de la notificación de finalización. Una notificación de finalización se envía a través de Azure Metadata Service —Scheduled Events— y la eliminación de la instancia se retrasa mientras dure el tiempo de espera de retraso configurado. Sin embargo, la creación de una nueva instancia para reemplazar las que tienen un estado incorrecto no espera a que se complete el tiempo de espera de retraso.

## <a name="enabling-automatic-repairs-policy-when-creating-a-new-scale-set"></a>Habilitación de la directiva de reparaciones automáticas al crear un nuevo conjunto de escalado

Para habilitar la directiva de reparaciones automáticas al crear un nuevo conjunto de escalado, asegúrese de que se cumplen todos los [requisitos](#requirements-for-using-automatic-instance-repairs) para optar a esta característica. El punto de conexión de la aplicación debe configurarse correctamente para las instancias del conjunto de escalado a fin de evitar la activación de reparaciones imprevistas mientras se configura el punto de conexión. En el caso de los conjuntos de escalado recién creados, cualquier reparación de instancias se realiza solo después de esperar la duración del período de gracia. Para habilitar la reparación automática de la instancia en un conjunto de escalado, use el objeto *automaticRepairsPolicy* en el modelo del conjunto de escalado de máquinas virtuales.

También puede usar esta [plantilla de inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vmss-automatic-repairs-slb-health-probe) para implementar un conjunto de escalado de máquinas virtuales con el sondeo de estado del equilibrador de carga y la reparación de instancias automática habilitados con un período de gracia de 30 minutos.

### <a name="azure-portal"></a>Portal de Azure

Los pasos siguientes permiten habilitar la directiva de reparaciones automáticas al crear un nuevo conjunto de escalado.

1. Vaya a **Conjuntos de escalado de máquinas virtuales**.
1. Seleccione **+ Agregar** para crear un nuevo conjunto de escalado.
1. Vaya a la pestaña **Mantenimiento**.
1. Busque la sección **Mantenimiento**.
1. Habilite la opción **Supervisar el mantenimiento de la aplicación**.
1. Busque la sección **Directiva de reparación automática**.
1. **Active** la opción **Reparación automática**.
1. En **Período de gracia (min)** , especifique el período de gracia en minutos; los valores permitidos están comprendidos entre 30 y 90 minutos.
1. Cuando haya terminado de crear el nuevo conjunto de escalado, seleccione el botón **Revisar y crear**.

### <a name="rest-api"></a>API DE REST

En el ejemplo siguiente se muestra cómo habilitar la reparación automática de instancias en un modelo de conjunto de escalado. Use la API versión 2018-10-01 o posteriores.

```
PUT or PATCH on '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-07-01'
```

```json
{
  "properties": {
    "automaticRepairsPolicy": {
            "enabled": "true",
            "gracePeriod": "PT30M"
        }
    }
}
```

### <a name="azure-powershell"></a>Azure PowerShell

La característica de reparación automática de instancias se puede habilitar al crear un nuevo conjunto de escalado mediante el cmdlet [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig). Este script de ejemplo le guía por la creación de un conjunto de escalado y los recursos asociados mediante el archivo de configuración: [Creación de un conjunto de escalado de máquinas virtuales completo](./scripts/powershell-sample-create-complete-scale-set.md) Puede configurar la directiva de reparaciones automáticas de instancias al agregar los parámetros *EnableAutomaticRepair* y *AutomaticRepairGracePeriod* al objeto de configuración para crear el conjunto de escalado. En el ejemplo siguiente se habilita la característica con un período de gracia de 30 minutos.

```azurepowershell-interactive
New-AzVmssConfig `
 -Location "EastUS" `
 -SkuCapacity 2 `
 -SkuName "Standard_DS2" `
 -UpgradePolicyMode "Automatic" `
 -EnableAutomaticRepair $true `
 -AutomaticRepairGracePeriod "PT30M"
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0

En el ejemplo siguiente se habilita la directiva de reparaciones automáticas al crear un nuevo conjunto de escalado mediante *[az vmss create](/cli/azure/vmss#az_vmss_create)* . En primer lugar, cree un grupo de recursos y, a continuación, cree un nuevo conjunto de escalado con un período de gracia de la directiva de reparaciones automáticas establecido en 30 minutos.

```azurecli-interactive
az group create --name <myResourceGroup> --location <VMSSLocation>
az vmss create \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --image UbuntuLTS \
  --admin-username <azureuser> \
  --generate-ssh-keys \
  --load-balancer <existingLoadBalancer> \
  --health-probe <existingHealthProbeUnderLoaderBalancer> \
  --automatic-repairs-grace-period 30
```

En el ejemplo anterior se usa un equilibrador de carga y un sondeo de mantenimiento existentes para supervisar el estado de mantenimiento de las aplicaciones en las instancias. Si, en su lugar, prefiere usar una extensión de estado de aplicación para la supervisión, puede crear un conjunto de escalado, configurar la extensión de estado de aplicación y, a continuación, habilitar la directiva de reparaciones automáticas de instancias mediante *az vmss update*, como se explica en la sección siguiente.

## <a name="enabling-automatic-repairs-policy-when-updating-an-existing-scale-set"></a>Habilitación de la directiva de reparaciones automáticas al actualizar un conjunto de escalado existente

Antes de habilitar la directiva de reparaciones automáticas en un conjunto de escalado existente, asegúrese de que se cumplen todos los [requisitos](#requirements-for-using-automatic-instance-repairs) para optar a esta característica. El punto de conexión de la aplicación debe configurarse correctamente para las instancias del conjunto de escalado a fin de evitar la activación de reparaciones imprevistas mientras se configura el punto de conexión. Para habilitar la reparación automática de la instancia en un conjunto de escalado, use el objeto *automaticRepairsPolicy* en el modelo del conjunto de escalado de máquinas virtuales.

Después de actualizar el modelo de un conjunto de escalado existente, asegúrese de que el modelo más reciente se aplique a todas las instancias de la escala. Consulte las instrucciones en [Actualización de las máquinas virtuales con el modelo de conjunto de escalado más reciente](./virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model).

### <a name="azure-portal"></a>Portal de Azure

Puede modificar la directiva de reparaciones automáticas de un conjunto de escalado existente a través de Azure Portal.

1. Vaya a un conjunto de escalado de máquinas virtuales existente.
1. En **Configuración** en el menú de la izquierda, seleccione **Mantenimiento y reparación**.
1. Habilite la opción **Supervisar el mantenimiento de la aplicación**.
1. Busque la sección **Directiva de reparación automática**.
1. **Active** la opción **Reparación automática**.
1. En **Período de gracia (min)** , especifique el período de gracia en minutos; los valores permitidos están comprendidos entre 30 y 90 minutos.
1. Cuando finalice, seleccione **Guardar**.

### <a name="rest-api"></a>API DE REST

En el ejemplo siguiente se habilita la directiva con un período de gracia de 40 minutos. Use la API versión 2018-10-01 o posteriores.

```
PUT or PATCH on '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-07-01'
```

```json
{
  "properties": {
    "automaticRepairsPolicy": {
            "enabled": "true",
            "gracePeriod": "PT40M"
        }
    }
}
```

### <a name="azure-powershell"></a>Azure PowerShell

Use el cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss) para modificar la configuración de la característica de reparación automática de instancias en un conjunto de escalado existente. En el ejemplo siguiente se actualiza el período de gracia a 40 minutos.

```azurepowershell-interactive
Update-AzVmss `
 -ResourceGroupName "myResourceGroup" `
 -VMScaleSetName "myScaleSet" `
 -EnableAutomaticRepair $true `
 -AutomaticRepairGracePeriod "PT40M"
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0

El siguiente es un ejemplo para actualizar la directiva de reparaciones automáticas de instancias de un conjunto de escalado existente, mediante *[az vmss update](/cli/azure/vmss#az_vmss_update)* .

```azurecli-interactive
az vmss update \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --enable-automatic-repairs true \
  --automatic-repairs-grace-period 30
```

## <a name="viewing-and-updating-the-service-state-of-automatic-instance-repairs-policy"></a>Visualización y actualización del estado de servicio de la directiva de reparaciones automáticas de instancias

### <a name="rest-api"></a>API DE REST

Use [Get Instance View](/rest/api/compute/virtualmachinescalesets/getinstanceview) con la API versión 2019-12-01 o superior para conjuntos de escalado de máquinas virtuales para ver el parámetro *serviceState* para las reparaciones automáticas en la propiedad *orchestrationServices*.

```http
GET '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/instanceView?api-version=2019-12-01'
```

```json
{
  "orchestrationServices": [
    {
      "serviceName": "AutomaticRepairs",
      "serviceState": "Running"
    }
  ]
}
```

Use la API *setOrchestrationServiceState* con la API versión 2019-12-01 o superior en un conjunto de escalado de máquinas virtuales para establecer el estado de las reparaciones automáticas. Una vez que el conjunto de escalado participe en la característica de reparaciones automáticas, puede usar esta API para suspender o reanudar las reparaciones automáticas del conjunto de escalado.

 ```http
 POST '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/setOrchestrationServiceState?api-version=2019-12-01'
 ```

```json
{
  "orchestrationServices": [
    {
      "serviceName": "AutomaticRepairs",
      "serviceState": "Suspend"
    }
  ]
}
```

### <a name="azure-cli"></a>Azure CLI

Use el cmdlet [get-instance-view](/cli/azure/vmss#az_vmss_get_instance_view) para ver el parámetro *serviceState* para las reparaciones automáticas de instancias.

```azurecli-interactive
az vmss get-instance-view \
    --name MyScaleSet \
    --resource-group MyResourceGroup
```

Use el cmdlet [set-orchestration-service-state](/cli/azure/vmss#az_vmss_set_orchestration_service_state) para actualizar el parámetro *serviceState* para las reparaciones automáticas de instancias. Una vez que el conjunto de escalado participe en la característica de reparaciones automáticas, puede usar este cmdlet para suspender o reanudar las reparaciones automáticas del conjunto de escalado.

```azurecli-interactive
az vmss set-orchestration-service-state \
    --service-name AutomaticRepairs \
    --action Resume \
    --name MyScaleSet \
    --resource-group MyResourceGroup
```
### <a name="azure-powershell"></a>Azure PowerShell

Use el cmdlet [Get-AzVmss](/powershell/module/az.compute/get-azvmss) con el parámetro *InstanceView* para ver el parámetro *ServiceState* de las reparaciones automáticas de instancias.

```azurepowershell-interactive
Get-AzVmss `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -InstanceView
```

Use el cmdlet Set-AzVmssOrchestrationServiceState para actualizar el parámetro *serviceState* para las reparaciones automáticas de instancias. Una vez que el conjunto de escalado participe en la característica de reparaciones automáticas, puede usar este cmdlet para suspender o reanudar las reparaciones automáticas del conjunto de escalado.

```azurepowershell-interactive
Set-AzVmssOrchestrationServiceState `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -ServiceName "AutomaticRepairs" `
    -Action "Suspend"
```

## <a name="troubleshoot"></a>Solución de problemas

**Error al habilitar la directiva de reparaciones automáticas**

Si recibe un error de "BadRequest" con un mensaje que indica que "No se pudo encontrar el miembro 'automaticRepairsPolicy' en el objeto de tipo 'properties'", compruebe la versión de la API usada para el conjunto de escalado de máquinas virtuales. Se requiere la versión 2018-10-01 de la API o una posterior para esta característica.

**La instancia no se repara incluso cuando la directiva está habilitada**.

La instancia puede estar en el período de gracia. Esta es la cantidad de tiempo que hay que esperar después de cualquier cambio de estado en la instancia antes de realizar reparaciones. Esto es para evitar reparaciones prematuras o accidentales. La acción de reparar debe realizarse una vez completado el período de gracia de la instancia.

**Visualización del estado de mantenimiento de la aplicación para instancias del conjunto de escalado**

Puede usar [Get Instance View API](/rest/api/compute/virtualmachinescalesetvms/getinstanceview) para las instancias de un conjunto de escalado de máquinas virtuales para ver el estado de mantenimiento de la aplicación. Con Azure PowerShell, puede usar el cmdlet [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) con la marca *-InstanceView*. El estado de mantenimiento de la aplicación se proporciona en la propiedad *vmHealth*.

En Azure Portal, también puede ver el estado de mantenimiento. Vaya a un conjunto de escalado existente, seleccione **Instancias** en el menú de la izquierda y examine la columna **Estado de mantenimiento** para ver el estado de mantenimiento de cada instancia del conjunto de escalado.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a configurar la [extensión de estado de la aplicación](./virtual-machine-scale-sets-health-extension.md) o los [sondeos de estado del equilibrador de carga](../load-balancer/load-balancer-custom-probe-overview.md) para los conjuntos de escalado.
