---
title: Exportación de grupos de recursos de Azure que contienen extensiones de máquina virtual
description: Exportar plantillas de Resource Manager que incluyen extensiones de máquina virtual.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: windows
ms.date: 12/05/2016
ms.openlocfilehash: d461be7d760e1e4fc99fe23a9fe2d5529237ebb3
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949830"
---
# <a name="exporting-resource-groups-that-contain-vm-extensions"></a>Exportación de grupos de recursos que contienen extensiones de VM

Los grupos de recursos de Azure se pueden exportar a una nueva plantilla de Resource Manager que, después, se puede volver a implementar. El proceso de exportación interpreta los recursos existentes y crea una plantilla de Resource Manager que, cuando se implementa, da como resultado un grupo de recursos similar. Cuando se usa la opción de exportación del grupo de recursos en un grupo de recursos que contiene extensiones de máquina virtual, se deben tener en cuenta varios elementos, como la compatibilidad de extensión y la configuración protegida.

En este documento se detalla cómo funciona el proceso de exportación del grupo de recursos con respecto a las extensiones de máquina virtual, incluida una lista de extensiones admitidas y detalles sobre el control de los datos seguros.

## <a name="supported-virtual-machine-extensions"></a>Extensiones de máquina virtual admitidas

Hay muchas extensiones de máquina virtual disponibles. No todas las extensiones se pueden exportar a una plantilla de Resource Manager mediante la característica de “Script de Automation”. Si no se admite una extensión de máquina virtual, debe volverse a colocar manualmente en la plantilla exportada.

Las siguientes extensiones se pueden exportar con la característica de script de automatización.

> Acronis Backup, Acronis Backup Linux, Bg Info, BMC CTM Agent Linux, BMC CTM Agent Windows, Chef Client, Custom Script, Custom Script Extension, Custom Script for Linux, Datadog Linux Agent, Datadog Windows Agent, Docker Extension, DSC Extension, Dynatrace Linux, Dynatrace Windows, HPE Security Application Defender, IaaS Antimalware, IaaS Diagnostics, Linux Chef Client, Linux Diagnostic, OS Patching For Linux, Puppet Agent, Site 24x7 Apm Insight, Site 24x7 Linux Server, Site 24x7 Windows Server, Trend Micro DSA, Trend Micro DSA Linux, VM Access For Linux, VM Access For Linux, VM Snapshot, VM Snapshot Linux

## <a name="export-the-resource-group"></a>Exportación del grupo de recursos

Para exportar un grupo de recursos a una plantilla reutilizable, complete los pasos siguientes:

1. Inicio de sesión en Azure Portal
2. En el menú Concentrador, haga clic en Grupos de recursos.
3. Seleccione el grupo de recursos de destino de la lista.
4. En la hoja Grupo de recursos, haga clic en Script de automatización.

![Exportación de plantilla](./media/export-templates/template-export.png)

El script de automatización de Azure Resource Manager genera una plantilla de Resource Manager, un archivo de parámetros y varios scripts de implementación de ejemplo, como PowerShell y la CLI de Azure. En este momento, la plantilla exportada se puede descargar utilizando el botón de descarga, agregar como una nueva plantilla a la biblioteca de plantillas o volver a implementar mediante el botón de implementación.

## <a name="configure-protected-settings"></a>Definición de la configuración protegida

Muchas extensiones de máquina virtual de Azure incluyen una configuración protegida, que cifra los datos confidenciales, como credenciales y cadenas de configuración. La configuración protegida no se exporta con el script de automatización. Si procede, la configuración protegida necesita volverse a insertar en la plantilla exportada.

### <a name="step-1---remove-template-parameter"></a>Paso 1: Quitar parámetro de plantilla

Cuando el grupo de recursos se exporta, se crea un parámetro de plantilla único para proporcionar un valor para la configuración protegida exportada. Este parámetro se puede quitar. Para quitar el parámetro, examine la lista de parámetros y elimine el parámetro que tiene un aspecto similar a este ejemplo JSON.

```json
"extensions_extensionname_protectedSettings": {
    "defaultValue": null,
    "type": "SecureObject"
}
```

### <a name="step-2---get-protected-settings-properties"></a>Paso 2: Obtener propiedades de configuración protegida

Dado que cada configuración protegida tiene un conjunto de propiedades obligatorias, es necesario recopilar una lista de estas propiedades. Cada parámetro de la configuración protegida se encuentra en el [esquema de Azure Resource Manager en GitHub](https://raw.githubusercontent.com/Azure/azure-resource-manager-schemas/master/schemas/2015-08-01/Microsoft.Compute.json). Este esquema solo incluye los conjuntos de parámetros para las extensiones que aparecen en la sección de información general de este documento. 

Desde dentro del repositorio de esquema, busque la extensión que desee, para este ejemplo `IaaSDiagnostics`. Una vez que el objeto `protectedSettings` de las extensiones se ha localizado, tome nota de cada parámetro. En el ejemplo de la extensión `IaasDiagnostic`, los parámetros requeridos son `storageAccountName`, `storageAccountKey` y `storageAccountEndPoint`.

```json
"protectedSettings": {
    "type": "object",
    "properties": {
        "storageAccountName": {
            "type": "string"
        },
        "storageAccountKey": {
            "type": "string"
        },
        "storageAccountEndPoint": {
            "type": "string"
        }
    },
    "required": [
        "storageAccountName",
        "storageAccountKey",
        "storageAccountEndPoint"
    ]
}
```

### <a name="step-3---re-create-the-protected-configuration"></a>Paso 3: Volver a crear la configuración protegida

En la plantilla exportada, busque `protectedSettings` y reemplace el objeto de configuración protegida exportado por uno nuevo que incluya los parámetros de extensión necesarios y un valor para cada uno de ellos.

En el ejemplo de la extensión `IaasDiagnostic`, la nueva configuración protegida sería similar al ejemplo siguiente:

```json
"protectedSettings": {
    "storageAccountName": "[parameters('storageAccountName')]",
    "storageAccountKey": "[parameters('storageAccountKey')]",
    "storageAccountEndPoint": "https://core.windows.net"
}
```

El recurso de extensión final tiene un aspecto similar al siguiente ejemplo JSON:

```json
{
    "name": "Microsoft.Insights.VMDiagnosticsSettings",
    "type": "extensions",
    "location": "[resourceGroup().location]",
    "apiVersion": "[variables('apiVersion')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "tags": {
        "displayName": "AzureDiagnostics"
    },
    "properties": {
        "publisher": "Microsoft.Azure.Diagnostics",
        "type": "IaaSDiagnostics",
        "typeHandlerVersion": "1.5",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), variables('vmName'), variables('wadcfgxend')))]",
            "storageAccount": "[parameters('existingdiagnosticsStorageAccountName')]"
        },
        "protectedSettings": {
            "storageAccountName": "[parameters('storageAccountName')]",
            "storageAccountKey": "[parameters('storageAccountKey')]",
            "storageAccountEndPoint": "https://core.windows.net"
        }
    }
}
```

Si utiliza parámetros de plantilla para proporcionar valores de propiedad, estos deben crearse. Al crear parámetros de plantilla para los valores de configuración protegida, asegúrese de usar el tipo de parámetro `SecureString` para que se protejan los valores confidenciales. Para más información sobre el uso de parámetros, consulte [Authoring Azure Resource Manager templates](../../azure-resource-manager/templates/syntax.md) (Creación de plantillas de Azure Resource Manager).

En el ejemplo de la extensión `IaasDiagnostic`, se crearán los siguientes parámetros en la sección de parámetros de la plantilla de Resource Manager.

```json
"storageAccountName": {
    "defaultValue": null,
    "type": "SecureString"
},
"storageAccountKey": {
    "defaultValue": null,
    "type": "SecureString"
}
```

En este momento, la plantilla se puede implementar mediante cualquier método de implementación de plantilla.