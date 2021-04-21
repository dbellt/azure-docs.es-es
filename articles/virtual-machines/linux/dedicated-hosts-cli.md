---
title: Implementación de instancias de conjuntos de escalado y máquinas virtuales en hosts dedicados mediante la CLI
description: Implemente las instancias de conjuntos de escalado y máquinas virtuales en hosts dedicados mediante la interfaz de la línea de comandos de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: dedicated-hosts
ms.topic: how-to
ms.date: 11/12/2020
ms.author: cynthn
ms.openlocfilehash: adc09bf2572be563ff52cf9fa3d0dea51263d032
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774420"
---
# <a name="deploy-to-dedicated-hosts-using-the-azure-cli"></a>Implementación en hosts dedicados mediante la CLI de Azure
 

En este artículo se ofrecen instrucciones para crear un [host dedicado](../dedicated-hosts.md) de Azure en el que se pueden hospedar máquinas virtuales (VM). 

Asegúrese de tener instalada la CLI de Azure, versión 2.16.0 o posterior, y de haber iniciado sesión en una cuenta de Azure con `az login`. 


## <a name="limitations"></a>Limitaciones

- Los tamaños y tipos de hardware disponibles para hosts dedicados varían según la región. Para más información, consulte la [página de precios](https://aka.ms/ADHPricing) de hosts.

## <a name="create-resource-group"></a>Creación de un grupo de recursos 
Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree el grupo de recursos con az group create. En el ejemplo siguiente se crea un grupo de recursos en la ubicación *Este de EE. UU.* con el nombre *myDHResourceGroup*.

```azurecli-interactive
az group create --name myDHResourceGroup --location eastus 
```
 
## <a name="list-available-host-skus-in-a-region"></a>Enumeración de las SKU de host disponibles en una región

No todas las SKU de host están disponibles en todas las regiones y zonas de disponibilidad. 

Enumere la disponibilidad del host y cualquier restricción de la oferta antes de iniciar el aprovisionamiento de hosts dedicados. 

```azurecli-interactive
az vm list-skus -l eastus2  -r hostGroups/hosts  -o table  
```
 
## <a name="create-a-host-group"></a>Creación de un grupo host 

Un **grupo host** es un recurso que representa una colección de hosts dedicados. Puede crear un grupo host en una región y una zona de disponibilidad, y agregarle hosts. Al planear la alta disponibilidad, hay otras opciones. Puede usar una o ambas de las dos opciones siguientes con los hosts dedicados: 
- Abarcar varias zonas de disponibilidad. En este caso, es necesario tener un grupo host en cada una de las zonas que quiera usar.
- Abarcar varios dominios de error que se asignan a bastidores físicos. 
 
En cualquier caso, es necesario proporcionar el número de dominios de error del grupo host. Si no quiere abarcar dominios de error en el grupo, use un número de dominios de error de 1. 

También puede usar zonas de disponibilidad y dominios de error a la vez. 


En este ejemplo, se usará [az vm host group create](/cli/azure/vm/host/group#az_vm_host_group_create) para crear un grupo host con zonas de disponibilidad y dominios de error. 

```azurecli-interactive
az vm host group create \
   --name myHostGroup \
   -g myDHResourceGroup \
   -z 1 \
   --platform-fault-domain-count 2 
``` 

Agregue el parámetro `--automatic-placement true` para que las máquinas virtuales y las instancias del conjunto de escalado se coloquen automáticamente en los hosts, dentro de un grupo host. Para obtener más información, vea [Selección de ubicación manual frente a automática ](../dedicated-hosts.md#manual-vs-automatic-placement).


### <a name="other-examples"></a>Otros ejemplos

También se puede usar [az vm host group create](/cli/azure/vm/host/group#az_vm_host_group_create) para crear un grupo host en la zona de disponibilidad 1, pero sin ningún dominio de error.

```azurecli-interactive
az vm host group create \
   --name myAZHostGroup \
   -g myDHResourceGroup \
   -z 1 \
   --platform-fault-domain-count 1 
```
 
En el ejemplo siguiente se usa [az vm host group create](/cli/azure/vm/host/group#az_vm_host_group_create) para crear un grupo host únicamente con dominios de error (para usarlo en regiones que no admitan zonas de disponibilidad). 

```azurecli-interactive
az vm host group create \
   --name myFDHostGroup \
   -g myDHResourceGroup \
   --platform-fault-domain-count 2 
```
 
## <a name="create-a-host"></a>Creación de un host 

Ahora vamos a crear un host dedicado en el grupo host. Además de un nombre para el host, se le pedirá que proporcione el SKU del host. El SKU del host registra la serie de máquinas virtuales admitidas, así como la generación de hardware del host dedicado.  

Para más información sobre los precios y los SKU de host, consulte [Precios de hosts dedicados de Azure](https://aka.ms/ADHPricing).

Use [az vm host create](/cli/azure/vm/host#az_vm_host_create) para crear un host. Si establece un número de dominios de error para el grupo host, se le pedirá que especifique el dominio de error para su host.  

```azurecli-interactive
az vm host create \
   --host-group myHostGroup \
   --name myHost \
   --sku DSv3-Type1 \
   --platform-fault-domain 1 \
   -g myDHResourceGroup
```


 
## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual 
Cree una máquina virtual en un host dedicado mediante [az vm create](/cli/azure/vm#az_vm_create). Si especificó una zona de disponibilidad al crear el grupo host, debe usar la misma zona al crear la máquina virtual.

```azurecli-interactive
az vm create \
   -n myVM \
   --image debian \
   --host-group myHostGroup \
   --generate-ssh-keys \
   --size Standard_D4s_v3 \
   -g myDHResourceGroup \
   --zone 1
```

Para colocar la máquina virtual en un host concreto, use `--host` en lugar de especificar el grupo host con `--host-group`.
 
> [!WARNING]
> Si crea una máquina virtual en un host que no tenga suficientes recursos, la máquina virtual se creará en un estado de error. 

## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado 

Cuando se implementa un conjunto de escalado, se especifica el grupo host.

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --host-group myHostGroup \
  --generate-ssh-keys \
  --size Standard_D4s_v3 \
  -g myDHResourceGroup \
  --zone 1
```

Si desea elegir manualmente en qué host se va a implementar el conjunto de escalado, agregue `--host` y el nombre del host.


## <a name="check-the-status-of-the-host"></a>Comprobación del estado del host

[az vm host get-instance-view](/cli/azure/vm/host#az_vm_host_get_instance_view) permite comprobar el estado de mantenimiento del host y el número de máquinas virtuales que todavía se pueden implementar.

```azurecli-interactive
az vm host get-instance-view \
   -g myDHResourceGroup \
   --host-group myHostGroup \
   --name myHost
```
 El resultado será similar al siguiente:
 
```json
{
  "autoReplaceOnFailure": true,
  "hostId": "6de80643-0f45-4e94-9a4c-c49d5c777b62",
  "id": "/subscriptions/10101010-1010-1010-1010-101010101010/resourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts/myHost",
  "instanceView": {
    "assetId": "12345678-1234-1234-abcd-abc123456789",
    "availableCapacity": {
      "allocatableVms": [
        {
          "count": 31.0,
          "vmSize": "Standard_D2s_v3"
        },
        {
          "count": 15.0,
          "vmSize": "Standard_D4s_v3"
        },
        {
          "count": 7.0,
          "vmSize": "Standard_D8s_v3"
        },
        {
          "count": 3.0,
          "vmSize": "Standard_D16s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32-8s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32-16s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D32s_v3"
        },
        {
          "count": 1.0,
          "vmSize": "Standard_D48s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64-16s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64-32s_v3"
        },
        {
          "count": 0.0,
          "vmSize": "Standard_D64s_v3"
        }
      ]
    },
    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Provisioning succeeded",
        "level": "Info",
        "message": null,
        "time": "2019-07-24T21:22:40.604754+00:00"
      },
      {
        "code": "HealthState/available",
        "displayStatus": "Host available",
        "level": "Info",
        "message": null,
        "time": null
      }
    ]
  },
  "licenseType": null,
  "location": "eastus2",
  "name": "myHost",
  "platformFaultDomain": 1,
  "provisioningState": "Succeeded",
  "provisioningTime": "2019-07-24T21:22:40.604754+00:00",
  "resourceGroup": "myDHResourceGroup",
  "sku": {
    "capacity": null,
    "name": "DSv3-Type1",
    "tier": null
  },
  "tags": null,
  "type": null,
  "virtualMachines": [
    {
      "id": "/subscriptions/10101010-1010-1010-1010-101010101010/resourceGroups/MYDHRESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/MYVM",
      "resourceGroup": "MYDHRESOURCEGROUP"
    }
  ]
}

```
 
## <a name="export-as-a-template"></a>Exportar como plantilla 
Si ahora quiere crear un entorno de desarrollo adicional con los mismos parámetros, o un entorno de producción que coincida, puede exportar una plantilla. Resource Manager usa plantillas JSON que definen todos los parámetros de su entorno. Puede crear entornos enteros haciendo referencia a esta plantilla JSON. Puede compilar plantillas JSON manualmente o exportar un entorno existente para que la plantilla JSON se cree automáticamente. Use [az group export](/cli/azure/group#az_group_export) para exportar su grupo de recursos.

```azurecli-interactive
az group export --name myDHResourceGroup > myDHResourceGroup.json 
```

Este comando crea el archivo `myDHResourceGroup.json` en el directorio de trabajo actual. Al crear un entorno a partir de esta plantilla, se le piden todos los nombres de recursos. Puede rellenar estos nombres en el archivo de plantilla si agrega el parámetro `--include-parameter-default-value` al comando `az group export`. Edite la plantilla JSON para especificar los nombres de los recursos o cree un archivo parameters.json que especifique los nombres de estos.
 
Para crear un entorno desde una plantilla, use [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create).

```azurecli-interactive
az deployment group create \ 
    --resource-group myNewResourceGroup \ 
    --template-file myDHResourceGroup.json 
```


## <a name="clean-up"></a>Limpieza 

Aunque no se implementen máquinas virtuales, se le cobrará por los hosts dedicados. Elimine los hosts que no use actualmente para ahorrar costos.  

Solo se puede eliminar un host cuando no haya ninguna máquina virtual que lo use. Elimine las máquinas virtuales con [az vm delete](/cli/azure/vm#az_vm_delete).

```azurecli-interactive
az vm delete -n myVM -g myDHResourceGroup
```

Después de eliminar las máquinas virtuales, puede eliminar el host mediante [az vm host delete](/cli/azure/vm/host#az_vm_host_delete).

```azurecli-interactive
az vm host delete -g myDHResourceGroup --host-group myHostGroup --name myHost 
```
 
Una vez que haya eliminado todos los hosts, puede eliminar el grupo host con [az vm host group delete](/cli/azure/vm/host/group#az_vm_host_group_delete).  
 
```azurecli-interactive
az vm host group delete -g myDHResourceGroup --host-group myHostGroup  
```
 
También puede eliminar todo el grupo de recursos con un solo comando. Se eliminarán todos los recursos creados en el grupo, lo que incluye las máquinas virtuales, los hosts y los grupos host.
 
```azurecli-interactive
az group delete -n myDHResourceGroup 
```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más detalles, consulte la introducción a los [hosts dedicados](../dedicated-hosts.md).

- También se pueden crear hosts dedicados con [Azure Portal](../dedicated-hosts-portal.md).

- [Aquí](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md) encontrará una plantilla de ejemplo en la que se usan zonas y dominios de error para obtener la máxima resistencia en una región.
