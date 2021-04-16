---
title: Habilitación de una identidad administrada para el enrutamiento de eventos (versión preliminar) mediante la CLI
titleSuffix: Azure Digital Twins
description: Vea cómo habilitar una identidad asignada por el sistema para Azure Digital Twins y úsela para reenviar eventos mediante la CLI de Azure.
author: baanders
ms.author: baanders
ms.date: 02/09/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: e9eb5950b5cf7d4e7d0270deed72866ee28e3962
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107106911"
---
# <a name="enable-a-managed-identity-for-routing-azure-digital-twins-events-preview-azure-cli"></a>Habilitación de una identidad administrada para el enrutamiento de eventos de Azure Digital Twins (versión preliminar): Azure CLI

[!INCLUDE [digital-twins-managed-service-identity-selector.md](../../includes/digital-twins-managed-service-identity-selector.md)]

En este artículo se describe cómo habilitar una [identidad asignada por el sistema para una instancia de Azure Digital Twins](concepts-security.md#managed-identity-for-accessing-other-resources-preview) (actualmente en versión preliminar) y usar la identidad al reenviar eventos a destinos admitidos como [centro de eventos ](../event-hubs/event-hubs-about.md), [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)  y [Azure Storage Container](../storage/blobs/storage-blobs-introduction.md).

En este artículo se describe el proceso que utiliza la [**CLI de Azure**](/cli/azure/what-is-azure-cli).

Estos son los pasos que se describen en este artículo: 

1. Creación una instancia de Azure Digital Twins con una identidad asignada por el sistema o habilitación de la identidad asignada por el sistema en una instancia existente de Azure Digital Twins. 
1. Adición de un rol adecuado a la identidad. Por ejemplo, asigne el rol **Emisor de datos de Centro de eventos de Azure** a la identidad si el punto de conexión es Centro de eventos, o bien el **rol Emisor de datos de Azure Service Bus** si es Service Bus.
1. Cree un punto de conexión en Azure Digital Twins que pueda usar identidades asignadas por el sistema para la autenticación.

## <a name="enable-system-managed-identities-for-an-instance"></a>Habilitación de identidades administradas por el sistema para una instancia 

Cuando se habilita una identidad asignada por el sistema en la instancia de Azure Digital Twins, Azure crea automáticamente una identidad para ella en [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md). Esa identidad se puede usar para autenticarse en los puntos de conexión de Azure Digital Twins para el reenvío de eventos.

Puede activar identidades administradas por el sistema para una instancia de Azure Digital Twins **como parte de la configuración inicial de la instancia**, o bien **habilitarlas más adelante en una instancia que ya exista**.

Cualquiera de estos métodos de creación proporcionará las mismas opciones de configuración y el mismo resultado final para la instancia. En esta sección se describe cómo realizar ambos.

### <a name="add-a-system-managed-identity-during-instance-creation"></a>Adición de una identidad administrada por el sistema durante la creación de la instancia

En esta sección, aprenderá a habilitar una identidad administrada por el sistema en una instancia de Azure Digital Twins que se está creando actualmente. 

Esto se hace agregando un parámetro `--assign-identity` al comando `az dt create` que se usa para crear la instancia. (Para más información acerca de este comando, consulte la [documentación de referencia](/cli/azure/dt#az_dt_create) o las [instrucciones generales para configurar una instancia de Azure Digital Twins](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance)).

Para crear una instancia con una identidad administrada por el sistema, agregue el parámetro `--assign-identity` de la siguiente manera:

```azurecli-interactive
az dt create -n {new_instance_name} -g {resource_group} --assign-identity
```

### <a name="add-a-system-managed-identity-to-an-existing-instance"></a>Adición de una identidad administrada por el sistema a una instancia existente

En esta sección, agregará una identidad administrada por el sistema a una instancia de Azure Digital Twins que ya existe.

Esto también se realiza con el comando `az dt create` y el parámetro `--assign-identity`. En lugar de proporcionar un nuevo nombre para la instancia que va a crear, puede proporcionar el nombre de una instancia de que ya existe para actualizar el valor de `--assign-identity` para esa instancia.

El comando para **habilitar** la identidad administrada es el mismo que el comando para crear una instancia con una identidad administrada por el sistema. Lo único que cambia es el valor del parámetro de nombre de la instancia:

```azurecli-interactive
az dt create -n {name_of_existing_instance} -g {resource_group} --assign-identity
```

Para **deshabilitar** la identidad administrada en una instancia en la que está habilitada actualmente, use el siguiente comando parecido para establecer `--assign-identity` en `false`.

```azurecli-interactive
az dt create -n {name_of_existing_instance} -g {resource_group} --assign-identity false
```

## <a name="assign-azure-roles-to-the-identity"></a>Asignación de roles de Azure a la identidad 

Una vez que se haya creado una identidad asignada por el sistema para la instancia de Azure Digital Twins, tendrá que asignarle los roles adecuados para que se autentique con distintos tipos de [puntos de conexión ](concepts-route-events.md) a fin de reenviar eventos a los destinos admitidos. En esta sección se describen las opciones de rol y cómo asignarlos a la identidad asignada por el sistema.

>[!NOTE]
> Este es un paso importante; sin él, la identidad no podrá acceder a los puntos de conexión y los eventos no se entregarán.

### <a name="supported-destinations-and-azure-roles"></a>Destinos admitidos y roles de Azure 

Estos son los roles mínimos que necesita una identidad para acceder a un punto de conexión, en función del tipo de destino. Los roles con permisos más altos (como los de propietario de datos) también funcionarán.

| Destination | Rol de Azure |
| --- | --- |
| Azure Event Hubs | Emisor de datos de Azure Event Hubs |
| Azure Service Bus | Emisor de datos de Azure Service Bus |
| Contenedor de almacenamiento de Windows Azure | Colaborador de datos de blobs de almacenamiento |

Para obtener más información sobre los puntos de conexión, las rutas y los tipos de destinos que se admiten para el enrutamiento en Azure Digital Twins, vea [*Conceptos: rutas de eventos*](concepts-route-events.md).

### <a name="assign-the-role"></a>Asignación del rol

[!INCLUDE [digital-twins-permissions-required.md](../../includes/digital-twins-permissions-required.md)]

Puede agregar el parámetro `--scopes` al comando `az dt create` para asignar la identidad a uno o más ámbitos con un rol especificado. Esto se puede usar cuando se crea la instancia por primera vez, o después pasando el nombre de una instancia que ya existe.

En este ejemplo se crea una instancia con una identidad administrada por el sistema y se asigna esa identidad a un rol personalizado denominado `MyCustomRole` de un centro de eventos.

```azurecli-interactive
az dt create -n {instance_name} -g {resource_group} --assign-identity --scopes "/subscriptions/<subscription ID>/resourceGroups/<resource_group>/providers/Microsoft.EventHub/namespaces/<Event_Hubs_namespace>/eventhubs/<event_hub_name>" --role MyCustomRole
```

Para obtener más ejemplos de asignaciones de roles con este comando, consulte la documentación de referencia del comando [**az dt create**](/cli/azure/dt#az_dt_create).

Como alternativa, también puede usar el grupo de comandos [**az role assignment**](/cli/azure/role/assignment) para crear y administrar roles. Se puede usar para respaldar escenarios adicionales en los que no desea agrupar la asignación de roles con el comando create.

## <a name="create-an-endpoint-with-identity-based-authentication"></a>Creación de un punto de conexión con autenticación basada en identidades

Después de configurar una identidad administrada por el sistema para la instancia de Azure Digital Twins y asignarle los roles adecuados, puede crear [puntos de conexión](how-to-manage-routes-portal.md#create-an-endpoint-for-azure-digital-twins) de Azure Digital Twins que sean capaces de usar la identidad para la autenticación. Esta opción solo está disponible para los puntos de conexión de centro de eventos y de tipo Service Bus (no se admite para Event Grid).

>[!NOTE]
> No se puede editar un punto de conexión que ya se haya creado con la identidad basada en claves para cambiar a la autenticación basada en identidades. Debe elegir el tipo de autenticación al crear el punto de conexión por primera vez.

Esto se hace agregando un parámetro `--auth-type` al comando `az dt endpoint create` que se usa para crear el punto de conexión. (Para más información acerca de este comando, consulte la [documentación de referencia](/cli/azure/dt/endpoint/create) o las [instrucciones generales para configurar un punto de conexión de Azure Digital Twins](how-to-manage-routes-apis-cli.md#create-the-endpoint)).

Para crear un punto de conexión que use la autenticación basada en identidades, especifique el tipo de autenticación `IdentityBased` con el parámetro `--auth-type`. En el ejemplo siguiente se muestra esto para un punto de conexión de Event Hubs.

```azurecli-interactive
az dt endpoint create eventhub --endpoint-name {endpoint_name} --eventhub-resource-group {eventhub_resource_group} --eventhub-namespace {eventhub_namespace} --eventhub {eventhub_name} --auth-type IdentityBased -n {instance_name}
```

## <a name="considerations-for-disabling-system-managed-identities"></a>Consideraciones para deshabilitar las identidades administradas por el sistema

Como una identidad se administra de forma independiente a los puntos de conexión que la usan, es importante tener en cuenta los efectos que cualquier cambio en la identidad o sus roles puede tener en los puntos de conexión de la instancia de Azure Digital Twins. Si la identidad está deshabilitada o se quita un rol necesario para un punto de conexión, este puede dejar de estar accesible y se interrumpirá el flujo de eventos.

Para seguir usando un punto de conexión que se ha configurado con una identidad administrada que ahora se ha deshabilitado, tendrá que eliminar el punto de conexión y [volver a crearlo](how-to-manage-routes-apis-cli.md#create-an-endpoint-for-azure-digital-twins) con otro tipo de autenticación. Los eventos pueden tardar hasta una hora en reanudar la entrega al punto de conexión después de este cambio.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las identidades administradas en Azure AD: 
* [*Identidades administradas de recursos de Azure*](../active-directory/managed-identities-azure-resources/overview.md)