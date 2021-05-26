---
title: 'Inicio rápido: Creación de una aplicación web en Azure Arc'
description: Introducción a App Service en Azure Arc para la implementación de la primera aplicación web
ms.topic: quickstart
ms.date: 05/11/2021
ms.openlocfilehash: b57c5e80ecef87901221c3ead49419f3cce89302
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388460"
---
# <a name="create-an-app-service-app-on-azure-arc-preview"></a>Creación de una aplicación de App Service en Azure Arc (versión preliminar)

En este inicio rápido, creará una [aplicación de App Service en un clúster de Kubernetes habilitado para Azure Arc](overview-arc-integration.md) (versión preliminar). Este escenario solo admite aplicaciones Linux, y se puede usar una pila de lenguaje integrada o un contenedor personalizado.

## <a name="prerequisites"></a>Requisitos previos

- [Configuración de Kubernetes habilitado para Azure Arc para la ejecución en App Service](manage-create-arc-environment.md)

[!INCLUDE [app-service-arc-cli-install-extensions](../../includes/app-service-arc-cli-install-extensions.md)]

## <a name="1-create-a-resource-group"></a>1. Creación de un grupo de recursos

Ejecute el siguiente comando.

```azurecli-interactive
az group create --name myResourceGroup --location eastus 
```

<!-- ## 2. Create an App Service plan

Run the following command and replace `<environment-name>` with the name of the App Service Kubernetes environment (see [Prerequisites](#prerequisites)).

```azurecli-interactive
az appservice plan create --resource-group myResourceGroup --name myAppServicePlan --custom-location <environment-name> --kube-sku K1
``` 

Currently does not work

-->

## <a name="2-get-the-custom-location"></a>2. Obtención de la ubicación personalizada

[!INCLUDE [app-service-arc-get-custom-location](../../includes/app-service-arc-get-custom-location.md)]


## <a name="3-create-an-app"></a>3. Creación de una aplicación

El ejemplo siguiente crea una aplicación Node.js. Reemplace `<app-name>` por un nombre que sea único en el clúster (los caracteres válidos son `a-z`, `0-9` y `-`). Para ver todos los entornos en tiempo de ejecución admitidos, ejecute [`az webapp list-runtimes --linux`](/cli/azure/webapp).

```azurecli-interactive
 az webapp create \
    --resource-group myResourceGroup \
    --name <app-name> \
    --custom-location $customLocationId \
    --runtime 'NODE|12-lts'
```

## <a name="4-deploy-some-code"></a>4. Implementación de código

> [!NOTE]
> `az webapp up` no se admite durante la versión preliminar pública.

Obtenga una aplicación Node.js de ejemplo mediante Git e impleméntela mediante la [implementación de ZIP](deploy-zip.md). Reemplace `<app-name>` por el nombre de la aplicación web.

```azurecli-interactive
git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
cd nodejs-docs-hello-world
zip -r package.zip .
az webapp deployment source config-zip --resource-group myResourceGroup --name <app-name> --src package.zip
```

## <a name="5-get-diagnostic-logs-using-log-analytics"></a>5. Obtención de los registros de diagnóstico con Log Analytics

> [!NOTE]
> Para usar Log Analytics, debe haberlo habilitado previamente al [instalar la extensión de App Service](manage-create-arc-environment.md#install-the-app-service-extension). Si instaló la extensión sin Log Analytics, omita este paso.

Vaya al [área de trabajo de Log Analytics configurada con la extensión de App Service](manage-create-arc-environment.md#install-the-app-service-extension) y, a continuación, haga clic en Registros en el panel de navegación izquierdo. Ejecute la consulta de ejemplo siguiente para mostrar los registros de las últimas 72 horas. Reemplace `<app-name>` por el nombre de la aplicación web. 

```kusto
let StartTime = ago(72h);
let EndTime = now();
AppServiceConsoleLogs_CL
| where TimeGenerated between (StartTime .. EndTime)
| where AppName_s =~ "<app-name>"
```

Los registros de aplicación de todas las aplicaciones hospedadas en el clúster de Kubernetes se registran en el área de trabajo de Log Analytics en la tabla de registros personalizada denominada `AppServiceConsoleLogs_CL`. 

**Log_s** contiene registros de aplicaciones de una instancia de App Service determinada, y **AppName_s** contiene el nombre de la aplicación de App Service. Además de los registros que escribe con el código de la aplicación, la columna Log_s también contiene registros sobre el inicio, el apagado y las aplicaciones de funciones del contenedor.

Puede obtener más información sobre las consultas de registro en [Introducción a Kusto](../azure-monitor/logs/get-started-queries.md).

## <a name="optional-deploy-a-custom-container"></a>(Opcional) Implementación de un contenedor personalizado

Para crear una aplicación de contenedor personalizado, ejecute [az webapp create](/cli/azure/webapp#az_webapp_create) con `--deployment-container-image-name`. Para un repositorio privado, agregue `--docker-registry-server-user` y `--docker-registry-server-password`.

Por ejemplo, pruebe lo siguiente:

```azurecli-interactive
az webapp create 
    --resource-group myResourceGroup \
    --name <app-name> \
    --custom-location $customLocationId \
    --deployment-container-image-name mcr.microsoft.com/appsvc/node:12-lts
```

<!-- `TODO: currently gets an error but the app is successfully created: "Error occurred in request., RetryError: HTTPSConnectionPool(host='management.azure.com', port=443): Max retries exceeded with url: /subscriptions/62f3ac8c-ca8d-407b-abd8-04c5496b2221/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/cephalin-arctest4/config/appsettings?api-version=2020-12-01 (Caused by ResponseError('too many 500 error responses',))"` -->

Para actualizar la imagen una vez creada la aplicación, vea [Cambio de la imagen de Docker de un contenedor personalizado](configure-custom-container.md?pivots=container-linux#change-the-docker-image-of-a-custom-container).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de una aplicación de ASP.NET](configure-language-dotnetcore.md?pivots=platform-linux)
- [Configuración de una aplicación de Node.js](configure-language-nodejs.md?pivots=platform-linux)
- [Configuración de una aplicación de PHP](configure-language-php.md?pivots=platform-linux)
- [Configuración de una aplicación de Python en Linux](configure-language-python.md)
- [Configuración de una aplicación Java](configure-language-java.md?pivots=platform-linux)
- [Configuración de una aplicación de Ruby en Linux](configure-language-ruby.md)
- [Configurar un contenedor personalizado](configure-custom-container.md?pivots=container-linux)