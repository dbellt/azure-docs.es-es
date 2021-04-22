---
title: Dominios de eventos de Azure Event Grid
description: En este artículo se describe cómo usar dominios de eventos para administrar el flujo de eventos personalizados a sus diversas organizaciones empresariales, clientes o aplicaciones.
ms.topic: conceptual
ms.date: 04/13/2021
ms.openlocfilehash: 32c06ac55f667ec9807c7952127c2cf0f0384024
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374716"
---
# <a name="understand-event-domains-for-managing-event-grid-topics"></a>Dominios de eventos para administrar temas de Event Grid

En este artículo se describe cómo usar dominios de eventos para administrar el flujo de eventos personalizados a sus diversas organizaciones empresariales, clientes o aplicaciones. Use dominios de eventos para:

* Administrar arquitecturas de eventos multiinquilino a escala.
* Administrar la autenticación y la autorización.
* Crear particiones de los temas sin tener que administrar cada uno individualmente.
* Evitar la publicación individual en cada uno de los puntos de conexión del tema.

## <a name="event-domain-overview"></a>Información general sobre dominios de eventos

Un dominio de eventos es una herramienta de administración para un gran número de temas de Event Grid relacionados con la misma aplicación. Se puede considerar un metatema que puede contener miles de temas individuales.

Los dominios de eventos le proporcionan la misma arquitectura que usan los servicios de Azure (por ejemplo, Storage e IoT Hub) para publicar sus eventos. Le permiten publicar eventos en miles de temas. Los dominios también proporcionan control de autenticación y autorización sobre cada tema para que pueda dividir los inquilinos.

## <a name="example-use-case"></a>Ejemplo de caso de uso
[!INCLUDE [event-grid-domain-example-use-case.md](../../includes/event-grid-domain-example-use-case.md)]

## <a name="access-management"></a>Administración de acceso

Con un dominio se logra un control de autenticación y autorización específico sobre cada tema por medio del control de acceso basado en rol de Azure (RBAC de Azure). Estos roles pueden usarse para restringir cada arrendatario de la aplicación únicamente a los temas a los que desea concederle acceso.

RBAC de Azure en dominios de eventos funciona del mismo modo que el [control de acceso administrado](security-authorization.md) en el resto de Event Grid y Azure. Use RBAC de Azure para crear y aplicar definiciones de roles personalizadas en dominios de eventos.

### <a name="built-in-roles"></a>Roles integrados

Event Grid tiene dos definiciones de roles integradas que facilitan el trabajo de RBAC de Azure con dominios de eventos. Estos roles son los de **colaborador de EventGrid EventSubscription (versión preliminar)** y **lector de EventGrid EventSubscription (versión preliminar)** . Estos roles se asignan a los usuarios que necesitan suscribirse a temas en el dominio de eventos. El ámbito de la asignación de roles se limita a solo el tema al que necesitan suscribirse los usuarios.

Para información sobre estos roles, consulte [Roles integrados para Event Grid](security-authorization.md#built-in-roles).

## <a name="subscribing-to-topics"></a>Suscribirse a temas

Suscribirse a eventos de un tema dentro de un dominio de eventos es lo mismo que [crear una suscripción a eventos en un tema personalizado](./custom-event-quickstart.md) o suscribirse a un evento desde un servicio de Azure.

> [!IMPORTANT]
> El tema de dominio se considera un recurso **autoadministrado** en Event Grid. Puede crear una suscripción de eventos en el ámbito del tema de dominio sin crear dicho tema. En este caso, Event Grid crea el tema de dominio en su nombre de forma automática. Por supuesto, aún puede optar por crear el tema de dominio manualmente. Este comportamiento le ayuda a tener que preocuparse por un recurso menos al tratar con un gran número de temas de dominio. Cuando se elimina la última suscripción a un tema de dominio, dicho tema también se elimina, sin tener en cuenta si se creó de forma manual o automática. 

### <a name="domain-scope-subscriptions"></a>Suscripciones del ámbito de dominio

Los dominios de eventos también permiten suscripciones de ámbito de dominio. Una suscripción a eventos en un dominio de eventos recibirá todos los eventos enviados al dominio, independientemente del tema al que se envían los eventos. Las suscripciones del ámbito de dominio pueden ser útiles para la administración y la auditoría.

## <a name="publishing-to-an-event-domain"></a>Publicación en un dominio de eventos

Cuando se crea un dominio de eventos, se le asigna un punto de conexión de publicación igual que si hubiera creado un tema de Event Grid. 

Para publicar eventos en cualquier tema en un dominio de eventos, inserte los eventos en el punto de conexión del dominio como [haría para un tema personalizado](./post-to-custom-topic.md). La única diferencia es que debe especificar el tema en el que quiere que se entregue el evento.

Por ejemplo, la siguiente matriz de eventos de publicación enviará eventos con `"id": "1111"` al tema `foo`, mientras que el evento con `"id": "2222"` se enviaría al tema `bar`:

```json
[{
  "topic": "foo",
  "id": "1111",
  "eventType": "maintenanceRequested",
  "subject": "myapp/vehicles/diggers",
  "eventTime": "2018-10-30T21:03:07+00:00",
  "data": {
    "make": "Contoso",
    "model": "Small Digger"
  },
  "dataVersion": "1.0"
},
{
  "topic": "bar",
  "id": "2222",
  "eventType": "maintenanceCompleted",
  "subject": "myapp/vehicles/tractors",
  "eventTime": "2018-10-30T21:04:12+00:00",
  "data": {
    "make": "Contoso",
    "model": "Big Tractor"
  },
  "dataVersion": "1.0"
}]
```

Los dominios de eventos controlan la publicación de temas por usted. En lugar de publicar los eventos en cada tema que administra individualmente, puede publicar todos los eventos en el punto de conexión del dominio. Event Grid garantiza que cada evento se envía al tema correcto.

## <a name="limits-and-quotas"></a>Límites y cuotas
Estos son los límites y cuotas relacionados con los dominios de eventos:

- 100 000 temas por dominio de eventos 
- 100 dominios de eventos por suscripción de Azure 
- 500 suscripciones a eventos por tema en un dominio de eventos
- 50 suscripciones del ámbito de dominio 
- 5000 eventos por tasa de ingesta por segundo (en un dominio)

Si estos límites no son adecuados para usted, abra una incidencia de soporte técnico o envíe un correo electrónico a [askgrid@microsoft.com](mailto:askgrid@microsoft.com). 

## <a name="pricing"></a>Precios
Los dominios de eventos usan el mismo [precio por operaciones](https://azure.microsoft.com/pricing/details/event-grid/) que el resto de características de Event Grid.

Las operaciones funcionan igual en dominios de eventos que en temas personalizados. Cada entrada de un evento a un dominio de eventos es una operación, y cada intento de entrega de un evento es una operación.



## <a name="next-steps"></a>Pasos siguientes

* Para información acerca de cómo configurar dominios de eventos, crear temas, crear suscripciones a eventos y publicar eventos, consulte [Administración de dominios de eventos](./how-to-event-domains.md).
