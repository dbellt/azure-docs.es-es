---
title: Entrega de eventos, identidad de servicio administrado y vínculo privado
description: En este artículo se describe cómo habilitar Managed Service Identity para un tema de Azure Event Grid. Úselo para reenviar eventos a los destinos admitidos.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: 76f10b4627dc9578b1e616a868eab03431b59b69
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625285"
---
# <a name="event-delivery-with-a-managed-identity"></a>Entrega de evento con una identidad administrada
En este artículo se describe cómo usar una [identidad de servicio administrada](../active-directory/managed-identities-azure-resources/overview.md) en un tema del sistema, un tema personalizado o un dominio de Azure Event Grid. Úselo para reenviar eventos a destinos compatibles, como colas y temas de Service Bus, centros de eventos y cuentas de almacenamiento.



## <a name="prerequisites"></a>Prerrequisitos
1. Asigne una identidad asignada por el sistema a un tema del sistema, a un tema personalizado o a un dominio. 
    - Para temas y dominios personalizados, consulte [Asignación de una identidad administrada por el sistema a un tema o un dominio personalizados de Event Grid](enable-identity-custom-topics-domains.md). 
    - Para los temas del sistema, consulte [Asignación de una identidad administrada por el sistema a un tema del sistema de Event Grid](enable-identity-system-topics.md).
1. Agregue la identidad a un rol adecuado (por ejemplo, Remitente de los datos de Service Bus) en el destino (por ejemplo, una cola de Service Bus). Para obtener pasos detallados, consulte [Adición de una identidad a los roles de Azure en destinos de Azure Event Grid](add-identity-roles.md).

    > [!NOTE]
    > Actualmente, no es posible enviar eventos mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md). Para obtener más información, consulte la sección [Puntos de conexión privado](#private-endpoints) al final de este artículo. 

## <a name="create-event-subscriptions-that-use-an-identity"></a>Creación de suscripciones de eventos que usan una identidad
Una vez que tenga un tema o un dominio del sistema o un tema personalizados de Event Grid con una identidad administrada por el sistema y haya agregado a la identidad el rol adecuado en el destino, estará listo para crear suscripciones que usen la identidad. 

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
Al crear una suscripción de eventos, se muestra una opción para habilitar el uso de una identidad asignada por el sistema para un punto de conexión en la sección **DETALLES DE PUNTO DE CONEXIÓN**. 

![Habilitación de una identidad al crear una suscripción de eventos de una cola de Service Bus](./media/managed-service-identity/service-bus-queue-subscription-identity.png)

También puede habilitar el uso de una identidad asignada por el sistema para que se use en la cola de mensajes fallidos en la pestaña **Características adicionales**. 

![Habilitación de la identidad asignada por el sistema para la cola de mensajes fallidos](./media/managed-service-identity/enable-deadletter-identity.png)

### <a name="use-the-azure-cli---service-bus-queue"></a>Uso de la CLI de Azure: cola de Service Bus 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a una cola de Service Bus. La identidad debe ser miembro del rol **Emisor de datos de Azure Service Bus**. También debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** en la cuenta de almacenamiento que se usa para la cola de mensajes fallidos. 

#### <a name="define-variables"></a>Definición de variables
En primer lugar, especifique los valores de las siguientes variables que se usarán en el comando de la CLI. 

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the service bus queue resource id
queueid=$(az servicebus queue show --namespace-name <SERVICE BUS NAMESPACE NAME> --name <QUEUE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
sb_esname = "<Specify a name for the event subscription>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 
Este comando de ejemplo crea una suscripción de eventos de un tema personalizado de Event Grid con un tipo de punto de conexión establecido en **Cola de Service Bus**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sb_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery-and-dead-lettering"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos
Este comando de ejemplo crea una suscripción de eventos de un tema personalizado de Event Grid con un tipo de punto de conexión establecido en **Cola de Service Bus**. También especifica que la identidad administrada por el sistema se va a usar para la cola de mensajes fallidos. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sb_esnameq 
```

### <a name="use-the-azure-cli---event-hubs"></a>Uso de la CLI de Azure: Event Hubs 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a un centro de eventos. La identidad debe ser miembro del rol **Emisor de datos de Azure Event Hubs**. También debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** en la cuenta de almacenamiento que se usa para la cola de mensajes fallidos. 

#### <a name="define-variables"></a>Definición de variables
```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID CUSTOM TOPIC NAME>"

hubid=$(az eventhubs eventhub show --name <EVENT HUB NAME> --namespace-name <NAMESPACE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
eh_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 
Este comando de ejemplo crea una suscripción de eventos de un tema personalizado de Event Grid con un tipo de punto de conexión establecido en **Event Hubs**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type eventhub 
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    -n $sbq_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos 
Este comando de ejemplo crea una suscripción de eventos de un tema personalizado de Event Grid con un tipo de punto de conexión establecido en **Event Hubs**. También especifica que la identidad administrada por el sistema se va a usar para la cola de mensajes fallidos. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    --deadletter-identity-endpoint $eh_deadletterendpoint
    --deadletter-identity systemassigned 
    -n $eh_esname 
```

### <a name="use-the-azure-cli---azure-storage-queue"></a>Uso de la CLI de Azure: cola de Azure Storage 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a una cola de Azure Storage. La identidad debe ser miembro del rol **Emisor de mensajes de datos de la cola de Storage** de la cuenta de almacenamiento. También debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** en la cuenta de almacenamiento que se usa para la cola de mensajes fallidos.

#### <a name="define-variables"></a>Definición de variables  

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID CUSTOM TOPIC NAME>"

# get the storage account resource id
storageid=$(az storage account show --name <STORAGE ACCOUNT NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)

# build the resource id for the queue
queueid="$storageid/queueservices/default/queues/<QUEUE NAME>" 

sa_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 

```azurecli-interactive
az eventgrid event-subscription create 
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sa_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sa_esname 
```

## <a name="private-endpoints"></a>Puntos de conexión privados
Actualmente, no es posible enviar eventos mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md). Es decir, no hay compatibilidad si tiene requisitos de aislamiento de red estrictos por los que el tráfico de eventos entregados no debe abandonar el espacio de la IP privada. 

Sin embargo, si sus requisitos exigen una forma segura de enviar eventos mediante un canal cifrado y una identidad conocida del remitente (en este caso, Event Grid) por medio de un espacio de IP pública, puede enviar eventos a Event Hubs, Service Bus o Azure Storage mediante un tema o un dominio personalizados de Azure Event Grid con una identidad administrada por el sistema configurada como se muestra en este artículo. Después, puede usar un vínculo privado configurado en Azure Functions o en el webhook implementado en la red virtual para extraer eventos. Vea el siguiente ejemplo: [Conexión a puntos de conexión privados con Azure Functions](/samples/azure-samples/azure-functions-private-endpoints/connect-to-private-endpoints-with-azure-functions/).

En esta configuración, el tráfico pasa por la dirección IP pública/Internet desde Event Grid hasta Event Hubs, Service Bus o Azure Storage, pero el canal se puede cifrar y se usa una identidad administrada de Event Grid. Si configura Azure Functions o el webhook implementado en la red virtual para usar una instancia de Event Hubs, Service Bus o Azure Storage a través de un vínculo privado, esa sección del tráfico permanecerá en Azure de forma evidente.


## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las identidades administradas, consulte [¿Qué son las identidades administradas de recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md).
