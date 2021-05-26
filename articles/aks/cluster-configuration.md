---
title: Configuración de clúster en Azure Kubernetes Service (AKS)
description: Aprenda a configurar un clúster en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 02/09/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: 3937e0a6c00de78acfa774ab6446d2b3d8e68206
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377130"
---
# <a name="configure-an-aks-cluster"></a>Configuración de un clúster de AKS

Como parte de la creación de un clúster de AKS, puede que tenga que personalizar la configuración del clúster para que se adapte a sus necesidades. En este artículo se presentan algunas opciones para personalizar el clúster de AKS.

## <a name="os-configuration"></a>Configuración del sistema operativo

AKS ahora es compatible con Ubuntu 18.04 como sistema operativo de nodo predeterminado (SO) en disponibilidad general (GA) para clústeres de versiones de Kubernetes superiores a 1.18. Para versiones inferiores a esa, AKS Ubuntu 16.04 sigue siendo la imagen base predeterminada. A partir de Kubernetes v1.18 y posteriores, la base predeterminada es AKS Ubuntu 18.04.

> [!IMPORTANT]
> Los grupos de nodos creados en Kubernetes v1.18 o versiones posteriores tienen una imagen de nodo `AKS Ubuntu 18.04` de forma predeterminada. Los grupos de nodos de una versión de Kubernetes compatible inferior a la versión 1.18 reciben `AKS Ubuntu 16.04` como la imagen de nodo, pero se actualizarán a `AKS Ubuntu 18.04` una vez que la versión Kubernetes del grupo de nodos se actualice a la versión 1.18 o posteriores.
> 
> Es muy recomendable que pruebe sus cargas de trabajo en los grupos de nodos de AKS Ubuntu 18.04 antes de usar los clústeres en la versión 1.18 o posteriores.


### <a name="use-aks-ubuntu-1804-ga-on-new-clusters"></a>Uso de AKS Ubuntu 18.04 (GA) en clústeres nuevos

Los clústeres creados en Kubernetes v1.18 o versiones posteriores tienen una imagen de nodo `AKS Ubuntu 18.04` de forma predeterminada. Los grupos de nodos en versiones de Kubernetes compatibles anteriores a la versión 1.18 seguirán recibiendo `AKS Ubuntu 16.04` como imagen de nodo, pero se actualizarán a `AKS Ubuntu 18.04` una vez que la versión de Kubernetes del clúster o del grupo de nodos se actualice a la versión 1.18 o posteriores.

Es muy recomendable que pruebe sus cargas de trabajo en los grupos de nodos de AKS Ubuntu 18.04 antes de usar los clústeres en la versión 1.18 o posteriores.

Para crear un clúster con la imagen de nodo `AKS Ubuntu 18.04`, solo tiene que crear un clúster que ejecute kubernetes v1.18 o una versión posterior, tal como se muestra a continuación:

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --kubernetes-version 1.18.14
```

### <a name="use-aks-ubuntu-1804-ga-on-existing-clusters"></a>Uso de AKS Ubuntu 18.04 (GA) en clústeres existentes

Los clústeres creados en Kubernetes v1.18 o versiones posteriores tienen una imagen de nodo `AKS Ubuntu 18.04` de forma predeterminada. Los grupos de nodos en versiones de Kubernetes compatibles anteriores a la versión 1.18 seguirán recibiendo `AKS Ubuntu 16.04` como imagen de nodo, pero se actualizarán a `AKS Ubuntu 18.04` una vez que la versión de Kubernetes del clúster o del grupo de nodos se actualice a la versión 1.18 o posteriores.

Es muy recomendable que pruebe sus cargas de trabajo en los grupos de nodos de AKS Ubuntu 18.04 antes de usar los clústeres en la versión 1.18 o posteriores.

Si los clústeres o grupos de nodos están listos para la imagen de nodo `AKS Ubuntu 18.04`, simplemente puede actualizarlos a una versión 1.18 o posterior, tal como se muestra a continuación.

```azurecli
az aks upgrade --name myAKSCluster --resource-group myResourceGroup --kubernetes-version 1.18.14
```

Si solo quiere actualizar un grupo de nodos:

```azurecli
az aks nodepool upgrade -name ubuntu1804 --cluster-name myAKSCluster --resource-group myResourceGroup --kubernetes-version 1.18.14
```

### <a name="test-aks-ubuntu-1804-ga-on-existing-clusters"></a>Prueba de AKS Ubuntu 18.04 (GA) en clústeres existentes

Los grupos de nodos creados en Kubernetes v1.18 o versiones posteriores tienen una imagen de nodo `AKS Ubuntu 18.04` de forma predeterminada. Los grupos de nodos en versiones de Kubernetes compatibles anteriores a la versión 1.18 seguirán recibiendo `AKS Ubuntu 16.04` como imagen de nodo, pero se actualizarán a `AKS Ubuntu 18.04` una vez que la versión de Kubernetes del grupo de nodos se actualice a la versión 1.18 o posteriores.

Es muy recomendable probar las cargas de trabajo en los grupos de nodos de AKS Ubuntu 18.04 antes de actualizar los grupos de nodos de producción.

Para crear un grupo de nodos con la imagen de nodo `AKS Ubuntu 18.04`, solo tiene que crear un grupo de nodos que ejecute kubernetes v1.18 o una versión posterior. El plano de control del clúster tiene que estar como mínimo en la versión 1.18 o una versión posterior, pero los demás grupos de nodos pueden permanecer en una versión anterior de Kubernetes.
En el ejemplo siguiente, primero se actualiza el plano de control y luego se crea un nuevo grupo de nodos con la versión 1.18 que recibirá la versión del sistema operativo de la nueva imagen de nodo.

```azurecli
az aks upgrade --name myAKSCluster --resource-group myResourceGroup --kubernetes-version 1.18.14 --control-plane-only

az aks nodepool add --name ubuntu1804 --cluster-name myAKSCluster --resource-group myResourceGroup --kubernetes-version 1.18.14
```

## <a name="container-runtime-configuration"></a>Configuración del entorno de ejecución de contenedor

Un entorno de ejecución de contenedor es un software que ejecuta contenedores y administra imágenes de contenedor en un nodo. El entorno de ejecución ayuda a abstraer la funcionalidad específica del sistema operativo o de sys-call para ejecutar contenedores en Linux o Windows. En el caso de los grupos de nodos de Linux, se usa `containerd` para los grupos de nodos con Kubernetes versión 1.19 y posteriores, y se usa Docker para los grupos de nodos con Kubernetes 1.18 y versiones anteriores. En el caso de los grupos de nodos de Windows Server 2019, `containerd` está disponible en versión preliminar y se puede usar en grupos de nodos con Kubernetes 1.20 y versiones posteriores, pero Docker todavía se usa de forma predeterminada.

[`Containerd`](https://containerd.io/) es un entorno de ejecución de contenedor básico compatible con [OCI](https://opencontainers.org/) (Open Container Initiative) que proporciona el conjunto mínimo de funciones necesarias para ejecutar contenedores y administrar imágenes en un nodo. Fue [donado](https://www.cncf.io/announcement/2017/03/29/containerd-joins-cloud-native-computing-foundation/) a la Cloud Native Compute Foundation (CNCF) en marzo de 2017. La versión actual de Moby (Docker ascendente) que AKS ya usa se basa en `containerd` y ya aprovecha sus ventajas, como se mostró anteriormente.

Con los grupos de nodos y los nodos basados en `containerd`, en lugar de comunicarse con `dockershim`, el kubelet se comunicará directamente con `containerd` mediante el complemento CRI (interfaz del entorno de ejecución de contenedor) y eliminará los saltos adicionales del flujo, en comparación con la implementación de CRI de Docker. Como tal, verá una mejor latencia de inicio del pod y menor uso de recursos (CPU y memoria).

Usar `containerd` para los nodos AKS mejora la latencia de inicio del pod y reduce el consumo de recursos de nodo del entorno de ejecución de contenedor. Estas mejoras son posibles gracias a la nueva arquitectura, en la que el kubelet se comunica directamente con `containerd` mediante el complemento CRI; sin embargo, en la arquitectura de Moby o Docker, el kubelet se comunica con `dockershim` y con el motor de Docker antes de llegar a `containerd` y, por lo tanto, tiene saltos adicionales en el flujo.

![Docker CRI 2](media/cluster-configuration/containerd-cri.png)

`Containerd` funciona en todas las versiones de disponibilidad general de Kubernetes en AKS y en todas las versiones de Kubernetes anteriores a la versión 1.19, y admite todas las características de Kubernetes y AKS.

> [!IMPORTANT]
> Los clústeres con grupos de nodos de Linux creados en Kubernetes versión 1.19 o versiones posteriores utilizan `containerd` de manera predeterminada para su entorno de ejecución de contenedores. Los clústeres con grupos de nodos en versiones de Kubernetes admitidas anteriormente reciben Docker para su entorno de ejecución de contenedores. Los grupos de nodos de Linux se actualizarán a `containerd` una vez que la versión de Kubernetes del grupo de nodos se actualice a una versión compatible con `containerd`. Puede seguir usando los clústeres y grupos de nodos de Docker en las versiones admitidas más antiguas hasta que se retire el soporte técnico.
> 
> El uso de `containerd` con grupos de nodos de Windows Server 2019 está actualmente en versión preliminar. Para más información, consulte [Adición de un grupo de nodos de Windows Server con `containerd`][aks-add-np-containerd].
> 
> Se recomienda encarecidamente probar las cargas de trabajo en grupos de nodos de AKS con `containerd` antes de usar clústeres con una versión de Kubernetes compatible con `containerd` para sus grupos de nodos.

### <a name="containerd-limitationsdifferences"></a>Limitaciones y diferencias de `Containerd`

* Para `containerd`, se recomienda usar [`crictl`](https://kubernetes.io/docs/tasks/debug-application-cluster/crictl) como CLI en lugar de la CLI de Docker para **solucionar problemas** de pods, contenedores e imágenes de contenedor en nodos Kubernetes (por ejemplo, `crictl ps`). 
   * No proporciona la funcionalidad completa de la CLI de Docker. Está pensado solo para solucionar problemas.
   * `crictl` ofrece una vista de los contenedores más compatible con Kubernetes, con conceptos como pods, etc.
* `Containerd` configura el registro con el formato de registro `cri` normalizado (que es diferente de lo que se obtiene actualmente del controlador JSON de Docker). La solución de registro debe admitir el formato de registro `cri` (como [Azure Monitor para contenedores](../azure-monitor/containers/container-insights-enable-new-cluster.md))
* Ya no puede tener acceso al motor de Docker, `/var/run/docker.sock`, ni usar Docker en Docker (DinD).
  * Si actualmente extrae los registros de aplicación o los datos de supervisión del motor de Docker, utilice en su lugar [Azure Monitor para contenedores](../azure-monitor/containers/container-insights-enable-new-cluster.md), por ejemplo. Además, AKS no admite la ejecución de comandos fuera de banda en los nodos del agente que podrían provocar inestabilidad.
  * Incluso si usa Docker, es muy desaconsejable crear imágenes y aprovechar directamente el motor de Docker con los métodos anteriores. Kubernetes no es totalmente consciente de esos recursos consumidos y esos enfoques presentan numerosos problemas, que se detallan [aquí](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) y [aquí](https://securityboulevard.com/2018/05/escaping-the-whale-things-you-probably-shouldnt-do-with-docker-part-1/), por ejemplo.
* Compilación de imágenes: puede seguir usando su flujo de trabajo de compilación de Docker como de costumbre, a menos que vaya a compilar imágenes dentro del clúster de AKS. En este caso, considere la posibilidad de utilizar el enfoque recomendado para compilar imágenes con [ACR Tasks](../container-registry/container-registry-quickstart-task-cli.md). O bien, una alternativa más segura dentro del clúster, como [docker buildx](https://github.com/docker/buildx).

## <a name="generation-2-virtual-machines"></a>Máquinas virtuales de generación 2

Azure admite [máquinas virtuales de generación 2 (Gen2)](../virtual-machines/generation-2.md). Las máquinas virtuales de generación 2 admiten características clave que no se admiten en las máquinas virtuales de generación 1 (Gen1). Estas características incluyen una memoria mayor, Intel Software Guard Extensions (SGX Intel) y memoria persistente virtualizada (vPMEM).

Las VM de generación 2 usan la nueva arquitectura de arranque basado en UEFI en lugar de la arquitectura basada en BIOS que utilizan las VM de generación 1.
Solo determinadas SKU y tamaños admiten máquinas virtuales de generación 2. Consulte la [lista de tamaños admitidos](../virtual-machines/generation-2.md#generation-2-vm-sizes) para ver si la SKU admite o requiere Gen2.

Además, no todas las imágenes de máquina virtual admiten Gen2; en ASK, las máquinas virtuales Gen2 usaran la nueva [imagen de Ubuntu 18.04 para AKS](#os-configuration). Esta imagen admite todas las SKU y tamaños de Gen2.

## <a name="ephemeral-os"></a>SO efímero

De manera predeterminada, Azure replica automáticamente el disco del sistema operativo de una máquina virtual en Azure Storage para evitar la pérdida de datos cuando hay que reubicar la máquina virtual en otro host. Sin embargo, como los contenedores no están diseñados para preservar el estado local, este comportamiento ofrece un valor limitado y, además, presenta algunos inconvenientes, como un aprovisionamiento más lento de los nodos y una mayor latencia de lectura y escritura.

En cambio, los discos de sistema operativo efímero solo se almacenan en el equipo host, al igual que los discos temporales. Esto proporciona una latencia de lectura y escritura menor, junto con escalabilidad de nodos y actualizaciones de clúster más rápidas.

Al igual que sucede con un disco temporal, el disco de sistema operativo efímero está incluido en el precio de la máquina virtual, por lo que no lleva asociado ningún costo adicional de almacenamiento.

> [!IMPORTANT]
>Si un usuario no solicita explícitamente los discos administrados para el sistema operativo, AKS tomará como valor predeterminado el sistema operativo efímero si es posible para una configuración de grupo de nodos determinada.

Cuando se usa un sistema operativo efímero, el disco del sistema operativo debe caber en la memoria caché de la máquina virtual. Los tamaños de la memoria caché de la máquina virtual están disponibles en la [documentación de Azure](../virtual-machines/dv3-dsv3-series.md) entre paréntesis junto a rendimiento de E/S ("tamaño de caché en GiB").

Con el tamaño de máquina virtual predeterminado de AKS Standard_DS2_v2 con el tamaño de disco de sistema operativo predeterminado de 100 GB como ejemplo, este tamaño de máquina virtual es compatible con el sistema operativo efímero pero solo tiene 86 GB de tamaño de caché. El valor predeterminado de esta configuración serán discos administrados si el usuario no lo especifica explícitamente. Si un usuario solicita explícitamente el sistema operativo efímero, recibirá un error de validación.

Si un usuario solicita el mismo tamaño Standard_DS2_v2 con un disco del sistema operativo de 60 GB, esta configuración sería el sistema operativo efímero de manera predeterminada: el tamaño solicitado de 60 GB es menor que el tamaño máximo de la caché de 86 GB.

Al usar el tamaño Standard_D8s_v3 con un disco de sistema operativo de 100 GB, este tamaño de máquina virtual es compatible con el sistema operativo efímero y tiene 200 GB de espacio en caché. Si un usuario no especifica el tipo de disco del sistema operativo, el grupo de nodos recibirá un sistema operativo efímero de manera predeterminada. 

El sistema operativo efímero requiere la versión 2.15.0 de la CLI de Azure.

### <a name="use-ephemeral-os-on-new-clusters"></a>Uso del sistema operativo efímero en clústeres nuevos

Configure el clúster para que utilice discos de sistema operativo efímero al crearlo. Use la marca `--node-osdisk-type` para establecer el sistema operativo efímero como el tipo de disco de sistema operativo para el nuevo clúster.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup -s Standard_DS3_v2 --node-osdisk-type Ephemeral
```

Si quiere crear un clúster normal con discos de sistema operativo conectados a la red, especifique la etiqueta `--node-osdisk-type=Managed`. También puede optar por añadir más grupos de nodos de sistema operativo efímero, tal y como se indica a continuación.

### <a name="use-ephemeral-os-on-existing-clusters"></a>Uso del sistema operativo efímero en clústeres existentes
Configure un nuevo grupo de nodos para usar discos de sistema operativo efímero. Utilice la marca `--node-osdisk-type` para establecerla como el tipo de disco de para ese grupo de nodos.

```azurecli
az aks nodepool add --name ephemeral --cluster-name myAKSCluster --resource-group myResourceGroup -s Standard_DS3_v2 --node-osdisk-type Ephemeral
```

> [!IMPORTANT]
> El sistema operativo efímero permite implementar imágenes de instancias y de máquinas virtuales cuyo tamaño máximo sea equivalente al de la memoria caché de máquina virtual. En el caso de AKS, la configuración predeterminada del disco de sistema operativo del nodo usa 128 GB, lo cual significa que necesitará una máquina virtual con una memoria caché cuyo tamaño sea superior a 128 GB. El tamaño de VM predeterminado Standard_DS2_v2 tiene un tamaño de caché de 86 GB, que no es lo suficientemente grande. Las máquinas virtuales Standard_DS3_v2 tienen un tamaño de caché de 172 GB, que es lo suficientemente grande. También puede reducir el tamaño predeterminado del disco de sistema operativo mediante `--node-osdisk-size`. El tamaño mínimo de las imágenes de AKS es 30 GB. 

Si quiere crear grupos de nodos con discos de sistema operativo conectados a red, puede especificar `--node-osdisk-type Managed` para hacerlo.

## <a name="custom-resource-group-name"></a>Nombre del grupo de recursos personalizado

Al implementar un clúster de Azure Kubernetes Service en Azure, se crea un segundo grupo de recursos para los nodos de trabajo. De forma predeterminada, AKS asignará un nombre al grupo de recursos del nodo `MC_resourcegroupname_clustername_location`, pero también puede proporcionar su propio nombre.

Para especificar su propio nombre de grupo de recursos, instale la versión de la extensión de la CLI de Azure aks-preview 0.3.2 o una posterior. Con la CLI de Azure, use el parámetro `--node-resource-group` del comando `az aks create` para especificar un nombre personalizado para el grupo de recursos. Si usa una plantilla de Azure Resource Manager para implementar un clúster de AKS, puede definir el nombre del grupo de recursos mediante la propiedad `nodeResourceGroup`.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --node-resource-group myNodeResourceGroup
```

El proveedor de recursos de Azure crea automáticamente el grupo de recursos secundario en su propia suscripción. Solo puede especificar el nombre del grupo de recursos personalizado al crear el clúster. 

Cuando trabaje con el grupo de recursos del nodo, tenga en cuenta que no puede:

- Especificar un grupo de recursos existente para el grupo de recursos del nodo.
- Especificar otra suscripción para el grupo de recursos del nodo.
- Cambiar el nombre del grupo de recursos del nodo una vez se haya creado el clúster.
- Especificar los nombres de los recursos administrados del grupo de recursos del nodo.
- Modificar o eliminar las etiquetas creadas en Azure de los recursos administrados del grupo de recursos del nodo.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [actualizar las imágenes del nodo](node-image-upgrade.md) del clúster.
- Consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)](upgrade-cluster.md) para más información sobre cómo actualizar el clúster a la versión más reciente de Kubernetes.
- Más información sobre [`containerd` y Kubernetes](https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/)
- Consulte la lista de [Preguntas más frecuentes sobre AKS](faq.md) para encontrar respuestas a algunas preguntas comunes sobre AKS.
- Obtenga más información sobre los [discos de sistema operativo efímero](../virtual-machines/ephemeral-os-disks.md).


<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[aks-add-np-containerd]: windows-container-cli.md#add-a-windows-server-node-pool-with-containerd-preview