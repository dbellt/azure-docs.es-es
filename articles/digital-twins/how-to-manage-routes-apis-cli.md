---
title: Administración de puntos de conexión y rutas (API y CLI)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar y administrar puntos de conexión y rutas de eventos para datos de Azure Digital Twins.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 11/18/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: e6b35031d976a11bdac6f38d74f9e02a0fc83302
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105936315"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-apis-and-cli"></a>Administración de puntos de conexión y rutas en Azure Digital Twins (API y CLI)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

En Azure Digital Twins, se pueden enrutar [notificaciones de eventos](how-to-interpret-event-data.md) a los servicios de bajada o recursos de proceso conectados. Para ello, primero se configuran los **puntos de conexión** que pueden recibir los eventos. Después, puede crear [**rutas de eventos**](concepts-route-events.md) que especifiquen qué eventos generados por Azure Digital Twins se entregan a los puntos de conexión.

Este artículo le guiará a través del proceso de creación de puntos de conexión y rutas con las [API REST](/rest/api/azure-digitaltwins/), el [SDK para .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client) y la [CLI de Azure Digital Twins](how-to-use-cli.md).

También puede administrar los puntos de conexión y las rutas con [Azure Portal](https://portal.azure.com). Para obtener una versión de este artículo que usa el portal en su lugar, consulte [*Procedimiento: Administración de puntos de conexión y rutas (portal)*](how-to-manage-routes-portal.md).

## <a name="prerequisites"></a>Requisitos previos

- Necesitará una **cuenta de Azure** (puede configurar una de forma gratuita [aquí](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)).
- Necesitará una **instancia de Azure Digital Twins** en su suscripción de Azure. Si no dispone de una instancia, puede crear una con los pasos de [*Procedimiento: Configuración de una instancia y autenticación*](how-to-set-up-instance-cli.md). Tenga a mano los siguientes valores de configuración para usarlos más adelante en este artículo:
    - Nombre de instancia
    - Grupo de recursos

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
    
## <a name="create-an-endpoint-for-azure-digital-twins"></a>Creación de un punto de conexión para Azure Digital Twins

Estos son los tipos de puntos de conexión admitidos que puede crear para la instancia:
* [Event Grid](../event-grid/overview.md)
* [Event Hubs](../event-hubs/event-hubs-about.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Para obtener más información sobre los distintos tipos de puntos de conexión, vea [*Elección entre los distintos servicios de mensajería de Azure*](../event-grid/compare-messaging-services.md).

En esta sección se explica cómo crear estos puntos de conexión mediante la CLI de Azure. También puede administrar los puntos de conexión con las [API del plano de control DigitalTwinsEndpoint](/rest/api/digital-twins/controlplane/endpoints).

[!INCLUDE [digital-twins-endpoint-resources.md](../../includes/digital-twins-endpoint-resources.md)]

### <a name="create-the-endpoint"></a>Creación del punto de conexión

Una vez que haya creado los recursos de punto de conexión, puede usarlos para un punto de conexión de Azure Digital Twins. En los siguientes ejemplos se muestra cómo crear puntos de conexión con el comando [az dt endpoint create](/cli/azure/ext/azure-iot/dt/endpoint/create) para la [CLI de Azure Digital Twins](how-to-use-cli.md). Reemplace los marcadores de posición de los comandos con los detalles de los recursos propios.

Para crear un punto de conexión de Event Grid:

```azurecli-interactive
az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
```

Para crear un punto de conexión de Event Hubs (autenticación basada en claves):
```azurecli-interactive
az dt endpoint create eventhub --endpoint-name <Event-Hub-endpoint-name> --eventhub-resource-group <Event-Hub-resource-group> --eventhub-namespace <Event-Hub-namespace> --eventhub <Event-Hub-name> --eventhub-policy <Event-Hub-policy> -n <your-Azure-Digital-Twins-instance-name>
```

Para crear un punto de conexión de tema de Service Bus (autenticación basada en claves):
```azurecli-interactive 
az dt endpoint create servicebus --endpoint-name <Service-Bus-endpoint-name> --servicebus-resource-group <Service-Bus-resource-group-name> --servicebus-namespace <Service-Bus-namespace> --servicebus-topic <Service-Bus-topic-name> --servicebus-policy <Service-Bus-topic-policy> -n <your-Azure-Digital-Twins-instance-name>
```

Después de ejecutar correctamente estos comandos, Event Grid, el centro de eventos o el tema de Service Bus estará disponible como un punto de conexión dentro de Azure Digital Twins, con el nombre que haya proporcionado con el argumento `--endpoint-name`. Normalmente se usará ese nombre como destino de una **ruta de eventos**, que se creará [más tarde en este artículo](#create-an-event-route).

#### <a name="create-an-endpoint-with-identity-based-authentication"></a>Creación de un punto de conexión con autenticación basada en identidades

También puede crear un punto de conexión que tenga autenticación basada en identidades, para usarlo con una [identidad administrada](concepts-security.md#managed-identity-for-accessing-other-resources-preview). Esta opción solo está disponible para los puntos de conexión de centro de eventos y de tipo Service Bus (no se admite para Event Grid).

A continuación se muestra el comando de la CLI para crear este tipo de punto de conexión. Necesitará los valores siguientes para conectarse a los marcadores de posición del comando:
* el id. de recurso de Azure de la instancia de Azure Digital Twins
* un nombre de punto de conexión
* un tipo de punto de conexión
* el espacio de nombres del recurso de punto de conexión
* el nombre del centro de eventos o del tema de Service Bus
* la ubicación de la instancia de Azure Digital Twins

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

### <a name="create-an-endpoint-with-dead-lettering"></a>Creación de un punto de conexión con colas de mensajes fallidos

Cuando un punto de conexión no puede entregar un evento en un período de tiempo determinado o después de haber intentado entregarlo un número determinado de veces, podrá enviar el evento sin entregar a una cuenta de almacenamiento. Este proceso se conoce como **colas de mensajes fallidos**.

Los puntos de conexión con colas de mensajes fallidos habilitadas se pueden configurar con la [CLI](how-to-use-cli.md) de Azure Digital Twins o las [API de plano de control](how-to-use-apis-sdks.md#overview-control-plane-apis).

Para obtener más información acerca de la opción para poner en cola los mensajes fallidos, consulte [*Conceptos: rutas de eventos*](concepts-route-events.md#dead-letter-events). Para obtener instrucciones sobre cómo configurar un punto de conexión con mensajes fallidos, continúe con el resto de esta sección.

#### <a name="set-up-storage-resources"></a>Configuración de recursos de almacenamiento

Antes de establecer la ubicación de mensajes fallidos, debe tener una [cuenta de almacenamiento](../storage/common/storage-account-create.md?tabs=azure-portal) con un [contenedor](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) configurado en la cuenta de Azure. 

Cuando cree el punto de conexión más tarde, proporcionará el URI de este contenedor. La ubicación de la cola de mensajes fallidos se proporcionará al punto de conexión como un URI de contenedor con un [token de SAS](../storage/common/storage-sas-overview.md). Ese token necesita permiso `write` para el contenedor de destino dentro de la cuenta de almacenamiento. El **URI de SAS de cola de mensajes fallidos** totalmente estructurado tendrá el formato `https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>`.

Siga los pasos que se indican a continuación para configurar estos recursos de almacenamiento en su cuenta de Azure, con el fin de preparar la configuración de la conexión del punto de conexión en la siguiente sección.

1. Siga los pasos descritos en [*Creación de una cuenta de Storage*](../storage/common/storage-account-create.md?tabs=azure-portal) para crear una **cuenta de almacenamiento** en la suscripción de Azure. Anote el nombre de la cuenta de almacenamiento para usarlo más adelante.
2. Siga los pasos descritos en [*Creación de un contenedor de blobs de Azure*](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) para crear un **contenedor** en la nueva cuenta de almacenamiento. Anote el nombre de contenedor para usarlo más adelante.
3. A continuación, cree un **token de SAS** para la cuenta de almacenamiento que el punto de conexión pueda usar para acceder a ella. Para empezar, vaya a la cuenta de almacenamiento en [Azure Portal](https://ms.portal.azure.com/#home) (puede buscarla por su nombre en la barra de búsqueda del portal).
4. En la página de la cuenta de almacenamiento, elija el vínculo _Firma de acceso compartido_ en la barra de navegación izquierda para empezar a configurar el token de SAS.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png" alt-text="Página de la cuenta de almacenamiento en Azure Portal" lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png":::

1. En la página *Firma de acceso compartido*, en *Servicios permitidos* y *Tipos de recursos permitidos*, seleccione la configuración que quiera. Es preciso que seleccione al menos una casilla en cada categoría. En *Permisos permitidos*, elija **Escritura** (también puede seleccionar otros permisos si lo desea).
1. Establezca los valores que quiera para el resto de la configuración.
1. Cuando finalice, seleccione el botón _Generar SAS y cadena de conexión_ para generar el token de SAS. 

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png" alt-text="Página de la cuenta de almacenamiento en Azure Portal que muestra la selección de todos los valores para generar un token de SAS y resalta el botón &quot;Generar SAS y cadena de conexión&quot;" lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png"::: 

1. Se generarán varios valores de cadena de conexión y SAS en la parte inferior de la misma página, debajo de las selecciones de configuración. Desplácese hacia abajo para ver los valores y use el icono *Copiar al portapapeles* para copiar el valor del **token de SAS**. Guárdela para usarla más adelante.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/copy-sas-token.png" alt-text="Copie el token de SAS que se va a usar en el secreto de mensajes fallidos." lightbox="./media/how-to-manage-routes-apis-cli/copy-sas-token.png":::
    
#### <a name="create-the-dead-letter-endpoint"></a>Creación del punto de conexión de cola de mensajes fallidos

Para crear un punto de conexión que tenga habilitada la cola de mensajes fallidos, agregue el siguiente parámetro de cola de mensajes fallidos al comando [az dt endpoint create](/cli/azure/ext/azure-iot/dt/endpoint/create) para la [CLI de Azure Digital Twins](how-to-use-cli.md).

El valor del parámetro es el **URI de SAS de cola de mensajes fallidos** formado por el nombre de la cuenta de almacenamiento, el nombre del contenedor y el token de SAS que ha recopilado en la [sección anterior](#set-up-storage-resources). Este parámetro crea el punto de conexión con la autenticación basada en claves.

```azurecli
--deadletter-sas-uri https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>
```

Agregue este parámetro al final de los comandos de creación del punto de conexión desde la sección [*Crear el punto de conexión*](#create-the-endpoint) anterior para crear un punto de conexión del tipo deseado que tenga habilitadas la cola de mensajes fallidos.

Como alternativa, puede crear puntos de conexión de cola de mensajes fallidos mediante las [API de plano de control de Azure Digital Twins](how-to-use-apis-sdks.md#overview-control-plane-apis) en lugar de la CLI. Para ello, consulte la [documentación de DigitalTwinsEndpoint](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) para ver cómo estructurar la solicitud y agregar los parámetros de cola de mensajes fallidos.

#### <a name="create-a-dead-letter-endpoint-with-identity-based-authentication"></a>Creación de un punto de conexión de cola de mensajes fallidos con autenticación basada en identidades

También puede crear un punto de conexión de cola de mensajes fallidos que tenga autenticación basada en identidades, para usarlo con una [identidad administrada](concepts-security.md#managed-identity-for-accessing-other-resources-preview). Esta opción solo está disponible para los puntos de conexión de centro de eventos y de tipo Service Bus (no se admite para Event Grid).

Para crear este tipo de punto de conexión, use el mismo comando de la CLI anterior para [crear un punto de conexión con autenticación basada en identidades](#create-an-endpoint-with-identity-based-authentication), con un campo adicional en la carga de JSON para un elemento `deadLetterUri`.

Estos son los valores que necesitará para conectarse a los marcadores de posición del comando:
* el id. de recurso de Azure de la instancia de Azure Digital Twins
* un nombre de punto de conexión
* un tipo de punto de conexión
* el espacio de nombres del recurso de punto de conexión
* el nombre del centro de eventos o del tema de Service Bus
* detalles del **URI de SAS de cola de mensajes fallidos**: nombre de la cuenta de almacenamiento, nombre de contenedor
* la ubicación de la instancia de Azure Digital Twins

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\", \"deadLetterUri\": \"https://<storage-account-name>.blob.core.windows.net/<container-name>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

#### <a name="message-storage-schema"></a>Esquema de almacenamiento de mensajes

Una vez configurado el punto de conexión con mensajes fallidos, los mensajes fallidos que están en cola se almacenarán con el formato siguiente en la cuenta de almacenamiento:

`{container}/{endpoint-name}/{year}/{month}/{day}/{hour}/{event-ID}.json`

Los mensajes fallidos en cola coinciden con el esquema del evento original que se diseñó para entregarse al punto de conexión original.

A continuación se muestra un ejemplo de un mensaje fallido en cola de una [notificación de creación gemela](how-to-interpret-event-data.md#digital-twin-lifecycle-notifications):

```json
{
  "specversion": "1.0",
  "id": "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "<your-instance>.api.<your-region>.da.azuredigitaltwins-test.net",
  "data": {
    "$dtId": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "$etag": "W/\"xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "TwinData": "some sample",
    "$metadata": {
      "$model": "dtmi:test:deadlettermodel;1",
      "room": {
        "lastUpdateTime": "2020-10-14T01:11:49.3576659Z"
      }
    }
  },
  "subject": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "time": "2020-10-14T01:11:49.3667224Z",
  "datacontenttype": "application/json",
  "traceparent": "00-889a9094ba22b9419dd9d8b3bfe1a301-f6564945cb20e94a-01"
}
```

## <a name="create-an-event-route"></a>Crear una ruta de eventos

**Requisito previo**: Debe crear puntos de conexión como se describió anteriormente en este artículo para poder continuar con la creación de una ruta. Puede continuar con la creación de una ruta de evento una vez finalizada la configuración de los puntos de conexión.

> [!NOTE]
> Si ha implementado recientemente los puntos de conexión, compruebe que se haya completado la implementación **antes** de intentar usarlos para una nueva ruta de evento. Si se produce un error en la implementación de la ruta porque los puntos de conexión no están listos, espere unos minutos y vuelva a intentarlo.
>
> Si crea scripts de este flujo, puede que le interese dejar entre 2 y 3 minutos de tiempo de espera para que el servicio del punto de conexión termine de implementarse antes de pasar a la configuración de la ruta.

Para enviar datos de Azure Digital Twins a un punto de conexión, debe definir una **ruta de eventos**. Las rutas de eventos se usan para conectar el flujo de eventos a lo largo del sistema y a los servicios de nivel inferior.

Una definición de ruta puede contener estos elementos:
* El nombre de ruta que quiere usar.
* Nombre del punto de conexión que quiere usar.
* Filtro que define los eventos que se envían al punto de conexión. 

Si no hay ningún nombre de ruta, no se enruta ningún mensaje fuera de Azure Digital Twins. Si hay un nombre de ruta y el filtro es `true`, todos los mensajes se enrutan al punto de conexión. Si hay un nombre de ruta y se agrega un filtro diferente, los mensajes se filtrarán de acuerdo con este.

Una ruta debe permitir que se seleccionen varias notificaciones y tipos de evento. 

Las rutas de eventos se pueden crear con las [API de plano de datos **EventRoutes**](/rest/api/digital-twins/dataplane/eventroutes) de Azure Digital Twins o los [comandos **az dt route** de la CLI](/cli/azure/ext/azure-iot/dt/route). En el resto de esta sección se le guía a través del proceso de creación.

### <a name="create-routes-with-the-apis-and-c-sdk"></a>Creación de rutas con las API y el SDK de C#

Una manera de definir rutas de eventos consiste en usar las [API de plano de datos](how-to-use-apis-sdks.md#overview-data-plane-apis). Los ejemplos de esta sección usan el [SDK de .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client).

`CreateOrReplaceEventRouteAsync` es la llamada de SDK que se usa para agregar una ruta de evento. A continuación se muestra un ejemplo del uso:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="CreateEventRoute":::
    
> [!TIP]
> Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

#### <a name="event-route-sample-sdk-code"></a>Código del SDK de ejemplo de ruta de eventos

En el método de ejemplo siguiente se muestra cómo crear, enumerar y eliminar una ruta de eventos con el SDK de C#:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="FullEventRouteSample":::

### <a name="create-routes-with-the-cli"></a>Creación de rutas con la CLI

Las rutas también se pueden administrar mediante los comandos [az dt route](/cli/azure/ext/azure-iot/dt/route) de la CLI de Azure Digital Twins. 

Para obtener más información sobre el uso de la CLI y los comandos disponibles, consulte [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

## <a name="filter-events"></a>Filtrado de eventos

Sin realizar el filtrado, los puntos de conexión reciben una serie de eventos de Azure Digital Twins:
* Telemetría que desencadena [Digital Twins](concepts-twins-graph.md) mediante la API del servicio Azure Digital Twins.
* Notificaciones relativas a cambios de propiedades de gemelos, desencadenadas en los cambios de una propiedad de cualquier gemelo en la instancia de Azure Digital Twins.
* Eventos del ciclo de vida, que se activa cuando se crean o eliminan gemelos o relaciones

Puede restringir los eventos que se envían mediante la incorporación de un **filtro** para un punto de conexión a una ruta de evento.

>[!NOTE]
> Los filtros **distinguen mayúsculas de minúsculas** y deben coincidir con el caso de la carga útil. 
>
> En el caso de los filtros de telemetría, esto significa que el uso de mayúsculas debe coincidir con el de la telemetría enviada por el dispositivo, y no necesariamente con el uso de mayúsculas definido en el modelo del gemelo. 

Para agregar un filtro, puede usar una solicitud PUT a *https://{Your-azure-digital-twins-hostname}/eventRoutes/{event-route-name}?api-version=2020-10-31* con el siguiente cuerpo:

:::code language="json" source="~/digital-twins-docs-samples/api-requests/filter.json":::

Estos son los filtros de ruta admitidos. Use los detalles de la columna *Filtrar esquema de texto* para reemplazar el marcador de posición `<filter-text>` en el cuerpo de la solicitud anterior.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Pasos siguientes

Lea acerca de los diferentes tipos de mensajes de evento que se pueden recibir:
* [*Procedimiento: Interpretación de los datos de evento*](how-to-interpret-event-data.md)
