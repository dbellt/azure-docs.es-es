---
title: 'Inicio rápido: Nueva asignación de directivas con .NET Core'
description: En este inicio rápido, se usa .NET Core para crear una asignación de Azure Policy para identificar recursos no compatibles.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ed5fd2013f3efd02961c486363042b30c7ad98f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108773996"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-net-core"></a>Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles mediante .NET Core

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos. En esta guía de inicio rápido, se crea una asignación de directiva para identificar máquinas virtuales que no usan discos administrados. Cuando haya finalizado, podrá identificar máquinas virtuales que _no estén conformes_.

La biblioteca de .NET Core se usa para administrar los recursos de Azure. En esta guía se explica cómo usar la biblioteca de .NET Core para Azure Policy a fin de crear una asignación de directiva.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Una entidad de servicio de Azure, que incluya _clientId_ y _clientSecret_. Si no tiene una entidad de servicio para usarla con Azure Policy o si desea crear una, consulte [Bibliotecas de administración de Azure para la autenticación de .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Omita el paso en el que se instalan los paquetes de .NET Core, ya que lo haremos en los pasos siguientes.

## <a name="create-the-azure-policy-project"></a>Creación del proyecto de Azure Policy

Para habilitar .NET Core para administrar Azure Policy, cree una aplicación de consola e instale los paquetes necesarios.

1. Compruebe que está instalada la versión más reciente de .NET Core (**3.1.8** como mínimo). Si aún no lo está, descárguela en [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Inicialice una nueva aplicación de consola de .NET Core denominada "policyAssignment":

   ```dotnetcli
   dotnet new console --name "policyAssignment"
   ```

1. Cambie los directorios a la nueva carpeta del proyecto e instale los paquetes necesarios para Azure Policy:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceManager --version 3.10.0-preview

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
   using Microsoft.Azure.Management.ResourceManager;
   using Microsoft.Azure.Management.ResourceManager.Models;

   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];
               string strDisplayName = args[5];
               string strPolicyDefID = args[6];
               string strDescription = args[7];
               string strScope = args[8];

               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));

               using (var client = new PolicyClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyAssignment = new PolicyAssignment
                   {
                       DisplayName = strDisplayName,
                       PolicyDefinitionId = strPolicyDefID,
                       Description = strDescription
                   };
                   var response = await client.PolicyAssignments.CreateAsync(strScope, strName, policyAssignment);
               }
           }
       }
   }
   ```

1. Compile y publique la aplicación de consola `policyAssignment`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En este inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

1. Cambie los directorios a `{run-folder}`, que ha definido con el comando `dotnet publish` anterior.

1. En el terminal, escriba el siguiente comando:

   ```bash
   policyAssignment.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks" `
      "Audit VMs without managed disks Assignment" `
      "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      "Shows all virtual machines not using managed disks" `
      "{scope}"
   ```

Los comandos anteriores usan la siguiente información:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{clientId}`: reemplácelo por el identificador de cliente de la entidad de servicio
- `{clientSecret}`: reemplácelo por el secreto de cliente de la entidad de servicio
- `{subscriptionId}`: reemplácelo por el identificador de suscripción
- **name**: el nombre único para el objeto de asignación de directiva. En el ejemplo anterior se usa _audit-vm-manageddisks_.
- **displayName**: nombre para mostrar de la asignación de directiva. En este caso, usará _Auditoría de máquinas virtuales sin discos administrados_.
- **policyDefID**: la ruta de acceso de la definición de directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Auditoría de máquinas virtuales que no usan discos administrados_.
- **description**: una explicación más detallada de lo que hace la directiva o de por qué se asigna a este ámbito.
- **scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Podría abarcar desde un grupo de administración a un recurso individual. Asegúrese de reemplazar `{scope}` por uno de los siguientes patrones:
  - Grupo de administración: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Suscripción: `/subscriptions/{subscriptionId}`
  - Grupos de recursos: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Recurso: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

Ahora ya está listo para identificar los recursos no compatibles a fin de conocer el estado de cumplimiento de su entorno.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos sin compatibilidad

Ahora que se ha creado la asignación de directiva, puede identificar los recursos que no son compatibles.

1. Inicialice una nueva aplicación de consola de .NET Core denominada "policyCompliance":

   ```dotnetcli
   dotnet new console --name "policyCompliance"
   ```

1. Cambie los directorios a la nueva carpeta del proyecto e instale los paquetes necesarios para Azure Policy:

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.PolicyInsights --version 3.1.0

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
   using Microsoft.Azure.Management.PolicyInsights;
   using Microsoft.Azure.Management.PolicyInsights.Models;

   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];

               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));

               using (var client = new PolicyInsightsClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyQueryOptions = new QueryOptions
                   {
                       Filter = $"IsCompliant eq false and PolicyAssignmentId eq '{strName}'",
                       Apply = "groupby(ResourceId)"
                   };

                   var response = await client.PolicyStates.ListQueryResultsForSubscriptionAsync(
                       "latest", strSubscriptionId, policyQueryOptions);
                   Console.WriteLine(response.Odatacount);
               }
           }
       }
   }
   ```

1. Compile y publique la aplicación de consola `policyAssignment`:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

1. Cambie los directorios a `{run-folder}`, que ha definido con el comando `dotnet publish` anterior.

1. En el terminal, escriba el siguiente comando:

   ```bash
   policyCompliance.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks"
   ```

Los comandos anteriores usan la siguiente información:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{clientId}`: reemplácelo por el identificador de cliente de la entidad de servicio
- `{clientSecret}`: reemplácelo por el secreto de cliente de la entidad de servicio
- `{subscriptionId}`: reemplácelo por el identificador de suscripción
- **name**: el nombre único para el objeto de asignación de directiva. En el ejemplo anterior se usa _audit-vm-manageddisks_.

Los resultados de `response` coinciden con lo que ve en la pestaña **Compatibilidad de recursos** de una asignación de directiva en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

- Elimine la asignación de directiva _Audit VMs without managed disks Assignment_ (Auditar VM sin asignación de discos administrados) a través del portal. La definición de directiva está integrada, por lo que no hay que quitar ninguna definición.

- Si desea quitar las aplicaciones de consola de .NET Core y los paquetes instalados, elimine las carpetas del proyecto `policyAssignment` y `policyCompliance`.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de definiciones de directivas para asegurarse de la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
