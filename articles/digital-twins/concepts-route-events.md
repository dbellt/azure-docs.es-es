---
title: Rutas de eventos
titleSuffix: Azure Digital Twins
description: Descripción de cómo enrutar eventos en Azure Digital Twins y a otros servicios de Azure.
author: baanders
ms.author: baanders
ms.date: 10/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 540631db87807312f96c96ddd5b8144d4e22ff91
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110095579"
---
# <a name="route-events-within-and-outside-of-azure-digital-twins"></a>Enrutar eventos dentro y fuera de Azure Digital Twins

Azure Digital Twins usa **rutas de eventos** para enviar datos a los consumidores fuera del servicio. 

Hay dos casos principales para el envío de datos de Azure Digital Twins:
* El envío de datos desde un gemelo del grafo de Azure Digital Twins a otro. Por ejemplo, cuando cambia una propiedad de un gemelo digital, puede que desee notificar y actualizar otro gemelo digital en consecuencia.
* El envío de datos a servicios de datos descendentes, para almacenamiento o procesamiento adicional (también conocido como *salida de datos*). Por ejemplo,
  - Un hospital puede querer enviar datos de eventos de Azure Digital Twins a [Time Series Insights (TSI)](../time-series-insights/overview-what-is-tsi.md), para registrar los datos de series temporales de eventos relacionados con el lavado de manos para análisis masivo.
  - Una empresa que ya use [Azure Maps](../azure-maps/about-azure-maps.md) podría querer usar Azure Digital Twins para mejorar su solución. Puede habilitar con rapidez un mapa de Azure después de configurar Azure Digital Twins, incorporar las entidades del mapa de Azure a Azure Digital Twins como [gemelos digitales](concepts-twins-graph.md) en el grafo de gemelos, o ejecutar consultas eficaces que aprovechen los datos de Azure Maps y Azure Digital Twins conjuntamente.

Las rutas de eventos se utilizan para ambos escenarios.

## <a name="about-event-routes"></a>Acerca de las rutas de eventos

Una ruta de eventos permite enviar datos de eventos desde gemelos digitales en Azure Digital Twins a puntos de conexión definidos por el usuario en las suscripciones. Actualmente se admiten tres servicios de Azure para puntos de conexión: [Centro de eventos](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md) y [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md). Cada uno de estos servicios de Azure puede estar conectado a otros servicios y actuar como intermediario, enviando datos a destinos finales, como TSI o Azure Maps, para cualquier procesamiento que necesite.

En el diagrama siguiente se ilustra el flujo de datos de eventos a través de una solución de IoT más grande con un aspecto de Azure Digital Twins:

:::image type="content" source="media/concepts-route-events/routing-workflow.png" alt-text="Azure Digital Twins que enruta datos a través de puntos de conexión a varios servicios descendentes" border="false":::

Los destinos descendentes típicos para las rutas de eventos son recursos como TSI, Azure Maps, almacenamiento y soluciones de análisis.

### <a name="event-routes-for-internal-digital-twin-events"></a>Rutas de eventos para eventos de gemelos digitales internos

Las rutas de eventos también se usan para controlar los eventos dentro del grafo de gemelos y enviar datos de un gemelo digital a otro. Esto se hace mediante la conexión de las rutas de eventos a través de Event Grid para calcular los recursos, como [Azure Functions](../azure-functions/functions-overview.md). A continuación, estas funciones definen el modo en que los gemelos deben recibir y responder a los eventos. 

Cuando un recurso de proceso quiere modificar el grafo de gemelos en función de un evento recibido a través de la ruta de eventos, es útil que sepa con anterioridad qué gemelo quiere modificar. 

Como alternativa, el mensaje de evento también contiene el id. de gemelo de origen que envió el mensaje, por lo que el recurso de proceso puede usar consultas o atravesar relaciones para encontrar un gemelo de destino para la operación deseada. 

El recurso de proceso también debe establecer los permisos de acceso y seguridad de forma independiente.

Para recorrer el proceso de configuración de una función de Azure para procesar eventos de gemelos digitales, consulte [Procedimiento: Configuración de una función de Azure para procesar datos](how-to-create-azure-function.md).

## <a name="create-an-endpoint"></a>Crear un punto de conexión

Para definir una ruta de eventos, los desarrolladores deben definir primero los puntos de conexión. Un **punto de conexión** es un destino fuera de Azure Digital Twins que admite una conexión de ruta. Destinos admitidos:
* Temas personalizados de Event Grid
* Centro de eventos
* Azure Service Bus

Para crear un punto de conexión, puede usar las [API REST de Azure Digital Twins, comandos de la CLI](how-to-manage-routes-apis-cli.md#create-an-endpoint-for-azure-digital-twins) o [Azure Portal](how-to-manage-routes-portal.md#create-an-endpoint-for-azure-digital-twins).

Al definir un punto de conexión, debe proporcionar:
* El nombre del punto de conexión
* El tipo de punto de conexión (Event Grid, centro de eventos o Service Bus)
* La cadena de conexión principal o la cadena de conexión secundaria 
* La ruta de acceso del tema del punto de conexión, como *your-topic.westus2.eventgrid.azure.net*

Las API del punto de conexión que están disponibles en el plano de control son:
* Crear punto de conexión
* Obtener una lista de puntos de conexión
* Obtener punto de conexión por nombre
* Eliminar punto de conexión por nombre

## <a name="create-an-event-route"></a>Crear una ruta de eventos
 
Para crear una ruta de eventos, puede usar las [API REST de Azure Digital Twins, comandos de la CLI](how-to-manage-routes-apis-cli.md#create-an-event-route) o [Azure Portal](how-to-manage-routes-portal.md#create-an-event-route).

Este es un ejemplo de cómo crear una ruta de eventos dentro de una aplicación cliente mediante una llamada al [SDK de .NET (C# )](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) `CreateOrReplaceEventRouteAsync`: 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="CreateEventRoute":::

1. En primer lugar, se crea un objeto `DigitalTwinsEventRoute` y el constructor toma el nombre de un punto de conexión. Este campo `endpointName` identifica un punto de conexión, como un centro de eventos, Event Grid o Service Bus. Estos puntos de conexión deben crearse en la suscripción y conectarse a Azure Digital Twins mediante las API de plano de control antes de realizar esta llamada de registro.

2. El objeto de ruta de eventos también tiene un campo [Filtro](how-to-manage-routes-apis-cli.md#filter-events), que se puede usar para restringir los tipos de eventos que siguen esta ruta. Un filtro de `true` habilita la ruta sin filtrado adicional (un filtro de `false` deshabilita la ruta). 

3. A continuación, este objeto de ruta de eventos se pasa a `CreateOrReplaceEventRouteAsync`, junto con un nombre para la ruta.

> [!TIP]
> Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

## <a name="dead-letter-events"></a>Eventos fallidos

Cuando un punto de conexión no puede entregar un evento en un período de tiempo determinado o después de haber intentado entregarlo un número determinado de veces, podrá enviar el evento sin entregar a una cuenta de almacenamiento. Este proceso se conoce como **colas de mensajes fallidos**. Azure Digital Twins incluirá en la cola de mensajes fallidos un evento cuando se cumpla **una de las siguientes condiciones**. 

* El evento no se entrega en el período de tiempo de vida
* El número de intentos de entrega del evento ha superado el límite

Si se cumple alguna de las condiciones, el evento se quita o pone en la cola de mensajes fallidos. De forma predeterminada, los puntos de conexión individuales **no** activan la cola de mensajes fallidos. Para habilitarla, debe especificar una cuenta de almacenamiento que incluya los eventos no entregados al crear el punto de conexión. Posteriormente, puede extraer los eventos de esta cuenta de almacenamiento para resolver las entregas.

Antes de establecer la ubicación de mensajes fallidos, debe tener una cuenta de almacenamiento con un contenedor. Tiene que proporcionar la dirección URL de este contenedor al crear el punto de conexión. La cola de mensajes fallidos se suministra como una dirección URL del contenedor con un token de SAS. Ese token solo necesita permiso `write` para el contenedor de destino dentro de la cuenta de almacenamiento. La dirección URL totalmente estructurada tendrá este formato: `https://<storageAccountname>.blob.core.windows.net/<containerName>?<SASToken>`.

Para obtener más información sobre los tokens de SAS, consulte: [Grant limited access to Azure Storage resources using shared access signatures (SAS)](../storage/common/storage-sas-overview.md) (Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido [SAS])

Para obtener información sobre cómo configurar un punto de conexión con una cola de mensajes fallidos, consulte:  [Administración de puntos de conexión y rutas en Azure Digital Twins (API y CLI)](how-to-manage-routes-apis-cli.md#create-an-endpoint-with-dead-lettering) 

### <a name="types-of-event-messages"></a>Tipos de mensajes de eventos

Los distintos tipos de eventos de IoT Hub y Azure Digital Twins generan distintos tipos de mensajes de notificación, como se describe a continuación.

[!INCLUDE [digital-twins-notifications.md](../../includes/digital-twins-notifications.md)]

## <a name="next-steps"></a>Pasos siguientes

Vea cómo configurar y administrar una ruta de eventos:
* [Procedimiento: Administración de puntos de conexión y rutas](how-to-manage-routes-apis-cli.md)

O bien, consulte cómo usar Azure Functions para enrutar eventos en Azure Digital Twins:
* [Procedimiento: Configuración de una función de Azure para procesar datos](how-to-create-azure-function.md)