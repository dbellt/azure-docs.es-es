---
title: Obtención de una cadena de conexión de Azure Event Hubs | Microsoft Docs
description: En este artículo se proporcionan instrucciones acerca de cómo obtener una cadena de conexión que los clientes pueden usar para conectarse a Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4975fd3f0b33306f2a2d2b9b6ff2144dd7529321
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692123"
---
# <a name="get-an-event-hubs-connection-string"></a>Obtención de una cadena de conexión de Event Hubs

Para utilizar Event Hubs, debe crear un espacio de nombres de Event Hubs. Un espacio de nombres es un contenedor de ámbito para varios centros de eventos o temas de Kafka. Este espacio de nombres proporciona un [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) único. Una vez creado un espacio de nombres, puede obtener la cadena de conexión necesaria para comunicarse con Event Hubs.

La cadena de conexión para Azure Event Hubs tiene los siguientes componentes incrustados dentro de ella:

* FQDN = el FQDN del espacio de nombres de Event Hubs que creó (incluye el espacio de nombres de Event Hubs seguido de servicebus.windows.net).
* SharedAccessKeyName = el nombre que eligió para las claves SAS de la aplicación.
* SharedAccessKey = el valor generado de la clave.

La plantilla de la cadena de conexión tiene el siguiente aspecto
```
Endpoint=sb://<FQDN>/;SharedAccessKeyName=<KeyName>;SharedAccessKey=<KeyValue>
```

Un ejemplo de cadena de conexión podría ser similar a `Endpoint=sb://dummynamespace.servicebus.windows.net/;SharedAccessKeyName=DummyAccessKeyName;SharedAccessKey=5dOntTRytoC24opYThisAsit3is2B+OGY1US/fuL3ly=`

En este artículo se explican distintas formas de obtener la cadena de conexión.

## <a name="get-connection-string-from-the-portal"></a>Obtención de la cadena de conexión del portal
1. Inicie sesión en el [portal de Azure](https://portal.azure.com). 
2. Seleccione **Todos los servicios** en el menú de navegación izquierdo. 
3. Seleccione **Event Hubs** en la sección **Analytics**. 
4. En la lista de centros de eventos, seleccione el centro de eventos.
6. En la página **Espacio de nombres de Event Hubs**, seleccione **Directivas de acceso compartido** en el menú de la izquierda.

    ![Elemento de menú Directivas de acceso compartido](./media/event-hubs-get-connection-string/event-hubs-get-connection-string1.png)
7. Seleccione una **directiva de acceso compartido** en la lista de directivas. El valor predeterminado es: **RootManageSharedAccessPolicy**. Puede agregar una directiva con los permisos adecuados (lectura y escritura) y usar esa directiva. 

    ![Directivas de acceso compartido de Event Hubs](./media/event-hubs-get-connection-string/event-hubs-get-connection-string2.png)
8. Seleccione el botón **Copiar** junto al campo **Cadena de conexión: clave principal**. 

    ![Obtención de la cadena de conexión de Event Hubs](./media/event-hubs-get-connection-string/event-hubs-get-connection-string3.png)

## <a name="getting-the-connection-string-with-azure-powershell"></a>Obtención de la cadena de conexión con Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede usar [Get-AzEventHubKey](/powershell/module/az.eventhub/get-azeventhubkey) para obtener la cadena de conexión del nombre de directiva o regla específica, tal como se muestra a continuación:

```azurepowershell-interactive
Get-AzEventHubKey -ResourceGroupName dummyresourcegroup -NamespaceName dummynamespace -AuthorizationRuleName RootManageSharedAccessKey
```

## <a name="getting-the-connection-string-with-azure-cli"></a>Obtención de la cadena de conexión con la CLI de Azure
Puede usar lo siguiente para obtener la cadena de conexión para el espacio de nombres:

```azurecli-interactive
az eventhubs namespace authorization-rule keys list --resource-group dummyresourcegroup --namespace-name dummynamespace --name RootManageSharedAccessKey
```

O bien puede usar lo siguiente para obtener la cadena de conexión para una entidad EventHub:

```azurecli-interactive
az eventhubs eventhub authorization-rule keys list --resource-group dummyresourcegroup --namespace-name dummynamespace --eventhub-name dummyeventhub --name RootManageSharedAccessKey
```

Para más información sobre los comandos de la CLI de Azure para Event Hubs, consulte [CLI de Azure para Event Hubs](/cli/azure/eventhubs).

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs](./event-hubs-about.md)
* [Creación de un Centro de eventos](event-hubs-create.md)
