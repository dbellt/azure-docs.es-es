---
title: Creación e implementación de especificaciones de plantillas
description: Describe cómo crear especificaciones de plantilla y compartirlas con otros usuarios de la organización.
ms.topic: conceptual
ms.date: 05/04/2021
ms.author: tomfitz
ms.custom: devx-track-azurepowershell
author: tfitzmac
ms.openlocfilehash: bfbf876f8621beec4d8eddb80c3db285cd6d4c49
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110688292"
---
# <a name="azure-resource-manager-template-specs"></a>Especificaciones de plantilla de Azure Resource Manager

Una especificación de plantilla es un tipo de recurso para almacenar una plantilla de Azure Resource Manager en Azure para su posterior implementación. Este tipo de recurso permite compartir plantillas de ARM con otros usuarios de la organización. Al igual que cualquier otro recurso de Azure, puede usar el control de acceso basado en rol (RBAC) de Azure para compartir la especificación de plantilla.

**Microsoft.Resources/templateSpecs** es el tipo de recurso para las especificaciones de plantilla. Consta de una plantilla principal y un número indefinido de plantillas vinculadas. Azure almacena de forma segura las especificaciones de plantilla en grupos de recursos. Las especificaciones de plantilla son compatibles con el [control de versiones](#versioning).

Para implementar la especificación de plantilla, use herramientas estándar de Azure como PowerShell, la CLI de Azure, Azure Portal, REST, y otros SDK y clientes compatibles. Utilice los mismos comandos que utilizaría para la plantilla.

> [!NOTE]
> Para usar especificaciones de plantilla con Azure PowerShell, debe instalar la [versión 5.0.0 o posterior](/powershell/azure/install-az-ps). Para usarlas con la CLI de Azure, utilice la [versión 2.14.2 o posterior](/cli/azure/install-azure-cli).

## <a name="why-use-template-specs"></a>¿Por qué usar especificaciones de plantilla?

Las especificaciones de plantilla ofrecen las ventajas siguientes:

* Puede usar plantillas de ARM estándar para la especificación de la plantilla.
* Puede administrar el acceso a través de Azure RBAC, en lugar de tokens de SAS.
* Los usuarios pueden implementar la especificación de plantilla sin tener acceso de escritura a la plantilla.
* Puede integrar la especificación de plantilla en el proceso de implementación existente, como el script de PowerShell o la canalización de DevOps.

Las especificaciones de plantilla permiten crear plantillas canónicas y compartirlas con los equipos de la organización. Las especificaciones de plantilla son seguras porque están disponibles para la implementación en Azure Resource Manager, pero no para los usuarios sin el permiso correcto. Los usuarios solo necesitan acceso de lectura a la especificación de plantilla para implementar su plantilla, por lo que puede compartir la plantilla sin permitir que otros la modifiquen.

Si actualmente tiene sus plantillas en un repositorio de GitHub o una cuenta de almacenamiento, se enfrenta a varios desafíos al intentar compartir y usar las plantillas. Para implementar la plantilla, debe hacer que sea accesible públicamente o administrar el acceso con tokens de SAS. Para evitar esta limitación, los usuarios pueden crear copias locales, que con el tiempo divergen de la plantilla original. Las especificaciones de plantilla simplifican el uso compartido de plantillas.

Los administradores de la organización deben comprobar las plantillas que se incluyen en una especificación de plantilla para seguir los requisitos y las instrucciones de la organización.

## <a name="create-template-spec"></a>Crear una especificación de plantilla

En el ejemplo siguiente se muestra una plantilla sencilla para crear una cuenta de almacenamiento en Azure.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ]
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[concat('store', uniquestring(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "kind": "StorageV2",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      }
    }
  ]
}
```

Cuando se crea la especificación de plantilla, los comandos de PowerShell o la CLI se pasan el archivo de plantilla principal. Si la plantilla principal hace referencia a las plantillas vinculadas, los comandos las buscarán y empaquetarán para crear la especificación de plantilla. Para obtener más información, consulte [Creación de una especificación de plantilla con plantillas vinculadas](#create-a-template-spec-with-linked-templates).

Cree una especificación de plantilla mediante:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec -Name storageSpec -Version 1.0a -ResourceGroupName templateSpecsRg -Location westus2 -TemplateFile ./mainTemplate.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json"
```

---

Puede ver todas las especificaciones de plantilla de su suscripción mediante:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts list
```

---

Puede ver los detalles de una especificación de plantilla, incluidas sus versiones con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec -ResourceGroupName templateSpecsRG -Name storageSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts show \
    --name storageSpec \
    --resource-group templateSpecRG \
    --version "1.0a"
```

---

## <a name="deploy-template-spec"></a>Implementación de la especificación de plantilla

Después de crear la especificación de plantilla, los usuarios con acceso de **lectura** a la especificación de plantilla pueden implementarla. Para obtener más información acerca de cómo conceder acceso, consulte [Tutorial: Concesión de acceso de grupo a recursos de Azure mediante Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

Las especificaciones de plantilla se pueden implementar a través del portal, de PowerShell, de la CLI de Azure o como una plantilla vinculada en una implementación de plantilla más grande. Los usuarios de una organización pueden implementar una especificación de plantilla en cualquier ámbito de Azure (grupo de recursos, suscripción, grupo de administración o inquilino).

En lugar de pasar una ruta de acceso o un URI para una plantilla, para implementar una especificación de plantilla puede proporcionar su identificador de recurso. El identificador de recurso tiene el formato siguiente:

**/subscriptions/{id-de-suscripción}/resourceGroups/{grupo-de-recursos}/providers/Microsoft.Resources/templateSpecs/{nombre-de-especificación-de-plantilla}/versions/{versión-de-especificación-de-plantilla}**

Observe que el identificador de recurso incluye un nombre de versión para la especificación de plantilla.

Por ejemplo, puede implementar una especificación de plantilla mediante el siguiente comando.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

En la práctica, normalmente ejecutará `Get-AzTemplateSpec` o `az ts show` para obtener el identificador de la especificación de plantilla que quiere implementar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0a).Versions.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = $(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0a" --query "id")

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

También puede abrir una dirección URL con el formato siguiente para implementar una especificación de plantilla:

```url
https://portal.azure.com/#create/Microsoft.Template/templateSpecVersionId/%2fsubscriptions%2f{subscription-id}%2fresourceGroups%2f{resource-group-name}%2fproviders%2fMicrosoft.Resources%2ftemplateSpecs%2f{template-spec-name}%2fversions%2f{template-spec-version}
```

## <a name="parameters"></a>Parámetros

Pasar parámetros a la especificación de plantilla es exactamente igual que pasar parámetros a una plantilla de Resource Manager. Agregue los valores de parámetro insertados o en un archivo de parámetros.

Para pasar un parámetro insertado, use:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -StorageAccountType Standard_GRS
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters storageAccountType='Standard_GRS'
```

---

Para crear un archivo de parámetros local, use:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountType": {
      "value": "Standard_GRS"
    }
  }
}
```

Y pase el archivo de parámetros con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -TemplateParameterFile ./mainTemplate.parameters.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters "./mainTemplate.parameters.json"
```

---

## <a name="versioning"></a>Control de versiones

Cuando se crea una especificación de plantilla, se debe proporcionar un nombre de versión para ella. A medida que realiza la iteración en el código de plantilla, puede actualizar una versión existente (para revisiones) o publicar una nueva versión. La versión es una cadena de texto. Puede optar por seguir cualquier sistema de control de versiones, incluido el control de versiones semántico. Los usuarios de la especificación de plantilla pueden proporcionar el nombre de versión que quieren usar al implementarla.

## <a name="use-tags"></a>Usar etiquetas

Las [etiquetas](../management/tag-resources.md) le ayudan a organizar los recursos de forma lógica. Puede agregar etiquetas a las especificaciones de plantilla mediante Azure PowerShell y la CLI de Azure:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0a `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateFile ./mainTemplate.json `
  -Tag @{Dept="Finance";Environment="Production"}
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json" \
  --tags Dept=Finance Environment=Production
```

---

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0a `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateFile ./mainTemplate.json `
  -Tag @{Dept="Finance";Environment="Production"}
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts update \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json" \
  --tags Dept=Finance Environment=Production
```

---

Al crear o modificar una especificación de plantilla con el parámetro "version" especificado, pero sin el parámetro "tag/tags":

- Si la especificación de la plantilla existe y tiene etiquetas, pero la versión no existe, la nueva versión hereda las mismas etiquetas que la especificación de plantilla existente.

Al crear o modificar una especificación de plantilla con los parámetros "tag/tags" y "version" especificados:

- Si no existen la especificación de plantilla ni la versión, las etiquetas se agregan a la nueva especificación de plantilla y a la nueva versión.
- Si existe la especificación de plantilla, pero la versión no, las etiquetas solo se agregan a la nueva versión.
- Si existen la especificación de plantilla y la versión, las etiquetas solo se aplican a la versión.

Al modificar una plantilla con el parámetro "tag/tags" especificado pero sin el parámetro "version" especificado, las etiquetas solo se agregan a la especificación de plantilla.

## <a name="create-a-template-spec-with-linked-templates"></a>Creación de una especificación de plantilla con plantillas vinculadas

Si la plantilla principal de la especificación de plantilla hace referencia a plantillas vinculadas, los comandos de PowerShell y de la CLI pueden buscar y empaquetar automáticamente las plantillas vinculadas desde la unidad local. No es necesario configurar manualmente las cuentas de almacenamiento ni los repositorios para hospedar las especificaciones de plantilla: todo es independiente en el recurso de especificación de plantilla.

El siguiente ejemplo consta de una plantilla principal con dos plantillas vinculadas. El ejemplo es solo un extracto de la plantilla. Observe que usa una propiedad denominada `relativePath` para la vinculación a las otras plantillas. Debe usar `apiVersion` `2020-06-01` o posterior para el recurso de implementaciones.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/webapp.json"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/database.json"
        }
      }
    }
  ],
  "outputs": {}
}
```

Cuando se ejecuta el comando de PowerShell o de la CLI para crear la especificación de plantilla del ejemplo anterior, el comando busca tres archivos: la plantilla principal, la plantilla de aplicación web (`webapp.json`) y la plantilla de base de datos (`database.json`), y las empaqueta en la especificación de plantilla.

Para más información, consulte el [Tutorial: Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).

## <a name="deploy-template-spec-as-a-linked-template"></a>Implementación de una especificación de plantilla como una plantilla vinculada

Una vez que haya creado una especificación de plantilla, es fácil reutilizarla desde una plantilla de ARM u otra especificación de plantilla. Para la vinculación a una especificación de plantilla, agregue su identificador de recurso a la plantilla. La especificación de plantilla vinculada se implementa automáticamente al implementar la plantilla principal. Este comportamiento permite desarrollar especificaciones de plantilla modulares y reutilizarlas según sea necesario.

Por ejemplo, puede crear una especificación de plantilla que implemente recursos de red y otra especificación de plantilla que implemente recursos de almacenamiento. En las plantillas de ARM, debe vincular estas dos especificaciones de plantilla cada vez que necesite configurar redes o recursos de almacenamiento.

El ejemplo siguiente es similar al anterior, pero se usa la propiedad `id` para la vinculación a una especificación de plantilla en lugar de la propiedad `relativePath` para la vinculación a una plantilla local. Use `2020-06-01` para la versión de API del recurso de implementaciones. En el ejemplo, las especificaciones de plantilla se encuentran en un grupo de recursos denominado **templateSpecsRG**.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "networkingDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'networkingSpec', '1.0a')]"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "storageDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0a')]"
        }
      }
    }
  ],
  "outputs": {}
}
```

Para obtener más información acerca de la vinculación de especificaciones de plantilla, consulte [Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada](template-specs-deploy-linked-template.md).

## <a name="next-steps"></a>Pasos siguientes

* Para crear e implementar una especificación de plantilla, consulte [Inicio rápido: Creación e implementación de una especificación de plantilla](quickstart-create-template-specs.md).

* Para obtener más información sobre la vinculación de plantillas en las especificaciones de plantilla, consulte [Tutorial: Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).

* Para obtener más información sobre la implementación de una especificación de plantilla como una plantilla vinculada, consulte [Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada](template-specs-deploy-linked-template.md).
