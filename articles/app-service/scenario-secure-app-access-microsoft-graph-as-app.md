---
title: 'Tutorial: Accesos de aplicaciones web a Microsoft Graph como aplicación | Azure'
description: En este tutorial, aprenderá a acceder a los datos de Microsoft Graph mediante identidades administradas.
services: microsoft-graph, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 01/28/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1, devx-track-azurepowershell
ms.openlocfilehash: 5bb52799836b1975de9d936e04fb53987effb300
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832633"
---
# <a name="tutorial-access-microsoft-graph-from-a-secured-app-as-the-app"></a>Tutorial: Acceso a Microsoft Graph desde una aplicación protegida como aplicación

Aprenda a acceder a Microsoft Graph desde una aplicación web que se ejecuta en Azure App Service.

:::image type="content" alt-text="Diagrama que muestra el acceso a Microsoft Graph." source="./media/scenario-secure-app-access-microsoft-graph/web-app-access-graph.svg" border="false":::

Quiere llamar a Microsoft Graph para la aplicación web. Una manera segura de dar acceso a los datos a la aplicación web es usar una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md). Una identidad administrada de Azure Active Directory permite a App Service acceder a los recursos a través del control de acceso basado en roles, sin necesidad de credenciales de aplicación. Después de asignar una identidad administrada a la aplicación web, Azure se encarga de la creación y distribución de un certificado. No tiene que preocuparse de administrar secretos ni credenciales de aplicaciones.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear una identidad administrada asignada por el sistema en una aplicación web.
> * Agregar permisos de Microsoft Graph API a una identidad administrada.
> * Llamar a Microsoft Graph desde una aplicación web mediante identidades administradas.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una aplicación web que se ejecuta en Azure App Service que tiene el [módulo de autenticación y autorización de App Service habilitado](scenario-secure-app-authentication-app-service.md).

## <a name="enable-managed-identity-on-app"></a>Habilitación de la identidad administrada en la aplicación

Si crea y publica la aplicación web mediante Visual Studio, habilitará la identidad administrada en la aplicación. En App Service, seleccione **Identidad** en el panel izquierdo y, a continuación, **Asignado por el sistema**. Compruebe que la opción **Estado** está establecida en **Activo**. Si no es así, seleccione **Guardar** y, a continuación, **Sí** para habilitar la identidad administrada asignada por el sistema. Una vez habilitada la identidad administrada, el estado se establece en **Activo** y el identificador del objeto está disponible.

Tome nota del valor de **Id. de objeto**, ya que lo necesitará en el paso siguiente.

:::image type="content" alt-text="Captura de pantalla que muestra la identidad asignada por el sistema." source="./media/scenario-secure-app-access-microsoft-graph/create-system-assigned-identity.png":::

## <a name="grant-access-to-microsoft-graph"></a>Concesión de acceso a Microsoft Graph

Al acceder a Microsoft Graph, la identidad administrada debe tener los permisos adecuados para la operación que desea realizar. Actualmente, no hay ninguna opción para asignar estos permisos mediante Azure Portal. El siguiente script agregará los permisos de Microsoft Graph API solicitados al objeto de la entidad de servicio de la identidad administrada.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Install the module. (You need admin on the machine.)
# Install-Module AzureAD.

# Your tenant ID (in the Azure portal, under Azure Active Directory > Overview).
$TenantID="<tenant-id>"
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp-20201102125811"

# Get the ID of the managed identity for the web app.
$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid

# Check the Microsoft Graph documentation for the permission you need for the operation.
$PermissionName = "User.Read.All"

Connect-AzureAD -TenantId $TenantID

# Get the service principal for Microsoft Graph.
# First result should be AppId 00000003-0000-0000-c000-000000000000
$GraphServicePrincipal = Get-AzureADServicePrincipal -SearchString "Microsoft Graph" | Select-Object -first 1

# Assign permissions to the managed identity service principal.
$AppRole = $GraphServicePrincipal.AppRoles | `
Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}

New-AzureAdServiceAppRoleAssignment -ObjectId $spID -PrincipalId $spID `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole.Id
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az login

webAppName="SecureWebApp-20201106120003"

spId=$(az resource list -n $webAppName --query [*].identity.principalId --out tsv)

graphResourceId=$(az ad sp list --display-name "Microsoft Graph" --query [0].objectId --out tsv)

appRoleId=$(az ad sp list --display-name "Microsoft Graph" --query "[0].appRoles[?value=='User.Read.All' && contains(allowedMemberTypes, 'Application')].id" --output tsv)

uri=https://graph.microsoft.com/v1.0/servicePrincipals/$spId/appRoleAssignments

body="{'principalId':'$spId','resourceId':'$graphResourceId','appRoleId':'$appRoleId'}"

az rest --method post --uri $uri --body $body --headers "Content-Type=application/json"
```

---

Después de ejecutar el script, puede comprobar en [Azure Portal](https://portal.azure.com) que los permisos de API solicitados están asignados a la identidad administrada.

Vaya a **Azure Active Directory** y, luego, seleccione **Aplicaciones empresariales**. Este panel muestra todas las entidades de servicio del inquilino. En **Todas las aplicaciones**, seleccione la entidad de servicio para la identidad administrada. 

Si sigue este tutorial, hay dos entidades de servicio con el mismo nombre para mostrar (SecureWebApp2020094113531, por ejemplo). La entidad de servicio que tiene una **dirección URL de la página principal** representa la aplicación web en su inquilino. La entidad de servicio sin la **dirección URL de la página principal** representa la identidad administrada asignada por el sistema para la aplicación web. El valor de **Id. de objeto** de la identidad administrada coincide con el identificador de objeto de la identidad administrada que creó anteriormente.

Seleccione la entidad de servicio para la identidad administrada.

:::image type="content" alt-text="Captura de pantalla que muestra la opción Todas las aplicaciones." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-all-applications.png":::

En **Información general**, seleccione **Permisos**; verá los permisos agregados para Microsoft Graph.

:::image type="content" alt-text="Captura de pantalla que muestra el panel Permisos." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-permissions.png":::

## <a name="call-microsoft-graph-net"></a>Llamada a Microsoft Graph (.NET)

Para obtener una credencial de token que el código pueda usar para autorizar solicitudes para Microsoft Graph, se utiliza la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential). Cree una instancia de la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), que usa la identidad administrada para capturar los tokens y asociarlos al cliente del servicio. En el ejemplo de código siguiente se obtiene la credencial de token autenticada y se usa para crear un objeto de cliente del servicio que, luego, obtiene los usuarios del grupo.

Para ver este código como parte de una aplicación de ejemplo, consulte el [ejemplo en GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/3-WebApp-graphapi-managed-identity).

### <a name="install-the-microsoftidentitywebmicrosoftgraph-client-library-package"></a>Instalación del paquete de la biblioteca cliente Microsoft.Identity.Web.MicrosoftGraph

Instale el [paquete NuGet Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) en el proyecto utilizando la interfaz de la línea de comandos de .NET Core o la consola del administrador de paquetes de Visual Studio.

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

Abra una línea de comandos y cambie al directorio que contiene el archivo del proyecto.

Ejecute los comandos de instalación.

```dotnetcli
dotnet add package Microsoft.Identity.Web.MicrosoftGraph
```

# <a name="package-manager"></a>[Administrador de paquetes](#tab/package-manager)

Abra el proyecto o la solución en Visual Studio y abra la consola mediante **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.

Ejecute los comandos de instalación.
```powershell
Install-Package Microsoft.Identity.Web.MicrosoftGraph
```

---

### <a name="example"></a>Ejemplo

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Azure.Identity;
using Microsoft.Graph.Core;
using System.Net.Http.Headers;

...

public IList<MSGraphUser> Users { get; set; }

public async Task OnGetAsync()
{
    // Create the Microsoft Graph service client with a DefaultAzureCredential class, which gets an access token by using the available Managed Identity.
    var credential = new DefaultAzureCredential();
    var token = credential.GetToken(
        new Azure.Core.TokenRequestContext(
            new[] { "https://graph.microsoft.com/.default" }));

    var accessToken = token.Token;
    var graphServiceClient = new GraphServiceClient(
        new DelegateAuthenticationProvider((requestMessage) =>
        {
            requestMessage
            .Headers
            .Authorization = new AuthenticationHeaderValue("bearer", accessToken);

            return Task.CompletedTask;
        }));

    List<MSGraphUser> msGraphUsers = new List<MSGraphUser>();
    try
    {
        var users =await graphServiceClient.Users.Request().GetAsync();
        foreach(var u in users)
        {
            MSGraphUser user = new MSGraphUser();
            user.userPrincipalName = u.UserPrincipalName;
            user.displayName = u.DisplayName;
            user.mail = u.Mail;
            user.jobTitle = u.JobTitle;

            msGraphUsers.Add(user);
        }
    }
    catch(Exception ex)
    {
        string msg = ex.Message;
    }

    Users = msGraphUsers;
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya ha terminado con este tutorial y no necesita la aplicación web ni los recursos asociados, [elimine los recursos que creó](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
>
> * Crear una identidad administrada asignada por el sistema en una aplicación web.
> * Agregar permisos de Microsoft Graph API a una identidad administrada.
> * Llamar a Microsoft Graph desde una aplicación web mediante identidades administradas.

Aprenda a conectar una [aplicación de .NET Core](tutorial-dotnetcore-sqldb-app.md), [Python](tutorial-python-postgresql-app.md), [Java](tutorial-java-spring-cosmosdb.md) o [Node.js](tutorial-nodejs-mongodb-app.md) a una base de datos.
