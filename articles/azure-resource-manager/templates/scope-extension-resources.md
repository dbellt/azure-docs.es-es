---
title: Ámbito en tipos de recursos de extensión
description: Aquí se describe cómo utilizar la propiedad del ámbito al implementar tipos de recursos de extensión.
ms.topic: conceptual
ms.date: 01/13/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 534011658502829df8fa23e44264c8ff2992d3cf
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960015"
---
# <a name="setting-scope-for-extension-resources-in-arm-templates"></a>Establecimiento del ámbito de los recursos de extensión en las plantillas de Resource Manager

Un recurso de extensión es un recurso que modifica otro recurso. Por ejemplo, puede asignar un rol a un recurso. La asignación de roles es un tipo de recurso de extensión.

Para obtener una lista completa de los tipos de recursos de extensión, consulte los [Tipos de recursos que amplían las capacidades de otros recursos](../management/extension-resource-types.md).

En este artículo se muestra cómo establecer el ámbito de un tipo de recurso de extensión cuando se implementa con una plantilla de Azure Resource Manager (plantilla de Resource Manager). Asimismo, describe la propiedad de ámbito que está disponible para los recursos de extensión cuando esta se aplica a un recurso.

> [!NOTE]
> La propiedad scope solo está disponible para los tipos de recursos de extensión. Para especificar un ámbito diferente para un tipo de recurso que no sea un tipo de extensión, use una implementación anidada o vinculada. Para obtener más información, consulte [implementaciones de grupos de recursos](deploy-to-resource-group.md), [implementaciones de suscripciones](deploy-to-subscription.md), [implementaciones de grupos de administración](deploy-to-management-group.md) e [implementaciones de inquilinos](deploy-to-tenant.md).

## <a name="apply-at-deployment-scope"></a>Aplicación en el ámbito de implementación

Para aplicar un tipo de recurso de extensión en el ámbito de implementación de destino, agregue el recurso a la plantilla, tal como haría con cualquier tipo de recurso. Los ámbitos disponibles son el [grupo de recursos](deploy-to-resource-group.md), la [suscripción](deploy-to-subscription.md), el [grupo de administración](deploy-to-management-group.md) y el [inquilino](deploy-to-tenant.md). El ámbito de implementación debe admitir el tipo de recurso.

En la plantilla siguiente se implementa un bloqueo.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/locktargetscope.json":::

Cuando se implementa en un grupo de recursos, bloquea el grupo de recursos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/locktargetscope.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
 New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/locktargetscope.json"
```

---

En el ejemplo siguiente se asigna un rol.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/roletargetscope.json":::

Cuando se implementa en una suscripción, asigna el rol a la suscripción.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment sub create \
  --name demoSubDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/roletargetscope.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzSubscriptionDeployment `
  -Name demoSubDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/roletargetscope.json"
```

---

## <a name="apply-to-resource"></a>Aplicación al recurso

Para aplicar un recurso de extensión a un recurso, use la propiedad `scope`. Establezca la propiedad de ámbito en el nombre del recurso al que va a agregar la extensión. La propiedad de ámbito es una propiedad raíz del tipo de recurso de extensión.

En el ejemplo siguiente se crea una cuenta de almacenamiento y se aplica un rol a la misma.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/storageandrole.json" highlight="56":::

## <a name="next-steps"></a>Pasos siguientes

* Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).
* Para obtener sugerencias para resolver los errores de implementación más comunes, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
* Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla de Resource Manager privada con el token de SAS](secure-template-with-sas-token.md).