---
title: 'Información general: Logic Apps habilitado para Azure Arc'
description: Obtenga información sobre los flujos de trabajo de Logic Apps de un solo inquilino que pueden ejecutarse en cualquier lugar donde se pueda ejecutar Kubernetes.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, ladolan, reylons, archidda, sopai, azla
ms.topic: overview
ms.date: 05/25/2021
ms.openlocfilehash: dcaa0e9fe10d26bb6b1e7d014e0b6e50f09a81f2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388476"
---
# <a name="what-is-azure-arc-enabled-logic-apps-preview"></a>¿Qué es Logic Apps habilitado para Azure Arc? (versión preliminar)

> [!NOTE]
> Esta funcionalidad está en versión preliminar y está sujeta a las [Condiciones de uso complementarias para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Con Logic Apps habilitado para Azure Arc, puede desarrollar y ejecutar aplicaciones lógicas basadas en un solo inquilino en cualquier lugar donde se pueda ejecutar Kubernetes. Por ejemplo, puede ejecutar los flujos de trabajo de su aplicación lógica en Azure, Azure Kubernetes Service, de forma local e incluso en otros proveedores de la nube. Esta oferta le proporciona una plataforma de administración centralizada de panel único a través de Azure Arc y Azure Portal para las siguientes funcionalidades: 

- Use Azure Logic Apps como plataforma de integración.
- Conecte los flujos de trabajo a todos sus servicios sin importar dónde estén hospedados.
- Ejecute las soluciones de integración directamente junto con los servicios.
- Cree y edite flujos de trabajo con Visual Studio Code.
- Realice implementaciones con canalizaciones de su elección para DevOps.
- Controle la infraestructura y los recursos en entornos de Azure, entornos que no sean de Azure, en varias nubes, de forma local y en entornos perimetrales.

Para más información, revise la siguiente documentación:

- [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)
- [Un solo inquilino en comparación con otros entornos de Logic Apps](../logic-apps/single-tenant-overview-compare.md)
- [Introducción a Azure Arc](../azure-arc/overview.md)
- [Información general de Azure Kubernetes Service](../aks/intro-kubernetes.md)
- [¿Qué es Kubernetes habilitado para Azure Arc?](../azure-arc/kubernetes/overview.md)
- [¿Qué es Kubernetes?](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)

<a name="why-use"></a>

## <a name="why-use-arc-enabled-logic-apps"></a>Por qué usar Logic Apps habilitado para Azure Arc

Con Logic Apps habilitado para Azure Arc, puede crear e implementar flujos de trabajo de aplicaciones lógicas de la misma manera que en la experiencia de un solo inquilino para Azure Logic Apps. También puede obtener más control y flexibilidad cuando tiene aplicaciones lógicas que se ejecutan en la infraestructura de Kubernetes que usted mismo opere y administre.

Existen pequeñas diferencias entre las experiencias de Azure Arc y de Logic Apps de un solo inquilino para crear, diseñar e implementar aplicaciones lógicas. Cuando usa Logic Apps habilitado para Azure Arc, la principal diferencia es que sus aplicaciones lógicas se ejecutan en una *ubicación personalizada*. Esta ubicación se asigna a un clúster de Kubernetes habilitado para Arc donde ha instalado y habilitado el conjunto de extensiones de la plataforma Azure App Service.

Por ejemplo, este clúster puede ser una instancia de Azure Kubernetes Service, una instancia de Kubernetes sin sistema operativo u otra configuración. El conjunto de extensiones le permite ejecutar servicios de plataforma como Azure Logic Apps, Azure Functions y Azure App Service en el clúster de Kubernetes.

Para más información, revise la siguiente documentación:

- [Un solo inquilino en comparación con otros entornos de Azure Logic Apps](../logic-apps/single-tenant-overview-compare.md)
- [Información general de Azure Kubernetes Service](../aks/intro-kubernetes.md)
- [¿Qué es Kubernetes habilitado para Azure Arc?](../azure-arc/kubernetes/overview.md)
- [Ubicaciones personalizadas en Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/conceptual-custom-locations.md)
- [App Service, Functions y Logic Apps en Azure Arc (versión preliminar)](../app-service/overview-arc-integration.md)
- [Configuración de un clúster de Kubernetes habilitado para Azure Arc para ejecutar App Service, Functions y Logic Apps (versión preliminar)](../app-service/manage-create-arc-environment.md)

<a name="when-to-use"></a>

## <a name="when-to-use-arc-enabled-logic-apps"></a>Cuándo usar Logic Apps habilitado para Arc

Aunque Kubernetes proporciona más control y flexibilidad, también tiene sobrecarga operativa. Si está satisfecho con el servicio Logic Apps porque satisface sus necesidades, es recomendable seguir usando este servicio. Sin embargo, considere la posibilidad de usar Logic Apps habilitado para Azure Arc cuando se encuentre en los siguientes escenarios:

- Ya ha ejecutado todas las aplicaciones y servicios en Kubernetes. Quiere extender estos procesos y controles a todos los demás servicios PaaS.

- Quiere usar Logic Apps como plataforma de integración. Sin embargo, necesitará redes más detalladas que cuenten con cierto control de procesos y flexibilidad. No quiere usar un entorno de servicio de integración (ISE) ni App Service Environment (ASE).

- Por motivos de seguridad, necesita controlar dónde se ejecutan las aplicaciones lógicas, por ejemplo, en su propia región o en su propio centro de datos. 

- Quiere ejecutar las aplicaciones lógicas en escenarios de varias nubes y usar el servicio Logic Apps como única plataforma de integración para todas las aplicaciones dondequiera que se ejecuten.

<a name="compare"></a>

## <a name="compare-offerings"></a>Comparación de ofertas

En esta tabla se proporciona una comparación de alto nivel entre las funcionalidades de las ofertas de Azure Logic Apps actuales:

:::row:::
   :::column:::
      **Funcionalidad**
   :::column-end:::
   :::column:::
      **Instancia de Logic Apps de multiinquilino (consumo)**
   :::column-end:::
   :::column:::
      **Instancia de Logic Apps de un solo inquilino (estándar)**
   :::column-end:::
   :::column:::
      **Contenedores independientes**
   :::column-end:::
   :::column:::
      **Azure Arc**
   :::column-end:::
:::row-end:::
:::row:::
   :::column:::
      Desarrollo local
   :::column-end:::
   :::column:::
      Visual Studio Code, Visual Studio
   :::column-end:::
   :::column:::
      Visual Studio Code, incluido el historial de ejecución y la información general con la depuración de puntos de interrupción.
   :::column-end:::
   :::column:::
      Visual Studio Code
   :::column-end:::
   :::column:::
      Visual Studio Code, incluido el historial de ejecución y la información general con la depuración de puntos de interrupción.
   :::column-end:::
:::row-end:::
:::row:::
   :::column:::
      Hospedaje
   :::column-end:::
   :::column:::
      Ejecución solo en Azure
   :::column-end:::
   :::column:::
      Ejecución solo en Azure
   :::column-end:::
   :::column:::
      Ejecución en cualquier lugar donde se ejecuten los contenedores
   :::column-end:::
   :::column:::
      Ejecución en cualquier lugar con un clúster de Kubernetes habilitado para Arc
   :::column-end:::
:::row-end:::
:::row:::
   :::column:::
      Administración
   :::column-end:::
   :::column:::
      Experiencia de Logic Apps totalmente administrada
   :::column-end:::
   :::column:::
      Experiencia de Logic Apps totalmente administrada
   :::column-end:::
   :::column:::
      Sin administrar
   :::column-end:::
   :::column:::
      Experiencia de Logic Apps administrada con control operativo en el nivel de Kubernetes
   :::column-end:::
:::row-end:::
:::row:::
   :::column:::
      Supervisión
   :::column-end:::
   :::column:::
      Supervisión en Azure Portal, incluido el historial de ejecución, la opción para volver a enviar la ejecución y funcionalidades de Application Insights, si es necesario.
   :::column-end:::
   :::column:::
      Supervisión en Azure Portal, incluido el historial de ejecución, la opción para volver a enviar la ejecución y funcionalidades de Application Insights, si es necesario.
   :::column-end:::
   :::column:::
      Supervisión únicamente con Application Insights u otras herramientas de supervisión de contenedores
   :::column-end:::
   :::column:::
      Supervisión en Azure Portal, incluido el historial de ejecución, la opción para volver a enviar la ejecución y funcionalidades de Application Insights, si es necesario.
   :::column-end:::
:::row-end:::
:::row:::
   :::column:::
      Ampliación
   :::column-end:::
   :::column:::
      Control del escalado mediante un plan de consumo
   :::column-end:::
   :::column:::
      Control del escalado mediante un plan de estándar
   :::column-end:::
   :::column:::
      No disponible
   :::column-end:::
   :::column:::
      Controle el escalado mediante el [Escalado automático controlado por eventos basado en Kubernetes (KEDA)](https://keda.sh/). Configure eventos de escalado en función de la longitud de la cola.
   :::column-end:::
:::row-end:::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación e implementación de flujos de trabajo con una instancia de Logic Apps habilitada para Arc](azure-arc-enabled-logic-apps-create-deploy-workflows.md)
