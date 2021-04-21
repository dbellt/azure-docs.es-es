---
title: Creación de un área de trabajo de Log Analytics mediante la CLI de Azure | Microsoft Docs
description: Aprenda a crear un área de trabajo de Log Analytics para habilitar soluciones de administración y recopilación de datos en sus entornos tanto locales como en la nube con la CLI de Azure.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020
ms.openlocfilehash: 175473f5abd74fa208962fd94852e9ddedfaf7e3
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105821"
---
# <a name="create-a-log-analytics-workspace-with-azure-cli-20"></a>Creación de un área de trabajo de Log Analytics con la CLI de Azure 2.0

La CLI de Azure 2.0 se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía de inicio rápido se muestra cómo usar la CLI de Azure 2.0 para implementar un área de trabajo de Log Analytics en Azure Monitor. Un área de trabajo de Log Analytics es un entorno único de datos de registro de Azure Monitor. Cada área de trabajo tiene su propio repositorio de datos y configuración, y las soluciones y orígenes de datos están configurados para almacenar sus datos en una determinada área de trabajo. Necesitará un área de trabajo de Log Analytics si tiene intención de recopilar datos de los orígenes siguientes:

* Recursos de Azure de la suscripción  
* Equipos locales supervisados por System Center Operations Manager  
* Colecciones de dispositivos de Configuration Manager  
* Datos de diagnóstico o registro de Azure Storage  

Para otros orígenes, como las máquinas virtuales de Azure y la máquinas virtuales Windows o Linux del entorno, consulte los temas siguientes:

* [Recopilación de datos de máquinas virtuales de Azure](../vm/quick-collect-azurevm.md)
* [Recopilación de datos de un equipo Linux híbrido](../vm/quick-collect-linux-computer.md)
* [Recopilación de datos de un equipo Windows híbrido](../vm/quick-collect-windows-computer.md)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.30 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-workspace"></a>Crear un área de trabajo
Cree un área de trabajo con [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create). En el ejemplo siguiente se creará un área de trabajo en la ubicación *Este de EE. UU.* mediante una plantilla de Resource Manager desde la máquina local. La plantilla JSON está configurada para solicitar solo el nombre del área de trabajo y especifica un valor predeterminado para los restantes parámetros que es probable que se utilice como configuración estándar en su entorno. O bien, puede almacenar la plantilla en una cuenta de Azure Storage para el acceso compartido en su organización. Para más información sobre cómo trabajar con plantillas, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md).

Para obtener información sobre las regiones compatibles, consulte [las regiones en las que Log Analytics está disponible](https://azure.microsoft.com/regions/services/) y busque Azure Monitor desde el campo **Buscar un producto**.

Los siguientes parámetros establecen un valor predeterminado:

* Ubicación: el valor predeterminado es Este de EE. UU.
* SKU: el valor predeterminado es el nuevo plan de tarifa por GB publicado en el modelo de precios de abril de 2018

>[!WARNING]
>Si se crea o configura un área de trabajo de Log Analytics en una suscripción que ha elegido el nuevo modelo de precios de abril de 2018, el único plan de tarifa válido de Log Analytics es **PerGB2018**.
>

### <a name="create-and-deploy-template"></a>Creación e implementación de una plantilla

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "eastus",
              "westus"
            ],
            "defaultValue": "eastus",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        },
        "sku": {
            "type": "String",
            "allowedValues": [
              "Standalone",
              "PerNode",
              "PerGB2018"
            ],
            "defaultValue": "PerGB2018",
            "metadata": {
            "description": "Specifies the service tier of the workspace: Standalone, PerNode, Per-GB"
        }
          }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "Name": "[parameters('sku')]"
                },
                "features": {
                    "searchVersion": 1
                }
            }
          }
       ]
    }
    ```

2. Edite la plantilla para adecuarla a sus requisitos. Consulte la referencia [Plantilla Microsoft.OperationalInsights/workspaces](/azure/templates/microsoft.operationalinsights/2015-11-01-preview/workspaces) para saber qué propiedades y valores son compatibles.
3. Guarde este archivo como **deploylaworkspacetemplate.json** en una carpeta local.   
4. Está listo para implementar esta plantilla. Use los siguientes comandos desde la carpeta que contiene la plantilla. Cuando se le pida un nombre de área de trabajo, proporcione un nombre que sea único en el grupo de recursos.

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deploylaworkspacetemplate.json
    ```

La implementación puede demorar unos minutos en completarse. Cuando termine, verá un mensaje similar al siguiente que incluye el resultado:

![Resultado de ejemplo cuando se completa la implementación](media/quick-create-workspace-cli/template-output-01.png)

## <a name="troubleshooting"></a>Solución de problemas
Al crear un área de trabajo que se eliminó en los últimos 14 días y en [estado de eliminación temporal](../logs/delete-workspace.md#soft-delete-behavior), la operación podría tener un resultado diferente en función de la configuración del área de trabajo:
1. Si proporciona el mismo nombre de área de trabajo, grupo de recursos, suscripción y región que en el área de trabajo eliminado, se recuperará el área de trabajo, incluidos los datos, la configuración y los agentes conectados.
2. Este nombre debe ser único por cada grupo de recursos. Si usa un nombre de área de trabajo que ya existe, también en la eliminación temporal del grupo de recursos, recibirá un error que indica que *el nombre del área de trabajo no es único* o entra en *conflicto*. Para invalidar la eliminación temporal y eliminar inmediatamente el área de trabajo y crear una con el mismo nombre, siga estos pasos para recuperar el área de trabajo primero y realizar una eliminación permanente:
   * [Recuperar](../logs/delete-workspace.md#recover-workspace) el área de trabajo
   * [Eliminación permanente del área de trabajo](../logs/delete-workspace.md#permanent-workspace-delete)
   * Crear una nueva área de trabajo con el mismo nombre

## <a name="next-steps"></a>Pasos siguientes
Ahora que tiene un área de trabajo disponible, puede configurar la recopilación de datos de telemetría de supervisión, realizar búsquedas en el registro para analizar dichos datos y agregar una solución de administración que proporcione datos adicionales e información de los análisis.  

* Para habilitar la recopilación de datos de recursos de Azure con Azure Diagnostics o Azure Storage, consulte [Recopilación de registros y métricas de Azure para servicios de Log Analytics](../essentials/resource-logs.md#send-to-log-analytics-workspace).  
* [Agregue System Center Operations Manager como origen de datos](../agents/om-agents.md) para recopilar datos de agentes que informan a su grupo de administración de Operations Manager y almacenarlos en el repositorio del área de trabajo de Log Analytics.  
* Conecte [Configuration Manager](../logs/collect-sccm.md) para importar equipos que son miembros de colecciones en la jerarquía.  
* Revise las [soluciones de supervisión](../insights/solutions.md) disponibles y cómo agregar o quitar una solución del área de trabajo.

