---
title: 'Configuraciones y GitOps: Kubernetes habilitado para Azure Arc'
services: azure-arc
ms.service: azure-arc
ms.date: 03/02/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo se proporciona información general conceptual de la funcionalidad de GitOps y configuraciones de Kubernetes habilitado para Azure Arc.
keywords: Kubernetes, Arc, Azure, contenedores, configuración, GitOps
ms.openlocfilehash: 198bd45374d998ef0d9d0e612e974484d52e2f20
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443313"
---
# <a name="configurations-and-gitops-with-azure-arc-enabled-kubernetes"></a>Configuraciones y GitOps con Kubernetes habilitado para Azure Arc

En relación con Kubernetes, GitOps es la práctica de declarar el estado deseado de las configuraciones de los clústeres de Kubernetes (implementaciones, espacios de nombres, etc.) en un repositorio de Git. A esta declaración le sigue una implementación de sondeo basada en extracción de estas configuraciones de clúster mediante un operador. El repositorio de Git puede contener:
* Manifiestos con formato YAML que describen cualquier recurso de Kubernetes válido, incluidos espacios de nombres, ConfigMaps, implementaciones, DaemonSets, etc.
* Gráficos de Helm para implementar aplicaciones.

[Flux](https://docs.fluxcd.io/), una popular herramienta de código abierto en el espacio de GitOps, se puede implementar en el clúster de Kubernetes para facilitar el flujo de configuraciones de un repositorio de Git a un clúster de Kubernetes. Flux admite la implementación de su operador tanto en el ámbito del clúster como del espacio de nombres. Un operador de Flux implementado con el ámbito del espacio de nombres solo puede implementar objetos de Kubernetes dentro de ese espacio de nombres específico. La capacidad de elegir entre el ámbito del clúster o del espacio de nombres ayuda a lograr patrones de implementación multiinquilino en el mismo clúster de Kubernetes.

## <a name="configurations"></a>Configurations

[ ![Arquitectura de configuración](./media/conceptual-configurations.png) ](./media/conceptual-configurations.png#lightbox)

La conexión entre el clúster y un repositorio de Git se crea como recurso de configuración (`Microsoft.KubernetesConfiguration/sourceControlConfigurations`) además del recurso de Kubernetes habilitado para Azure Arc (representado por `Microsoft.Kubernetes/connectedClusters`) en Azure Resource Manager. 

Las propiedades del recurso de configuración se usan para implementar el operador de Flux en el clúster con los parámetros adecuados, como el repositorio de Git desde el que se extraerán los manifiestos y el intervalo de sondeo en el que se van a extraer. Los datos del recurso de configuración se almacenan cifrados en reposo en una base de datos de Azure Cosmos DB para garantizar su confidencialidad.

El `config-agent` en ejecución en el clúster es responsable de:
* El seguimiento de los recursos de configuración, nuevos o actualizados, en el recurso de Kubernetes habilitado para Azure Arc.
* La implementación de un operador de Flux para inspeccionar el repositorio de Git para cada recurso de configuración.
* La aplicación de actualizaciones realizadas a cualquier recurso de configuración. 

Puede crear varios recursos de configuración limitados al espacio de nombres en el mismo clúster de Kubernetes habilitado para Azure Arc para lograr la función de multiinquilino.

> [!NOTE]
> * `config-agent` supervisa la disponibilidad de recursos de configuración, nuevos o actualizados, en el recurso Kubernetes habilitado para Arc. Por lo tanto, los agentes requieren conectividad para que el estado deseado se lleve al clúster. Si los agentes no pueden conectarse a Azure, se produce un retraso en la propagación del estado deseado al clúster.
> * Las entradas de clientes confidenciales, como la clave privada, el contenido de hosts conocidos, el nombre de usuario HTTPS y el token/contraseña, no se almacenan durante más de 48 horas en los servicios de Kubernetes habilitados para Azure Arc. Si usa entradas confidenciales para las configuraciones, ponga en línea los clústeres lo más regularmente posible.

## <a name="apply-configurations-at-scale"></a>Aplicación de configuraciones a escala

Como Azure Resource Manager administra las configuraciones, puede automatizar la creación de la misma configuración en todos los recursos de Kubernetes habilitado para Azure Arc, en el ámbito de una suscripción o de un grupo de recursos. 

Esta aplicación a escala garantiza que se pueda aplicar una configuración de línea de base común (que contiene configuraciones, como ClusterRoleBindings, RoleBindings y NetworkPolicy) a toda la flota o inventario de clústeres de Kubernetes habilitado para Azure Arc.

## <a name="next-steps"></a>Pasos siguientes

* Siga el artículo de inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Cree configuraciones](./tutorial-use-gitops-connected-cluster.md) en el clúster de Kubernetes habilitado para Azure Arc.
* [Use Azure Policy para aplicar configuraciones a escala](./use-azure-policy.md).