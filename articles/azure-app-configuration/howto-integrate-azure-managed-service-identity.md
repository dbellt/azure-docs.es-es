---
title: Uso de identidades administradas para acceder a App Configuration
titleSuffix: Azure App Configuration
description: Autenticación en Azure App Configuration mediante identidades administradas
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.custom: devx-track-csharp, fasttrack-edit
ms.topic: conceptual
ms.date: 04/08/2021
ms.openlocfilehash: b4b67a6bf4eea72b3a81bd01398cc60cc33c0d0a
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227048"
---
# <a name="use-managed-identities-to-access-app-configuration"></a>Uso de identidades administradas para acceder a App Configuration

Las [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) de Azure Active Directory simplifican la administración de secretos de su aplicación de nube. Con una identidad administrada, el código puede usar la entidad de servicio creada para el servicio de Azure donde se ejecuta. Usar una identidad administrada en lugar de una credencial diferente que se almacenan en Azure Key Vault o una cadena de conexión local.

Azure App Configuration y sus bibliotecas cliente .NET Core, .NET Framework y Java Spring incluyen compatibilidad integrada con la identidad administrada. Aunque su uso no es necesario, la identidad administrada elimina la necesidad de un token de acceso que contenga los secretos. El código solo puede acceder al almacén de App Configuration mediante el punto de conexión de servicio. Puede insertar esta dirección URL en el código directamente sin exponer ningún secreto.

En este artículo se muestra cómo puede aprovechar la identidad administrada para acceder a App Configuration. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, primero [cree una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

> [!NOTE]
> En este artículo se usa Azure App Service como ejemplo, pero el mismo concepto se aplica a cualquier otro servicio de Azure compatible con la identidad administrada, por ejemplo, [Azure Kubernetes Service](../aks/use-azure-ad-pod-identity.md), la [Máquina virtual de Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md) y [Azure Container Instances](../container-instances/container-instances-managed-identity.md). Si la carga de trabajo está hospedada en uno de esos servicios, también puede aprovechar la compatibilidad con la identidad administrada del servicio.

En este artículo se muestra también cómo puede usar la identidad administrada junto con las referencias a Key Vault de App Configuration. Con una única identidad administrada, puede acceder sin problemas a los secretos desde Key Vault y los valores de configuración desde App Configuration. Si desea explorar esta funcionalidad, termine primero [Uso de referencias de Key Vault con ASP.NET Core](./use-key-vault-references-dotnet-core.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Conceder a una identidad administrada acceso a App Configuration.
> * Configurar la aplicación para usar una identidad administrada al conectarse a App Configuration.
> * También puede configurar la aplicación para que use una identidad administrada cuando se conecte a Key Vault mediante una referencia de Key Vault de App Configuration.

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, necesitará lo siguiente:

* [SDK de .NET Core](https://www.microsoft.com/net/download/windows).
* [Azure Cloud Shell configurado](../cloud-shell/quickstart.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="add-a-managed-identity"></a>Agregar una identidad administrada

Para configurar una identidad administrada en el portal, primero crea una aplicación y luego habilita la característica.

1. Cree una instancia de App Services en [Azure Portal](https://portal.azure.com) como lo haría normalmente. Vaya a ella en el portal.

1. Desplácese hacia abajo hasta el grupo **Configuración** en el panel de navegación izquierdo y seleccione **Identidad**.

1. En la pestaña **Asignado por el sistema**, cambie **Estado** a **Activado** y seleccione **Guardar**.

1. Responda **Sí** cuando se le pida que habilite la identidad administrada asignada por el sistema.

    ![Configurar la identidad administrada en App Service](./media/set-managed-identity-app-service.png)

## <a name="grant-access-to-app-configuration"></a>Conceder acceso a App Configuration

1. En [Azure Portal](https://portal.azure.com), seleccione **Todos los recursos** y elija el almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Access Control (IAM)** .

1. En la pestaña **Comprobar el acceso**, seleccione **Agregar** en la interfaz de usuario de la tarjeta **Agregar una asignación de roles**.

1. En **Rol**, seleccione **Lector de los datos de App Configuration**. En **Asignar acceso**, seleccione **App Service** en **Identidad administrada asignada por el sistema**.

1. En **Suscripción**, seleccione su suscripción de Azure. Seleccione el recurso de App Service para la aplicación.

1. Seleccione **Guardar**.

    ![Agregar una identidad administrada](./media/add-managed-identity.png)

1. Opcional: Si también desea conceder acceso a Key Vault, siga las instrucciones de [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-portal.md).

## <a name="use-a-managed-identity"></a>Uso de una identidad administrada

1. Agregue una referencia al paquete *Azure.Identity*.

    ```bash
    dotnet add package Azure.Identity
    ```

1. Busque el punto de conexión en el almacén de App Configuration. Esta dirección URL aparece en la pestaña **Claves de acceso** para el almacén en la Azure Portal.

1. Abra *appsettings.json* y agregue el siguiente script. Reemplace *\<service_endpoint>* , incluidos los corchetes, por la dirección URL del almacén de App Configuration.

    ```json
    "AppConfig": {
        "Endpoint": "<service_endpoint>"
    }
    ```

1. Abra *Program.cs* y agregue una referencia a los espacios de nombres `Azure.Identity` y `Microsoft.Azure.Services.AppAuthentication`:

    ```csharp-interactive
    using Azure.Identity;
    ```

1. Si desea acceder solo a los valores almacenados directamente en App Configuration, actualice el método `CreateWebHostBuilder` reemplazando el método `config.AddAzureAppConfiguration()` (se encuentra en el paquete `Microsoft.Azure.AppConfiguration.AspNetCore`).

    > [!IMPORTANT]
    > `CreateHostBuilder` reemplaza a `CreateWebHostBuilder` en .NET Core 3.0.  Seleccione la sintaxis correcta en función de su entorno.

    ### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration((hostingContext, config) =>
               {
                   var settings = config.Build();
                   config.AddAzureAppConfiguration(options =>
                       options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential()));
               })
               .UseStartup<Startup>();
    ```

    ### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential()));
                });
            })
            .UseStartup<Startup>());
    ```
    ---

    > [!NOTE]
    > En el caso de que quiera usar una **identidad administrada asignada por el usuario**, asegúrese de especificar el valor de clientId al crear [ManagedIdentityCredential](/dotnet/api/azure.identity.managedidentitycredential).
    >```
    >config.AddAzureAppConfiguration(options =>
    >   options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential(<your_clientId>)));
    >```
    >Como se explica en las [preguntas más frecuentes sobre identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/managed-identities-faq.md#what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request), hay una manera predeterminada de resolver qué identidad administrada se usa. En este caso, la biblioteca de identidades de Azure le exige que especifique la identidad deseada para evitar posibles problemas de tiempo de ejecución en el futuro (por ejemplo, si se agrega una nueva identidad administrada asignada por el usuario o si está habilitada la identidad administrada asignada por el sistema). Por lo tanto, tendrá que especificar el valor de clientId incluso si solo se define una identidad administrada asignada por el usuario y no hay ninguna identidad administrada asignada por el sistema.


1. Para usar los valores de App Configuration y las referencias de Key Vault, actualice *Program.cs* como se muestra a continuación. Este código llama a `SetCredential` como parte de `ConfigureKeyVault` para indicar al proveedor de configuración qué credencial debe usar al autenticarse en Key Vault.

    ### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration((hostingContext, config) =>
               {
                   var settings = config.Build();
                   var credentials = new ManagedIdentityCredential();

                   config.AddAzureAppConfiguration(options =>
                   {
                       options.Connect(new Uri(settings["AppConfig:Endpoint"]), credentials)
                              .ConfigureKeyVault(kv =>
                              {
                                 kv.SetCredential(credentials);
                              });
                   });
               })
               .UseStartup<Startup>();
    ```

    ### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    var credentials = new ManagedIdentityCredential();

                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(new Uri(settings["AppConfig:Endpoint"]), credentials)
                               .ConfigureKeyVault(kv =>
                               {
                                   kv.SetCredential(credentials);
                               });
                    });
                });
            })
            .UseStartup<Startup>());
    ```
    ---

    Ya puede acceder a las referencias de Key Vault de igual forma que a cualquier otra clave de App Configuration. El proveedor de configuración usará el objeto `ManagedIdentityCredential` para autenticarse en Key Vault y recuperará el valor.

    > [!NOTE]
    > `ManagedIdentityCredential` solo funciona en entornos de Azure de servicios que admiten la autenticación de identidad administrada. No funciona en el entorno local. Use [`DefaultAzureCredential`](/dotnet/api/azure.identity.defaultazurecredential) para que el código funcione en entornos locales y de Azure, ya que se revertirá a algunas opciones de autenticación, incluida la identidad administrada.
    > 
    > En caso de que quiera usar una **identidad administrada asignada por el usuario** con `DefaultAzureCredential` cuando se implemente en Azure, [especifique el valor de clientId](/dotnet/api/overview/azure/identity-readme#specifying-a-user-assigned-managed-identity-with-the-defaultazurecredential).

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="deploy-from-local-git"></a>Implementación desde Git local

La manera más fácil de habilitar la implementación de Git local para la aplicación con el servidor de compilación Kudu es utilizar [Azure Cloud Shell](https://shell.azure.com).

### <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="enable-local-git-with-kudu"></a>Habilitación de GIT local con Kudu
Si no tiene un repositorio de Git local para la aplicación, deberá inicializar uno. Para inicializar un repositorio de Git local, ejecute los comandos siguientes desde el directorio del proyecto de la aplicación:

```cmd
git init
git add .
git commit -m "Initial version"
```

Para habilitar la implementación de Git local para la aplicación con el servidor de compilación Kudu, ejecute [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/#az-webapp-deployment-source-config-local-git) en Cloud Shell.

```azurecli-interactive
az webapp deployment source config-local-git --name <app_name> --resource-group <group_name>
```

Este comando genera algo similar a la salida siguiente:

```json
{
  "url": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git"
}
```

### <a name="deploy-your-project"></a>Implementación del proyecto

En la _ventana del terminal local_, agregue una instancia remota de Azure al repositorio de Git local. Reemplace _\<url>_ por la dirección URL del Git remoto que ha obtenido en [Habilitación de GIT local con Kudu](#enable-local-git-with-kudu).

```bash
git remote add azure <url>
```

Realice la insercion en la instancia remota de Azure para implementar la aplicación con el comando siguiente. Cuando se le pida una contraseña, escriba la contraseña que creó en [Configuración de un usuario de implementación](#configure-a-deployment-user). No use la misma contraseña que para iniciar sesión en Azure Portal.

```bash
git push azure main
```

Es posible que vea la automatización específica para el entorno de tiempo de ejecución en la salida, como MSBuild para ASP.NET, `npm install` para Node.js y `pip install` para Python.

### <a name="browse-to-the-azure-web-app"></a>Navegación hasta la aplicación web de Azure

Vaya a la aplicación web mediante un explorador para comprobar que el contenido se ha implementado.

```bash
http://<app_name>.azurewebsites.net
```

## <a name="use-managed-identity-in-other-languages"></a>Usar identidades administradas en otros idiomas

Los proveedores de App Configuration para .NET Framework y Java Spring también incluyen compatibilidad integrada con identidades administradas. Puede usar el punto de conexión de dirección URL del almacén en lugar de su cadena de conexión completa al configurar uno de estos proveedores.

Por ejemplo, puede actualizar la aplicación de consola de .NET Framework que se creó en el inicio rápido para especificar la siguiente configuración en el archivo *App.config*:

```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>

    <configBuilders>
        <builders>
            <add name="MyConfigStore" mode="Greedy" endpoint="${Endpoint}" type="Microsoft.Configuration.ConfigurationBuilders.AzureAppConfigurationBuilder, Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration" />
            <add name="Environment" mode="Greedy" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
        </builders>
    </configBuilders>

    <appSettings configBuilders="Environment,MyConfigStore">
        <add key="AppName" value="Console App Demo" />
        <add key="Endpoint" value ="Set via an environment variable - for example, dev, test, staging, or production endpoint." />
    </appSettings>
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha agregado una identidad administrada de Azure para optimizar el acceso a App Configuration y mejorar la administración de credenciales de su aplicación. Para más información sobre App Configuration, continúe con los ejemplos de la CLI de Azure.

> [!div class="nextstepaction"]
> [Ejemplos de CLI](./cli-samples.md)