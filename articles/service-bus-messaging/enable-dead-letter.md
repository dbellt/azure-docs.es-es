---
title: Habilitación de la cola de mensajes fallidos para las colas y las suscripciones de Azure Service Bus
description: En este artículo se explica cómo habilitar la cola de mensajes fallidos para las colas y las suscripciones mediante Azure Portal, PowerShell, la CLI y los lenguajes de programación (C#, Java, Python y JavaScript).
ms.topic: how-to
ms.date: 04/20/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 13082574510a0668076b7191b6b973e5a1a7cd77
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671484"
---
# <a name="enable-dead-lettering-on-message-expiration-for-azure-service-bus-queues-and-subscriptions"></a>Habilitación de la cola de mensajes fallidos cuando expire un mensaje para las colas y las suscripciones de Azure Service Bus
Las colas y suscripciones para temas de Azure Service Bus proporcionan una subcola secundaria, llamada cola de mensajes fallidos (DLQ). No es necesario crear explícitamente la cola de mensajes fallidos y no se puede eliminar ni administrar independientemente de la entidad principal. La finalidad de la cola de mensajes fallidos es mantener los mensajes que no se pueden entregar a ningún destinatario o los mensajes que no se pudieron procesar. Para más información, consulte [Introducción a las colas de mensajes fallidos de Service Bus](service-bus-dead-letter-queues.md). En este artículo se muestran las distintas formas de habilitar la cola de mensajes fallidos para las colas y las suscripciones de Service Bus. 

## <a name="using-azure-portal"></a>Uso de Azure Portal
Al crear una **cola** o una **suscripción** para un tema en Azure Portal, seleccione **Habilitar la cola de mensajes fallidos cuando expire un mensaje**, como se muestra en los ejemplos siguientes. 

### <a name="create-a-queue-with-dead-lettering-enabled"></a>Creación de una cola con la cola de mensajes fallidos habilitada
:::image type="content" source="./media/enable-dead-letter/create-queue.png" alt-text="Habilitación de la cola de mensajes fallidos en el momento de creación de la cola":::

### <a name="create-a-subscription-with-dead-lettering-enabled"></a>Creación de una suscripción con la cola de mensajes fallidos habilitada
:::image type="content" source="./media/enable-dead-letter/create-subscription.png" alt-text="Habilitación de la cola de mensajes fallidos en el momento de creación de la suscripción":::

### <a name="update-the-dead-lettering-on-message-expiration-setting-for-an-existing-queue"></a>Actualización de la opción de cola de mensajes fallidos cuando expire un mensaje para una cola existente
En la página **Información general** de la cola de Service Bus, seleccione el valor actual de la opción de la **cola de mensajes fallidos** al expirar un mensaje. En el ejemplo siguiente, el valor actual es **Disabled**. Puede habilitar o deshabilitar la cola de mensajes fallidos al expirar un mensaje en la ventana emergente. 

:::image type="content" source="./media/enable-dead-letter/queue-configuration.png" alt-text="Habilitación de la cola de mensajes fallidos al expirar un mensaje para una cola existente":::

### <a name="update-the-dead-lettering-on-message-expiration-setting-for-an-existing-subscription"></a>Actualización de la opción de cola de mensajes fallidos cuando expire un mensaje para una suscripción existente
En la página **Información general** de la suscripción de Service Bus, seleccione el valor actual de la opción de la **cola de mensajes fallidos** al expirar un mensaje. En el ejemplo siguiente, el valor actual es **Disabled**. Puede habilitar o deshabilitar la cola de mensajes fallidos al expirar un mensaje en la ventana emergente. 

:::image type="content" source="./media/enable-dead-letter/subscription-configuration.png" alt-text="Habilitación de la cola de mensajes fallidos al expirar un mensaje para una suscripción existente":::

## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para **crear una cola con la opción de cola de mensajes fallidos al expirar un mensaje habilitada**, use el comando [`az servicebus queue create`](/cli/azure/servicebus/queue#az_servicebus_queue_create) con `--enable-dead-lettering-on-message-expiration` configurado en `true`. 

```azurecli-interactive
az servicebus queue create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --enable-dead-lettering-on-message-expiration true
```

Para **habilitar la opción de cola de mensajes fallidos al expirar un mensaje en una cola existente**, use el comando [`az servicebus queue update`](/cli/azure/servicebus/queue#az_servicebus_queue_update) con `--enable-dead-lettering-on-message-expiration` establecido en `true`. 

```azurecli-interactive
az servicebus queue update \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --enable-dead-lettering-on-message-expiration true
```


Para **crear una suscripción a un tema con la opción de cola de mensajes fallidos al expirar un mensaje habilitada**, use el comando [`az servicebus topic subscription create`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_create) con `--enable-dead-lettering-on-message-expiration` configurado en `true`.

```azurecli-interactive
az servicebus topic subscription create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysubscription \
    --enable-dead-lettering-on-message-expiration true
```

Para **habilitar la opción de cola de mensajes fallidos al expirar un mensaje en una suscripción a un tema**, use el comando [`az servicebus topic subscription update`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_update) con `--enable-dead-lettering-on-message-expiration` establecido en `true`.

```azurecli-interactive
az servicebus topic subscription create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysubscription \
    --enable-dead-lettering-on-message-expiration true
```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Para **crear una cola con la opción de cola de mensajes fallidos al expirar un mensaje habilitada**, use el comando [`New-AzServiceBusQueue`](/powershell/module/az.servicebus/new-azservicebusqueue) con `-DeadLetteringOnMessageExpiration` configurado en `$True`. 

```azurepowershell-interactive
New-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -DeadLetteringOnMessageExpiration $True
```

Para **habilitar la opción de cola de mensajes fallidos al expirar un mensaje en una cola existente**, use el comando [`Set-AzServiceBusQueue`](/powershell/module/az.servicebus/set-azservicebusqueue) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
$queue=Get-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue 

$queue.DeadLetteringOnMessageExpiration=$True

Set-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -QueueObj $queue
``` 

Para **crear una suscripción a un tema con la opción de cola de mensajes fallidos al expirar un mensaje habilitada**, use el comando [`New-AzServiceBusSubscription`](/powershell/module/az.servicebus/new-azservicebussubscription) con `-DeadLetteringOnMessageExpiration` configurado en `$True`. 

```azurepowershell-interactive
New-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -TopicName mytopic `
    -SubscriptionName mysubscription `
    -DeadLetteringOnMessageExpiration $True
```

Para **habilitar la opción de cola de mensajes fallidos al expirar un mensaje en una suscripción existente**, vea el ejemplo siguiente.

```azurepowershell-interactive
$subscription=Get-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -TopicName mytopic `
    -SubscriptionName mysub

$subscription.DeadLetteringOnMessageExpiration=$True

Set-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic `
    -SubscriptionName mysub `
    -SubscriptionObj $subscription 
```

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager
Para **crear una cola con la opción de cola de mensajes fallidos al expirar un mensaje**, establezca `deadLetteringOnMessageExpiration` en la sección de las propiedades de la cola en `true`. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/queues](/azure/templates/microsoft.servicebus/namespaces/queues?tabs=json). 

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
            "deadLetteringOnMessageExpiration": true
          }
        }
      ]
    }
  ]
}

```

Para **crear una suscripción a un tema con la opción de cola de mensajes fallidos al expirar un mensaje**, establezca `deadLetteringOnMessageExpiration` en la sección de las propiedades de la cola en `true`. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus/namespaces/topics/subscriptions](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions?tabs=json). 

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
                "deadLetteringOnMessageExpiration": true
              }
            }
          ]
        }
      ]
    }
  ]
}
```


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
