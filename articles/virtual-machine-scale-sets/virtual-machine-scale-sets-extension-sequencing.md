---
title: Uso de la secuenciación de extensiones con conjuntos de escalado de máquinas virtuales de Azure
description: Aprenda a secuenciar el aprovisionamiento de extensiones al implementar varias extensiones en conjuntos de escalado de máquinas virtuales.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 01/30/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 1b5aea1f0f0101231408dc9ad7b57a30f2c86256
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788164"
---
# <a name="sequence-extension-provisioning-in-virtual-machine-scale-sets"></a>Secuenciación del aprovisionamiento de extensiones en conjuntos de escalado de máquinas virtuales
Las extensiones de máquina virtual de Azure proporcionan funcionalidades como configuración y administración posteriores a la implementación, supervisión, seguridad y mucho más. Para lograr los resultados deseados, las implementaciones en producción suelen usar una combinación de varias extensiones configuradas para las instancias de máquina virtual.

Cuando se usan varias extensiones en una máquina virtual, es importante asegurarse de que aquellas que requieren los mismos recursos del sistema operativo no intenten adquirirlos al mismo tiempo. Algunas extensiones también dependen de otras para proporcionar las configuraciones necesarias, como la configuración del entorno y los secretos. Si no se dispone de una ordenación y una secuenciación correctas, las implementaciones de extensiones dependientes producirán un error.

En este artículo se detalla cómo puede secuenciar las extensiones a fin de configurarlas para las instancias de máquina virtual de los conjuntos de escalado de máquinas virtuales.

## <a name="prerequisites"></a>Prerequisites
En este artículo se da por hecho que está familiarizado con:
-   [Extensiones](../virtual-machines/extensions/overview.md) de máquina virtual de Azure
-   [Modificación](virtual-machine-scale-sets-upgrade-scale-set.md) de conjuntos de escalado de máquinas virtuales

## <a name="when-to-use-extension-sequencing"></a>Cuándo usar la secuenciación de extensiones
La secuenciación de extensiones no es obligatoria en los conjuntos de escalado y, salvo que se especifique otra cosa, se pueden aprovisionar las extensiones de una instancia del conjunto de escalado en cualquier orden.

Por ejemplo, si el modelo del conjunto de escalado tiene dos extensiones, ExtensiónA y ExtensiónB, especificadas en el modelo, puede tener lugar cualquiera de las siguientes secuencias de aprovisionamiento:
-   ExtensiónA -> ExtensiónB
-   ExtensiónB -> ExtensiónA

Si la aplicación requiere que la Extensión A se aprovisione siempre antes que la Extensión B, debe usar la secuenciación de extensiones como se describe en este artículo. Con la secuenciación de extensiones, ahora solo se produce una secuencia:
-   ExtensiónA - > ExtensiónB

Las extensiones no especificadas en una secuencia de aprovisionamiento definida se pueden aprovisionar en cualquier momento: antes, después o durante una secuencia definida. La secuenciación de extensiones solo especifica que una extensión determinada se aprovisionará después de otra extensión determinada. No afecta el aprovisionamiento de ninguna otra extensión definida en el modelo.

Por ejemplo, si el modelo del conjunto de escalado tiene tres extensiones, Extensión A, Extensión B y Extensión C, especificadas en el modelo, y la Extensión C se establece para aprovisionarse después de la Extensión A, puede tener lugar una de las siguientes secuencias de aprovisionamiento:
-   ExtensiónA -> ExtensiónC -> ExtensiónB
-   ExtensiónB -> ExtensiónA -> ExtensiónC
-   ExtensiónA -> ExtensiónB -> ExtensiónC

Si tiene que asegurarse de que no se haya aprovisionado ninguna otra extensión mientras se ejecuta la secuencia de extensiones definida, se recomienda secuenciar todas las extensiones del modelo del conjunto de escalado. En el ejemplo anterior, la Extensión B se puede establecer para aprovisionarse después de la Extensión C, tal que solo pueda tener lugar una secuencia:
-   ExtensiónA -> ExtensiónC -> ExtensiónB


## <a name="how-to-use-extension-sequencing"></a>Cómo usar la secuenciación de extensiones
Para secuenciar el aprovisionamiento de extensiones, debe actualizar la definición de extensión en el modelo del conjunto de escalado para incluir la propiedad "provisionAfterExtensions", que acepta una matriz de nombres de extensión. Las extensiones mencionadas en el valor de matriz de la propiedad se deben definir completamente en el modelo del conjunto de escalado.

### <a name="template-deployment"></a>Implementación de plantilla
En el ejemplo siguiente se define una plantilla donde el conjunto de escalado tiene tres extensiones, ExtensiónA, ExtensiónB y ExtensiónC, tal que las extensiones se aprovisionan en el orden:
-   ExtensiónA -> ExtensiónB -> ExtensiónC

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA"
          ],
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

Puesto que la propiedad "provisionAfterExtensions" acepta una matriz de nombres de extensión, se puede modificar el ejemplo anterior tal que la ExtensiónC se aprovisione después de la ExtensiónA y la ExtensiónB, pero no se requiera ninguna ordenación entre la ExtensiónA y la ExtensiónB. La plantilla siguiente puede usarse para lograr este escenario:

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA","ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

### <a name="rest-api"></a>API DE REST
En el ejemplo siguiente se agrega una nueva extensión denominada ExtensiónC a un modelo del conjunto de escalado. La ExtensiónC tiene dependencias de la ExtensiónA y la ExtensiónB, que ya se han definido en el modelo del conjunto de escalado.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "name": "ExtensionC",
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ],
    "publisher": "ExtensionC.Publisher",
    "settings": {},
    "typeHandlerVersion": "3.0",
    "autoUpgradeMinorVersion": true,
    "type": "ExtensionC" 
  }                  
}
```

Si la ExtensiónC se ha definido anteriormente en el modelo del conjunto de escalado y ahora quiere agregar sus dependencias, puede ejecutar una operación `PATCH` para editar las propiedades de la extensión ya implementada.

```
PATCH on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ]
  }                  
}
```
Los cambios realizados en las instancias del conjunto de escalado existente se copian en la siguiente [actualización](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model).

### <a name="azure-powershell"></a>Azure PowerShell
Use el cmdlet [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) para agregar la extensión Estado de la aplicación a la definición del modelo del conjunto de escalado. La secuenciación de extensiones requiere el uso de Azure PowerShell 1.2.0 o superior.

En el siguiente ejemplo se agrega la [extensión Estado de la aplicación](virtual-machine-scale-sets-health-extension.md) a `extensionProfile` en un modelo de un conjunto de escalado basado en Windows. La extensión Estado de la aplicación se aprovisionará después de aprovisionar la [extensión Script personalizado](../virtual-machines/extensions/custom-script-windows.md), ya definida en el conjunto de escalado.

```azurepowershell-interactive
# Define the scale set variables
$vmScaleSetName = "myVMScaleSet"
$vmScaleSetResourceGroup = "myVMScaleSetResourceGroup"

# Define the Application Health extension properties
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"};
$extensionName = "myHealthExtension"
$extensionType = "ApplicationHealthWindows"
$publisher = "Microsoft.ManagedServices"

# Get the scale set object
$vmScaleSet = Get-AzVmss `
  -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName

# Add the Application Health extension to the scale set model
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet `
  -Name $extensionName `
  -Publisher $publisher `
  -Setting $publicConfig `
  -Type $extensionType `
  -TypeHandlerVersion "1.0" `
  -ProvisionAfterExtension "CustomScriptExtension" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0
Use el [conjunto de extensiones az vmss](/cli/azure/vmss/extension#az_vmss_extension_set) para agregar la extensión Estado de la aplicación a la definición del modelo del conjunto de escalado. La secuenciación de extensiones requiere el uso de la CLI de Azure 2.0.55 o superior.

En el siguiente ejemplo se agrega la [extensión Estado de la aplicación](virtual-machine-scale-sets-health-extension.md) al modelo del conjunto de escalado basado en Windows. La extensión Estado de la aplicación se aprovisionará después de aprovisionar la [extensión Script personalizado](../virtual-machines/extensions/custom-script-windows.md), ya definida en el conjunto de escalado.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthWindows \
  --publisher Microsoft.ManagedServices \
  --version 1.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --provision-after-extensions CustomScriptExtension \
  --settings ./extension.json
```


## <a name="troubleshoot"></a>Solución de problemas

### <a name="not-able-to-add-extension-with-dependencies"></a>No se pueden agregar extensiones con dependencias
1. Asegúrese de que las extensiones especificadas en provisionAfterExtensions estén definidas en el modelo del conjunto de escalado.
2. Asegúrese de que no se haya introducido ninguna dependencia circular. Por ejemplo, la siguiente secuencia no está permitida: ExtensionA-> ExtensionB-> ExtensionC-> ExtensionA
3. Asegúrese de que todas las extensiones de las que toma dependencias tengan la propiedad "settings" bajo la opción "properties" de la extensión. Por ejemplo, si la ExtensiónB se debe aprovisionar después de la ExtensiónA, la ExtensiónA debe tener el campo "settings" bajo la opción "properties". Puede especificar una propiedad "settings" vacía si la extensión no impone una configuración obligatoria.

### <a name="not-able-to-remove-extensions"></a>No se pueden quitar las extensiones
Asegúrese de que las extensiones que se van a quitar no aparezcan en provisionAfterExtensions para ninguna otra extensión.

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [implementar la aplicación](virtual-machine-scale-sets-deploy-app.md) en conjuntos de escalado de máquinas virtuales.
