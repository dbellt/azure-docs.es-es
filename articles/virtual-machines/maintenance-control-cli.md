---
title: Control del mantenimiento de máquinas virtuales de Azure mediante la CLI
description: Obtenga información sobre cómo controlar cuándo se aplica mantenimiento a las máquinas virtuales de Azure mediante el control de mantenimiento y la CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/20/2020
ms.author: cynthn
ms.openlocfilehash: c57f66eca5d15024c6b10e8fad12ddb575b9f894
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765906"
---
# <a name="control-updates-with-maintenance-control-and-the-azure-cli"></a>Control de las actualizaciones con el control de mantenimiento y la CLI de Azure

El control de mantenimiento permite decidir cuándo aplicar las actualizaciones de plataforma a la infraestructura de host de las máquinas virtuales aisladas y los hosts dedicados de Azure. En este tema se tratan las opciones de la CLI de Azure para el control de mantenimiento. Para más información sobre las ventajas de usar el control de mantenimiento, sus limitaciones y otras opciones de administración, consulte el artículo sobre la [administración de las actualizaciones de las distintas plataformas con control de mantenimiento](maintenance-control.md).

## <a name="create-a-maintenance-configuration"></a>Creación de una configuración de mantenimiento

Use `az maintenance configuration create` para crear una configuración de mantenimiento. En este ejemplo se crea una configuración de mantenimiento denominada *myConfig* en el ámbito del host. 

```azurecli-interactive
az group create \
   --location eastus \
   --name myMaintenanceRG
az maintenance configuration create \
   -g myMaintenanceRG \
   --resource-name myConfig \
   --maintenance-scope host\
   --location eastus
```

Copie el identificador de configuración de la salida para usarlo más adelante.

El uso de `--maintenance-scope host` garantiza que la configuración de mantenimiento se utiliza para controlar las actualizaciones de la infraestructura de host.

Si intenta crear una configuración con el mismo nombre, pero en una ubicación diferente, obtendrá un error. Los nombres de configuración deben ser únicos dentro de su grupo de recursos.

Puede consultar las configuraciones de mantenimiento disponibles mediante `az maintenance configuration list`.

```azurecli-interactive
az maintenance configuration list --query "[].{Name:name, ID:id}" -o table 
```

### <a name="create-a-maintenance-configuration-with-scheduled-window"></a>Creación de una configuración de mantenimiento con una ventana programada
También puede declarar una ventana programada cuando Azure vaya a aplicar las actualizaciones en los recursos. En este ejemplo se crea una configuración de mantenimiento denominada myConfig con una ventana programada de 5 horas el cuarto lunes de cada mes. Una vez que cree una ventana programada, ya no tendrá que aplicar las actualizaciones manualmente.

```azurecli-interactive
az maintenance configuration create \
   -g myMaintenanceRG \
   --resource-name myConfig \
   --maintenance-scope host \
   --location eastus \
   --maintenance-window-duration "05:00" \
   --maintenance-window-recur-every "Month Fourth Monday" \
   --maintenance-window-start-date-time "2020-12-30 08:00" \
   --maintenance-window-time-zone "Pacific Standard Time"
```

> [!IMPORTANT]
> La **duración** del mantenimiento debe ser de *2 horas* o más. La **periodicidad** del mantenimiento debe establecerse en al menos una vez en 35 días.

La periodicidad del mantenimiento puede expresarse como programaciones diarias, semanales o mensuales. Ejemplos:
- **daily**- maintenance-window-recur-every: "Day" **o** "3Days"
- **weekly**- maintenance-window-recur-every: "3Weeks" **o** "Week Saturday,Sunday"
- **monthly**- maintenance-window-recur-every: "Month day23,day24" **o** "Month Last Sunday" **o** "Month Fourth Monday"


## <a name="assign-the-configuration"></a>Asignación de la configuración

Use `az maintenance assignment create` para asignar la configuración a la máquina virtual aislada o al host de Azure Dedicated Host.

### <a name="isolated-vm"></a>Máquina virtual aislada

Aplique la configuración a una máquina virtual con el identificador de la configuración. Especifique `--resource-type virtualMachines` y proporcione el nombre de la máquina virtual para `--resource-name`, y el grupo de recursos para la máquina virtual en `--resource-group`, así como la ubicación de la máquina virtual para `--location`. 

```azurecli-interactive
az maintenance assignment create \
   --resource-group myMaintenanceRG \
   --location eastus \
   --resource-name myVM \
   --resource-type virtualMachines \
   --provider-name Microsoft.Compute \
   --configuration-assignment-name myConfig \
   --maintenance-configuration-id "/subscriptions/1111abcd-1a11-1a2b-1a12-123456789abc/resourcegroups/myMaintenanceRG/providers/Microsoft.Maintenance/maintenanceConfigurations/myConfig"
```

### <a name="dedicated-host"></a>Host dedicado

Para aplicar una configuración a un host dedicado, debe incluir `--resource-type hosts`, `--resource-parent-name` con el nombre del grupo host y `--resource-parent-type hostGroups`. 

El parámetro `--resource-id` es el identificador del host. Puede usar [az vm host get-instance-view](/cli/azure/vm/host#az_vm_host_get_instance_view) para obtener el identificador del host dedicado.

```azurecli-interactive
az maintenance assignment create \
   -g myDHResourceGroup \
   --resource-name myHost \
   --resource-type hosts \
   --provider-name Microsoft.Compute \
   --configuration-assignment-name myConfig \
   --maintenance-configuration-id "/subscriptions/1111abcd-1a11-1a2b-1a12-123456789abc/resourcegroups/myDhResourceGroup/providers/Microsoft.Maintenance/maintenanceConfigurations/myConfig" \
   -l eastus \
   --resource-parent-name myHostGroup \
   --resource-parent-type hostGroups 
```

## <a name="check-configuration"></a>Comprobación de configuración

Puede verificar que la configuración se aplicó correctamente o consultar qué configuración se aplica actualmente mediante `az maintenance assignment list`.

### <a name="isolated-vm"></a>Máquina virtual aislada

```azurecli-interactive
az maintenance assignment list \
   --provider-name Microsoft.Compute \
   --resource-group myMaintenanceRG \
   --resource-name myVM \
   --resource-type virtualMachines \
   --query "[].{resource:resourceGroup, configName:name}" \
   --output table
```

### <a name="dedicated-host"></a>Host dedicado 

```azurecli-interactive
az maintenance assignment list \
   --resource-group myDHResourceGroup \
   --resource-name myHost \
   --resource-type hosts \
   --provider-name Microsoft.Compute \
   --resource-parent-name myHostGroup \
   --resource-parent-type hostGroups 
   --query "[].{ResourceGroup:resourceGroup,configName:name}" \
   -o table
```


## <a name="check-for-pending-updates"></a>Búsqueda de actualizaciones pendientes

Use `az maintenance update list` para ver si hay actualizaciones pendientes. Actualice --subscription para que sea el identificador de la suscripción que contiene la máquina virtual.

Si no hay ninguna actualización, el comando devolverá un mensaje de error, que contendrá el texto: `Resource not found...StatusCode: 404`.

Si hay actualizaciones, solo se devolverá una, aunque haya varias actualizaciones pendientes. Los datos de esta actualización se devolverán en un objeto:

```text
[
  {
    "impactDurationInSec": 9,
    "impactType": "Freeze",
    "maintenanceScope": "Host",
    "notBefore": "2020-03-03T07:23:04.905538+00:00",
    "resourceId": "/subscriptions/9120c5ff-e78e-4bd0-b29f-75c19cadd078/resourcegroups/DemoRG/providers/Microsoft.Compute/hostGroups/demoHostGroup/hosts/myHost",
    "status": "Pending"
  }
]
  ```

### <a name="isolated-vm"></a>Máquina virtual aislada

Compruebe si hay actualizaciones pendientes para una máquina virtual aislada. En este ejemplo, se aplica a la salida formato de tabla para facilitar la lectura.

```azurecli-interactive
az maintenance update list \
   -g myMaintenanceRg \
   --resource-name myVM \
   --resource-type virtualMachines \
   --provider-name Microsoft.Compute \
   -o table
```

### <a name="dedicated-host"></a>Host dedicado

Compruebe si hay actualizaciones pendientes para un host dedicado. En este ejemplo, se aplica a la salida formato de tabla para facilitar la lectura. Reemplace los valores de los recursos por los suyos propios.

```azurecli-interactive
az maintenance update list \
   --subscription 1111abcd-1a11-1a2b-1a12-123456789abc \
   -g myHostResourceGroup \
   --resource-name myHost \
   --resource-type hosts \
   --provider-name Microsoft.Compute \
   --resource-parentname myHostGroup \
   --resource-parent-type hostGroups \
   -o table
```

## <a name="apply-updates"></a>Aplicación de actualizaciones

Use `az maintenance apply update` para aplicar actualizaciones pendientes. Si se ejecuta correctamente, este comando devolverá el archivo JSON que contiene los detalles de la actualización.

### <a name="isolated-vm"></a>Máquina virtual aislada

Cree una solicitud para aplicar actualizaciones a una máquina virtual aislada.

```azurecli-interactive
az maintenance applyupdate create \
   --subscription 1111abcd-1a11-1a2b-1a12-123456789abc \
   --resource-group myMaintenanceRG \
   --resource-name myVM \
   --resource-type virtualMachines \
   --provider-name Microsoft.Compute
```


### <a name="dedicated-host"></a>Host dedicado

Aplique las actualizaciones a un host dedicado.

```azurecli-interactive
az maintenance applyupdate create \
   --subscription 1111abcd-1a11-1a2b-1a12-123456789abc \
   --resource-group myHostResourceGroup \
   --resource-name myHost \
   --resource-type hosts \
   --provider-name Microsoft.Compute \
   --resource-parent-name myHostGroup \
   --resource-parent-type hostGroups
```

## <a name="check-the-status-of-applying-updates"></a>Comprobación del estado de aplicación de las actualizaciones 

Puede comprobar el progreso de las actualizaciones mediante `az maintenance applyupdate get`. 

Puede usar `default` como nombre de la actualización para ver los resultados de la última actualización o reemplazar `myUpdateName` por el nombre de la actualización que se devolvió cuando se ejecutó `az maintenance applyupdate create`.

```text
Status         : Completed
ResourceId     : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso
LastUpdateTime : 1/1/2020 12:00:00 AM
Id             : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso/providers/Microsoft.Maintenance/applyUpdates/default
Name           : default
Type           : Microsoft.Maintenance/applyUpdates
```
LastUpdateTime será la hora a la que se completó la actualización, tanto si la inició el usuario o la plataforma en caso de que no se usara la ventana de mantenimiento automático. Si nunca se ha aplicado una actualización mediante el control de mantenimiento, se mostrará el valor predeterminado.

### <a name="isolated-vm"></a>Máquina virtual aislada

```azurecli-interactive
az maintenance applyupdate get \
   --resource-group myMaintenanceRG \
   --resource-name myVM \
   --resource-type virtualMachines \
   --provider-name Microsoft.Compute \
   --apply-update-name default 
```

### <a name="dedicated-host"></a>Host dedicado

```azurecli-interactive
az maintenance applyupdate get \
   --subscription 1111abcd-1a11-1a2b-1a12-123456789abc \ 
   --resource-group myMaintenanceRG \
   --resource-name myHost \
   --resource-type hosts \
   --provider-name Microsoft.Compute \
   --resource-parent-name myHostGroup \ 
   --resource-parent-type hostGroups \
   --apply-update-name myUpdateName \
   --query "{LastUpdate:lastUpdateTime, Name:name, ResourceGroup:resourceGroup, Status:status}" \
   --output table
```


## <a name="delete-a-maintenance-configuration"></a>Eliminación de una configuración de mantenimiento

Use `az maintenance configuration delete` para eliminar una configuración de mantenimiento. Al eliminar la configuración se quita el control de mantenimiento de los recursos asociados.

```azurecli-interactive
az maintenance configuration delete \
   --subscription 1111abcd-1a11-1a2b-1a12-123456789abc \
   -g myResourceGroup \
   --resource-name myConfig
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información, consulte [Mantenimiento y actualizaciones](maintenance-and-updates.md).
