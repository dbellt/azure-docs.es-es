---
title: Migrar a Azure Kubernetes Service (AKS)
description: Migrar a Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/25/2021
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 123f4e5c2442b913a53288602d1c56f199b131a6
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107872792"
---
# <a name="migrate-to-azure-kubernetes-service-aks"></a>Migrar a Azure Kubernetes Service (AKS)

Para ayudarle a planear y ejecutar una migración correcta a Azure Kubernetes Service (AKS), en esta guía se proporcionan detalles para la configuración de AKS que se recomienda actualmente. Aunque en este artículo no se tratan todos los escenarios, se incluyen vínculos a información más detallada para planear una migración correcta.

Este documento ayuda con los siguientes escenarios:

* Incluir en contenedores ciertas aplicaciones y migrarlas a AKS con [Azure Migrate](../migrate/migrate-services-overview.md).
* Migrar un clúster de AKS respaldado por [Conjuntos de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) a [Virtual Machine Scale Sets](../virtual-machine-scale-sets/overview.md).
* Migrar un clúster de AKS para usar un [equilibrador de carga de SKU estándar](./load-balancer-standard.md).
* Migrar desde [Azure Container Service (ACS), que se retirará el 31 de enero de 2020](https://azure.microsoft.com/updates/azure-container-service-will-retire-on-january-31-2020/), a AKS.
* Migrar desde el [motor de AKS](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) a AKS.
* Migrar desde clústeres de Kubernetes no basados en Azure a AKS.
* Trasladar recursos existentes a otra región.

Al realizar la migración, asegúrese de que la versión de Kubernetes de destino se encuentre en la ventana admitida para AKS. Es posible que las versiones anteriores no se encuentren dentro del intervalo admitido y que AKS requiera una actualización de versión. Para más información, consulte [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)](./supported-kubernetes-versions.md).

Si va a migrar a una versión más reciente de Kubernetes, consulte [Versión de Kubernetes y directiva de soporte de asimetría de versiones](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-versions).

Varias herramientas de código abierto pueden ayudarle con la migración, en función de su escenario:

* [Velero](https://velero.io/) (requiere Kubernetes 1.7+)
* [Extensión de la CLI de Azure Kube](https://github.com/yaron2/azure-kube-cli)
* [ReShifter](https://github.com/mhausenblas/reshifter)

En este artículo se resumen los detalles de la migración para lo siguiente:

> [!div class="checklist"]
> * Incluir aplicaciones en contenedores a través de Azure Migrate 
> * AKS con Standard Load Balancer y Virtual Machine Scale Sets
> * Servicios de Azure asociados existentes
> * Garantizar cuotas válidas
> * Alta disponibilidad y continuidad empresarial
> * Consideraciones sobre las aplicaciones sin estado
> * Consideraciones sobre las aplicaciones con estado
> * Implementación de la configuración del clúster

## <a name="use-azure-migrate-to-migrate-your-applications-to-aks"></a>Use Azure Migrate para migrar sus aplicaciones a AKS

Azure Migrate ofrece una plataforma unificada para evaluar y migrar a servidores, infraestructura, aplicaciones y datos locales de Azure. En el caso de AKS, puede usar Azure Migrate para las tareas siguientes:

* [Inclusión de aplicaciones ASP.NET en contenedores y migración a AKS](../migrate/tutorial-containerize-aspnet-kubernetes.md)
* [Inclusión de aplicaciones web de Java en contenedores y migración a AKS](../migrate/tutorial-containerize-java-kubernetes.md)

## <a name="aks-with-standard-load-balancer-and-virtual-machine-scale-sets"></a>AKS con Standard Load Balancer y Virtual Machine Scale Sets

AKS es un servicio administrado que ofrece funcionalidades únicas con una sobrecarga de administración menor. Como AKS es un servicio administrado, debe seleccionar un conjunto de [regiones](./quotas-skus-regions.md) que AKS admita. Quizá deba modificar las aplicaciones existentes para mantenerlas en buen estado en el plano de control administrado por AKS durante la transición del clúster existente a AKS.

Se recomienda el uso de clústeres de AKS respaldados por [Virtual Machine Scale Sets](../virtual-machine-scale-sets/index.yml) y [Azure Standard Load Balancer](./load-balancer-standard.md) para garantizar características como las siguientes:
* [Varios grupos de nodos](./use-multiple-node-pools.md)
* [Zonas de disponibilidad](../availability-zones/az-overview.md)
* [Intervalos IP autorizados](./api-server-authorized-ip-ranges.md)
* [Escalador automático de clústeres](./cluster-autoscaler.md)
* [Azure Policy para AKS](../governance/policy/concepts/policy-for-kubernetes.md)
* Otras características nuevas que vayan saliendo.

Los clústeres de AKS respaldados por [Virtual Machine Availability Sets](../virtual-machines/availability.md#availability-sets) no tienen compatibilidad con muchas de estas características.

En el ejemplo siguiente se crea un clúster de AKS con un único grupo de nodos respaldado por un conjunto de escalado de máquinas virtuales. El clúster:
* Usa una instancia de Standard Load Balancer. 
* Habilita el escalador automático de clústeres en el grupo de nodos del clúster.
* Establece un mínimo de *1* y un máximo de *3* nodos.

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

## <a name="existing-attached-azure-services"></a>Servicios de Azure asociados existentes

Al migrar clústeres quizá haya asociado servicios externos de Azure. Aunque los siguientes servicios no requieren la recreación de los recursos, requerirán la actualización de las conexiones de los clústeres anteriores a los nuevos para mantener la funcionalidad.

* Azure Container Registry
* Log Analytics
* Application Insights
* Traffic Manager
* Cuenta de almacenamiento
* Bases de datos externas

## <a name="ensure-valid-quotas"></a>Garantizar cuotas válidas

Dado que otras máquinas virtuales se implementarán en su suscripción durante la migración, debe comprobar que los límites y las cuotas sean suficientes para estos recursos. Si es necesario, solicite un aumento de la [cuota de CPU virtual](../azure-portal/supportability/per-vm-quota-requests.md).

Es posible que tenga que solicitar un aumento de las [cuotas de red](../azure-portal/supportability/networking-quota-requests.md) para asegurarse de no agotar las direcciones IP. Para más información, consulte [Redes e intervalos IP para AKS](./configure-kubenet.md).

Para más información, consulte [Azure subscription and service limits](../azure-resource-manager/management/azure-subscription-service-limits.md) (Límites de suscripción y servicio de Azure). Para comprobar las cuotas actuales, en Azure Portal, vaya a la [hoja de suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade), seleccione la suscripción y seleccione **Uso y cuotas**.

## <a name="high-availability-and-business-continuity"></a>Alta disponibilidad y continuidad empresarial

Si la aplicación no puede controlar el tiempo de inactividad, debe seguir los procedimientos recomendados para los escenarios de migración de alta disponibilidad. Más información sobre los [Procedimientos recomendados para la planeación de la continuidad empresarial compleja, la recuperación ante desastres y la maximización del tiempo de actividad en Azure Kubernetes Service (AKS)](./operator-best-practices-multi-region.md).

Normalmente, las aplicaciones complejas se migran en varias veces en lugar de en una sola vez,por lo que los entornos antiguos y los nuevos quizá deban comunicarse a través de la red. Es posible que aplicaciones que anteriormente usaban servicios `ClusterIP` para comunicarse deban exponerse como tipo `LoadBalancer` y estar protegidas adecuadamente.

Para completar la migración, querrá dirigir los clientes a los nuevos servicios que se ejecutan en AKS. Se recomienda redirigir el tráfico mediante la actualización de DNS para que apunte a la instancia de Load Balancer que se encuentra frente al clúster de AKS.

[Azure Traffic Manager](../traffic-manager/index.yml) puede dirigir a los clientes a la instancia de la aplicación y los clústeres de Kubernetes deseados. Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que puede distribuir el tráfico de red entre regiones. Para conseguir los máximos niveles de rendimiento y redundancia, dirija todo el tráfico de la aplicación con Traffic Manager antes de que pase al clúster de AKS. 

En una implementación de varios clústeres, los clientes deben conectarse a un nombre DNS de Traffic Manager que apunte a los servicios en cada clúster de AKS. Para definir estos servicios, use los puntos de conexión de Traffic Manager. Cada punto de conexión es la *dirección IP del equilibrador de carga del servicio*. Use esta configuración para dirigir el tráfico de red desde el punto de conexión de Traffic Manager de una región al punto de conexión de otra.

![AKS con Traffic Manager](media/operator-best-practices-bc-dr/aks-azure-traffic-manager.png)

[Azure Front Door Service](../frontdoor/front-door-overview.md) es otra opción para enrutar el tráfico de los clústeres de AKS. Con Azure Front Door Service podrá definir, administrar y supervisar el enrutamiento global para el tráfico web mediante la optimización para obtener mejor rendimiento y conmutación por error global instantánea para la alta disponibilidad. 

### <a name="considerations-for-stateless-applications"></a>Consideraciones sobre las aplicaciones sin estado

La migración de aplicaciones sin estado es el caso más sencillo:
1. Aplique las definiciones de recurso (YAML o Helm) al nuevo clúster.
1. Asegúrese de que todo funciona según lo previsto.
1. Redirija el tráfico para activar el nuevo clúster.

### <a name="considerations-for-stateful-applications"></a>Consideraciones sobre las aplicaciones con estado

Planee meticulosamente la migración de aplicaciones con estado para evitar la pérdida de datos o tiempos de inactividad inesperados.

* Si usa Azure Files, puede montar el recurso compartido de archivos como un volumen en el nuevo clúster. Consulte [Montaje de instancias estáticas de Azure Files como volumen](./azure-files-volume.md#mount-file-share-as-an-persistent-volume).
* Si usa Azure Managed Disks, solo podrá montar el disco si no está conectado a ninguna máquina virtual. Consulte [Montaje de disco estático de Azure como volumen](./azure-disk-volume.md#mount-disk-as-volume).
* Si ninguno de estos métodos funciona, puede usar las opciones de copia de seguridad y restauración. Consulte [Velero en Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md).

#### <a name="azure-files"></a>Archivos de Azure

A diferencia de los discos, Azure Files puede montarse simultáneamente en varios hosts. En el clúster de AKS, Azure y Kubernetes no impiden crear un pod que el clúster de AKS siga usando. Para evitar la pérdida de datos y un comportamiento inesperado, asegúrese de que los clústeres no escriben en los mismos archivos simultáneamente.

Si la aplicación puede hospedar varias réplicas que apunten al mismo recurso compartido de archivos, siga los pasos de la migración sin estado e implemente las definiciones de YAML en el nuevo clúster. 

Si no es así, un posible enfoque de migración implica los pasos siguientes:

1. Compruebe que la aplicación funciona correctamente.
1. Apunte el tráfico en directo al nuevo clúster de AKS.
1. Desconecte el clúster anterior.

Si quiere comenzar con un recurso compartido vacío y luego realizar una copia de los datos de origen, puede usar los comandos [`az storage file copy`](/cli/azure/storage/file/copy) para migrar los datos.


#### <a name="migrating-persistent-volumes"></a>Migración de volúmenes persistentes

Si migra volúmenes persistentes existentes a AKS, generalmente seguirá estos pasos:

1. Desactivar las escrituras en la aplicación. 
    * Este paso es opcional y requiere tiempo de inactividad.
1. Tomar instantáneas de los discos.
1. Crear discos administrados nuevos a partir de las instantáneas.
1. Crear volúmenes persistentes en AKS.
1. Actualizar las especificaciones de pod para [usar volúmenes existentes](./azure-disk-volume.md) en lugar de PersistentVolumeClaims (aprovisionamiento estático).
1. Implemente la aplicación en AKS.
1. Compruebe que la aplicación funciona correctamente.
1. Apunte el tráfico en directo al nuevo clúster de AKS.

> [!IMPORTANT]
> Si decide no desactivar las escrituras, deberá replicar los datos en la nueva implementación. En caso contrario, perderá los datos que se escriban después de tomar las instantáneas de disco.

Algunas herramientas de código abierto pueden ayudarle a crear discos administrados y a migrar volúmenes entre clústeres de Kubernetes:

* La [extensión de copia de disco de la CLI de Azure](https://github.com/noelbundick/azure-cli-disk-copy-extension) copia y convierte los discos entre grupos de recursos y regiones de Azure.
* La [extensión de la CLI de Azure Kube](https://github.com/yaron2/azure-kube-cli) enumera los volúmenes de ACS Kubernetes y los migra a un clúster de AKS.


### <a name="deployment-of-your-cluster-configuration"></a>Implementación de la configuración del clúster

Se recomienda usar la canalización existente de integración continua (CI) y entrega continua (CD) para implementar una configuración válida conocida en AKS. Puede usar Azure Pipelines para [compilar e implementar las aplicaciones en AKS](/azure/devops/pipelines/ecosystems/kubernetes/aks-template). Clone las tareas de implementación existentes y asegúrese de que `kubeconfig` apunta al nuevo clúster de AKS.

Si no es posible, exporte las definiciones de recursos desde el clúster de Kubernetes existente y luego aplíquelas a AKS. Puede usar `kubectl` para exportar objetos.

```console
kubectl get deployment -o=yaml --export > deployments.yaml
```

### <a name="moving-existing-resources-to-another-region"></a>Movimiento de recursos existentes a otra región

Puede que quiera trasladar el clúster de AKS a [otra región compatible con AKS][region-availability]. Se recomienda crear un nuevo clúster en la otra región e implementar los recursos y las aplicaciones en el nuevo clúster. 

Además, si tiene servicios como [Azure Dev Spaces][azure-dev-spaces] en ejecución en el clúster de AKS, deberá instalar y configurar esos servicios en el clúster en la nueva región.


En este artículo se proporciona un resumen de los detalles de la migración para:

> [!div class="checklist"]
> * AKS con Standard Load Balancer y Virtual Machine Scale Sets
> * Servicios de Azure asociados existentes
> * Garantizar cuotas válidas
> * Alta disponibilidad y continuidad empresarial
> * Consideraciones sobre las aplicaciones sin estado
> * Consideraciones sobre las aplicaciones con estado
> * Implementación de la configuración del clúster


[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[azure-dev-spaces]: ../dev-spaces/index.yml
