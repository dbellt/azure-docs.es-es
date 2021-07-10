---
title: 'Azure Event Grid en Kubernetes: webhook como controlador de eventos'
description: En este artículo se describe cómo crear un tema de Event Grid en un clúster de Kubernetes que está conectado a Azure Arc y, a continuación, crear una suscripción para el tema.
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: quickstart
ms.openlocfilehash: d29583cecb1498c10320a844923067a48693480a
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112030312"
---
# <a name="route-cloud-events-to-webhooks-with-azure-event-grid-on-kubernetes"></a>Enrutar eventos en la nube a webhooks con Azure Event Grid en Kubernetes
En este inicio rápido, creará un tema en Event Grid en Kubernetes, creará una suscripción para el tema y, a continuación, enviará un evento de ejemplo al tema para probar el escenario. 

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]


## <a name="prerequisites"></a>Requisitos previos

1. [Conexión del clúster de Kubernetes a Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md).
1. [Instalación de la extensión de Event Grid en el clúster de Kubernetes](install-k8s-extension.md). Esta extensión implementa Event Grid en un clúster de Kubernetes. 
1. [Creación de una ubicación personalizada](../../azure-arc/kubernetes/custom-locations.md). Una ubicación personalizada representa un espacio de nombres del clúster y es el lugar donde se implementan los temas y las suscripciones de eventos.

## <a name="create-a-topic"></a>de un tema

### <a name="azure-cli"></a>Azure CLI
Ejecute el comando de la CLI de Azure siguiente para crear un tema:

```azurecli-interactive
az eventgrid topic create --name <EVENT GRID TOPIC NAME> \
                        --resource-group <RESOURCE GROUP NAME> \
                        --location <REGION> \
                        --kind azurearc \
                        --extended-location-name /subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.ExtendedLocation/customLocations/<CUSTOM LOCATION NAME> \
                        --extended-location-type customlocation \
                        --input-schema CloudEventSchemaV1_0
```
Especifique valores para los marcadores de posición antes de ejecutar el comando:
- Nombre del grupo de recursos de Azure en el que quiere que se cree el tema de Event Grid. 
- Nombre del tema. 
- Región del tema.
- En el identificador de recurso de la ubicación personalizada, especifique los valores siguientes:
    - Identificador de la suscripción de Azure en la que existe la ubicación personalizada.
    - Nombre del grupo de recursos que contiene la ubicación personalizada.
    - Nombre de la ubicación personalizada

Para obtener más información sobre el comando de la CLI, consulte [`az eventgrid topic create`](/cli/azure/eventgrid/topic#az_eventgrid_topic_create).

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje
Antes de crear una suscripción para el tema personalizado, cree un punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, se implementa una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. En la página del artículo, seleccione **Implementar en Azure** para implementar la solución en su suscripción. En Azure Portal, proporcione valores para los parámetros.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"  alt="Button to Deploy to Aquent." /></a>
1. La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

    Si se produce un error en la implementación, compruebe el mensaje de error. Puede deberse a que ya se ha utilizado el nombre del sitio web. Vuelva a implementar la plantilla y elija un nombre diferente para el sitio. 
1. Verá el sitio, pero aún no se ha publicado en él ningún evento.

   ![Visualización del nuevo sitio](../media/custom-event-quickstart-portal/view-site.png)

## <a name="create-a-subscription"></a>una suscripción
Los suscriptores pueden registrarse en eventos publicados en un tema. Para recibir cualquier evento, deberá crear una suscripción de Event Grid para un tema de interés. Una suscripción de eventos define el destino al que se envían esos eventos. Para obtener información sobre todos los destinos o controladores que se admiten, consulte [Controladores de eventos](event-handlers.md).


### <a name="azure-cli"></a>Azure CLI
Para crear una suscripción de eventos con un destino de WebHook (punto de conexión HTTPS), ejecute el comando de la CLI de Azure siguiente:

```azurecli-interactive
az eventgrid event-subscription create --name <EVENT SUBSCRIPTION NAME> \
                                    --source-resource-id /subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<TOPIC'S RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAme> \
                                    --endpoint https://<SITE NAME>.azurewebsites.net/api/updates
```

Especifique valores para los marcadores de posición antes de ejecutar el comando:
- Nombre de la suscripción de eventos que se va a crear. 

- En el **identificador de recurso del tema**, especifique los valores siguientes:
    - Identificador de la suscripción de Azure en la que quiere crear la suscripción. 
    - Nombre del grupo de recursos que contiene el tema.
    - Nombre del tema. 
- Para el punto de conexión, especifique el nombre del sitio web del visor de Event Grid.
    
Para obtener más información sobre el comando de la CLI, consulte [`az eventgrid event-subscription create`](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create).


## <a name="send-events-to-the-topic"></a>Envío de eventos al tema
1. Ejecute el comando siguiente para obtener el **punto de conexión** para el tema: Después de copiar y pegar el comando, actualice el **nombre del tema** y el **nombre del grupo de recursos** antes de ejecutar el comando. Publicará eventos de ejemplo en este punto de conexión del tema. 

    ```azurecli
    az eventgrid topic show --name <topic name> -g <resource group name> --query "endpoint" --output tsv
    ```
2. Ejecute el siguiente comando para obtener la **clave** para el tema personalizado: Después de copiar y pegar el comando, actualice el **nombre del tema** y el **nombre del grupo de recursos** antes de ejecutar el comando. Es la clave principal del tema. Para obtener esta clave en Azure Portal, cambie a la pestaña **Claves de acceso** de la página **Tema de Event Grid**. Para poder publicar un evento en un tema personalizado, necesita la clave de acceso. 

    ```azurecli
    az eventgrid topic key list --name <topic name> -g <resource group name> --query "key1" --output tsv
    ```
1. Ejecute el comando de **CURL** siguiente para publicar el evento. Especifique la dirección URL y la clave del punto de conexión de los pasos 1 y 2 antes de ejecutar el comando. 

    ```bash
    curl  -k -X POST -H "Content-Type: application/cloudevents-batch+json" -H "aeg-sas-key: <KEY_FROM_STEP_2>" -g <ENDPOINT_URL_FROM_STEP_1> \
    -d  '[{ 
          "specversion": "1.0",
          "type" : "orderCreated",
          "source": "myCompanyName/us/webCommerceChannel/myOnlineCommerceSiteBrandName",
          "id" : "eventId-n",
          "time" : "2020-12-25T20:54:07+00:00",
          "subject" : "account/acct-123224/order/o-123456",
          "dataSchema" : "1.0",
          "data" : {
             "orderId" : "123",
             "orderType" : "PO",
             "reference" : "https://www.myCompanyName.com/orders/123"
          }
    }]'
    ```
    
    Si la dirección URL del punto de conexión del tema del paso 1 es una dirección IP privada, como cuando el tipo de servicio del agente de Event Grid es ClusterIP, puede ejecutar **Curl** desde otro pod del clúster para tener acceso a esa dirección IP. Por ejemplo, puede realizar los pasos siguientes:

    1. Cree un archivo de manifiesto con la configuración siguiente. Es posible que desee ajustar ``dnsPolicy`` de acuerdo con sus necesidades. Para más información, consulte [DNS para servicios y pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).
    
        ```yml
        apiVersion: v1
        dnsPolicy: ClusterFirstWithHostNet
        hostNetwork: true
        kind: Pod
        metadata: 
          name: test-pod
        spec: 
          containers: 
            - 
              name: nginx
          emptyDir: {}
          image: nginx
          volumeMounts: 
            - 
              mountPath: /usr/share/nginx/html
              name: shared-data
          volumes: 
            - 
              name: shared-data  
        ```
    1. Cree el pod.
        ```bash
            kubectl apply -f <name_of_your_yaml_manifest_file>
        ```
    1. Compruebe que el pod se está ejecutando.
        ```bash
            kubectl get pod test-pod
        ```
    1. Inicio de una sesión de shell desde el contenedor
        ```bash
            kubectl exec --stdin --tty test-pod -- /bin/bash
        ```

    En este momento, tiene una sesión de shell desde un contenedor en ejecución en el clúster desde el que puede ejecutar el comando **Curl** descrito en un paso anterior.

    > [!NOTE]
    > Para obtener información sobre cómo enviar eventos en la nube mediante lenguajes de programación, consulte los ejemplos siguientes: 
    > - [C#](/samples/azure/azure-sdk-for-net/azure-event-grid-sdk-samples/).
    > - [Java](/samples/azure/azure-sdk-for-java/eventgrid-samples/)
    > - [JavaScript](/samples/azure/azure-sdk-for-js/eventgrid-javascript/) y [TypeScript](/samples/azure/azure-sdk-for-js/eventgrid-typescript/)
    > - [Python](/samples/azure/azure-sdk-for-python/eventgrid-samples/)

### <a name="verify-in-the-event-grid-viewer"></a>Comprobación en Visor de Event Grid
Ha desencadenado el evento y Event Grid ha enviado el mensaje al punto de conexión que configuró al realizar la suscripción. Vaya a la aplicación web para ver el evento que acaba de enviar.

:::image type="content" source="./media/create-topic-subscription/viewer-received-event.png" alt-text="Visualización del evento recibido en el visor de Event Grid":::

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 
- [Controladores de eventos y destinos](event-handlers.md): proporciona información sobre todos los controladores de eventos y los destinos compatibles con Event Grid en Kubernetes. 
- [Filtrado de eventos](filter-events.md): proporciona información sobre el filtrado de eventos en las suscripciones de eventos. 