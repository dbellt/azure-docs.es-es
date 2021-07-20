---
title: 'Inicio rápido: Enrutamiento de eventos de Azure Cache for Redis al punto de conexión web mediante Azure Portal'
description: Use Azure Event Grid para suscribirse a Azure Cache for Redis, enviar los eventos a un webhook y controlar los eventos en una aplicación web.
author: curib
ms.author: cauribeg
ms.date: 1/5/2021
ms.topic: quickstart
ms.service: cache
ms.custom:
- mode-portal
ms.openlocfilehash: b53eb606fc12cf77eeaafcccf7c7286c73d46c64
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112283040"
---
# <a name="quickstart-route-azure-cache-for-redis-events-to-web-endpoint-with-the-azure-portal"></a>Inicio rápido: Enrutamiento de eventos de Azure Cache for Redis al punto de conexión web mediante Azure Portal

Azure Event Grid es un servicio de eventos para la nube. En esta guía de inicio rápido, usará Azure Portal para crear una instancia de Azure Cache for Redis, suscribirse a eventos para esa instancia, desencadenar un evento y ver los resultados. Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. Sin embargo, para simplificar la guía de inicio rápido, enviará eventos a una aplicación web que recopilará y mostrará los mensajes.

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

Cuando haya terminado, verá que los datos del evento se han enviado a la aplicación web.

:::image type="content" source="media/cache-event-grid-portal/event-grid-scaling.png" alt-text="Escalado del visor de Azure Event Grid en formato JSON.":::

## <a name="create-an-azure-cache-for-redis-cache-instance"></a>Creación de una instancia de Azure Cache for Redis

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Antes de suscribirse a los eventos para la instancia de caché, vamos a crear el punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, implementará una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de evento. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. Seleccione **Implementar en Azure** en el archivo Léame de GitHub para implementar la solución en su suscripción.

:::image type="content" source="media/cache-event-grid-portal/deploy-to-azure.png" alt-text="Botón Implementar en Azure.":::

1. En la página **Implementación personalizada**, siga estos pasos:
    1. En **Grupo de recursos**, seleccione el grupo que usó para crear la instancia de caché. Cuando acabe el tutorial elimine el grupo de recursos.  
    2. En **Nombre del sitio**, escriba el nombre de la aplicación web.
    3. En **Nombre del plan de hospedaje**, escriba el nombre del plan de App Service que se va a usar para hospedar la aplicación web.
    4. Seleccione la casilla **Acepto los términos y condiciones indicados anteriormente**.
    5. Seleccione **Comprar**.

    | Valor      | Valor sugerido  | Descripción |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que va a crear esta aplicación web. |
    | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. |
    | **Nombre del sitio** | Escriba un nombre para la aplicación web. | Este valor no puede estar en blanco. |
    | **Nombre del plan de hospedaje** | Especifique un nombre para el plan de App Service que se va usar para hospedar la aplicación web. | Este valor no puede estar en blanco. |

1. Seleccione Alertas (el icono de la campana) en el portal y, después, seleccione **Ir al grupo de recursos**.

    :::image type="content" source="media/cache-event-grid-portal/deployment-notification.png" alt-text="Notificación de implementación de Azure Portal.":::

1. En la página **Grupo de recursos**, en la lista de recursos, seleccione la aplicación web que ha creado. En esta lista, también verá el plan de App Service y la instancia de caché.

1. En la página **App Service** de la aplicación web, seleccione la dirección URL para ir al sitio web. La dirección URL debe tener este formato: `https://<your-site-name>.azurewebsites.net`.

1. Confirme que ve el sitio, pero que aún no se han publicado eventos en él.

    :::image type="content" source="media/cache-event-grid-portal/blank-event-grid-viewer.png" alt-text="El sitio del visor de Event Grid vacío.":::

[!INCLUDE [event-grid-register-provider-portal.md](../../includes/event-grid-register-provider-portal.md)]

## <a name="subscribe-to-the-azure-cache-for-redis-instance"></a>Suscripción a la instancia de Azure Cache for Redis

En este paso, se va a suscribir a un tema para indicar a Event Grid los eventos a los que desea hacer seguimiento y el lugar al que se enviarán.

1. En el portal, navegue a la instancia de caché que creó anteriormente.
1. En la página **Azure Cache for Redis**, seleccione **Eventos** en el menú de la izquierda.
1. Seleccione un **webhook**. Al enviar los eventos a la aplicación de visor, va a usar un webhook como punto de conexión.

     :::image type="content" source="media/cache-event-grid-portal/event-grid-web-hook.png" alt-text="Página de eventos de Azure Portal.":::

1. En la página **Crear suscripción de eventos**, especifique lo siguiente:

    | Valor      | Valor sugerido  | Descripción |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre** | Escriba un nombre para la suscripción a eventos. | El valor debe tener entre 3 y 64 caracteres de longitud. Solo puede contener letras, números y guiones. |
    | **Tipos de eventos** | Despliegue y seleccione los tipos de evento que desea insertar en el destino. En esta guía de inicio rápido, vamos a escalar la instancia de caché. | Las opciones disponibles son la revisión, el escalado, la importación y la exportación. |
    | **Tipo de punto de conexión** | Seleccione un **webhook**. | Controlador de eventos para recibir los eventos. |
    | **Punto de conexión** | Seleccione **Seleccionar un punto de conexión** y escriba la dirección URL de la aplicación web y agregue `api/updates` a la dirección URL de la página principal (por ejemplo: `https://cache.azurewebsites.net/api/updates`) y, a continuación, seleccione **Confirmar selección**. | Esta es la dirección URL de la aplicación web que creó anteriormente. |

1. Ahora, en la página **Crear suscripción de eventos**, seleccione **Crear**.

1. Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

    :::image type="content" source="media/cache-event-grid-portal/subscription-event.png" alt-text="El visor de Azure Event Grid.":::

## <a name="send-an-event-to-your-endpoint"></a>Envío de un evento al punto de conexión

Ahora, vamos a desencadenar un evento para ver cómo Event Grid distribuye el mensaje al punto de conexión. Vamos a escalar la instancia de Azure Cache for Redis.

1. En Azure Portal, vaya a la instancia de Azure Cache for Redis y seleccione **Escalar** en el menú de la izquierda.

1. Seleccione el plan de tarifa deseado en la página **Escalar** y seleccione **Seleccionar**.

    Puede escalar a un plan de tarifa diferente con las siguientes restricciones:

    * No se puede escalar desde un plan de tarifa superior a un plan de tarifa inferior.
      * No puede cambiar de una memoria caché **Premium** a una memoria caché **Estándar** o **Básica**.
      * No puede cambiar de una memoria caché **Estándar** a una memoria caché **Básica**.
    * Puede cambiar de una memoria caché **Básica** a una memoria caché **Estándar**, pero no puede cambiar el tamaño al mismo tiempo. Si necesita un tamaño distinto, puede realizar una operación de escalado posterior hasta el tamaño deseado.
    * No puede escalar de una memoria caché **Básica** directamente a una memoria caché **Premium**. En primer lugar, escale desde **Básica** a **Estándar** en una operación de escalado y, después, desde **Estándar** a **Premium** en una operación de escalado posterior.
    * No puede escalar desde un tamaño mayor hasta el tamaño **C0 (250 MB)** .

    Durante la operación de escalado de la memoria caché al nuevo plan de tarifa, se muestra el estado **Escalando** a la izquierda de **Azure Cache for Redis** . Cuando se completa el escalado, el estado cambia de **Escalado** a **En ejecución**.

1. Ha desencadenado el evento y Event Grid ha enviado el mensaje al punto de conexión que configuró al realizar la suscripción. El mensaje está en formato JSON y contiene una matriz con uno o más eventos. En el ejemplo siguiente, el mensaje JSON contiene una matriz con un evento. Vea la aplicación web y observe que se ha recibido un evento **ScalingCompleted**.

    :::image type="content" source="media/cache-event-grid-portal/event-grid-scaling.png" alt-text="Escalado del visor de Azure Event Grid en formato JSON.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si piensa seguir trabajando con este evento, no elimine los recursos creados en este artículo. De lo contrario, elimine los recursos que ha creado en este inicio rápido.

Seleccione el grupo de recursos y seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear suscripciones a temas personalizados y eventos, aprenda más acerca de cómo puede ayudarle Event Grid:

* [Reacción a eventos de Azure Cache for Redis](cache-event-grid.md)
* [Una introducción a Azure Event Grid](../event-grid/overview.md)
