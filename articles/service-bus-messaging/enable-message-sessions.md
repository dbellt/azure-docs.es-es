---
title: Habilitación de las sesiones de mensajes de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo habilitar las sesiones de mensajes mediante Azure Portal, PowerShell, la CLI y los lenguajes de programación (C#, Java, Python y JavaScript).
ms.topic: how-to
ms.date: 04/19/2021
ms.openlocfilehash: 53791bb7dc1dec9bdf6c2c4aa8dda14728c2c465
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989151"
---
# <a name="enable-message-sessions-for-an-azure-service-bus-queue-or-a-subscription"></a>Habilitación de las sesiones de mensajes para una cola o suscripción de Azure Service Bus
Las sesiones de Azure Service Bus permiten la administración ordenada y conjunta de secuencias sin enlace de mensajes relacionados. Se pueden usar sesiones en patrones **FIFO (primero en entrar, primero en salir)** y de **solicitud-respuesta**. Para obtener más información, consulte [Sesiones de mensajes](message-sessions.md). En este artículo se muestran distintas formas de habilitar las sesiones para una cola o una suscripción de Service Bus. 

> [!IMPORTANT]
> - El nivel Básico de Service Bus no admite sesiones. Los niveles Estándar y Premium admiten sesiones. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).
> - Las sesiones de mensajes no se pueden habilitar ni deshabilitar después de crear la cola o la suscripción. Solo puede hacerlo en el momento de creación de la cola o suscripción. 

## <a name="using-azure-portal"></a>Uso de Azure Portal
Al crear una **cola** en Azure Portal, seleccione **Habilitar sesiones**, como se muestra en la imagen siguiente. 

:::image type="content" source="./media/message-sessions/queue-sessions.png" alt-text="Habilitación de la sesión en el momento de la creación de la cola":::

Al crear una suscripción para un tema en Azure Portal, seleccione **Habilitar sesiones**, como se muestra en la imagen siguiente. 

:::image type="content" source="./media/message-sessions/subscription-sessions.png" alt-text="Habilitación de la sesión en el momento de la creación de la suscripción":::

## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para **crear una cola con las sesiones de mensajes habilitadas**, use el comando [`az servicebus queue create`](/cli/azure/servicebus/queue#az_servicebus_queue_create) con `--enable-session` establecido en `true`.

```azurecli-interactive
az servicebus queue create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --enable-session true
```

Para **crear una suscripción para un tema con las sesiones de mensajes habilitadas**, use el comando [`az servicebus topic subscription create`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_create) con `--enable-session` establecido en `true`.

```azurecli-interactive
az servicebus topic subscription create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysubscription \
    --enable-session true
```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Para **crear una cola con las sesiones de mensajes habilitadas**, use el comando [`New-AzServiceBusQueue`](/powershell/module/az.servicebus/new-azservicebusqueue) con `-RequiresSession` establecido en `$True`. 

```azurepowershell-interactive
New-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -RequiresSession $True
```

Para **crear una suscripción para un tema con las sesiones de mensajes habilitadas**, use el comando [`New-AzServiceBusSubscription`](/powershell/module/az.servicebus/new-azservicebussubscription) con `-RequiresSession` establecido en `true`. 

```azurepowershell-interactive
New-AzServiceBusSubscription -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -TopicName mytopic `
    -SubscriptionName mysubscription `
    -RequiresSession $True
```

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager
Para **crear una cola con las sesiones de mensajes habilitadas**, establezca `requiresSession` en `true` en la sección de propiedades de la cola. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/queues](/azure/templates/microsoft.servicebus/namespaces/queues?tabs=json). 

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
            "requiresSession": true
          }
        }
      ]
    }
  ]
}

```

Para **crear una suscripción para un tema con las sesiones de mensajes habilitadas**, establezca `requiresSession` en `true` en la sección de propiedades de la suscripción. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus/namespaces/topics/subscriptions](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions?tabs=json). 

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
                "requiresSession": true
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