---
title: Cómo enviar eventos de Azure SignalR Service a Event Grid
description: Una guía para mostrarle cómo habilitar los eventos de Event Grid para SignalR Service y cómo enviar eventos de dispositivo conectado o desconectado de la conexión de cliente a una aplicación de ejemplo.
services: signalr
author: chenyl
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: chenyl
ms.openlocfilehash: 65fb54dbfc5158ef8cc2b488f267c92f601502f6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784612"
---
# <a name="how-to-send-events-from-azure-signalr-service-to-event-grid"></a>Cómo enviar eventos de Azure SignalR Service a Event Grid

Azure Event Grid es un servicio de enrutamiento de eventos completamente administrado que ofrece un consumo de eventos uniforme mediante un modelo de publicación-suscripción. En esta guía, usará la CLI de Azure para crear una instancia de Azure SignalR Service, suscribirse a eventos de conexión e implementar una aplicación web de ejemplo para recibir los eventos. Por último, puede conectar y desconectar y ver la carga del evento en la aplicación de ejemplo.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Los comandos de la CLI de Azure que aparecen en este artículo tienen un formato para la shell de **Bash**. Si usa un shell diferente, como PowerShell o el símbolo del sistema, puede que tenga que ajustar los caracteres de continuación de línea o las líneas de asignación de variable según corresponda. En este artículo se usan variables para minimizar la edición de comandos necesaria.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. El comando [az group create][az-group-create] siguiente crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*. Si desea usar otro nombre para el grupo de recursos, establezca `RESOURCE_GROUP_NAME` en otro valor.

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup

az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## <a name="create-a-signalr-service"></a>Creación de una instancia de SignalR Service

A continuación, implemente una instancia de Azure SignalR Service en el grupo de recursos con los siguientes comandos.
```azurecli-interactive
SIGNALR_NAME=SignalRTestSvc

az signalr create --resource-group $RESOURCE_GROUP_NAME --name $SIGNALR_NAME --sku Free_F1
```

Una vez creada la instancia, la CLI de Azure devuelve una salida similar a la siguiente:

```json
{
  "externalIp": "13.76.156.152",
  "hostName": "clitest.servicedev.signalr.net",
  "hostNamePrefix": "clitest",
  "id": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/clitest1/providers/Microsoft.SignalRService/SignalR/clitest",
  "location": "southeastasia",
  "name": "clitest",
  "provisioningState": "Succeeded",
  "publicPort": 443,
  "resourceGroup": "clitest1",
  "serverPort": 443,
  "sku": {
    "capacity": 1,
    "family": null,
    "name": "Free_F1",
    "size": "F1",
    "tier": "Free"
  },
  "tags": null,
  "type": "Microsoft.SignalRService/SignalR",
  "version": "1.0"
}

```

## <a name="create-an-event-endpoint"></a>Creación de un punto de conexión de evento

En esta sección, se usará una plantilla de Resource Manager ubicada en un repositorio de GitHub para implementar una aplicación web de ejemplo creada previamente en Azure App Service. Posteriormente, se suscribirá a los eventos de Event Grid del registro y especificará esta aplicación como punto de conexión al que se enviarán los eventos.

Para implementar la aplicación de ejemplo, establezca `SITE_NAME` en un nombre único para la aplicación web y ejecute los comandos siguientes. El nombre del sitio debe ser único dentro de Azure porque forma parte del nombre de dominio completo (FQDN) de la aplicación web. En una sección posterior, navegará al FQDN de la aplicación en un explorador web para ver los eventos del registro.

```azurecli-interactive
SITE_NAME=<your-site-name>

az deployment group create \
    --resource-group $RESOURCE_GROUP_NAME \
    --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
    --parameters siteName=$SITE_NAME hostingPlanName=$SITE_NAME-plan
```

Una vez que haya realizado correctamente la implementación (podría tardar unos minutos), abra un explorador y vaya a la aplicación web para asegurarse de que se está ejecutando:

`http://<your-site-name>.azurewebsites.net`

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-registry-events"></a>Suscripción a eventos del registro

En Event Grid, suscríbase a un *tema* para indicarle a qué eventos desea realizar un seguimiento y adónde enviarlos. El siguiente comando [az eventgrid event-subscription create][az-eventgrid-event-subscription-create] se suscribe a la instancia de Azure SignalR Service que creó y especifica la dirección URL de la aplicación web como punto de conexión al que deben enviarse los eventos. Las variables de entorno que rellenó en las secciones anteriores se reutilizan aquí, por lo que no se requiere ninguna modificación.

```azurecli-interactive
SIGNALR_SERVICE_ID=$(az signalr show --resource-group $RESOURCE_GROUP_NAME --name $SIGNALR_NAME --query id --output tsv)
APP_ENDPOINT=https://$SITE_NAME.azurewebsites.net/api/updates

az eventgrid event-subscription create \
    --name event-sub-signalr \
    --source-resource-id $SIGNALR_SERVICE_ID \
    --endpoint $APP_ENDPOINT
```

Una vez completada la suscripción, debería ver una salida similar a la siguiente:

```JSON
{
  "deadLetterDestination": null,
  "destination": {
    "endpointBaseUrl": "https://$SITE_NAME.azurewebsites.net/api/updates",
    "endpointType": "WebHook",
    "endpointUrl": null
  },
  "filter": {
    "includedEventTypes": [
      "Microsoft.SignalRService.ClientConnectionConnected",
      "Microsoft.SignalRService.ClientConnectionDisconnected"
    ],
    "isSubjectCaseSensitive": null,
    "subjectBeginsWith": "",
    "subjectEndsWith": ""
  },
  "id": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/myResourceGroup/providers/Microsoft.SignalRService/SignalR/SignalRTestSvc/providers/Microsoft.EventGrid/eventSubscriptions/event-sub-signalr",
  "labels": null,
  "name": "event-sub-signalr",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "retryPolicy": {
    "eventTimeToLiveInMinutes": 1440,
    "maxDeliveryAttempts": 30
  },
  "topic": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/myResourceGroup/providers/microsoft.signalrservice/signalr/SignalRTestSvc",
  "type": "Microsoft.EventGrid/eventSubscriptions"
}
```

## <a name="trigger-registry-events"></a>Desencadenamiento de eventos del registro

Cambie al modo de servicio en `Serverless Mode` y configure una conexión de cliente a SignalR Service. Puede seguir este [ejemplo sin servidor](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/Serverless) como referencia.

```bash
git clone git@github.com:aspnet/AzureSignalR-samples.git

cd samples/Management

# Start the negotiation server
# Negotiation server is responsible for generating access token for clients
cd NegotitationServer
dotnet user-secrets set Azure:SignalR:ConnectionString "<Connection String>"
dotnet run

# Use a separate command line
# Start a client
cd SignalRClient
dotnet run
```

## <a name="view-registry-events"></a>Visualización de eventos del registro

Ahora se ha conectado un cliente a SignalR Service. Vaya a la aplicación web de Visor de Event Grid, debería ver un evento `ClientConnectionConnected`. Si termina el cliente, también verá un evento `ClientConnectionDisconnected`.

<!-- LINKS - External -->
[azure-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[sample-app]: https://github.com/dbarkol/azure-event-grid-viewer

<!-- LINKS - Internal -->
[az-eventgrid-event-subscription-create]: /cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create
[az-group-create]: /cli/azure/group#az_group_create
