---
title: Configuración de Red Hat OpenShift en Azure v4.x con Container insights | Microsoft Docs
description: En este artículo se describe cómo configurar la supervisión de un clúster de Kubernetes con Azure Monitor hospedado en la versión 4 o posterior de Red Hat OpenShift en Azure.
ms.topic: conceptual
ms.date: 03/05/2021
ms.openlocfilehash: 11c702d1f46725a12e90a01dc1b38467344a1123
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784648"
---
# <a name="configure-azure-red-hat-openshift-v4x-with-container-insights"></a>Configuración de Red Hat OpenShift en Azure v4.x con Container insights

Container Insights proporciona una experiencia de supervisión enriquecida para los clústeres de Azure Kubernetes Service (AKS) y del motor de AKS. En este artículo se describe cómo lograr una experiencia de supervisión similar mediante la habilitación de la supervisión de los clústeres de Kubernetes hospedados en la versión 4.x de [Red Hat OpenShift en Azure](../../openshift/intro-openshift.md).

>[!NOTE]
>La compatibilidad con Red Hat OpenShift en Azure es una característica que se encuentra en versión preliminar pública en este momento.
>

Container Insights se puede habilitar para una o más implementaciones existentes de la versión 4.x de Red Hat OpenShift en Azure con los métodos admitidos que se describen en este artículo.

Para un clúster existente, ejecute este [script de Bash en la CLI de Azure](/cli/azure/openshift#az_openshift_create&preserve-view=true).

## <a name="supported-and-unsupported-features"></a>Características admitidas y no admitidas

Container Insights admite la supervisión de la versión 4.x de Red Hat OpenShift en Azure, tal y como se describe en el artículo [Introducción a Container Insights](container-insights-overview.md), excepto para las siguientes características:

- Datos en directo (versión preliminar)
- [Recopilación de métricas](container-insights-update-metrics.md) de nodos y pods del clúster y su almacenamiento en la base de datos de métricas de Azure Monitor

## <a name="prerequisites"></a>Requisitos previos

- La CLI de Azure, versión 2.0.72 o posterior  

- La herramienta de la CLI [Helm 3](https://helm.sh/docs/intro/install/)

- La versión más reciente de la [CLI de OpenShift](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html)

- [Bash versión 4](https://www.gnu.org/software/bash/)

- La herramienta de la línea de comandos [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- Un [área de trabajo de Log Analytics.](../logs/design-logs-deployment.md)

    Container Insights admite un área de trabajo de Log Analytics en las regiones enumeradas en los [productos por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor) de Azure. Para crear el área de trabajo, puede configurarla mediante [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o en el [Azure Portal](../logs/quick-create-workspace.md).

- Para habilitar y acceder a las características de Container Insights debe ser miembro, como mínimo, del rol *Colaborador* de Azure de la suscripción de Azure y miembro del rol [*Colaborador de Log Analytics*](../logs/manage-access.md#manage-access-using-azure-permissions) del área de trabajo de Log Analytics configurada con Container Insights.

- Para ver los datos de supervisión, debe tener el rol [*Lector de Log Analytics*](../logs/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics configurada con Container Insights.

## <a name="enable-monitoring-for-an-existing-cluster"></a>Habilitación de la supervisión para un clúster existente

Realice los siguientes pasos para habilitar la supervisión de un clúster de la versión 4 o posterior de Red Hat OpenShift en Azure implementado en Azure mediante el script de Bash proporcionado:

1. Inicie sesión en Azure ejecutando el siguiente comando:

    ```azurecli
    az login
    ```

1. Descargue y guarde en una carpeta local el script que configura el clúster con el complemento de supervisión mediante la ejecución de los siguientes comandos:

    `curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script`

1. Conéctese al clúster de ARO v4 mediante las instrucciones que se indican en el [Tutorial: Conexión a un clúster de la versión 4 de Red Hat OpenShift en Azure](../../openshift/tutorial-connect-cluster.md).


### <a name="integrate-with-an-existing-workspace"></a>Integración en un área de trabajo existente

En esta sección, se habilita la supervisión del clúster mediante el script de Bash que descargó anteriormente. Para la integración en un área de trabajo de Log Analytics existente, comience por identificar el identificador de recurso completo del área de trabajo de Log Analytics necesario para el parámetro `logAnalyticsWorkspaceResourceId` y, a continuación, ejecute el comando para habilitar el complemento de supervisión en el área de trabajo especificada.

Si no dispone de un área de trabajo para especificar, puede ir a la sección [Integración con el área de trabajo predeterminada](#integrate-with-the-default-workspace) y dejar que el script cree una nueva área de trabajo.

1. Enumere todas las suscripciones a las que tiene acceso mediante la ejecución del siguiente comando:

    ```azurecli
    az account list --all -o table
    ```

    El resultado tendrá un aspecto similar al siguiente:

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
    ```

1. Copie el valor de **SubscriptionId**.

1. Cambie a la suscripción que hospeda el área de trabajo de Log Analytics mediante la ejecución del siguiente comando:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

1. Muestre la lista de áreas de trabajo de sus suscripciones en el formato JSON predeterminado mediante la ejecución del siguiente comando:

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

1. En la salida, busque el nombre del área de trabajo y copie el identificador de recurso completo de esa área de trabajo de Log Analytics en el campo **identificador**.

1. Ejecute el comando siguiente para habilitar la supervisión. Reemplace los valores de los parámetros `azureAroV4ClusterResourceId` y `logAnalyticsWorkspaceResourceId`.

    ```bash
    export azureAroV4ClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.RedHatOpenShift/OpenShiftClusters/<clusterName>"
    export logAnalyticsWorkspaceResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>" 
    ```

    Este es el comando que debe ejecutar una vez que haya rellenado las 3 variables con comandos Export:

    `bash enable-monitoring.sh --resource-id $azureAroV4ClusterResourceId --workspace-id $logAnalyticsWorkspaceResourceId`

Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver las métricas de estado del clúster.

### <a name="integrate-with-the-default-workspace"></a>Integración con el área de trabajo predeterminada

En esta sección se habilita la supervisión del clúster de la versión 4.x de Red Hat OpenShift en Azure mediante el script de Bash que descargó.

En este ejemplo, no es necesario crear previamente ni especificar un área de trabajo existente. Este comando le simplifica el proceso al crear un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción del clúster, si aún no existe en la región.

El área de trabajo predeterminada que se crea tiene el formato *DefaultWorkspace-\<GUID>-\<Region>* .  

Reemplace el valor del parámetro `azureAroV4ClusterResourceId`.

```bash
export azureAroV4ClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.RedHatOpenShift/OpenShiftClusters/<clusterName>"
```

Por ejemplo:

"bash enable-monitoring.sh --resource-id $azureAroV4ClusterResourceId" 

Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

### <a name="enable-monitoring-from-the-azure-portal"></a>Habilitación de la supervisión desde Azure Portal

La vista de varios clústeres de Container Insights resalta los clústeres de Red Hat OpenShift en Azure que no tienen la supervisión habilitada en la pestaña **Clústeres no supervisados**. La opción **Habilitar** situada junto al clúster no inicia la incorporación de la supervisión desde el portal. Se le redirigirá a este artículo para habilitar manualmente la supervisión según los pasos descritos anteriormente en este artículo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el panel izquierdo o desde la página principal, seleccione **Azure Monitor**.

1. En la sección **Información detallada** , seleccione **Contenedores**.

1. En la página **Supervisión - Contenedores**, seleccione **Clústeres sin supervisar**.

1. En la lista de clústeres no supervisados, seleccione el clúster y, a continuación, seleccione **Habilitar**.

    Puede identificar los resultados en la lista buscando el valor **ARO** en la columna **Tipo de clúster**. Después de seleccionar **Habilitar**, se le redirigirá a este artículo.

## <a name="next-steps"></a>Pasos siguientes

- Ahora que ha habilitado la supervisión para recopilar el estado y la utilización de recursos del clúster de la versión 4.x de Red Hat OpenShift y las cargas de trabajo que se ejecutan en estos, obtenga información sobre [cómo usar](container-insights-analyze.md) Container Insights.

- De forma predeterminada, el agente en contenedores recopila los registros de contenedor *stdout* y *stderr* de todos los contenedores que se ejecutan en todos los espacios de nombres excepto kube-system. Para configurar la recopilación de registros de contenedor específica de uno o varios espacios de nombres determinados, consulte [Configuración del agente de Container Insights](container-insights-agent-config.md) para configurar las opciones de recopilación de datos que desee en el archivo de configuraciones *ConfigMap*.

- Para extraer y analizar las métricas de Prometheus desde el clúster, consulte [Configuración de la extracción de métricas de Prometheus](container-insights-prometheus-integration.md).

- Para aprender a detener la supervisión del clúster con Container Insights, consulte [Cómo detener la supervisión del clúster de Red Hat OpenShift en Azure](./container-insights-optout-openshift-v3.md).
