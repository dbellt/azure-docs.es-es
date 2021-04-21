---
title: Uso de la CLI para implementar máquinas virtuales de acceso puntual de Azure
description: Aprenda a usar la CLI para implementar máquinas virtuales de acceso puntual de Azure con la finalidad de ahorrar costos.
author: cynthn
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 03/22/2021
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 8e8bdaa7a812d8c7accfea59b58b75a58d50e21e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789616"
---
# <a name="deploy-azure-spot-virtual-machines-using-the-azure-cli"></a>Implementación de máquinas virtuales de acceso puntual de Azure con la CLI de Azure

El uso de [máquinas virtuales de acceso puntual de Azure](../spot-vms.md) permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará las máquinas virtuales de acceso puntual de Azure. Por lo tanto, las máquinas virtuales de acceso puntual de Azure son excelentes para cargas de trabajo que puedan soportar interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo/pruebas, las grandes cargas de trabajo de proceso, etc.

Los precios de las máquinas virtuales de acceso puntual de Azure son variables, en función de la región y la SKU. Para más información, consulte precios de las máquinas virtuales para [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 

Puede establecer el precio máximo por hora que esté dispuesto por la máquina virtual. El precio máximo de una máquina virtual de acceso puntual de Azure se puede establecer en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la máquina virtual no se expulsará por precio. El precio de la máquina virtual será el precio actual de la máquina virtual de acceso puntual de Azure o, de ser menor, el de una máquina virtual estándar, siempre que haya capacidad y cuota disponibles. Para más información sobre cómo se establece el precio máximo, consulte [Máquinas virtuales de acceso puntual de Azure: precios](../spot-vms.md#pricing).

El proceso de creación de una máquina virtual de acceso puntual de Azure con la CLI de Azure es el mismo que el que se detalla en el [artículo de inicio rápido](./quick-create-cli.md). Tan solo agregue el parámetro "--priority Spot", establezca `--eviction-policy` en Desasignar (este es el valor predeterminado) o en `Delete` e indique un precio máximo o `-1`. 


## <a name="install-azure-cli"></a>Instalación de la CLI de Azure

Para crear máquinas virtuales de acceso puntual de Azure, debe ejecutar la versión 2.0.74 o posterior de la CLI de Azure. Para saber qué versión tiene, ejecute el comando **az --version**. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

Inicie sesión en Azure mediante [az login](/cli/azure/reference-index#az_login).

```azurecli-interactive
az login
```

## <a name="create-an-azure-spot-virtual-machine"></a>Creación de una máquina virtual de acceso puntual de Azure

En este ejemplo se muestra cómo implementar una máquina virtual de acceso puntual de Azure en Linux que no se expulse en función del precio. La directiva de expulsión se establece para desasignar la VM, de modo que se pueda reiniciar en otro momento. Si quiere eliminar la VM y el disco subyacente cuando se expulsa la VM, establezca `--eviction-policy` en `Delete`.

```azurecli-interactive
az group create -n mySpotGroup -l eastus
az vm create \
    --resource-group mySpotGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --max-price -1 \
    --eviction-policy Deallocate
```



Una vez creada la máquina virtual, puede realizar una consulta para ver el precio máximo de facturación de todas las máquinas virtuales del grupo de recursos.

```azurecli-interactive
az vm list \
   -g mySpotGroup \
   --query '[].{Name:name, MaxPrice:billingProfile.maxPrice}' \
   --output table
```

## <a name="simulate-an-eviction"></a>Simulación de una expulsión

Puede simular la expulsión de una máquina virtual de acceso puntual de Azure usando REST, PowerShell o la CLI para probar la capacidad de respuesta de una aplicación ante una expulsión repentina.

En la mayoría de los casos, querrá usar la API de REST [Máquinas virtuales: simulación de expulsión](/rest/api/compute/virtualmachines/simulateeviction) para facilitar las pruebas automatizadas de las aplicaciones. En REST un `Response Code: 204` significa que la expulsión simulada se ha realizado correctamente. Puede combinar expulsiones simuladas con el [servicio de eventos programados](scheduled-events.md) para automatizar el modo en que responderá la aplicación cuando se expulse la máquina virtual.

Para ver los eventos programados en acción, consulte [Azure Friday: uso de eventos programados de Azure para preparar el mantenimiento de la VM](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance).


### <a name="quick-test"></a>Prueba rápida

Para realizar una prueba rápida para mostrar cómo funcionará una expulsión simulada, vamos a consultar el servicio de eventos programados para ver su aspecto cuando se simula una expulsión mediante la CLI de Azure.

El servicio de eventos programados se habilita para su servicio la primera vez que se realiza una solicitud de eventos. 

Conéctese de forma remota a la VM y, a continuación, abra un símbolo del sistema. 

En el símbolo del sistema de la VM, escriba:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Esta primera respuesta podría tardar hasta 2 minutos. A partir ahora, se deben mostrar los resultados de salida casi de inmediato.

En un equipo que tenga instalada la CLI de Azure (por ejemplo, su equipo local), simule una expulsión mediante [az vm simulate-eviction](https://docs.microsoft.com/cli/azure/vm#az_vm_simulate_eviction). Reemplace el nombre del grupo de recursos y el nombre de la VM con los suyos. 

```azurecli-interactive
az vm simulate-eviction --resource-group mySpotRG --name mySpot
```

La salida de respuesta tendrá el valor `Status: Succeeded` si la solicitud se realizó correctamente.

Vuelva rápidamente a la conexión remota de la máquina virtual de acceso puntual y vuelva a consultar el punto de conexión de Scheduled Events. Repita el siguiente comando hasta que obtenga un resultado que contenga más información:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Cuando el servicio de eventos programados obtiene la notificación de expulsión, recibirá una respuesta similar a la siguiente:

```output
{"DocumentIncarnation":1,"Events":[{"EventId":"A123BC45-1234-5678-AB90-ABCDEF123456","EventStatus":"Scheduled","EventType":"Preempt","ResourceType":"VirtualMachine","Resources":["myspotvm"],"NotBefore":"Tue, 16 Mar 2021 00:58:46 GMT","Description":"","EventSource":"Platform"}]}
```

Puede ver que `"EventType":"Preempt"` y el recurso es el recurso de la VM `"Resources":["myspotvm"]`. 

También puede ver cuándo se expulsará la MV comprobando `"NotBefore"`: la MV no se expulsará antes del tiempo dado y ese será el margen que tiene la aplicación para cerrarse correctamente.


## <a name="next-steps"></a>Pasos siguientes

También puede crear una máquina virtual de acceso puntual de Azure mediante [Azure PowerShell](../windows/spot-powershell.md), el [portal](../spot-portal.md) o una [plantilla](spot-template.md).

Para información sobre las máquinas virtuales de acceso puntual de Azure, consulte la información sobre precios usando la [API de precios de venta directa de Azure](/rest/api/cost-management/retail-prices/azure-retail-prices). Tanto `meterName` como `skuName` contendrán `Spot`.

Si se produce un error, consulte [Códigos de error](../error-codes-spot.md).
