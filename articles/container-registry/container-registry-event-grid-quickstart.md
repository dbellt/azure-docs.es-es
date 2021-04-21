---
title: 'Inicio rápido: Envío de eventos a Event Grid'
description: En este tutorial, habilita eventos de Event Grid para el registro de contenedor, luego envía los eventos push y delete de la imagen del contenedor a una aplicación de ejemplo.
ms.topic: article
ms.date: 08/23/2018
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 43dea2640c9c9445ea464205f6c586bc1e486206
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784032"
---
# <a name="quickstart-send-events-from-private-container-registry-to-event-grid"></a>Inicio rápido: Envío de eventos de registro de contenedor privado a Event Grid

Azure Event Grid es un servicio de enrutamiento de eventos completamente administrado que ofrece un consumo de eventos uniforme mediante un modelo de publicación-suscripción. En este tutorial de inicio rápido, usará la CLI de Azure para crear un registro de contenedor, suscribirse a eventos del registro e implementar una aplicación web de ejemplo para recibir los eventos. Por último, desencadenará los eventos `push` y `delete` de la imagen de contenedor, y verá la carga del evento en la aplicación de ejemplo.

Después de completar los pasos descritos en este artículo, los eventos enviados desde el registro de contenedor a Event Grid aparecerán en la aplicación web de ejemplo:

![Explorador web que representa la aplicación web de ejemplo con tres eventos recibidos][sample-app-01]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Los comandos de la CLI de Azure que aparecen en este artículo tienen un formato para la shell de **Bash**. Si usa un shell diferente, como PowerShell o el símbolo del sistema, puede que tenga que ajustar los caracteres de continuación de línea o las líneas de asignación de variable según corresponda. En este artículo se usan variables para minimizar la edición de comandos necesaria.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. El comando [az group create][az-group-create] siguiente crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*. Si desea usar otro nombre para el grupo de recursos, establezca `RESOURCE_GROUP_NAME` en otro valor.

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup

az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

A continuación, implemente un registro de contenedor en el grupo de recursos con los siguientes comandos. Antes de ejecutar el comando [az acr create][az-acr-create], establezca `ACR_NAME` en un nombre para el registro. El nombre debe ser único dentro de Azure y está restringido a entre 5 y 50 caracteres alfanuméricos.

```azurecli-interactive
ACR_NAME=<acrName>

az acr create --resource-group $RESOURCE_GROUP_NAME --name $ACR_NAME --sku Basic
```

Una vez creado el registro, la CLI de Azure devuelve un resultado similar al siguiente:

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-08-16T20:02:46.569509+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry",
  "location": "eastus",
  "loginServer": "myregistry.azurecr.io",
  "name": "myregistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
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

Debería ver la aplicación de ejemplo que se representa sin ningún mensaje de evento:

![Explorador web que muestra la aplicación web de ejemplo sin ningún evento][sample-app-02]

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-registry-events"></a>Suscripción a eventos del registro

En Event Grid, suscríbase a un *tema* para indicarle a qué eventos desea realizar un seguimiento y adónde enviarlos. El siguiente comando [az eventgrid event-subscription create][az-eventgrid-event-subscription-create] se suscribe al registro de contenedor que creó y especifica la dirección URL de la aplicación web como punto de conexión al que deben enviarse los eventos. Las variables de entorno que rellenó en las secciones anteriores se reutilizan aquí, por lo que no se requiere ninguna modificación.

```azurecli-interactive
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
APP_ENDPOINT=https://$SITE_NAME.azurewebsites.net/api/updates

az eventgrid event-subscription create \
    --name event-sub-acr \
    --source-resource-id $ACR_REGISTRY_ID \
    --endpoint $APP_ENDPOINT
```

Una vez completada la suscripción, debería ver una salida similar a la siguiente:

```json
{
  "destination": {
    "endpointBaseUrl": "https://eventgridviewer.azurewebsites.net/api/updates",
    "endpointType": "WebHook",
    "endpointUrl": null
  },
  "filter": {
    "includedEventTypes": [
      "All"
    ],
    "isSubjectCaseSensitive": null,
    "subjectBeginsWith": "",
    "subjectEndsWith": ""
  },
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry/providers/Microsoft.EventGrid/eventSubscriptions/event-sub-acr",
  "labels": null,
  "name": "event-sub-acr",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "topic": "/subscriptions/<Subscription ID>/resourceGroups/myresourcegroup/providers/microsoft.containerregistry/registries/myregistry",
  "type": "Microsoft.EventGrid/eventSubscriptions"
}
```

## <a name="trigger-registry-events"></a>Desencadenamiento de eventos del registro

Ahora que la aplicación de ejemplo está en funcionamiento y que se suscribió al registro en Event Grid, está listo para generar algunos eventos. En esta sección, se usa ACR Tasks para compilar e insertar una imagen de contenedor en el registro. ACR Tasks es una característica de Azure Container Registry que permite crear imágenes de contenedor en la nube, sin necesidad de que el motor de Docker esté instalado en el equipo local.

### <a name="build-and-push-image"></a>Compilación e inserción de la imagen

Ejecute el siguiente comando de CLI de Azure para crear una imagen de contenedor desde el contenido de un repositorio de GitHub. De forma predeterminada, ACR Tasks inserta automáticamente una imagen compilada correctamente en el registro, lo que genera el evento `ImagePushed`.

```azurecli-interactive
az acr build --registry $ACR_NAME --image myimage:v1 -f Dockerfile https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
```

Debería ver un resultado similar al siguiente mientras ACR Tasks se compila e inserta la imagen. El resultado de ejemplo siguiente se ha truncado por razones de espacio.

```output
Sending build context to ACR...
Queued a build with build ID: aa2
Waiting for build agent...
2018/08/16 22:19:38 Using acb_vol_27a2afa6-27dc-4ae4-9e52-6d6c8b7455b2 as the home volume
2018/08/16 22:19:38 Setting up Docker configuration...
2018/08/16 22:19:39 Successfully set up Docker configuration
2018/08/16 22:19:39 Logging in to registry: myregistry.azurecr.io
2018/08/16 22:19:55 Successfully logged in
Sending build context to Docker daemon  94.72kB
Step 1/5 : FROM node:9-alpine
...
```

Para comprobar que la imagen compilada está en el registro, ejecute el siguiente comando para ver las etiquetas en el repositorio "myimage":

```azurecli-interactive
az acr repository show-tags --name $ACR_NAME --repository myimage
```

La etiqueta "v1" de la imagen que se ha compilado debe aparecer en el resultado, similar al siguiente:

```output
[
  "v1"
]
```

### <a name="delete-the-image"></a>Eliminación de la imagen

Ahora, genere un evento `ImageDeleted` eliminando la imagen con el comando [az acr repository delete][az-acr-repository-delete]:

```azurecli-interactive
az acr repository delete --name $ACR_NAME --image myimage:v1
```

Debería ver un resultado similar al siguiente, que pida confirmación para eliminar el manifiesto y las imágenes asociadas:

```output
This operation will delete the manifest 'sha256:f15fa9d0a69081ba93eee308b0e475a54fac9c682196721e294b2bc20ab23a1b' and all the following images: 'myimage:v1'.
Are you sure you want to continue? (y/n): 
```

## <a name="view-registry-events"></a>Visualización de eventos del registro

Ahora ha insertado una imagen en el registro y la ha eliminado. Vaya a la aplicación web de Visor de Event Grid, debería ver ambos eventos, `ImageDeleted` y `ImagePushed`. También puede que vea un evento de validación de suscripción generado al ejecutar el comando en la sección [Suscripción a eventos del registro](#subscribe-to-registry-events).

La captura de pantalla siguiente muestra la aplicación de ejemplo con los tres eventos, y el evento `ImageDeleted` está expandido para mostrar sus detalles.

![Explorador Web que muestra la aplicación de ejemplo con eventos ImagePushed y ImageDeleted][sample-app-03]

Felicidades. Si ve los eventos `ImagePushed` y `ImageDeleted`, el registro envía eventos a Event Grid, y Event Grid reenvía esos eventos al punto de conexión de la aplicación web.

## <a name="clean-up-resources"></a>Limpieza de recursos

Una vez que haya terminado con los recursos creados en este inicio rápido, puede eliminarlos todos con el siguiente comando de la CLI de Azure. Al eliminar un grupo de recursos, se eliminan de manera permanente todos los recursos que contiene.

**ADVERTENCIA**: Esta operación es irreversible. Asegúrese de que ya no necesita ninguno de los recursos en el grupo antes de ejecutar el comando.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP_NAME
```

## <a name="event-grid-event-schema"></a>Esquema de eventos de Event Grid

Puede encontrar la referencia del esquema de mensajes de eventos de Azure Container Registry en la documentación de Event Grid:

[Esquema de eventos de Azure Event Grid para Container Registry](../event-grid/event-schema-container-registry.md)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado un registro de contenedor, compilado una imagen con ACR Tasks, la ha eliminado y ha consumido los eventos del registro desde Event Grid con una aplicación de ejemplo. A continuación, vaya al tutorial de ACR Tasks para más información acerca de la compilación de imágenes de contenedor en la nube, incluidas compilaciones automatizadas en la actualización de imagen base:

> [!div class="nextstepaction"]
> [Compilación de imágenes de contenedor en la nube con ACR Tasks](container-registry-tutorial-quick-task.md)

<!-- IMAGES -->
[sample-app-01]: ./media/container-registry-event-grid-quickstart/sample-app-01.png
[sample-app-02]: ./media/container-registry-event-grid-quickstart/sample-app-02-no-events.png
[sample-app-03]: ./media/container-registry-event-grid-quickstart/sample-app-03-with-events.png

<!-- LINKS - External -->
[azure-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[sample-app]: https://github.com/dbarkol/azure-event-grid-viewer

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr/repository
[az-acr-repository-delete]: /cli/azure/acr/repository#az_acr_repository_delete
[az-eventgrid-event-subscription-create]: /cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create
[az-group-create]: /cli/azure/group#az_group_create
