---
title: Concesión de acceso a una aplicación de funciones de Azure a una cuenta de Media Services
description: Supongamos que quiere construir una señal de "On Air" (En el aire) para su estudio de difusión. Puede determinar cuándo se ejecutan los eventos en directo de Media Services mediante la API de Media Services, pero puede ser difícil de llamar desde un dispositivo insertado. En su lugar, podría exponer una API HTTP para el dispositivo insertado mediante Azure Functions. Azure Functions podría llamar a Media Services para obtener el estado del evento en directo.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: inhenkel
ms.openlocfilehash: 6352c86581da356f4b2bab1a80dd463d502a9ae3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110481788"
---
# <a name="tutorial-give-an-azure-function-app-access-to-a-media-services-account"></a>Tutorial: Concesión de acceso a una aplicación de funciones de Azure a una cuenta de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Supongamos que quiere que los visitantes de su sitio web o aplicación sepan que están "en el aire" en el estudio de difusión. Puede determinar cuándo se ejecutan los eventos en directo de Media Services mediante la API de Media Services, pero puede ser difícil de llamar desde un dispositivo insertado. En su lugar, podría exponer una API HTTP para el dispositivo insertado mediante Azure Functions. Azure Functions podría llamar a Media Services para obtener el estado del evento en directo.

:::image type="content" source="media/diagrams/managed-identities-scenario-function-app-access-media-services-account.svg" alt-text="Identidades administradas que dan acceso a una aplicación de funciones a una cuenta de Media Services":::

En este tutorial se utiliza la API de Media Services 2020-05-01.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para usar cualquiera de los comandos de este artículo, primero debe haber iniciado sesión en la suscripción que desea usar.

 [!INCLUDE [Sign in to Azure with the CLI](./includes/task-sign-in-azure-cli.md)]

### <a name="set-subscription"></a>Definir la suscripción

Utilice este comando para establecer la suscripción con la que quiere trabajar.

[!INCLUDE [Sign in to Azure with the CLI](./includes/task-set-azure-subscription-cli.md)]

## <a name="prerequisites"></a>Requisitos previos

> [!IMPORTANT]
> Se recomienda trabajar con el inicio rápido de [creación de una función de C# en Azure desde la línea de comandos](../../azure-functions/create-first-function-cli-csharp.md) antes de intentar este tutorial.  Esto se debe a que los pasos de configuración incluidos son los mismos que se necesitan aquí.  También le dará la oportunidad de trabajar con un ejemplo sencillo en el que se basa este tutorial.

## <a name="resource-names"></a>Nombres de recurso

Antes de empezar, decida los nombres de los recursos que va a crear.  Deben identificarse fácilmente como un conjunto, especialmente si no planea usarlos una vez que haya terminado las pruebas. Las reglas de nomenclatura son diferentes para muchos tipos de recursos, por lo que es mejor mantener las minúsculas. Por ejemplo, "mediatest1rg" para el nombre del grupo de recursos y "mediatest1stor" para el nombre de la cuenta de almacenamiento. Use los mismos nombres para cada paso de este artículo.

Verá estos nombres mencionados en los comandos siguientes.  Los nombres de los recursos que necesitará son:

- your-resource-group-name
- your-storage-account-name
- your-media-services-account-name
- your-region
- your-function-name: use "OnAir"
- your-live-event-name: use "live1"
- your-ip-address: use "0.0.0./32"

> [!NOTE]
> Los guiones anteriores solo se usan para separar las palabras de guía. Debido a la incoherencia de asignar nombres a los recursos en los servicios de Azure, no use guiones al asignar un nombre a los recursos.<br/><br/>
> Todo lo representado por 00000000-0000-0000-0000000000 es el identificador único del recurso.  Normalmente, una respuesta JSON devuelve este valor. También se recomienda copiar y pegar las respuestas JSON en el Bloc de notas u otro editor de texto, ya que esas respuestas contendrán valores que necesitará para los comandos de la CLI posteriores.<br/><br/>
> Además, no se crea el nombre de la región.  Azure determina el nombre de la región.

### <a name="list-azure-regions"></a>Enumeración de regiones de Azure

Si no está seguro del nombre real de la región que se va a usar, utilice este comando para obtener una lista:

[!INCLUDE [List Azure regions with the CLI](./includes/task-list-azure-regions-cli.md)]

## <a name="sequence"></a>Secuencia

Cada uno de los pasos siguientes se realiza en un orden determinado porque uno o varios valores de las respuestas JSON se usan en el paso siguiente de la secuencia.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los recursos que va a crear deben pertenecer a un grupo de recursos. Cree primero el grupo de recursos. Utilizará `your-resource-group-name` en el paso de creación de la cuenta de Media Services y los pasos posteriores.

[!INCLUDE [Create a resource group with the CLI](./includes/task-create-resource-group-cli.md)]

## <a name="create-a-storage-account"></a>Creación de una cuenta de Storage

La cuenta de Media Services que va a crear debe tener una cuenta de almacenamiento asociada. Cree primero la cuenta de almacenamiento de la cuenta de Media Services. Usará `your-storage-account-name` en los pasos posteriores.

[!INCLUDE [Create a Storage account with the CLI](./includes/task-create-storage-account-cli.md)]

## <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

Ahora, cree la cuenta de Media Services. Busque lo siguiente.

[!INCLUDE [Create a Media Services account with the CLI](./includes/task-create-media-services-account-cli.md)]

## <a name="set-up-the-azure-function"></a>Configuración de Azure Functions

En esta sección, configurará la función de Azure.

### <a name="get-the-code"></a>Obtención del código

Use Azure Functions para crear el proyecto de función y recuperar el código de la plantilla HTTP.

```azurecli-interactive
func init MediaServicesLiveMonitor –dotnet
```

### <a name="change-directory"></a>Cambio de directorio

Asegúrese de cambiar el directorio de trabajo por el directorio del proyecto.  De lo contrario, se obtienen errores.

```azurecli-interactive
cd .\MediaServicesLiveMonitor\
```

### <a name="name-your-function"></a>Asignar un nombre a la función

```azurecli-interactive
func new --name OnAir --template "HTTP trigger" --authlevel "anonymous"
```

## <a name="configure-the-functions-project"></a>Configuración del proyecto de funciones

### <a name="add-items-to-the-csproj-file"></a>Adición de elementos en el archivo .csproj

En el archivo ".csproj" generado automáticamente, agregue las líneas siguientes a la primera `<ItemGroup>`:

```xml
<PackageReference Include="Microsoft.Azure.Management.Fluent" Version="1.37.0" />
<PackageReference Include="Microsoft.Azure.Management.Media" Version="3.0.4" />
```

### <a name="edit-the-onaircs-code"></a>Edición del código de OnAir.cs

Cambie el archivo `OnAir.cs`. Cambie las variables `subscriptionId`, `resourceGroup` y `mediaServicesAccountName` por las que ha decidido anteriormente.

```aspx-csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.Management.Media;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.Media.Models;

namespace MediaServicesLiveMonitor
{
    public static class LatestAsset
    {
        [FunctionName("OnAir")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string name = req.Query["name"];

            if (string.IsNullOrWhiteSpace(name))
            {
                return new BadRequestObjectResult("Missing 'name' URL parameter");
            }
            
            var credentials = SdkContext.AzureCredentialsFactory.FromSystemAssignedManagedServiceIdentity(
                MSIResourceType.AppService,
                AzureEnvironment.AzureGlobalCloud);

            var subscriptionId = "00000000-0000-0000-000000000000";    // Update
            var resourceGroup = "<your-resource-group-name>";                                    // Update
            var mediaServicesAccountName = "<your-media-services-account-name>";                    // Update

            var mediaServices = new AzureMediaServicesClient(credentials)
            {
                SubscriptionId = subscriptionId
            };

            var liveEvent = await mediaServices.LiveEvents.GetAsync(resourceGroup, mediaServicesAccountName, name);

            if (liveEvent == null)
            {
                return new NotFoundResult();
            }
            
            return new OkObjectResult(liveEvent.ResourceState == LiveEventResourceState.Running ? "On air" : "Off air");
        }
    }
}
```

## <a name="create-the-function-app"></a>Creación de la aplicación de funciones

Cree la aplicación de funciones para hospedar la función. El nombre es el mismo que el que ha descargado antes, `MediaServicesLiveMonitorApp`.

```azurecli-interactive

az functionapp create --resource-group <your-resource-group-name> --consumption-plan-location your-region --runtime dotnet --functions-version 3 --name MediaServicesLiveMonitorApp --storage-account mediatest3store --assign-identity "[system]"

```

Busque `principalId` en la respuesta JSON:

```json
{
...  
"identity": {
//Note the principalId value for the following step
    "principalId": "00000000-0000-0000-000000000000",
    "tenantId": "00000000-0000-0000-000000000000",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
  }
...
```

## <a name="grant-the-function-app-access-to-the-media-services-account-resource"></a>Concesión de acceso a la aplicación de funciones al recurso de la cuenta de servidor de Media Services

Para esta solicitud:

- `assignee` es el `principalId` que se encuentra en la respuesta JSON de `az functionapp create`.
- `scope` es el `id` que se encuentra en la respuesta JSON de `az ams account create`.  Consulte el ejemplo de respuesta JSON anterior.

```azurecli-interactive
az role assignment create --assignee 00000000-0000-0000-000000000000 --role "Media Services Account Administrator" --scope "/subscriptions/<the-subscription-id>/resourceGroups/<your-resource-group>/providers/Microsoft.Media/mediaservices/<your-media-services-account-name>"
```

## <a name="publish-the-function"></a>Publicación de la función

```azurecli-interactive
func azure functionapp publish MediaServicesLiveMonitorApp
```

## <a name="validation"></a>Validación

En un explorador, vaya a la dirección URL de la función, por ejemplo:

`https://mediaserviceslivemonitorapp.azurewebsites.net/api/onair?name=live1`

Esto debería devolver un error 404 (No encontrado) ya que el evento en directo aún no existe.

### <a name="create-a-live-event"></a>Creación de un evento en directo

```azurecli-interactive
az ams live-event create --resource-group test3 --account-name mediatest3 --name live1 --streaming-protocol RTMP
```

En un explorador, vaya a la dirección URL de la función, por ejemplo:

`https://mediaserviceslivemonitorapp.azurewebsites.net/api/onair?name=live1`

Ahora debería mostrarse como "Off Air" (Fuera del aire).

### <a name="start-the-live-event"></a>Inicio del evento en directo

Si inicia el evento en directo, la función debe devolver "On Air" (En el aire).

```azurecli-interactive
az ams live-event start live1
```

Esta función permite el acceso a cualquier persona. La protección del acceso a la función de Azure y el cableado de una luz "On Air" (En el aire) están fuera del ámbito de este documento.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene previsto usar los recursos que ha creado, elimine el grupo de recursos.

[!INCLUDE [Create a Media Services account with the CLI](./includes/clean-up-resources-cli.md)]
