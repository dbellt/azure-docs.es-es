---
title: Procedimientos recomendados para la continuidad empresarial y recuperación ante desastres en AKS
description: Conozca los procedimientos recomendados del operador de un clúster para conseguir el máximo tiempo de actividad de las aplicaciones, ofreciendo una alta disponibilidad y preparándose para la recuperación ante desastres en Azure Kubernetes Service (AKS).
services: container-service
author: lastcoolnameleft
ms.topic: conceptual
ms.date: 03/11/2021
ms.author: thfalgou
ms.custom: fasttrack-edit
ms.openlocfilehash: 5bcd30c22132bc53ff28fdefcb73f686e08e34ea
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105092"
---
# <a name="best-practices-for-business-continuity-and-disaster-recovery-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), el tiempo de actividad de la aplicación pasa a ser importante. De forma predeterminada, AKS proporciona alta disponibilidad mediante el uso de varios nodos en un [conjunto de escalado de máquinas virtuales (VMSS)](../virtual-machine-scale-sets/overview.md). Pero estos múltiples nodos no protegen al sistema frente a un error de la región. Para maximizar el tiempo de actividad, planee con antelación para mantener la continuidad empresarial y preparar la recuperación ante desastres.

Este artículo se centra en cómo planear la continuidad empresarial y la recuperación ante desastres en AKS. Aprenderá a:

> [!div class="checklist"]
> * Planear los clústeres de AKS en varias regiones.
> * Enrutar el tráfico entre varios clústeres con Azure Traffic Manager.
> * Usar la replicación geográfica para los registros de imágenes de contenedor.
> * Planear el estado de la aplicación entre varios clústeres.
> * Replicar el almacenamiento entre varias regiones.

## <a name="plan-for-multiregion-deployment"></a>Planeamiento de la implementación en varias regiones

> **Procedimiento recomendado**
>
> Al implementar varios clústeres de AKS, elija las regiones en las que AKS está disponible. Use regiones emparejadas.

Un clúster de AKS se implementa en una sola región. Para proteger el sistema frente al error en una región, implemente la aplicación en varios clústeres de AKS en diferentes regiones. Cuando planee dónde implementar el clúster de AKS, tenga en cuenta:

* [**Disponibilidad por región de AKS**](./quotas-skus-regions.md#region-availability)
    * Elija regiones cerca de los usuarios. 
    * AKS se expande continuamente en nuevas regiones.
* [**Regiones emparejadas de Azure**](../best-practices-availability-paired-regions.md)
    * Para su área geográfica, elija dos regiones emparejadas entre sí.
    * Las regiones emparejadas coordinan las actualizaciones de la plataforma y dan prioridad a los esfuerzos de recuperación cuando resulta necesario.
* **Disponibilidad del servicio**
    * Decida si las regiones emparejadas deben ser las del nivel de acceso frecuente/frecuente, frecuente/normal o frecuente/poco frecuente.
    * ¿Desea ejecutar ambas regiones al mismo tiempo, con una región *preparada* para empezar a atender el tráfico? O bien,
    * ¿Desea dar a una región tiempo para prepararse para atender el tráfico?

La disponibilidad por región de AKS y las regiones emparejadas son una consideración conjunta. Implemente los clústeres de AKS en regiones emparejadas diseñadas para administrar la recuperación ante desastres de las regiones de forma conjunta. Por ejemplo, AKS está disponible en el Este de EE. UU. y en el Oeste de EE. UU. Estas regiones están emparejadas. Elija estas dos regiones al crear una estrategia de continuidad empresarial y recuperación ante desastres en AKS.

Al implementar la aplicación, agregue otro paso a la canalización de CI/CD para implementar en estos varios clústeres de AKS. La actualización de las canalizaciones de implementación impide que las aplicaciones se implementen solo en una de las regiones y clústeres de AKS. En ese escenario, el tráfico de cliente que se dirige a una región secundaria no recibirá las actualizaciones más recientes de código.

## <a name="use-azure-traffic-manager-to-route-traffic"></a>Uso de Azure Traffic Manager para enrutar el tráfico

> **Procedimiento recomendado**
>
> Azure Traffic Manager puede dirigirlo a la instancia de la aplicación y los clústeres de AKS más cercanos. Para conseguir los máximos niveles de rendimiento y redundancia, dirija todo el tráfico de la aplicación con Traffic Manager antes de que pase al clúster de AKS.

Si tiene varios clústeres de AKS en distintas regiones, use Traffic Manager para controlar el flujo de tráfico a las aplicaciones que se ejecutan en cada clúster. [Azure Traffic Manager](../traffic-manager/index.yml) es un equilibrador de carga de tráfico basado en DNS que puede distribuir el tráfico de red entre regiones. Use Traffic Manager para enrutar a los usuarios en función del tiempo de respuesta del clúster o de la ubicación geográfica.

![AKS con Traffic Manager](media/operator-best-practices-bc-dr/aks-azure-traffic-manager.png)

Si tiene un único clúster de AKS, normalmente lo conecta a la dirección IP de servicio o al nombre DNS de una aplicación determinada. En una implementación de varios clústeres, debe conectarse a un nombre DNS de Traffic Manager que apunte a los servicios en cada clúster de AKS. Para definir estos servicios, use los puntos de conexión de Traffic Manager. Cada punto de conexión es la *dirección IP del equilibrador de carga del servicio*. Use esta configuración para dirigir el tráfico de red desde el punto de conexión de Traffic Manager de una región al punto de conexión de otra.

![Enrutamiento geográfico con Traffic Manager](media/operator-best-practices-bc-dr/traffic-manager-geographic-routing.png)

Traffic Manager realiza las búsquedas de DNS y devuelve el punto de conexión más adecuado. Los perfiles anidados pueden dar prioridad a una ubicación principal. Por ejemplo, debería conectarse a su región geográfica más cercana. Si esa región tiene un problema, Traffic Manager lo dirige a una región secundaria. Este enfoque garantiza que pueda conectarse a una instancia de una aplicación, incluso aunque su región geográfica más próxima no esté disponible.

Para información sobre cómo configurar estos puntos de conexión y el enrutamiento, consulte [Configuración del método de enrutamiento de tráfico geográfico con Traffic Manager](../traffic-manager/traffic-manager-configure-geographic-routing-method.md).

### <a name="application-routing-with-azure-front-door-service"></a>Enrutamiento de aplicaciones con Azure Front Door Service

Mediante el protocolo de difusión por proximidad basado en división TCP, [Azure Front Door Service](../frontdoor/front-door-overview.md) conecta inmediatamente a los usuarios finales con el punto de presencia (POP) de Front Door más cercano. Más características de Azure Front Door Azure:
* Finalización de TLS
* Dominio personalizado
* Firewall de aplicaciones web
* Reescritura de direcciones URL
* Afinidad de sesión 

Revise los requisitos del tráfico de su aplicación para saber qué solución es la más conveniente.

### <a name="interconnect-regions-with-global-virtual-network-peering"></a>Interconexión de regiones con el emparejamiento de redes virtuales globales

Conecte ambas redes virtuales entre sí a través del [emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md) para habilitar la comunicación entre clústeres. El emparejamiento de red virtual interconecta las redes virtuales, lo que ofrece un alto ancho de banda en toda la red troncal de Microsoft, incluso en distintas regiones geográficas.

Antes de emparejar redes virtuales con clústeres de AKS en ejecución, use la instancia de Load Balancer estándar en el clúster de AKS. Este requisito previo permite comunicarse con los servicios de Kubernetes a través del emparejamiento de red virtual.

## <a name="enable-geo-replication-for-container-images"></a>Habilitar la replicación geográfica para las imágenes de contenedor

> **Procedimiento recomendado**
> 
> almacene las imágenes de contenedor en Azure Container Registry y replique geográficamente el registro en cada región de AKS.

Para implementar y ejecutar sus aplicaciones en AKS, necesita una manera de almacenar y extraer las imágenes de contenedor. Container Registry se integra con AKS de modo que puede almacenar de forma segura sus imágenes de contenedor o los gráficos de Helm. Container Registry admite la replicación geográfica de arquitectura multimaestro para replicar automáticamente las imágenes en las regiones de Azure de todo el mundo. 

Para mejorar el rendimiento y la disponibilidad:
1. Use la replicación geográfica de Container Registry para crear un registro en cada región donde haya un clúster de AKS. 
1. Cada clúster de AKS, a continuación, extrae las imágenes del registro de contenedor local en la misma región:

![Replicación geográfica de Container Registry para imágenes de contenedor](media/operator-best-practices-bc-dr/acr-geo-replication.png)

Cuando se usa la replicación geográfica de Container Registry para extraer imágenes de la misma región, los resultados son:

* **Más rápidos**: puede extraer imágenes de conexiones de red de alta velocidad y baja latencia dentro de la misma región de Azure.
* **Más confiables**: si una región no está disponible, el clúster de AKS extrae las imágenes desde un registro de contenedor disponible.
* **Más baratos**: no se genera ningún cargo por salida de red entre los centros de datos.

La replicación geográfica es una característica de los registros de contenedor de SKU *Prémium*. Para información sobre cómo configurar la replicación geográfica, consulte [Replicación geográfica en Azure Container Registry](../container-registry/container-registry-geo-replication.md).

## <a name="remove-service-state-from-inside-containers"></a>Quitar el estado de servicio de dentro de los contenedores

> **Procedimiento recomendado**
> 
> Evite almacenar el estado del servicio dentro del contenedor. En su lugar, use una plataforma como servicio (PaaS) de Azure que admita la replicación en varias regiones.

*Estado de servicio* hace referencia a los datos en memoria o en disco necesarios para que el servicio funcione. El estado incluye las variables de miembro y las estructuras de datos que el servicio lee y escribe. Según el diseño del servicio, el estado también puede incluir archivos u otros recursos almacenados en el disco. Por ejemplo, el estado podría incluir los archivos que una base de datos usa para almacenar los registros de transacciones y de datos.

El estado se puede externalizar o colocar con el código que lo está manipulando. Generalmente, la externalización del estado se realiza con una base de datos u otro almacén de datos que se ejecuta en distintas máquinas de la red o fuera del proceso en la misma máquina.

Los contenedores y los microservicios son más resistentes si los procesos que se ejecutan dentro de ellos no conservan el estado. Ya que las aplicaciones casi siempre contienen algún estado, use una solución PaaS, como:
* Azure Cosmos DB
* Azure Database for PostgreSQL
* Azure Database for MySQL
* Azure SQL Database

Para crear aplicaciones portátiles, consulte las siguientes directrices:

* [La metodología de las aplicaciones con doce factores](https://12factor.net/)
* [Ejecución de una aplicación web en varias regiones de Azure](/azure/architecture/reference-architectures/app-service-web-app/multi-region)

## <a name="create-a-storage-migration-plan"></a>Crear un plan de migración de almacenamiento

> **Procedimiento recomendado**
>
> si usa Azure Storage, prepárelo y pruebe cómo migrar el almacenamiento desde la región principal a la región de copia de seguridad.

Las aplicaciones pueden utilizar Azure Storage para sus datos. En caso afirmativo, las aplicaciones se extienden entre varios clústeres de AKS en distintas regiones. Debe mantener sincronizado el almacenamiento. Existen dos formas habituales de replicar el almacenamiento:

* Replicación asincrónica basada en la infraestructura
* Replicación asincrónica basada en la aplicación

### <a name="infrastructure-based-asynchronous-replication"></a>Replicación asincrónica basada en la infraestructura

Las aplicaciones pueden requerir almacenamiento persistente, incluso después de eliminar un pod. En Kubernetes, puede usar los volúmenes persistentes para conservar el almacenamiento de datos. Los volúmenes persistentes se montan en la máquina virtual de un nodo y, a continuación, se exponen a los pods. Los volúmenes persistentes realizan el seguimiento de los pods, aunque estos se muevan a un nodo diferente dentro del mismo clúster.

La estrategia de replicación que use dependerá de la solución de almacenamiento. Las siguientes soluciones de almacenamiento comunes proporcionan sus propias instrucciones sobre la recuperación ante desastres y la replicación:
* [Gluster](https://docs.gluster.org/en/latest/Administrator-Guide/Geo-Replication/)
* [Ceph](https://docs.ceph.com/docs/master/cephfs/disaster-recovery/)
* [Rook](https://rook.io/docs/rook/v1.2/ceph-disaster-recovery.html)
* [Portworx](https://docs.portworx.com/scheduler/kubernetes/going-production-with-k8s.html#disaster-recovery-with-cloudsnaps) 

Normalmente, se proporciona un punto de almacenamiento común en el que las aplicaciones escriben sus datos. A continuación, estos datos se replican entre regiones y se accede a ellos de manera local.

![Replicación asincrónica basada en la infraestructura](media/operator-best-practices-bc-dr/aks-infra-based-async-repl.png)

Si usa Azure Managed Disks, puede emplear [Velero en Azure][velero] y [Kasten][kasten] para controlar la replicación y la recuperación ante desastres. Estas opciones son soluciones de copia de seguridad nativas de Kubernetes, pero no reciben soporte de dicha plataforma.

### <a name="application-based-asynchronous-replication"></a>Replicación asincrónica basada en la aplicación

Actualmente, Kubernetes no proporciona ninguna implementación para la replicación asincrónica basada en la aplicación. Ya que los contenedores y Kubernetes apenas tienen conexión, cualquier enfoque de lenguaje o aplicación tradicional debe funcionar. Generalmente, las propias aplicaciones replican las solicitudes de almacenamiento, que posteriormente se escriben en el almacenamiento de datos subyacente de cada clúster.

![Replicación asincrónica basada en la aplicación](media/operator-best-practices-bc-dr/aks-app-based-async-repl.png)

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en las consideraciones sobre continuidad empresarial y recuperación ante desastres en los clústeres de AKS. Para más información acerca de las operaciones de clúster en AKS, consulte en los siguientes artículos los procedimientos recomendados:

* [Aislamiento multiempresa y de clúster][aks-best-practices-cluster-isolation]
* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]

<!-- INTERNAL LINKS -->
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md

[velero]: https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md
[kasten]: https://www.kasten.io/