---
title: Uso de las zonas de disponibilidad en Azure Kubernetes Service (AKS)
description: Aprenda a crear un clúster que distribuya nodos a través de las zonas de disponibilidad en Azure Kubernetes Service (AKS)
services: container-service
ms.custom: fasttrack-edit, references_regions, devx-track-azurecli
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: 13a14854f373ca7297e454ddbdc9f475849dc0b8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110100547"
---
# <a name="create-an-azure-kubernetes-service-aks-cluster-that-uses-availability-zones"></a>Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad

Un clúster de Azure Kubernetes Service (AKS) distribuye recursos como nodos y almacenamiento en secciones lógicas de infraestructura subyacente de Azure. Este modelo de implementación con zonas de disponibilidad garantiza que los nodos de una zona de disponibilidad determinada estén físicamente separados de los definidos en otra zona de disponibilidad. Los clústeres de AKS implementados con varias zonas de compatibilidad configuradas en un clúster proporcionan un alto nivel de disponibilidad para proteger frente a errores de hardware o eventos de mantenimiento planeados.

Si se definen los grupos de nodos de un clúster de modo que abarquen varias zonas, los nodos de un grupo de nodos determinado pueden seguir funcionando aunque una zona esté fuera de servicio. Las aplicaciones pueden seguir estando disponibles aunque se produzca un error físico en un único centro de datos, siempre que este esté orquestado para tolerar errores de un subconjunto de nodos.

En este artículo le mostraremos cómo crear un clúster de AKS y distribuir los componentes del nodo en las zonas de disponibilidad.

## <a name="before-you-begin"></a>Antes de empezar

Es preciso que esté instalada y configurada la versión 2.0.76 de la CLI de Azure, o cualquier otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="limitations-and-region-availability"></a>Limitaciones y disponibilidad de región

Los clústeres de AKS se pueden crear actualmente mediante zonas de disponibilidad en las siguientes regiones:

* Este de Australia
* Sur de Brasil
* Centro de Canadá
* Centro de EE. UU.
* Este de EE. UU. 
* Este de EE. UU. 2
* Centro de Francia
* Centro-oeste de Alemania
* Japón Oriental
* Norte de Europa
* Sudeste de Asia
* Centro-sur de EE. UU.
* Sur de Reino Unido 2
* US Gov - Virginia
* Oeste de Europa
* Oeste de EE. UU. 2

Las siguientes limitaciones se aplican cuando crea un clúster de AKS mediante zonas de disponibilidad:

* Solo se pueden definir zonas de disponibilidad cuando se crea el clúster o grupo de nodos.
* La configuración de la zona de disponibilidad no se puede actualizar después de crear el clúster. Tampoco puede actualizar un clúster de zona sin disponibilidad para usar zonas de disponibilidad.
* El tamaño de nodo (SKU de VM) seleccionado debe estar disponible en todas las zonas de disponibilidad seleccionadas.
* Los clústeres con zonas de disponibilidad habilitadas requieren el uso de equilibradores de carga estándar de Azure para la distribución entre zonas. Este tipo de equilibrador de carga solo se puede definir en el momento de la creación del clúster. Para obtener más información y las limitaciones del equilibrador de carga estándar, consulte las [limitaciones de la SKU estándar del equilibrador de carga de Azure][standard-lb-limitations].

### <a name="azure-disks-limitations"></a>Limitaciones de los discos de Azure

Los volúmenes que usan discos administrados de Azure actualmente no son recursos con redundancia de zona. Los volúmenes no se pueden conectar entre zonas y deben estar ubicados en la misma zona que el nodo concreto que hospeda al pod de destino.

Kubernetes tiene en cuenta las zonas de disponibilidad de Azure desde la versión 1.12. Puede implementar un objeto PersistentVolumeClaim que haga referencia a un disco administrado de Azure en un clúster de AKS de varias zonas y [Kubernetes se encargará de programar](https://kubernetes.io/docs/setup/best-practices/multiple-zones/#storage-access-for-zones) cualquier pod que reclame este objeto PVC en la zona de disponibilidad correcta.

### <a name="azure-resource-manager-templates-and-availability-zones"></a>Plantillas de Azure Resource Manager y zonas de disponibilidad

Al *crear* un clúster de AKS, si define explícitamente un [valor NULL en una plantilla][arm-template-null] con una sintaxis de tipo `"availabilityZones": null`, la plantilla de Resource Manager trata la propiedad como si no existiera, lo que significa que el clúster no tendrá habilitadas las zonas de disponibilidad. Además, si crea un clúster con una plantilla de Resource Manager que omite la propiedad availability zones, las zonas de disponibilidad se deshabilitan.

No se puede actualizar la configuración de zonas de disponibilidad en un clúster existente, por lo que el comportamiento es diferente al actualizar un clúster de AKS con plantillas de Resource Manager.  Si establece explícitamente un valor NULL en la plantilla para las zonas de disponibilidad y *actualiza* el clúster, no se realizan cambios en el clúster para las zonas de disponibilidad. Sin embargo, si omite la propiedad availability zones con una sintaxis de tipo `"availabilityZones": []`, la implementación intenta deshabilitar las zonas de disponibilidad en el clúster de AKS existente y **produce un error**.

## <a name="overview-of-availability-zones-for-aks-clusters"></a>Introducción a las zonas de disponibilidad para los clústeres de AKS

Las zonas de disponibilidad son una oferta de alta disponibilidad que protege las aplicaciones y datos de los errores del centro de datos. Las zonas son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, siempre hay más de una zona en todas las regiones habilitadas para zonas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos.

Para más información, consulte [¿Qué son las zonas de disponibilidad en Azure?][az-overview]

Los clústeres de AKS que se implementan mediante zonas de disponibilidad que pueden distribuir nodos en varias zonas dentro de una sola región. Por ejemplo, un clúster en la región  *Este de EE. UU. 2* puede crear nodos en las tres zonas de disponibilidad del *Este de EE. UU. 2*. Esta distribución de los recursos del clúster de AKS mejora la disponibilidad del clúster, ya que son resistentes a los errores de una zona específica.

![Distribución de nodos de AKS en zonas de disponibilidad](media/availability-zones/aks-availability-zones.png)

Si una sola zona deja de estar disponible, las aplicaciones siguen ejecutándose siempre que el clúster esté distribuido entre varias zonas.

## <a name="create-an-aks-cluster-across-availability-zones"></a>Crear un clúster de AKS en zonas de disponibilidad

Cuando crea un clúster con el comando [az aks create][az-aks-create], el parámetro `--zones` define en qué zonas se implementan los nodos de agente. Los componentes del plano de control, como etcd o la API, se reparten entre las zonas disponibles en la región si define el parámetro `--zones` en el momento de la creación del clúster. Las zonas específicas entre las que se han distribuido los componentes del plano de control son independientes de las zonas explícitas que se han seleccionado para el grupo de nodos inicial.

Si no se define ninguna zona para el grupo de agentes predeterminado al crear un clúster de AKS, no se garantiza que los componentes del plano de control se distribuyan entre zonas de disponibilidad. Puede agregar grupos de nodos adicionales mediante el comando [az aks nodepool add][az-aks-nodepool-add] y especificar `--zones` para los nuevos nodos, pero esto no cambia la forma en que se ha distribuido el plano de control entre zonas. La configuración de zonas de disponibilidad solo se puede definir en el momento de la creación del clúster o el grupo de nodos.

En el ejemplo siguiente se crea un clúster de AKS denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*. Se crean un total de *3* nodos: un agente en la zona *1*, uno en la *2* y otro en la *3*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus2

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --node-count 3 \
    --zones 1 2 3
```

El clúster de AKS tarda unos minutos en crearse.

A la hora de decidir la zona a la que debe pertenecer un nodo nuevo, un grupo de nodos determinado de AKS usa la [mejor opción de equilibrio de zonas ofrecida por la instancia subyacente de Azure Virtual Machine Scale Sets][vmss-zone-balancing]. Un grupo de nodos dado de AKS se considera "equilibrado" si cada zona tiene el mismo número de máquinas virtuales o +\- 1 máquina virtual en todas las demás zonas del conjunto de escalado.

## <a name="verify-node-distribution-across-zones"></a>Comprobar la distribución de nodos entre zonas

Cuando el clúster esté listo, enumere los nodos de agente en el conjunto de escalado para ver en qué zona de disponibilidad se implementarán.

Primero, obtenga las credenciales del clúster de AKS mediante el comando [az aks get-credentials][az-aks-get-credentials]:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

A continuación, use el comando [kubectl describe][kubectl-describe] para enumerar los nodos del clúster y filtrar por el valor de *failure-domain.beta.kubernetes.io/zone*. El ejemplo siguiente es para un shell de Bash.

```console
kubectl describe nodes | grep -e "Name:&quot; -e &quot;failure-domain.beta.kubernetes.io/zone"
```

El siguiente resultado de ejemplo muestra los tres nodos distribuidos en la región especificada y las zonas de disponibilidad, como *eastus2-1* para la primera zona de disponibilidad y *eastus2-2* para la segunda zona de disponibilidad:

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
```

A medida que agrega nodos adicionales a un grupo de agentes, la plataforma Azure distribuye automáticamente las VM subyacentes en las zonas de disponibilidad especificadas.

Tenga en cuenta que en las versiones más recientes de Kubernetes (1.17.0 y versiones posteriores), AKS usa la etiqueta `topology.kubernetes.io/zone` más reciente además de la `failure-domain.beta.kubernetes.io/zone` en desuso. Puede obtener el mismo resultado que antes ejecutando el siguiente script:

```console
kubectl get nodes -o custom-columns=NAME:'{.metadata.name}',REGION:'{.metadata.labels.topology\.kubernetes\.io/region}',ZONE:'{metadata.labels.topology\.kubernetes\.io/zone}'
```

Que le proporcionará una salida más concisa:

```console
NAME                                REGION   ZONE
aks-nodepool1-34917322-vmss000000   eastus   eastus-1
aks-nodepool1-34917322-vmss000001   eastus   eastus-2
aks-nodepool1-34917322-vmss000002   eastus   eastus-3
```

## <a name="verify-pod-distribution-across-zones"></a>Comprobar la distribución de pods entre zonas

Como se documenta en [Etiquetas, anotaciones y taints ampliamente conocidas][kubectl-well_known_labels], Kubernetes usa la etiqueta `failure-domain.beta.kubernetes.io/zone` para distribuir automáticamente los pods en un controlador o servicio de replicación en las diferentes zonas disponibles. Para probarlo, puede escalar verticalmente el clúster de 3 a 5 nodos para comprobar la propagación correcta del pod:

```azurecli-interactive
az aks scale \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 5
```

Cuando la operación de escalado se complete después de unos minutos, el comando `kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"` de un shell de Bash debe proporcionar una salida similar a la de este ejemplo:

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
Name:       aks-nodepool1-28993262-vmss000003
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000004
            failure-domain.beta.kubernetes.io/zone=eastus2-2
```

Ahora hay dos nodos adicionales en las zonas 1 y 2. Puede implementar una aplicación que consta de tres réplicas. Se va a usar NGINX como ejemplo:

```console
kubectl create deployment nginx --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
kubectl scale deployment nginx --replicas=3
```

Si se miran los nodos en los que se están ejecutando los pods, se observa que estos se ejecutan en los nodos correspondientes a tres zonas de disponibilidad diferentes. Por ejemplo, con el comando `kubectl describe pod | grep -e "^Name:" -e "^Node:"` de un shell de Bash se obtendría una salida similar a esta:

```console
Name:         nginx-6db489d4b7-ktdwg
Node:         aks-nodepool1-28993262-vmss000000/10.240.0.4
Name:         nginx-6db489d4b7-v7zvj
Node:         aks-nodepool1-28993262-vmss000002/10.240.0.6
Name:         nginx-6db489d4b7-xz6wj
Node:         aks-nodepool1-28993262-vmss000004/10.240.0.8
```

Como puede ver en la salida anterior, el primer pod se ejecuta en el nodo 0, que se encuentra en la zona de disponibilidad `eastus2-1`. El segundo pod se ejecuta en el nodo 2, que corresponde a `eastus2-3`, y el tercero, en el nodo 4, en `eastus2-2`. Sin necesidad de configuración adicional, Kubernetes extiende los pods correctamente en las tres zonas de disponibilidad.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se detalla cómo crear un clúster de AKS que usa zonas de disponibilidad. Para obtener más información sobre los clústeres de alta disponibilidad, consulte [Best practices for business continuity and disaster recovery in AKS][best-practices-bc-dr] (Procedimientos recomendados para la continuidad empresarial y recuperación ante desastres en AKS).

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-overview]: ../availability-zones/az-overview.md
[best-practices-bc-dr]: operator-best-practices-multi-region.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[standard-lb-limitations]: load-balancer-standard.md#limitations
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-aks-nodepool-add]: /cli/azure/aks/nodepool#az_aks_nodepool_add
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[vmss-zone-balancing]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md#zone-balancing
[arm-template-null]: ../azure-resource-manager/templates/template-expressions.md#null-values

<!-- LINKS - external -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-well_known_labels]: https://kubernetes.io/docs/reference/labels-annotations-taints/
