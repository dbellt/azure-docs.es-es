---
title: Aprovisionamiento de una aplicación web con Azure Cache for Redis
description: Use una plantilla de Azure Resource Manager para implementar una aplicación web con Azure Cache for Redis.
services: app-service
author: yegu-ms
ms.service: app-service
ms.topic: conceptual
ms.date: 01/06/2017
ms.author: yegu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8232582231a359ea6a9cf49b20685a3c9e4ef0d0
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006318"
---
# <a name="create-a-web-app-plus-azure-cache-for-redis-using-a-template"></a>Creación de una aplicación web y Azure Cache for Redis mediante una plantilla

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este artículo, aprenderá a crear una plantilla de Azure Resource Manager que implementa una aplicación web de Azure Cache for Redis. Aprenderá los siguientes detalles de implementación:

- Cómo definir qué recursos se implementan 
- Cómo definir los recursos que se implementan y los parámetros que se especifican cuando se ejecuta la implementación

Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades.

Para obtener más información sobre la creación de plantillas, consulte [Creación de plantillas de Administrador de recursos de Azure](../azure-resource-manager/templates/syntax.md). Para información sobre la sintaxis y las propiedades de JSON para los tipos de recursos de caché, consulte [Tipos de recursos Microsoft.Cache](/azure/templates/microsoft.cache/allversions).

Para ver la plantilla completa, consulte [Plantilla Aplicación web con Azure Cache for Redis](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-with-redis-cache/azuredeploy.json).

## <a name="what-you-will-deploy"></a>Lo que implementará
En esta plantilla, implementará lo siguiente:

* Aplicación web de Azure
* Azure Cache for Redis

Para ejecutar automáticamente la implementación, seleccione el botón siguiente:

[![Implementación en Azure](./media/cache-web-app-arm-with-redis-cache-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fweb-app-with-redis-cache%2Fazuredeploy.json)

## <a name="parameters-to-specify"></a>Parámetros para especificar
[!INCLUDE [app-service-web-deploy-web-parameters](../../includes/app-service-web-deploy-web-parameters.md)]

[!INCLUDE [cache-deploy-parameters](../../includes/cache-deploy-parameters.md)]

## <a name="variables-for-names"></a>Variables de nombres
Esta plantilla usa variables para construir los nombres de los recursos. Usa la función [uniqueString](../azure-resource-manager/templates/template-functions-string.md#uniquestring) para construir un valor basado en el identificador del grupo de recursos.

```json
"variables": {
  "hostingPlanName": "[concat('hostingplan', uniqueString(resourceGroup().id))]",
  "webSiteName": "[concat('webSite', uniqueString(resourceGroup().id))]",
  "cacheName": "[concat('cache', uniqueString(resourceGroup().id))]"
},
```


## <a name="resources-to-deploy"></a>Recursos para implementar
[!INCLUDE [app-service-web-deploy-web-host](../../includes/app-service-web-deploy-web-host.md)]

### <a name="azure-cache-for-redis"></a>Azure Cache for Redis
Crea Azure Cache for Redis que se usa con la aplicación web. El nombre de la memoria caché se especifica en la variable **cacheName** .

La plantilla crea la memoria caché en la misma ubicación que el grupo de recursos.

```json
{
  "name": "[variables('cacheName')]",
  "type": "Microsoft.Cache/Redis",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-08-01",
  "dependsOn": [ ],
  "tags": {
    "displayName": "cache"
  },
  "properties": {
    "sku": {
      "name": "[parameters('cacheSKUName')]",
      "family": "[parameters('cacheSKUFamily')]",
      "capacity": "[parameters('cacheSKUCapacity')]"
    }
  }
}
```


### <a name="web-app-azure-cache-for-redis"></a>Aplicación web (Azure Cache for Redis)
Crea la aplicación web con el nombre especificado en la variable **webSiteName** .

Observe que la aplicación web está configurada con las propiedades de configuración de la aplicación que permiten trabajar con Azure Cache for Redis. Estos valores de configuración de la aplicación se crean dinámicamente de acuerdo con los valores proporcionados durante la implementación.

```json
{
  "apiVersion": "2015-08-01",
  "name": "[variables('webSiteName')]",
  "type": "Microsoft.Web/sites",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Web/serverFarms/', variables('hostingPlanName'))]"
  ],
  "tags": {
    "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('hostingPlanName'))]": "empty",
    "displayName": "Website"
  },
  "properties": {
    "name": "[variables('webSiteName')]",
    "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
  },
  "resources": [
    {
      "apiVersion": "2015-08-01",
      "type": "config",
      "name": "appsettings",
      "dependsOn": [
        "[concat('Microsoft.Web/Sites/', variables('webSiteName'))]",
        "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
      ],
      "properties": {
       "CacheConnection": "[concat(variables('cacheHostName'),'.redis.cache.windows.net,abortConnect=false,ssl=true,password=', listKeys(resourceId('Microsoft.Cache/Redis', variables('cacheName')), '2015-08-01').primaryKey)]"
      }
    }
  ]
}
```


### <a name="web-app-redisenterprise"></a>Aplicación web (RedisEnterprise)
Para RedisEnterprise, dado que los tipos de recursos son ligeramente diferentes, la manera de hacer **listKeys** es distinta:

```json
{
  "apiVersion": "2015-08-01",
  "name": "[variables('webSiteName')]",
  "type": "Microsoft.Web/sites",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Web/serverFarms/', variables('hostingPlanName'))]"
  ],
  "tags": {
    "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('hostingPlanName'))]": "empty",
    "displayName": "Website"
  },
  "properties": {
    "name": "[variables('webSiteName')]",
    "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
  },
  "resources": [
    {
      "apiVersion": "2015-08-01",
      "type": "config",
      "name": "appsettings",
      "dependsOn": [
        "[concat('Microsoft.Web/Sites/', variables('webSiteName'))]",
        "[concat('Microsoft.Cache/RedisEnterprise/databases/', variables('cacheName'), "/default")]",
      ],
      "properties": {
       "CacheConnection": "[concat(variables('cacheHostName'),abortConnect=false,ssl=true,password=', listKeys(resourceId('Microsoft.Cache/RedisEnterprise', variables('cacheName'), 'default'), '2020-03-01').primaryKey)]"
      }
    }
  ]
}
```

## <a name="commands-to-run-deployment"></a>Comandos para ejecutar la implementación
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```azurepowershell
New-AzResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-with-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/web-app-with-redis-cache/azuredeploy.json -g ExampleDeployGroup
```