---
title: 'Inicio rápido: Creación de una aplicación de varios contenedores'
description: Para empezar a trabajar con aplicaciones de varios contenedores en Azure App Service, primero es preciso implementar la primera de estas aplicaciones.
keywords: servicio de aplicación de azure, aplicación web, linux, docker, compose, multicontenedor, varios contenedores, aplicación web para contenedores, contenedor, wordpress, azure db for mysql, base de datos de producción con contenedores
author: msangapu-msft
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 0ea55c36c239b5ecdb51ef3dc7a3ff762718bd1e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769018"
---
# <a name="create-a-multi-container-preview-app-using-a-docker-compose-configuration"></a>Creación de una aplicación de varios contenedores (versión preliminar) mediante una configuración de Docker Compose

> [!NOTE]
> La aplicación de varios contenedores está en versión preliminar.

[Web App for Containers](overview.md#app-service-on-linux) proporciona una manera flexible de utilizar imágenes de Docker. En este inicio rápido se muestra cómo implementar una aplicación de varios contenedores (versión preliminar) en Web App for Containers en [Cloud Shell](../cloud-shell/overview.md) mediante una configuración de Docker Compose.

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

En este artículo se necesita la versión 2.0.32 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="download-the-sample"></a>Descarga del ejemplo

En esta guía de inicio rápido se usa el archivo de Compose de [Docker](https://docs.docker.com/compose/wordpress/#define-the-project). El archivo de configuración se pueden encontrar en los [ejemplos de Azure](https://github.com/Azure-Samples/multicontainerwordpress).

[!code-yml[Main](../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

En Cloud Shell, cree un directorio de inicio rápido y luego cambie a él.

```bash
mkdir quickstart

cd $HOME/quickstart
```

A continuación, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en el directorio de inicio rápido. Después, cambie al directorio `multicontainerwordpress`.

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [resource group intro text](../../includes/resource-group.md)]

En Cloud Shell, cree un grupo de recursos con el comando [`az group create`](/cli/azure/group#az_group_create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *Centro-sur de EE. UU.* Para ver todas las ubicaciones de App Service que se admiten en Linux en el nivel **Estándar**, ejecute el comando [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice#az_appservice_list_locations).

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

Generalmente se crean el grupo de recursos y los recursos en una región cercana.

Cuando finaliza el comando, una salida de JSON muestra las propiedades del grupo de recursos.

## <a name="create-an-azure-app-service-plan"></a>Crear un plan de Azure App Service

En Cloud Shell, cree un plan de App Service en el grupo de recursos con el comando [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create).

En el ejemplo siguiente se crea un plan de App Service denominado `myAppServicePlan` en el plan de tarifa **Estándar** (`--sku S1`) y en un contenedor Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

Cuando se crea el plan de App Service, la CLI de Azure muestra información similar al ejemplo siguiente:

<pre>
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
</pre>

## <a name="create-a-docker-compose-app"></a>Creación de una aplicación de Docker Compose

> [!NOTE]
> En este momento, Docker Compose en Azure App Services tiene un límite de 4.000 caracteres.

En el terminal de Cloud Shell, cree una [aplicación web](overview.md#app-service-on-linux) de varios contenedores en el plan de App Service `myAppServicePlan` con el comando [az webapp create](/cli/azure/webapp#az_webapp_create). No olvide reemplazar _\<app_name>_ por un nombre de aplicación único (los caracteres válidos son `a-z`, `0-9` y `-`).

```azurecli
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará información similar a la del ejemplo siguiente:

<pre>
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app_name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app_name>.azurewebsites.net`). La aplicación puede tardar unos minutos en cargarse. Si recibe un error, deje pasar unos minutos, y actualice el explorador.

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

**Enhorabuena**, ha creado una aplicación de varios contenedores en Web App for Containers.

[!INCLUDE [Clean-up section](../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de WordPress con varios contenedores](tutorial-multi-container-app.md)

> [!div class="nextstepaction"]
> [Configurar un contenedor personalizado](configure-custom-container.md)

<!--Image references-->
[1]: media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
