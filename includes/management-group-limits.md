---
title: archivo de inclusión
description: archivo de inclusión
author: tfitzmac
ms.service: governance
ms.topic: include
ms.date: 03/26/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 53e3f37d14153f3a2d7b5886a49b08ca9052b128
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800124"
---
| Resource | Límite |
| --- | --- |
| Grupos de administración por inquilino de Azure AD | 10 000 |
| Suscripciones por grupo de administración | Sin límite. |
| Niveles de jerarquía de grupos de administración | Nivel raíz más 6 niveles<sup>1</sup> |
| Grupo de administración primario directo por grupo de administración | Uno |
| [Implementaciones de nivel de grupo de administración](../articles/azure-resource-manager/templates/deploy-to-management-group.md) por ubicación | 800<sup>2</sup> |

<sup>1</sup>Los 6 niveles no incluyen el nivel de suscripción.

<sup>2</sup>Si se alcanza el límite de 800 implementaciones, elimine las implementaciones que ya no necesite del historial. Para eliminar las implementaciones de nivel de grupo de administración, use [Remove-AzManagementGroupDeployment](/powershell/module/az.resources/Remove-AzManagementGroupDeployment) o [az deployment mg delete](/cli/azure/deployment/mg#az_deployment_mg_delete).
