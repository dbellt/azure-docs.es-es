---
title: 'Extensiones de clústeres: Kubernetes con Azure Arc habilitado'
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo, encontrará información general y algunos conceptos sobre la funcionalidad de extensiones de clústeres de Kubernetes con Azure Arc habilitado.
ms.openlocfilehash: 4b9a3991b51ec45e7a64acd546c031d4241c97af
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450923"
---
# <a name="cluster-extensions-on-azure-arc-enabled-kubernetes"></a>Extensiones de clústeres de Kubernetes con Azure Arc habilitado

Los [gráficos de Helm](https://helm.sh/) le ayudan a administrar las aplicaciones de Kubernetes, ya que contienen los elementos necesarios para definir, instalar y actualizar las aplicaciones de Kubernetes, incluso las más complejas. La característica de extensiones de clústeres está diseñada para utilizarse con los componentes de empaquetado de Helm. Para ello, proporciona una experiencia controlada por Azure Resource Manager que permite instalar las extensiones de clústeres, como Azure Monitor y Azure Defender para Kubernetes, así como administrar su ciclo de vida. La característica de extensiones de clústeres ofrece, además, estas otras ventajas en comparación con las funcionalidades nativas que están disponibles con los gráficos de Helm:

- Permite realizar un inventario de todos los clústeres y las extensiones instaladas en ellos.
- Permite utilizar Azure Policy para automatizar la implementación a escala de las extensiones de los clústeres.
- Permite suscribirse a los trenes de versiones de cada extensión.
- Permite configurar la actualización automática de las extensiones.
- Permite crear instancias de las extensiones y admite los eventos de administración del ciclo de vida de actualización y eliminación.

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="architecture"></a>Architecture

[ ![Arquitectura de las extensiones de clústeres](./media/conceptual-extensions.png) ](./media/conceptual-extensions.png#lightbox)

La instancia de una extensión de clúster se crea como un recurso de extensión de Azure Resource Manager (`Microsoft.KubernetesConfiguration/extensions`) encima del recurso de Kubernetes con Azure Arc habilitado (representado por `Microsoft.Kubernetes/connectedClusters`) en Azure Resource Manager. La representación de Azure Resource Manager permite crear una directiva que compruebe todos los recursos de Kubernetes con Azure Arc habilitado, tanto si tienen una extensión de clúster específica como si no. Una vez que haya determinado qué clústeres carecen de extensiones con los valores de propiedad deseados, puede corregirlos con Azure Policy.

El componente `config-agent` que se ejecuta en el clúster supervisa los recursos de extensión nuevos o actualizados del recurso de Kubernetes con Azure Arc habilitado. El componente `extensions-manager` que se ejecuta en el clúster extrae el gráfico de Helm de Azure Container Registry o Microsoft Container Registry y lo instala en el clúster. 

Los componentes `config-agent` y `extensions-manager` que se ejecutan en el clúster administran las actualizaciones de versiones y la eliminación de las instancias de las extensiones.

> [!NOTE]
> * `config-agent` supervisa si hay disponibles recursos de configuración nuevos o actualizados en el recurso de Kubernetes con Arc habilitado. Por tanto, estos agentes necesitan tener conectividad para que el estado deseado se lleve al clúster. Si los agentes no pueden conectarse a Azure, se producirá un retraso en la propagación del estado deseado al clúster.
> * Los valores de configuración protegidos de una extensión se guardan durante un máximo de 48 horas en los servicios de Kubernetes con Azure Arc habilitado. Por tanto, si el clúster permanece desconectado durante las 48 horas siguientes a las que se creó el recurso de extensión en Azure, la extensión pasa de tener un estado `Pending` a tener un estado `Failed`. Es recomendable poner los clústeres en línea con la mayor frecuencia posible.

## <a name="next-steps"></a>Pasos siguientes

* Utilice el manual de inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Implemente extensiones de clústeres](./extensions.md) en un clúster de Kubernetes con Azure Arc habilitado.