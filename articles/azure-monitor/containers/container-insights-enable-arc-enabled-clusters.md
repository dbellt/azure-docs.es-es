---
title: Supervisión de clústeres de Kubernetes habilitado para Azure Arc
ms.date: 04/05/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
description: Recopile métricas y registros de clústeres de Kubernetes habilitado para Azure Arc con Azure Monitor
ms.openlocfilehash: 55beedec85b5e2a426954f179b738fcf81eb4982
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109845751"
---
# <a name="azure-monitor-container-insights-for-azure-arc-enabled-kubernetes-clusters"></a>Azure Monitor Container Insights para contenedores con clústeres de Kubernetes habilitado para Azure Arc

[Azure Monitor Container Insights](container-insights-overview.md) proporciona una experiencia de supervisión enriquecida para clústeres de Kubernetes habilitado para Azure Arc.

[!INCLUDE [preview features note](../../azure-arc/kubernetes/includes/preview/preview-callout.md)]

## <a name="supported-configurations"></a>Configuraciones admitidas

- Azure Monitor Container Insights admite la supervisión de Kubernetes habilitado para Azure Arc (versión preliminar) tal y como se describe en el artículo [Introducción](container-insights-overview.md), excepto para la característica de datos en directo (versión preliminar). Además, no es necesario que los usuarios tengan permisos de [Propietario](../../role-based-access-control/built-in-roles.md#owner) para [habilitar las métricas](container-insights-update-metrics.md).
- `Docker`, `Moby` y entornos en tiempos de ejecución de contenedor compatibles con CRI, como `CRI-O` y `containerd`.
- Se admiten un proxy de salida sin autenticación y un proxy de salida con autenticación básica. Actualmente no se admite un proxy de salida que espera certificados de confianza.

## <a name="prerequisites"></a>Requisitos previos

- Ha cumplido los requisitos previos incluidos en la [documentación acerca de las extensiones de clúster genéricas](../../azure-arc/kubernetes/extensions.md#prerequisites).
- Un área de trabajo de Log Analytics: Azure Monitor Container Insights admite un área de trabajo de Log Analytics en las regiones que se enumeran en la [página de productos por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor) de Azure. Puede usar [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md) o [Azure Portal](../logs/quick-create-workspace.md) para crear su propia área de trabajo.
- Debe tener una asignación de roles de [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) en la suscripción a Azure que contiene el recurso de Kubernetes habilitado para Azure Arc. Si el área de trabajo Log Analytics está en otra suscripción, se necesita la asignación de roles [Colaborador de Log Analytics](../logs/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics.
- Para ver los datos de supervisión, debe tener la asignación de roles [Lector de Log Analytics](../logs/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics.
- Los siguientes puntos de conexión deben estar habilitados para el acceso de salida, además de los que se mencionan en [Conexión de un clúster de Kubernetes a Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md#meet-network-requirements).

    | Punto de conexión | Port |
    |----------|------|
    | `*.ods.opinsights.azure.com` | 443 |
    | `*.oms.opinsights.azure.com` | 443 |
    | `dc.services.visualstudio.com` | 443 |
    | `*.monitoring.azure.com` | 443 |
    | `login.microsoftonline.com` | 443 |

    Si el recurso de Kubernetes habilitado para Arc está en el entorno de Azure US Government, es necesario habilitar los puntos de conexión siguientes para el acceso de salida:

    | Punto de conexión | Port |
    |----------|------|
    | `*.ods.opinsights.azure.us` | 443 |
    | `*.oms.opinsights.azure.us` | 443 |
    | `dc.services.visualstudio.com` | 443 |
    

- Si anteriormente implementó Azure Monitor Container Insights en este clúster mediante script sin extensiones de clúster, siga las instrucciones que se indican [aquí](container-insights-optout-hybrid.md) para eliminar este gráfico de Helm. Después, puede continuar con la creación de una instancia de la extensión de clúster para Container Insights de Azure Monitor.

    >[!NOTE]
    > La versión basada en script de la implementación de Azure Monitor Container Insights(versión preliminar) se está reemplazando por la forma de implementación de la [extensión de clúster](../../azure-arc/kubernetes/extensions.md). Azure Monitor implementado anteriormente mediante script solo se admite hasta junio de 2021 y, por lo tanto, se recomienda migrar a la forma de implementación de extensión de clúster cuanto antes.

### <a name="identify-workspace-resource-id"></a>Identificación del id. del recurso de área de trabajo

Ejecute los siguientes comandos para buscar el identificador completo de Azure Resource Manager del área de trabajo de Log Analytics. 

1. Enumere todas las suscripciones a las que tiene acceso con el siguiente comando:

    ```azurecli
    az account list --all -o table
    ```

2. Cambie a la suscripción que hospeda el área de trabajo de Log Analytics con el siguiente comando:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. En el ejemplo siguiente se muestra la lista de áreas de trabajo de sus suscripciones en el formato JSON predeterminado.

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    En la salida, busque el nombre del área de trabajo de interés. El campo `id` representa el identificador de Azure Resource Manager de esa área de trabajo de Log Analytics.

    >[!TIP]
    > Este `id` también se puede encontrar en la hoja *Información general* del área de trabajo de Log Analytics a través de Azure Portal.

## <a name="create-extension-instance-using-azure-cli"></a>Creación de la instancia de extensión mediante la CLI de Azure

### <a name="option-1---with-default-values"></a>Opción 1: Con valores predeterminados

Esta opción usa los siguientes valores predeterminados:

- Crea o usa el área de trabajo de Log Analytics predeterminada existente que corresponde a la región del clúster.
- La actualización automática está habilitada para la extensión de clúster de Azure Monitor.

```azurecli
az k8s-extension create --name azuremonitor-containers --cluster-name <cluster-name> --resource-group <resource-group> --cluster-type connectedClusters --extension-type Microsoft.AzureMonitor.Containers
```

### <a name="option-2---with-existing-azure-log-analytics-workspace"></a>Opción 2: Con el área de trabajo existente de Azure Log Analytics

Puede usar un área de trabajo de Azure Log Analytics existente en cualquier suscripción en la que tenga la asignación de roles *Colaborador* o una más permisiva.

```azurecli
az k8s-extension create --name azuremonitor-containers --cluster-name <cluster-name> --resource-group <resource-group> --cluster-type connectedClusters --extension-type Microsoft.AzureMonitor.Containers --configuration-settings logAnalyticsWorkspaceResourceID=<armResourceIdOfExistingWorkspace>
```

### <a name="option-3---with-advanced-configuration"></a>Opción 3: Con configuración avanzada

Si quiere retocar las solicitudes y los límites de recursos predeterminados, puede usar las opciones de configuración avanzada:

```azurecli
az k8s-extension create --name azuremonitor-containers --cluster-name <cluster-name> --resource-group <resource-group> --cluster-type connectedClusters --extension-type Microsoft.AzureMonitor.Containers --configuration-settings  omsagent.resources.daemonset.limits.cpu=150m omsagent.resources.daemonset.limits.memory=600Mi omsagent.resources.deployment.limits.cpu=1 omsagent.resources.deployment.limits.memory=750Mi
```

Consulte la [sección de solicitudes y límites de recursos del gráfico Helm](https://github.com/helm/charts/blob/master/incubator/azuremonitor-containers/values.yaml) para conocer las opciones de configuración disponibles.

### <a name="option-4---on-azure-stack-edge"></a>Opción 4: En Azure Stack Edge

Si el clúster de Kubernetes habilitado para Azure Arc está en Azure Stack Edge, se debe usar una ruta de acceso de montaje personalizada `/home/data/docker`.

```azurecli
az k8s-extension create --name azuremonitor-containers --cluster-name <cluster-name> --resource-group <resource-group> --cluster-type connectedClusters --extension-type Microsoft.AzureMonitor.Containers --configuration-settings omsagent.logsettings.custommountpath=/home/data/docker
```

>[!NOTE]
> Si especifica explícitamente la versión de la extensión que se va a instalar en el comando CREATE, asegúrese de que la versión especificada sea >= 2.8.2.

## <a name="create-extension-instance-using-azure-portal"></a>Creación de una instancia de extensión mediante Azure Portal

>[!IMPORTANT]
>  Si va a implementar Azure Monitor en un clúster de Kubernetes que se ejecuta en Azure Stack Edge, se debe seguir la opción de la CLI de Azure en lugar de la opción de Azure Portal, ya que para estos clústeres se debe establecer la ruta de montaje personalizada.    

### <a name="onboarding-from-the-azure-arc-enabled-kubernetes-resource-blade"></a>Incorporación desde la hoja de recursos de Kubernetes habilitado para Azure Arc

1. En Azure Portal, seleccione el clúster de Kubernetes habilitado para Arc que quiere supervisar.

2. Seleccione el elemento "Insights (versión preliminar)" en la sección "Supervisión" de la hoja de recursos.

3. En la página de incorporación, seleccione el botón "Configurar Azure Monitor".

4. Ahora puede elegir el [área de trabajo de Log Analytics](../logs/quick-create-workspace.md) a la que quiere enviar los datos de métricas y registros.

5. Seleccione el botón "Configurar" para implementar la extensión de clúster de Azure Monitor Container Insights.

### <a name="onboarding-from-azure-monitor-blade"></a>Incorporación desde la hoja Azure Monitor

1. En Azure Portal, vaya a la hoja "Supervisar" y seleccione la opción "Contenedores" en el menú "Insights".

2. Seleccione la pestaña "Unmonitored clusters" (Clústeres no supervisados) para ver los clústeres de Kubernetes habilitado para Azure Arc para los que puede habilitar la supervisión.

3. Haga clic en el vínculo "Habilitar" situado junto al clúster para el que quiere habilitar la supervisión.

4. Elija el área de trabajo de Log Analytics y seleccione el botón "Configurar" para continuar.

## <a name="create-extension-instance-using-azure-resource-manager"></a>Creación de una instancia de extensión mediante Azure Resource Manager

1. Descargue la plantilla de Azure Resource Manager y el archivo de parámetros:

    ```console
    curl -L https://aka.ms/arc-k8s-azmon-extension-arm-template -o arc-k8s-azmon-extension-arm-template.json
    curl -L https://aka.ms/arc-k8s-azmon-extension-arm-template-params -o  arc-k8s-azmon-extension-arm-template-params.json
    ```

2. Actualice los valores de parámetro en el archivo arc-k8s-azmon-extension-arm-template-params.json. En el caso de la nube pública de Azure, debe usarse `opinsights.azure.com` como valor de workspaceDomain.

3. Implementación de la plantilla para crear una extensión de Azure Monitor Container Insights 

    ```azurecli
    az login
    az account set --subscription "Subscription Name"
    az deployment group create --resource-group <resource-group> --template-file ./arc-k8s-azmon-extension-arm-template.json --parameters @./arc-k8s-azmon-extension-arm-template-params.json
    ```

## <a name="delete-extension-instance"></a>Eliminación de una instancia de extensión

El siguiente comando solo elimina la instancia de la extensión, pero no elimina el área de trabajo de Log Analytics. Los datos del recurso de Log Analytics permanecen intactos.

```bash
az k8s-extension delete --name azuremonitor-containers --cluster-type connectedClusters --cluster-name <cluster-name> --resource-group <resource-group>
```

## <a name="disconnected-cluster"></a>Clúster desconectado
Si el clúster está desconectado de Azure durante > 48 horas, Azure Resource Graph no tendrá información sobre el clúster. Como resultado, la hoja Insights puede mostrar información incorrecta sobre el estado del clúster.

## <a name="next-steps"></a>Pasos siguientes

- Con la supervisión habilitada para recopilar el estado y el uso de recursos de su clúster de Kubernetes habilitado para Arc y las cargas de trabajo que se ejecutan en ellos, aprenda [cómo usar](container-insights-analyze.md) Container Insights.

- De forma predeterminada, el agente en contenedores recopila los registros de contenedor stdout y stderr de todos los contenedores que se ejecutan en todos los espacios de nombres excepto kube-system. Para configurar la recopilación de registros de contenedor específica de uno o varios espacios de nombres determinados, consulte [Configuración del agente de Container Insights](container-insights-agent-config.md) para configurar las opciones de recopilación de datos que desee en el archivo de configuraciones ConfigMap.

- Para extraer y analizar las métricas de Prometheus desde el clúster, consulte [Configuración de la extracción de métricas de Prometheus](container-insights-prometheus-integration.md).
