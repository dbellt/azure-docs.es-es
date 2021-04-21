---
title: Controles de Cumplimiento normativo de Azure Policy para Azure Cognitive Search
description: En este artículo se muestran los controles de Cumplimiento normativo de Azure Policy disponibles para Azure Cognitive Search. Estas definiciones de directivas integradas proporcionan enfoques comunes para administrar el cumplimiento de los recursos de Azure.
ms.date: 04/14/2021
ms.topic: sample
author: HeidiSteen
ms.author: heidist
ms.service: search
ms.custom: subject-policy-compliancecontrols
ms.openlocfilehash: 807201d5626bcdbc421d182f06b888c5775dcc48
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107497430"
---
# <a name="azure-policy-regulatory-compliance-controls-for-azure-cognitive-search"></a>Controles de Cumplimiento normativo de Azure Policy para Azure Cognitive Search

Si usa [Azure Policy](../governance/policy/overview.md) para hacer cumplir las recomendaciones de [Azure Security Benchmark](../security/benchmarks/introduction.md),es probable que sepa que puede crear directivas para identificar los servicios que no sean compatibles y repararlos. Estas directivas pueden ser personalizadas, o bien se pueden basar en definiciones integradas que proporcionan criterios de cumplimiento y soluciones apropiadas para procedimientos recomendados bien entendidos.

En el caso de Azure Cognitive Search, actualmente hay una definición integrada, que se muestra a continuación, que puede usar en una asignación de directiva. La definición integrada es para el registro y la supervisión. Si usa esta definición en una [directiva que cree](../governance/policy/assign-policy-portal.md), el sistema examinará los servicios de búsqueda que no tengan [registro de diagnóstico](search-monitor-logs.md)y, después, los habilitará en consecuencia.

El [cumplimiento normativo de Azure Policy](../governance/policy/concepts/regulatory-compliance.md) proporciona definiciones de iniciativas creadas y administradas por Microsoft, conocidas como _definiciones integradas_, para los **dominios de cumplimiento** y **controles de seguridad** relacionados con diferentes estándares de cumplimiento. En esta página se enumeran los **dominios de cumplimiento** y los **controles de seguridad** para Azure Cognitive Search. Para que los recursos de Azure sean compatibles con el estándar específico, puede asignar las integraciones a un **control de seguridad** de manera individual.

[!INCLUDE [azure-policy-compliancecontrols-introwarning](../../includes/policy/standards/intro-warning.md)]

[!INCLUDE [azure-policy-compliancecontrols-search](../../includes/policy/standards/byrp/microsoft.search.md)]

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [cumplimiento normativo de Azure Policy](../governance/policy/concepts/regulatory-compliance.md).
- Los elementos integrados se pueden encontrar en el [repositorio de GitHub de Azure Policy](https://github.com/Azure/azure-policy).
