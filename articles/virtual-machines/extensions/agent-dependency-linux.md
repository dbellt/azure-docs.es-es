---
title: Extensión de máquina virtual de Azure Monitor Dependency para Linux
description: Implemente Dependency Agent de Azure Monitor en la máquina virtual Linux con una extensión de máquina virtual.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: mgoedtel
ms.author: magoedte
ms.collection: linux
ms.date: 06/01/2021
ms.openlocfilehash: 97f557ec45530de3f42dd61ee1cded57fd7c33a0
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110793752"
---
# <a name="azure-monitor-dependency-virtual-machine-extension-for-linux"></a>Extensión de máquina virtual de Azure Monitor Dependency para Linux

La característica Service Map de Azure Monitor para VM obtiene sus datos de Microsoft Dependency Agent. Microsoft, como editor de la extensión de máquina virtual de Dependency Agent de Azure VM para Linux, es quien presta los servicios de soporte técnico para esta solución. La extensión instala Dependency Agent en las máquinas virtuales de Azure. En este documento se especifican las plataformas compatibles, las configuraciones y las opciones de implementación de la extensión de máquina virtual de Dependency Agent de Azure VM para Linux.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo

La extensión de Dependency Agent de Azure VM para Linux se puede ejecutar en los sistemas operativos compatibles que se indican en la sección [Sistemas operativos compatibles](../../azure-monitor/vm/vminsights-enable-overview.md#supported-operating-systems) del artículo de implementación de Azure Monitor para VM.

## <a name="extension-schema"></a>Esquema de extensión

El siguiente JSON muestra el esquema de la extensión de Dependency Agent de Azure VM en una máquina virtual Linux de Azure. 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Linux Azure VM."
      }
    }
  },
  "variables": {
    "vmExtensionsApiVersion": "2017-03-30"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "[variables('vmExtensionsApiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentLinux",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
      }
    }
  ],
    "outputs": {
    }
}
```

### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor/ejemplo |
| ---- | ---- |
| apiVersion | 2015-01-01 |
| publisher | Microsoft.Azure.Monitoring.DependencyAgent |
| type | DependencyAgentLinux |
| typeHandlerVersion | 9.5 |

## <a name="template-deployment"></a>Implementación de plantilla

Puede implementar las extensiones de VM de Azure con plantillas de Azure Resource Manager. Puede usar el esquema JSON detallado en la sección anterior en una plantilla de Azure Resource Manager para ejecutar la extensión de Dependency Agent de Azure VM durante la implementación de dicha plantilla.

El JSON de una extensión de máquina virtual se puede anidar dentro del recurso de máquina virtual. O bien, puede colocarlo en la raíz o en el nivel superior de una plantilla JSON de Resource Manager. La colocación de la plantilla JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md).

En el siguiente ejemplo se da por supuesto que la extensión de Dependency Agent está anidada dentro del recurso de máquina virtual. Cuando se anidan los recursos de extensión, el JSON se coloca en el objeto `"resources": []` de la máquina virtual.


```json
{
    "type": "extensions",
    "name": "DAExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentLinux",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
    }
}
```

Al colocar el JSON de la extensión en la raíz de la plantilla, el nombre de recurso incluye una referencia a la máquina virtual principal. El tipo refleja la configuración anidada. 

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "<parentVmResource>/DAExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentLinux",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
    }
}
```

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

Puede usar la CLI de Azure para implementar la extensión de máquina virtual de Dependency Agent en una máquina virtual que ya tenga.  

```azurecli

az vm extension set \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 
```

## <a name="automatic-upgrade-preview"></a>Actualización automática (versión preliminar)
Ahora hay disponible una nueva característica para actualizar automáticamente las versiones secundarias de la extensión Dependency en versión preliminar pública. Debe realizar los siguientes cambios de configuración para habilitar esta característica.

-   Use uno de los métodos de [Habilitación del acceso en versión preliminar](../automatic-extension-upgrade.md#enabling-preview-access) para habilitar la característica en la suscripción.
- Agregue el atributo `enableAutomaticUpgrade` a la plantilla.

El esquema del control de versiones de la extensión Dependency Agent sigue el formato siguiente:

```
<MM.mm.bb.rr> where M = Major version number, m = minor version number, b = bug number, r = revision number.
```

Los atributos `enableAutomaticUpgrade` y `autoUpgradeMinorVersion` funcionan conjuntamente para determinar cómo se controlarán las actualizaciones de las máquinas virtuales de la suscripción.

| enableAutomaticUpgrade | autoUpgradeMinorVersion | Efecto |
|:---|:---|:---|
| true | false | Actualice Dependency Agent si existe una versión más reciente de bb.rr. Por ejemplo, si ejecuta 9.6.0.1355 y la versión más reciente es 9.6.2.1366, las máquinas virtuales de las suscripciones habilitadas se actualizarán a 9.6.2.1366. |
| true | true |  Esto actualizará Dependency Agent si existe una versión más reciente de mm.bb.rr o bb.rr. Por ejemplo, si ejecuta 9.6.0.1355 y la versión más reciente es 9.7.1.1416, las máquinas virtuales de las suscripciones habilitadas se actualizarán a 9.7.1.1416. Además, si ejecuta 9.6.0.1355 y la versión más reciente es 9.6.2.1366, las máquinas virtuales de las suscripciones habilitadas se actualizarán a 9.6.2.1366. |
| false | true o false | La actualización automática está deshabilitada.

> [!IMPORTANT]
> Si agrega `enableAutomaticUpgrade` a la plantilla, asegúrese de usar al menos la versión de API 2019-12-01.

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure:

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

El resultado de la ejecución de las extensiones se registra en el archivo siguiente:

```
/opt/microsoft/dependency-agent/log/install.log 
```

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, póngase en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). O bien, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**. Para obtener más información sobre cómo usar el soporte técnico de Azure, lea las [preguntas más frecuentes del soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
