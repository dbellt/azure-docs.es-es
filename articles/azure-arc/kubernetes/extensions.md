---
title: Extensiones de clústeres de Kubernetes habilitados para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
description: Implementación y administración del ciclo de vida de las extensiones en Kubernetes habilitado para Azure Arc
ms.openlocfilehash: 63fb14818d148dcc579300fdb4c89d636b47fd05
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450914"
---
# <a name="kubernetes-cluster-extensions"></a>Extensiones de clúster de Kubernetes

La característica de extensiones de Kubernetes habilita lo siguiente en los clústeres de Kubernetes habilitados para Azure Arc:

* Implementación basada en Azure Resource Manager de la extensión de clúster.
* Administración del ciclo de vida de los gráficos de Helm de la extensión.

Puede encontrar información general y algunos conceptos sobre esta característica en el artículo [Extensiones de clúster: Kubernetes habilitado para Azure Arc](conceptual-extensions.md).

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Instale o actualice la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) a la versión 2.16.0 o posteriores.
- Las extensiones de la CLI de Azure `connectedk8s` (versión 1.1.0 o posteriores) y `k8s-extension` (versión 0.2.0 y superiores). Ejecute los siguientes comandos para instalar estas extensiones de la CLI de Azure:
  
    ```azurecli
    az extension add --name connectedk8s
    az extension add --name k8s-extension
    ```
    
    Si las extensiones `connectedk8s` y `k8s-extension` ya están instaladas, puede actualizarlas a su versión más reciente con el siguiente comando:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-extension
    ```

- Un clúster conectado de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, use nuestra [guía de inicio rápido](quickstart-connect-cluster.md).
    - [Actualice los agentes](agent-upgrade.md#manually-upgrade-agents) a la versión 1.1.0 o posteriores.

## <a name="currently-available-extensions"></a>Extensiones disponibles actualmente

| Extensión | Descripción |
| --------- | ----------- |
| [Azure Monitor](../../azure-monitor/containers/container-insights-enable-arc-enabled-clusters.md?toc=/azure/azure-arc/kubernetes/toc.json) | Proporciona visibilidad sobre el rendimiento de las cargas de trabajo implementadas en el clúster de Kubernetes. Recopila métricas de uso de memoria y CPU de los controladores, nodos y contenedores. |
| [Azure Defender](../../security-center/defender-for-kubernetes-azure-arc.md?toc=/azure/azure-arc/kubernetes/toc.json) | Recopila datos de registro de auditoría de los nodos del plano de control que pertenecen al clúster de Kubernetes. Proporciona recomendaciones y alertas de amenazas basadas en los datos recopilados. |

## <a name="usage-of-cluster-extensions"></a>Uso de las extensiones de clúster

### <a name="create-extensions-instance"></a>Creación de una instancia de extensión

Cree una instancia de la extensión con `k8s-extension create`; para ello, pase los valores de los parámetros obligatorios. El siguiente comando crea una instancia de la extensión de Azure Monitor para contenedores en el clúster de Kubernetes habilitado para Azure Arc:

```azurecli
az k8s-extension create --name azuremonitor-containers  --extension-type Microsoft.AzureMonitor.Containers --scope cluster --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

**Salida:**

```json
{
  "autoUpgradeMinorVersion": true,
  "configurationProtectedSettings": null,
  "configurationSettings": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/defaultresourcegroup-eus/providers/microsoft.operationalinsights/workspaces/defaultworkspace-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-eus"
  },
  "creationTime": "2021-04-02T12:13:06.7534628+00:00",
  "errorInfo": {
    "code": null,
    "message": null
  },
  "extensionType": "microsoft.azuremonitor.containers",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/demo/providers/Microsoft.Kubernetes/connectedClusters/demo/providers/Microsoft.KubernetesConfiguration/extensions/azuremonitor-containers",
  "identity": null,
  "installState": "Pending",
  "lastModifiedTime": "2021-04-02T12:13:06.753463+00:00",
  "lastStatusTime": null,
  "name": "azuremonitor-containers",
  "releaseTrain": "Stable",
  "resourceGroup": "demo",
  "scope": {
    "cluster": {
      "releaseNamespace": "azuremonitor-containers"
    },
    "namespace": null
  },
  "statuses": [],
  "systemData": null,
  "type": "Microsoft.KubernetesConfiguration/extensions",
  "version": "2.8.2"
}
```

> [!NOTE]
> * El servicio no puede conservar información confidencial durante más de 48 horas. Si los agentes de Kubernetes habilitados para Azure Arc no tienen conectividad de red durante más de 48 horas y no pueden determinar si se va a crear una extensión en el clúster, dicha extensión cambiará al estado `Failed`. Una vez en el estado `Failed`, tendrá que ejecutar `k8s-extension create` de nuevo para crear un nuevo recurso de extensión de Azure.
> * * Azure Monitor para contenedores es una extensión singleton (solo se necesita una por clúster). Deberá limpiar todas las instalaciones de gráficos de Helm anteriores de Azure Monitor para contenedores (sin extensiones) antes de instalarlos a través de la extensión. Siga las instrucciones para [eliminar el gráfico Helm antes de ejecutar `az k8s-extension create`](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-optout-hybrid).

**Parámetros obligatorios**

| Nombre de parámetro | Descripción |
|----------------|------------|
| `--name` | Nombre de la instancia de extensión. |
| `--extension-type` | Tipo de extensión que quiere instalar en el clúster. Por ejemplo: Microsoft.AzureMonitor.Containers, microsoft.azuredefender.kubernetes. | 
| `--scope` | Ámbito de instalación de la extensión: `cluster` o `namespace`. |
| `--cluster-name` | Nombre del recurso de Kubernetes habilitado para Azure Arc en el que se debe crear la instancia de extensión. |
| `--resource-group` | Grupo de recursos que contiene el recurso de Kubernetes habilitado para Azure Arc. |
| `--cluster-type` | Tipo de clúster en el que se debe crear la instancia de extensión. Actualmente, el único valor aceptado es `connectedClusters`, que corresponde a Kubernetes habilitado para Azure Arc. |

**Parámetros opcionales**

| Nombre de parámetro | Descripción |
|--------------|------------|
| `--auto-upgrade-minor-version` | Propiedad booleana que especifica si la versión secundaria de la extensión se actualizará automáticamente o no. Predeterminado: `true`.  Si este parámetro se establece en true, no se puede establecer el valor del parámetro`version`, ya que la versión se actualizará dinámicamente. Si se establece en `false`, la extensión no se actualizará automáticamente, ni siquiera para las versiones de revisión. |
| `--version` | Versión de la extensión que se va a instalar (versión específica en la que se fijará la instancia de extensión). No se debe proporcionar si auto-upgrade-minor-version está establecido en `true`. |
| `--configuration-settings` | Configuración que se puede pasar a la extensión para controlar su funcionalidad. Se deben pasar como pares `key=value` separados por espacios después del nombre del parámetro. Si se usa este parámetro, no se podrá emplear `--configuration-settings-file` en el mismo comando. |
| `--configuration-settings-file` | Ruta de acceso al archivo JSON que contiene los pares clave-valor que se usarán para pasar los valores de configuración a la extensión. Si se usa este parámetro, no se podrá emplear `--configuration-settings` en el mismo comando. |
| `--configuration-protected-settings` | Estos valores de configuración no se pueden recuperar mediante llamadas API `GET` o comandos `az k8s-extension show` y, por tanto, se usan para pasar valores de configuración confidenciales. Se deben pasar como pares `key=value` separados por espacios después del nombre del parámetro. Si se usa este parámetro, no se podrá emplear `--configuration-protected-settings-file` en el mismo comando. |
| `--configuration-protected-settings-file` | Ruta de acceso al archivo JSON que contiene los pares clave-valor que se usarán para pasar configuraciones confidenciales a la extensión. Si se usa este parámetro, no se podrá emplear `--configuration-protected-settings` en el mismo comando. |
| `--release-namespace` | Este parámetro indica el espacio de nombres en el que se debe crear la versión. Este parámetro solo es pertinente si el parámetro `scope` se establece en `cluster`. |
| `--release-train` |  Los autores de las extensiones pueden publicar las versiones en series de versiones diferentes, como `Stable`, `Preview`, etc. Si este parámetro no se establece explícitamente, `Stable` se usa como valor predeterminado. Este parámetro no se puede usar cuando el parámetro `autoUpgradeMinorVersion` está establecido en `false`. |
| `--target-namespace` | Este parámetro indica el espacio de nombres en el que se creará la versión. El permiso de la cuenta del sistema creada para esta instancia de extensión está restringido a este espacio de nombres. Este parámetro solo es pertinente si el parámetro `scope` se establece en `namespace`. |

### <a name="show-details-of-an-extension-instance"></a>Visualización de los detalles de una instancia de extensión

Vea los detalles de una instancia de extensión instalada actualmente con `k8s-extension show`; para ello, pase valores para los parámetros obligatorios:

```azurecli
az k8s-extension show --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

**Salida:**

```json
{
  "autoUpgradeMinorVersion": true,
  "configurationProtectedSettings": null,
  "configurationSettings": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/defaultresourcegroup-eus/providers/microsoft.operationalinsights/workspaces/defaultworkspace-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-eus"
  },
  "creationTime": "2021-04-02T12:13:06.7534628+00:00",
  "errorInfo": {
    "code": null,
    "message": null
  },
  "extensionType": "microsoft.azuremonitor.containers",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/demo/providers/Microsoft.Kubernetes/connectedClusters/demo/providers/Microsoft.KubernetesConfiguration/extensions/azuremonitor-containers",
  "identity": null,
  "installState": "Installed",
  "lastModifiedTime": "2021-04-02T12:13:06.753463+00:00",
  "lastStatusTime": "2021-04-02T12:13:49.636+00:00",
  "name": "azuremonitor-containers",
  "releaseTrain": "Stable",
  "resourceGroup": "demo",
  "scope": {
    "cluster": {
      "releaseNamespace": "azuremonitor-containers"
    },
    "namespace": null
  },
  "statuses": [],
  "systemData": null,
  "type": "Microsoft.KubernetesConfiguration/extensions",
  "version": "2.8.2"
}
```

### <a name="list-all-extensions-installed-on-the-cluster"></a>Lista de todas las extensiones instaladas en el clúster

Muestre una lista de todas las extensiones instaladas en un clúster con `k8s-extension list`; para ello, pase los valores de los parámetros obligatorios.

```azurecli
az k8s-extension list --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

**Salida:**

```json
[
  {
    "autoUpgradeMinorVersion": true,
    "creationTime": "2020-09-15T02:26:03.5519523+00:00",
    "errorInfo": {
      "code": null,
      "message": null
    },
    "extensionType": "Microsoft.AzureMonitor.Containers",
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myRg/providers/Microsoft.Kubernetes/connectedClusters/myCluster/providers/Microsoft.KubernetesConfiguration/extensions/myExtInstanceName",
    "identity": null,
    "installState": "Pending",
    "lastModifiedTime": "2020-09-15T02:48:45.6469664+00:00",
    "lastStatusTime": null,
    "name": "myExtInstanceName",
    "releaseTrain": "Stable",
    "resourceGroup": "myRG",
    "scope": {
      "cluster": {
        "releaseNamespace": "myExtInstanceName1"
      }
    },
    "statuses": [],
    "type": "Microsoft.KubernetesConfiguration/extensions",
    "version": "0.1.0"
  },
  {
    "autoUpgradeMinorVersion": true,
    "creationTime": "2020-09-02T00:41:16.8005159+00:00",
    "errorInfo": {
      "code": null,
      "message": null
    },
    "extensionType": "microsoft.azuredefender.kubernetes",
    "id": "/subscriptions/0e849346-4343-582b-95a3-e40e6a648ae1/resourceGroups/myRg/providers/Microsoft.Kubernetes/connectedClusters/myCluster/providers/Microsoft.KubernetesConfiguration/extensions/defender",
    "identity": null,
    "installState": "Pending",
    "lastModifiedTime": "2020-09-02T00:41:16.8005162+00:00",
    "lastStatusTime": null,
    "name": "microsoft.azuredefender.kubernetes",
    "releaseTrain": "Stable",
    "resourceGroup": "myRg",
    "scope": {
      "cluster": {
        "releaseNamespace": "myExtInstanceName2"
      }
    },
    "type": "Microsoft.KubernetesConfiguration/extensions",
    "version": "0.1.0"
  }
]
```

### <a name="update-an-existing-extension-instance"></a>Actualización de una instancia de extensión existente

Actualice una instancia de extensión en un clúster con `k8s-extension update`; para ello, pase los valores que se van a actualizar.  Este comando solo actualiza las propiedades `auto-upgrade-minor-version`, `release-train` y `version`. Por ejemplo:

- **Actualice la serie de versiones:**

    ```azurecli
    az k8s-extension update --name azuremonitor-containers --cluster-type connectedClusters --cluster-name <clusterName> --resource-group <resourceGroupName> --release-train Preview
    ```

- **Desactive la actualización automática y fije la instancia de extensión en una versión específica:**

    ```azurecli
    az k8s-extension update --name azuremonitor-containers --cluster-type connectedClusters --cluster-name <clusterName> --resource-group <resourceGroupName> --auto-upgrade-minor-version false --version 2.2.2
    ```

- **Active la actualización automática para la instancia de extensión:**

    ```azurecli
    az k8s-extension update --name azuremonitor-containers --cluster-type connectedClusters --cluster-name <clusterName> --resource-group <resourceGroupName> --auto-upgrade-minor-version true
    ```

> [!NOTE]
> El parámetro `version` solo se puede establecer cuando `--auto-upgrade-minor-version` está establecido en `false`.

### <a name="delete-extension-instance"></a>Eliminación de una instancia de extensión

Elimine una instancia de extensión en un clúster con `k8s-extension delete`; para ello, pase los valores de los parámetros obligatorios.

```azurecli
az k8s-extension delete --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

>[!NOTE]
> El recurso de Azure que representa esta extensión se elimina inmediatamente. La versión de Helm del clúster asociado a esta extensión solo se eliminará cuando los agentes que se ejecutan en el clúster de Kubernetes tengan conectividad de red y puedan ponerse en contacto con los servicios de Azure de nuevo para recuperar el estado deseado.


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las extensiones de clúster disponibles actualmente para Kubernetes habilitado para Azure Arc:
> [!div class="nextstepaction"]
> [Azure Monitor](../../azure-monitor/containers/container-insights-enable-arc-enabled-clusters.md?toc=/azure/azure-arc/kubernetes/toc.json)
> [Azure Defender](../../security-center/defender-for-kubernetes-azure-arc.md?toc=/azure/azure-arc/kubernetes/toc.json)