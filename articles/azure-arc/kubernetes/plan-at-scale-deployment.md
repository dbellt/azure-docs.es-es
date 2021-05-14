---
title: Planeación e implementación de Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/12/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: Incorporación de un gran número de clústeres a Kubernetes habilitado para Azure Arc para la administración de la configuración
ms.openlocfilehash: 4b34cee08db508728f01d262ae4b1ee4ed4754e1
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107315260"
---
# <a name="plan-and-deploy-azure-arc-enabled-kubernetes"></a>Planeación e implementación de Kubernetes habilitado para Azure Arc

La implementación de un servicio de infraestructura de TI o de una aplicación empresarial es un reto para cualquier empresa. Para evitar sorpresas no deseadas y costos imprevistos, debe planearla minuciosamente para asegurarse de que está lo más preparado posible. Este plan debe identificar los criterios de diseño e implementación que deben cumplirse para completar las tareas.

Para que la implementación se lleve a cabo sin problemas, el plan debe incluir una comprensión clara de los siguientes aspectos:

* Roles y responsabilidades.
* Inventario de todos los clústeres de Kubernetes.
* Cumplimiento de los requisitos de red.
* El conjunto de habilidades y la formación requeridos para habilitar la implementación correcta y la administración continua.
* Criterios de aceptación y seguimiento de su grado de cumplimiento.
* Herramientas o métodos que se van a usar para automatizar las implementaciones.
* Riesgos identificados y planes de mitigación para evitar retrasos e interrupciones.
* Cómo evitar la interrupción durante la implementación.
* ¿Cuál es la ruta de escalación cuando se produce un problema importante?

El propósito de este artículo es asegurarse de que está preparado para una implementación correcta de Kubernetes habilitado para Azure Arc en varios clústeres de producción en su entorno.

## <a name="prerequisites"></a>Requisitos previos

* Un clúster de Kubernetes existente. Si no tiene ninguno, puede crear un clúster mediante una de estas opciones:
    - [Kubernetes en Docker (KIND)](https://kind.sigs.k8s.io/)
    - Creación de un clúster de Kubernetes con Docker para [Mac](https://docs.docker.com/docker-for-mac/#kubernetes) o [Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
    - Clúster de Kubernetes autoadministrado mediante la [API de clúster](https://cluster-api.sigs.k8s.io/user/quick-start.html)

* Los equipos tienen conectividad desde la red local u otro entorno de nube a recursos de Azure, ya sea directamente o a través de un servidor proxy. Puede encontrar más detalles en los [requisitos previos de red](quickstart-connect-cluster.md#meet-network-requirements).

* Un archivo `kubeconfig` que apunte al clúster que quiere conectar a Azure Arc.
* Permisos de "lectura" y "escritura" para el usuario o la entidad de servicio que crea el tipo de recurso Kubernetes habilitado para Azure Arc de `Microsoft.Kubernetes/connectedClusters`.

## <a name="pilot"></a>Piloto

Antes de realizar la implementación en todos los clústeres de producción, empiece por evaluar este proceso de implementación antes de adoptarlo ampliamente en su entorno. Para una prueba piloto, identifique un muestreo representativo de clústeres que no sean críticos para la capacidad de su empresa de llevar a cabo sus actividades. Asegúrese de dejar tiempo suficiente para ejecutar la prueba piloto y evaluar su efecto: se recomienda aproximadamente 30 días.

Establezca un plan formal que describa el ámbito y los detalles de la prueba piloto. El siguiente plan de ejemplo debería ayudarle a empezar.

* **Objetivos**: describe los factores técnicos y empresariales que llevaron a la decisión de que es necesaria una prueba piloto.
* **Criterios de selección**: especifica los criterios que se usan para seleccionar los aspectos de la solución que se mostrarán a través de una prueba piloto.
* **Ámbito:** abarca los componentes de la solución, la programación esperada, la duración del piloto y el número de clústeres de destino.
* **Criterios y métricas de éxito**: defina los criterios de éxito de la prueba piloto y las medidas específicas empleadas para determinar el nivel de éxito.
* **Plan de formación**: describe el plan de formación de los ingenieros de sistemas, administradores, etc. que son nuevos en Azure y sus servicios durante la prueba piloto.
* **Plan de transición**: describe la estrategia y los criterios que se usan para guiar la transición de la prueba piloto a producción.
* **Reversión**: describe los procedimientos para revertir una prueba piloto al estado anterior a la implementación.
* **Riesgos**: enumera todos los riesgos identificados para la realización de la prueba piloto y los asociados a la implementación de producción.

## <a name="phase-1-build-a-foundation"></a>Fase 1: Creación de una base

En esta fase, los ingenieros o administradores del sistema realizan las actividades principales, como la creación de grupos de recursos, etiquetas y asignaciones de roles para que los recursos de Kubernetes habilitado para Azure Arc puedan crearse y operar.

|Tarea |Detail |Duration |
|-----|-------|---------|
| [Cree un grupo de recursos](../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) | Un grupo de recursos dedicado para incluir solo recursos de Kubernetes habilitado para Azure Arc y centralizar la administración y supervisión de estos recursos. | Una hora |
| Aplique [etiquetas](../../azure-resource-manager/management/tag-resources.md) para ayudar a organizar las máquinas. | Evalúe y desarrolle una [estrategia de etiquetado](/azure/cloud-adoption-framework/decision-guides/resource-tagging/) alineada con el personal de TI. Esto puede ayudar a reducir la complejidad de la administración de los recursos de Kubernetes habilitado para Azure Arc y a simplificar la toma de decisiones de administración. | Un día |
| Identificación de [configuraciones](tutorial-use-gitops-connected-cluster.md) para GitOps | Identifique las configuraciones de aplicación o línea de base, como `PodSecurityPolicy`, `NetworkPolicy` que desea implementar en los clústeres. | Un día |
| Desarrollo de un plan de gobernanza de [Azure Policy](../../governance/policy/overview.md) | Determine cómo implementará la gobernanza de los clústeres de Kubernetes habilitado para Azure Arc en el ámbito de suscripción o grupo de recursos con Azure Policy. | Un día |
| Configuración del [control de acceso basado en rol](../../role-based-access-control/overview.md) (RBAC) | Desarrolle un plan de acceso para identificar quién tiene permisos de lectura, escritura o todos los permisos en los clústeres. | Un día |

## <a name="phase-2-deploy-azure-arc-enabled-kubernetes"></a>Fase 2: Implementación de Kubernetes habilitado para Azure Arc

En esta fase, conectaremos los clústeres de Kubernetes a Azure:

|Tarea |Detail |Duration |
|-----|-------|---------|
| [Conexión del primer clúster de Kubernetes a Azure Arc](quickstart-connect-cluster.md) | Como parte de la conexión del primer clúster a Azure Arc, configure el entorno de incorporación con todas las herramientas necesarias, como la CLI de Azure, Helm y la extensión `connectedk8s` para la CLI de Azure. | 15 minutos |
| [Creación de una entidad de servicio](create-onboarding-service-principal.md) | Cree una entidad de servicio para conectar clústeres de Kubernetes de forma no interactiva mediante la CLI de Azure o PowerShell. | Una hora |


## <a name="phase-3-manage-and-operate"></a>Fase 3: Administración y funcionamiento

En esta fase, implementaremos aplicaciones y configuraciones de línea de base en los clústeres de Kubernetes.

|Tarea |Detail |Duration |
|-----|-------|---------|
|[Creación de configuraciones](tutorial-use-gitops-connected-cluster.md) en los clústeres | Cree configuraciones para implementar las aplicaciones en el recurso de Kubernetes habilitado para Azure Arc. | 15 minutos |
|[Uso de Azure Policy](use-azure-policy.md) para la aplicación a escala de configuraciones | Cree asignaciones de directivas para automatizar la implementación de configuraciones de línea de base en todos los clústeres en un ámbito de suscripción o grupo de recursos. | 15 minutos |
| [Actualización de agentes de Azure Arc](agent-upgrade.md) | Si ha deshabilitado la actualización automática de agentes en los clústeres, actualice los agentes manualmente a la versión más reciente para asegurarse de que tiene las correcciones de errores y seguridad más recientes. | 15 minutos |

## <a name="next-steps"></a>Pasos siguientes

* Siga el artículo de inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Cree configuraciones](./tutorial-use-gitops-connected-cluster.md) en el clúster de Kubernetes habilitado para Azure Arc.
* [Use Azure Policy para aplicar configuraciones a escala](./use-azure-policy.md).