---
title: Creación de un proveedor de recursos
description: Se describe cómo crear un proveedor de recursos e implementar sus tipos de recursos personalizados.
author: MSEvanhi
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: evanhi
ms.openlocfilehash: 915856e3f9875c0a3318bc19138eaae8742bf72f
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107871748"
---
# <a name="quickstart-create-a-custom-provider-and-deploy-custom-resources"></a>Inicio rápido: Creación de un proveedor de recursos e implementación de recursos personalizados

En este inicio rápido, creará su propio proveedor de recursos e implementará distintos tipos de recursos personalizados para dicho proveedor de recursos. Para más información sobre los proveedores personalizados, vea la [introducción a los proveedores personalizados de Azure en versión preliminar](overview.md).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Para completar los pasos de este inicio rápido, es preciso llamar a operaciones `REST`. Hay [distintas formas de enviar solicitudes REST](/rest/api/azure/).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Prepare el entorno para la CLI de Azure.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

Los ejemplos de la CLI de Azure usan `az rest` para las solicitudes de `REST`. Para más información, consulte [az rest](/cli/azure/reference-index#az_rest).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Los comandos de PowerShell se ejecutan de forma local con PowerShell 7 o posterior, y los módulos de Azure PowerShell. Para más información, vea [Instalar Azure PowerShell](/powershell/azure/install-az-ps).
- Si no dispone de una herramienta para las operaciones `REST`, instale [ARMClient](https://github.com/projectkudu/ARMClient). Se trata de una herramienta de línea de comandos de código abierto que simplifica la invocación de la API de Azure Resource Manager.
- Una vez instalado el **ARMClient**, puede mostrar la información de uso de un símbolo del sistema de PowerShell si escribe `armclient.exe`. O bien, vaya a la [wiki de ARMClient](https://github.com/projectkudu/ARMClient/wiki).

---

## <a name="deploy-custom-provider"></a>Implementación de un proveedor personalizado

Para configurar el proveedor personalizado, implemente una [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/custom-providers/customprovider.json) en su suscripción de Azure.

Después de implementar la plantilla, la suscripción tiene los siguientes recursos:

- Aplicación de función con las operaciones para los recursos y las acciones.
- Cuenta de almacenamiento para almacenar los usuarios que se crean mediante el proveedor personalizado.
- Proveedor personalizado que define las acciones y los tipos de recursos personalizados. Usa el punto de conexión de la aplicación de función para enviar las solicitudes.
- Recurso personalizado del proveedor personalizado.

Para implementar el proveedor personalizado, use la CLI de Azure, PowerShell o Azure Portal:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En este ejemplo se le pide que especifique un grupo de recursos, la ubicación y el nombre de la aplicación de función del proveedor. Los nombres se almacenan en variables que se usan en otros comandos. Los comandos [az group create](/cli/azure/group#az_group_create) y [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create) implementan los recursos.

```azurecli-interactive
read -p "Enter a resource group name:" rgName &&
read -p "Enter the location (i.e. eastus):" location &&
read -p "Enter the provider's function app name:" funcName &&
templateUri="https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json" &&
az group create --name $rgName --location "$location" &&
az deployment group create --resource-group $rgName --template-uri $templateUri --parameters funcName=$funcName &&
echo "Press [ENTER] to continue ..." &&
read
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En este ejemplo se le pide que especifique un grupo de recursos, la ubicación y el nombre de la aplicación de función del proveedor. Los nombres se almacenan en variables que se usan en otros comandos. Los comandos [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) y [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) implementan los recursos.

```powershell
$rgName = Read-Host -Prompt "Enter a resource group name"
$location = Read-Host -Prompt "Enter the location (i.e. eastus)"
$funcName = Read-Host -Prompt "Enter the provider's function app name"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json"
New-AzResourceGroup -Name $rgName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $rgName -TemplateUri $templateUri -funcName $funcName
Read-Host -Prompt "Press [ENTER] to continue ..."
```

---

También puede implementar la solución desde Azure Portal. Haga clic en el botón **Implementar en Azure** para abrir la plantilla en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-docs-json-samples%2Fmaster%2Fcustom-providers%2Fcustomprovider.json)

## <a name="view-custom-provider-and-resource"></a>Ver recurso y proveedor personalizado

En el portal, el proveedor personalizado es un tipo de recurso oculto. Para confirmar que se ha implementado el proveedor de recursos, vaya al grupo de recursos. Seleccione la opción **Mostrar tipos ocultos**.

![Mostrar tipos de recursos ocultos](./media/create-custom-provider/show-hidden.png)

Para ver el tipo de recurso personalizado que ha implementado, use la operación `GET` en el tipo de recurso.

```http
GET https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
subID=$(az account show --query id --output tsv)
requestURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"
az rest --method get --uri $requestURI
```

Recibirá la respuesta:

```json
{
  "value": [
    {
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "properties": {
        "FullName": "Santa Claus",
        "Location": "NorthPole",
        "provisioningState": "Succeeded"
      },
      "resourceGroup": "<rg-name>",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$subID = (Get-AzContext).Subscription.Id
$requestURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"

armclient GET $requestURI
```

Recibirá la respuesta:

```json
{
  "value": [
    {
      "properties": {
        "provisioningState": "Succeeded",
        "FullName": "Santa Claus",
        "Location": "NorthPole"
      },
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

---

## <a name="call-action"></a>Acción llamar

El proveedor personalizado también tiene una acción denominada `ping`. El código que procesa la solicitud se implementa en la aplicación de función. La acción `ping` responde con un saludo.

Para enviar una solicitud `ping`, use la operación `POST` en su proveedor personalizado.

```http
POST https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/ping?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
pingURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"
az rest --method post --uri $pingURI
```

Recibirá la respuesta:

```json
{
  "message": "hello <function-name>.azurewebsites.net",
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  }
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$pingURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"

armclient POST $pingURI
```

Recibirá la respuesta:

```json
{
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  },
  "message": "hello <function-name>.azurewebsites.net"
}
```

---

## <a name="create-a-resource-type"></a>Creación de un tipo de recurso

Para crear el tipo de recurso personalizado, puede implementar el recurso en una plantilla. Este método se muestra en la plantilla que se ha implementado en este inicio rápido. También puede enviar una solicitud `PUT` para el tipo de recurso.

```http
PUT https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/<resource-name>?api-version=2018-09-01-preview

{"properties":{"FullName": "Test User", "Location": "Earth"}}
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
addURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
az rest --method put --uri $addURI --body "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"
```

Recibirá la respuesta:

```json
{
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "properties": {
    "FullName": "Test User",
    "Location": "Earth",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "<rg-name>",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$addURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
$requestBody = "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"

armclient PUT $addURI $requestBody
```

Recibirá la respuesta:

```json
{
  "properties": {
    "provisioningState": "Succeeded",
    "FullName": "Test User",
    "Location": "Earth"
  },
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

---

## <a name="custom-resource-provider-commands"></a>Comandos del proveedor de recursos personalizado

Use los comandos [custom-providers](/cli/azure/custom-providers/resource-provider) para trabajar con el proveedor de recursos personalizado.

### <a name="list-custom-resource-providers"></a>Enumeración de los proveedores de recursos personalizados

Use el comando `list` para mostrar todos los proveedores de recursos personalizados en una suscripción. De forma predeterminada, se enumeran los proveedores de recursos personalizados de la suscripción actual, o puede especificar el parámetro `--subscription`. Para enumerar un grupo de recursos, use el parámetro `--resource-group`.

```azurecli-interactive
az custom-providers resource-provider list --subscription $subID
```

```json
[
  {
    "actions": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "ping",
        "routingType": "Proxy"
      }
    ],
    "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceproviders/<provider-name>",
    "location": "eastus",
    "name": "<provider-name>",
    "provisioningState": "Succeeded",
    "resourceGroup": "<rg-name>",
    "resourceTypes": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "users",
        "routingType": "Proxy, Cache"
      }
    ],
    "tags": {},
    "type": "Microsoft.CustomProviders/resourceproviders",
    "validations": null
  }
]
```

### <a name="show-the-properties"></a>Presentación de las propiedades

Use el comando `show` para mostrar las propiedades del proveedor de recursos personalizado. El formato de salida es similar a la salida de `list`.

```azurecli-interactive
az custom-providers resource-provider show --resource-group $rgName --name $funcName
```

### <a name="create-a-new-resource"></a>Crear un nuevo recurso

Use el comando `create` para crear o actualizar un proveedor de recursos personalizado. En este ejemplo se actualizan `actions` y `resourceTypes`.

```azurecli-interactive
az custom-providers resource-provider create --resource-group $rgName --name $funcName \
--action name=ping endpoint=https://myTestSite.azurewebsites.net/api/{requestPath} routing_type=Proxy \
--resource-type name=users endpoint=https://myTestSite.azurewebsites.net/api{requestPath} routing_type="Proxy, Cache"
```

```json
"actions": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api/{requestPath}",
    "name": "ping",
    "routingType": "Proxy"
  }
],

"resourceTypes": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api{requestPath}",
    "name": "users",
    "routingType": "Proxy, Cache"
  }
],
```

### <a name="update-the-providers-tags"></a>Actualización de las etiquetas del proveedor

El comando `update` solo actualiza las etiquetas de un proveedor de recursos personalizado. En Azure Portal, el servicio de aplicaciones del proveedor de recursos personalizado muestra la etiqueta.

```azurecli-interactive
az custom-providers resource-provider update --resource-group $rgName --name $funcName --tags new=tag
```

```json
"tags": {
  "new": "tag"
},
```

### <a name="delete-a-custom-resource-provider"></a>Eliminación de un proveedor de recursos personalizado

El comando `delete` le avisa y elimina solo el proveedor de recursos personalizado. No se eliminan la cuenta de almacenamiento, el servicio de aplicaciones y el plan del servicio de aplicaciones. Una vez eliminado el proveedor, regresa a un símbolo del sistema.

```azurecli-interactive
az custom-providers resource-provider delete --resource-group $rgName --name $funcName
```

## <a name="next-steps"></a>Pasos siguientes

En el siguiente artículo puede encontrar una introducción a los proveedores personalizados:

> [!div class="nextstepaction"]
> [Introducción a los proveedores personalizados de Azure en versión preliminar](overview.md)
