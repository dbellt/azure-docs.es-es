---
title: Personalización de la configuración de nodos en los grupos de nodos de Azure Kubernetes Service (AKS) (versión preliminar)
description: Aprenda a personalizar la configuración de los nodos y los grupos de nodos de clúster de Azure Kubernetes Service (AKS).
ms.service: container-service
ms.topic: article
ms.date: 12/03/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: c2173118b58ca92d69286fb36014872c19058bd6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779982"
---
# <a name="customize-node-configuration-for-azure-kubernetes-service-aks-node-pools-preview"></a>Personalización de la configuración de nodos en los grupos de nodos de Azure Kubernetes Service (AKS) (versión preliminar)

La personalización de la configuración de los nodos permite configurar o ajustar los valores del sistema operativo (SO) o los parámetros de kubelet para que se adapten a las necesidades de las cargas de trabajo. Al crear un clúster de AKS o agregar un grupo de nodos al clúster, puede personalizar un subconjunto de los valores de configuración de kubelet y del sistema operativo más usados. Para configurar otros valores aparte de los de este subconjunto, [use un demonio establecido para personalizar las configuraciones necesarias sin perder la compatibilidad de AKS con los nodos](support-policies.md#shared-responsibility).

## <a name="register-the-customnodeconfigpreview-preview-feature"></a>Registro de la característica en vista previa (GB) `CustomNodeConfigPreview`

Para crear un clúster o un grupo de nodos de AKS para personalizar los parámetros de kubelet o la configuración del sistema operativo, debe habilitar la marca de características `CustomNodeConfigPreview`.

Registre la marca de la característica `CustomNodeConfigPreview` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "CustomNodeConfigPreview"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/CustomNodeConfigPreview')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

Para crear un clúster o un grupo de nodos de AKS para personalizar los parámetros de kubelet o la configuración del sistema operativo, necesita la extensión más reciente de la CLI de Azure: *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

## <a name="use-custom-node-configuration"></a>Uso de la configuración de nodos personalizada

### <a name="kubelet-custom-configuration"></a>Configuración personalizada de kubelet

A continuación, se enumeran los parámetros admitidos y los valores aceptados de kubelet.

| Parámetro | Valores o intervalo permitidos | Default | Descripción |
| --------- | ----------------------- | ------- | ----------- |
| `cpuManagerPolicy` | ninguno, estático | ninguno | La directiva estática permite que los contenedores de [pods garantizados](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) con un número entero de solicitudes de CPU accedan a las CPU exclusivas del nodo. |
| `cpuCfsQuota` | true, false | true |  Habilita o deshabilita la aplicación de cuotas de CFS de CPU para contenedores que especifican límites de CPU. | 
| `cpuCfsQuotaPeriod` | Intervalo en milisegundos (ms) | `100ms` | Establece el valor del período de cuota de CFS de CPU. | 
| `imageGcHighThreshold` | 0-100 | 85 | Porcentaje de uso del disco después del cual siempre se ejecuta la recolección de elementos no utilizados de la imagen. Uso mínimo de disco que **desencadenará** la recolección de elementos no utilizados. Para deshabilitar la recolección de elementos no utilizados de la imagen, establezca este valor en 100. | 
| `imageGcLowThreshold` | 0-100, no mayor que `imageGcHighThreshold`. | 80 | Porcentaje de uso del disco antes del cual la recolección de elementos no utilizados de la imagen nunca se ejecuta. Uso mínimo de disco que **puede** desencadenar la recolección de elementos no utilizados. |
| `topologyManagerPolicy` | ninguno, mejor esfuerzo, restringido, un solo nodo numa | ninguno | Optimiza la alineación de nodos NUMA. Consulte [aquí](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/) para más información. Solo kubernetes v1.18+. |
| `allowedUnsafeSysctls` | `kernel.shm*`, `kernel.msg*`, `kernel.sem`, `fs.mqueue.*`, `net.*` | None | Lista permitida de patrones no seguros sysctls o sysctl. | 

### <a name="linux-os-custom-configuration"></a>Configuración personalizada del sistema operativo Linux

A continuación, se enumeran las configuraciones del sistema admitidas y los valores aceptados.

#### <a name="file-handle-limits"></a>Límites del identificador de archivo

Cuando se atiende mucho tráfico, es habitual que el tráfico que se atiende provenga de un gran número de archivos locales. Puede modificar los siguientes valores del kernel y los límites integrados para que pueda administrar una mayor cantidad, a cambio de usar algo más de memoria del sistema.

| Configuración | Valores o intervalo permitidos | Default | Descripción |
| ------- | ----------------------- | ------- | ----------- |
| `fs.file-max` | 8192 - 12000500 | 709620 | Número máximo de identificadores de archivo que asignará el kernel de Linux; si aumenta este valor, puede aumentar el número máximo de archivos abiertos permitidos. |
| `fs.inotify.max_user_watches` | 781250 - 2097152 | 1 048 576 | Número máximo de inspecciones de archivos permitidos por el sistema. Cada *inspección* tiene aproximadamente 90 bytes en un kernel de 32 bits y unos 160 bytes en un kernel de 64 bits. | 
| `fs.aio-max-nr` | 65536 - 6553500 | 65536 | El elemento aio-nr muestra el número actual de solicitudes io asincrónicas en todo el sistema. aio-max-nr permite cambiar el valor máximo hasta el que puede aumentar aio-nr. |
| `fs.nr_open` | 8192 - 20000500 | 1 048 576 | El número máximo de identificadores de archivo que un proceso puede asignar. |


#### <a name="socket-and-network-tuning"></a>Ajuste de socket y red

En el caso de los nodos de agente, que se espera que administren un gran número de sesiones simultáneas, puede usar el subconjunto de opciones de red y TCP que se muestra a continuación, que puede modificar en cada grupo de nodos. 

| Configuración | Valores o intervalo permitidos | Default | Descripción |
| ------- | ----------------------- | ------- | ----------- |
| `net.core.somaxconn` | 4096 - 3240000 | 16384 | Número máximo de solicitudes de conexión que se pueden poner en cola para cualquier socket de escucha determinado. Un límite superior para el valor del parámetro de trabajo pendiente que se pasa a la función [listen(2)](http://man7.org/linux/man-pages/man2/listen.2.html). Si el argumento de trabajo pendiente es mayor que `somaxconn`, se trunca de forma silenciosa hasta este límite.
| `net.core.netdev_max_backlog` | 1000 - 3240000 | 1000 | Número máximo de paquetes, en cola en el lado INPUT, cuando la interfaz recibe paquetes a una velocidad mayor de la que el kernel puede procesarlos. |
| `net.core.rmem_max` | 212992 - 134217728 | 212992 | Tamaño máximo del búfer de sockets de recepción en bytes. |
| `net.core.wmem_max` | 212992 - 134217728 | 212992 | Tamaño máximo del búfer de sockets de envío en bytes. | 
| `net.core.optmem_max` | 20480 - 4194304 | 20480 | Tamaño máximo del búfer auxiliar (búfer de memoria de opción) permitido por socket. La memoria de opción de socket se utiliza en algunos casos para almacenar estructuras adicionales relacionadas con el uso del socket. | 
| `net.ipv4.tcp_max_syn_backlog` | 128 - 3240000 | 16384 | Número máximo de solicitudes de conexión en cola que todavía no han recibido una confirmación del cliente que se está conectando. Si se supera este número, el kernel comenzará a rechazar solicitudes. |
| `net.ipv4.tcp_max_tw_buckets` | 8000 - 1440000 | 32 768 | Número máximo de sockets `timewait` retenidos por el sistema simultáneamente. Si se supera este número, el socket time-wait se destruye inmediatamente y se imprime la advertencia. |
| `net.ipv4.tcp_fin_timeout` | 5 - 120 | 60 | La cantidad de tiempo que una conexión huérfana (ninguna aplicación hace referencia a ella) permanecerá en el estado FIN_WAIT_2 antes de que se anule en el extremo local. |
| `net.ipv4.tcp_keepalive_time` | 30 - 432000 | 7200 | Frecuencia con la que TCP envía mensajes `keepalive` cuando `keepalive` está habilitado. |
| `net.ipv4.tcp_keepalive_probes` | 1 - 15 | 9 | Número de sondeos `keepalive` que envía TCP, hasta que decide que la conexión se ha interrumpido. |
| `net.ipv4.tcp_keepalive_intvl` | 1 - 75 | 75 | Frecuencia con la que se envían los sondeos. Multiplicado por `tcp_keepalive_probes` constituye el tiempo para terminar una conexión que no responde, después de que se inicien los sondeos. | 
| `net.ipv4.tcp_tw_reuse` | 0 o 1 | 0 | Permite reutilizar sockets `TIME-WAIT` con nuevas conexiones cuando sea seguro desde el punto de vista del protocolo. | 
| `net.ipv4.ip_local_port_range` | Primero: 1024 - 60999 y Último: 32768 - 65000] | Primero: 32768 y Último: 60999 | El intervalo de puertos local que utiliza el tráfico TCP y UDP para elegir el puerto local. Formado por dos números: el primer número es el primer puerto local que permite tráfico TCP y UDP en el nodo de agente, el segundo es el último número de puerto local. | 
| `net.ipv4.neigh.default.gc_thresh1`|  128 - 80000 | 4096 | Número mínimo de entradas que pueden encontrarse en la caché de ARP. La recolección de elementos no utilizados no se desencadena si el número de entradas está por debajo de este valor. | 
| `net.ipv4.neigh.default.gc_thresh2`|  512 - 90000 | 8192 | Número máximo temporal de entradas que pueden encontrarse en la caché de ARP. Esta configuración es posiblemente la más importante, ya que la recolección de elementos no utilizados ARP se desencadenará unos 5 segundos después de alcanzar este máximo. |
| `net.ipv4.neigh.default.gc_thresh3`|  1024 - 100000 | 16384 | Número máximo fijo de entradas en la caché de ARP. |
| `net.netfilter.nf_conntrack_max` | 131072 - 589824 | 131 072 | `nf_conntrack` es un módulo que realiza un seguimiento de las entradas de conexión de NAT en Linux. El módulo `nf_conntrack` usa una tabla hash para anotar el registro de la *conexión establecida* del protocolo TCP. `nf_conntrack_max` es el número máximo de nodos de la tabla hash, es decir, el número máximo de conexiones admitidas por el módulo `nf_conntrack` o el tamaño de la tabla de seguimiento de la conexión. | 
| `net.netfilter.nf_conntrack_buckets` | 65536 - 147456 | 65536 | `nf_conntrack` es un módulo que realiza un seguimiento de las entradas de conexión de NAT en Linux. El módulo `nf_conntrack` usa una tabla hash para anotar el registro de la *conexión establecida* del protocolo TCP. `nf_conntrack_buckets` es el tamaño de la tabla hash. | 

#### <a name="worker-limits"></a>Límites de trabajo

Al igual que los límites del descriptor de archivos, el número de trabajos o subprocesos que un proceso puede crear está limitado por la configuración del kernel y los límites de usuarios. El límite de usuarios en AKS es ilimitado. 

| Configuración | Valores o intervalo permitidos | Default | Descripción |
| ------- | ----------------------- | ------- | ----------- |
| `kernel.threads-max` | 20 - 513785 | 55601 | Los procesos pueden iniciar los subprocesos de trabajo. El número máximo de subprocesos que se pueden crear se establece con la configuración del kernel `kernel.threads-max`. | 

#### <a name="virtual-memory"></a>Memoria virtual

La configuración siguiente se puede usar para optimizar el funcionamiento del subsistema de memoria virtual (VM) del kernel de Linux y el valor de `writeout` de datos sucios en el disco.

| Configuración | Valores o intervalo permitidos | Default | Descripción |
| ------- | ----------------------- | ------- | ----------- |
| `vm.max_map_count` |  65530 - 262144 | 65530 | Este archivo contiene el número máximo de áreas de asignación de memoria que puede tener un proceso. Las áreas de asignación de memoria se usan como efectos secundarios de la llamada a `malloc`, directamente mediante `mmap`, `mprotect` y `madvise`, y también cuando se cargan bibliotecas compartidas. | 
| `vm.vfs_cache_pressure` | 1 - 500 | 100 | Este valor de porcentaje controla la tendencia del kernel a reclamar la memoria, que se usa para el almacenamiento en caché de objetos de directorio e inode. |
| `vm.swappiness` | 0 - 100 | 60 | Este control se usa para definir la intensidad con la que el kernel cambiará las páginas de memoria. Los valores más altos aumentarán la intensidad y los valores más bajos reducirán la cantidad de intercambio. Un valor de 0 indica al kernel que no inicie el intercambio hasta que la cantidad de páginas libres y de copia de seguridad de archivos sea menor que la marca de límite superior de una zona. | 
| `swapFileSizeMB` | 1 MB: tamaño del [disco temporal ](../virtual-machines/managed-disks-overview.md#temporary-disk) (/dev/sdb) | None | SwapFileSizeMB especifica que se creará un tamaño en MB de un archivo de intercambio en los nodos de agente de este grupo de nodos. | 
| `transparentHugePageEnabled` | `always`, `madvise`, `never` | `always` | [Transparent Hugepages](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html#admin-guide-transhuge) es una característica del kernel de Linux diseñada para mejorar el rendimiento al hacer un uso más eficaz del hardware de asignación de memoria del procesador. Cuando está habilitada, el kernel intenta asignar elementos `hugepages` siempre que sea posible y cualquier proceso de Linux recibirá páginas de 2 MB si la región `mmap` tiene una alineación natural de 2 MB. En determinados casos, cuando están habilitados elementos `hugepages` en todo el sistema, las aplicaciones pueden acabar asignando más recursos de memoria. Una aplicación puede usar `mmap` con una región grande, pero solo tocar 1 byte de ella, en cuyo caso, es posible que se asigne una página de 2 MB en lugar de una página de 4k por falta de una buena razón para hacerlo. Este caso es el motivo por el que es posible deshabilitar elementos `hugepages` en todo el sistema o solo tenerlos dentro de regiones `MADV_HUGEPAGE madvise`. | 
| `transparentHugePageDefrag` | `always`, `defer`, `defer+madvise`, `madvise`, `never` | `madvise` | Este valor controla si el kernel debe hacer un uso intensivo de la compactación de memoria para que haya más elementos `hugepages` disponibles. | 

> [!IMPORTANT]
> Para facilitar la búsqueda y la legibilidad, la configuración del sistema operativo se muestra en este documento por su nombre, pero debe agregarse al archivo JSON de configuración o a la API AKS mediante la [convención de uso de mayúsculas camelCase](/dotnet/standard/design-guidelines/capitalization-conventions).

Cree un archivo `kubeletconfig.json` con el siguiente contenido:

```json
{
 "cpuManagerPolicy": "static",
 "cpuCfsQuota": true,
 "cpuCfsQuotaPeriod": "200ms",
 "imageGcHighThreshold": 90,
 "imageGcLowThreshold": 70,
 "topologyManagerPolicy": "best-effort",
 "allowedUnsafeSysctls": [
  "kernel.msg*",
  "net.*"
],
 "failSwapOn": false
}
```
Cree un archivo `linuxosconfig.json` con el siguiente contenido:

```json
{
 "transparentHugePageEnabled": "madvise",
 "transparentHugePageDefrag": "defer+madvise",
 "swapFileSizeMB": 1500,
 "sysctls": {
  "netCoreSomaxconn": 163849,
  "netIpv4TcpTwReuse": true,
  "netIpv4IpLocalPortRange": "32000 60000"
 }
}
```

Cree un clúster que especifique las configuraciones de kublet y del sistema operativo con los archivos JSON creados en el paso anterior. 

> [!NOTE]
> Al crear un clúster, puede especificar la configuración de kubelet, la configuración del sistema operativo, o ambas. Si especifica una configuración al crear un clúster, solo se aplicará esa configuración a los nodos del grupo de nodos inicial. Cualquier valor que no esté configurado en el archivo JSON conservará el valor predeterminado.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json --linux-os-config ./linuxosconfig.json
```

Agregue un nuevo grupo de nodos que especifique los parámetros de Kubelet mediante el archivo JSON que creó.

> [!NOTE]
> Al agregar un grupo de nodos a un clúster existente, puede especificar la configuración de kubelet, la configuración del sistema operativo, o ambas. Si especifica una configuración al agregar un grupo de nodos, solo se aplicará esa configuración a los nodos del grupo de nodos nuevo. Cualquier valor que no esté configurado en el archivo JSON conservará el valor predeterminado.

```azurecli
az aks nodepool add --name mynodepool1 --cluster-name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json
```

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [configurar su clúster de AKS](cluster-configuration.md).
- Aprenda a [actualizar las imágenes de nodo](node-image-upgrade.md) del clúster.
- Consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)](upgrade-cluster.md) para más información sobre cómo actualizar el clúster a la versión más reciente de Kubernetes.
- Consulte la lista de [Preguntas más frecuentes sobre AKS](faq.md) para encontrar respuestas a algunas preguntas comunes sobre AKS.

<!-- LINKS - internal -->
[aks-faq]: faq.md
[aks-faq-node-resource-group]: faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group
[aks-multiple-node-pools]: use-multiple-node-pools.md
[aks-scale-apps]: tutorial-kubernetes-scale.md
[aks-support-policies]: support-policies.md
[aks-upgrade]: upgrade-cluster.md
[aks-view-master-logs]: ./view-control-plane-logs.md#enable-resource-logs
[autoscaler-profile-properties]: #using-the-autoscaler-profile
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-update]: /cli/azure/aks#az_aks_update
[az-aks-scale]: /cli/azure/aks#az_aks_scale
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[upgrade-cluster]: upgrade-cluster.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade


<!-- LINKS - external -->
[az-aks-update-preview]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[az-aks-nodepool-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview#enable-cluster-auto-scaler-for-a-node-pool
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
[kubernetes-faq]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#ca-doesnt-work-but-it-used-to-work-yesterday-why