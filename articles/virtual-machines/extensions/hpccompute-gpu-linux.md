---
title: 'Extensión del controlador de GPU de NVIDIA: Linux Virtual Machines de Azure'
description: Extensión de Microsoft Azure para instalar controladores de GPU de NVIDIA en VMs de cálculo de la serie N que ejecutan Linux.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 01/21/2021
ms.author: amverma
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 493ce6ea8bee59a98be75592f89c85980b6ce285
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110673335"
---
# <a name="nvidia-gpu-driver-extension-for-linux"></a>Extensión del controlador de GPU de NVIDIA para Linux

## <a name="overview"></a>Información general

Esta extensión instala los controladores de GPU de NVIDIA en VMs de serie N de Linux. En función de la familia de VM, la extensión instala los controladores CUDA o GRID. Al instalar controladores de NVIDIA mediante esta extensión, acepta y está de acuerdo con los términos del [contrato de licencia de usuario final de NVIDIA](https://go.microsoft.com/fwlink/?linkid=874330). Durante el proceso de instalación, es posible que la máquina virtual se reinicie para completar la instalación del controlador.

Puede consultar [aquí](../linux/n-series-driver-setup.md) instrucciones sobre la instalación manual de los controladores y las versiones que son compatibles actualmente.
También se dispone de una extensión para instalar controladores de GPU de NVIDIA en [MVs de la serie N para Windows](hpccompute-gpu-windows.md).

## <a name="prerequisites"></a>Requisitos previos

### <a name="operating-system"></a>Sistema operativo

Esta extensión admite las siguientes distribuciones del sistema operativo, dependiendo de la compatibilidad de controladores para la versión específica del sistema operativo.

| Distribución | Versión |
|---|---|
| Linux: Ubuntu | 16.04 LTS, 18.04 LTS |
| Linux: Red Hat Enterprise Linux | 7.3, 7.4, 7.5, 7.6, 7.7, 7.8 |
| Linux: CentOS | 7.3, 7.4, 7.5, 7.6, 7.7, 7.8 |

### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión de Microsoft Azure para los controladores de GPU de NVIDIA requiere que la máquina virtual de destino esté conectada a Internet y tenga acceso.

## <a name="extension-schema"></a>Esquema de extensión

En el siguiente JSON, se muestra el esquema para la extensión.

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="properties"></a>Propiedades

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.HpcCompute | string |
| type | NvidiaGpuDriverLinux | string |
| typeHandlerVersion | 1.3 | int |

### <a name="settings"></a>Configuración

Todos los parámetros son opcionales. El comportamiento predeterminado consiste en no actualizar el kernel si no es necesario para la instalación del controlador, instalar el controlador compatible más reciente y el kit de herramientas CUDA (según corresponda).

| Nombre | Descripción | Valor predeterminado | Valores válidos | Tipo de datos |
| ---- | ---- | ---- | ---- | ---- |
| updateOS | Actualice el kernel, incluso si no es necesario para la instalación del controlador | false | true, false | boolean |
| driverVersion | NV: versión del controlador de GRID<br> NC o ND: versión del kit de herramientas CUDA. Los controladores más recientes del CUDA elegido se instalan automáticamente. | latest | [Lista](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json) de versiones de controlador compatibles | string |
| installCUDA | Instale el kit de herramientas CUDA. Pertinente solo para las máquinas virtuales de la serie NC/ND. | true | true, false | boolean |


## <a name="deployment"></a>Implementación


### <a name="azure-resource-manager-template"></a>Plantilla de Azure Resource Manager 

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas resultan ideales al implementar una o varias máquinas virtuales que requieren configurarse tras la implementación.

La configuración JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual o colocada en la raíz o nivel superior de una plantilla JSON de Resource Manager. La colocación de la configuración JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md). 

En el siguiente ejemplo se da por supuesto que la extensión está anidada dentro del recurso de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto `"resources": []` de la máquina virtual.

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="powershell"></a>PowerShell

```powershell
Set-AzVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "NvidiaGpuDriverLinux" `
    -ExtensionType "NvidiaGpuDriverLinux" `
    -TypeHandlerVersion 1.3 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

En el ejemplo siguiente refleja los ejemplos anteriores de Azure Resource Manager anterior y PowerShell,

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 
```

y también se agregan dos configuraciones personalizadas opcionales de ejemplo para la instalación de los controladores no predeterminados. En concreto, actualiza el kernel del sistema operativo a la versión más reciente e instala un controlador específico para la versión del kit de herramientas CUDA. De nuevo, tenga en cuenta que "--settings" es opcional y un valor predeterminado. Tenga en cuenta que la actualización del kernel puede aumentar las horas de instalación de la extensión. También es posible que elija una versión del kit de herramientas de CUDA específica (una versión anterior) que no siempre es compatible con los kernels más recientes.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 \
  --settings '{ \
    "updateOS": true, \
    "driverVersion": "10.0.130" \
  }'
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante Azure PowerShell y la CLI de Azure. Para ver el estado de implementación de las extensiones de una VM determinada, ejecute el comando siguiente.

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

La salida de la ejecución de las extensiones se registra en el archivo siguiente: Consulte este archivo para realizar un seguimiento del estado de la instalación (de cualquier ejecución de larga duración), así como para solucionar los errores.

```bash
/var/log/azure/nvidia-vmext-status
```

### <a name="exit-codes"></a>Códigos de salida

| Código de salida | Significado | Acción posible |
| :---: | --- | --- |
| 0 | Operación correcta |
| 1 | Uso incorrecto de la extensión. | Compruebe el registro de salida de ejecución |
| 10 | Linux Integration Services para Hyper-V y Azure no están disponibles o no están instalados | Comprobación de la salida de lspci |
| 11 | No se encuentra GPU de NVIDIA en este tamaño de VM | Use un [tamaño de VM y SO admitido](../linux/n-series-driver-setup.md) |
| 12 | Oferta de imagen no compatible |
| 13 | Tamaño de máquina virtual no compatible | Use una máquina virtual de la serie N para la implementación |
| 14 | La operación no se realizó correctamente | Compruebe el registro de salida de ejecución |


### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/community/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las extensiones, consulte el artículo [Características y extensiones de las máquinas virtuales para Linux](features-linux.md).

Para obtener más información acerca de las VM de serie N, consulte el artículo [Tamaños de máquinas virtuales optimizadas para GPU](../sizes-gpu.md).
