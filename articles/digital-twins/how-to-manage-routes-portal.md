---
title: Administración de puntos de conexión y rutas (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar y administrar puntos de conexión y rutas de eventos para datos de Azure Digital Twins mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: a7159b6528c07a2cb2734c06d644a4db3753d48a
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790890"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-portal"></a>Administración de puntos de conexión y rutas en Azure Digital Twins (portal)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

En Azure Digital Twins, se pueden enrutar [notificaciones de eventos](concepts-event-notifications.md) a los servicios de bajada o recursos de proceso conectados. Para ello, primero se configuran los **puntos de conexión** que pueden recibir los eventos. Después, puede crear [rutas de eventos](concepts-route-events.md) que especifican qué eventos generados por Azure Digital Twins se entregan a cada punto de conexión.

Este artículo le guía por el proceso de creación de puntos de conexión y rutas mediante [Azure Portal](https://portal.azure.com).

También puede administrar los puntos de conexión y las rutas con las [API de Event Routes](/rest/api/digital-twins/dataplane/eventroutes), los [SDK](concepts-apis-sdks.md#overview-data-plane-apis) o la [CLI de Azure Digital Twins](concepts-cli.md). Se puede encontrar una versión de este artículo donde se usan estos mecanismos en lugar del portal en [Procedimiento: Administración de puntos de conexión y rutas (API y CLI)](how-to-manage-routes-apis-cli.md) .

## <a name="prerequisites"></a>Requisitos previos

* Necesitará una **cuenta de Azure**, que [se puede configurar de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Necesitará una **instancia de Azure Digital Twins** en su suscripción de Azure. Si no dispone de una instancia, puede crear una con los pasos de [Procedimiento: Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md). Tenga a mano los siguientes valores de configuración para usarlos más adelante en este artículo:
    - Nombre de instancia
    - Grupo de recursos

Estos detalles se pueden encontrar en [Azure Portal](https://portal.azure.com) después de configurar la instancia. Inicie sesión en el portal y busque el nombre de la instancia en la barra de búsqueda del portal.
 
:::image type="content" source="media/how-to-manage-routes-portal/search-field-portal.png" alt-text="Captura de pantalla de la barra de búsqueda de Azure Portal." lightbox="media/how-to-manage-routes-portal/search-field-portal.png":::

En los resultados, seleccione la instancia para ver su página Información general:

:::image type="content" source="media/how-to-manage-routes-portal/instance-details.png" alt-text="Captura de pantalla de la página Información general de una instancia de Azure Digital Twins. Se resaltan el nombre y el grupo de recursos.":::

## <a name="create-an-endpoint-for-azure-digital-twins"></a>Creación de un punto de conexión para Azure Digital Twins

Estos son los tipos de puntos de conexión admitidos que puede crear para la instancia:
* [Event Grid](../event-grid/overview.md) 
* [Event Hubs](../event-hubs/event-hubs-about.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Para obtener más información sobre los distintos tipos de puntos de conexión, vea [Elección entre los distintos servicios de mensajería de Azure](../event-grid/compare-messaging-services.md).

En esta sección se explica cómo crear uno de estos puntos de conexión en [Azure Portal](https://portal.azure.com).

[!INCLUDE [digital-twins-endpoint-resources.md](../../includes/digital-twins-endpoint-resources.md)]

### <a name="create-the-endpoint"></a>Creación del punto de conexión 

Una vez que haya creado los recursos de punto de conexión, puede usarlos para un punto de conexión de Azure Digital Twins. Para crear un punto de conexión, vaya a la página de la instancia en [Azure Portal](https://portal.azure.com) (para buscar la instancia, escriba su nombre en la barra de búsqueda del portal).

1. En el menú de la instancia, seleccione _Endpoints_ (Puntos de conexión). Luego, en la página *Endpoints* (Puntos de conexión) que le sigue, seleccione *+ Create an endpoint* (+ Crear un punto de conexión). Se abrirá la página *Crear un punto de conexión*, donde rellenará los campos en los pasos siguientes.

    :::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-grid.png" alt-text="Captura de pantalla de la creación de un punto de conexión de tipo Event Grid." lightbox="media/how-to-manage-routes-portal/create-endpoint-event-grid.png":::

1. Escriba un **Nombre** para el punto de conexión y elija el **Tipo de punto de conexión**.

1. Complete los demás detalles necesarios para el tipo de punto de conexión, incluida la suscripción y los recursos del punto de conexión descritos [antes](#prerequisite-create-endpoint-resources).
    1. Solo para los puntos de conexión de centro de eventos y Service Bus, debe seleccionar un **Tipo de autenticación**. Puede usar la autenticación basada en claves con una regla de autorización creada previamente, o bien la autenticación basada en identidades si va a utilizar el punto de conexión con una [identidad administrada](concepts-security.md#managed-identity-for-accessing-other-resources-preview) para la instancia de Azure Digital Twins. 

    :::row:::
        :::column:::
            :::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-hub-authentication.png" alt-text="Captura de pantalla de la creación de un punto de conexión de tipo de centro de eventos." lightbox="media/how-to-manage-routes-portal/create-endpoint-event-hub-authentication.png":::
        :::column-end:::
        :::column:::
        :::column-end:::
    :::row-end:::

1. Seleccione _Guardar_ para terminar de crear el punto de conexión.

>[!IMPORTANT]
> Para usar correctamente la autenticación basada en identidades para el punto de conexión, tendrá que crear una identidad administrada para la instancia mediante los pasos descritos en [Procedimiento para habilitar una identidad administrada para el enrutamiento de eventos (versión preliminar)](./how-to-enable-managed-identities-portal.md) .

Después de crear el punto de conexión, puede comprobar que se ha creado correctamente si examina el icono de notificación de la barra superior de Azure Portal: 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-notifications.png" alt-text="Captura de pantalla de la notificación para comprobar la creación del punto de conexión. El icono en forma de campana de la barra superior del portal está seleccionado y aparece una notificación en la que se indica &quot;El punto de conexión ADT-eh-endpoint se ha creado correctamente&quot;.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Si se produce un error en la creación del punto de conexión, observe el mensaje de error y vuelva a intentarlo al cabo de unos minutos.

También se puede ver el punto de conexión que se ha creado de vuelta en la página *Endpoints* (Puntos de conexión) de la instancia de Azure Digital Twins.

Ahora, Event Grid, el centro de eventos o el tema de Service Bus está disponible como punto de conexión dentro de Azure Digital Twins, con el nombre que haya elegido para el punto de conexión. Normalmente se usará ese nombre como destino de una **ruta de eventos**, que se creará [más tarde en este artículo](#create-an-event-route).

### <a name="create-an-endpoint-with-dead-lettering"></a>Creación de un punto de conexión con colas de mensajes fallidos

Cuando un punto de conexión no puede entregar un evento en un período de tiempo determinado o después de haber intentado entregarlo un número determinado de veces, podrá enviar el evento sin entregar a una cuenta de almacenamiento. Este proceso se conoce como **colas de mensajes fallidos**.

Para crear un punto de conexión con las colas de mensajes fallidos habilitadas, debe usar los [comandos de la CLI](concepts-cli.md) o las [API del plano de control](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) en lugar de Azure Portal para crear el punto de conexión.

Para obtener instrucciones sobre cómo hacerlo con estas herramientas, vea la versión para [API y CLI](how-to-manage-routes-apis-cli.md#create-an-endpoint-with-dead-lettering) de este artículo.

## <a name="create-an-event-route"></a>Crear una ruta de eventos

Para enviar datos de Azure Digital Twins a un punto de conexión, debe definir una **ruta de eventos**. Estas rutas permiten a los desarrolladores conectar el flujo de eventos a lo largo del sistema y a los servicios de nivel inferior. Obtenga más información sobre las rutas de eventos en [Conceptos: Enrutamiento de eventos de Azure Digital Twins](concepts-route-events.md).

**Requisito previo**: Debe crear puntos de conexión como se describió anteriormente en este artículo para poder continuar con la creación de una ruta. Puede continuar con la creación de una ruta de evento una vez finalizada la configuración de los puntos de conexión.

>[!NOTE]
>Si ha implementado recientemente los puntos de conexión, compruebe que se haya completado la implementación **antes** de intentar usarlos para una nueva ruta de evento. Si no puede configurar la ruta porque los puntos de conexión no están listos, espere unos minutos y vuelva a intentarlo.

### <a name="creation-steps-with-the-azure-portal"></a>Pasos de creación con Azure Portal

Una definición de ruta de eventos contiene estos elementos:
* El nombre de ruta que quiere usar.
* Nombre del punto de conexión que quiere usar.
* Filtro que define los eventos que se envían al punto de conexión.
    - Para deshabilitar la ruta de modo que no se envíe ningún evento, use un valor de filtro de `false`.
    - Para habilitar una ruta que no tenga filtrado específico, use un valor de filtro de `true`.
    - Para más información sobre cualquier otro tipo de filtro, consulte la sección [Eventos de filtro](#filter-events) a continuación.

Una ruta puede permitir que se seleccionen varias notificaciones y tipos de eventos.

Para crear una ruta de eventos, vaya a la página de detalles de la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com) (para buscar la instancia, escriba su nombre en la barra de búsqueda del portal).

En el menú de la instancia, seleccione _Event routes_ (Rutas de eventos). Luego, en la página *Event routes* (Rutas de eventos) que le sigue, seleccione *+ + Create an event route* (+ Crear una ruta de evento). 

En la página *Create an event route* (Crear una ruta de evento) que se abre, elija como mínimo:
* Un nombre para la ruta en el campo _Name_ (Nombre).
* El valor de _Endpoint_ (Punto de conexión) que quiere usar para crear la ruta. 

Para que la ruta esté habilitada, también se debe **agregar un filtro de ruta de evento** de al menos `true`. (Si se deja el valor predeterminado de `false`, se creará la ruta, pero no se le enviarán eventos). Para ello, cambie el botón de conmutación de _Advanced editor_ (Editor avanzado) para habilitarlo y escriba `true` en el cuadro *Filter* (Filtro).

:::image type="content" source="media/how-to-manage-routes-portal/create-event-route-no-filter.png" alt-text="Captura de pantalla de la creación de una ruta de evento para la instancia." lightbox="media/how-to-manage-routes-portal/create-event-route-no-filter.png":::

Cuando termine, seleccione el botón _Save_ (Guardar) para crear la ruta de eventos.

## <a name="filter-events"></a>Filtrado de eventos

Tal y como se ha descrito anteriormente, las rutas tienen un campo **Filter** (Filtro). Si el valor de filtro de la ruta es `false`, no se enviará ningún evento al punto de conexión. 

Después de habilitar el filtro mínimo de `true`, los puntos de conexión recibirán una variedad de eventos de Azure Digital Twins:
* Telemetría que desencadena [Digital Twins](concepts-twins-graph.md) mediante la API del servicio Azure Digital Twins.
* Notificaciones relativas a cambios de propiedades de gemelos, desencadenadas en los cambios de una propiedad de cualquier gemelo en la instancia de Azure Digital Twins.
* Eventos de ciclo de vida, desencadenados cuando se crean o se eliminan gemelos o relaciones.

Se pueden restringir los tipos de eventos que se envían mediante la definición de un filtro más específico.

Para agregar un filtro de eventos mientras se crea una ruta de eventos, use la sección "Agregar un filtro de ruta de eventos" de la página *Creación de una ruta de eventos*. 

Puede seleccionar entre algunas opciones básicas de filtros comunes o usar las opciones avanzadas de filtro para escribir sus propios filtros personalizados.

>[!NOTE]
> Los filtros **distinguen mayúsculas de minúsculas** y deben coincidir con las mayúsculas y minúsculas de la carga útil. 
>
> Para los filtros de telemetría, esto significa que el uso de mayúsculas debe coincidir con el de la telemetría enviada por el dispositivo, y no necesariamente con el uso de mayúsculas definido en el modelo del gemelo.

#### <a name="use-the-basic-filters"></a>Uso de los filtros básicos

Para usar los filtros básicos, expanda la opción _Event types_ (Tipos de evento) y seleccione las casillas correspondientes a los eventos que quiere enviar al punto de conexión. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-1.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro básico. Activación de las casillas de los eventos.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

El cuadro de texto de filtro se rellena automáticamente con el texto del filtro que ha seleccionado:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-2.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro básico. Se muestra el texto del filtro rellenado automáticamente después de seleccionar los eventos.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

#### <a name="use-the-advanced-filters"></a>Uso de los filtros avanzados

También puede usar la opción de filtro avanzado para escribir sus propios filtros personalizados.

Para crear una ruta de evento con opciones avanzadas de filtro, cambie el botón de conmutación de _Advanced editor_ (Editor avanzado) para habilitarlo. Después, puede escribir sus propios filtros de eventos en el cuadro *Filter* (Filtro):

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-advanced.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro avanzado.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Estos son los filtros de ruta admitidos. Los detalles de la columna *Filter Text Schema* (Esquema de texto de filtro) es el texto que se puede escribir en el cuadro de filtro.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Pasos siguientes

Lea acerca de los diferentes tipos de mensajes de evento que se pueden recibir:
* [Conceptos: notificaciones de eventos](concepts-event-notifications.md)