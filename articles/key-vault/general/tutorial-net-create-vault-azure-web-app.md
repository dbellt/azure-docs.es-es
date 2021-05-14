---
title: 'Tutorial: Uso de Azure Key Vault con una aplicación web de Azure en .NET'
description: En este tutorial, va a configurar una aplicación web de Azure en una aplicación de ASP.NET Core para leer un secreto del almacén de claves.
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: fdefca482dfa1afeb3eebb284b08eaadf4f1af60
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897061"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-web-app-in-net"></a>Tutorial: Uso de una identidad administrada para conectar Key Vault a una aplicación web de Azure en .NET

[Azure Key Vault](./overview.md) proporciona una manera de almacenar credenciales y otros secretos con mayor seguridad. Pero el código debe autenticarse en Key Vault para recuperarlos. Las [identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md) ayudan a resolver este problema al ofrecer a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (Azure AD). Puede usar esta identidad para autenticar cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de tener que mostrar las credenciales en el código.

En este tutorial, creará e implementará una aplicación web de Azure en [Azure App Service](../../app-service/overview.md). Usará una identidad administrada para autenticar la aplicación web de Azure con un almacén de claves de Azure mediante la [biblioteca cliente de secretos de Azure Key Vault para .NET](/dotnet/api/overview/azure/key-vault) y la [CLI de Azure](/cli/azure/get-started-with-azure-cli). Se aplican los mismos principios básicos cuando se usa el lenguaje de desarrollo de su elección, Azure PowerShell o Azure Portal.

Para más información sobre las aplicaciones web de Azure App Service y la implementación presentada en este tutorial, consulte:
- [Información general de App Service](../../app-service/overview.md)
- [Creación de una aplicación web de ASP.NET Core en Azure App Service](../../app-service/quickstart-dotnetcore.md).
- [Implementación de Git local en Azure App Service](../../app-service/deploy-local-git.md)

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Suscripción a Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* El [SDK para .NET Core 3.1 (o posterior)](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Una instalación de [Git](https://www.git-scm.com/downloads), versión 2.28.0 o posterior.
* La [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/)
* [Azure Key Vault](./overview.md). Puede crear un almacén de claves mediante [Azure Portal](quick-create-portal.md), la [CLI de Azure](quick-create-cli.md) o [Azure PowerShell](quick-create-powershell.md).
* Un [secreto](../secrets/about-secrets.md) de Key Vault. Puede crear un secreto mediante [Azure Portal](../secrets/quick-create-portal.md), [PowerShell](../secrets/quick-create-powershell.md) o la [CLI de Azure](../secrets/quick-create-cli.md).

Si ya tiene la aplicación web implementada en Azure App Service, puede pasar directamente a [configurar el acceso de la aplicación web a un almacén de claves](#create-and-assign-a-managed-identity) y [modificar las secciones del código de la aplicación web](#modify-the-app-to-access-your-key-vault).

## <a name="create-a-net-core-app"></a>Creación de una aplicación .NET Core
En este paso, va a configurar el proyecto de .NET Core local.

En una ventana de terminal de la máquina, cree un directorio llamado `akvwebapp` y conviértalo en el directorio actual.

```bash
mkdir akvwebapp
cd akvwebapp
```

Cree una aplicación de .NET Core mediante el comando [dotnet new web](/dotnet/core/tools/dotnet-new):

```bash
dotnet new web
```

Ejecute la aplicación localmente para que sepa cómo debe ser el aspecto cuando la implemente en Azure:

```bash
dotnet run
```

En un explorador web, vaya a la aplicación en `http://localhost:5000`.

Verá el mensaje "Hola mundo" Mensaje de la aplicación de ejemplo que se muestra en la página.

Para más información sobre la creación de aplicaciones web para Azure, consulte [Creación de una aplicación web de ASP.NET Core en Azure App Service](../../app-service/quickstart-dotnetcore.md).

## <a name="deploy-the-app-to-azure"></a>Implementar la aplicación en Azure

En este paso, va a implementar la aplicación de .NET Core en Azure App Service mediante Git local. Para más información sobre cómo crear e implementar aplicaciones, consulte [Creación de una aplicación web ASP.NET Core en Azure](../../app-service/quickstart-dotnetcore.md).

### <a name="configure-the-local-git-deployment"></a>Configuración de la implementación de Git local

En la ventana de terminal, seleccione **Ctrl + C** para cerrar el servidor Web.  Inicialice un repositorio de Git para el proyecto de .NET Core:

```bash
git init --initial-branch=main
git add .
git commit -m "first commit"
```

Puede utilizar FTP y Git local para implementar una aplicación web de Azure mediante un *usuario de implementación*. Después de configurar el usuario de implementación, podrá usarlo para todas las implementaciones de Azure. El nombre de usuario y la contraseña de implementación en el nivel de cuenta son diferentes de las credenciales de la suscripción de Azure. 

Para configurar el usuario de implementación, ejecute el comando [az webapp deployment user set](/cli/azure/webapp/deployment/user?#az_webapp_deployment_user_set). Elija un nombre de usuario y una contraseña que sigan estas siguientes directrices: 

- El nombre de usuario debe ser único en Azure. Para las inserciones locales de Git, no puede contener el símbolo de arroba (@). 
- La contraseña debe tener al menos ocho caracteres de longitud y contener dos de los tres elementos siguientes: letras, números y símbolos. 

```azurecli-interactive
az webapp deployment user set --user-name "<username>" --password "<password>"
```

La salida JSON muestra la contraseña como `null`. Si se produce un error `'Conflict'. Details: 409`, cambie el nombre de usuario. Si se produce un error `'Bad Request'. Details: 400`, use una contraseña más segura. 

Anote el nombre de usuario y la contraseña que se usarán para implementar las aplicaciones web.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos para contener tanto almacén de claves como la aplicación web mediante el comando [az group create](/cli/azure/group?#az_group_create):

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

Cree un [plan de App Service](../../app-service/overview-hosting-plans.md) mediante el comando [az appservice plan create](/cli/azure/appservice/plan) de la CLI de Azure. En el siguiente ejemplo, se crea un plan de App Service denominado `myAppServicePlan` con el plan de tarifa `FREE`:

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

Cuando se ha creado el plan de App Service, la CLI de Azure muestra información similar a la que se ve aquí:

<pre>
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "West Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "West Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
} 
</pre>

Para más información, consulte [Cambio de una aplicación a otro plan de App Service](../../app-service/app-service-plan-manage.md).

### <a name="create-a-web-app"></a>Creación de una aplicación web

Creación de una [aplicación web de Azure](../../app-service/overview.md) en el plan de App Service `myAppServicePlan`. 

> [!Important]
> Al igual que sucede con un almacén de claves, una aplicación web de Azure debe tener un nombre único. Reemplace `<your-webapp-name>` por el nombre de la aplicación web en los ejemplos siguientes.


```azurecli-interactive
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "<your-webapp-name>" --deployment-local-git
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará una salida similar a la que se ve aquí:

<pre>
Local git is configured with url of 'https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;ayour-webapp-name&gt;.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "clientCertExclusionPaths": null,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;your-webapp-name&gt;.azurewebsites.net",
  "deploymentLocalGitUrl": "https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;your-webapp-name&gt;.git",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

La dirección URL de Git remoto se muestra en la propiedad `deploymentLocalGitUrl`, con el formato `https://<username>@<your-webapp-name>.scm.azurewebsites.net/<your-webapp-name>.git`. Guarde esta dirección URL. Lo necesitará más adelante.

Ahora configure la aplicación web para implementarla desde la rama `main`:

```azurecli-interactive
 az webapp config appsettings set -g MyResourceGroup --name "<your-webapp-name>" --settings deployment_branch=main
```

Vaya a la nueva aplicación con el siguiente comando. Reemplace `<your-webapp-name>` por el nombre de la aplicación.

```bash
https://<your-webapp-name>.azurewebsites.net
```

Verá la página web predeterminada de una nueva aplicación web de Azure.

### <a name="deploy-your-local-app"></a>Implementación de la aplicación local

En la ventana del terminal local, agregue una instancia remota de Azure al repositorio de Git local. En el siguiente comando, reemplace `<deploymentLocalGitUrl-from-create-step>` por la dirección URL de Git remoto que guardó en la sección [Creación de una aplicación web](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Realice la inserción en la instancia remota de Azure para implementar la aplicación con el comando siguiente. Cuando el administrador de credenciales de Git le solicite las credenciales, utilice las que creó en la sección [Configuración de la implementación de Git local](#configure-the-local-git-deployment).

```bash
git push azure main
```

Este comando puede tardar unos minutos en ejecutarse. Mientras se ejecuta, muestra información similar a la que se ve aquí:
<pre>
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 285 bytes | 95.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Deploy Async
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'd6b54472f7'.
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Build orchestrated by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote:
remote: Oryx Version      : 0.2.20200114.13, Commit: 204922f30f8e8d41f5241b8c218425ef89106d1d, ReleaseTagName: 20200114.13
remote: Build Operation ID: |imoMY2y77/s=.40ca2a87_
remote: Repository Commit : d6b54472f7e8e9fd885ffafaa64522e74cf370e1
.
.
.
remote: Deployment successful.
remote: Deployment Logs : 'https://&lt;your-webapp-name&gt;.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/d6b54472f7e8e9fd885ffafaa64522e74cf370e1/log'
To https://&lt;your-webapp-name&gt;.scm.azurewebsites.net:443/&lt;your-webapp-name&gt;.git
   d87e6ca..d6b5447  main -> main
</pre>

Vaya a la aplicación implementada, o actualícela, en el explorador web:

```bash
http://<your-webapp-name>.azurewebsites.net
```

Verá el mensaje "Hola mundo" que se mostró anteriormente al visitar `http://localhost:5000`.

Para más información sobre la implementación de una aplicación web con Git, consulte [Implementación de Git local en Azure App Service](../../app-service/deploy-local-git.md).
 
## <a name="configure-the-web-app-to-connect-to-key-vault"></a>Configuración de la aplicación web para conectarse a Key Vault

En esta sección va a configurar el acceso web a Key Vault y a actualizar el código de la aplicación para recuperar un secreto desde Key Vault.

### <a name="create-and-assign-a-managed-identity"></a>Creación y asignación de una identidad administrada

En este tutorial usaremos una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) para la autenticación en Key Vault. La identidad administrada administra automáticamente las credenciales de la aplicación.

Para crear la identidad de la aplicación, ejecute el comando [az webapp-identity assign](/cli/azure/webapp/identity?#az_webapp_identity_assign) en la CLI de Azure:

```azurecli-interactive
az webapp identity assign --name "<your-webapp-name>" --resource-group "myResourceGroup"
```

El comando devolverá este fragmento de código JSON:

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

Para conceder permisos a la aplicación web para realizar operaciones **get** y **list** en el almacén de claves, pase el valor de `principalId` al comando [az keyvault set-policy](/cli/azure/keyvault?#az_keyvault_set_policy) de la CLI de Azure:

```azurecli-interactive
az keyvault set-policy --name "<your-keyvault-name>" --object-id "<principalId>" --secret-permissions get list
```

También puede asignar directivas de acceso mediante [Azure Portal](./assign-access-policy-portal.md) o [PowerShell](./assign-access-policy-powershell.md).

### <a name="modify-the-app-to-access-your-key-vault"></a>Modificación de la aplicación para acceder al almacén de claves

En este tutorial, usará la [biblioteca cliente de secretos de Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.secrets-readme) con fines de demostración. También puede usar la [biblioteca cliente de certificados de Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.certificates-readme) o la [biblioteca cliente de claves de Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.keys-readme).

#### <a name="install-the-packages"></a>Instalación de los paquetes

En la ventana de terminal, instale los paquetes de la biblioteca cliente de secretos de Azure Key Vault para .NET y la biblioteca cliente Azure Identity:

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

#### <a name="update-the-code"></a>Actualización del código

Busque y abra el archivo Startup.cs en su proyecto akvwebapp. 

Agregue estas líneas al encabezado:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

Agregue las líneas siguientes antes de la llamada a `app.UseEndpoints` y actualice el identificador URI para que refleje el valor de `vaultUri` del almacén de claves. Este código usa [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) para autenticarse en Key Vault, que usa un token de la identidad administrada para la autenticación. Para más información sobre la autenticación en Key Vault, consulte [Guía del desarrollador de Azure Key Vault](./developers-guide.md#authenticate-to-key-vault-in-code). El código también utiliza el retroceso exponencial para los reintentos en el caso de que se limite el almacén de claves. Para más información sobre los límites de transacciones de Key Vault, consulte [Guía de las limitaciones de Azure Key Vault](./overview-throttling.md).

```csharp
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
var client = new SecretClient(new Uri("https://<your-unique-key-vault-name>.vault.azure.net/"), new DefaultAzureCredential(),options);

KeyVaultSecret secret = client.GetSecret("<mySecret>");

string secretValue = secret.Value;
```

Actualice la línea `await context.Response.WriteAsync("Hello World!");` para que tenga el aspecto de esta línea:

```csharp
await context.Response.WriteAsync(secretValue);
```

Asegúrese de guardar los cambios antes de continuar con el siguiente paso.

#### <a name="redeploy-your-web-app"></a>Nueva implementación de la aplicación web

Ahora que ha actualizado el código, puede volver a implementarlo en Azure mediante estos comandos de Git:

```bash
git add .
git commit -m "Updated web app to access my key vault"
git push azure main
```

## <a name="go-to-your-completed-web-app"></a>Ir a la aplicación web completada

```bash
http://<your-webapp-name>.azurewebsites.net
```

Donde antes aparecía "Hola mundo!", ahora debería ver que se muestra el valor del secreto.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Uso de Azure Key Vault con una máquina virtual en .NET](./tutorial-net-virtual-machine.md)
- Más información sobre las [identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md)
- Consulte [Guía del desarrollador de Azure Key Vault](./developers-guide.md).
- [Protección del acceso a un almacén de claves](./security-features.md)
