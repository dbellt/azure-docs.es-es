---
title: Procedimientos recomendados para el aislamiento de clústeres
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados del operador de clústeres para el aislamiento en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/09/2021
ms.openlocfilehash: e51689d33711f127f775c63c9d7fc8ad4c901604
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105177"
---
# <a name="best-practices-for-cluster-isolation-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para el aislamiento de clústeres en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), a menudo necesita aislar los equipos y las cargas de trabajo. AKS proporciona flexibilidad acerca de cómo puede ejecutar clústeres multiempresa y aislar recursos. Para maximizar la inversión en Kubernetes, primero debe comprender e implementar las funciones multiempresa y de aislamiento de AKS.

Este artículo de procedimientos recomendados se centra en el aislamiento de los operadores de clústeres. En este artículo aprenderá a:

> [!div class="checklist"]
> * Planificar clústeres multiempresa y separación de recursos.
> * Usar el aislamiento lógico o físico de los clústeres de AKS.

## <a name="design-clusters-for-multi-tenancy"></a>Diseño de clústeres para la configuración multiempresa

Kubernetes permite aislar lógicamente equipos y cargas de trabajo en el mismo clúster. El objetivo es proporcionar el número mínimo de privilegios en el ámbito de los recursos que necesita cada equipo. Un [espacio de nombres][k8s-namespaces] de Kubernetes crea un límite de aislamiento lógico. A continuación, se incluyen otras características y consideraciones de Kubernetes sobre el aislamiento y la arquitectura multiinquilino:

### <a name="scheduling"></a>Scheduling

La *programación* usa características básicas, como las cuotas de recursos y los presupuestos de interrupciones del pod. Para más información acerca de estas características, consulte [Procedimientos recomendados para características básicas del programador en Azure Kubernetes Service (AKS)][aks-best-practices-scheduler].

Las siguientes son algunas de las características más avanzadas del programador:
* Intolerancias y tolerancias
* Selectores de nodos
* Afinidad o antiafinidad de nodos y pods. 

Para más información acerca de estas características, consulte [Procedimientos recomendados para características avanzadas del programador en Azure Kubernetes Service (AKS)][aks-best-practices-advanced-scheduler].

### <a name="networking"></a>Redes

Las *redes* usan directivas de red para controlar el flujo de tráfico de entrada y salida de pods.

### <a name="authentication-and-authorization"></a>Autenticación y autorización

Usos de la *autenticación y autorización*:
* Control de acceso basado en roles (RBAC)
* Integración de Azure Active Directory (AD)
* Identidades de pod
* Secretos en Azure Key Vault 

Para más información sobre estas características, consulte [Procedimientos recomendados para la autenticación y autorización en Azure Kubernetes Service (AKS)][aks-best-practices-identity].

### <a name="containers"></a>Contenedores
Los *contenedores* incluyen:
* El complemento de Azure Policy para AKS con el fin de aplicar la seguridad del pod.
* El uso de contextos de seguridad de pod.
* Examen de las imágenes y el entorno de ejecución en busca de vulnerabilidades. 
* El uso de AppArmor o Seccomp (informática segura) para restringir el acceso del contenedor al nodo subyacente.

## <a name="logically-isolate-clusters"></a>Aislamiento de clústeres de forma lógica

> **Guía de procedimientos recomendados**
>
> Separe los equipos y proyectos mediante el *aislamiento lógico*. Minimice el número de clústeres físicos de AKS que implementa para aislar los equipos o las aplicaciones.

Con el aislamiento lógico, un único clúster de AKS puede utilizarse para varios entornos, equipos o cargas de trabajo. Los [espacios de nombres][k8s-namespaces] de Kubernetes forman el límite de aislamiento lógico para las cargas de trabajo y los recursos.

![Aislamiento lógico de un clúster de Kubernetes en AKS](media/operator-best-practices-cluster-isolation/logical-isolation.png)

La separación lógica de clústeres normalmente proporciona una mayor densidad de pods que los clústeres aislados físicamente, con una menor capacidad de proceso inactiva en el clúster. Cuando se combina con el escalador automático de clúster de Kubernetes, puede escalar vertical u horizontalmente el número de nodos para satisfacer las necesidades. Esta estrategia de procedimiento recomendado para el escalado automático minimiza los costos al ejecutar solo el número de nodos necesario.

Actualmente, los entornos de Kubernetes no están completamente seguros ante el uso de varios inquilinos hostiles. En un entorno multiinquilino, varios inquilinos trabajan en una infraestructura compartida común. Si no se puede confiar en todos los inquilinos, necesitará de planeamiento adicional para evitar que dichos inquilinos afecten a la seguridad y el servicio de otros.

Las características de seguridad adicionales, como las *directivas de seguridad de pods* o Kubernetes RBAC para nodos, bloquean eficazmente las vulnerabilidades de seguridad. Para una verdadera seguridad al ejecutar cargas de trabajo multiinquilino hostiles, debe confiar solo en un hipervisor. El dominio de seguridad de Kubernetes se convierte en todo el clúster, no en un nodo específico. 

En el caso de estos tipos de cargas de trabajo multiinquilino hostiles, debe usar clústeres que estén físicamente aislados.

## <a name="physically-isolate-clusters"></a>Aislamiento de clústeres de forma física

> **Guía de procedimientos recomendados** 
>
> Minimice el uso del aislamiento físico para cada implementación de la aplicación o equipo independientes. En su lugar, use el aislamiento *lógico*, como se describe en la sección anterior.

Separar físicamente los clústeres de AKS es un enfoque común para el aislamiento de clústeres. En este modelo de aislamiento, los equipos o las cargas de trabajo se asignan a su propio clúster de AKS. Aunque el aislamiento físico podría parecer la manera más fácil de aislar cargas de trabajo o equipos, agrega una sobrecarga financiera y administrativa. Ahora, debe mantener estos clústeres, así como proporcionar acceso y asignar permisos de manera individual. También se le facturará por cada nodo individual.

![Aislamiento físico de clústeres individuales de Kubernetes en AKS](media/operator-best-practices-cluster-isolation/physical-isolation.png)

Los clústeres físicamente independientes suelen tener una densidad de pods baja. Ya que cada equipo o carga de trabajo tiene su propio clúster de AKS, el clúster a menudo se aprovisiona en exceso con recursos de proceso. Con frecuencia, se programa un pequeño número de pods en esos nodos. La capacidad de los nodos no utilizada no se puede usar para aplicaciones o servicios que están desarrollando otros equipos. Estos recursos en exceso contribuyen a los costos adicionales en los clústeres físicamente independientes.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en el aislamiento de clústeres. Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]
* [Características avanzadas del programador de Kubernetes][aks-best-practices-advanced-scheduler]
* [Autenticación y autorización][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->

<!-- INTERNAL LINKS -->
[k8s-namespaces]: concepts-clusters-workloads.md#namespaces
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-identity]: operator-best-practices-identity.md
