---
title: 'Inicio rápido: Creación de un grupo de administración con .NET Core'
description: En este inicio rápido, se usa .NET Core para crear un grupo de administración para organizar los recursos en una jerarquía de recursos.
ms.date: 05/01/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 82382160465684b0da40094012c6cb6561bf5036
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108323292"
---
# <a name="quickstart-create-a-management-group-with-net-core"></a>Inicio rápido: Creación de un grupo de administración con .NET Core

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Una entidad de servicio de Azure, que incluya _clientId_ y _clientSecret_. Si no tiene una entidad de servicio para usarla con Azure Policy o si desea crear una, consulte [Bibliotecas de administración de Azure para la autenticación de .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Omita el paso en el que se instalan los paquetes de .NET Core, ya que lo haremos en los pasos siguientes.

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Configuración de la aplicación

Para habilitar .NET Core para administrar Azure grupos de administración, cree una aplicación de consola e instale los paquetes necesarios.

1. Compruebe que está instalada la versión más reciente de .NET Core (**3.1.8** como mínimo). Si aún no lo está, descárguela en [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Inicialice una nueva aplicación de consola de .NET Core denominada "mgCreate":

   ```dotnetcli
   dotnet new console --name "mgCreate"
   ```

1. Cambie los directorios a la nueva carpeta del proyecto e instale los paquetes necesarios para Azure Policy:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ManagementGroups --version 1.1.1-preview

   # Add the Azure app auth package for .NET Core
   dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.5.0
   ```

1. Reemplace el archivo `program.cs` predeterminado por el código siguiente y guarde el archivo actualizado:

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ManagementGroups;
   using Microsoft.Azure.Management.ManagementGroups.Models;
   
   namespace mgCreate
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strGroupId = args[3];
               string strDisplayName = args[4];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                      "https://management.core.windows.net",
                      new ClientCredential(strClientId, strClientSecret));
   
               using (var client = new ManagementGroupsAPIClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var mgRequest = new CreateManagementGroupRequest
                   {
                       DisplayName = strDisplayName
                   };
                   var response = await client.ManagementGroups.CreateOrUpdateAsync(strGroupId, mgRequest);
               }
           }
       }
   }
   ```

1. Compile y publique la aplicación de consola `mgCreate`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-the-management-group"></a>Creación del grupo de administración

En este inicio rápido, creará un nuevo grupo de administración en el grupo de administración raíz.

1. Cambie los directorios a `{run-folder}`, que ha definido con el comando `dotnet publish` anterior.

1. En el terminal, escriba el siguiente comando:

   ```bash
   mgCreate.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{groupID}" `
      "{displayName}"
   ```

Los comandos anteriores usan la siguiente información:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{clientId}`: reemplácelo por el identificador de cliente de la entidad de servicio
- `{clientSecret}`: reemplácelo por el secreto de cliente de la entidad de servicio
- `{groupID}`: reemplácelo por el id. del grupo de administración
- `{displayName}`: reemplácelo por el nombre descriptivo del grupo de administración

El resultado es un nuevo grupo de administración en el grupo de administración raíz.

## <a name="clean-up-resources"></a>Limpieza de recursos

- Elimine el grupo de administración nuevo a través del portal.

- Si desea quitar las aplicaciones de consola de .NET Core y los paquetes instalados, elimine la carpeta del proyecto `mgCreate`.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)