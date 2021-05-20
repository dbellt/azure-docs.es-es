---
title: 'Tutorial: Enrutamiento de eventos de cambio de estado de directivas a Event Grid con la CLI de Azure'
description: En este tutorial, configurará Event Grid para escuchar eventos de cambio de estado de directivas y llamar a un webhook.
ms.date: 03/29/2021
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.openlocfilehash: d2017b4e85c4d064fc206d9b908be126559032bd
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108752046"
---
# <a name="tutorial-route-policy-state-change-events-to-event-grid-with-azure-cli"></a>Tutorial: Enrutamiento de eventos de cambio de estado de directivas a Event Grid con la CLI de Azure

En este artículo, aprenderá a configurar las suscripciones a eventos de Azure Policy para enviar eventos de cambio de estado de directivas a un punto de conexión web. Los usuarios de Azure Policy pueden suscribirse a los eventos emitidos cuando se producen cambios en el estado de la directiva en los recursos. Estos eventos pueden desencadenar webhooks, instancias de [Azure Functions](../../../azure-functions/index.yml), [colas de Azure Storage](../../../storage/queues/index.yml) o cualquier otro controlador de eventos que sea compatible con [Azure Event Grid](../../../event-grid/index.yml). Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. Sin embargo, para simplificar este tutorial, los eventos se envían a una aplicación web que recopila y muestra los mensajes.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Para realizar este inicio rápido es necesaria la versión 2.0.76 o superior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

- Incluso si ha usado anteriormente Azure Policy o Event Grid, vuelva a registrar sus proveedores de recursos respectivos:

  ```azurecli-interactive
  # Log in first with az login if you're not using Cloud Shell

  # Provider register: Register the Azure Policy provider
  az provider register --namespace Microsoft.PolicyInsights

  # Provider register: Register the Azure Event Grid provider
  az provider register --namespace Microsoft.EventGrid
  ```

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los temas de Event Grid son recursos de Azure y se deben colocar en un grupo de recursos de Azure. El grupo de recursos de Azure es una colección lógica en la que se implementan y administran los recursos de Azure.

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group).

En el ejemplo siguiente se crea un grupo de recursos llamado `<resource_group_name>` en la ubicación _eastus_. Reemplace `<resource_group_name>` por un nombre único para grupo de recursos.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az group create --name <resource_group_name> --location westus
```

## <a name="create-an-event-grid-system-topic"></a>Creación de un tema del sistema de Event Grid

Ahora que tenemos un grupo de recursos, vamos a crear un [tema del sistema](../../../event-grid/system-topics.md). Un tema del sistema en Event Grid representa uno o varios eventos publicados por los servicios de Azure, como Azure Policy y Azure Event Hubs. Este tema del sistema usa el tipo de tema `Microsoft.PolicyInsights.PolicyStates` para los cambios de estado de Azure Policy. Reemplace `<SubscriptionID>` en el parámetro **scope** por el identificador de la suscripción y `<resource_group_name>` en el parámetro **resource-group** por el grupo de recursos creado anteriormente.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az eventgrid system-topic create --name PolicyStateChanges --location global --topic-type Microsoft.PolicyInsights.PolicyStates --source "/subscriptions/<SubscriptionID>" --resource-group "<resource_group_name>"
```

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Antes de suscribirse al tema, vamos a crear el punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, se implementa una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

Reemplace `<your-site-name>` por un nombre único para la aplicación web. El nombre de la aplicación web debe ser único, ya que es parte de la entrada DNS.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az deployment group create \
  --resource-group <resource_group_name> \
  --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
  --parameters siteName=<your-site-name> hostingPlanName=viewerhost
```

La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

Debería ver el sitio, que no muestra ningún mensaje actualmente.

## <a name="subscribe-to-the-system-topic"></a>Suscripción al tema del sistema

Suscríbase a un tema que indique a Event Grid los eventos cuyo seguimiento desea realizar y el lugar al que deben enviarse dichos eventos. El ejemplo siguiente se suscribe al tema personalizado que ha creado y pasa la dirección URL de la aplicación web como el punto de conexión para recibir notificaciones de eventos. Reemplace `<event_subscription_name>` por un nombre para la suscripción de eventos. En `<resource_group_name>` y `<your-site-name>`, use los valores que creó anteriormente.

El punto de conexión de la aplicación web debe incluir el sufijo `/api/updates/`.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Create the subscription
az eventgrid system-topic event-subscription create \
  --name <event_subscription_name> \
  --resource-group <resource_group_name> \
  --system-topic-name PolicyStateChanges \
  --endpoint https://<your-site-name>.azurewebsites.net/api/updates
```

Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

:::image type="content" source="../media/route-state-change-events/view-subscription-event.png" alt-text="Captura de pantalla del evento de validación de la suscripción de Event Grid en la aplicación web creada previamente.":::

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En esta guía de inicio rápido, creará una asignación de directiva y asignará la definición **Requerir una etiqueta en los grupos de recursos**. Esta definición de directiva identifica los grupos de recursos a los que les falta configurar la etiqueta durante la asignación de directivas.

Ejecute el siguiente comando para crear una asignación de directiva cuyo ámbito sea el grupo de recursos que ha creado para contener el tema de Event Grid:

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az policy assignment create --name 'requiredtags-events' --display-name 'Require tag on RG' --scope '<ResourceGroupScope>' --policy '<policy definition ID>' --params '{ "tagName": { "value": "EventTest" } }'
```

El comando anterior usa la siguiente información:

- **Nombre**: el nombre real de la asignación. Para este ejemplo se ha utilizado _requiredtags-events_.
- **DisplayName**: nombre para mostrar de la asignación de directiva. En este caso, va a utilizar _Requerir una etiqueta en los grupos de recursos_.
- **Scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Puede abarcar desde una suscripción hasta grupos de recursos. Asegúrese de sustituir &lt;scope&gt; por el nombre del grupo de recursos. El formato de un ámbito de grupo de recursos es `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>`.
- **Directiva**: identificador de la definición de directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Requerir una etiqueta en los grupos de recursos_. Para obtener el identificador de definición de directiva, ejecute este comando: `az policy definition list --query "[?displayName=='Require a tag on resource groups']"`

Después de crear la asignación de directiva, espere a que aparezca en la aplicación web la notificación de eventos **Microsoft.PolicyInsights.PolicyStateCreated**. El grupo de recursos creado muestra el valor _NonCompliant_ para `data.complianceState` en el inicio.

:::image type="content" source="../media/route-state-change-events/view-policystatecreated-event.png" alt-text="Captura de pantalla del evento de creación de estado de directiva de la suscripción de Event Grid para el grupo de recursos en la aplicación web creada previamente.":::

> [!NOTE]
> Si el grupo de recursos hereda otras asignaciones de directiva de la suscripción o de la jerarquía de grupos de administración, también se muestran los eventos de cada uno de ellos. Confirme que el evento corresponde a la asignación de este tutorial mediante la evaluación de la propiedad `data.policyDefinitionId`.

## <a name="trigger-a-change-on-the-resource-group"></a>Desencadenamiento de un cambio en el grupo de recursos

Para que el grupo de recursos sea compatible, se requiere una etiqueta con el nombre **EventTest**. Agregue la etiqueta al grupo de recursos con el siguiente comando; para ello, reemplace `<SubscriptionID>` por el identificador de la suscripción y `<ResourceGroup>` por el nombre del grupo de recursos:

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az tag create --resource-id '/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>' --tags EventTest=true
```

Después de agregar la etiqueta necesaria al grupo de recursos, espere a que aparezca en la aplicación web la notificación de eventos **Microsoft.PolicyInsights.PolicyStateChanged**. Al expandir el evento, ahora `data.complianceState` muestra el valor _Compliant_.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando con esta aplicación web y la suscripción a eventos de Azure Policy, no elimine los recursos creados en este artículo. Si no va a continuar, use el siguiente comando para eliminar los recursos creados en este artículo.

Sustituya `<resource_group_name>` por el nombre del grupo de recursos que ha creado.

```azurecli-interactive
az group delete --name <resource_group_name>
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear temas y suscripciones a eventos, puede obtener información sobre los eventos de cambio de estado de directivas y lo que Event Grid puede ayudarle a hacer:

- [Reacción a los eventos de cambio de estado de Azure Policy](../concepts/event-overview.md)
- [Azure Policy como un origen de Event Grid](../../../event-grid/event-schema-policy.md)
- [Una introducción a Azure Event Grid](../../../event-grid/overview.md)
