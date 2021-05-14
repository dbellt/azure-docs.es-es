---
title: Creación de una cuenta de Purview mediante el SDK de .NET
description: Cree una cuenta de Azure Purview mediante el SDK de .NET.
author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 4/2/2021
ms.author: nayenama
ms.openlocfilehash: b3dc7bf8ac7650a7219c15a09a31d4dcf84a40bf
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587806"
---
# <a name="quickstart-create-a-purview-account-using-net-sdk"></a>Inicio rápido: Creación de una cuenta de Purview mediante el SDK de .NET

En este inicio rápido se describe cómo usar el SDK de .NET para crear una cuenta de Azure Purview. 

> [!IMPORTANT]
> Azure Purview se encuentra actualmente en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* La cuenta debe tener permiso para crear recursos en la suscripción

* Si **Azure Policy** impide a todas las aplicaciones crear una **cuenta de Storage** y un **espacio de nombres de EventHub**, debe crear una excepción de directiva mediante etiqueta, que se especificará durante el proceso de creación de una cuenta de Purview. La razón principal es que con cada cuenta de Purview creada se debe crear un grupo de recursos administrado y dentro de este, una cuenta de Storage y un espacio de nombres de EventHub. Para más información, consulte [Creación del portal del catálogo](create-catalog-portal.md).

### <a name="visual-studio"></a>Visual Studio

En el tutorial de este artículo se usa Visual Studio 2019. Los procedimientos para Visual Studio 2013, 2015 o 2017 son ligeramente diferentes.

### <a name="azure-net-sdk"></a>SDK de .NET para Azure

Descargue e instale el [SDK de .NET para Azure](https://azure.microsoft.com/downloads/) en la máquina.

## <a name="create-an-application-in-azure-active-directory"></a>Creación de una aplicación en Azure Active Directory

En las secciones de *Instrucciones: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos*, siga las instrucciones para realizar las siguientes tareas:

1. En [Crear una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal), cree una aplicación que represente la aplicación .NET que se va a crear en este tutorial. Para la dirección URL de inicio de sesión, puede proporcionar una dirección URL ficticia, tal como se muestra en el artículo (`https://contoso.org/exampleapp`).
2. En [Obtener valores para iniciar sesión](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), obtenga el **identificador de aplicación** y el **identificador de inquilino** y tome nota de estos valores que se usa más adelante en este tutorial. 
3. En [Certificados y secretos](../active-directory/develop/howto-create-service-principal-portal.md#authentication-two-options), obtenga la **clave de autenticación** y tome nota de este valor que se usa más adelante en este tutorial.
4. En [Asignar la aplicación a un rol](../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application), asigne la aplicación al rol **Colaborador** en el nivel de suscripción para que la aplicación pueda crear factorías de datos en la suscripción.

## <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

A continuación, cree una aplicación de consola .NET de C# en Visual Studio:

1. Inicie **Visual Studio**.
2. En la ventana Inicio, seleccione **Crear un nuevo proyecto** > **Aplicación de consola (.NET Framework)** . Se requiere .NET versión 4.5.2 o posterior.
3. En **Nombre del proyecto**, escriba **PurviewQuickStart**.
4. Seleccione **Crear** para crear el proyecto.

## <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.
2. En el panel **Consola del Administrador de paquetes**, ejecute los comandos siguientes para instalar los paquetes. Para más información, consulte el [paquete NuGet Microsoft.Azure.Management.Purview](https://www.nuget.org/packages/Microsoft.Azure.Management.Purview/).

    ```powershell
    Install-Package Microsoft.Azure.Management.Purview
    Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-purview-client"></a>Creación de un cliente de Purview

1. Abra **Program.cs** e incluya las siguientes instrucciones para agregar referencias a espacios de nombres.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Rest.Serialization;
      using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.Purview;
      using Microsoft.Azure.Management.Purview.Models;
      using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Agregue el siguiente código al método **main** que define las variables. Reemplace los marcadores de posición por sus propios valores. Para encontrar una lista de las regiones de Azure en las que Purview está disponible actualmente, busque en **Azure Purview** y seleccione las regiones que le interesen en la página siguiente: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).

   ```csharp
   // Set variables
   string tenantID = "<your tenant ID>";
   string applicationId = "<your application ID>";
   string authenticationKey = "<your authentication key for the application>";
   string subscriptionId = "<your subscription ID where the data factory resides>";
   string resourceGroup = "<your resource group where the data factory resides>";
   string region = "<the location of your resource group>";
   string purviewAccountName = 
       "<specify the name of purview account to create. It must be globally unique.>";
   ```

3. Agregue el código siguiente al método **Main**, que crea una instancia de la clase **PurviewManagementClient**. Este objeto se usa para crear una cuenta de Purview.

   ```csharp
   // Authenticate and create a purview management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync(
   "https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new PurviewManagementClient(cred)
   {
      SubscriptionId = subscriptionId           
   };
   ```

## <a name="create-a-purview-account"></a>Creación de una cuenta de Purview

Agregue el código siguiente al método **Main**, que crea una **cuenta de Purview**. 

```csharp
    // Create a purview Account
    Console.WriteLine("Creating Purview Account " + purviewAccountName + "...");
    Account account = new Account()
    {
    Location = region,
    Identity = new Identity(type: "SystemAssigned"),
    Sku = new AccountSku(name: "Standard", capacity: 4)
    };            
    try
    {
      client.Accounts.CreateOrUpdate(resourceGroup, purviewAccountName, account);
      Console.WriteLine(client.Accounts.Get(resourceGroup, purviewAccountName).ProvisioningState);                
    }
    catch (ErrorResponseModelException purviewException)
    {
      Console.WriteLine(purviewException.StackTrace);
    }
    Console.WriteLine(
      SafeJsonConvert.SerializeObject(account, client.SerializationSettings));
    while (client.Accounts.Get(resourceGroup, purviewAccountName).ProvisioningState ==
           "PendingCreation")
    {
      System.Threading.Thread.Sleep(1000);
    }
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
```

## <a name="run-the-code"></a>Ejecución del código

Compile e inicie la aplicación y, luego, compruebe la ejecución.

La consola imprime el progreso de la creación de la cuenta de Purview.

### <a name="sample-output"></a>Salida de ejemplo

```json
Creating Purview Account testpurview...
Succeeded
{
  "sku": {
    "capacity": 4,
    "name": "Standard"
  },
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "southcentralus"
}

Press any key to exit...
```

## <a name="verify-the-output"></a>Comprobación del resultado

Vaya a la página **Purview accounts** (Cuentas de Purview) en [Azure Portal](https://portal.azure.com) y compruebe la cuenta creada con el código anterior. 

## <a name="delete-purview-account"></a>Eliminación de la cuenta de Purview

Para eliminar una cuenta de Purview mediante programación, agregue las líneas de código siguientes al programa: 

```csharp
    Console.WriteLine("Deleting the Purview Account");
    client.Accounts.Delete(resourceGroup, purviewAccountName);
```

## <a name="check-if-purview-account-name-is-available"></a>Comprobación de si el nombre de la cuenta de Purview está disponible

Para comprobar la disponibilidad de una cuenta Purview, use el código siguiente: 

```csharp
    CheckNameAvailabilityRequest checkNameAvailabilityRequest = new CheckNameAvailabilityRequest()
    {
      Name = purviewAccountName,
      Type =  "Microsoft.Purview/accounts"
    };
    Console.WriteLine("Check Purview account name");
    Console.WriteLine(client.Accounts.CheckNameAvailability(checkNameAvailabilityRequest).NameAvailable);
```

El código anterior imprime "true" si el nombre está disponible y "false" si el nombre no está disponible.


## <a name="next-steps"></a>Pasos siguientes

Con el código de este tutorial se crea y elimina una cuenta de Purview y se comprueba la disponibilidad del nombre de la cuenta. Ahora puede descargar el SDK de .NET y conocer otras acciones del proveedor de recursos que puede realizar para una cuenta de Purview.

Vaya al siguiente artículo, donde aprenderá a permitir que los usuarios accedan a su cuenta de Azure Purview. 

> [!div class="nextstepaction"]
> [Adición de usuarios a una cuenta de Azure Purview](catalog-permissions.md)
