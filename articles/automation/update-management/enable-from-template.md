---
title: Habilitación de Update Management mediante una plantilla de Azure Resource Manager
description: En este artículo se indica cómo usar una plantilla de Azure Resource Manager para habilitar Update Management.
services: automation
ms.subservice: update-management
ms.topic: conceptual
ms.date: 09/18/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 246c86b2e1f2e49af22d290ccb0a80f3e8c98c64
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958215"
---
# <a name="enable-update-management-using-azure-resource-manager-template"></a>Habilitación de Update Management mediante una plantilla de Azure Resource Manager

Puede usar una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md) para habilitar la característica de Update Management de Azure Automation en un grupo de recursos. En este artículo se proporciona una plantilla de ejemplo que automatiza lo siguiente:

* Automatiza la creación de un área de trabajo de Log Analytics en Azure Monitor.
* Automatiza la creación de una cuenta de Azure Automation.
* El vínculo de la cuenta de Automation al área de trabajo de Log Analytics.
* Agrega runbooks de Automation de ejemplo a la cuenta.
* Habilita la característica Update Management.

Si ya tiene un área de trabajo de Log Analytics y una cuenta de Automation implementada en una región admitida de la suscripción, no se vinculan. Con esta plantilla se crea correctamente el vínculo y se implementa Update Management.

>[!NOTE]
>No se admite la creación de una cuenta de ejecución de Automation cuando se usa una plantilla de ARM. Para crear una cuenta de ejecución manualmente desde el portal o mediante PowerShell, consulte cómo [crear una cuenta de ejecución](../create-run-as-account.md).

Después de completar estos pasos, debe [Configurar las opciones de diagnóstico](../automation-manage-send-joblogs-log-analytics.md) para que la cuenta de Automation envíe el estado del trabajo de runbook y los flujos de trabajo al área de trabajo de Log Analytics vinculada.

## <a name="api-versions"></a>Versiones de API

En la tabla siguiente se muestra la versión de API de los recursos usados en este ejemplo.

| Resource | Tipo de recurso | Versión de API |
|:---|:---|:---|
| [Área de trabajo](/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 01-03-2020 versión preliminar |
| [Cuenta de Automation](/azure/templates/microsoft.automation/automationaccounts) | automation | 2020-01-13-preview |
| [Servicios vinculados del área de trabajo](/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 01-03-2020 versión preliminar |
| [Soluciones](/azure/templates/microsoft.operationsmanagement/solutions) | solutions | 2015-11-01-preview |

## <a name="before-using-the-template"></a>Antes de usar la plantilla

La plantilla JSON se configura para pedir al usuario:

* El nombre del área de trabajo.
* La región en la que crear un área de trabajo.
* El nombre de la cuenta de Automation.
* La región en la que crear la cuenta de Automation.

Los siguientes parámetros de la plantilla se configuran con un valor predeterminado para el área de trabajo de Log Analytics:

* *SKU* el valor predeterminado es el plan de tarifa por GB publicado en el modelo de precios de abril de 2018.
* *dataRetention* tiene como valor predeterminado 30 días.

>[!WARNING]
>Si quiere crear o configurar un área de trabajo de Log Analytics en una suscripción que ha elegido el modelo de precios de abril de 2018, el único plan de tarifa válido de Log Analytics es *PerGB2018*.
>

La plantilla JSON especifica un valor predeterminado para el resto de parámetros que es probable que se utilice como configuración estándar en el entorno. Puede almacenar la plantilla en una cuenta de Azure Storage para el acceso compartido en la organización. Para más información sobre el trabajo con plantillas, consulte [Implementación de recursos con plantillas de ARM y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md).

Si no está familiarizado con Azure Automation y Azure Monitor, es importante que comprenda los siguientes detalles de configuración. Pueden ayudarle a evitar errores al intentar crear, configurar y usar un área de trabajo de Log Analytics vinculada a la nueva cuenta de Automation.

* Revise los [detalles adicionales](../../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace) para conocer totalmente las opciones de configuración del área de trabajo, como el modo de control de acceso, el plan de tarifa, la retención y el nivel de reserva de capacidad.

* Revise las [asignaciones de área de trabajo](../how-to/region-mappings.md) para especificar las regiones admitidas en línea o en un archivo de parámetros. Solo en determinadas regiones se puede vincular un área de trabajo de Log Analytics y una cuenta de Automation en la suscripción.

* Si no está familiarizado con Azure Monitor registros y aún no ha implementado un área de trabajo, debe revisar la [guía de diseño del área de trabajo](../../azure-monitor/logs/design-logs-deployment.md). Le ayudará a obtener información sobre el control de acceso y a comprender las estrategias de implementación de diseño que recomendamos para su organización.

## <a name="deploy-template"></a>Implementar plantilla

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                    "description": "Workspace name"
                }
            },
            "sku": {
                "type": "string",
                "allowedValues": [
                    "pergb2018",
                    "Free",
                    "Standalone",
                    "PerNode",
                    "Standard",
                    "Premium"
                ],
                "defaultValue": "pergb2018",
                "metadata": {
                    "description": "Pricing tier: perGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium), which are not available to all customers."
                }
            },
            "dataRetention": {
                "type": "int",
                "defaultValue": 30,
                "minValue": 7,
                "maxValue": 730,
                "metadata": {
                    "description": "Number of days to retain data."
                }
            },
            "location": {
                "type": "string",
                "defaultValue": "[resourceGroup().location]",
                "metadata": {
                    "description": "Specifies the location in which to create the workspace."
                }
            },
            "automationAccountName": {
                "type": "string",
                "metadata": {
                    "description": "Automation account name"
                }
            },
            "automationAccountLocation": {
                "type": "string",
                "metadata": {
                    "description": "Specifies the location in which to create the Automation account."
                }
            },
            "sampleGraphicalRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorial"
            },
            "sampleGraphicalRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePowerShellRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialScript"
            },
            "samplePowerShellRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePython2RunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialPython2"
            },
            "samplePython2RunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "_artifactsLocation": {
                "type": "string",
                "defaultValue": "[deployment().properties.templateLink.uri]",
                "metadata": {
                    "description": "URI to artifacts location"
                }
            },
            "_artifactsLocationSasToken": {
                "type": "securestring",
                "defaultValue": "",
                "metadata": {
                    "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated"
                }
            }
        },
        "variables": {
        "Updates": {
            "name": "[concat('Updates', '(', parameters('workspaceName'), ')')]",
            "galleryName": "Updates"
          }
        },
        "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "apiVersion": "2020-03-01-preview",
                "name": "[parameters('workspaceName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "sku": {
                        "name": "[parameters('sku')]"
                    },
                    "retentionInDays": "[parameters('dataRetention')]",
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "location": "[parameters('location')]",
                "name": "[variables('Updates').name]",
                "type": "Microsoft.OperationsManagement/solutions",
                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationsManagement/solutions/', variables('Updates').name)]",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                ],
                "properties": {
                    "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                },
                "plan": {
                    "name": "[variables('Updates').name]",
                    "publisher": "Microsoft",
                    "promotionCode": "",
                    "product": "[concat('OMSGallery/', variables('Updates').galleryName)]"
                }
            },
            {
                "type": "Microsoft.Automation/automationAccounts",
                "apiVersion": "2020-01-13-preview",
                "name": "[parameters('automationAccountName')]",
                "location": "[parameters('automationAccountLocation')]",
                "dependsOn": [
                    "[parameters('workspaceName')]"
                ],
                "properties": {
                    "sku": {
                        "name": "Basic"
                    }
                },
                "resources": [
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('sampleGraphicalRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "GraphPowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('sampleGraphicalRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.graphrunbook', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePowerShellRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "PowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePowerShellRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.ps1', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePython2RunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "Python2",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePython2RunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorialPython2.py', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    }
                ]
            },
            {
                "type": "Microsoft.OperationalInsights/workspaces/linkedServices",
                "apiVersion": "2020-03-01-preview",
                "name": "[concat(parameters('workspaceName'), '/' , 'Automation')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[parameters('workspaceName')]",
                    "[parameters('automationAccountName')]"
                ],
                "properties": {
                    "resourceId": "[resourceId('Microsoft.Automation/automationAccounts', parameters('automationAccountName'))]"
                }
            }
        ]
    }
    ```

2. Edite la plantilla para adecuarla a sus requisitos. Plantéese la posibilidad de crear un [archivo de parámetros de Resource Manager](../../azure-resource-manager/templates/parameter-files.md) en lugar de pasar los parámetros como valores insertados.

3. Guarde este archivo como **deployUMSolutiontemplate.json** en una carpeta local.

4. Está listo para implementar esta plantilla. Puede usar la CLI de Azure o PowerShell. Cuando se le pida un nombre de cuenta de Automation o un área de trabajo, proporcione un nombre que sea globalmente único en todas las suscripciones de Azure.

    **PowerShell**

    ```powershell
    New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile deployUMSolutiontemplate.json
    ```

    **CLI de Azure**

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deployUMSolutiontemplate.json
    ```

    La implementación puede demorar unos minutos en completarse. Cuando termine, verá un mensaje similar al siguiente que incluye el resultado:

    ![Resultado de ejemplo cuando se completa la implementación](media/enable-from-template/template-output.png)

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, abra la cuenta de Automation que ha creado.

3. En el panel izquierdo, seleccione **Runbooks**. En la página **Runbooks**, se muestran tres runbooks de tutorial creados con la cuenta de Automation.

    ![Runbooks de tutorial creados con la cuenta de Automation](../media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. En el panel izquierdo, seleccione **Área de trabajo vinculada**. En la página **Área de trabajo vinculada**, se muestra el área de trabajo de Log Analytics que especificó anteriormente como vinculada a la cuenta de Automation.

    ![Cuenta de Automation vinculada al área de trabajo Log Analytics](../media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

5. En el panel izquierdo, seleccione **Update Management**. En la página **Update Management** se muestra la página de la evaluación sin información, porque se acaba de habilitar y las máquinas no están configuradas para la administración.

    ![Vista de evaluación de características de Update Management](./media/enable-from-template/update-management-assessment-view.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no las necesite, elimine la solución **Updates** en el área de trabajo de Log Analytics, desvincule la cuenta de Automation del área de trabajo y, a continuación, elimine la cuenta de Automation y el área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

* Para usar Update Management para máquinas virtuales, consulte [Administración de actualizaciones y revisiones para las máquinas virtuales](manage-updates-for-vm.md).

* Si ya no desea usar Update Management y desea quitarlo, consulte las instrucciones en [Eliminación de la característica Update Management](remove-feature.md).

* Para eliminar máquinas virtuales de Update Management, consulte [Eliminación de una máquina virtual desde Update Management](remove-vms.md).