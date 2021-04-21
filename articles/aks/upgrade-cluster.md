---
title: Actualización de un clúster de Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo actualizar un clúster de Azure Kubernetes Service (AKS) para obtener las últimas características y actualizaciones de seguridad.
services: container-service
ms.topic: article
ms.date: 12/17/2020
ms.openlocfilehash: d6a5ed468541090d433dba732707a59841e6ff41
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779622"
---
# <a name="upgrade-an-azure-kubernetes-service-aks-cluster"></a>Actualización de un clúster de Azure Kubernetes Service (AKS)

Parte del ciclo de vida del clúster de AKS implica efectuar actualizaciones periódicas a la versión más reciente de Kubernetes. Es importante que aplique las versiones de seguridad más recientes o que actualice el software para obtener las últimas características. En este artículo se muestra cómo actualizar los componentes principales o un único grupo de nodos predeterminado en un clúster de AKS.

Para los clústeres de AKS que usan varios grupos de nodos o nodos de Windows Server, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

## <a name="before-you-begin"></a>Antes de empezar

Para este artículo es preciso usar la versión 2.0.65 de la CLI de Azure, o cualquier versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

> [!WARNING]
> Una actualización del clúster de AKS desencadena un acordonamiento y purga de los nodos. Si tiene una cuota de proceso baja disponible, se puede producir un error en la actualización. Para más información, consulte [Solicitudes de aumento de cuota](../azure-portal/supportability/resource-manager-core-quotas-request.md).

## <a name="check-for-available-aks-cluster-upgrades"></a>Compruebe las actualizaciones disponibles del clúster de AKS

Para comprobar qué versiones de Kubernetes están disponibles para su clúster, use el comando [az aks get-upgrades][az-aks-get-upgrades]. En el siguiente ejemplo se comprueban las actualizaciones disponibles para *myAKSCluster* en *myResourceGroup*:

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

> [!NOTE]
> Cuando se actualiza un clúster de AKS compatible, no pueden omitirse las versiones secundarias de Kubernetes. Todas las actualizaciones deben realizarse secuencialmente con arreglo al número de versión principal. Por ejemplo, las actualizaciones comprendidas entre *1.14.x* -> *1.15.x* o *1.15.x* -> *1.16.x* están permitidas, pero las comprendidas entre *1.14.x* -> *1.16.x* no lo están. 
> > Solo se pueden omitir varias versiones al actualizar de una _versión que no es compatible_ a una _versión compatible_. Por ejemplo, se puede efectuar una actualización de una versión *1.10.x* no compatible a una versión *1.15.x* compatible.

La siguiente salida de ejemplo muestra que el clúster se puede actualizar a las versiones *1.19.1* y *1.19.3*:

```console
Name     ResourceGroup    MasterVersion    Upgrades
-------  ---------------  ---------------  --------------
default  myResourceGroup  1.18.10          1.19.1, 1.19.3
```

Si no hay ninguna actualización disponible, verá el mensaje:

```console
ERROR: Table output unavailable. Use the --query option to specify an appropriate query. Use --debug for more info.
```

## <a name="customize-node-surge-upgrade"></a>Personalización de la actualización de sobrecargas de nodo

> [!Important]
> Los sobrecargas de nodo requieren la cuota de suscripción para el recuento máximo de sobrecargas solicitado para cada operación de actualización. Por ejemplo, un clúster con 5 grupos de nodos, cada uno con un recuento de 4 nodos, tiene un total de 20 nodos. Si cada grupo de nodos tiene un valor de sobrecarga máxima del 50 %, se requiere un proceso adicional y una cuota de IP de 10 nodos (2 nodos x 5 grupos) para completar la actualización.
>
> Si usa Azure CNI, compruebe que haya direcciones IP disponibles en la subred, además de [cumplir los requisitos de direcciones IP de Azure CNI](configure-azure-cni.md).

De forma predeterminada, AKS configura las actualizaciones para la sobrecarga con un nodo adicional. Un valor predeterminado de uno para la configuración de sobrecarga máxima permitirá a AKS minimizar la interrupción de la carga de trabajo mediante la creación de un nodo adicional antes de acordonar o purgar las aplicaciones existentes para reemplazar un nodo de una versión anterior. El valor de sobrecarga máxima se puede personalizar por grupo de nodos para permitir un equilibrio entre la velocidad de actualización y la interrupción de la actualización. Al aumentar el valor de sobrecarga máxima, el proceso de actualización se completa más rápido, pero si se establece un valor alto para la sobrecarga máxima, pueden producirse interrupciones durante el proceso de actualización. 

Por ejemplo, un valor de sobrecarga máxima del 100 % proporciona el proceso de actualización más rápido posible (se duplica el número de nodos), pero también hace que todos los nodos del grupo de nodos se purguen simultáneamente. Es posible que desee usar un valor más alto, como este, para los entornos de prueba. En el caso de los grupos de nodos de producción, se recomienda un valor de sobrecarga máxima del 33 %.

AKS acepta valores enteros y un valor de porcentaje para la sobrecarga máxima. Un entero como "5" indica cinco nodos adicionales para la sobrecarga. Un valor de "50 %" indica un valor de sobrecarga de la mitad del número de nodos actual del grupo. Los valores de porcentaje de sobrecarga máxima pueden ser de 1 % como mínimo y 100 % como máximo. Un valor de porcentaje se redondea al número de nodos más próximo. Si el valor de sobrecarga máxima es inferior al número de nodos actual en el momento de la actualización, el número de nodos actual se usa para el valor de sobrecarga máxima.

Durante una actualización, el valor de sobrecarga máxima puede ser 1 como mínimo y un valor igual al número de nodos del grupo de nodos como máximo. Puede establecer valores mayores, pero el número máximo de nodos que se usan para la sobrecarga máxima no será mayor que el número de nodos del grupo en el momento de la actualización.

> [!Important]
> La configuración de sobrecarga máxima en un grupo de nodos es permanente.  Las actualizaciones posteriores de Kubernetes o las actualizaciones de la versión de nodo usarán esta configuración. Puede cambiar el valor de sobrecarga máxima para los grupos de nodos en cualquier momento. En el caso de los grupos de nodos de producción, se recomienda un valor de sobrecarga máxima del 33 %.

Use los siguientes comandos para establecer los valores de sobrecarga máxima para los grupos de nodos nuevos o existentes.

```azurecli-interactive
# Set max surge for a new node pool
az aks nodepool add -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 33%
```

```azurecli-interactive
# Update max surge for an existing node pool 
az aks nodepool update -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 5
```

## <a name="upgrade-an-aks-cluster"></a>Actualización de un clúster de AKS

Con una lista de las versiones disponibles para el clúster de AKS, use el comando [az aks upgrade][az-aks-upgrade] para realizar la actualización. Durante el proceso de actualización, AKS se ocupará de lo siguiente: 
- Agregar un nuevo nodo de búfer (o tantos nodos como se hayan configurado en [sobrecarga máxima](#customize-node-surge-upgrade)) al clúster que ejecuta la versión especificada de Kubernetes. 
- [Acordonar y purgar][kubernetes-drain] uno de los nodos antiguos para minimizar la interrupción de las aplicaciones en ejecución. Si se usa la sobrecarga máxima, [acordonará y purgará][kubernetes-drain] tantos nodos al mismo tiempo como el número de nodos de búfer que se haya especificado. 
- Cuando el nodo anterior se ha purgado por completo, se restablecerá la imagen inicial para recibir la nueva versión y se convertirá en el nodo de búfer para el siguiente nodo que se va a actualizar. 
- Este proceso se repite hasta que se hayan actualizado todos los nodos del clúster. 
- Al final del proceso, se eliminará el último nodo del búfer, manteniendo el número de nodos de agente existentes, así como el equilibrio de zona.

```azurecli-interactive
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION
```

Dicha actualización tarda varios minutos. El tiempo exacto dependerá del número de nodos que tenga.

> [!IMPORTANT]
> Asegúrese de que cualquier `PodDisruptionBudgets` (PDB) permita que se mueva al menos una réplica de pod a la vez; en caso contrario, se producirá un error en la operación de purga o expulsión.
> Si se produce un error en la operación de purga, se producirá un error por diseño en la operación de actualización para garantizar que no se interrumpan las aplicaciones. Corrija lo que hizo que se detuviese la operación (PDB incorrectos, falta de cuota, etc.) y vuelva a intentar la operación.

Para confirmar que la actualización se ha realizado correctamente, use el comando [az aks show][az-aks-show]:

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

En la salida de ejemplo siguiente se muestra que el clúster ahora ejecuta la versión *1.18.10*:

```json
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.18.10              Succeeded            myakscluster-dns-379cbbb9.hcp.eastus.azmk8s.io
```

## <a name="set-auto-upgrade-channel"></a>Establecimiento de un canal de actualización automática

Además de actualizar manualmente un clúster, puede establecer un canal de actualización automática en el clúster. Están disponibles los siguientes canales de actualización:

|Canal| Acción | Ejemplo
|---|---|---|
| `none`| Deshabilita las actualizaciones automáticas y mantiene la versión actual del clúster de Kubernetes.| Si no se hacen cambios, se usa la configuración predeterminada.|
| `patch`| Actualiza automáticamente el clúster a la última versión de revisión compatible cuando esté disponible, mientras se mantiene la misma versión secundaria.| Por ejemplo, si un clúster ejecuta la versión *1.17.7* y las versiones *1.17.9*, *1.18.4*, *1.17.9* y *1.19.1* están disponibles, se actualizará a la *1.18.6*.|
| `stable`| Actualiza automáticamente el clúster a la última versión de revisión compatible en la versión secundaria *N-1*, donde *N* es la última versión secundaria compatible.| Por ejemplo, si un clúster ejecuta la versión *1.17.7* y las versiones *1.17.9*, *1.18.4*, *1.18.6* y *1.19.1* están disponibles, el clúster se actualiza a *1.18.6*.
| `rapid`| Actualiza automáticamente el clúster a la última versión de revisión compatible en la última versión secundaria compatible.| En los casos en los que el clúster está en una versión de Kubernetes que se encuentra en una versión secundaria *N-2* donde *N* es la última versión secundaria, el clúster se actualiza primero a la última versión de revisión compatible en la versión secundaria *N-1*. Por ejemplo, si un clúster ejecuta la versión *1.17.7* y las versiones *1.17.9*, *1.18.4*, *1.18.6* y *1.19.1* están disponibles, el clúster se actualiza primero a *1.18.6* y, a continuación, se actualiza a *1.19.1*.

> [!NOTE]
> La actualización automática del clúster solo se actualiza a las versiones de disponibilidad general de Kubernetes y no se actualizará a las versiones preliminares.

La actualización automática de un clúster sigue el mismo proceso que la manual. Para obtener más detalles, consulte [Actualización de un clúster de AKS][upgrade-cluster].

La actualización automática del clúster para los clústeres de AKS es una característica en vista previa (GB).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Registre la marca de la característica `AutoUpgradePreview` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n AutoUpgradePreview
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AutoUpgradePreview')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

Para establecer el canal de actualización automática al crear un clúster, use el parámetro *auto-upgrade-channel*, similar al ejemplo siguiente.

```azurecli-interactive
az aks create --resource-group myResourceGroup --name myAKSCluster --auto-upgrade-channel stable --generate-ssh-keys
```

Para establecer el canal de actualización automática en el clúster existente, actualice el parámetro *auto-upgrade-channel*, similar al ejemplo siguiente.

```azurecli-interactive
az aks update --resource-group myResourceGroup --name myAKSCluster --auto-upgrade-channel stable
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo actualizar un clúster de AKS existente. Para más información acerca de la implementación y administración de clústeres de AKS, consulte el conjunto de tutoriales.

> [!div class="nextstepaction"]
> [Tutoriales de AKS][aks-tutorial-prepare-app]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az_aks_get_upgrades
[az-aks-upgrade]: /cli/azure/aks#az_aks_upgrade
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-provider-register]: /cli/azure/provider#az_provider_register
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[upgrade-cluster]:  #upgrade-an-aks-cluster
