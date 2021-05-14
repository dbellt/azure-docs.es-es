---
title: Evento de creación de grupo de Azure Batch
description: Referencia del evento de creación de grupo de Batch, que se emite después de crear un grupo. El contenido del registro va a exponer información general acerca del grupo.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: 5e75b56a6fd5de5fd17107c3960004e0edb1b799
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988269"
---
# <a name="pool-create-event"></a>Evento de creación del grupo

 Este evento se genera cuando se ha creado un grupo. El contenido del registro va a exponer información general acerca del grupo. Tenga en cuenta que si el tamaño de destino del grupo es mayor que 0 nodos de proceso, un evento de inicio de cambio de tamaño del grupo seguirá inmediatamente después de este evento.

 En el siguiente ejemplo, se muestra el cuerpo de un evento de creación del grupo.

```
{
    "id": "myPool1",
    "displayName": "Production Pool",
    "vmSize": "Standard_F1s",
    "imageType": "VirtualMachineConfiguration",
    "cloudServiceConfiguration": {
        "osFamily": "3",
        "targetOsVersion": "*"
    },
    "networkConfiguration": {
        "subnetId": " "
    },
    "virtualMachineConfiguration": {
          "imageReference": {
            "publisher": " ",
            "offer": " ",
            "sku": " ",
            "version": " "
          },
          "nodeAgentId": " "
        },
    "resizeTimeout": "300000",
    "targetDedicatedNodes": 2,
    "targetLowPriorityNodes": 2,
    "taskSlotsPerNode": 1,
    "vmFillType": "Spread",
    "enableAutoScale": false,
    "enableInterNodeCommunication": false,
    "isAutoPool": false
}
```

|Elemento|Tipo|Notas|
|-------------|----------|-----------|
|`id`|String|El identificador del grupo.|
|`displayName`|String|El nombre para mostrar del grupo.|
|`vmSize`|String|El tamaño de las máquinas virtuales que se usan en el grupo. Todas las máquinas virtuales de un grupo son del mismo tamaño. <br/><br/> Para obtener información sobre los tamaños disponibles de máquinas virtuales para los grupos de Cloud Services (grupos creados con cloudServiceConfiguration), consulte [Tamaños para Cloud Services](../cloud-services/cloud-services-sizes-specs.md). Batch admite todas las VM de Cloud Services excepto `ExtraSmall`.<br/><br/> Para obtener información sobre los tamaños de VM disponibles para grupos con imágenes del Virtual Machines Marketplace (grupos creados con virtualMachineConfiguration), consulte [Tamaños de máquinas virtuales](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux) o [Tamaños de máquinas virtuales](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows). Batch admite todos los tamaños de máquina virtual de Azure excepto `STANDARD_A0` y aquellos con Premium Storage (series `STANDARD_GS`, `STANDARD_DS` y `STANDARD_DSV2`).|
|`imageType`|String|El método de implementación de la imagen. Estos son los valores que se admiten: `virtualMachineConfiguration` o `cloudServiceConfiguration`|
|[`cloudServiceConfiguration`](#bk_csconf)|Tipo complejo|La configuración de los servicios en la nube para el grupo.|
|[`virtualMachineConfiguration`](#bk_vmconf)|Tipo complejo|La configuración de la máquina virtual para el grupo.|
|[`networkConfiguration`](#bk_netconf)|Tipo complejo|La configuración de red para el grupo.|
|`resizeTimeout`|Time|El tiempo de espera para la asignación de nodos de proceso para el grupo especificado para la última operación de cambio de tamaño en el grupo.  (El tamaño inicial cuando se crea el grupo cuenta como un cambio de tamaño).|
|`targetDedicatedNodes`|Int32|El número de nodos de proceso dedicados solicitados para el grupo.|
|`targetLowPriorityNodes`|Int32|El número de nodos de proceso de baja prioridad solicitados para el grupo.|
|`enableAutoScale`|Bool|Especifica si el tamaño del grupo se ajusta automáticamente con el tiempo.|
|`enableInterNodeCommunication`|Bool|Especifica si el grupo está configurado para la comunicación directa entre nodos.|
|`isAutoPool`|Bool|Especifica si se ha creado el grupo a través del mecanismo AutoPool de un trabajo.|
|`taskSlotsPerNode`|Int32|El número máximo de tareas que pueden ejecutarse simultáneamente en un solo nodo de ejecución en el grupo.|
|`vmFillType`|String|Define cómo el servicio de Batch distribuye las tareas entre los nodos de proceso del grupo. Los valores válidos son Spread o Pack.|

###  <a name="cloudserviceconfiguration"></a><a name="bk_csconf"></a> cloudServiceConfiguration

> [!WARNING]
> Los grupos de configuración de Cloud Services están [en desuso](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/). En su lugar, utilice los grupos de configuración de máquina virtual.

|Nombre del elemento|Tipo|Notas|
|------------------|----------|-----------|
|`osFamily`|String|La familia del SO invitado de Azure para instalar en las máquinas virtuales en el grupo.<br /><br /> Los valores posibles son:<br /><br /> **2**: familia 2 de SO, que equivale a Windows Server 2008 R2 SP1.<br /><br /> **3**: familia 3 de SO, que equivale a Windows Server 2012.<br /><br /> **4**: familia 4 de SO, que equivale a Windows Server 2012 R2.<br /><br /> Para obtener más información, consulte [Versiones del SO invitado de Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|
|`targetOSVersion`|String|La versión del SO invitado de Azure para instalar en las máquinas virtuales en el grupo.<br /><br /> El valor predeterminado es **\*** , que especifica la versión del sistema operativo más reciente para la familia especificada.<br /><br /> Para otros valores permitidos, consulte [Versiones del SO invitado de Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|

###  <a name="virtualmachineconfiguration"></a><a name="bk_vmconf"></a> virtualMachineConfiguration

|Nombre del elemento|Tipo|Notas|
|------------------|----------|-----------|
|[`imageReference`](#bk_imgref)|Tipo complejo|Especifica información sobre la plataforma o imagen de Marketplace que se usará.|
|`nodeAgentId`|String|El SKU agente del nodo de Batch proporcionado en el nodo de ejecución.|
|[`windowsConfiguration`](#bk_winconf)|Tipo complejo|Especifica la configuración del sistema operativo de Windows en la máquina virtual. Esta propiedad no se debe especificar si el imageReference hace referencia a una imagen del sistema operativo de Linux.|

###  <a name="imagereference"></a><a name="bk_imgref"></a> imageReference

|Nombre del elemento|Tipo|Notas|
|------------------|----------|-----------|
|`publisher`|String|El publicador de la imagen.|
|`offer`|String|La oferta publicador de la imagen.|
|`sku`|String|El SKU de la imagen.|
|`version`|String|La versión de la imagen.|

###  <a name="windowsconfiguration"></a><a name="bk_winconf"></a> windowsConfiguration

|Nombre del elemento|Tipo|Notas|
|------------------|----------|-----------|
|`enableAutomaticUpdates`|Boolean|Indica si la máquina virtual está habilitada para las actualizaciones automáticas. Si esta propiedad no se especifica, se usa el valor predeterminado.|

###  <a name="networkconfiguration"></a><a name="bk_netconf"></a> networkConfiguration

|Nombre del elemento|Tipo|Notas|
|------------------|--------------|----------|
|`subnetId`|String|Especifica el identificador de recursos de la subred en la que se crean los nodos de proceso del grupo.|