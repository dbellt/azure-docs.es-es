---
title: 'Conceptos: Ingeniería de software sostenible en Azure Kubernetes Services (AKS)'
description: Obtenga información sobre ingeniería de software sostenible en Azure Kubernetes Services (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/29/2021
ms.openlocfilehash: c43c65dfa2f3930510bd59aaa24c798525bd691b
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107011498"
---
# <a name="sustainable-software-engineering-principles-in-azure-kubernetes-service-aks"></a>Principios de ingeniería de software sostenible en Azure Kubernetes Services (AKS)

Los principios de ingeniería de software sostenible son un conjunto de competencias que le ayudarán a definir, crear y ejecutar aplicaciones sostenibles. El objetivo general es reducir la huella de carbono de cada aspecto de la aplicación. Los [principios de ingeniería de software sostenible][principles-sse] incluyen una visión general de estos principios.

La ingeniería de software sostenible supone un cambio de prioridades y de enfoque. En muchos casos, la forma en que se diseña y ejecuta la mayoría del software prioriza un rendimiento rápido y una latencia baja. Por su parte, la ingeniería de software sostenible se centra en reducir las emisiones de carbono tanto como sea posible. Considere:

* La aplicación de principios de ingeniería de software sostenible puede proporcionar un rendimiento más rápido o una menor latencia, por ejemplo, reduciendo el recorrido total de la red. 
* Reducir las emisiones de carbono puede ralentizar el rendimiento o aumentar la latencia, como el retraso de las cargas de trabajo de prioridad baja. 

Antes de aplicar principios de ingeniería de software sostenible a una aplicación, revise las prioridades, las necesidades y las ventajas e inconvenientes de su aplicación.

## <a name="measure-and-optimize"></a>Medición y optimización

Para reducir la huella de carbono de los clústeres de AKS, necesita comprender cómo se usan los recursos del clúster. [Azure Monitor][azure-monitor] proporciona detalles sobre el uso de recursos del clúster, como pueden ser el uso de memoria y la CPU. Estos datos fundamentan su decisión de reducir la huella de carbono del clúster y reflejan el efecto de los cambios. 

También puede instalar [Microsoft Sustainability Calculator][sustainability-calculator] para ver la huella de carbono de todos los recursos de Azure.

## <a name="increase-resource-utilization"></a>Aumento de la utilización de recursos

Un enfoque para reducir la huella de carbono es reducir el tiempo de inactividad. Reducir el tiempo de inactividad implica aumentar el uso de los recursos de proceso. Por ejemplo:
1. Suponga que tenía cuatro nodos en el clúster, todos funcionando al 50 % de su capacidad. Por tanto, los cuatro nodos tenían un 50 % de capacidad sin usar inactiva. 
1. Decide reducir el clúster a tres nodos, cada uno funcionando al 67 % de capacidad con la misma carga de trabajo. Así lograría reducir la capacidad sin usar en un 33 % en cada nodo e incrementaría su uso.

> [!IMPORTANT]
> Al considerar la posibilidad de cambiar los recursos del clúster, compruebe que los [grupos del sistema][system-pools] tengan suficientes recursos para mantener la estabilidad de los componentes principales del sistema del clúster. No reduzca **nunca** los recursos del clúster hasta el punto en el que el clúster pueda volverse inestable.

Después de revisar el uso del clúster, considere la posibilidad de usar las características que [varios grupos de nodos][multiple-node-pools] ofrecen: 

* Ajuste de tamaño del nodo

    Use el [ajuste de tamaño de nodo][node-sizing] para definir grupos de nodos con perfiles de CPU y de memoria específicos, lo que le permite adaptar los nodos a sus necesidades de carga de trabajo. Al ajustar el tamaño de los nodos a sus necesidades de carga de trabajo, puede ejecutar unos cuantos nodos con un uso mayor. 

* Escalado de clústeres

    Configure el modo en que el clúster [se escala][scale]. Use el [escalador automático de pod horizontal][scale-horizontal] y el [escalador automático de clúster][scale-auto] para escalar el clúster automáticamente en función de la configuración. Controle el modo en que el clúster se escala para mantener todos los nodos en ejecución con un uso elevado y, al mismo tiempo, mantener sincronizados los cambios en la carga de trabajo del clúster. 

* Grupos de acceso puntual

    En los casos en los que una carga de trabajo es tolerante a interrupciones o finalizaciones repentinas, puede usar [grupos de acceso puntual][spot-pools]. Estos grupos aprovechan la capacidad inactiva en Azure. Por ejemplo, los grupos de acceso puntual pueden funcionar bien para los trabajos por lotes o los entornos de desarrollo.

> [!NOTE]
>Aumentar la utilización también puede reducir el exceso de nodos, lo que disminuye la energía consumida por las [reservas de recursos en cada nodo][resource-reservations].

Por último, revise las *solicitudes* y los *límites* de la CPU y la memoria en los manifiestos de Kubernetes de las aplicaciones. 
* A medida que reduce los valores para la memoria y la CPU, hay más memoria y CPU disponibles para el clúster para ejecutar otras cargas de trabajo. 
* A medida que ejecuta más cargas de trabajo con menor CPU y memoria, su clúster se asigna más densamente, lo que aumenta su utilización. 

Al reducir la CPU y la memoria para las aplicaciones, el comportamiento de estas se puede degradar o pasar a ser inestable si establece unos valores para la memoria y la CPU demasiado bajos. Antes de cambiar las *solicitudes* y los *límites* de la CPU y la memoria, ejecute algunas pruebas comparativas para comprobar si los valores están establecidos correctamente. No reduzca nunca estos valores hasta un punto en que la aplicación se vuelva inestable.

## <a name="reduce-network-travel"></a>Reducción del viaje de red

Al reducir la distancia que recorren las solicitudes y respuestas hacia y desde el clúster, puede reducir las emisiones de carbono y el consumo de electricidad de los dispositivos de red. Después de revisar el tráfico de red, considere la posibilidad de crear clústeres [en regiones][regions] más cerca del origen del tráfico de red. Puede usar [Azure Traffic Manager][azure-traffic-manager] para enrutar el tráfico al clúster más cercano y los [grupos con ubicación por proximidad][proiximity-placement-groups] para reducir la distancia entre los recursos de Azure.

> [!IMPORTANT]
> Al considerar la posibilidad de realizar cambios en las redes del clúster, no reduzca nunca el viaje de red a costa de cumplir los requisitos de la carga de trabajo. Por ejemplo, el uso de [zonas de disponibilidad][availability-zones] provoca más viaje de red en el clúster, pero es posible que sean necesarias para controlar los requisitos de la carga de trabajo.

## <a name="demand-shaping"></a>Modelado de la demanda

Siempre que sea posible, considere la posibilidad de cambiar la demanda de los recursos del clúster a las horas o regiones en las que puede usar exceso de capacidad. Por ejemplo, considere lo siguiente:
* Cambiar la hora o la región de ejecución de un trabajo por lotes.
* Usar [grupos de acceso puntual][spot-pools]. 
* Refactorizar la aplicación para usar una cola para diferir las cargas de trabajo en ejecución que no necesitan un procesamiento inmediato.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de las siguientes características de AKS mencionadas en este artículo:

* [Varios grupos de nodos][multiple-node-pools]
* [Ajuste de tamaño del nodo][node-sizing]
* [Escala de un clúster][scale]
* [Escalador automático horizontal de pod][scale-horizontal]
* [Cluster Autoscaler][scale-auto]
* [Grupos de acceso puntual][spot-pools]
* [Grupos del sistema][system-pools]
* [Reservas de recursos][resource-reservations]
* [Grupos con ubicación por proximidad][proiximity-placement-groups]
* [Zonas de disponibilidad][availability-zones]

[availability-zones]: availability-zones.md
[azure-monitor]: ../azure-monitor/containers/container-insights-overview.md
[azure-traffic-manager]: ../traffic-manager/traffic-manager-overview.md
[proiximity-placement-groups]: reduce-latency-ppg.md
[regions]: faq.md#which-azure-regions-currently-provide-aks
[resource-reservations]: concepts-clusters-workloads.md#resource-reservations
[scale]: concepts-scale.md
[scale-auto]: concepts-scale.md#cluster-autoscaler
[scale-horizontal]: concepts-scale.md#horizontal-pod-autoscaler
[spot-pools]: spot-node-pool.md
[multiple-node-pools]: use-multiple-node-pools.md
[node-sizing]: use-multiple-node-pools.md#specify-a-vm-size-for-a-node-pool
[sustainability-calculator]: https://azure.microsoft.com/blog/microsoft-sustainability-calculator-helps-enterprises-analyze-the-carbon-emissions-of-their-it-infrastructure/
[system-pools]: use-system-pools.md
[principles-sse]: https://docs.microsoft.com/learn/modules/sustainable-software-engineering-overview/