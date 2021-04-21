---
title: Configuración de redes de Azure CNI en Azure Kubernetes Service (AKS)
description: Aprenda a configurar redes de Azure CNI (avanzadas) en Azure Kubernetes Service (AKS), incluida la implementación de un clúster de AKS en una red virtual existente y una subred.
services: container-service
ms.topic: article
ms.date: 06/03/2019
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 839aa012cedaaa6f5bd3d1edad60e3ea7278133b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775896"
---
# <a name="configure-azure-cni-networking-in-azure-kubernetes-service-aks"></a>Configuración de redes de Azure CNI en Azure Kubernetes Service (AKS)

De forma predeterminada, los clústeres de AKS usan [kubenet][kubenet], y una red virtual y una subred se crean automáticamente. Con *kubenet*, los nodos obtienen una dirección IP de una subred de la red virtual. Luego, la traducción de direcciones de red (NAT) se configura en los nodos, y los pods reciben una dirección IP "oculta" detrás de la dirección IP del nodo. Este enfoque reduce el número de direcciones IP que se deben reservar en el espacio de red para su uso por parte de los pods.

Con [Azure Container Networking Interface (CNI)][cni-networking], cada pod obtiene una dirección IP de la subred, y se puede acceder a él directamente. Estas direcciones IP deben ser únicas en el espacio de red y deben planearse de antemano. Cada nodo tiene un parámetro de configuración para el número máximo de pods que admite. Luego, el número equivalente de direcciones IP por nodo se reserva por adelantado para ese nodo. Este enfoque requiere más planificación y a menudo lleva al agotamiento de direcciones IP o a la necesidad de volver a generar los clústeres en una subred mayor, a medida que crecen las exigencias de la aplicación.

En este artículo se muestra cómo usar las redes de *Azure CNI* para crear y usar la subred una red virtual con un clúster de AKS. Para más información sobre las opciones y consideraciones de red, consulte el artículo sobre los [conceptos de red para Kubernetes y AKS][aks-network-concepts].

## <a name="prerequisites"></a>Requisitos previos

* La red virtual del clúster AKS debe permitir la conectividad saliente de Internet.
* Los clústeres de AKS no pueden usar `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16` ni `192.0.2.0/24` para el intervalo de direcciones del servicio de Kubernetes, el intervalo de direcciones de pod, o el intervalo de direcciones de la red virtual del clúster.
* La identidad de clúster que usa el clúster de AKS debe tener como mínimo permisos de [Colaborador de la red](../role-based-access-control/built-in-roles.md#network-contributor) en la subred de la red virtual. Si quiere definir un [rol personalizado](../role-based-access-control/custom-roles.md) en lugar de usar el rol integrado de colaborador de red, se requieren los permisos siguientes:
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`
* La subred asignada al grupo de nodos AKS no puede ser una [subred delegada](../virtual-network/subnet-delegation-overview.md).

## <a name="plan-ip-addressing-for-your-cluster"></a>Planeamiento de direccionamiento IP del clúster

Los clústeres configurados con redes de Azure CNI requieren planeamiento adicional. Los tamaños de red virtual y de subred deben ajustarse al número de pods que tiene previsto ejecutar, así como al número de nodos del clúster.

Se asignan direcciones IP para los pods y los nodos del clúster desde la subred especificada dentro de la red virtual. Cada nodo se configura con una dirección IP principal. De forma predeterminada, Azure CNI configura previamente treinta direcciones IP adicionales que se asignan a los pods programados en el nodo. Cuando se escala horizontalmente el clúster, cada nodo del mismo modo se configura de modo similar con direcciones IP de la subred. También puede ver el [número máximo de pods por nodo](#maximum-pods-per-node).

> [!IMPORTANT]
> El número de direcciones IP necesarias debe incluir consideraciones sobre las operaciones de actualización y escalabilidad. Si establece el intervalo de direcciones IP para que solo se admita un número fijo de nodos, no se podrá actualizar o escalar el clúster.
>
> * Al **actualizar** el clúster de AKS, se implementa un nuevo nodo en el clúster. Los servicios y las cargas de trabajo comienzan a ejecutarse en el nuevo nodo y el nodo antiguo se quita del clúster. Para realizar este proceso de actualización gradual es necesario que haya disponible un bloque adicional de direcciones IP. El recuento de nodos es entonces `n + 1`.
>   * Esta consideración es especialmente importante al utilizar grupos de nodos de Windows Server. Los nodos de Windows Server en AKS no aplican automáticamente las actualizaciones de Windows, en su lugar, se realiza una actualización en el grupo de nodos. Esta actualización implementa los nuevos nodos con las revisiones de seguridad y de la imagen de nodo base de Windows Server 2019 más recientes. Para más información sobre cómo actualizar un grupo de nodos de Windows Server, consulte el artículo de [actualización de un grupo de nodos en AKS][nodepool-upgrade].
>
> * Al **escalar** un clúster de AKS, se implementa un nuevo nodo en el clúster. Los servicios y las cargas de trabajo comienzan a ejecutarse en el nuevo nodo. El intervalo de direcciones IP debe tener en cuenta cómo podría escalar verticalmente el número de nodos y los pods que el clúster puede admitir. También se debe incluir un nodo adicional para las operaciones de actualización. El recuento de nodos es entonces `n + number-of-additional-scaled-nodes-you-anticipate + 1`.

Si espera que los nodos ejecuten el número máximo de pods, y destruye e implementa pods con regularidad, también debe contar con algunas direcciones IP adicionales por nodo. Estas direcciones IP adicionales tienen en cuenta que la eliminación de un servicio y la implementación de la dirección IP liberada para un nuevo servicio pueden tardar unos segundos, y adquieren la dirección.

El plan de direcciones IP de un clúster AKS consta de una red virtual, al menos una subred para los nodos y pods, y un intervalo de direcciones del servicio de Kubernetes.

| Intervalo de direcciones / recurso de Azure | Límites y tamaño |
| --------- | ------------- |
| Virtual network | El tamaño de Azure Virtual Network puede ser de /8, pero se limita a 65 536 direcciones IP configuradas. Antes de configurar el espacio de direcciones, tenga en cuenta todos sus requisitos de red, incluida la comunicación con los servicios de otras redes virtuales. Por ejemplo, si configura un espacio de direcciones demasiado grande, es posible que surjan problemas de superposición con otros espacios de direcciones dentro de la red.|
| Subnet | Debe ser lo suficientemente grande como para dar cabida a los nodos, los pods y a todos los recursos de Kubernetes y de Azure que se pueden aprovisionar en el clúster. Por ejemplo, si implementa una instancia de Azure Load Balancer interna, sus direcciones IP de front-end se asignan desde la subred del clúster, no las direcciones IP públicas. El tamaño de la subred también debe tener en cuenta las operaciones de actualización o las necesidades futuras de escalabilidad.<p />Para calcular el tamaño *mínimo* de subred, incluido un nodo adicional para operaciones de actualización: `(number of nodes + 1) + ((number of nodes + 1) * maximum pods per node that you configure)`<p/>Ejemplo de un clúster de 50 nudos: `(51) + (51  * 30 (default)) = 1,581` (/21 o superior)<p/>Ejemplo de un clúster de 50 nodos que también incluye aprovisionamiento para escalar verticalmente 10 nodos adicionales: `(61) + (61 * 30 (default)) = 1,891` (/21 o superior)<p>Si no especifica un número máximo de pods por nodo al crear el clúster, el número máximo de pods por nodo se establece en *30*. El número mínimo de direcciones IP requeridas se basa en ese valor. Si se calculan los requisitos mínimos de dirección IP según otro valor máximo distinto, vea [cómo configurar el número máximo de pods por nodo](#configure-maximum---new-clusters) para establecer este valor cuando se implementa el clúster. |
| Intervalo de direcciones del servicio de Kubernetes | Este intervalo no lo debe usar ningún elemento de red de esta red virtual o que esté conectado a ella. El CIDR de la dirección del servicio debe ser menor que /12. Puede reutilizar este intervalo en diferentes clústeres de AKS. |
| Dirección IP del servicio DNS de Kubernetes | Dirección IP del intervalo de direcciones del servicio de Kubernetes que se usará en la detección de servicios de clúster. No use la primera dirección IP en el intervalo de direcciones, como .1. La primera dirección del intervalo de la subred se usa para la dirección *kubernetes.default.svc.cluster.local*. |
| Dirección de puente de Docker | La dirección de red del puente de Docker representa la dirección de red del puente de *docker0* predeterminada presente en todas las instalaciones de Docker. Aunque los pods o los clústeres de AKS no usan el puente de *docker0*, debe configurar esta dirección para seguir admitiendo escenarios como la *compilación de Docker* en el clúster de AKS. Es necesario seleccionar un CIDR para la dirección de red del puente de Docker, ya que, de lo contrario, Docker seleccionará automáticamente una subred que podría entrar en conflicto con otros CIDR. Debe elegir un espacio de direcciones que no entre en conflicto con el resto de los CIDR de las redes, incluidos el CIDR de servicio del clúster y el CIDR del pod. El valor predeterminado es 172.17.0.1/16. Puede reutilizar este intervalo en diferentes clústeres de AKS. |

## <a name="maximum-pods-per-node"></a>Pods máximos por nodo

El número máximo de pods por nodo en un clúster de AKS es 250. El número máximo *predeterminado* de pods por nodo varía entre las redes de *kubenet* y de *Azure CNI*, y el método de implementación del clúster.

| Método de implementación | Kubenet predeterminado | Azure CNI predeterminado | Configurable en la implementación |
| -- | :--: | :--: | -- |
| Azure CLI | 110 | 30 | Sí (hasta 250) |
| Plantilla de Resource Manager | 110 | 30 | Sí (hasta 250) |
| Portal | 110 | 110 (configurados en la pestaña Grupos de nodos) | No |

### <a name="configure-maximum---new-clusters"></a>Configurar máximo: nuevos clústeres

Puede configurar el número máximo de pods por nodo solo en tiempo de implementación del clúster o al agregar nuevos grupos de nodos. Si realiza la implementación con la CLI de Azure o con una plantilla de Resource Manager, puede establecer un valor máximo de pods por nodo de hasta 250.

Si no especifica maxPods al crear grupos de nodos nuevos, recibirá un valor predeterminado de 30 para Azure CNI.

Se exige un valor mínimo para los pods máximos por nodo para garantizar el espacio de los pods del sistema críticos para el estado del clúster. El valor mínimo que se puede establecer para los pods máximos por nodo es diez si y solo si la configuración de cada grupo de nodos tiene espacio para un mínimo de treinta pods. Por ejemplo, si se establece el número máximo de pods por nodo en el mínimo de diez, cada grupo de nodos individual debe tener un mínimo de tres nodos. Este requisito se aplica también a cada nuevo grupo de nodos que se crea, por lo que si se define diez como el número máximo de pods por nodo, cada grupo de nodos que se agregue después debe tener al menos tres nodos.

| Redes | Mínima | Máxima |
| -- | :--: | :--: |
| CNI de Azure | 10 | 250 |
| Kubenet | 10 | 110 |

> [!NOTE]
> El valor mínimo de la tabla anterior lo aplica estrictamente el servicio AKS. No se puede establecer un valor de maxPods inferior al mínimo que se muestra; de lo contrario, podría impedirse el inicio del clúster.

* **CLI de Azure**: especifique el argumento `--max-pods` cuando implemente un clúster con el comando [az aks create][az-aks-create]. El valor máximo es 250.
* **Plantilla de Resource Manager**: especifique la propiedad `maxPods` del objeto [ManagedClusterAgentPoolProfile] cuando implemente un clúster con una plantilla de Resource Manager. El valor máximo es 250.
* **Portal de Azure**: no puede modificar el número máximo de pods por nodo cuando implemente un clúster con Azure Portal. Los clústeres de redes de Azure CNI están limitados a 30 pods por nodo cuando se implementan mediante Azure Portal.

### <a name="configure-maximum---existing-clusters"></a>Configurar máximo: clústeres existentes

La configuración de maxPod por nodo se puede definir cuando se crea un nuevo grupo de nodos. Si necesita aumentar el valor de maxPod por nodo en un clúster existente, agregue un nuevo grupo de nodos con el nuevo recuento de maxPod deseado. Después de migrar los pods al nuevo grupo, elimine el grupo anterior. Para eliminar cualquier grupo anterior en un clúster, asegúrese de que está configurando los modos de grupo de nodos tal como se define en el [documento de grupos de nodos del sistema][system-node-pools].

## <a name="deployment-parameters"></a>Parámetros de implementación

Cuando crea un clúster de AKS, los parámetros siguientes son configurables para redes de Azure CNI:

**Red virtual**: la red virtual en la que desea implementar el clúster de Kubernetes. Si desea crear una red virtual nueva para el clúster, seleccione *Crear nueva* y siga los pasos descritos en la sección *Creación de red virtual*. Para información acerca de límites y cuotas para Azure Virtual Network, vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).

**Subred**: la subred dentro de la red virtual en la que desea implementar el clúster. Si desea crear una nueva subred en la red virtual para el clúster, seleccione *Crear nueva* y siga los pasos descritos en la sección *Creación de subred*. Para la conectividad híbrida, el intervalo de direcciones no debe solaparse con ninguna otra red virtual de su entorno.

**Complemento de red de Azure**: cuando se usa el complemento de red de Azure, no se puede acceder al servicio Load Balancer interno con "externalTrafficPolicy=Local" desde VM con una dirección IP en clusterCIDR que no pertenezca al clúster de AKS.

**Intervalo de direcciones de servicio de Kubernetes**: es parámetro es el conjunto de direcciones IP virtuales que Kubernetes asigna a [servicios][services] internos del clúster. Puede usar cualquier intervalo de direcciones privado que cumpla los requisitos siguientes:

* No debe estar dentro del intervalo de direcciones IP de la red virtual del clúster.
* No deben superponerse con ninguna otra red virtual del mismo nivel que la red virtual del clúster.
* No debe superponerse con ninguna dirección IP local.
* No debe estar dentro de los intervalos `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16` ni `192.0.2.0/24`

Aunque es técnicamente posible especificar un intervalo de direcciones de servicio en la misma red virtual que el clúster, no se recomienda. Puede producirse un comportamiento impredecible si se usan intervalos IP superpuestos. Para más información, consulte la sección [P+F](#frequently-asked-questions) de este artículo. Para más información sobre los servicios de Kubernetes, consulte [Servicios][services] en la documentación de Kubernetes.

**Dirección IP del servicio DNS de Kubernetes**:  la dirección IP del servicio DNS del clúster. Esta dirección debe estar dentro del  *intervalo de direcciones del servicio Kubernetes*. No use la primera dirección IP en el intervalo de direcciones, como .1. La primera dirección del intervalo de la subred se usa para la dirección *kubernetes.default.svc.cluster.local*.

**Dirección del puente de Docker**: La dirección de red del puente de Docker representa la dirección de red del puente de *docker0* predeterminada presente en todas las instalaciones de Docker. Aunque los pods o los clústeres de AKS no usan el puente de *docker0*, debe configurar esta dirección para seguir admitiendo escenarios como la *compilación de Docker* en el clúster de AKS. Es necesario seleccionar un CIDR para la dirección de red del puente de Docker, ya que, de lo contrario, Docker seleccionará automáticamente una subred que podría entrar en conflicto con otros CIDR. Debe elegir un espacio de direcciones que no entre en conflicto con el resto de los CIDR de las redes, incluidos el CIDR de servicio del clúster y el CIDR del pod.

## <a name="configure-networking---cli"></a>Configuración de redes: CLI

Al crear un clúster de AKS con la CLI de Azure, también puede configurar redes de Azure CNI. Use los siguientes comandos para crear un nuevo clúster de AKS con las redes de Azure CNI habilitadas.

En primer lugar, obtenga el identificador de recursos de subred para la subred existente a la que se unirá el clúster de AKS:

```azurecli-interactive
$ az network vnet subnet list \
    --resource-group myVnet \
    --vnet-name myVnet \
    --query "[0].id" --output tsv

/subscriptions/<guid>/resourceGroups/myVnet/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default
```

Use el comando [az aks create][az-aks-create] con el argumento `--network-plugin azure` para crear un clúster con redes avanzadas. Actualice el valor `--vnet-subnet-id` con el identificador de subred recopilado en el paso anterior:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 \
    --generate-ssh-keys
```

## <a name="configure-networking---portal"></a>Configuración de redes: portal

La siguiente captura de pantalla de Azure Portal muestra un ejemplo de la configuración de estos valores durante la creación del clúster de AKS:

![Configuración avanzada de redes en Azure Portal][portal-01-networking-advanced]

## <a name="dynamic-allocation-of-ips-and-enhanced-subnet-support-preview"></a>Asignación dinámica de direcciones IP y compatibilidad mejorada con subredes (versión preliminar)

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

> [!NOTE] 
> La característica en vista previa (GB) está disponible actualmente en las siguientes regiones:
>
> * Centro-Oeste de EE. UU.

Un inconveniente de la CNI tradicional es el agotamiento de las direcciones IP Pod a medida que crece el clúster de AKS, de modo que se vuelve necesario recompilar todo el clúster en una subred más grande. La nueva funcionalidad de asignación dinámica de direcciones IP de Azure CNI soluciona este problema al asignar direcciones IP de pod de una subred independiente de la subred que hospeda el clúster de AKS.  Esto reporta las siguientes ventajas:

* **Mejor uso de IP**: las direcciones IP se asignan dinámicamente a los pods de clúster desde la subred de pod. Esto conduce a un mejor uso de las direcciones IP en el clúster en comparación con la solución CNI tradicional, que realiza una asignación estática de direcciones IP para cada nodo.  

* **Escalable y flexible**: las subredes de nodo y pod se pueden escalar de manera independiente. Una sola subred de pod puede compartirse en varios grupos de nodos de un clúster o en varios clústeres de AKS implementados en la misma red virtual. También puede configurar una subred de pod independiente para un grupo de nodos.  

* **Alto rendimiento**: ya que a los pods se les asignan direcciones IP de red virtual, tienen conectividad directa con los recursos y el pod de otros clústeres de la red virtual. La solución admite clústeres muy grandes sin ninguna reducción del rendimiento.

* **Directivas de red virtual independientes para pods**: dado que los pods tienen una subred independiente, puede configurar directivas de red virtual independientes para ellas distintas de las directivas de nodo. Esto permite muchos escenarios útiles, como la habilitación de la conectividad a Internet solo para pods y no para los nodos, la corrección de la IP de origen para pods en un grupo de nodos mediante una NAT de red virtual y el uso de NSG para filtrar el tráfico entre grupos de nodos.  

* **Directivas de red de Kubernetes**: las directivas de red de Azure y Calico funcionan con esta nueva solución.  

### <a name="install-the-aks-preview-azure-cli"></a>Instalación de la CLI de Azure `aks-preview`

Necesitará la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="register-the-podsubnetpreview-preview-feature"></a>Registro de la característica en vista previa (GB) `PodSubnetPreview`

Para usar la característica, también debe habilitar la marca de característica `PodSubnetPreview` en la suscripción.

Registre la marca de la característica `PodSubnetPreview` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "PodSubnetPreview"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/PodSubnetPreview')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

### <a name="additional-prerequisites"></a>Requisitos previos adicionales

Los requisitos previos ya indicados para Azure CNI siguen siendo aplicables, pero hay algunas limitaciones adicionales:

* Solo se admiten clústeres de nodo de Linux y los grupos de nodos.
* No se admiten los clústeres de AKS Engine y de implementación personal.

### <a name="planning-ip-addressing"></a>Planeación del direccionamiento IP

Al usar esta característica, la planeación es mucho más sencilla. Dado que los nodos y pods se escalan de manera independiente, los espacios de direcciones también se pueden planear por separado. Como las subredes de pod se pueden configurar en la granularidad de un grupo de nodos, los clientes siempre podrán agregar una nueva subred cuando agreguen un grupo de nodos. Los pods de sistema de un grupo de clústeres/nodos también reciben direcciones IP de la subred de pod, por lo que es necesario tener en cuenta este comportamiento.

La planeación de las direcciones IP para los servicios K8S y el puente de Docker permanecen sin cambios.

### <a name="maximum-pods-per-node-in-a-cluster-with-dynamic-allocation-of-ips-and-enhanced-subnet-support"></a>Máximo de pods por nodo en un clúster con asignación dinámica de direcciones IP y compatibilidad mejorada con subredes

Los valores de pods por nodo al usar Azure CNI con asignación dinámica de direcciones IP han cambiado ligeramente del comportamiento de CNI tradicional:

|CNI|Método de implementación|Valor predeterminado|Configurable en la implementación|
|--|--| :--: |--|
|Azure CNI tradicional|Azure CLI|30|Sí (hasta 250)|
|Azure CNI con asignación dinámica de direcciones IP|Azure CLI|250|Sí (hasta 250)|

Todas las demás instrucciones relacionadas con la configuración del máximo de nodos por pod siguen siendo las mismas.

### <a name="additional-deployment-parameters"></a>Parámetros de implementación adicionales

Los parámetros de implementación descritos anteriormente siguen siendo válidos, con una excepción:

* El parámetro **subnet** ahora hace referencia a la subred relacionada con los nodos del clúster.
* Se usa un parámetro adicional **pod subnet** para especificar la subred cuyas direcciones IP se asignarán dinámicamente a los pods.

### <a name="configure-networking---cli-with-dynamic-allocation-of-ips-and-enhanced-subnet-support"></a>Configuración de redes: CLI con asignación dinámica de direcciones IP y compatibilidad mejorada con subredes

El uso de la asignación dinámica de direcciones IP y la compatibilidad mejorada con subredes en el clúster es similar al método predeterminado para configurar un clúster de Azure CNI. En el ejemplo siguiente se describe cómo crear una nueva red virtual con una subred para nodos y una subred para pods, y cómo crear un clúster que usa Azure CNI con asignación dinámica de direcciones IP y compatibilidad mejorada con subredes. Asegúrese de reemplazar las variables como `$subscription` con sus propios valores:

En primer lugar, cree la red virtual con dos subredes:

```azurecli-interactive
$resourceGroup="myResourceGroup"
$vnet="myVirtualNetwork"

# Create our two subnet network 
az network vnet create -g $rg --name $vnet --address-prefixes 10.0.0.0/8 -o none 
az network vnet subnet create -g $rg --vnet-name $vnet --name nodesubnet --address-prefixes 10.240.0.0/16 -o none 
az network vnet subnet create -g $rg --vnet-name $vnet --name podsubnet --address-prefixes 10.241.0.0/16 -o none 
```

A continuación, cree el clúster. Para ello, haga referencia a la subred de nodo mediante `--vnet-subnet-id` y la subred de pod con `--pod-subnet-id`:

```azurecli-interactive
$clusterName="myAKSCluster"
$location="eastus"
$subscription="aaaaaaa-aaaaa-aaaaaa-aaaa"

az aks create -n $clusterName -g $resourceGroup -l $location --max-pods 250 --node-count 2 --network-plugin azure --vnet-subnet-id /subscriptions/$subscription/resourceGroups/$resourceGroup/providers/Microsoft.Network/virtualNetworks/$vnet/subnets/nodesubnet --pod-subnet-id /subscriptions/$subscription/resourceGroups/$resourceGroup/providers/Microsoft.Network/virtualNetworks/$vnet/subnets/podsubnet  
```

#### <a name="adding-node-pool"></a>Adición de grupo de nodos

Al agregar un grupo de nodos, haga referencia a la subred de nodo mediante `--vnet-subnet-id` y la subred de pod mediante `--pod-subnet-id`. En el ejemplo siguiente se crean dos nuevas subredes a las que se hace referencia durante la creación de un nuevo grupo de nodos:

```azurecli-interactive
az network vnet subnet create -g $resourceGroup --vnet-name $vnet --name node2subnet --address-prefixes 10.242.0.0/16 -o none 
az network vnet subnet create -g $resourceGroup --vnet-name $vnet --name pod2subnet --address-prefixes 10.243.0.0/16 -o none 

az aks nodepool add --cluster-name $clusterName -g $resourceGroup  -n newNodepool --max-pods 250 --node-count 2 --vnet-subnet-id /subscriptions/$subscription/resourceGroups/$resourceGroup/providers/Microsoft.Network/virtualNetworks/$vnet/subnets/node2subnet  --pod-subnet-id /subscriptions/$subscription/resourceGroups/$resourceGroup/providers/Microsoft.Network/virtualNetworks/$vnet/subnets/pod2subnet --no-wait 
```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

Las siguientes preguntas y respuestas se aplican a la configuración de red de **Azure CNI**.

* *¿Puedo implementar máquinas virtuales en la subred del clúster?*

  Sí.

* *¿Qué dirección IP de origen ven los sistemas externos para el tráfico que se origina en un pod habilitado para Azure CNI?*

  Los sistemas de la misma red virtual que el clúster de AKS ven la IP del pod como la dirección de origen para todo el tráfico del pod. Los sistemas que están fuera de la red virtual del clúster de AKS ven la IP del nodo como la dirección de origen para todo el tráfico del pod.

* *¿Puedo configurar las directivas de red por pod?*

  Sí, la directiva de red de Kubernetes está disponible en AKS. Para comenzar, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][network-policy].

* *¿Es configurable el número máximo de pods que se puede implementar en un nodo ?*

  Sí, al implementar un clúster con la CLI de Azure o una plantilla de Resource Manager. Consulte [Pods máximos por nodo](#maximum-pods-per-node).

  No se puede cambiar el número máximo de pods por nodo en un clúster existente.

* *¿Cómo se pueden configurar propiedades adicionales para la subred que he creado durante la creación del clúster de AKS? Por ejemplo, los puntos de conexión de servicio.*

  La lista completa de propiedades de la red virtual y las subredes que se crean durante la creación del clúster de AKS puede configurarse en la página de configuración de red virtual estándar en Azure Portal.

* *¿Puedo usar otra subred dentro de mi red virtual de clúster como* **intervalo de direcciones de servicio de Kubernetes**?

  Aunque no se recomienda, esta configuración es posible. El rango de direcciones de servicio es un conjunto de direcciones IP virtuales (VIP) que Kubernetes asigna a los servicios internos del clúster. Las redes de Azure no tiene ninguna visibilidad sobre el intervalo de direcciones IP de servicio del clúster de Kubernetes. Debido a ello, es posible crear posteriormente una nueva subred en la red virtual del clúster que se superponga con el intervalo de direcciones de servicio. Si se produce una superposición de este tipo, Kubernetes podría asignar a un servicio una dirección IP que ya esté en uso por otro recurso de la subred, lo que provocaría un comportamiento impredecible o errores. Al tener la seguridad de usar un intervalo de direcciones que se encuentra fuera de la red virtual del clúster puede evitar este riesgo de superposición.

### <a name="dynamic-allocation-of-ip-addresses-and-enhanced-subnet-support-faqs"></a>Preguntas frecuentes sobre la asignación dinámica de direcciones IP y la compatibilidad mejorada con subredes

Las siguientes preguntas y respuestas se aplican a la **configuración de red de Azure CNI cuando se usa la asignación dinámica de direcciones IP y la compatibilidad mejorada con subredes**.

* *¿Puedo asignar varias subredes de pod a un grupo de clústeres/nodos?*

  Solo se puede asignar una subred a un grupo de clústeres o nodos. Sin embargo, varios clústeres o grupos de nodos pueden compartir una sola subred.

* *¿Puedo asignar subredes de pod de una red virtual diferente?*

  La subred de pod debe ser de la misma red virtual que el clúster.  

* *¿Algunos de los grupos de nodos de un clúster pueden usar el CNI tradicional mientras otros usan el nuevo CNI?*

  Todo el clúster debe usar un solo tipo de CNI.

## <a name="aks-engine"></a>Motor de AKS

El [motor de Azure Kubernetes Service (motor de AKS)][aks-engine] es un proyecto de código abierto que genera plantillas de Azure Resource Manager que puede usar para implementar clústeres de Kubernetes en Azure.

Los clústeres de Kubernetes creados con AKS Engine admiten los complementos [kubenet][kubenet] y [Azure CNI][cni-networking]. Por lo tanto, ambos escenarios de redes son compatibles con AKS Engine.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de las redes en AKS en los siguientes artículos:

* [Uso de una dirección IP estática con el equilibrador de carga de Azure Kubernetes Service (AKS)](static-ip.md)
* [Uso de un equilibrador de carga interno con Azure Container Service (AKS)](internal-lb.md)

* [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
* [Habilitación del complemento de enrutamiento de aplicación HTTP][aks-http-app-routing]
* [Creación de un controlador de entrada que use una red privada interna y una dirección IP][aks-ingress-internal]
* [Creación de un controlador de entrada con una dirección IP pública dinámica y configuración de Let's Encrypt para generar certificados TLS de forma automática][aks-ingress-tls]
* [Creación de un controlador de entrada con una dirección IP pública estática y configuración de Let's Encrypt para generar certificados TLS de forma automática][aks-ingress-static-tls]
<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png [portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[services]: https://kubernetes.io/docs/concepts/services-networking/service/
[portal]: https://portal.azure.com
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[az-aks-create]: /cli/azure/aks#az_aks_create
[aks-ssh]: ssh.md
[ManagedClusterAgentPoolProfile]: /azure/templates/microsoft.containerservice/managedclusters#managedclusteragentpoolprofile-object
[aks-network-concepts]: concepts-network.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-internal]: ingress-internal-ip.md
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[network-policy]: use-network-policies.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[network-comparisons]: concepts-network.md#compare-network-models
[system-node-pools]: use-system-pools.md
