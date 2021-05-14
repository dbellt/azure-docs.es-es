---
title: Habilitación del estado de invitado de VM Insights (versión preliminar)
description: Se describe cómo habilitar el estado de invitado de VM Insights en la suscripción y cómo incorporar máquinas virtuales.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 04/05/2021
ms.custom: references_regions, devx-track-azurepowershell
ms.openlocfilehash: 472700f50175c115305bad332efa6b0e77fb74ce
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315298"
---
# <a name="enable-vm-insights-guest-health-preview"></a>Habilitación del estado de invitado de VM Insights (versión preliminar)
El estado de invitado de VM Insights permite ver el estado de una máquina virtual conforme a la definición de un conjunto de medidas de rendimiento que se muestrean a intervalos regulares. En este artículo se explica cómo habilitar esta característica en la suscripción, así como la supervisión de invitado en cada máquina virtual.

## <a name="current-limitations"></a>Limitaciones actuales
El estado de invitado de VM Insights tiene las siguientes limitaciones en la versión preliminar pública:

- Actualmente solo se admiten máquinas virtuales de Azure. Azure Arc para servidores no se admite de momento.
- Los proxies de red no se admiten de momento.


## <a name="supported-operating-systems"></a>Sistemas operativos admitidos
La máquina virtual debe ejecutar uno de los siguientes sistemas operativos: 

  - CentOS 7.5, 7.6, 7.7, 7.8, 7.9
  - RedHat 7.5, 7.6, 7.7, 7.8, 7.9
  - Ubuntu 16.04 LTS, Ubuntu 18.04 LTS
  - Windows Server 2012 o superior

## <a name="supported-regions"></a>Regiones admitidas

La máquina virtual debe encontrarse en una de las siguientes regiones:

- Centro de Australia
- Este de Australia
- Sudeste de Australia
- Centro de Canadá
- Centro de la India
- Centro de EE. UU.
- Este de Asia
- Este de EE. UU.
- Este de EE. UU. 2
- EUAP de Este de EE. UU. 2
- Centro de Francia
- Centro-oeste de Alemania
- Japón Oriental
- Centro de Corea del Sur
- Centro-Norte de EE. UU
- Norte de Europa
- Centro-sur de EE. UU.
- Norte de Sudáfrica
- Sudeste de Asia
- Norte de Suiza
- Sur de Reino Unido
- Oeste de Reino Unido
- Centro-Oeste de EE. UU.
- Oeste de Europa
- Oeste de EE. UU.
- Oeste de EE. UU. 2


El área de trabajo de Log Analytics debe encontrarse en una de las siguientes regiones:

- Centro de Australia
- Este de Australia
- Sudeste de Australia
- Centro de Canadá
- Canadá India
- Centro de EE. UU.
- Este de Asia
- Este de EE. UU.
- Este de EE. UU. 2
- EUAP de Este de EE. UU. 2
- Centro de Francia
- Japón Oriental
- Centro-Norte de EE. UU
- Norte de Europa
- Centro-sur de EE. UU.
- Sudeste de Asia
- Norte de Suiza
- Sur de Reino Unido
- Región Oeste de Europa
- Oeste de EE. UU.
- Oeste de EE. UU. 2

## <a name="prerequisites"></a>Prerrequisitos

- La máquina virtual se debe incorporar a VM Insights.
- El usuario que ejecuta los pasos de incorporación debe tener un acceso mínimo de nivel Colaborador a la suscripción en la que se encuentran la máquina virtual y la regla de recopilación de datos.
- Los proveedores de recursos de Azure necesarios deben registrarse como se explica en la sección siguiente.

## <a name="register-required-azure-resource-providers"></a>Registro de proveedores de recursos de Azure necesarios
Los siguientes proveedores de recursos de Azure se deben registrar en la suscripción para habilitar el estado de invitado de VM Insights. 

- Microsoft.WorkloadMonitor
- Microsoft.Insights

Puede usar cualquiera de los métodos disponibles para registrar un proveedor de recursos como se explica en [Tipos y proveedores de recursos de Azure](../../azure-resource-manager/management/resource-providers-and-types.md). También puede usar el siguiente comando de ejemplo con ARMClient, Postman u otro método para realizar llamadas autenticadas a Azure Resource Manager:

```
POST https://management.azure.com/subscriptions/[subscriptionId]/providers/Microsoft.WorkloadMonitor/register?api-version=2019-10-01
POST https://management.azure.com/subscriptions/[subscriptionId]/providers/Microsoft.Insights/register?api-version=2019-10-01
```


## <a name="enable-a-virtual-machine-using-the-azure-portal"></a>Habilitación de una máquina virtual mediante Azure Portal
Cuando se habilita el estado de invitado en una máquina virtual en Azure Portal, toda la configuración necesaria se realiza automáticamente. Esto incluye la creación de la regla de recopilación de datos necesaria, la instalación de la extensión de estado de invitado en la máquina virtual y la creación de una asociación con la regla de recopilación de datos.

En la vista **Introducción** de VM Insights, haga clic en el vínculo situado junto al mensaje de actualización de una máquina virtual y luego en el botón **Actualizar**. También puede seleccionar varias máquinas virtuales para actualizarlas a la vez.

![Habilitación de la característica de estado en una máquina virtual](media/vminsights-health-enable/enable-agent.png)


## <a name="enable-a-virtual-machine-using-resource-manager-template"></a>Habilitación de una máquina virtual mediante una plantilla de Resource Manager
Hay tres pasos necesarios para habilitar máquinas virtuales mediante Azure Resource Manager.

- Cree una regla de recopilación de datos.
- Instalación de la extensión de estado de invitado en cada máquina virtual
- Cree una asociación entre la máquina virtual y la regla de recopilación de datos.

### <a name="create-data-collection-rule-dcr"></a>Creación de una regla de recopilación de datos (DCR)

> [!NOTE]
> Si habilita una máquina virtual mediante Azure Portal, la regla de recopilación de datos que se describe aquí se crea automáticamente. En ese caso, no es necesario realizar este paso.

La configuración de los monitores del estado de invitado de VM Insights se almacena en [Reglas de recopilación de datos (DCR)](../agents/data-collection-rule-overview.md). Cada máquina virtual con la extensión de estado de invitado necesita una asociación con esta regla.

> [!NOTE]
> Puede crear reglas de recopilación de datos adicionales para modificar la configuración predeterminada de los monitores, como se describe en [Configuración de la supervisión en el estado de invitado de VM Insights (versión preliminar)](vminsights-health-configure.md).

La plantilla necesita valores para los siguientes parámetros:

- **defaultHealthDataCollectionRuleName**: mantenga el nombre predeterminado definido en la plantilla.
- **destinationWorkspaceResourceId**: identificador de recurso del área de trabajo de Log Analytics usada para la recopilación de datos de máquina virtual.
- **dataCollectionRuleLocation**: región de la regla de recopilación de datos. Debe coincidir con la región del área de trabajo de Log Analytics.


Implemente la plantilla mediante cualquier [método de implementación de plantillas de Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md). Los siguientes comandos implementan la plantilla y el archivo de parámetros mediante PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
New-AzResourceGroupDeployment -Name GuestHealthDataCollectionRule -ResourceGroupName my-resource-group -TemplateFile Health.DataCollectionRule.template.json -TemplateParameterFile Health.DataCollectionRule.template.parameters.json
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create --name GuestHealthDataCollectionRule --resource-group my-resource-group --template-file Health.DataCollectionRule.template.json --parameters Health.DataCollectionRule.template.parameters.json
```

---

La regla de recopilación de datos definida en la plantilla de Resource Manager siguiente habilita todos los monitores de las máquinas virtuales con la extensión de estado de invitado. Debe incluir los orígenes de datos de cada uno de los contadores de rendimiento que usen los monitores.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "defaultHealthDataCollectionRuleName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the data collection rule to create."
      },
      "defaultValue": "Microsoft-VMInsights-Health"
    },
    "destinationWorkspaceResourceId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the Azure resource ID of the Log Analytics workspace to use to store virtual machine health data."
      }
    },
    "dataCollectionRuleLocation": {
      "type": "string",
      "metadata": {
        "description": "The location code in which the data collection rule should be deployed. Examples: eastus, westeurope, etc"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Insights/dataCollectionRules",
      "name": "[parameters('defaultHealthDataCollectionRuleName')]",
      "location": "[parameters('dataCollectionRuleLocation')]",
      "apiVersion": "2019-11-01-preview",
      "properties": {
        "description": "Data collection rule for VM Insights health.",
        "dataSources": {
          "performanceCounters": [
              {
                  "name": "VMHealthPerfCounters",
                  "streams": [ "Microsoft-Perf" ],
                  "scheduledTransferPeriod": "PT1M",
                  "samplingFrequencyInSeconds": 60,
                  "counterSpecifiers": [
                      "\\LogicalDisk(*)\\% Free Space",
                      "\\Memory\\Available Bytes",
                      "\\Processor(_Total)\\% Processor Time"
                  ]
              }
          ],
          "extensions": [
            {
              "name": "Microsoft-VMInsights-Health",
              "streams": [
                "Microsoft-HealthStateChange"
              ],
              "extensionName": "HealthExtension",
              "extensionSettings": {
                "schemaVersion": "1.0",
                "contentVersion": "",
                "healthRuleOverrides": [
                  {
                    "scopes": [ "*" ],
                    "monitors": ["root"],
                    "alertConfiguration": {
                      "isEnabled": true
                    }
                  }
                ]
              },
              "inputDataSources": [
                  "VMHealthPerfCounters"
              ]

            }
          ]
        },
        "destinations": {
          "logAnalytics": [
            {
              "workspaceResourceId": "[parameters('destinationWorkspaceResourceId')]",
              "name": "Microsoft-HealthStateChange-Dest"
            }
          ]
        },                  
        "dataFlows": [
          {
            "streams": [
              "Microsoft-HealthStateChange"
            ],
            "destinations": [
              "Microsoft-HealthStateChange-Dest"
            ]
          }
        ]
      }
    }
  ]
}
```

### <a name="sample-parameter-file"></a>Archivo de parámetros de ejemplo

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "destinationWorkspaceResourceId": {
        "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace"
      },
      "dataCollectionRuleLocation": {
        "value": "eastus"
      }
  }
}
```



### <a name="install-guest-health-extension-and-associate-with-data-collection-rule"></a>Instalación de la extensión de estado de invitado y asociación con una regla de recopilación de datos
Use la siguiente plantilla de Resource Manager para habilitar una máquina virtual para el estado de invitado. Esto instala la extensión de estado de invitado y crea la asociación con la regla de recopilación de datos. Puede implementar esta plantilla mediante cualquier [método de implementación de plantillas de Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md).


Por ejemplo, use los siguientes comandos para implementar el archivo de plantilla y parámetros en un grupo de recursos mediante PowerShell o la CLI de Azure.


# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
New-AzResourceGroupDeployment -Name GuestHealthDeployment -ResourceGroupName my-resource-group -TemplateFile azure-monitor-deploy.json -TemplateParameterFile azure-monitor-deploy.parameters.json
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create --name GuestHealthDeployment --resource-group my-resource-group --template-file Health.VirtualMachine.template.json --parameters Health.VirtualMachine.template.parameters.json
```

---

### <a name="template-file"></a>Archivo de plantilla

``` json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualMachineName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the virtual machine."
      }
    },
    "virtualMachineLocation": {
      "type": "string",
      "metadata": {
        "description": "The location code of the virtual machine region (location). Examples: eastus, westeurope, etc"
      }
    },
    "virtualMachineOsType": {
      "type": "string",
      "metadata": {
        "description": "Specifies operating system type of the target virtual machine."
      },
      "allowedValues": ["windows", "linux"]
    },
    "dataCollectionRuleAssociationName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the data collection rule association to create."
      },
      "defaultValue": "VM-Health-Dcr-Association"
    },
    "healthDataCollectionRuleResourceId": {
      "type": "string",
      "metadata": {
        "description": "Specifies resource id of Azure Monitor Data Collection Rule for virtual machine health data."
      }
    }
  },
  "variables": {
    "healthExtensionProperties": {
      "windows": {
        "publisher": "Microsoft.Azure.Monitor.VirtualMachines.GuestHealth",
        "type": "GuestHealthWindowsAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true
      },
      "linux": {
        "publisher": "Microsoft.Azure.Monitor.VirtualMachines.GuestHealth",
        "type": "GuestHealthLinuxAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true
      }
    },
    "azureMonitorAgentExtensionProperties": {
      "windows": {
        "publisher": "Microsoft.Azure.Monitor", 
        "type": "AzureMonitorWindowsAgent", 
        "typeHandlerVersion": "1.0", 
        "autoUpgradeMinorVersion": false 
      },
      "linux": {
        "publisher": "Microsoft.Azure.Monitor", 
        "type": "AzureMonitorLinuxAgent", 
        "typeHandlerVersion": "1.5", 
        "autoUpgradeMinorVersion": false 
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('virtualMachineName')]",
      "location": "[parameters('virtualMachineLocation')]",
      "apiVersion": "2017-03-30",
      "identity": {
        "type": "SystemAssigned"
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines/providers/dataCollectionRuleAssociations",
      "name": "[concat(parameters('virtualMachineName'), '/Microsoft.Insights/', parameters('dataCollectionRuleAssociationName'))]",
      "apiVersion": "2019-11-01-preview",
      "properties": {
        "description": "Association of data collection rule for VM Insights Health.",
        "dataCollectionRuleId": "[parameters('healthDataCollectionRuleResourceId')]"
      }
    },
    { 
      "type": "Microsoft.Compute/virtualMachines/extensions", 
      "apiVersion": "2019-12-01", 
      "name": "[concat(parameters('virtualMachineName'), '/', variables('azureMonitorAgentExtensionProperties')[parameters('virtualMachineOsType')].type)]",
      "location": "[parameters('virtualMachineLocation')]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines/providers/dataCollectionRuleAssociations', parameters('virtualMachineName'), 'Microsoft.Insights', parameters('dataCollectionRuleAssociationName'))]"
      ], 
      "properties": "[variables('azureMonitorAgentExtensionProperties')[parameters('virtualMachineOsType')]]"
    },
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('virtualMachineName'), '/', variables('healthExtensionProperties')[parameters('virtualMachineOsType')].type)]",
      "location": "[parameters('virtualMachineLocation')]",
      "apiVersion": "2018-06-01",
      "dependsOn": [ 
        "[resourceId('Microsoft.Compute/virtualMachines/extensions', parameters('virtualMachineName'), variables('azureMonitorAgentExtensionProperties')[parameters('virtualMachineOsType')].type)]",
        "[resourceId('Microsoft.Compute/virtualMachines/providers/dataCollectionRuleAssociations', parameters('virtualMachineName'), 'Microsoft.Insights', parameters('dataCollectionRuleAssociationName'))]"
      ],    
      "properties": "[variables('healthExtensionProperties')[parameters('virtualMachineOsType')]]"
    }               
  ]
}
```

### <a name="sample-parameter-file"></a>Archivo de parámetros de ejemplo

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "virtualMachineName": {
        "value": "myvm"
      },
      "virtualMachineLocation": {
        "value": "eastus"
      },
      "virtualMachineOsType": {
        "value": "linux"
      },
      "healthDataCollectionRuleResourceId": {
        "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Insights/dataCollectionRules/Microsoft-VMInsights-Health"
      }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Personalización de monitores habilitados por VM Insights](vminsights-health-configure.md)
