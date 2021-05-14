---
title: Habilitación de la creación de particiones en las colas y los temas de Azure Service Bus
description: En este artículo se explica cómo habilitar la creación de particiones en las colas y los temas de Azure Service Bus mediante Azure Portal, PowerShell, la CLI y los lenguajes de programación (C#, Java, Python y JavaScript).
ms.topic: how-to
ms.date: 04/19/2021
ms.openlocfilehash: ac77dfc2e72d1d83c4424c05da40c5adba1e8e9d
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076696"
---
# <a name="enable-partitioning-for-an-azure-service-bus-queue-or-a-topic"></a>Habilitación de la creación de particiones para una cola o un tema de Azure Service Bus
Las particiones de Service Bus permiten que las colas y temas, o entidades de mensajería, se dividan entre varios agentes de mensajes y almacenes de mensajería. Las particiones significan que el rendimiento general de una entidad particionada ya no está limitado por el rendimiento de un solo agente o almacén de mensajería. Además, una interrupción temporal de un almacén de mensajería no hace que una cola o tema con particiones deje de estar disponible. Las colas y los temas con particiones pueden contener todas las características avanzadas de Service Bus, como la compatibilidad con transacciones y sesiones. Para obtener más información, consulte [Temas y colas con particiones](service-bus-partitioning.md). En este artículo se muestran las distintas formas de habilitar la detección de mensajes duplicados para una cola o un tema de Service Bus. 

> [!IMPORTANT]
> - La creación de particiones está disponible en la creación de la entidad para todas las colas y temas de las SKU básicas o estándar. No está disponible para la SKU de mensajería Premium, pero las entidades con particiones ya existentes en espacios de nombres Premium funcionarán según lo esperado.
> - No se puede cambiar la opción de creación de particiones en ninguna de las colas o temas existentes. La opción solo puede establecerse al crear una cola o un tema. 
> - En un espacio de nombres de nivel **Estándar**, puede crear colas y temas de Service Bus en tamaños de 1, 2, 3, 4 o 5 GB (el valor predeterminado es 1 GB). Con las particiones habilitadas, Service Bus crea 16 copias (16 particiones) de la entidad, cada una del mismo tamaño especificado. Por lo tanto, si crea una cola con un tamaño de 5 GB, con 16 particiones, el tamaño de cola máximo se convierte en (5 \* 16) = 80 GB. 
> - En un espacio de nombres de nivel **Premium** no se admiten entidades de creación de particiones. Aunque puede crear colas y temas de Service Bus en tamaños de 1, 2, 3, 4, 5, 10, 20, 40 u 80 GB (el valor predeterminado es 1 GB). Puede ver el tamaño máximo de la cola o el tema con particiones en la página de **información general** de [Azure Portal](https://portal.azure.com).


## <a name="using-azure-portal"></a>Uso de Azure Portal
Al crear una **cola** en Azure Portal, seleccione **Habilitar la creación de particiones**, como se muestra en la imagen siguiente. 

:::image type="content" source="./media/enable-partitions/create-queue.png" alt-text="Habilitación de la creación de particiones en el momento de creación de la cola":::

Al crear un tema en Azure Portal, seleccione **Habilitar la creación de particiones**, como se muestra en la imagen siguiente. 

:::image type="content" source="./media/enable-partitions/create-topic.png" alt-text="Habilitación de la creación de particiones en el momento de creación del tema":::

## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para **crear una cola con la creación de particiones habilitada**, use el comando [`az servicebus queue create`](/cli/azure/servicebus/queue#az_servicebus_queue_create) con `--enable-partitioning` establecido en `true`.

```azurecli-interactive
az servicebus queue create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --enable-partitioning true
```

Para **crear un tema con la creación de particiones habilitada**, use el comando [`az servicebus topic create`](/cli/azure/servicebus/topic#az_servicebus_topic_create) con `--enable-partitioning` establecido en `true`.

```azurecli-interactive
az servicebus topic create \
    --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name mytopic \
    --enable-partitioning true
```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Para **crear una cola con la creación de particiones habilitada**, use el comando [`New-AzServiceBusQueue`](/powershell/module/az.servicebus/new-azservicebusqueue) con `-EnablePartitioning` establecido en `$True`. 

```azurepowershell-interactive
New-AzServiceBusQueue -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -QueueName myqueue `
    -EnablePartitioning $True
```

Para **crear un tema con la creación de particiones habilitada**, use el comando [`New-AzServiceBusTopic`](/powershell/module/az.servicebus/new-azservicebustopic) con `-EnablePartitioning` establecido en `true`. 

```azurepowershell-interactive
New-AzServiceBusTopic -ResourceGroup myresourcegroup `
    -NamespaceName mynamespace `
    -Name mytopic `
    -EnablePartitioning $True
```

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager
Para **crear una cola con la creación de particiones habilitada**, establezca `enablePartitioning` en `true` en la sección de propiedades de la cola. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/queues](/azure/templates/microsoft.servicebus/namespaces/queues?tabs=json). 

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
            "enablePartitioning": true
          }
        }
      ]
    }
  ]
}

```

Para **crear un tema con la detección de duplicados habilitada**, establezca `enablePartitioning` en `true` en la sección de propiedades del tema. Para obtener más información, consulte el documento de [referencia de la plantilla de Microsoft.ServiceBus namespaces/topics](/azure/templates/microsoft.servicebus/namespaces/topics?tabs=json). 

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
            "enablePartitioning": true
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