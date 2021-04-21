---
title: Datos personales
description: Aprenda a administrar los datos personales asociados a operaciones de Azure Resource Manager.
ms.topic: conceptual
ms.date: 05/14/2018
ms.openlocfilehash: 9087d3e46f38aab3de7774ea341ebd9cbc2d7d1f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785962"
---
# <a name="manage-personal-data-associated-with-azure-resource-manager"></a>Administración de datos personales asociados a Azure Resource Manager

Para no exponer información confidencial, elimine toda la información personal que pueda haber proporcionado en implementaciones, grupos de recursos o etiquetas. Azure Resource Manager proporciona operaciones que le permiten administrar los datos personales que haya proporcionado en implementaciones, grupos de recursos o etiquetas.

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="delete-personal-data-in-deployment-history"></a>Eliminación de datos personales en el historial de implementaciones

En el caso de las implementaciones, Resource Manager mantiene los valores de los parámetros y los mensajes de estado en el historial de implementaciones. Estos valores se conservan hasta que se elimina la implementación del historial. Para ver si ha proporcionado datos personales en estos valores, enumere las implementaciones. Si encuentra datos personales, elimine las implementaciones del historial.

Para enumerar las **implementaciones** en el historial, use:

* [Enumerar por grupo de recursos](/rest/api/resources/deployments/listbyresourcegroup)
* [Get-AzResourceGroupDeployment](/powershell/module/az.resources/Get-AzResourceGroupDeployment)
* [az deployment group list](/cli/azure/deployment/group#az_deployment_group_list)

Para eliminar **implementaciones** del historial, use:

* [Eliminar](/rest/api/resources/deployments/delete)
* [Remove-AzResourceGroupDeployment](/powershell/module/az.resources/Remove-AzResourceGroupDeployment)
* [az deployment group delete](/cli/azure/deployment/group#az_deployment_group_delete)

## <a name="delete-personal-data-in-resource-group-names"></a>Eliminación de datos personales en los nombres de los grupos de recursos

El nombre del grupo de recursos se conserva hasta que se elimina el grupo de recursos. Para ver si ha proporcionado datos personales en los nombres, enumere los grupos de recursos. Si encuentra datos personales, [mueva los recursos](move-resource-group-and-subscription.md) a otro grupo de recursos y elimine el grupo de recursos que tenga los datos personales en el nombre.

Para enumerar los **grupos de recursos**, use:

* [Lista](/rest/api/resources/resourcegroups/list)
* [Get-AzResourceGroup](/powershell/module/az.resources/Get-AzResourceGroup)
* [az group list](/cli/azure/group#az_group_list)

Para eliminar los **resource groups**, use:

* [Eliminar](/rest/api/resources/resourcegroups/delete)
* [Remove-AzResourceGroup](/powershell/module/az.resources/Remove-AzResourceGroup)
* [az group delete](/cli/azure/group#az_group_delete)

## <a name="delete-personal-data-in-tags"></a>Eliminación de datos personales de etiquetas

Los nombres y valores de las etiquetas se conservan hasta que se elimina o modifica la etiqueta. Para ver si ha especificado datos personales en las etiquetas, enumérelas. Si encuentra datos personales, elimine las etiquetas.

Para enumerar las **etiquetas**, use:

* [Lista](/rest/api/resources/tags/list)
* [Get-AzTag](/powershell/module/az.resources/Get-AzTag)
* [az tag list](/cli/azure/tag#az_tag_list)

Para eliminar las **etiquetas**, use:

* [Eliminar](/rest/api/resources/tags/delete)
* [Remove-AzTag](/powershell/module/az.resources/Remove-AzTag)
* [az tag delete](/cli/azure/tag#az_tag_delete)

## <a name="next-steps"></a>Pasos siguientes
* Para ver una introducción de Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md)