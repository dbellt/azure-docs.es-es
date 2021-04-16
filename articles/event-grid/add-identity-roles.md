---
title: Adición de identidad administrada a un rol en un destino de Azure Event Grid
description: En este artículo se describe cómo agregar identidad administrada a roles de Azure en destinos como Azure Service Bus y Azure Event Hubs.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: 1578e4c24201614ce89351b3c3cee52a09cadc30
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280486"
---
# <a name="grant-managed-identity-the-access-to-event-grid-destination"></a>Concesión a la identidad administrada el acceso al destino de Event Grid
En esta sección se describe cómo agregar un rol de Azure a la identidad del tema del sistema, de un tema personalizado o del dominio. 

## <a name="prerequisites"></a>Requisitos previos
Para asignar una identidad administrada asignada por el sistema, siga las instrucciones de los siguientes artículos:

- [Dominios o temas personalizados](enable-identity-custom-topics-domains.md)
- [Temas del sistema](enable-identity-system-topics.md)

## <a name="supported-destinations-and-azure-roles"></a>Destinos admitidos y roles de Azure
Después de habilitar la identidad para el tema o el dominio personalizados de Event Grid, Azure crea automáticamente una identidad en Azure Active Directory. Agregue esta identidad a los roles de Azure adecuados para que el tema o el dominio personalizados puedan reenviar eventos a los destinos admitidos. Por ejemplo, agregue a la identidad el rol **Remitente de los datos de Azure Event Hubs** en un espacio de nombres de Azure Event Hubs de modo que el tema personalizado de Event Grid pueda reenviar eventos a los centros de eventos en dicho espacio de nombres. 

Actualmente, Azure Event Grid admite temas o dominios personalizados configurados con una identidad administrada asignada por el sistema para reenviar eventos a los siguientes destinos. En esta tabla también se proporcionan los roles que debe tener la identidad para que el tema personalizado pueda reenviar los eventos.

| Destination | Rol de Azure | 
| ----------- | --------- | 
| Colas y temas de Service Bus | [Emisor de datos de Azure Service Bus](../service-bus-messaging/authenticate-application.md#azure-built-in-roles-for-azure-service-bus) |
| Azure Event Hubs | [Emisor de datos de Azure Event Hubs](../event-hubs/authorize-access-azure-active-directory.md#azure-built-in-roles-for-azure-event-hubs) | 
| Azure Blob Storage | [Colaborador de datos de blobs de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) |
| Azure Queue Storage |[Emisor de mensajes de datos de la cola de Storage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) | 


## <a name="use-the-azure-portal"></a>Uso de Azure Portal
Puede usar Azure Portal para asignar un rol adecuado a la identidad del tema o del dominio personalizados, de modo que el tema o el dominio personalizados puedan reenviar eventos al destino. 

En el siguiente ejemplo se agrega el rol **Remitente de los datos de Azure Event Hubs** a una identidad administrada de un tema personalizado de Event Grid denominado **msitesttopic** en un espacio de nombres de Service Bus que contiene un recurso de cola o tema. Al agregar el rol en el nivel de espacio de nombres, el tema personalizado de Event Grid puede reenviar eventos a todas las entidades del espacio de nombres. 

1. Vaya al **espacio de nombres de Service Bus** en [Azure Portal](https://portal.azure.com). 
1. Seleccione **Control de acceso** en el panel izquierdo. 
1. Seleccione **Agregar** en la sección **Agregar una asignación de roles**. 
1. En la página **Agregar una asignación de roles**, siga estos pasos:
    1. Seleccione el rol. En este caso, es **Emisor de datos de Azure Event Hubs** 
    1. Seleccione la **identidad** del tema o del dominio personalizados de Event Grid. 
    1. Para guardar la configuración, seleccione **Guardar**.

Los pasos son similares para agregar una identidad a otros roles que se mencionan en la tabla. 

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
En el ejemplo de esta sección se muestra cómo usar la CLI de Azure para agregar una identidad a un rol de Azure. Los comandos de ejemplo son para temas personalizados de Event Grid. Los comandos de los dominios de Event Grid son similares. 

### <a name="get-the-principal-id-for-the-custom-topics-system-identity"></a>Obtención del identificador de entidad de seguridad de la identidad del sistema del tema personalizado 
En primer lugar, obtenga el identificador de entidad de seguridad de la identidad administrada por el sistema del tema personalizado y asigne a la identidad los roles adecuados.

```azurecli-interactive
topic_pid=$(az ad sp list --display-name "$<TOPIC NAME>" --query [].objectId -o tsv)
```

### <a name="create-a-role-assignment-for-event-hubs-at-various-scopes"></a>Creación de una asignación de roles para centros de eventos en varios ámbitos 
En el siguiente ejemplo de la CLI se muestra cómo agregar el rol **Emisor de datos de Azure Event Hubs** a la identidad de un tema personalizado en el nivel de espacio de nombres o en el nivel de centro de eventos. Si crea la asignación de roles en el nivel de espacio de nombres, el tema personalizado puede reenviar eventos a todos los centros de eventos de ese espacio de nombres. Si la crea en el nivel de centro de eventos, el tema personalizado solo puede reenviar eventos a ese centro de eventos específico. 


```azurecli-interactive
role="Azure Event Hubs Data Sender" 
namespaceresourceid=$(az eventhubs namespace show -n $<EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 
eventhubresourceid=$(az eventhubs eventhub show -n <EVENT HUB NAME> --namespace-name <EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one event hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$eventhubresourceid" 
```

### <a name="create-a-role-assignment-for-a-service-bus-topic-at-various-scopes"></a>Creación de una asignación de roles de un tema de Service Bus en varios ámbitos 
En el siguiente ejemplo con la CLI se muestra cómo agregar el rol **Emisor de datos de Azure Event Hubs** a la identidad de un tema personalizado de Event Grid en el nivel de espacio de nombres o en el nivel de tema de Service Bus. Si crea la asignación de roles en el nivel de espacio de nombres, el tema de Event Grid puede reenviar eventos a todas las entidades (colas o temas de Service Bus ) dentro de ese espacio de nombres. Si la crea en el nivel de cola o tema de Service Bus, el tema personalizado de Event Grid solo puede reenviar eventos a esa cola o tema de Service Bus específicos. 

```azurecli-interactive
role="Azure Service Bus Data Sender" 
namespaceresourceid=$(az servicebus namespace show -n $RG\SB -g "$RG" --query "{I:id}" -o tsv 
sbustopicresourceid=$(az servicebus topic show -n topic1 --namespace-name $RG\SB -g "$RG" --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$sbustopicresourceid" 
```

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha asignado una identidad asignada por el sistema al tema del sistema, a un tema personalizado o a un dominio, y ha agregado la identidad a los roles adecuados en los destinos, consulte [Entrega de eventos con una identidad administrada](managed-service-identity.md) sobre la entrega de eventos a destinos con la identidad.


