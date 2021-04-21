---
title: Eliminación del grupo de recursos y los recursos
description: Se describe cómo eliminar grupos de recursos y recursos. También se describe cómo Azure Resource Manager ordena la eliminación de recursos al eliminar un grupo de recursos. Describe los códigos de respuesta y cómo Resource Manager los controla para determinar si la eliminación se realizó correctamente.
ms.topic: conceptual
ms.date: 03/18/2021
ms.custom: seodec18
ms.openlocfilehash: 3c062c2f775e145347129f24b201748ee517daf4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768676"
---
# <a name="azure-resource-manager-resource-group-and-resource-deletion"></a>Eliminación de grupos de recursos y recursos en Azure Resource Manager

En este artículo se muestra cómo eliminar grupos de recursos y recursos. Se describe cómo Azure Resource Manager ordena la eliminación de recursos al eliminar un grupo de recursos.

## <a name="how-order-of-deletion-is-determined"></a>Cómo se determina el orden de eliminación

Cuando se elimina un grupo de recursos, Resource Manager determina el orden para eliminar los recursos. Usa el orden siguiente:

1. Se eliminan todos los recursos secundarios (anidados).

2. Los recursos que administran otros recursos se eliminan a continuación. Un recurso puede tener establecida la propiedad `managedBy` para indicar que un recurso diferente lo administra. Cuando se establece esta propiedad, se elimina el recurso que administra el otro recurso antes que los demás.

3. El resto de los recursos se elimina después de las dos categorías anteriores.

Después de determinar el orden, el Administrador de recursos emite una operación de eliminación para cada recurso. Espera a que todas las dependencias finalicen antes de continuar.

En las operaciones sincrónicas, los códigos de respuesta correcta esperados son:

* 200
* 204
* 404

En las operaciones asincrónicas, la respuesta correcta esperada es 202. Resource Manager realiza el seguimiento del encabezado de ubicación o del encabezado de la operación asincrónica de Azure para determinar el estado de la operación de eliminación asincrónica.
  
### <a name="deletion-errors"></a>Errores de eliminación

Cuando una operación de eliminación devuelve un error, Resource Manager vuelve a intentar la llamada a DELETE. Los reintentos se producen para los códigos de estado 5xx, 429 y 408. De forma predeterminada, el período de tiempo de reintento es de 15 minutos.

## <a name="after-deletion"></a>Después de la eliminación

Resource Manager emite una llamada GET en cada recurso que ha intentado eliminar. Se espera que la respuesta de la llamada sea 404. Cuando Resource Manager obtiene un error 404, considera que la eliminación se ha completado correctamente. Resource Manager quita el recurso de su memoria caché.

Sin embargo, si la llamada a GET en el recurso devuelve una respuesta 200 o 201, Resource Manager vuelve a crear el recurso.

Si la operación GET devuelve un error, Resource Manager vuelve a intentar la operación GET para el código de error siguiente:

* Menor que 100
* 408
* 429
* Mayor que 500

Con otros códigos de error, Resource Manager no puede eliminar el recurso.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible.

## <a name="delete-resource-group"></a>Eliminación de un grupo de recursos

Use uno de los métodos siguientes para eliminar el grupo de recursos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name ExampleResourceGroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group delete --name ExampleResourceGroup
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el [portal](https://portal.azure.com), seleccione el grupo de recursos que quiere eliminar.

1. Seleccione **Eliminar grupo de recursos**.

   ![Eliminación de un grupo de recursos](./media/delete-resource-group/delete-group.png)

1. Para confirmar la eliminación, escriba el nombre del grupo de recursos.

---

## <a name="delete-resource"></a>Eliminación de un recurso

Use uno de los métodos siguientes para eliminar un recurso.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Remove-AzResource `
  -ResourceGroupName ExampleResourceGroup `
  -ResourceName ExampleVM `
  -ResourceType Microsoft.Compute/virtualMachines
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az resource delete \
  --resource-group ExampleResourceGroup \
  --name ExampleVM \
  --resource-type "Microsoft.Compute/virtualMachines"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el [portal](https://portal.azure.com), seleccione el recurso que quiere eliminar.

1. Seleccione **Eliminar**. La captura de pantalla siguiente muestra las opciones de administración de una máquina virtual.

   ![Eliminación de un recurso](./media/delete-resource-group/delete-resource.png)

1. Cuando se le solicite, confirme la eliminación.

---

## <a name="required-access"></a>Acceso necesario

Para eliminar un grupo de recursos, debe tener acceso a la acción de eliminación para el recurso **Microsoft.Resources/subscriptions/resourceGroups**. También debe eliminar todos los recursos del grupo de recursos.

Para obtener una lista de las operaciones, consulte [Operaciones del proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md). Para ver una lista de los roles integrados, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

Si tiene el acceso necesario, pero se produce un error en la solicitud de eliminación, puede deberse a la existencia de un [bloqueo](lock-resources.md) en el grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

* Para comprender los conceptos de Resource Manager, consulte [Información general sobre Azure Resource Manager](overview.md).
* Para los comandos de eliminación, consulte [PowerShell](/powershell/module/az.resources/Remove-AzResourceGroup), [CLI de Azure](/cli/azure/group#az_group_delete), y [API de REST](/rest/api/resources/resourcegroups/delete).
