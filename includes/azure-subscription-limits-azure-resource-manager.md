---
title: archivo de inclusión
description: archivo de inclusión
services: azure-resource-manager
author: tfitzmac
ms.service: cost-management-billing
ms.topic: include
ms.date: 06/04/2021
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 12571cb0a34df67eab67eb9ebb45c64a95e90bf5
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111761232"
---
| Recurso | Límite |
| --- | --- |
| Suscripciones [asociadas a un inquilino de Azure Active Directory](../articles/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md) | Sin límite |
| [Coadministradores](../articles/cost-management-billing/manage/add-change-subscription-administrator.md) por suscripción |Sin límite |
| [Grupos de recursos](../articles/azure-resource-manager/management/overview.md) por suscripción |980 |
| Tamaño de solicitud de API de Azure Resource Manager |4 194 304 bytes |
| Etiquetas por suscripción<sup>1</sup> |50 |
| Cálculos de etiquetas únicas por suscripción<sup>1</sup> | 80 000 |
| [Implementaciones de nivel de suscripción](../articles/azure-resource-manager/templates/deploy-to-subscription.md) por ubicación | 800<sup>2</sup> |

<sup>1</sup>Puede aplicar hasta 50 etiquetas directamente a una suscripción. Sin embargo, la suscripción puede contener un número ilimitado de etiquetas que se aplican a los grupos de recursos y recursos de la misma. El número de etiquetas por recurso o grupo de recursos se limita a 50. Resource Manager devuelve una [lista de valores y nombres de etiqueta únicos](/rest/api/resources/tags) en la suscripción solo cuando haya 80 000 etiquetas, o menos. Sin embargo, aunque haya más, es posible encontrar un recurso por etiqueta.

<sup>2</sup>Las implementaciones se eliminan automáticamente del historial a medida se aproxima al límite. Para obtener más información, consulte [Eliminaciones automáticas del historial de implementaciones](../articles/azure-resource-manager/templates/deployment-history-deletions.md).