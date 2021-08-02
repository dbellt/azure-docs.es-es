---
title: Habilitación de Azure Monitor en un dispositivo de Azure Stack Edge Pro con GPU
description: Se describe cómo habilitar Azure Monitor en un dispositivo de Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 06/03/2021
ms.author: alkohli
ms.openlocfilehash: bbb4ce6442cc38e2554c6c1f745223a193ca793d
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950750"
---
# <a name="enable-azure-monitor-on-your-azure-stack-edge-pro-gpu-device"></a>Habilitación de Azure Monitor en un dispositivo de Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

La supervisión de contenedores en el dispositivo de Azure Stack Edge Pro con GPU es fundamental, especialmente cuando se ejecutan varias aplicaciones de proceso. Azure Monitor le permite recopilar registros de contenedor y métricas de memoria y de procesador desde el clúster de Kubernetes que se ejecuta en el dispositivo.

En este artículo se describen los pasos necesarios para habilitar Azure Monitor en el dispositivo y recopilar los registros de contenedor en el área de trabajo de Log Analytics. Actualmente, el almacén de métricas de Azure Monitor no es compatible con el dispositivo de Azure Stack Edge Pro con GPU. 

> [!NOTE]
> Si Azure Arc está habilitado en el clúster de Kubernetes del dispositivo, siga los pasos descritos en [Azure Monitor Container Insights para contenedores con clústeres de Kubernetes habilitado para Azure Arc](../azure-monitor/containers/container-insights-enable-arc-enabled-clusters.md?toc=%2fazure%2fazure-arc%2fkubernetes%2ftoc.json) para configurar la supervisión de contenedores.


## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, necesitará:

- Un dispositivo Azure Stack Edge Pro. Asegúrese de que el dispositivo se ha activado según los pasos descritos en [Tutorial: Activación del dispositivo](azure-stack-edge-gpu-deploy-activate.md).
- Ya completó el paso **Configurar el proceso** según el [tutorial: Configure el proceso en su dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md). Su dispositivo debe tener un recurso de IoT Hub asociado, un dispositivo IoT y un dispositivo IoT Edge.


## <a name="create-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

Realice los siguientes pasos para crear un área de trabajo de Log Analytics. Un área de trabajo de Log Analytics es la unidad de almacenamiento lógica en la que se recopilan y almacenan los datos de registro.

1. En Azure Portal, seleccione **+ Crear un recurso**, busque un **área de trabajo de Log Analytics** y seleccione **Crear**. 
1. En **Crear un área de trabajo de Log Analytics**, configure las siguientes opciones. Acepte el resto como valor predeterminado.

    1. En la pestaña **Datos básicos**,indique la suscripción, el grupo de recursos, el nombre y la región del área de trabajo. 

        ![Pestaña Datos básicos del área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-basics-1.png)  

    1. En la pestaña **Plan de tarifa**, acepte el **plan de pago por uso** predeterminado.

        ![Pestaña Plan de tarifa del área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-pricing-1.png) 

    1. En la pestaña **Revisar y crear**, revise la información del área de trabajo y seleccione **Crear**.

        ![Pestaña Revisar y crear del área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-review-create-1.png)

Para obtener más información, consulte los pasos detallados que se indican en [Creación de un área de trabajo de Log Analytics a través de Azure Portal](../azure-monitor/logs/quick-create-workspace.md).



## <a name="enable-container-insights"></a>Habilitación de la información de contenedor

Siga estos pasos para habilitar la información de contenedor en el área de trabajo. 

1. Siga los pasos detallados que se indican en [Cómo agregar la solución Azure Monitor para contenedores](../azure-monitor/containers/container-insights-hybrid-setup.md#how-to-add-the-azure-monitor-containers-solution). Use el siguiente archivo de plantilla `containerSolution.json`:

    ```yml
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
            }
        ]
    }
    ```

1. Obtenga el id. de recurso y la ubicación. Ir a `Your Log Analytics workspace > General > Properties`. Copie la siguiente información:

    - **Id. de recurso**, que es el identificador de recurso completo de Azure del área de trabajo de Azure Log Analytics. 
    - **Ubicación**, que corresponde a la región de Azure.

    ![Propiedades del área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-properties-1.png) 

1. Utilice el siguiente archivo de parámetros `containerSolutionParams.json`. Reemplace `workspaceResourceId` por el identificador de recurso y `workspaceRegion` por la ubicación copiada en el paso anterior.

    ```yaml
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "workspaceResourceId": {
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
        },
        "workspaceRegion": {
        "value": "westus"
        }
        }
    }
    ```
    
    A continuación, se muestra una salida de ejemplo de un área de trabajo Log Analytics con información de contenedor habilitada:
    
    ```output
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...
    MOTD: Switch to Bash from PowerShell: bash
    VERBOSE: Authenticating to Azure ...
    VERBOSE: Building your Azure drive ...
    
    PS /home/myaccount> az account set -s fa68082f-8ff7-4a25-95c7-ce9da541242f
    PS /home/myaccount> ls
    clouddrive  containerSolution.json
    PS /home/myaccount> ls
    clouddrive  containerSolution.json  containerSolutionParams.json
    PS /home/myaccount> az deployment group create --resource-group myaserg --name Testdeployment1 --template-file containerSolution.json --parameters containerSolutionParams.json
    {- Finished ..
        "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.Resources/deployments/Testdeployment1",
        "location": null,
        "name": "Testdeployment1",
        "properties": {
        "correlationId": "3a9045fe-2de0-428c-b17b-057508a8c575",
        "debugSetting": null,
        "dependencies": [],
        "duration": "PT11.1588316S",
        "error": null,
        "mode": "Incremental",
        "onErrorDeployment": null,
        "outputResources": [
            {
            "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.OperationsManagement/solutions/ContainerInsights(myaseloganalyticsws)",
            "resourceGroup": "myaserg"
            }
        ],
        "outputs": null,
        "parameters": {
            "workspaceRegion": {
            "type": "String",
            "value": "westus"
            },
            "workspaceResourceId": {
            "type": "String",
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
            }
        },
        "parametersLink": null,
        "providers": [
            {
            "id": null,
            "namespace": "Microsoft.Resources",
            "registrationPolicy": null,
            "registrationState": null,
            "resourceTypes": [
                {
                "aliases": null,
                "apiProfiles": null,
                "apiVersions": null,
                "capabilities": null,
                "defaultApiVersion": null,
                "locations": [
                    null
                ],
                "properties": null,
                "resourceType": "deployments"
                }
            ]
            }
        ],
        "provisioningState": "Succeeded",
        "templateHash": "10500027184662969395",
        "templateLink": null,
        "timestamp": "2020-11-06T22:09:56.908983+00:00",
        "validatedResources": null
        },
        "resourceGroup": "myaserg",
        "tags": null,
        "type": "Microsoft.Resources/deployments"
    }
    PS /home/myaccount>
    ```

## <a name="configure-azure-monitor-on-your-device"></a>Configuración de Azure Monitor en el dispositivo

1. Vaya al recurso de Log Analytics recién creado y copie el **id. de área de trabajo** y la **clave principal** (clave del área de trabajo).

    ![Administración de agentes en el área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-agents-management-1.png)

    Guarde esta información, ya que la usará en un paso posterior.

1. [Conexión a la interfaz de PowerShell del dispositivo](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 
    
1. Use el identificador y la clave del área de trabajo de Log Analytics con el siguiente cmdlet:

    `Set-HcsKubernetesAzureMonitorConfiguration -WorkspaceId <> -WorkspaceKey <>`

1. Una vez habilitado Azure Monitor, debería ver los registros en el área de trabajo Log Analytics. Para ver el estado del clúster Kubernetes implementado en el dispositivo, vaya a **Azure Monitor > Información > Contenedores**. En la opción de entorno, seleccione **Todos**. 

    ![Métricas en el área de trabajo de Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-metrics-1.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la [Supervisión de las cargas de trabajo de Kubernetes a través del panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).
- Aprenda a [administrar las notificaciones de alertas de eventos de dispositivos](azure-stack-edge-gpu-manage-device-event-alert-notifications.md).