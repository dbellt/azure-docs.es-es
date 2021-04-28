---
title: Uso de un grupo dedicado para ejecutar tareas - Tasks
description: Configure un grupo de proceso dedicado (grupo de agentes) en el registro para ejecutar una tarea de Azure Container Registry.
ms.topic: article
ms.date: 10/12/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: c23d2ab866f621db27488860ab62a41765faef40
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763710"
---
# <a name="run-an-acr-task-on-a-dedicated-agent-pool"></a>Ejecución de una tarea de ACR en un grupo de agentes dedicado

Configure un grupo de VM administradas por Azure (*grupo de agentes*) para habilitar la ejecución de [tareas de Azure Container Registry][acr-tasks] en un entorno de proceso dedicado. Después de configurar uno o varios grupos en el registro, puede elegir un grupo para ejecutar una tarea en lugar del entorno de proceso predeterminado del servicio.

Un grupo de agentes proporciona:

- **Compatibilidad con redes virtuales**: asigne un grupo de agentes a una red virtual de Azure, lo que proporciona acceso a los recursos de la red virtual, como un registro de contenedor, un almacén de claves o almacenamiento.
- **Escale según sea necesario**: aumente el número de instancias de un grupo de agentes para tareas de proceso intensivo o escale a cero. La facturación se basa en la asignación de grupos. Para obtener información detallada, vea [Precios](https://azure.microsoft.com/pricing/details/container-registry/).
- **Opciones flexibles**: elija entre distintos [niveles de grupo](#pool-tiers) y opciones de escalado para satisfacer sus necesidades de carga de trabajo de tareas.
- **Administración de Azure**: los grupos de tareas se revisan y mantienen en Azure, lo que proporciona una asignación reservada sin necesidad de mantener las VM individuales.

Esta característica está disponible en el nivel de servicio de un registro de contenedor **Premium**. Para obtener información sobre los límites y niveles de servicio de registro, consulte [SKU de Azure Container Registry][acr-tiers].

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar y se [aplican algunas limitaciones](#preview-limitations). Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.
>

## <a name="preview-limitations"></a>Limitaciones de vista previa

- Actualmente, los grupos de agentes de tareas admiten nodos de Linux. Los nodos con Windows no se admiten en este momento.
- El grupo de agentes de tareas está disponible en versión preliminar en las siguientes regiones: Oeste de EE. UU. 2, Centro-sur de EE. UU., Este de EE. UU. 2, Este de EE. UU., Centro de EE. UU., Oeste de Europa, Norte de Europa, Centro de Canadá, USGov Arizona, USGov Texas y USGov Virginia.
- Para cada registro, la cuota total predeterminada de vCPU (núcleos) es de 16 para todos los grupos de agentes estándar y de 0 para los grupos de agentes aislados. Abra una [solicitud de soporte técnico][open-support-ticket] para la asignación adicional.
- Actualmente no se puede cancelar la ejecución de una tarea en un grupo de agentes.

## <a name="prerequisites"></a>Requisitos previos

* Para usar los pasos de la CLI de Azure de este artículo, se requieren la versión 2.3.1 de la CLI de Azure o una versión posterior. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli]. O bien ejecute en [Azure Cloud Shell](../cloud-shell/quickstart.md).
* Si aún no tiene un registro de contenedor, [créelo][create-reg-cli] (se requiere un nivel Premium) en una región de la versión preliminar.

## <a name="pool-tiers"></a>Niveles de grupos

Los niveles de grupos de agentes proporcionan los siguientes recursos por instancia en el grupo.

|Nivel    | Tipo  |  CPU  |Memoria (GB)  |
|---------|---------|---------|---------|
|S1     |  estándar    | 2       |    3     |
|S2     |  estándar    | 4       |    8     |
|S3     |  estándar    | 8       |   16     |
|I6     |  aislados    | 64     |   216     |


## <a name="create-and-manage-a-task-agent-pool"></a>Crear y administrar un grupo de agentes de tareas

### <a name="set-default-registry-optional"></a>Definición del registro predeterminado (opcional)

Para simplificar los comandos de la CLI de Azure que siguen, establezca el registro predeterminado ejecutando el comando [az configure][az-configure]:

```azurecli
az configure --defaults acr=<registryName>
```

En los siguientes ejemplos se supone que ha establecido el registro predeterminado. Si no es así, pase un parámetro `--registry <registryName>` en cada comando `az acr`.

### <a name="create-agent-pool"></a>Creación de un grupo de agentes

Cree un grupo de agentes con el comando [az acr agentpool create][az-acr-agentpool-create]. En el ejemplo siguiente se crea una instancia de grupo S2 (4 CPU/instancia). De forma predeterminada, el grupo contiene 1 instancia.

```azurecli
az acr agentpool create \
    --name myagentpool \
    --tier S2
```

> [!NOTE]
> La creación de un grupo de agentes y otras operaciones de administración del grupo tarda varios minutos en completarse.

### <a name="scale-pool"></a>Escalado de grupos

Escale o reduzca verticalmente el tamaño del grupo con el comando [az acr agentpool update][az-acr-agentpool-update]. En el ejemplo siguiente, se escala el grupo a 2 instancias. Puede escalar a 0 instancias.

```azurecli
az acr agentpool update \
    --name myagentpool \
    --count 2
```

## <a name="create-pool-in-a-virtual-network"></a>Creación de un grupo en una red virtual

### <a name="add-firewall-rules"></a>Adición de reglas de firewall

Los grupos de agentes de tareas requieren acceso a los siguientes servicios de Azure. Las siguientes reglas de firewall deben agregarse a los grupos de seguridad de red o rutas definidas por el usuario existentes.

| Dirección | Protocolo | Source         | Puerto de origen | Destination          | Puerto de destino | Utilizado    |
|-----------|----------|----------------|-------------|----------------------|-----------|---------|
| Salida  | TCP      | VirtualNetwork | Any         | AzureKeyVault        | 443       | Valor predeterminado |
| Salida  | TCP      | VirtualNetwork | Any         | Almacenamiento              | 443       | Valor predeterminado |
| Salida  | TCP      | VirtualNetwork | Any         | EventHub             | 443       | Valor predeterminado |
| Salida  | TCP      | VirtualNetwork | Any         | AzureActiveDirectory | 443       | Valor predeterminado |
| Salida  | TCP      | VirtualNetwork | Any         | AzureMonitor         | 443       | Valor predeterminado |

> [!NOTE]
> Si las tareas requieren recursos adicionales de la red pública de Internet, agregue las reglas correspondientes. Por ejemplo, se necesitan reglas adicionales para ejecutar una tarea de compilación de Docker que extraiga las imágenes base de Docker Hub o restaure un paquete de NuGet.

### <a name="create-pool-in-vnet"></a>Crear grupo en VNet

En el ejemplo siguiente se crea un grupo de agentes en la subred *mysubnet* de la red *myvnet*:

```azurecli
# Get the subnet ID
subnetId=$(az network vnet subnet show \
        --resource-grop myresourcegroup \
        --vnet-name myvnet \
        --name mysubnetname \
        --query id --output tsv)

az acr agentpool create \
    --name myagentpool \
    --tier S2 \
    --subnet-id $subnetId
```

## <a name="run-task-on-agent-pool"></a>Ejecutar tarea en el grupo de agentes

En los siguientes ejemplos se muestra cómo especificar un grupo de agentes al poner en cola una tarea.

> [!NOTE]
> Para usar un grupo de agentes en una tarea de ACR, asegúrese de que el grupo contiene al menos una instancia.
>

### <a name="quick-task"></a>Tarea rápida

Ponga en cola una tarea rápida en el grupo de agentes mediante el comando [az acr build][az-acr-build] y pase el parámetro `--agent-pool`:

```azurecli
az acr build \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --file Dockerfile \
    https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
```

### <a name="automatically-triggered-task"></a>Tarea desencadenada automáticamente

Por ejemplo, cree una tarea programada en el grupo de agentes con el comando [az acr task create][az-acr-task-create] y pase el parámetro `--agent-pool`.

```azurecli
az acr task create \
    --name mytask \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --schedule "0 21 * * *" \
    --file Dockerfile \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
    --commit-trigger-enabled false
```

Para comprobar la configuración de la tarea, ejecute [az acr task run][az-acr-task-run]:

```azurecli
az acr task run \
    --name mytask
```

### <a name="query-pool-status"></a>Consulta del estado del grupo

Para averiguar el número de ejecuciones programadas actualmente en el grupo de agentes, ejecute [az acr agentpool show][az-acr-agentpool-show].

```azurecli
az acr agentpool show \
    --name myagentpool \
    --queue-count
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ejemplos de compilaciones y mantenimiento de imágenes de contenedor en la nube, consulte la [serie de tutoriales de ACR Tasks](container-registry-tutorial-quick-task.md).



[acr-tasks]:           container-registry-tasks-overview.md
[acr-tiers]:           container-registry-skus.md
[azure-cli]:           /cli/azure/install-azure-cli
[open-support-ticket]: https://aka.ms/acr/support/create-ticket
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[az-configure]: /cli/azure#az_configure
[az-acr-agentpool-create]: /cli/azure/acr/agentpool#az_acr_agentpool_create
[az-acr-agentpool-update]: /cli/azure/acr/agentpool#az_acr_agentpool_update
[az-acr-agentpool-show]: /cli/azure/acr/agentpool#az_acr_agentpool_show
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[create-reg-cli]: container-registry-get-started-azure-cli.md
