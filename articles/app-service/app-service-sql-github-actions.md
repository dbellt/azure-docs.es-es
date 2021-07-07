---
title: 'Tutorial: uso de Acciones de GitHub para implementar en App Service para Containers y conectarse a una base de datos'
description: Aprenda a implementar una aplicación principal de ASP.NET en Azure y en Azure SQL Database con Acciones de GitHub
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/22/2021
ms.author: jukullam
ms.custom: github-actions-azure
ms.openlocfilehash: 5f27a0cbfedd9b5021be2aa54ab5fb021b48d0b2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110103297"
---
# <a name="tutorial-use-github-actions-to-deploy-to-app-service-for-containers-and-connect-to-a-database"></a>Tutorial: uso de Acciones de GitHub para implementar en App Service para Containers y conectarse a una base de datos

En este tutorial le guiaremos a través de la configuración de un flujo de trabajo de Acciones de GitHub para implementar una aplicación ASP.NET Core en contenedores con un back-end de [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md). Cuando haya terminado, tendrá una aplicación ASP.NET en ejecución en Azure y conectada a SQL Database. En primer lugar, deberá crear los recursos de Azure con una [plantilla de ARM](/azure/azure-resource-manager/templates/overview) del flujo de trabajo de Acciones de GitHub.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Usar un flujo de trabajo de Acciones de GitHub para agregar recursos a Azure con una plantilla de Azure Resource Manager (plantilla ARM).
> - Usar un flujo de trabajo de Acciones de GitHub para crear un contenedor con los últimos cambios de la aplicación web.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join).
  - Un repositorio de GitHub para almacenar las plantillas de Resource Manager y los archivos del flujo de trabajo. Para crear uno, vea [Creación de un repositorio](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository).

## <a name="download-the-sample"></a>Descarga del ejemplo

[Bifurque el proyecto de ejemplo](https://github.com/Azure-Samples/dotnetcore-containerized-sqldb-ghactions/) en el repositorio de muestras de Azure.

```
https://github.com/Azure-Samples/dotnetcore-containerized-sqldb-ghactions/
```

## <a name="create-the-resource-group"></a>Creación del grupo de recursos

Abra Azure Cloud Shell en https://shell.azure.com. También puede usar la CLI de Azure si la ha instalado localmente. (Para obtener más información sobre Cloud Shell, consulte el artículo de información general de Cloud Shell).

```azurecli-interactive
    az group create --name {resource-group-name} --location {resource-group-location}
```

## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

Deberá autenticarse con una entidad de servicio para que funcione el script de implementación de recursos. Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

```azurecli-interactive
    az ad sp create-for-rbac --name "{service-principal-name}" --sdk-auth --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}
```

En este ejemplo, reemplace los marcadores de posición por su id. de suscripción, el nombre del grupo de recursos y el nombre de la entidad de servicio. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación de App Service. Copie este objeto JSON para más adelante. Para obtener ayuda, consulte la [configuración de las credenciales de implementación](https://github.com/Azure/login#configure-deployment-credentials).

```output
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. El ámbito del ejemplo anterior se limita a la aplicación específica de App Service y no a todo el grupo de recursos.

## <a name="configure-the-github-secret-for-authentication"></a>Configuración del secreto de GitHub para la autenticación

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para utilizar las [credenciales de nivel de usuario](#generate-deployment-credentials), pegue toda la salida JSON del comando CLI de Azure en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_CREDENTIALS`.

## <a name="add-a-sql-server-secret"></a>Adición de un secreto de SQL Server

Cree un nuevo secreto en el repositorio para `SQL_SERVER_ADMIN_PASSWORD`. Este secreto puede ser cualquier contraseña que cumpla con los estándares de Azure para la seguridad de contraseñas. Tenga en cuenta que no podrá volver a acceder a esta contraseña, así que guárdela por separado.

## <a name="create-azure-resources"></a>Creación de recursos de Azure

El flujo de trabajo de creación de recursos de Azure ejecuta una [plantilla de ARM](/azure/azure-resource-manager/templates/overview) para implementar recursos en Azure. El flujo de trabajo:

- Restaura el código fuente con la [acción de restauración](https://github.com/marketplace/actions/checkout).
- Inicia sesión en Azure con la [acción de inicio de sesión de Azure](https://github.com/marketplace/actions/azure-login) y recopila información del entorno y los recursos de Azure.
- Implementa recursos con la [acción para implementar Azure Resource Manager](https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template).

Para ejecutar el flujo de trabajo de creación de recursos de Azure:

1. Abra el archivo `azuredeploy.yaml` en `.github/workflows` en el repositorio.

1. Actualice el valor de `AZURE_RESOURCE_GROUP` al nombre de su grupo de recursos.

1. Vaya a **Acciones** y seleccione **Ejecutar flujo de trabajo**.

   :::image type="content" source="media/github-actions-workflows/github-actions-run-workflow.png" alt-text="Ejecute el flujo de trabajo de Acciones de GitHub para agregar recursos.":::

1. Compruebe que su acción se haya ejecutado correctamente; para ello, compruebe si hay una marca de verificación verde en la página **Acciones**.

   :::image type="content" source="media/github-actions-workflows/create-resources-success.png" alt-text="Ejecución exitosa de creación de recursos. ":::

## <a name="add-container-registry-and-sql-secrets"></a>Adición de registros de contenedor y secretos de SQL

1. En Azure Portal, abra la instancia recién creada de Azure Container Registry en su grupo de recursos.

1. Vaya a **Claves de acceso** y copie los valores de nombre de usuario y contraseña.

1. Cree nuevos secretos de GitHub para `ACR_USERNAME` y una contraseña `ACR_PASSWORD` en el repositorio.

1. En Azure Portal, abra la base de datos Azure SQL. Haga clic en **Mostrar cadenas de conexión** y copie el valor.

1. Cree un secreto para `SQL_CONNECTION_STRING`. Reemplace `{your_password}` con su `SQL_SERVER_ADMIN_PASSWORD`.

## <a name="build-push-and-deploy-your-image"></a>Compilación, inserción e implementación de la imagen

El flujo de trabajo de compilación, inserción e implementación crea un contenedor con los últimos cambios de la aplicación, envía el contenedor a [Azure Container Registry](/azure/container-registry/) y actualiza el espacio de ensayo de la aplicación web para que apunte al último contenedor insertado. El flujo de trabajo contiene un trabajo de compilación e implementación:

- El trabajo de compilación restaura el código fuente con la [acción de restauración](https://github.com/marketplace/actions/checkout). A continuación, el trabajo usa la [acción de inicio de sesión de Docker](https://github.com/marketplace/actions/docker-login) y un script personalizado para autenticarse con Azure Container Registry, compilar una imagen de contenedor e implementarla en Azure Container Registry.
- El trabajo de implementación inicia sesión en Azure con la [acción de inicio de sesión de Azure](https://github.com/marketplace/actions/azure-login) y recopila información del entorno y los recursos de Azure. A continuación, el trabajo actualiza la configuración de la aplicación web con la [acción de configuración de Azure App Service](https://github.com/marketplace/actions/azure-app-service-settings) y se implementa en un espacio de ensayo de App Service con la [acción de Azure Web Deploy](https://github.com/marketplace/actions/azure-webapp). Por último, el trabajo ejecuta un script personalizado para actualizar la base de datos de SQL y cambia el espacio de ensayo a producción.

Para ejecutar el flujo de trabajo de compilación, inserción e implementación:

1. Abra el archivo `build-deploy.yaml` en `.github/workflows` en el repositorio.

1. Compruebe que las variables de entorno de `AZURE_RESOURCE_GROUP` y `WEB_APP_NAME` coinciden con las de `azuredeploy.yaml`.

1. Actualice el valor `ACR_LOGIN_SERVER` del servidor de inicio de sesión de Azure Container Registry.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre la integración de Azure y GitHub](/azure/developer/github/)
