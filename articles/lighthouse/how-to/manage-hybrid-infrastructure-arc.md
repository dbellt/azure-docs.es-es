---
title: Administración de la infraestructura híbrida a gran escala con Azure Arc
description: Azure Lighthouse ayuda a administrar de forma eficaz las máquinas de sus clientes y los clústeres de Kubernetes fuera de Azure.
ms.date: 03/12/2021
ms.topic: how-to
ms.openlocfilehash: edb9553288ff909e5a5f51bf1440c29914672cb9
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082089"
---
# <a name="manage-hybrid-infrastructure-at-scale-with-azure-arc"></a>Administración de la infraestructura híbrida a gran escala con Azure Arc

Como proveedor de servicios, es posible que haya incorporado varios inquilinos de cliente para [Azure Lighthouse](../overview.md). Azure Lighthouse permite a los proveedores de servicios realizar operaciones a escala a través de varios inquilinos de Azure Active Directory (Azure AD) a la vez, lo que hace que las tareas de administración sean más eficaces.

[Azure Arc](../../azure-arc/overview.md) ayuda a simplificar entornos complejos y distribuidos en entornos locales, perimetrales y multinube, lo que permite implementar los servicios de Azure en cualquier lugar y amplía la administración de Azure a cualquier infraestructura.

Con los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md), los clientes pueden administrar las máquinas Windows y Linux hospedadas fuera de Azure en la red corporativa, del mismo modo que se administran las máquinas virtuales nativas de Azure. Al vincular una máquina híbrida a Azure, esta se conecta y se trata como un recurso de Azure. Después, los proveedores de servicios pueden administrar estas máquinas que no son de Azure junto con los recursos de Azure de sus clientes.

[Kubernetes habilitado para Azure Arc](../../azure-arc/kubernetes/overview.md) permite a los clientes asociar y configurar clústeres de Kubernetes dentro y fuera de Azure. Cuando un clúster de Kubernetes está asociado a Azure Arc, aparecerá en Azure Portal, con un identificador de Azure Resource Manager y una identidad administrada. Los clústeres se asocian a suscripciones estándar de Azure, viven en un grupo de recursos y pueden recibir etiquetas como cualquier otro recurso de Azure.

En este tema se proporciona información general sobre cómo los proveedores de servicios pueden usar los servidores habilitados para Azure Arc y Kubernetes habilitado para Azure Arc de forma escalable para administrar el entorno híbrido de sus clientes, con visibilidad en todos los inquilinos de los clientes administrados.

> [!TIP]
> Aunque en este tema hacemos referencia a proveedores de servicios y clientes, esta guía es aplicable también a [empresas que usan Azure Lighthouse para administrar varios inquilinos](../concepts/enterprise.md).

## <a name="manage-hybrid-servers-at-scale-with-azure-arc-enabled-servers"></a>Administración de servidores híbridos a gran escala con servidores habilitados para Azure Arc

Como proveedor de servicios, puede administrar máquinas Windows Server o Linux locales fuera de Azure que los clientes han conectado a su suscripción mediante el [agente de Azure Connected Machine](../../azure-arc/servers/agent-overview.md).

Al ver los recursos de una suscripción delegada en Azure Portal, verá estas máquinas conectadas etiquetadas con **Azure Arc**. Puede administrar estas máquinas conectadas mediante construcciones de Azure, como Azure Policy y el etiquetado, del mismo modo que administra los recursos de Azure del cliente. También puede trabajar en los inquilinos de los clientes para administrar todas las máquinas híbridas conectadas.

Por ejemplo, puede [asegurarse de que se aplica el mismo conjunto de directivas a las máquinas híbridas de los clientes](../../azure-arc/servers/learn/tutorial-assign-policy-portal.md). También puede usar Azure Security Center para supervisar el cumplimiento en todos los entornos híbridos de sus clientes, o [usar Azure Monitor para recopilar datos directamente de las máquinas híbridas](../../azure-arc/servers/learn/tutorial-enable-vm-insights.md) en un área de trabajo de Log Analytics. Las [extensiones de máquina virtual](../../azure-arc/servers/manage-vm-extensions.md) pueden implementarse en máquinas virtuales Windows y Linux que no sean de Azure, lo que simplifica la administración de las máquinas híbridas del cliente.

## <a name="manage-hybrid-kubernetes-clusters-at-scale-with-azure-arc-enabled-kubernetes"></a>Administración de clústeres híbridos de Kubernetes a gran escala con Kubernetes habilitado para Azure Arc

Puede administrar clústeres de Kubernetes que se han [conectado a la suscripción de un cliente con Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md), como si se estuvieran ejecutando en Azure.

Si el cliente ha creado una [cuenta de entidad de servicio para incorporar clústeres de Kubernetes a Azure Arc](../../azure-arc/kubernetes/create-onboarding-service-principal.md), puede acceder a esta cuenta de entidad de servicio para incorporar y administrar clústeres. Esto lo pueden realizar los usuarios del inquilino de administración a los que se concedió el rol integrado de Azure "Clúster de Kubernetes: incorporación de Azure Arc", cuando la suscripción que contiene la cuenta de entidad de servicio se [incorporó a Azure Lighthouse](onboard-customer.md).

Puede implementar [configuraciones](../../azure-arc/kubernetes/tutorial-use-gitops-connected-cluster.md) y [gráficos de Helm](../../azure-arc/kubernetes/use-gitops-with-helm.md) con GitOps para clústeres conectados.

También puede supervisar los clústeres conectados con Azure Monitor y [usar Azure Policy para aplicar configuraciones de clúster a gran escala](../../azure-arc/kubernetes/use-azure-policy.md).

## <a name="next-steps"></a>Pasos siguientes

- Explore las guías de inicio rápidio y los ejemplos en el [repositorio de Azure Arc en GitHub](https://github.com/microsoft/azure_arc).
- Obtenga información sobre [escenarios compatibles con los servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md#supported-scenarios).
- Obtenga información sobre [distribuciones de Kubernetes compatibles con Azure Arc](../../azure-arc/kubernetes/overview.md#supported-kubernetes-distributions).
- Obtenga información sobre cómo [implementar una directiva a gran escala](policy-at-scale.md).
- Obtenga información sobre cómo [usar registros de Azure Monitor a gran escala](monitor-at-scale.md).