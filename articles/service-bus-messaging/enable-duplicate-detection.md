---
title: 'Habilitación de la detección de mensajes duplicados: Azure Service Bus'
description: En este artículo se explica cómo habilitar la detección de mensajes duplicados mediante Azure Portal, PowerShell, la CLI y los lenguajes de programación (C#, Java, Python y JavaScript).
ms.topic: how-to
ms.date: 04/19/2021
ms.openlocfilehash: 1e299b86991a7785cc89f2e72227e7a8ced883d7
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989133"
---
# <a name="enable-duplicate-message-detection-for-an-azure-service-bus-queue-or-a-topic"></a>Habilitación de la detección de mensajes duplicados para una cola o un tema de Azure Service Bus
Cuando se habilita la detección de duplicados para una cola o un tema, Azure Service Bus conserva el historial de todos los mensajes enviados a la cola o al tema durante un período de tiempo configurado. Durante ese intervalo, la cola o el tema no almacenarán ningún mensaje duplicado. Al habilitar esta propiedad, se garantiza exactamente una entrega durante un intervalo de tiempo concreto definido por el usuario. Para obtener más información, consulte [Detección de duplicados](duplicate-detection.md). En este artículo se muestran las distintas formas de habilitar la detección de mensajes duplicados para una cola o un tema de Service Bus. 


> [!NOTE]
> - El nivel Básico de Service Bus no admite la detección de duplicados. Los niveles Estándar y Premium admiten la detección de duplicados. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).
> - No puede habilitar ni deshabilitar la detección de duplicados después de crear la cola o el tema. Solo puede hacerlo en el momento de creación de la cola o el tema. 

## <a name="using-azure-portal"></a>Uso de Azure Portal
Al crear una **cola** en Azure Portal, seleccione **Habilitar detección de duplicados**, como se muestra en la imagen siguiente. Puede configurar el tamaño de la ventana de detección de duplicados al crear una cola o un tema. 

:::image type="content" source="./media/enable-duplicate-detection/create-queue.png" alt-text="Habilitación de la detección de duplicados en el momento de la creación de la cola":::

Al crear un tema en Azure Portal, seleccione **Habilitar detección de duplicados**, como se muestra en la imagen siguiente. 

:::image type="content" source="./media/enable-duplicate-detection/create-topic.png" alt-text="Habilitación de la detección de duplicados en el momento de la creación del tema":::

También puede configurar este valor para una cola o un tema existente, si había habilitado la detección de duplicados en el momento de crearlos. 

### <a name="update-duplicate-detection-window-size-for-an-existing-queue-or-a-topic"></a>Actualización del tamaño de la ventana de detección de duplicados de una cola o un tema existente
Para cambiar el tamaño de la ventana de detección de duplicados de una cola o un tema existente, seleccione **Cambiar** en la página **Información general** de la **Ventana de detección de duplicados**.  

#### <a name="queue"></a>Cola
:::image type="content" source="./media/enable-duplicate-detection/window-size.png" alt-text="Establecer el tamaño de la ventana de detección de duplicados para una cola":::

#### <a name="topic"></a>Tema
:::image type="content" source="./media/enable-duplicate-detection/window-size-topic.png" alt-text="Establecer el tamaño de la ventana de detección de duplicados para un tema":::


## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para **crear una cola con la detección de duplicados habilitada**, use el comando [`az servicebus queue create`](/cli/azure/servicebus/queue#az_servicebus_queue_create) con `--enable-duplicate-detection` establecido en `true`. 

```azurecli-interactive
az servicebus queue create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --enable-duplicate-detection true \
    --duplicate-detection-history-time-window P1D
```

Para **crear un tema con la detección de duplicados habilitada**, use el comando [`az servicebus topic create`](/cli/azure/servicebus/topic#az_servicebus_topic_create) con `--enable-duplicate-detection` establecido en `true`.

```azurecli-interactive
az servicebus topic create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name mytopic \
    --enable-duplicate-detection true \
    --duplicate-detection-history-time-window P1D
```

En los ejemplos anteriores, también se establece el tamaño de la ventana de detección de duplicados mediante el parámetro `--duplicate-detection-history-time-window`. El tamaño de la ventana se establece en un día. El valor predeterminado es de 10 minutos y el valor máximo permitido es de siete días.

Para **actualizar una cola con un nuevo tamaño de la ventana de detección**, use el comando [`az servicebus queue update`](/cli/azure/servicebus/queue#az_servicebus_queue_update) con el parámetro `--duplicate-detection-history-time-window`. En este ejemplo, el tamaño de la ventana se actualiza a siete días. 

```azurecli-interactive
az servicebus queue update \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --duplicate-detection-history-time-window P7D
```

Para **actualizar un tema con un nuevo tamaño de la ventana de detección**, use el comando [`az servicebus topic update`](/cli/azure/servicebus/topic#az_servicebus_topic_update) con el parámetro `--duplicate-detection-history-time-window`. En este ejemplo, el tamaño de la ventana se actualiza a siete días.

```azurecli-interactive
az servicebus topic update \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --duplicate-detection-history-time-window P7D
```
 
## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Para **crear una cola con la detección de duplicados habilitada**, use el comando [`New-AzServiceBusQueue`](/powershell/module/az.servicebus/new-azservicebusqueue) con `-RequiresDuplicateDetection` establecido en `$True`. 

```azurepowershell-interactive
New-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -RequiresDuplicateDetection $True `
    -DuplicateDetectionHistoryTimeWindow P1D
```

Para **crear un tema con la detección de duplicados habilitada**, use el comando [`New-AzServiceBusTopic`](/powershell/module/az.servicebus/new-azservicebustopic) con `-RequiresDuplicateDetection` establecido en `true`. 

```azurepowershell-interactive
New-AzServiceBusTopic -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic `
    -RequiresDuplicateDetection $True
    -DuplicateDetectionHistoryTimeWindow P1D
```

En los ejemplos anteriores, también se establece el tamaño de la ventana de detección de duplicados mediante el parámetro `-DuplicateDetectionHistoryTimeWindow`. El tamaño de la ventana se establece en un día. El valor predeterminado es de 10 minutos y el valor máximo permitido es de siete días.

Para **actualizar una cola con un nuevo tamaño de la ventana de detección**, vea el ejemplo siguiente.  En este ejemplo, el tamaño de la ventana se actualiza a siete días.

```azurepowershell-interactive
$queue=Get-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue 

$queue.DuplicateDetectionHistoryTimeWindow='P7D'

Set-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -QueueObj $queue
```

Para **actualizar un tema con un nuevo tamaño de la ventana de detección**, vea el ejemplo siguiente. En este ejemplo, el tamaño de la ventana se actualiza a siete días.

```azurepowershell-interactive
$topic=Get-AzServiceBusTopic -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic

$topic.DuplicateDetectionHistoryTimeWindow='P7D'

Set-AzServiceBusTopic -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic `
    -TopicObj $topic
```

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager
Para **crear una cola con la detección de duplicados habilitada**, establezca `requiresDuplicateDetection` en `true` en la sección de propiedades de la cola. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/queues](/azure/templates/microsoft.servicebus/namespaces/queues?tabs=json). Especifique un valor para la propiedad `duplicateDetectionHistoryTimeWindow` a fin de establecer el tamaño de la ventana de detección de duplicados. En el ejemplo siguiente, se establece en un día.  

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
            "requiresDuplicateDetection": true,
            "duplicateDetectionHistoryTimeWindow": "P1D"
          }
        }
      ]
    }
  ]
}

```

Para **crear un tema con la detección de duplicados habilitada**, establezca `requiresDuplicateDetection` en `true` en la sección de propiedades del tema. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/topics](/azure/templates/microsoft.servicebus/namespaces/topics?tabs=json). 

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
            "requiresDuplicateDetection": true,
            "duplicateDetectionHistoryTimeWindow": "P1D"
          }
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