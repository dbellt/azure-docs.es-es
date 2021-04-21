---
title: Preguntas más frecuentes sobre Azure Kubernetes Service (AKS)
description: Encuentre respuestas a algunas de las preguntas comunes sobre Azure Kubernetes Service (AKS).
ms.topic: conceptual
ms.date: 08/06/2020
ms.custom: references_regions
ms.openlocfilehash: f13d7a33ce1dc04700932072fe0af80a901c681f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783204"
---
# <a name="frequently-asked-questions-about-azure-kubernetes-service-aks"></a>Preguntas más frecuentes sobre Azure Kubernetes Service (AKS)

En este artículo se abordan las preguntas más frecuentes sobre Azure Kubernetes Service (AKS).

## <a name="which-azure-regions-currently-provide-aks"></a>¿Qué regiones de Azure proporcionan actualmente AKS?

Para obtener una lista completa de las regiones disponibles, consulte [AKS regions and availability][aks-regions] (Regiones y disponibilidad de AKS).

## <a name="can-i-spread-an-aks-cluster-across-regions"></a>¿Puedo propagar un clúster de AKS entre regiones?

No. Los clústeres de AKS son recursos regionales y no pueden abarcar regiones. Consulte [Procedimientos recomendados de continuidad empresarial y recuperación ante desastres][bcdr-bestpractices] para obtener instrucciones sobre cómo crear una arquitectura que incluya varias regiones.

## <a name="can-i-spread-an-aks-cluster-across-availability-zones"></a>¿Puedo propagar un clúster de AKS entre zonas de disponibilidad?

Sí. Puede implementar un clúster de AKS en una o más [zonas de disponibilidad][availability-zones] de [regiones que las admitan][az-regions].

## <a name="can-i-limit-who-has-access-to-the-kubernetes-api-server"></a>¿Puedo limitar quién tiene acceso al servidor de API de Kubernetes?

Sí. Hay dos opciones para limitar el acceso al servidor de API:

- Use [intervalos de direcciones IP autorizadas del servidor de API][api-server-authorized-ip-ranges] si quiere mantener un punto de conexión público para el servidor de API, pero restringir el acceso a un conjunto de intervalos de direcciones IP de confianza.
- Use [un clúster privado][private-clusters] si quiere limitar el servidor de API para que *solo* sea accesible desde dentro de la red virtual.

## <a name="can-i-have-different-vm-sizes-in-a-single-cluster"></a>¿Puedo tener diferentes tamaños de máquina virtual en un único clúster?

Sí, puede usar diferentes tamaños de máquinas virtuales en el clúster de AKS mediante la creación de [varios grupos de nodos][multi-node-pools].

## <a name="are-security-updates-applied-to-aks-agent-nodes"></a>¿Se aplican las actualizaciones de seguridad a los nodos de agente de AKS?

Azure aplica automáticamente revisiones de seguridad a los nodos de Linux del clúster siguiendo una programación nocturna. Sin embargo, es responsabilidad suya asegurarse de que los nodos de Linux se reinician según sea necesario. Cuenta con varias opciones para reiniciar los nodos:

- Manualmente, mediante Azure Portal o la CLI de Azure.
- Mediante la actualización del clúster de AKS. Las actualizaciones del clúster [acordonan y purgan los nodos][cordon-drain] automáticamente, y luego ponen un nuevo nodo en línea con la imagen de Ubuntu más reciente y una nueva versión de revisión o una versión secundaria de Kubernetes. Para más información, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].
- Mediante el uso de la [actualización de imágenes de nodo](node-image-upgrade.md).

### <a name="windows-server-nodes"></a>Nodos de Windows Server

Para los nodos de Windows Server, Windows Update no ejecuta ni aplica las actualizaciones más recientes de manera automática. En una programación normal del ciclo de versiones de Windows Update y su proceso de validación propio, debe realizar una actualización en el clúster y los grupos de nodos de Windows Server en el clúster de AKS. Este proceso de actualización crea nodos que ejecutan la imagen y las revisiones más recientes de Windows Server y elimina los nodos anteriores. Para obtener más información sobre este proceso, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

## <a name="why-are-two-resource-groups-created-with-aks"></a>¿Por qué se crean dos grupos de recursos con AKS?

AKS se basa en diversos recursos de infraestructura de Azure, como los conjuntos de escalado de máquinas virtuales, las redes virtuales y los discos administrados. Esto le permite utilizar muchas de las funcionalidades básicas de la plataforma de Azure en el entorno de Kubernetes administrado que proporciona AKS. Por ejemplo, la mayoría de los tipos de máquinas virtuales de Azure se pueden usar directamente con AKS, y Azure Reservations se puede usar para recibir descuentos automáticamente en esos recursos.

Para habilitar esta arquitectura, cada implementación de AKS abarca dos grupos de recursos:

1. Debe cree el primer grupo de recursos. Este grupo solo contiene el recurso del servicio de Kubernetes. El proveedor de recursos de AKS crea automáticamente el segundo grupo de recursos durante la implementación. Un ejemplo del segundo grupo de recursos es *MC_myResourceGroup_myAKSCluster_eastus*. Para obtener información sobre cómo especificar el nombre de este segundo grupo de recursos, consulte la sección siguiente.
1. El segundo grupo de recursos, conocido como el *grupo de recursos del nodo*, contiene todos los recursos de infraestructura asociados con el clúster. Estos recursos incluyen las máquinas virtuales de nodos de Kubernetes, las redes virtuales y el almacenamiento. De forma predeterminada, el grupo de recursos del nodo tiene un nombre como *MC_myResourceGroup_myAKSCluster_eastus*. AKS elimina automáticamente el recurso del nodo cada vez que se elimina el clúster, por lo que solo se debe usar para los recursos que comparten el ciclo de vida del clúster.

## <a name="can-i-provide-my-own-name-for-the-aks-node-resource-group"></a>¿Puedo proporcionar mi propio nombre para el grupo de recursos del nodo de AKS?

Sí. De forma predeterminada, AKS asignará el nombre *MC_resourcegroupname_clustername_location* al grupo de recursos del nodo, pero también puede proporcionar su propio nombre.

Para especificar un nombre de su elección para el grupo de recursos, instale la versión de la extensión de la CLI de Azure [aks-preview][aks-preview-cli]*0.3.2* o una posterior. Cuando cree un clúster de AKS mediante el comando [az aks create][az-aks-create], use el parámetro `--node-resource-group` y especifique un nombre para el grupo de recursos. Si [usa una plantilla de Azure Resource Manager][aks-rm-template] para implementar un clúster de AKS, puede definir el nombre del grupo de recursos mediante la propiedad *nodeResourceGroup*.

* El proveedor de recursos de Azure crea automáticamente el grupo de recursos secundario en su propia suscripción.
* Solo puede especificar un nombre personalizado para el grupo de recursos cuando cree el clúster.

Cuando trabaje con el grupo de recursos del nodo, tenga en cuenta que no puede:

* Especificar un grupo de recursos existente para el grupo de recursos del nodo.
* Especificar otra suscripción para el grupo de recursos del nodo.
* Cambiar el nombre del grupo de recursos del nodo una vez se haya creado el clúster.
* Especificar los nombres de los recursos administrados del grupo de recursos del nodo.
* Modificar o eliminar las etiquetas creadas en Azure de los recursos administrados del grupo de recursos del nodo. (Puede ver la información adicional en la sección siguiente).

## <a name="can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group"></a>¿Puedo modificar etiquetas y otras propiedades de los recursos de AKS en el grupo de recursos del nodo?

Si modifica o elimina etiquetas creadas por Azure y otras propiedades de recursos en el grupo de recursos del nodo, es posible que obtenga resultados inesperados, como errores de escalado y actualización. AKS permite crear y modificar etiquetas personalizadas generadas por usuarios finales, que se pueden agregar al [crear un grupo de nodos](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool). Es posible que quiera crear o modificar etiquetas personalizadas, por ejemplo, para asignar un centro de costos o una unidad de negocio. Esto también puede conseguirse creando directivas de Azure cuyo ámbito sea el grupo de recursos administrado.

Sin embargo, la modificación de las **etiquetas creadas por Azure** en los recursos del grupo de recursos del nodo en el clúster de AKS es una acción no admitida que interrumpe el objetivo de nivel de servicio (SLO). Para obtener más información, consulte [¿AKS ofrece un contrato de nivel de servicio?](#does-aks-offer-a-service-level-agreement)

## <a name="what-kubernetes-admission-controllers-does-aks-support-can-admission-controllers-be-added-or-removed"></a>¿Qué controladores de admisión de Kubernetes admite AKS? ¿Se pueden agregar o eliminar los controladores de admisión?

AKS admite los siguientes [controladores de admisión][admission-controllers]:

- *NamespaceLifecycle*
- *LimitRanger*
- *ServiceAccount*
- *DefaultStorageClass*
- *DefaultTolerationSeconds*
- *MutatingAdmissionWebhook*
- *ValidatingAdmissionWebhook*
- *ResourceQuota*
- *PodNodeSelector*
- *PodTolerationRestriction*
- *ExtendedResourceToleration*

Actualmente, no puede modificar la lista de controladores de admisión en AKS.

## <a name="can-i-use-admission-controller-webhooks-on-aks"></a>¿Puedo utilizar webhooks de controlador de admisión en AKS?

Sí, puede usar webhooks de controlador de admisión en AKS. Se recomienda excluir los espacios de nombres internos de AKS que estén marcados con la **etiqueta de plano de control**. Por ejemplo, al agregar lo siguiente a la configuración de webhook:

```
namespaceSelector:
    matchExpressions:
    - key: control-plane
      operator: DoesNotExist
```

AKS pasa por el firewall la salida del servidor de la API, por lo que los webhooks del controlador de admisión deben ser accesibles desde el clúster.

## <a name="can-admission-controller-webhooks-impact-kube-system-and-internal-aks-namespaces"></a>¿Los webhooks de controlador de admisión pueden afectar a los espacios de nombres internos de kube-system y de AKS?

Para proteger la estabilidad del sistema y evitar que los controladores de admisión personalizados afecten a los servicios internos de kube-system, el espacio de nombres de AKS tiene un **ejecutor de admisiones**, que excluye automáticamente los espacios de nombres internos de kube-system y de AKS. Este servicio garantiza que los controladores de admisión personalizados no afecten a los servicios que se ejecutan en kube-system.

Si tiene un caso de uso crítico por tener algo implementado en kube-system (no recomendado) que necesita que esté incluido en el webhook de admisión personalizado, puede agregar la siguiente etiqueta o anotación para que el factor de admisión lo omita.

Etiqueta: ```"admissions.enforcer/disabled": "true"``` o anotación: ```"admissions.enforcer/disabled": true```

## <a name="is-azure-key-vault-integrated-with-aks"></a>¿Azure Key Vault se integra con AKS?

AKS no está integrado de forma nativa con Azure Key Vault en estos momentos. Sin embargo, el [proveedor de Azure Key Vault para el almacén de secretos de CSI][csi-driver] permite realizar una integración directa desde pods de Kubernetes a los secretos de Key Vault.

## <a name="can-i-run-windows-server-containers-on-aks"></a>¿Puedo ejecutar contenedores de Windows Server en AKS?

Sí, los contenedores de Windows Server están disponibles en AKS. Para ejecutar los contenedores de Windows Server en AKS, cree un grupo de nodos que ejecute Windows Server como sistema operativo invitado. Los contenedores de Windows Server solo pueden usar Windows Server 2019. Para empezar, vea [Creación de un clúster de AKS con un grupo de nodos de Windows Server][aks-windows-cli].

La compatibilidad de Windows Server con el grupo de nodos incluye algunas limitaciones que forman parte de la versión de Windows Server ascendente en el proyecto de Kubernetes. Para obtener más información sobre estas limitaciones, consulte [Windows Server containers in AKS limitations][aks-windows-limitations] (Limitaciones de los contenedores de Windows Server en AKS).

## <a name="does-aks-offer-a-service-level-agreement"></a>¿AKS ofrece un Acuerdo de Nivel de Servicio?

AKS proporciona garantías de Acuerdo de Nivel de Servicio como una característica opcional de complemento con [Acuerdo de Nivel de Servicio de tiempo de actividad][uptime-sla]. 

La SKU gratuita que se ofrece de forma predeterminada no tiene ningún *Acuerdo* de Nivel de Servicio asociado, pero tiene un *Objetivo* de Nivel de Servicio del 99,5 %. Podría suceder que aparecieran problemas de conectividad transitorios en el caso de actualizaciones, nodos subyacentes incorrectos, mantenimiento de la plataforma, aplicaciones que saturan el servidor de la API con solicitudes, etc. Si la carga de trabajo no tolera reinicios del servidor de API, se recomienda usar el Acuerdo de nivel de servicio de tiempo de actividad.

## <a name="can-i-apply-azure-reservation-discounts-to-my-aks-agent-nodes"></a>¿Puedo aplicar descuentos de las reservas de Azure en mis nodos de agente de AKS?

Los nodos de agente de AKS se facturan como máquinas virtuales de Azure estándares, por lo que, si ha comprado [reservas Azure][reservation-discounts] para el tamaño de máquina virtual que está usando en AKS, los descuentos se aplican automáticamente.

## <a name="can-i-movemigrate-my-cluster-between-azure-tenants"></a>¿Puedo mover o migrar mi clúster entre inquilinos de Azure?

Actualmente, no puede mover el clúster de AKS entre inquilinos.

## <a name="can-i-movemigrate-my-cluster-between-subscriptions"></a>¿Puedo mover o migrar mi clúster entre suscripciones?

Actualmente no se admite el movimiento de clústeres entre suscripciones.

## <a name="can-i-move-my-aks-clusters-from-the-current-azure-subscription-to-another"></a>¿Puedo trasladar los clústeres de AKS de la suscripción actual de Azure a otra?

No se admite el traslado de clústeres de AKS y de los recursos asociados entre suscripciones de Azure.

## <a name="can-i-move-my-aks-cluster-or-aks-infrastructure-resources-to-other-resource-groups-or-rename-them"></a>¿Puedo mover el clúster de AKS o los recursos de infraestructura de AKS a otros grupos de recursos o cambiarles el nombre?

No se admite mover el clúster de AKS y sus recursos asociados ni cambiarles el nombre.

## <a name="why-is-my-cluster-delete-taking-so-long"></a>¿Por qué tarda tanto la eliminación del clúster?

La mayoría de los clústeres se eliminan en el momento de la solicitud del usuario; en algunos casos, especialmente en aquellos en los que los clientes traen su propio grupo de recursos o se están realizando tareas entre grupos de recursos, la eliminación puede tardar más tiempo o producir un error. Si tiene un problema con las eliminaciones, compruebe que no tiene bloqueos en el grupo de recursos, que los recursos situados fuera del grupo están desasociados de este, etc.

## <a name="if-i-have-pod--deployments-in-state-nodelost-or-unknown-can-i-still-upgrade-my-cluster"></a>Si tengo un pod o implementaciones con el estado "NodeLost" (Nodo perdido) o "Unknown" (Desconocido), ¿puedo actualizar el clúster?

Sí, pero no se recomienda en AKS. Las actualizaciones deberían realizarse si el estado del clúster es conocido o correcto.

## <a name="if-i-have-a-cluster-with-one-or-more-nodes-in-an-unhealthy-state-or-shut-down-can-i-perform-an-upgrade"></a>Si tengo un clúster con uno o varios nodos en un estado incorrecto o apagado, ¿puedo realizar una actualización?

No, elimine o quite los nodos con un estado erróneo antes de actualizar.

## <a name="i-ran-a-cluster-delete-but-see-the-error-errno-11001-getaddrinfo-failed"></a>Ejecuté una eliminación de clúster, pero me aparece el error `[Errno 11001] getaddrinfo failed`

Normalmente, esto se debe a usuarios que tienen uno o varios grupos de seguridad de red (NSG) todavía en uso y asociados al clúster.  Quítelos e intente de nuevo la eliminación.

## <a name="i-ran-an-upgrade-but-now-my-pods-are-in-crash-loops-and-readiness-probes-fail"></a>Ejecuté una actualización, pero ahora mis pods están en bucles de bloqueo y se producen errores en los sondeos de preparación.

Confirme que la entidad de servicio no ha expirado.  Consulte: [Entidad de servicio de AKS](./kubernetes-service-principal.md) y [Credenciales de actualización de AKS](./update-credentials.md).

## <a name="my-cluster-was-working-but-suddenly-cant-provision-loadbalancers-mount-pvcs-etc"></a>Mi clúster estaba funcionando, pero, de repente, no puede aprovisionar equilibradores de carga, montar PVC, etc.

Confirme que la entidad de servicio no ha expirado.  Consulte: [Entidad de servicio de AKS](./kubernetes-service-principal.md) y [Credenciales de actualización de AKS](./update-credentials.md).

## <a name="can-i-scale-my-aks-cluster-to-zero"></a>¿Puedo escalar el clúster de AKS a cero?
Puede [detener completamente un clúster de AKS en ejecución](start-stop-cluster.md), lo que ahorrará los costos de proceso respectivos. Además, puede optar por [escalar manual o automáticamente todos los grupos de nodos `User` o algunos específicos](scale-cluster.md#scale-user-node-pools-to-0) a 0, manteniendo solo la configuración necesaria del clúster.
No se pueden escalar directamente [grupos de nodos del sistema](use-system-pools.md) a 0.

## <a name="can-i-use-the-virtual-machine-scale-set-apis-to-scale-manually"></a>¿Puedo usar las API del conjunto de escalado de máquinas virtuales para escalar manualmente?

No, no se admiten las operaciones de escalado mediante el uso de las API del conjunto de escalado de máquinas virtuales. Use las API de AKS (`az aks scale`).

## <a name="can-i-use-virtual-machine-scale-sets-to-manually-scale-to-zero-nodes"></a>¿Puedo usar conjuntos de escalado de máquinas virtuales para escalar manualmente a 0 nodos?

No, no se admiten las operaciones de escalado mediante el uso de las API del conjunto de escalado de máquinas virtuales. Puede usar la API de AKS para escalar a 0 grupos de nodos que no sean del sistema, o bien [detener el clúster](start-stop-cluster.md) en su lugar.

## <a name="can-i-stop-or-de-allocate-all-my-vms"></a>¿Puedo detener o cancelar la asignación de todas las máquinas virtuales?

Aunque AKS tiene mecanismos de resistencia para admitir este tipo de configuración y recuperarse de él, no se admite esta configuración. [Detenga el clúster](start-stop-cluster.md) en su lugar.

## <a name="can-i-use-custom-vm-extensions"></a>¿Puedo usar extensiones de máquina virtual personalizadas?

Se admite el agente de Log Analytics porque es una extensión que administra Microsoft. De lo contrario, no, AKS es un servicio administrado y no se admite la manipulación de los recursos de IaaS. Para instalar componentes personalizados, use los mecanismos y las API de Kubernetes. Por ejemplo, aproveche DaemonSets para instalar los componentes necesarios.

## <a name="does-aks-store-any-customer-data-outside-of-the-clusters-region"></a>¿AKS guarda datos de los clientes fuera de la región del clúster?

La característica que permite almacenar los datos de clientes en una única región solo está disponible actualmente en la región de Sudeste Asiático (Singapur) de la geoárea Asia Pacífico y en la región Sur de Brasil (Estado de São Paulo) de la geoárea Brasil. En todas las demás regiones, los datos de clientes se almacenan en la geoárea.

## <a name="are-aks-images-required-to-run-as-root"></a>¿Las imágenes de AKS deben ejecutarse como raíz?

Excepto en el caso de las dos imágenes siguientes, no es necesario que las imágenes de AKS se ejecuten como raíz:

- *mcr.microsoft.com/oss/kubernetes/coredns*
- *mcr.microsoft.com/azuremonitor/containerinsights/ciprod*

## <a name="what-is-azure-cni-transparent-mode-vs-bridge-mode"></a>¿Qué diferencias hay entre el modo transparente de Azure CNI y el modo puente?

A partir de la versión 1.2.0, Azure CNI tendrá habilitado el modo transparente de manera predeterminada para las implementaciones CNI de un solo inquilino en Linux. El modo transparente va a reemplazar al modo puente. En esta sección, indagaremos en las diferencias entre ambos modos y cuáles son las ventajas y limitaciones de usar el modo transparente de Azure CNI.

### <a name="bridge-mode"></a>Modo puente

Tal y como sugiere el nombre, el modo puente de Azure CNI, mediante "just-in-time", creará un puente L2 denominado "azure0". Todas las interfaces de par `veth` de los pods del host se conectarán a este puente. Por lo tanto, la comunicación entre pods de las máquinas virtuales y el tráfico restante pasa a través de este puente. El puente en cuestión es un dispositivo virtual de capa 2 que, por su cuenta, no puede recibir ni transmitir nada, a menos que enlace uno o más dispositivos reales a él. Por este motivo, la interfaz eth0 de la máquina virtual Linux debe convertirse en un puente subordinado a "azure0". Esto crea una topología de red compleja dentro de la máquina virtual Linux y, como desventaja, CNI tenía que encargarse de otras funciones de red, como la actualización del servidor DNS, etc.

:::image type="content" source="media/faq/bridge-mode.png" alt-text="Topología del modo puente":::

A continuación se muestra un ejemplo de cuál es el aspecto de la configuración de la ruta IP en el modo puente. Independientemente del número de pods que tenga el nodo, solo habrá dos rutas. En la primera se indica que todo el tráfico, excluido el local en azure0, irá a la puerta de enlace predeterminada de la subred a través de la interfaz con la dirección IP "src 10.240.0.4" (que es la dirección IP principal del nodo). En la segunda, se indica que el espacio "10.20.x.x" es de pod a kernel, y el kernel toma la decisión.

```bash
default via 10.240.0.1 dev azure0 proto dhcp src 10.240.0.4 metric 100
10.240.0.0/12 dev azure0 proto kernel scope link src 10.240.0.4
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
root@k8s-agentpool1-20465682-1:/#
```

### <a name="transparent-mode"></a>Modo transparente
El modo transparente adopta un enfoque directo para configurar redes de Linux. En este modo, Azure CNI no cambia las propiedades de la interfaz eth0 en la máquina virtual Linux. Este enfoque mínimo para cambiar las propiedades de red de Linux ayuda a reducir los problemas complejos que podrían encontrarse los clústeres en el modo puente. En el modo transparente, Azure CNI creará y agregará interfaces de par `veth` de los pods del host que se agregarán a la red del host. La comunicación entre pods de las máquinas virtuales se realiza a través de las rutas IP que CNI agregará. Básicamente, la comunicación entre pods se realiza a través de la capa 3 y el tráfico de los pods se enruta mediante reglas de enrutamiento L3.

:::image type="content" source="media/faq/transparent-mode.png" alt-text="Topología del modo transparente":::

A continuación se muestra un ejemplo de configuración de la ruta de IP del modo transparente, donde la interfaz de cada pod obtendrá una ruta estática asociada para que el tráfico con la misma dirección IP de destino que el pod se envíe directamente a la interfaz de par `veth` de los pods del host.

```bash
10.240.0.216 dev azv79d05038592 proto static
10.240.0.218 dev azv8184320e2bf proto static
10.240.0.219 dev azvc0339d223b9 proto static
10.240.0.222 dev azv722a6b28449 proto static
10.240.0.223 dev azve7f326f1507 proto static
10.240.0.224 dev azvb3bfccdd75a proto static
168.63.129.16 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
169.254.169.254 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

### <a name="benefits-of-transparent-mode"></a>Ventajas del modo transparente

- Proporciona mitigación para la condición de carrera paralela de DNS `conntrack` y evita los problemas de latencia de DNS de 5 segundos sin necesidad de configurar el DNS local del nodo (puede seguir usándolo por motivos de rendimiento).
- Elimina la latencia inicial de 5 segundos de DNS que el modo puente de CNI causa actualmente debido a la configuración del puente "just in time".
- Uno de los casos extremos del modo puente es que Azure CNI no puede actualizar constantemente las listas de servidores DNS personalizados que los usuarios agregan a la red virtual o la NIC. Como resultado, CNI recoge solo la primera instancia de la lista de servidores DNS. Este problema se resuelve en el modo transparente porque CNI no cambia las propiedades de eth0. Consulte más información [aquí](https://github.com/Azure/azure-container-networking/issues/713).
- Se puede controlar mejor el tráfico UDP y se mitigan los desbordamientos de UDP cuando se agota el tiempo de espera de ARP. En el modo puente, cuando el puente no conoce una dirección MAC del pod de destino en la comunicación entre pods de las máquinas virtuales, de forma predeterminada, se genera una avalancha de paquetes en todos los puertos. Este problema se resuelve en el modo transparente, ya que no hay ningún dispositivo L2 en la ruta de acceso. Consulte más información [aquí](https://github.com/Azure/azure-container-networking/issues/704).
- El modo transparente funciona mejor en la comunicación entre pods de las máquinas virtuales en términos de rendimiento y latencia, en comparación con el modo puente.

## <a name="how-to-avoid-permission-ownership-setting-slow-issues-when-the-volume-has-a-lot-of-files"></a>¿Cómo evitar los problemas de lentitud debido a la configuración de propiedad de permisos cuando el volumen tiene muchos archivos?

Tradicionalmente, si el pod se ejecuta como un usuario no raíz (como debería ser), debe especificar un elemento `fsGroup` dentro del contexto de seguridad del pod para que el pod pueda leer y escribir en el volumen. Este requisito se explica con más detalle [aquí](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

No obstante, un efecto secundario de la configuración `fsGroup` es que, cada vez que se monta un volumen, Kubernetes debe ejecutar `chown()` y `chmod()` en todos los archivos y directorios dentro del volumen, con algunas excepciones que se indican a continuación. Esto sucede incluso si la propiedad del grupo del volumen ya coincide con el `fsGroup` solicitado y puede resultar bastante costoso para volúmenes mayores con muchos archivos pequeños, por lo que el inicio del pod tarda mucho tiempo. Esta situación ha sido un problema conocido antes de la versión v1.20 y la solución alternativa es configurar la ejecución del pod como raíz:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 0
    fsGroup: 0
```

El problema se resolvió con Kubernetes v1.20, consulte [Kubernetes 1.20: Control granular de los cambios de permisos de volumen](https://kubernetes.io/blog/2020/12/14/kubernetes-release-1.20-fsgroupchangepolicy-fsgrouppolicy/) para obtener más detalles.


<!-- LINKS - internal -->

[aks-upgrade]: ./upgrade-cluster.md
[aks-cluster-autoscale]: ./cluster-autoscaler.md
[aks-advanced-networking]: ./configure-azure-cni.md
[aks-rbac-aad]: ./azure-ad-integration-cli.md
[node-updates-kured]: node-updates-kured.md
[aks-preview-cli]: /cli/azure/ext/aks-preview/aks
[az-aks-create]: /cli/azure/aks#az_aks_create
[aks-rm-template]: /azure/templates/microsoft.containerservice/2019-06-01/managedclusters
[aks-cluster-autoscaler]: cluster-autoscaler.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[aks-windows-cli]: windows-container-cli.md
[aks-windows-limitations]: ./windows-faq.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
[api-server-authorized-ip-ranges]: ./api-server-authorized-ip-ranges.md
[multi-node-pools]: ./use-multiple-node-pools.md
[availability-zones]: ./availability-zones.md
[private-clusters]: ./private-clusters.md
[bcdr-bestpractices]: ./operator-best-practices-multi-region.md#plan-for-multiregion-deployment
[availability-zones]: ./availability-zones.md
[az-regions]: ../availability-zones/az-region.md
[uptime-sla]: ./uptime-sla.md

<!-- LINKS - external -->
[aks-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[auto-scaler]: https://github.com/kubernetes/autoscaler
[cordon-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[admission-controllers]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[private-clusters-github-issue]: https://github.com/Azure/AKS/issues/948
[csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
