---
title: Recopilación de métricas de máquinas virtuales Windows en Azure Monitor con plantillas
description: Enviar métricas de SO invitado al almacén de métricas de Azure Monitor con una plantilla de Resource Manager para un conjunto de escalado de máquinas virtuales de Windows
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: bwren
ms.openlocfilehash: c0e8ae9e642caad0486b862b48d94ba392256a45
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109839498"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-by-using-an-azure-resource-manager-template-for-a-windows-virtual-machine"></a>Enviar métricas de SO invitado al almacén de métricas de Azure Monitor con una plantilla de Azure Resource Manager para una máquina virtual Windows
Los datos de rendimiento del sistema operativo invitado de las máquinas virtuales de Azure no se recopilan automáticamente como otras [métricas de la plataforma](./monitor-azure-resource.md#monitoring-data). Instale la [extensión de diagnósticos](../agents/diagnostics-extension-overview.md) de Azure Monitor para recopilar métricas del sistema operativo invitado en la base de datos de métricas, de modo que se pueda usar con todas las características de métricas de Azure Monitor, incluidos alertas, gráficos, enrutamiento y acceso casi en tiempo real de una API REST. En este artículo se describe el proceso de envío de métricas de rendimiento del SO invitado para una máquina virtual Windows a la base de datos de métricas mediante una plantilla de Resource Manager.

> [!NOTE]
> Para obtener más información sobre la configuración de la extensión de diagnósticos para recopilar métricas del SO invitado mediante Azure Portal, consulte [Instalación y configuración de la extensión de Azure Diagnostics (WAD) para Windows](../agents/diagnostics-extension-windows-install.md).


Si no está familiarizado con las plantillas de Resource Manager, obtenga información sobre las [implementaciones de plantilla](../../azure-resource-manager/management/overview.md) y su estructura y sintaxis.

## <a name="prerequisites"></a>Prerrequisitos

- La suscripción debe estar registrada en [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md).

- Debe tener instalado [Azure PowerShell](/powershell/azure) o [Azure Cloud Shell](../../cloud-shell/overview.md).

- El recurso de máquina virtual debe estar en una [región que admita métricas personalizadas](./metrics-custom-overview.md#supported-regions).


## <a name="set-up-azure-monitor-as-a-data-sink"></a>Configuración de Azure Monitor como receptor de datos
La extensión Azure Diagnostics usa una característica denominada "receptores de datos" para enrutar las métricas y los registros a distintas ubicaciones. En los pasos siguientes se muestra cómo usar una plantilla de Resource Manager y PowerShell para implementar una VM con el nuevo receptor de datos "Azure Monitor".

## <a name="author-resource-manager-template"></a>Creación de una plantilla de Resource Manager
En este ejemplo, puede usar una plantilla de ejemplo disponible públicamente. Las plantillas iniciales se encuentran en https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-simple-windows.

- **Azuredeploy.json** es una plantilla de Resource Manager configurada previamente para la implementación de una máquina virtual.

- **Azuredeploy.Parameters.json** es un archivo de parámetros que almacena información como el nombre de usuario y la contraseña que le gustaría establecer para la VM. Durante la implementación, la plantilla de Resource Manager usa los parámetros establecidos en este archivo.

Descargue y guarde ambos archivos localmente.

### <a name="modify-azuredeployparametersjson"></a>Modificación de azuredeploy.parameters.json
Abra el archivo *azuredeploy.parameters.json*.

1. Especifique los valores de **adminUsername** y **adminPassword** para la VM. Estos parámetros se utilizan para el acceso remoto a la VM. Para evitar el secuestro de la VM, NO use los valores de esta plantilla. Los bots buscan por Internet los nombres de usuario y las contraseñas en repositorios públicos de GitHub. Es probable que sean VM de prueba con estos valores predeterminados.

1. Cree un dnsname único para la VM.

### <a name="modify-azuredeployjson"></a>Modificación de azuredeploy.json

Abra el archivo *azuredeploy.json*.

Agregue un identificador de cuenta de almacenamiento a la sección de **variables** de la plantilla después de la entrada para **storageAccountName**.

```json
// Find these lines.
"variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",

// Add this line directly below.
    "accountid": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
```

Agregue esta extensión de Managed Service Identity (MSI) a la plantilla en la parte superior de la sección **resources**. La extensión garantiza que Azure Monitor acepte las métricas que se emiten.

```json
//Find this code.
"resources": [
// Add this code directly below.
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(variables('vmName'), '/', 'WADExtensionSetup')]",
        "apiVersion": "2017-12-01",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]" ],
        "properties": {
            "publisher": "Microsoft.ManagedIdentity",
            "type": "ManagedIdentityExtensionForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "port": 50342
            }
        }
    },
```

Agregue la configuración de **identity** en el recurso de VM para asegurarse de que Azure asigne a la extensión MSI una identidad del sistema. Este paso garantiza que la VM pueda emitir métricas de invitado sobre sí misma a Azure Monitor.

```json
// Find this section
                "subnet": {
            "id": "[variables('subnetRef')]"
            }
        }
        }
    ]
    }
},
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    // add these 3 lines below
    "identity": {
    "type": "SystemAssigned"
    },
    //end of added lines
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
    "hardwareProfile": {
    ...
```

Agregue la siguiente configuración para habilitar la extensión de diagnóstico en una máquina virtual Windows. En una máquina virtual sencilla basada en Resource Manager, se puede agregar la configuración de extensión a la matriz de recursos de la máquina virtual. La línea "sinks"&mdash; "AzMonSink" y la correspondiente "SinksConfig" más adelante en la sección&mdash;permiten que la extensión emita métricas directamente a Azure Monitor. No dude en agregar o quitar contadores de rendimiento según sea necesario.


```json
        "networkProfile": {
            "networkInterfaces": [
            {
                "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
            }
            ]
        },
"diagnosticsProfile": {
    "bootDiagnostics": {
    "enabled": true,
    "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
    }
}
},
//Start of section to add
"resources": [
{
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "name": "[concat(variables('vmName'), '/', 'Microsoft.Insights.VMDiagnosticsSettings')]",
            "apiVersion": "2017-12-01",
            "location": "[resourceGroup().location]",
            "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
            ],
            "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.12",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "WadCfg": {
                "DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "DiagnosticInfrastructureLogs": {
                    "scheduledTransferLogLevelFilter": "Error"
        },
                    "Directories": {
                    "scheduledTransferPeriod": "PT1M",
    "IISLogs": {
                        "containerName": "wad-iis-logfiles"
                    },
                    "FailedRequestLogs": {
                        "containerName": "wad-failedrequestlogs"
                    }
                    },
                    "PerformanceCounters": {
                    "scheduledTransferPeriod": "PT1M",
                    "sinks": "AzMonSink",
                    "PerformanceCounterConfiguration": [
                        {
                        "counterSpecifier": "\\Memory\\Available Bytes",
                        "sampleRate": "PT15S"
                        },
                        {
                        "counterSpecifier": "\\Memory\\% Committed Bytes In Use",
                        "sampleRate": "PT15S"
                        },
                        {
                        "counterSpecifier": "\\Memory\\Committed Bytes",
                        "sampleRate": "PT15S"
                        }
                    ]
                    },
                    "WindowsEventLog": {
                    "scheduledTransferPeriod": "PT1M",
                    "DataSource": [
                        {
                        "name": "Application!*"
                        }
                    ]
                    },
                    "Logs": {
                    "scheduledTransferPeriod": "PT1M",
                    "scheduledTransferLogLevelFilter": "Error"
                    }
                },
                "SinksConfig": {
                    "Sink": [
                    {
                        "name" : "AzMonSink",
                        "AzureMonitor" : {}
                    }
                    ]
                }
                },
                "StorageAccount": "[variables('storageAccountName')]"
            },
            "protectedSettings": {
                "storageAccountName": "[variables('storageAccountName')]",
                "storageAccountKey": "[listKeys(variables('accountid'),'2015-06-15').key1]",
                "storageAccountEndPoint": "https://core.windows.net/"
            }
            }
        }
        ]
//End of section to add
```


Guarde y cierre ambos archivos.


## <a name="deploy-the-resource-manager-template"></a>Implementación de la plantilla de Resource Manager

> [!NOTE]
> Debe estar ejecutando la versión de la extensión 1.5 o superior de Azure Diagnostics Y tener la propiedad **autoUpgradeMinorVersion**: establecida en "true" en la plantilla de Resource Manager. A continuación, Azure carga la extensión adecuada cuando se inicia la VM. Si no tiene estas opciones en la plantilla, modifíquelas y vuelva a implementar la plantilla.


Para implementar la plantilla de Resource Manager, se usa Azure PowerShell.

1. Inicie PowerShell.
1. Inicie sesión en Azure con `Login-AzAccount`.
1. Obtenga una lista de suscripciones con `Get-AzSubscription`.
1. Establezca la suscripción que se usa para crear o actualizar la máquina virtual en:

   ```powershell
   Select-AzSubscription -SubscriptionName "<Name of the subscription>"
   ```
1. Para crear un nuevo grupo de recursos para la VM que se está implementando, ejecute el comando siguiente:

   ```powershell
    New-AzResourceGroup -Name "<Name of Resource Group>" -Location "<Azure Region>"
   ```
   > [!NOTE]
   > Recuerde [usar una región de Azure que esté habilitada para las métricas personalizadas](./metrics-custom-overview.md).

1. Ejecute los comandos siguientes para implementar la máquina virtual mediante la plantilla de Resource Manager.
   > [!NOTE]
   > Si quiere actualizar una VM existente, simplemente agregue *-Mode Incremental* al final del comando siguiente.

   ```powershell
   New-AzResourceGroupDeployment -Name "<NameThisDeployment>" -ResourceGroupName "<Name of the Resource Group>" -TemplateFile "<File path of your Resource Manager template>" -TemplateParameterFile "<File path of your parameters file>"
   ```

1. Una vez que la implementación se realice correctamente, la máquina virtual debe estar en Azure Portal, emitiendo métricas a Azure Monitor.

   > [!NOTE]
   > Puede que surjan errores en torno al parámetro vmSkuSize seleccionado. Si ocurre, vuelva al archivo azuredeploy.json y actualice el valor predeterminado del parámetro vmSkuSize. En este caso, se recomienda probar "Standard_DS1_v2").

## <a name="chart-your-metrics"></a>Gráfico de las métricas

1. Inicie sesión en el Portal de Azure.

2. En el menú de la izquierda, seleccione **Monitor**.

3. En la página Monitor, seleccione **Métricas**.

   ![Página Métricas](media/collect-custom-metrics-guestos-resource-manager-vm/metrics.png)

4. Cambie el período de agregación a **Últimos 30 minutos**.

5. En el menú desplegable de recursos, seleccione la VM que ha creado. Si no ha cambiado el nombre de la plantilla, debe ser *SimpleWinVM2*.

6. En el menú desplegable de espacios de nombres, seleccione **azure.vm.windows.guest**.

7. En el menú desplegable de métricas, seleccione **Memory\%Committed Bytes in Use** (Memoria%Bytes confirmados en uso).


## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de las [métricas personalizadas](./metrics-custom-overview.md).