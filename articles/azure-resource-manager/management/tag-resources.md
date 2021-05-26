---
title: Etiquetado de recursos, grupos de recursos y suscripciones para una organización lógica
description: Muestra cómo aplicar etiquetas para organizar los recursos de Azure para la facturación y administración.
ms.topic: conceptual
ms.date: 05/05/2021
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 0106ae6bda21544629616f2268ccc111c1946b47
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110480312"
---
# <a name="use-tags-to-organize-your-azure-resources-and-management-hierarchy"></a>Uso de etiquetas para organizar los recursos de Azure y la jerarquía de administración

Puede aplicar etiquetas a los recursos, grupos de recursos y suscripciones de Azure con el fin de organizarlos de manera lógica en una taxonomía. Cada etiqueta consta de un nombre y un par de valores. Por ejemplo, puede aplicar el nombre _Entorno_ y el valor _Producción_ a todos los recursos de producción.

Para recomendaciones sobre cómo implementar una estrategia de etiquetado, consulte [Guía de decisiones de nomenclatura y etiquetado de recursos](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).

> [!IMPORTANT]
> Las nombres de etiqueta no distinguen mayúsculas de minúsculas para las operaciones. Una etiqueta con un nombre de etiqueta, independientemente del uso de mayúsculas o minúsculas, se actualiza o se recupera. Sin embargo, el proveedor de recursos podría mantener el uso de mayúsculas y minúsculas para el nombre de la etiqueta. Puede ver dicho uso de mayúsculas y minúsculas en los informes de costos.
>
> Los valores de etiqueta distinguen mayúsculas de minúsculas.

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="required-access"></a>Acceso necesario

Hay dos maneras de obtener el acceso necesario a los recursos de etiqueta.

- Puede tener acceso de escritura al tipo de recurso `Microsoft.Resources/tags`. Este acceso permite etiquetar cualquier recurso, incluso si no tiene acceso al propio recurso. El rol [Colaborador de etiquetas](../../role-based-access-control/built-in-roles.md#tag-contributor) concede este acceso. Actualmente, el rol colaborador de etiquetas no puede aplicar etiquetas a recursos o grupos de recursos a través del portal. Puede aplicar etiquetas a las suscripciones a través del portal. Este admite todas las operaciones de etiqueta a través de PowerShell y la API REST.

- Puede tener acceso de escritura al propio recurso. El rol [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) concede el acceso necesario para aplicar etiquetas a cualquier entidad. Para aplicar etiquetas a un solo tipo de recurso, use el rol de colaborador para ese recurso. Por ejemplo, para aplicar etiquetas a las máquinas virtuales, use [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

## <a name="powershell"></a>PowerShell

### <a name="apply-tags"></a>Aplicación de etiquetas

Azure PowerShell ofrece dos comandos para aplicar etiquetas: [New-AzTag](/powershell/module/az.resources/new-aztag) y [Update-AzTag](/powershell/module/az.resources/update-aztag). Debe tener el módulo `Az.Resources` 1.12.0 o una versión posterior. Puede consultar su versión con `Get-InstalledModule -Name Az.Resources`. Puede instalar ese módulo o [Azure PowerShell](/powershell/azure/install-az-ps) 3.6.1 o una versión posterior.

`New-AzTag` reemplaza todas las etiquetas en el recurso, el grupo de recursos o la suscripción. Al llamar al comando, pase el id. de recurso de la entidad que desea etiquetar.

En el ejemplo siguiente se aplica un conjunto de etiquetas a una cuenta de almacenamiento:

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
New-AzTag -ResourceId $resource.id -Tag $tags
```

Cuando se complete el comando, observe que el recurso tiene dos etiquetas.

```output
Properties :
        Name    Value
        ======  =======
        Dept    Finance
        Status  Normal
```

Si vuelve a ejecutar el comando, pero esta vez con etiquetas diferentes, observe que se quitaron las etiquetas anteriores.

```azurepowershell-interactive
$tags = @{"Team"="Compliance"; "Environment"="Production"}
New-AzTag -ResourceId $resource.id -Tag $tags
```

```output
Properties :
        Name         Value
        ===========  ==========
        Environment  Production
        Team         Compliance
```

Para agregar etiquetas a un recurso que ya tiene etiquetas, use `Update-AzTag`. Establezca el parámetro `-Operation` en `Merge`.

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

Observe que las dos etiquetas nuevas se agregaron a las dos etiquetas existentes.

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Normal
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Cada nombre de etiqueta solo puede tener un valor. Si proporciona un valor nuevo para una etiqueta, el valor anterior se reemplaza incluso si se usa la operación de combinación. En el ejemplo siguiente la etiqueta `Status` cambia de _Normal_ a _Green_ (Verde).

```azurepowershell-interactive
$tags = @{"Status"="Green"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Green
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Al establecer el parámetro `-Operation` en `Replace`, el nuevo conjunto de etiquetas reemplaza a las etiquetas existentes.

```azurepowershell-interactive
$tags = @{"Project"="ECommerce"; "CostCenter"="00123"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Replace
```

Solo las etiquetas nuevas siguen en el recurso.

```output
Properties :
        Name        Value
        ==========  =========
        CostCenter  00123
        Team        Web
        Project     ECommerce
```

Los mismos comandos también funcionan con grupos de recursos o suscripciones. Puede pasar el identificador del grupo de recursos o de la suscripción que quiere etiquetar.

Para agregar un nuevo conjunto de etiquetas a un grupo de recursos, use:

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags
```

Para actualizar las etiquetas de un grupo de recursos, use:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Production"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Update-AzTag -ResourceId $resourceGroup.ResourceId -Tag $tags -Operation Merge
```

Para agregar un nuevo conjunto de etiquetas a una suscripción, use:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Dev"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
New-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags
```

Para actualizar las etiquetas de una suscripción, use:

```azurepowershell-interactive
$tags = @{"Team"="Web Apps"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Update-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags -Operation Merge
```

Puede tener más de un recurso con el mismo nombre en un grupo de recursos. En ese caso, puede establecer cada recurso con los siguientes comandos:

```azurepowershell-interactive
$resource = Get-AzResource -ResourceName sqlDatabase1 -ResourceGroupName examplegroup
$resource | ForEach-Object { Update-AzTag -Tag @{ "Dept"="IT"; "Environment"="Test" } -ResourceId $_.ResourceId -Operation Merge }
```

### <a name="list-tags"></a>Lista de etiquetas

Para obtener las etiquetas de un recurso, un grupo de recursos o una suscripción, use el comando [Get-AzTag](/powershell/module/az.resources/get-aztag) y pase el id. de recurso de la entidad.

Para ver las etiquetas de un recurso, use:

```azurepowershell-interactive
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
Get-AzTag -ResourceId $resource.id
```

Para ver las etiquetas de un grupo de recursos, use:

```azurepowershell-interactive
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Get-AzTag -ResourceId $resourceGroup.ResourceId
```

Para ver las etiquetas de una suscripción, use:

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Get-AzTag -ResourceId "/subscriptions/$subscription"
```

### <a name="list-by-tag"></a>Enumerar por etiqueta

Para obtener recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurepowershell-interactive
(Get-AzResource -Tag @{ "CostCenter"="00123"}).Name
```

Para obtener recursos que tengan un nombre de etiqueta específico y cualquier valor de etiqueta, use:

```azurepowershell-interactive
(Get-AzResource -TagName "Dept").Name
```

Para obtener grupos de recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurepowershell-interactive
(Get-AzResourceGroup -Tag @{ "CostCenter"="00123" }).ResourceGroupName
```

### <a name="remove-tags"></a>Eliminación de etiquetas

Para quitar etiquetas específicas, use `Update-AzTag` y establezca `-Operation` en `Delete`. Pase las etiquetas que quiere eliminar.

```azurepowershell-interactive
$removeTags = @{"Project"="ECommerce"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $removeTags -Operation Delete
```

Se quitan las etiquetas especificadas.

```output
Properties :
        Name        Value
        ==========  =====
        CostCenter  00123
```

Para quitar todas las etiquetas, use el comando [Remove-AzTag](/powershell/module/az.resources/remove-aztag).

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Remove-AzTag -ResourceId "/subscriptions/$subscription"
```

## <a name="azure-cli"></a>Azure CLI

### <a name="apply-tags"></a>Aplicación de etiquetas

La CLI de Azure ofrece dos comandos para aplicar etiquetas: [az tag create](/cli/azure/tag#az_tag_create) y [az tag update](/cli/azure/tag#az_tag_update). Debe tener la CLI de Azure 2.10.0 o una versión posterior. Puede consultar su versión con `az version`. Para actualizar o instalar, consulte [Instalación de CLI de Azure](/cli/azure/install-azure-cli).

`az tag create` reemplaza todas las etiquetas en el recurso, el grupo de recursos o la suscripción. Al llamar al comando, pase el id. de recurso de la entidad que desea etiquetar.

En el ejemplo siguiente se aplica un conjunto de etiquetas a una cuenta de almacenamiento:

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag create --resource-id $resource --tags Dept=Finance Status=Normal
```

Cuando se complete el comando, observe que el recurso tiene dos etiquetas.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Status": "Normal"
  }
},
```

Si vuelve a ejecutar el comando, pero esta vez con etiquetas diferentes, observe que se quitaron las etiquetas anteriores.

```azurecli-interactive
az tag create --resource-id $resource --tags Team=Compliance Environment=Production
```

```output
"properties": {
  "tags": {
    "Environment": "Production",
    "Team": "Compliance"
  }
},
```

Para agregar etiquetas a un recurso que ya tiene etiquetas, use `az tag update`. Establezca el parámetro `--operation` en `Merge`.

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Dept=Finance Status=Normal
```

Observe que las dos etiquetas nuevas se agregaron a las dos etiquetas existentes.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Normal",
    "Team": "Compliance"
  }
},
```

Cada nombre de etiqueta solo puede tener un valor. Si proporciona un valor nuevo para una etiqueta, el valor anterior se reemplaza incluso si se usa la operación de combinación. En el ejemplo siguiente la etiqueta `Status` cambia de _Normal_ a _Green_ (Verde).

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Status=Green
```

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Green",
    "Team": "Compliance"
  }
},
```

Al establecer el parámetro `--operation` en `Replace`, el nuevo conjunto de etiquetas reemplaza a las etiquetas existentes.

```azurecli-interactive
az tag update --resource-id $resource --operation Replace --tags Project=ECommerce CostCenter=00123 Team=Web
```

Solo las etiquetas nuevas siguen en el recurso.

```output
"properties": {
  "tags": {
    "CostCenter": "00123",
    "Project": "ECommerce",
    "Team": "Web"
  }
},
```

Los mismos comandos también funcionan con grupos de recursos o suscripciones. Puede pasar el identificador del grupo de recursos o de la suscripción que quiere etiquetar.

Para agregar un nuevo conjunto de etiquetas a un grupo de recursos, use:

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag create --resource-id $group --tags Dept=Finance Status=Normal
```

Para actualizar las etiquetas de un grupo de recursos, use:

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags CostCenter=00123 Environment=Production
```

Para agregar un nuevo conjunto de etiquetas a una suscripción, use:

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag create --resource-id /subscriptions/$sub --tags CostCenter=00123 Environment=Dev
```

Para actualizar las etiquetas de una suscripción, use:

```azurecli-interactive
az tag update --resource-id /subscriptions/$sub --operation Merge --tags Team="Web Apps"
```

### <a name="list-tags"></a>Lista de etiquetas

Para obtener las etiquetas de un recurso, un grupo de recursos o una suscripción, use el comando [az tag list](/cli/azure/tag#az_tag_list) y pase el id. de recurso de la entidad.

Para ver las etiquetas de un recurso, use:

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag list --resource-id $resource
```

Para ver las etiquetas de un grupo de recursos, use:

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag list --resource-id $group
```

Para ver las etiquetas de una suscripción, use:

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag list --resource-id /subscriptions/$sub
```

### <a name="list-by-tag"></a>Enumerar por etiqueta

Para obtener recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurecli-interactive
az resource list --tag CostCenter=00123 --query [].name
```

Para obtener recursos que tengan un nombre de etiqueta específico y cualquier valor de etiqueta, use:

```azurecli-interactive
az resource list --tag Team --query [].name
```

Para obtener grupos de recursos que tengan un nombre y valor de etiqueta específicos, use:

```azurecli-interactive
az group list --tag Dept=Finance
```

### <a name="remove-tags"></a>Eliminación de etiquetas

Para quitar etiquetas específicas, use `az tag update` y establezca `--operation` en `Delete`. Pase las etiquetas que quiere eliminar.

```azurecli-interactive
az tag update --resource-id $resource --operation Delete --tags Project=ECommerce Team=Web
```

Se quitan las etiquetas especificadas.

```output
"properties": {
  "tags": {
    "CostCenter": "00123"
  }
},
```

Para quitar todas las etiquetas, use el comando [az tag delete](/cli/azure/tag#az_tag_delete).

```azurecli-interactive
az tag delete --resource-id $resource
```

### <a name="handling-spaces"></a>Control de los espacios

Si los nombres o valores de etiqueta incluyen espacios, escríbalos entre comillas dobles.

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags "Cost Center"=Finance-1222 Location="West US"
```

## <a name="arm-templates"></a>Plantillas de ARM

Puede etiquetar recursos, grupos de recursos y suscripciones durante la implementación con una plantilla de Azure Resource Manager (plantilla de ARM).

> [!NOTE]
> Las etiquetas que se aplican mediante la plantilla de ARM o el archivo de Bicep sobrescriben las existentes.

### <a name="apply-values"></a>Aplicación de valores

En el ejemplo siguiente se implementa una cuenta de almacenamiento con tres etiquetas. Dos de las etiquetas (`Dept` y `Environment`) se establecen en valores literales. Una etiqueta (`LastDeployed`) se establece en un parámetro que tiene como valor predeterminado la fecha actual.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "utcShort": {
      "type": "string",
      "defaultValue": "[utcNow('d')]"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[concat('storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production",
        "LastDeployed": "[parameters('utcShort')]"
      },
      "properties": {}
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param location string = resourceGroup().location
param utcShort string = utcNow('d')

resource stgAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'storage${uniqueString(resourceGroup().id)}'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  tags: {
    Dept: 'Finance'
    Environment: 'Production'
    LastDeployed: utcShort
  }
}
```

---

### <a name="apply-an-object"></a>Aplicación de un objeto

Puede definir un parámetro de objeto que almacene varias etiquetas y aplicar ese objeto al elemento de etiqueta. Este enfoque proporciona más flexibilidad que el ejemplo anterior, porque el objeto puede tener propiedades diferentes. Cada propiedad del objeto se convierte en una etiqueta independiente para el recurso. El siguiente ejemplo tiene un parámetro denominado `tagValues` que se aplica al elemento de etiqueta.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "tagValues": {
      "type": "object",
      "defaultValue": {
        "Dept": "Finance",
        "Environment": "Production"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[concat('storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "tags": "[parameters('tagValues')]",
      "properties": {}
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param location string = resourceGroup().location
param tagValues object = {
  Dept: 'Finance'
  Environment: 'Production'
}

resource stgAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'storage${uniqueString(resourceGroup().id)}'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  tags: tagValues
}
```

---

### <a name="apply-a-json-string"></a>Aplicación de una cadena JSON

Para almacenar muchos valores en una única etiqueta, aplique una cadena JSON que represente los valores. Toda la cadena JSON se almacena como una etiqueta que no puede superar los 256 caracteres. En el ejemplo siguiente se muestra una etiqueta denominada `CostCenter` que contiene varios valores de una cadena JSON:

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[concat('storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "tags": {
        "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
      },
      "properties": {}
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param location string = resourceGroup().location

resource stgAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'storage${uniqueString(resourceGroup().id)}'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  tags: {
    CostCenter: '{"Dept":"Finance","Environment":"Production"}'
  }
}
```

---

### <a name="apply-tags-from-resource-group"></a>Aplicación de etiquetas del grupo de recursos

Para aplicar etiquetas desde un grupo de recursos a un recurso, use la función [resourceGroup()](../templates/template-functions-resource.md#resourcegroup). Cuando obtenga el valor de la etiqueta, use la sintaxis `tags[tag-name]` en lugar de la sintaxis `tags.tag-name`, porque algunos caracteres no se analizan correctamente en la notación de puntos.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[concat('storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "tags": {
        "Dept": "[resourceGroup().tags['Dept']]",
        "Environment": "[resourceGroup().tags['Environment']]"
      },
      "properties": {}
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param location string = resourceGroup().location

resource stgAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'storage${uniqueString(resourceGroup().id)}'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  tags: {
    Dept: resourceGroup().tags['Dept']
    Environment: resourceGroup().tags['Environment']
  }
}
```

---

### <a name="apply-tags-to-resource-groups-or-subscriptions"></a>Aplicación de etiquetas a grupos de recursos o suscripciones

Puede agregar etiquetas a un grupo de recursos o una suscripción mediante la implementación del tipo de recurso `Microsoft.Resources/tags`. Las etiquetas se aplican al grupo de recursos o a la suscripción de destino correspondiente a la implementación. Cada vez que implemente la plantilla, reemplace cualquier etiqueta que haya aplicado anteriormente.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "tagName": {
      "type": "string",
      "defaultValue": "TeamName"
    },
    "tagValue": {
      "type": "string",
      "defaultValue": "AppTeam1"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/tags",
      "name": "default",
      "apiVersion": "2021-04-01",
      "properties": {
        "tags": {
          "[parameters('tagName')]": "[parameters('tagValue')]"
        }
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param tagName string = 'TeamName'
param tagValue string = 'AppTeam1'

resource applyTags 'Microsoft.Resources/tags@2021-04-01' = {
  name: 'default'
  properties: {
    tags: {
      '${tagName}': tagValue
    }
  }
}
```

---

Para aplicar las etiquetas a un grupo de recursos, use PowerShell o la CLI de Azure. Implemente en el grupo de recursos que quiere etiquetar.

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName exampleGroup -TemplateFile https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment group create --resource-group exampleGroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Para aplicar las etiquetas a una suscripción, use PowerShell o la CLI de Azure. Implemente en la suscripción que quiere etiquetar.

```azurepowershell-interactive
New-AzSubscriptionDeployment -name tagresourcegroup -Location westus2 -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment sub create --name tagresourcegroup --location westus2 --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Para más información sobre las implementaciones de suscripciones, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](../templates/deploy-to-subscription.md).

La plantilla siguiente agrega las etiquetas de un objeto a un grupo de recursos o a una suscripción.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "tags": {
      "type": "object",
      "defaultValue": {
        "TeamName": "AppTeam1",
        "Dept": "Finance",
        "Environment": "Production"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/tags",
      "apiVersion": "2021-04-01",
      "name": "default",
      "properties": {
        "tags": "[parameters('tags')]"
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
targetScope = 'subscription'

param tagObject object = {
  TeamName: 'AppTeam1'
  Dept: 'Finance'
  Environment: 'Production'
}

resource applyTags 'Microsoft.Resources/tags@2021-04-01' = {
  name: 'default'
  properties: {
    tags: tagObject
  }
}
```

---

## <a name="portal"></a>Portal

[!INCLUDE [resource-manager-tag-resource](../../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>API DE REST

Para trabajar con etiquetas a través de la API REST de Azure, use:

* [Etiquetas: crear o actualizar en el ámbito](/rest/api/resources/tags/createorupdateatscope) (operación PUT)
* [Etiquetas: actualizar en el ámbito](/rest/api/resources/tags/updateatscope) (operación PATCH)
* [Etiquetas: obtener en el ámbito](/rest/api/resources/tags/getatscope) (operación GET)
* [Etiquetas: eliminar en el ámbito](/rest/api/resources/tags/deleteatscope) (operación DELETE)

## <a name="inherit-tags"></a>Herencia de etiquetas

Los recursos no heredan las etiquetas aplicadas al grupo de recursos ni a la suscripción. Para aplicar etiquetas de un grupo de recursos o una suscripción a los recursos, consulte [Azure Policies: etiquetas](tag-policies.md).

## <a name="tags-and-billing"></a>Etiquetas y facturación

Puede usar etiquetas a fin de agrupar los datos de facturación. Por ejemplo, si va a ejecutar varias máquinas virtuales para distintas organizaciones, use las etiquetas para agrupar el uso por centro de costo. También puede usar etiquetas para clasificar los costos por entorno de tiempo de ejecución; por ejemplo, el uso de facturación en máquinas virtuales que se ejecutan en el entorno de producción.

Puede recuperar información sobre las etiquetas si descarga el archivo de uso, un archivo de valores separados por comas (CSV) disponible en Azure Portal. Para más información, consulte [Procedimiento para descargar las datos de uso diario y de factura de Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md). En los servicios que admiten etiquetas con facturación, las etiquetas aparecen en la columna **Etiquetas**.

Para las operaciones de API de REST, vea [Referencia de API de REST de facturación de Azure](/rest/api/billing/).

## <a name="limitations"></a>Limitaciones

Se aplican las siguientes limitaciones a las etiquetas:

* No todos los tipos de recursos admiten etiquetas. Para determinar si se puede aplicar una etiqueta a un tipo de recurso determinado, consulte [Tag support for Azure resources](tag-support.md) (Compatibilidad con etiquetas para los recursos de Azure).
* Cada recurso, grupo de recursos y suscripción puede tener un máximo de 50 pares nombre-valor de etiqueta. Si necesita aplicar más etiquetas que el número máximo permitido, use una cadena JSON para el valor de etiqueta. La cadena JSON puede contener muchos valores que se aplican a un sol nombre de etiqueta. Un grupo de recursos o una suscripción puede contener muchos recursos con 50 pares clave-valor de etiqueta cada uno.
* El nombre de etiqueta está limitado a 512 caracteres y el valor de la etiqueta, a 256. En las cuentas de almacenamiento, el nombre de etiqueta se limita a 128 caracteres y el valor de la etiqueta, a 256.
* No se pueden aplicar etiquetas a recursos clásicos como Cloud Services.
* Los nombres de etiqueta no pueden contener estos caracteres: `<`, `>`, `%`, `&`, `\`, `?`, `/`

   > [!NOTE]
   > * Las zonas Azure DNS y Traffic Manager no admiten el uso de espacios en la etiqueta o una etiqueta que empieza por un número.
   >
   > * Azure Front Door no admite el uso de `#` en el nombre de la etiqueta.
   >
   > * Azure Automation y Azure CDN solo admiten 15 etiquetas en los recursos.

## <a name="next-steps"></a>Pasos siguientes

* No todos los tipos de recursos admiten etiquetas. Para determinar si se puede aplicar una etiqueta a un tipo de recurso determinado, consulte [Tag support for Azure resources](tag-support.md) (Compatibilidad con etiquetas para los recursos de Azure).
* Para recomendaciones sobre cómo implementar una estrategia de etiquetado, consulte [Guía de decisiones de nomenclatura y etiquetado de recursos](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).
