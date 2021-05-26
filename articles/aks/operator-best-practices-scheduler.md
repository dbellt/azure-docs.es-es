---
title: 'Procedimientos recomendados de operador: características básicas del programador en Azure Kubernetes Service (AKS)'
description: Conozca las prácticas recomendadas para utilizar características básicas del programador, como las cuotas de recursos y los presupuestos de interrupciones de pods en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/09/2021
ms.openlocfilehash: 034b7e18d3114804f846d77aa4feb001492c8883
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467070"
---
# <a name="best-practices-for-basic-scheduler-features-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para características básicas del programador en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), a menudo necesita aislar los equipos y las cargas de trabajo. El programador de Kubernetes le permite controlar la distribución de los recursos de proceso o limitar el impacto de los eventos de mantenimiento.

Este artículo de procedimientos recomendados se centra en las características básicas de Kubernetes para operadores de clúster. En este artículo aprenderá a:

> [!div class="checklist"]
> * Utilizar las cuotas de recursos para proporcionar una cantidad fija de recursos para cargas de trabajo o equipos
> * Limitar el impacto del mantenimiento programado utilizando presupuestos de interrupciones de pods

## <a name="enforce-resource-quotas"></a>Aplicación de cuotas de recursos

> **Guía de procedimientos recomendados** 
> 
> Planee y aplique cuotas de recursos en el nivel del espacio de nombres. Si los pods no definen solicitudes y límites de recursos, rechace la implementación. Supervise el uso de recursos y ajuste las cuotas según sea necesario.

Las solicitudes y límites de recursos se colocan en la especificación de pod. El programador de Kubernetes usa los límites en el momento de la implementación para buscar un nodo disponible en el clúster. Los límites y las solicitudes funcionan en el nivel de pod individual. Para obtener más información sobre cómo definir estos valores, vea [Definición de los límites y solicitudes de recursos del pod][resource-limits].

Para proporcionar una manera de reservar y limitar los recursos a través de un proyecto o equipo de desarrollo, debe usar *las cuotas de recursos*. Estas cuotas se definen en un espacio de nombres y se pueden usar para establecer cuotas de la siguiente manera:

* **Recursos de proceso**, como CPU y memoria, o GPU.
* **Recursos de almacenamiento**, que incluyen el número total de volúmenes o la cantidad de espacio en disco para una clase de almacenamiento determinada.
* **Recuento de objetos**, como el número máximo de secretos, servicios o trabajos que se pueden crear.

Kubernetes no sobrecarga los recursos. Una vez que el total de solicitudes de recursos acumuladas supera la cuota asignada, todas las demás implementaciones no se realizarán correctamente.

Al definir las cuotas de recursos, todos los pods creados en el espacio de nombres deben proporcionar límites o recursos en sus especificaciones de pods. Si no proporcionan estos valores, puede rechazar la implementación. En su lugar, puede [configurar solicitudes y límites predeterminados para un espacio de nombres][configure-default-quotas].

El siguiente manifiesto YAML de ejemplo llamado *dev-app-team-quotas.yaml* establece un límite rígido de un total de *10* CPU, *20Gi* de memoria y *10* pods:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-app-team
spec:
  hard:
    cpu: "10"
    memory: 20Gi
    pods: "10"
```

Esta cuota de recursos se puede aplicar mediante la especificación del espacio de nombres, como *dev-apps*:

```console
kubectl apply -f dev-app-team-quotas.yaml --namespace dev-apps
```

Trabaje con los desarrolladores y propietarios de su aplicación para conocer sus necesidades y aplicar las cuotas de recursos adecuadas.

Para obtener más información sobre los objetos de recursos, los ámbitos y las prioridades disponibles, consulte [Resource quotas][k8s-resource-quotas] (Cuotas de recursos) en Kubernetes.

## <a name="plan-for-availability-using-pod-disruption-budgets"></a>Planeación de disponibilidad mediante presupuestos de interrupciones de pods

> **Guía de procedimientos recomendados** 
>
> Para mantener la disponibilidad de las aplicaciones, defina presupuestos de interrupciones de pods para asegurarse de que haya un número mínimo de pods disponible en el clúster.

Hay dos eventos de interrupción que provocan que se quiten los pods:

### <a name="involuntary-disruptions"></a>Interrupciones involuntarias

Las *interrupciones involuntarias* son eventos más allá del control típico del operador del clúster o el propietario de la aplicación. Inclusión:
* Error de hardware en la máquina física
* Pánico de kernel
* Eliminación de una máquina virtual de nodo

Las interrupciones involuntarias se pueden mitigar mediante:
* El uso de varias réplicas de los pods en una implementación. 
* La ejecución de varios nodos en el clúster de AKS. 

### <a name="voluntary-disruptions"></a>Interrupciones voluntarias

Las *interrupciones voluntarias* son eventos solicitados por el operador del clúster o el propietario de la aplicación. Inclusión:
* Actualizaciones de clústeres
* Plantilla de implementación actualizada
* Eliminación accidental de un pod

Kubernetes proporciona *presupuestos de interrupciones de pods* para interrupciones voluntarias, lo que le permite planear cómo responden las implementaciones o los conjuntos de réplicas cuando se produce un evento de interrupción voluntaria. Mediante los presupuestos de interrupciones de pods, los operadores del clúster pueden definir un número máximo de recursos no disponibles o un número mínimo de recursos disponibles. 

Si actualiza un clúster o una plantilla de implementación, el programador de Kubernetes programará pods adicionales en otros nodos antes de permitir que continúen los eventos de interrupción voluntaria. El programador espera a que se reinicie un nodo hasta que la cantidad definida de pods se programe correctamente en otros nodos del clúster.

Veamos un ejemplo de un conjunto de réplicas con cinco pods que ejecutan NGINX. Se ha asignado la etiqueta `app: nginx-frontend` a los pods del conjunto de réplicas. Durante un evento de interrupción voluntaria, como una actualización de clúster, desea asegurarse de que al menos tres pods continúen ejecutándose. El siguiente manifiesto YAML para un objeto *PodDisruptionBudget* define estos requisitos:

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
   name: nginx-pdb
spec:
   minAvailable: 3
   selector:
    matchLabels:
      app: nginx-frontend
```

También puede definir un porcentaje, como *60 %* , que permite compensar automáticamente el conjunto de réplicas escalando verticalmente el número de pods.

Puede definir un número máximo de instancias no disponibles en un conjunto de réplicas. De nuevo, también se puede definir un porcentaje máximo de pods no disponibles. El siguiente manifiesto YAML de presupuesto de interrupciones de pods define que no estarán no disponibles más de dos pods en el conjunto de réplicas:

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
   name: nginx-pdb
spec:
   maxUnavailable: 2
   selector:
    matchLabels:
      app: nginx-frontend
```

Una vez que se ha definido su presupuesto de interrupciones de pods, créelo en el clúster de AKS al igual que cualquier otro objeto de Kubernetes:

```console
kubectl apply -f nginx-pdb.yaml
```

Trabaje con los desarrolladores y propietarios de su aplicación para conocer sus necesidades y aplicar los presupuestos de interrupciones de pods adecuados.

Para obtener más información sobre el uso de los presupuestos de interrupciones de pods, vea [Especificando un presupuesto de disrupción para tu aplicación][k8s-pdbs].

## <a name="next-steps"></a>Pasos siguientes

Este artículo se ha centrado en características básicas del programador de Kubernetes. Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Aislamiento de clúster y de multiinquilinato][aks-best-practices-cluster-isolation]
* [Características avanzadas del programador de Kubernetes][aks-best-practices-advanced-scheduler]
* [Autenticación y autorización][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-resource-quotas]: https://kubernetes.io/docs/concepts/policy/resource-quotas/
[configure-default-quotas]: https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
[k8s-pdbs]: https://kubernetes.io/docs/tasks/run-application/configure-pdb/

<!-- INTERNAL LINKS -->
[resource-limits]: developer-best-practices-resource-management.md#define-pod-resource-requests-and-limits
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
