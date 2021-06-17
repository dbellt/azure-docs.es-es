---
title: Habilitación del reenvío automático para las colas y las suscripciones de Azure Service Bus
description: En este artículo se explica cómo habilitar el reenvío automático para las colas y las suscripciones mediante Azure Portal, PowerShell, la CLI y los lenguajes de programación (C#, Java, Python y JavaScript).
ms.topic: how-to
ms.date: 04/19/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: da5af50941263c54a08de27df43fc85b5bd75b4a
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671501"
---
# <a name="enable-auto-forwarding-for-azure-service-bus-queues-and-subscriptions"></a>Habilitación del reenvío automático para las colas y las suscripciones de Azure Service Bus
La característica de reenvío automático de Service Bus permite encadenar una cola o suscripción a otra cola u otro tema que formen parte del mismo espacio de nombres. Cuando el reenvío automático está habilitado, Service Bus elimina automáticamente los mensajes que se colocan en la primera cola o suscripción (origen) y los coloca en la segunda cola o en el segundo tema (destino). Todavía se puede enviar un mensaje a la entidad de destino directamente. Para obtener más información, consulte [Encadenamiento de entidades de Service Bus con reenvío automático](service-bus-auto-forwarding.md). En este artículo se muestran las distintas formas de habilitar el reenvío automático para las colas y las suscripciones de Service Bus. 

> [!IMPORTANT]
> El nivel básico de Service Bus no admite la característica de reenvío automático. Los niveles Estándar y Prémium admiten la característica. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

## <a name="using-azure-portal"></a>Uso de Azure Portal
Al crear una **cola** o una **suscripción** para un tema en Azure Portal, seleccione la opción para **reenviar mensajes a la cola o el tema**, como se muestra en los ejemplos siguientes. A continuación, especifique si desea que los mensajes se reenvíen a una cola o a un tema. En este ejemplo, se elige la opción **Cola** y se selecciona una cola del mismo espacio de nombres.

### <a name="create-a-queue-with-auto-forwarding-enabled"></a>Creación de una cola con el reenvío automático habilitado
:::image type="content" source="./media/enable-auto-forward/create-queue.png" alt-text="Habilitación del reenvío automático en el momento de creación de la cola":::

### <a name="create-a-subscription-for-a-topic-with-auto-forwarding-enabled"></a>Creación de una suscripción para un tema con el reenvío automático habilitado
:::image type="content" source="./media/enable-auto-forward/create-subscription.png" alt-text="Habilitación del reenvío automático en el momento de creación de la suscripción":::

### <a name="update-the-auto-forward-setting-for-an-existing-queue"></a>Actualización de la configuración de reenvío automático para una cola existente
En la página **Información general** de la cola de Service Bus, seleccione el valor actual de la opción **Reenviar mensajes a**. En el ejemplo siguiente, el valor actual es **Deshabilitado**. En la ventana **Reenviar mensajes a cola o tema**, puede seleccionar la cola o el tema en el que desea que se reenvíen los mensajes. 

:::image type="content" source="./media/enable-auto-forward/queue-auto-forward.png" alt-text="Habilitación del reenvío automático para una cola existente":::

### <a name="update-the-auto-forward-setting-for-an-existing-subscription"></a>Actualización de la configuración de reenvío automático para una suscripción existente
En la página **Información general** de la suscripción de Service Bus, seleccione el valor actual de la opción **Reenviar mensajes a**. En el ejemplo siguiente, el valor actual es **Deshabilitado**. En la ventana **Reenviar mensajes a cola o tema**, puede seleccionar la cola o el tema en el que desea que se reenvíen los mensajes. 

:::image type="content" source="./media/enable-auto-forward/subscription-auto-forward.png" alt-text="Habilitación del reenvío automático para una suscripción existente":::

## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para **crear una cola con el reenvío automático habilitado**, use el comando [`az servicebus queue create`](/cli/azure/servicebus/queue#az_servicebus_queue_create) con `--forward-to` establecido en el nombre de la cola o el tema al que desea reenviar los mensajes. 

```azurecli-interactive
az servicebus queue create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --forward-to myqueue2
```

Para **actualizar la configuración de reenvío automático para una cola existente**, use el comando [`az servicebus queue update`](/cli/azure/servicebus/queue#az_servicebus_queue_update) con `--forward-to` establecido en el nombre de la cola o tema al que desea reenviar los mensajes. 

```azurecli-interactive
az servicebus queue update \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --forward-to myqueue2
```


Para **crear una suscripción para un tema con el reenvío automático habilitado**, use el comando [`az servicebus topic subscription create`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_create) con `--forward-to` establecido en el nombre de la cola o el tema al que desea reenviar los mensajes.

```azurecli-interactive
az servicebus topic subscription create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysubscription \
    --forward-to myqueue2
```

Para **actualizar la configuración de reenvío automático para una suscripción para un tema**, use el comando [`az servicebus topic subscription update`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_update) con `--forward-to` establecido en el nombre de la cola o tema al que desea reenviar los mensajes.

```azurecli-interactive
az servicebus topic subscription create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysubscription \
    --forward-to myqueue2
```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Para **crear una cola con el reenvío automático habilitado**, use el comando [`New-AzServiceBusQueue`](/powershell/module/az.servicebus/new-azservicebusqueue) con `-ForwardTo` establecido en el nombre de la cola o el tema al que desea reenviar los mensajes. 

```azurepowershell-interactive
New-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -ForwardTo myqueue2
```

Para **actualizar la configuración de reenvío automático para una cola existente**, use el comando [`Set-AzServiceBusQueue`](/powershell/module/az.servicebus/set-azservicebusqueue) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
$queue=Get-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue 

$queue.ForwardTo='myqueue2'

Set-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -QueueObj $queue
``` 

Para **crear una suscripción para un tema con el reenvío automático habilitado**, use el comando [`New-AzServiceBusSubscription`](/powershell/module/az.servicebus/new-azservicebussubscription) con `-ForwardTo` establecido en el nombre de la cola o el tema al que desea reenviar los mensajes.

```azurepowershell-interactive
New-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -TopicName mytopic `
    -SubscriptionName mysubscription `
    -ForwardTo myqueue2
```

Para **actualizar la configuración de reenvío automático para una suscripción existente**, vea el ejemplo siguiente.

```azurepowershell-interactive
$subscription=Get-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -TopicName mytopic `
    -SubscriptionName mysub

$subscription.ForwardTo='mytopic2'

Set-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic `
    -SubscriptionName mysub `
    -SubscriptionObj $subscription 
```

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager
Para **crear una cola con el reenvío automático habilitado**, establezca `forwardTo`, que aparece en la sección de propiedades de la cola, en el nombre de la cola o el tema al que desea reenviar los mensajes. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/queues](/azure/templates/microsoft.servicebus/namespaces/queues?tabs=json). 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serviceBusNamespaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusQueueName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Queue"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.ServiceBus/namespaces",
      "apiVersion": "2018-01-01-preview",
      "name": "[parameters('serviceBusNamespaceName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "type": "Queues",
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusQueueName')]",
          "dependsOn": [
            "[resourceId('Microsoft.ServiceBus/namespaces', parameters('serviceBusNamespaceName'))]"
          ],
          "properties": {
            "forwardTo": "myqueue2"
          }
        }
      ]
    }
  ]
}

```

Para **crear una suscripción para un tema con el reenvío automático habilitado**, establezca `forwardTo`, que aparece en la sección de propiedades de la cola, en el nombre de la cola o el tema al que desea reenviar los mensajes. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus/namespaces/topics/subscriptions](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions?tabs=json). 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "service_BusNamespace_Name": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusTopicName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Topic"
      }
    },
    "serviceBusSubscriptionName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Subscription"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "resources": [
    {
      "apiVersion": "2018-01-01-preview",
      "name": "[parameters('service_BusNamespace_Name')]",
      "type": "Microsoft.ServiceBus/namespaces",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusTopicName')]",
          "type": "topics",
          "dependsOn": [
            "[resourceId('Microsoft.ServiceBus/namespaces/', parameters('service_BusNamespace_Name'))]"
          ],
          "properties": {
            "maxSizeInMegabytes": 1024
          },
          "resources": [
            {
              "apiVersion": "2017-04-01",
              "name": "[parameters('serviceBusSubscriptionName')]",
              "type": "Subscriptions",
              "dependsOn": [
                "[parameters('serviceBusTopicName')]"
              ],
              "properties": {
                "forwardTo": "myqueue2"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="net"></a>.NET 

### <a name="azuremessagingservicebus-latest"></a>Azure.Messaging.ServiceBus (más reciente)
Para habilitar la característica de reenvío automático, puede establecer [CreateQueueOptions.ForwardTo](/dotnet/api/azure.messaging.servicebus.administration.createqueueoptions.forwardto) o [CreateSubscriptionOptions.ForwardTo](/dotnet/api/azure.messaging.servicebus.administration.createsubscriptionoptions.forwardto) y, a continuación, usar los métodos [CreateQueueAsync](/dotnet/api/azure.messaging.servicebus.administration.servicebusadministrationclient.createqueueasync#Azure_Messaging_ServiceBus_Administration_ServiceBusAdministrationClient_CreateQueueAsync_Azure_Messaging_ServiceBus_Administration_CreateQueueOptions_System_Threading_CancellationToken_) o [CreateSubscriptionAsync](/dotnet/api/azure.messaging.servicebus.administration.servicebusadministrationclient.createsubscriptionasync#Azure_Messaging_ServiceBus_Administration_ServiceBusAdministrationClient_CreateSubscriptionAsync_Azure_Messaging_ServiceBus_Administration_CreateSubscriptionOptions_System_Threading_CancellationToken_) que toman parámetros `CreateQueueOptions` o `CreateSubscriptionOptions`. 

### <a name="microsoftazureservicebus-legacy"></a>Microsoft.Azure.ServiceBus (heredado)
Para habilitar el reenvío automático, establezca las propiedades [QueueDescription.ForwardTo](/dotnet/api/microsoft.servicebus.messaging.queuedescription) o [SubscriptionDescription.ForwardTo](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) para el origen, como en este ejemplo:

```csharp
SubscriptionDescription srcSubscription = new SubscriptionDescription (srcTopic, srcSubscriptionName);
srcSubscription.ForwardTo = destTopic;
namespaceManager.CreateSubscription(srcSubscription));
```

## <a name="java"></a>Java

### <a name="azure-messaging-servicebus-latest"></a>azure-messaging-servicebus (más reciente)
Puede habilitar la característica de reenvío automático mediante el método [CreateQueueOptions.setForwardTo(String forwardTo)](/java/api/com.azure.messaging.servicebus.administration.models.createqueueoptions.setforwardto) o [CreateSubscriptionOptions.setForwardTo(String forwardTo)](/java/api/com.azure.messaging.servicebus.administration.models.createsubscriptionoptions.setforwardto) y, a continuación, mediante el método [createQueue](/java/api/com.azure.messaging.servicebus.administration.servicebusadministrationclient.createqueue#com_azure_messaging_servicebus_administration_ServiceBusAdministrationClient_createQueue_java_lang_String_com_azure_messaging_servicebus_administration_models_CreateQueueOptions_) o el método [createSubscription](/java/api/com.azure.messaging.servicebus.administration.servicebusadministrationclient.createsubscription#com_azure_messaging_servicebus_administration_ServiceBusAdministrationClient_createSubscription_java_lang_String_java_lang_String_com_azure_messaging_servicebus_administration_models_CreateSubscriptionOptions_) que toman los parámetros `CreateQueueOptions` o `CreateSubscriptionOptions`. 

### <a name="azure-servicebus-legacy"></a>azure-servicebus (heredado)
Puede habilitar el reenvío automático mediante [QueueDescription.setForwardTo(String forwardTo)](/java/api/com.microsoft.azure.servicebus.management.queuedescription.setforwardto#com_microsoft_azure_servicebus_management_QueueDescription_setForwardTo_java_lang_String_) o [SubscriptionDescription.setForwardTo(String forwardTo)](/java/api/com.microsoft.azure.servicebus.management.subscriptiondescription.setforwardto) para el origen. 


## <a name="next-steps"></a>Pasos siguientes
Pruebe los ejemplos en el lenguaje que prefiera para explorar las características de Azure Service Bus. 

- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión más reciente)](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/) 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión más reciente)](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)

A continuación, encontrará ejemplos de las bibliotecas cliente de .NET y Java anteriores:
- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus)
