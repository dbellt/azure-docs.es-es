---
title: Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure
description: Obtenga información sobre los comandos comunes para automatizar la administración de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure.
author: TheovanKraay
ms.service: managed-instance-apache-cassandra
ms.topic: how-to
ms.date: 03/15/2021
ms.author: thvankra
ms.openlocfilehash: ee35faf70066ece0f1c799b7d04317a8cd28729d
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517215"
---
# <a name="manage-azure-managed-instance-for-apache-cassandra-resources-using-azure-cli-preview"></a>Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure (versión preliminar)

En este artículo se describen los comandos comunes para automatizar la administración de Azure Managed Instance para clústeres de Apache Cassandra mediante la CLI de Azure.

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

> [!IMPORTANT]
> Este artículo requiere la CLI de Azure 2.17.1 o una versión posterior. Si usa Azure Cloud Shell, la versión más reciente ya está instalada.
>
> La administración de Azure Managed Instance para recursos de Apache Cassandra no se puede cambiar de nombre, ya que esto infringe el funcionamiento de Azure Resource Manager con los URI de recursos.

## <a name="azure-managed-instance-for-apache-cassandra-clusters"></a>Azure Managed Instance para clústeres de Apache Cassandra

En las secciones siguientes se muestra cómo administrar clústeres de Azure Managed Instance for Apache Cassandra:

* [Creación de un clúster de instancia administrada](#create-cluster)
* [Eliminación de un clúster de instancia administrada](#delete-cluster)
* [Obtención de los detalles del clúster](#get-cluster-details)
* [Obtención del estado del nodo de clúster](#get-cluster-status)
* [Listado de clústeres por grupo de recursos](#list-clusters-resource-group)
* [Listado de clústeres por identificador de suscripción](#list-clusters-subscription)

### <a name="create-a-managed-instance-cluster"></a><a id="create-cluster"></a>Creación de un clúster de instancia administrada

Cree un clúster de Azure Managed Instance for Apache Cassandra con el comando [az managed-cassandra cluster create](/cli/azure/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_cluster_create):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
location='West US'
delegatedManagementSubnetId='/subscriptions/<subscription id>/resourceGroups/customer-vnet-rg/providers/Microsoft.Network/virtualNetworks/customer-vnet/subnets/management'
initialCassandraAdminPassword='myPassword'

# You can override the cluster name if the original name is not legal for an Azure resource.
# overrideClusterName='ClusterNameIllegalForAzureResource'
# the default Cassandra version is v3.11

az managed-cassandra cluster create \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName \
    --location $location \
    --delegated-management-subnet-id $delegatedManagementSubnetId \
    --initial-cassandra-admin-password $initialCassandraAdminPassword \
```

### <a name="delete-a-managed-instance-cluster"></a><a id="delete-cluster"></a>Eliminación de un clúster de instancia administrada

Elimine un clúster con el comando [az managed-cassandra cluster delete](/cli/azure/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_cluster_delete):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra cluster delete \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="get-the-cluster-details"></a><a id="get-cluster-details"></a>Obtención de los detalles del clúster

Obtenga los detalles del clúster con el comando [az managed-cassandra cluster show](/cli/azure/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_cluster_show):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra cluster show \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="get-the-cluster-node-status"></a><a id="get-cluster-status"></a>Obtención del estado del nodo de clúster

Obtenga los detalles del clúster con el comando [az managed-cassandra cluster node-status](/cli/azure/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_cluster_node_status):

```azurecli-interactive
clusterName='cassandra-hybrid-cluster'
resourceGroupName='MyResourceGroup'

az managed-cassandra cluster node-status \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="list-the-clusters-by-resource-group"></a><a id="list-clusters-resource-group"></a>Listado de clústeres por grupo de recursos

Enumere los clústeres por grupo de recursos con el comando [az managed-cassandra cluster list](/cli/azure/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_cluster_list):

```azurecli-interactive
subscriptionId='MySubscriptionId'
resourceGroupName='MyResourceGroup'

az managed-cassandra cluster list\
    --resource-group $resourceGroupName
```

### <a name="list-clusters-by-subscription-id"></a><a id="list-clusters-subscription"></a>Listado de clústeres por identificador de suscripción

Enumere los clústeres por id. de suscripción con el comando [az managed-cassandra cluster list](/cli/azure/managed-cassandra?view=azure-cli-latest&preserve-view=true):

```azurecli-interactive
# set your subscription id
az account set -s <subscription id>

az managed-cassandra cluster list
```

## <a name="the-managed-instance-datacenters"></a><a id="managed-instance-datacenter"></a>Centros de datos de instancia administrada

En las secciones siguientes se muestra cómo administrar centro de datos de Azure Managed Instance for Apache Cassandra:

* [Creación de un centro de datos](#create-datacenter)
* [Eliminación de un centro de datos](#delete-datacenter)
* [Obtención de los detalles del centro de datos](#get-datacenter-details)
* [Obtención de centro de datos en un clúster](#get-datacenters-cluster)
* [Actualización o escalabilidad de un centro de datos](#update-datacenter)
* [Actualización de la configuración de Cassandra](#update-yaml)


### <a name="create-a-datacenter"></a><a id="create-datacenter"></a>Creación de un centro de datos

Cree un centro de datos con el comando [az managed-cassandra datacenter create](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_create):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'
dataCenterLocation='eastus2'
delegatedSubnetId='/subscriptions/<Subscription_ID>/resourceGroups/customer-vnet-rg/providers/Microsoft.Network/virtualNetworks/customer-vnet/subnets/dc1-subnet'

az managed-cassandra datacenter create \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName \
    --data-center-location $dataCenterLocation \
    --delegated-subnet-id $delegatedSubnetId \
    --node-count 3 
```

### <a name="delete-a-datacenter"></a><a id="delete-datacenter"></a>Eliminación de un centro de datos

Elimine un centro de datos con el comando [az managed-cassandra datacenter delete](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_delete):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'

az managed-cassandra datacenter delete \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName 
```

### <a name="get-datacenter-details"></a><a id="get-datacenter-details"></a>Obtención de detalles del centro de datos

Obtenga los detalles del centro de datos con el comando [az managed-cassandra datacenter show](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_show):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'

az managed-cassandra datacenter show \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName 
```

### <a name="update-or-scale-a-datacenter"></a><a id="update-datacenter"></a>Actualización o escalabilidad de un centro de datos

Actualice o escale un centro de datos (para escalar cambie el valor de nodeCount) con el comando [az managed-cassandra datacenter update](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_update):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'
dataCenterLocation='eastus'

az managed-cassandra datacenter update \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName \
    --node-count 13 
```

### <a name="update-cassandra-configuration"></a><a id="update-yaml"></a>Actualización de la configuración de Cassandra

Cambie la configuración de Cassandra en un centro de datos mediante el comando [az managed-cassandra datacenter update](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_update). Deberá codificar el fragmento de YAML en base64 mediante una [herramienta en línea](https://www.base64encode.org/). Se admiten los siguientes valores de configuración de YAML:

- column_index_size_in_kb
- compaction_throughput_mb_per_sec
- read_request_timeout_in_ms
- range_request_timeout_in_ms
- aggregated_request_timeout_in_ms
- write_request_timeout_in_ms
- internode_compression
- batchlog_replay_throttle_in_kb

Por ejemplo, el fragmento de YAML siguiente:

```yaml
column_index_size_in_kb: 16
read_request_timeout_in_ms: 10000
```

Cuando se codifica, YAML se convierte en `Y29sdW1uX2luZGV4X3NpemVfaW5fa2I6IDE2CnJlYWRfcmVxdWVzdF90aW1lb3V0X2luX21zOiAxMDAwMA==`. 

Observe a continuación:

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'
dataCenterLocation='eastus'
yamlFragment='Y29sdW1uX2luZGV4X3NpemVfaW5fa2I6IDE2CnJlYWRfcmVxdWVzdF90aW1lb3V0X2luX21zOiAxMDAwMA=='

az managed-cassandra datacenter update \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName \
    --base64-encoded-cassandra-yaml-fragment $yamlFragment
```

### <a name="get-the-datacenters-in-a-cluster"></a><a id="get-datacenters-cluster"></a>Obtención de los centro de datos de un clúster

Obtenga los centros de datos de un clúster con el comando [az managed-cassandra datacenter list](/cli/azure/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#az_managed_cassandra_datacenter_list):

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra datacenter list \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un clúster de instancia administrada desde Azure Portal](create-cluster-portal.md)
* [Implementación de un clúster de Apache Spark administrado con Azure Databricks](deploy-cluster-databricks.md)
