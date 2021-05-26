---
title: Automatización de la implementación de recursos de la aplicación de función en Azure
description: Obtenga información sobre cómo crear una plantilla de Azure Resource Manager que implemente su aplicación de función.
ms.assetid: d20743e3-aab6-442c-a836-9bcea09bfd32
ms.topic: conceptual
ms.date: 04/03/2019
ms.custom: fasttrack-edit, devx-track-azurepowershell
ms.openlocfilehash: 072aa17783382c7d46298b2757b3bda9390e5e29
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368757"
---
# <a name="automate-resource-deployment-for-your-function-app-in-azure-functions"></a>Automatización de la implementación de recursos para una aplicación de función en Azure Functions

Puede usar una plantilla de Azure Resource Manager para implementar una aplicación de función. En este artículo se describen los recursos y los parámetros necesarios para hacerlo. Es posible que tenga que implementar recursos adicionales, dependiendo de los [desencadenadores y enlaces](functions-triggers-bindings.md) de la aplicación de funciones.

Para más información sobre la creación de plantillas, consulte [Creación de plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Para obtener las plantillas de ejemplo, vea:
- [Aplicación de función en el plan de consumo]
- [Aplicación de función en el plan Azure App Service]

## <a name="required-resources"></a>Recursos necesarios

Normalmente, una implementación de Azure Functions consta de estos recursos:

| Resource                                                                           | Requisito | Referencia de sintaxis y propiedades                                                         |
|------------------------------------------------------------------------------------|-------------|-----------------------------------------------------------------------------------------|
| Una aplicación de función.                                                                     | Obligatorio    | [Microsoft.Web/sites](/azure/templates/microsoft.web/sites)                             |
| Una cuenta de [Azure Storage](../storage/index.yml).                                   | Obligatorio    | [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) |
| Un componente de [Application Insights](../azure-monitor/app/app-insights-overview.md) | Opcional    | [Microsoft.Insights/components](/azure/templates/microsoft.insights/components)         |
| Un [Plan de hospedaje](./functions-scale.md)                                             | Opcional<sup>1</sup>    | [Microsoft.Web/serverfarms](/azure/templates/microsoft.web/serverfarms)                 |

<sup>1</sup>Un plan de hospedaje solo es necesario si decide ejecutar la aplicación de función en un [plan Premium](./functions-premium-plan.md) o en un [plan de App Service](../app-service/overview-hosting-plans.md).

> [!TIP]
> Aunque no es necesario, se recomienda encarecidamente configurar Application Insights para su aplicación.

<a name="storage"></a>
### <a name="storage-account"></a>Cuenta de almacenamiento

Se necesita una cuenta de Azure Storage para una aplicación de función. Se necesita una cuenta de uso general que admita blobs, tablas, colas y archivos. Para más información, vea [Requisitos de la cuenta de almacenamiento de Azure Functions](storage-considerations.md#storage-account-requirements).

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2019-06-01",
    "location": "[resourceGroup().location]",
    "kind": "StorageV2",
    "sku": {
        "name": "[parameters('storageAccountType')]"
    }
}
```

También debe especificar la propiedad `AzureWebJobsStorage` como una opción de aplicación en la configuración del sitio. Si la aplicación de función no usa Application Insights para la supervisión, debe especificar también `AzureWebJobsDashboard` como configuración de aplicación.

El tiempo de ejecución de Azure Functions usa la cadena de conexión `AzureWebJobsStorage` para crear colas internas.  Cuando Application Insights no está habilitado, el entorno de tiempo de ejecución usa la cadena de conexión `AzureWebJobsDashboard` para iniciar sesión en Azure Table Storage y activar la pestaña **Supervisión** en el portal.

Estas propiedades se especifican en la colección `appSettings` del objeto `siteConfig`:

```json
"appSettings": [
    {
        "name": "AzureWebJobsStorage",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    },
    {
        "name": "AzureWebJobsDashboard",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    }
]
```

### <a name="application-insights"></a>Application Insights

Para la supervisión de las aplicaciones de funciones, se recomienda Application Insights. El recurso de Application Insights se define con el tipo **Microsoft.Insights/components** y el tipo **web**:

```json
        {
            "apiVersion": "2015-05-01",
            "name": "[variables('appInsightsName')]",
            "type": "Microsoft.Insights/components",
            "kind": "web",
            "location": "[resourceGroup().location]",
            "tags": {
                "[concat('hidden-link:', resourceGroup().id, '/providers/Microsoft.Web/sites/', variables('functionAppName'))]": "Resource"
            },
            "properties": {
                "Application_Type": "web",
                "ApplicationId": "[variables('appInsightsName')]"
            }
        },
```

Además, se debe proporcionar la clave de instrumentación a la aplicación de funciones mediante la configuración de la aplicación `APPINSIGHTS_INSTRUMENTATIONKEY`. Esta propiedad se especifica en la colección `appSettings` del objeto `siteConfig`:

```json
"appSettings": [
    {
        "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
        "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2015-05-01').InstrumentationKey]"
    }
]
```

### <a name="hosting-plan"></a>Plan de hospedaje

La definición del plan de hospedaje varía y puede ser una de las siguientes:
* [Plan de consumo](#consumption) (opción predeterminada)
* [Plan Premium](#premium)
* [plan de App Service](#app-service-plan)

### <a name="function-app"></a>Aplicación de función

El recurso de aplicación de funciones se define mediante un recurso de tipo **Microsoft.Web/sites** y ewl tipo **functionapp**:

```json
{
    "apiVersion": "2015-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
    ]
}
```

> [!IMPORTANT]
> Si está definiendo un plan de hospedaje explícitamente, es posible que necesite un elemento adicional en la matriz de dependsOn: `"[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"`

Una aplicación de funciones debe incluir esta configuración de la aplicación:

| Nombre del valor                 | Descripción                                                                               | Valores de ejemplo                        |
|------------------------------|-------------------------------------------------------------------------------------------|---------------------------------------|
| AzureWebJobsStorage          | Cadena de conexión a una cuenta de almacenamiento que Functions Runtime usará para la cola interna | Consulte [Cuenta de almacenamiento](#storage).       |
| FUNCTIONS_EXTENSION_VERSION  | Versión del entorno de ejecución de Azure Functions.                                                | `~3`                                  |
| FUNCTIONS_WORKER_RUNTIME     | Pila de lenguaje que se usará para las funciones de esta aplicación.                                   | `dotnet`, `node`, `java`, `python` o `powershell` |
| WEBSITE_NODE_DEFAULT_VERSION | Solo es necesario si usa la pila de lenguaje `node` y especifica la versión que se usará.              | `10.14.1`                             |

Estas propiedades se especifican en la colección `appSettings` de la propiedad `siteConfig`:

```json
"properties": {
    "siteConfig": {
        "appSettings": [
            {
                "name": "AzureWebJobsStorage",
                "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
            },
            {
                "name": "FUNCTIONS_WORKER_RUNTIME",
                "value": "node"
            },
            {
                "name": "WEBSITE_NODE_DEFAULT_VERSION",
                "value": "10.14.1"
            },
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~3"
            }
        ]
    }
}
```

<a name="consumption"></a>

## <a name="deploy-on-consumption-plan"></a>Implementación en el plan de consumo

El plan de consumo asigna automáticamente potencia de proceso cuando se ejecuta código, se escala horizontalmente cuando es necesario para gestionar la carga y se reduce horizontalmente cuando no se ejecuta código. No tiene que pagar por VM inactivas y no tiene que reservar capacidad de antemano. Para más información, consulte [Escalado y hospedaje de Azure Functions](consumption-plan.md).

Para obtener una plantilla de Azure Resource Manager de ejemplo, vea [Aplicación de función en el plan de consumo].

### <a name="create-a-consumption-plan"></a>Crear un plan de consumo

No es necesario definir un plan de consumo. Se creará o se seleccionará uno automáticamente por región al crear el propio recurso de la aplicación de funciones.

El plan de consumo es un tipo especial de recurso de "granja de servidores". Para Windows, se especifica mediante el valor `Dynamic` para las propiedades `computeMode` y `sku`:

```json
{
   "type":"Microsoft.Web/serverfarms",
   "apiVersion":"2016-09-01",
   "name":"[variables('hostingPlanName')]",
   "location":"[resourceGroup().location]",
   "properties":{
      "name":"[variables('hostingPlanName')]",
      "computeMode":"Dynamic"
   },
   "sku":{
      "name":"Y1",
      "tier":"Dynamic",
      "size":"Y1",
      "family":"Y",
      "capacity":0
   }
}
```

> [!NOTE]
> El plan de consumo no se puede definir explícitamente para Linux. Se creará automáticamente.

Si define explícitamente su plan de consumo, deberá establecer la propiedad `serverFarmId` en la aplicación, de forma que apunte al identificador de recurso del plan. Debe asegurarse de que la aplicación de funciones también tiene una opción `dependsOn` para el plan.

### <a name="create-a-function-app"></a>Creación de una aplicación de función

La configuración requerida por una aplicación de funciones que se ejecuta en el plan de consumo es distinta en Windows y Linux.

#### <a name="windows"></a>Windows

En Windows, un plan de consumo requiere un valor adicional en la configuración del sitio: [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring). Esta propiedad configura la cuenta de almacenamiento donde se almacenan el código de aplicación de funciones y la configuración.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```

> [!IMPORTANT]
> No establezca la configuración [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare), ya que se genera automáticamente cuando se crea el sitio por primera vez.

#### <a name="linux"></a>Linux

En Linux, la aplicación de funciones debe tener `kind` establecido en `functionapp,linux` y la propiedad `reserved` establecida en `true`.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp,linux",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountName'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        },
        "reserved": true
    }
}
```

No se admiten los valores de configuración [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring) y [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare) en Linux.

<a name="premium"></a>
## <a name="deploy-on-premium-plan"></a>Implementación en el plan Premium

El plan Prémium ofrece la misma escala que el plan de consumo, pero incluye funcionalidades adicionales y recursos dedicados. Para obtener más información, consulte [Plan Premium de Azure Functions](./functions-premium-plan.md).

### <a name="create-a-premium-plan"></a>Creación de un plan Premium

Un plan Premium es un tipo especial de recurso de "granja de servidores". Puede especificarlo mediante `EP1`, `EP2` o `EP3` para el valor de la propiedad `Name` en el [objeto de descripción](/azure/templates/microsoft.web/2018-02-01/serverfarms#skudescription-object) de `sku`.

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[parameters('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "properties": {
        "name": "[parameters('hostingPlanName')]",
        "workerSize": "[parameters('workerSize')]",
        "workerSizeId": "[parameters('workerSizeId')]",
        "numberOfWorkers": "[parameters('numberOfWorkers')]",
        "hostingEnvironment": "[parameters('hostingEnvironment')]",
        "maximumElasticWorkerCount": "20"
    },
    "sku": {
        "Tier": "ElasticPremium",
        "Name": "EP1"
    }
}
```

### <a name="create-a-function-app"></a>Creación de una aplicación de función

Una aplicación de funciones en un plan Premium debe tener la propiedad `serverFarmId` establecida en el identificador de recurso del plan creado anteriormente. Además, un plan Prémium requiere un valor adicional en la configuración del sitio: [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring). Esta propiedad configura la cuenta de almacenamiento donde se almacenan el código de aplicación de funciones y la configuración.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```
> [!IMPORTANT]
> No establezca la configuración [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare), ya que se genera automáticamente cuando se crea el sitio por primera vez.

<a name="app-service-plan"></a>

## <a name="deploy-on-app-service-plan"></a>Implementación de plan de App Service

En el plan de App Service, la aplicación de función se ejecuta en máquinas virtuales dedicadas en las SKU de los niveles Básico, Estándar y Premium, de un modo similar a las aplicaciones web. Para más información acerca del funcionamiento del plan de App Service, consulte [Introducción detallada sobre los planes de Azure App Service](../app-service/overview-hosting-plans.md).

Para obtener una plantilla de Azure Resource Manager de ejemplo, vea [Aplicación de función en el plan Azure App Service].

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

Un plan de App Service se define mediante un recurso de "granja de servidores".

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

Para ejecutar la aplicación en Linux, también debe establecer `kind` en `Linux`:

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "kind": "Linux",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

### <a name="create-a-function-app"></a>Creación de una aplicación de función

Una aplicación de funciones en un plan de App Service debe tener la propiedad `serverFarmId` establecida en el identificador de recurso del plan creado anteriormente.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```

Las aplicaciones de Linux también deberían incluir una propiedad `linuxFxVersion` en `siteConfig`. Si solo implementa código, el valor viene determinado por la pila en tiempo de ejecución deseada con el formato de ```runtime|runtimeVersion```:

| Pila            | Valor de ejemplo                                         |
|------------------|-------------------------------------------------------|
| Python           | `python|3.7`      |
| JavaScript       | `node|12`          |
| .NET             | `dotnet|3.1` |

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ],
            "linuxFxVersion": "node|12"
        }
    }
}
```

Si está [implementando una imagen de contenedor personalizada](./functions-create-function-linux-custom-image.md), debe especificarla con `linuxFxVersion` e incluir una configuración que permita extraer la imagen, como en [Web Apps for Containers](../app-service/index.yml). Además, establezca `WEBSITES_ENABLE_APP_SERVICE_STORAGE` en `false`, ya que el contenido de la aplicación se proporciona en el propio contenedor:

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_URL",
                    "value": "[parameters('dockerRegistryUrl')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_USERNAME",
                    "value": "[parameters('dockerRegistryUsername')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
                    "value": "[parameters('dockerRegistryPassword')]"
                },
                {
                    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
                    "value": "false"
                }
            ],
            "linuxFxVersion": "DOCKER|myacr.azurecr.io/myimage:mytag"
        }
    }
}
```

## <a name="deploy-to-azure-arc"></a>Implementación en Azure Arc

Azure Functions se puede implementar en [Kubernetes habilitado para Azure Arc](../app-service/overview-arc-integration.md). Este proceso sigue en gran medida la [implementación en un plan de App Service](#deploy-on-app-service-plan), con algunas diferencias que se deben tener en cuenta.

Para crear la aplicación y planear los recursos, ya debe haber [creado un entorno de Kubernetes de App Service](../app-service/manage-create-arc-environment.md) para un clúster de Kubernetes habilitado para Azure Arc. En estos ejemplos se supone que tiene el identificador de recurso de la ubicación personalizada y el entorno de Kubernetes de App Service en el que va a realizar la implementación. En la mayoría de las plantillas, puede proporcionar estos como parámetros.

```json
{
    "parameters": {
        "kubeEnvironmentId" : {
            "type": "string"
        },
        "customLocationId" : {
            "type": "string"
        }
    }
}
```

Tanto los sitios como los planes deben hacer referencia a la ubicación personalizada mediante un campo `extendedLocation`. Este bloque se encuentra fuera de `properties`, en el mismo nivel que `kind` y `location`:

```json
{
    "extendedLocation": {
        "type": "customlocation",
        "name": "[parameters('customLocationId')]"
    },
}
```

El recurso de plan debe usar la SKU de Kubernetes (K1) y su campo `kind` debe ser "linux,kubernetes". Dentro de `properties`, `reserved` debe ser "true", y `kubeEnvironmentProfile.id` debe establecerse en el identificador de recurso del entorno de Kubernetes de App Service. Un archivo de ejemplo puede tener el siguiente aspecto:

```json
{
    "type": "Microsoft.Web/serverfarms",
    "name": "[variables('hostingPlanName')]",
    "location": "[parameters('location')]",
    "apiVersion": "2020-12-01",
    "kind": "linux,kubernetes",
    "sku": {
        "name": "K1",
        "tier": "Kubernetes"
    },
    "extendedLocation": {
        "type": "customlocation",
        "name": "[parameters('customLocationId')]"
    },
    "properties": {
        "name": "[variables('hostingPlanName')]",
        "location": "[parameters('location')]",
        "workerSizeId": "0",
        "numberOfWorkers": "1",
        "kubeEnvironmentProfile": {
            "id": "[parameters('kubeEnvironmentId')]"
        },
        "reserved": true
    }
}
```

El recurso de aplicación de funciones debe tener su campo `kind` establecido en "functionapp,linux,kubernetes" o "functionapp,linux,kubernetes,container", según si tiene pensado realizar la implementación mediante código o un contenedor. Por ejemplo, una aplicación de funciones podría tener el aspecto siguiente:

```json
 {
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "kind": "kubernetes,functionapp,linux,container",
    "location": "[parameters('location')]",
    "extendedLocation": {
        "type": "customlocation",
        "name": "[parameters('customLocationId')]"
    },
    "dependsOn": [
        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[variables('hostingPlanId')]"
    ],
    "properties": {
        "serverFarmId": "[variables('hostingPlanId')]",
        "siteConfig": {
            "linuxFxVersion": "DOCKER|mcr.microsoft.com/azure-functions/dotnet:3.0-appservice-quickstart",
            "appSettings": [
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                },
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2015-05-01-preview').key1)]"

                },
                {
                    "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
                    "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2015-05-01').InstrumentationKey]"
                }
            ],
            "alwaysOn": true
        }
    }
}
```

## <a name="customizing-a-deployment"></a>Personalización de una implementación

Una aplicación de función tiene muchos recursos secundarios que puede usar en la implementación, incluidas la configuración de la aplicación y las opciones de control del código fuente. Es posible que también elija quitar el recurso secundario **sourcecontrols** y usar otra [opción de implementación](functions-continuous-deployment.md) en su lugar.

> [!IMPORTANT]
> Para implementar la aplicación de forma correcta mediante Azure Resource Manager, es importante comprender cómo se implementan los recursos en Azure. En el siguiente ejemplo, se aplican configuraciones de nivel superior mediante **siteConfig**. Es importante establecer estas configuraciones en un nivel superior porque transmiten información al motor de implementación y en tiempo de ejecución de Functions. Se necesita información de nivel superior antes de aplicar el recurso secundario **sourcecontrols/web**. Aunque es posible configurar estas opciones en el recurso de nivel secundario **config/appSettings**, en algunos casos debe implementar la aplicación de función *antes de* que se aplique **config/appSettings**. Por ejemplo, cuando se usan funciones con [Logic Apps](../logic-apps/index.yml), las funciones son una dependencia de otro recurso.

```json
{
  "apiVersion": "2015-08-01",
  "name": "[parameters('appName')]",
  "type": "Microsoft.Web/sites",
  "kind": "functionapp",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Web/serverfarms', parameters('appName'))]"
  ],
  "properties": {
     "serverFarmId": "[variables('appServicePlanName')]",
     "siteConfig": {
        "alwaysOn": true,
        "appSettings": [
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~3"
            },
            {
                "name": "Project",
                "value": "src"
            }
        ]
     }
  },
  "resources": [
     {
        "apiVersion": "2015-08-01",
        "name": "appsettings",
        "type": "config",
        "dependsOn": [
          "[resourceId('Microsoft.Web/Sites', parameters('appName'))]",
          "[resourceId('Microsoft.Web/Sites/sourcecontrols', parameters('appName'), 'web')]",
          "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
        ],
        "properties": {
          "AzureWebJobsStorage": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "AzureWebJobsDashboard": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "FUNCTIONS_EXTENSION_VERSION": "~3",
          "FUNCTIONS_WORKER_RUNTIME": "dotnet",
          "Project": "src"
        }
     },
     {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "sourcecontrols",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites/', parameters('appName'))]"
          ],
          "properties": {
            "RepoUrl": "[parameters('sourceCodeRepositoryURL')]",
            "branch": "[parameters('sourceCodeBranch')]",
            "IsManualIntegration": "[parameters('sourceCodeManualIntegration')]"
          }
     }
  ]
}
```
> [!TIP]
> En esta plantilla se usa el valor de configuración de aplicación [Project](https://github.com/projectkudu/kudu/wiki/Customizing-deployments#using-app-settings-instead-of-a-deployment-file), que establece el directorio base en el que el motor de implementación de Functions (Kudu) busca código que se pueda implementar. En nuestro repositorio, nuestras funciones están en una subcarpeta de la carpeta **src**. Por tanto, en el ejemplo anterior, se establece el valor de configuración de la aplicación en `src`. Si las funciones se encuentran en la raíz del repositorio, o si no va a implementar desde el control de código fuente, puede quitar este valor de configuración de la aplicación.

## <a name="deploy-your-template"></a>Implementación de la plantilla

Puede usar cualquiera de los siguientes métodos para implementar la plantilla:

* [PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
* [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md)
* [Azure Portal](../azure-resource-manager/templates/deploy-portal.md)
* [REST API](../azure-resource-manager/templates/deploy-rest.md)

### <a name="deploy-to-azure-button"></a>Botón Implementación en Azure

Reemplace ```<url-encoded-path-to-azuredeploy-json>``` por una versión [codificada de la URL](https://www.bing.com/search?q=url+encode) de la ruta de acceso sin formato del archivo `azuredeploy.json` en GitHub.

A continuación se muestra un ejemplo que usa Markdown:

```markdown
[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>)
```

A continuación se muestra un ejemplo que usa HTML:

```html
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"></a>
```

### <a name="deploy-using-powershell"></a>Implementación mediante PowerShell

Los siguientes comandos de PowerShell crean un grupo de recursos e implementan una plantilla que crea una aplicación de funciones con los recursos necesarios. Para la ejecución local, es preciso tener instalado [Azure PowerShell](/powershell/azure/install-az-ps). Ejecute [`Connect-AzAccount`](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión.

```powershell
# Register Resource Providers if they're not already registered
Register-AzResourceProvider -ProviderNamespace "microsoft.web"
Register-AzResourceProvider -ProviderNamespace "microsoft.storage"

# Create a resource group for the function app
New-AzResourceGroup -Name "MyResourceGroup" -Location 'West Europe'

# Create the parameters for the file, which for this template is the function app name.
$TemplateParams = @{"appName" = "<function-app-name>"}

# Deploy the template
New-AzResourceGroupDeployment -ResourceGroupName "MyResourceGroup" -TemplateFile template.json -TemplateParameterObject $TemplateParams -Verbose
```

Para probar esta implementación, puede usar una [plantilla como esta](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/function-app-create-dynamic/azuredeploy.json), que crea una aplicación de funciones en Windows en un plan de consumo. Reemplace `<function-app-name>` por un nombre único para la aplicación de funciones.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a desarrollar y configurar Azure Functions.

* [Referencia para desarrolladores de Azure Functions](functions-reference.md)
* [Configuración de aplicaciones de función de Azure](functions-how-to-use-azure-function-app-settings.md)
* [Creación de su primera función de Azure](./functions-get-started.md)

<!-- LINKS -->

[Aplicación de función en el plan de consumo]: https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.web/function-app-create-dynamic/azuredeploy.json
[Aplicación de función en el plan Azure App Service]: https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.web/azuredeploy.json
