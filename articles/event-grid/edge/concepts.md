---
title: Conceptos de Azure Event Grid IoT Edge | Microsoft Docs
description: Conceptos de Event Grid en IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 05/10/2021
ms.subservice: iot-edge
ms.topic: article
ms.openlocfilehash: ed3a0b1f27919e322a76b6f9ce1b04617538f9aa
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370708"
---
# <a name="event-grid-concepts"></a>Conceptos de Event Grid

En este artículo se describen los principales conceptos de Azure Event Grid.

## <a name="events"></a>Eventos

Una evento es la cantidad mínima de información que describe completamente algo que se ha producido en el sistema. Todos los eventos tienen información común, como: origen del evento, hora en que el evento ha tenido lugar e identificador único. Cada evento tiene además información específica que solo es relevante para el tipo de evento concreto. La compatibilidad de un evento con un tamaño de hasta 1 MB se encuentra actualmente en versión preliminar.

Para obtener las propiedades que se incluyen en un evento, consulte [Esquema de eventos de Azure Event Grid](event-schemas.md).

## <a name="publishers"></a>Publicadores

Un publicador es el usuario o la organización que decide enviar eventos a Event Grid. Puede publicar eventos desde su propia aplicación.

## <a name="event-sources"></a>Orígenes de eventos

Un origen de evento es donde se produce el evento. Cada origen de evento está relacionado con uno o más tipos de eventos. Por ejemplo, Azure Storage es el origen de evento de los eventos creados por blob. La aplicación es el origen de evento de los eventos personalizados que defina. Los orígenes de evento son responsables de enviar eventos a Event Grid.

## <a name="topics"></a>Temas

En el tema de Event Grid se proporciona un punto de conexión al que el origen envía los eventos. El publicador crea el tema de Event Grid y decide si un origen de evento necesita un tema o más de un tema. Un tema se usa para una colección de eventos relacionados. Para responder a determinados tipos de eventos, los suscriptores deciden los temas a los que se suscriben.

Cuando diseñe la aplicación, tiene la flexibilidad para decidir cuántos temas se van a crear. Para soluciones grandes, cree un tema personalizado para cada categoría de eventos relacionados. Por ejemplo, considere una aplicación que envía eventos relacionados con la modificación de las cuentas de usuario y el procesamiento de pedidos. Es poco probable que algún controlador de eventos quiera ambas categorías de eventos. Cree dos temas personalizados y deje que los controladores de eventos se suscriban a uno que les interese. Para soluciones pequeñas, puede que prefiera enviar todos los eventos a un solo tema. Los suscriptores de eventos se pueden filtrar por los tipos de evento que desean.

Consulte la [documentación de la API REST](api.md) sobre cómo administrar los temas en Event Grid.

## <a name="event-subscriptions"></a>Suscripciones a eventos

Las suscripciones le indican a Event Grid qué eventos de un tema les interesa recibir. Al crear la suscripción, proporciona un punto de conexión para controlar el evento. Puede filtrar los eventos que se envían al punto de conexión. 

Consulte la [documentación de la API REST](api.md) sobre cómo administrar las suscripciones en Event Grid.

## <a name="event-handlers"></a>Controladores de eventos

Desde la perspectiva de Event Grid, un controlador de eventos es el lugar al que se envía el evento. El controlador realiza acciones adicionales para procesar el evento. Event Grid admite varios tipos de controladores. Puede usar un servicio de Azure admitido o su propio webhook como controlador. Según el tipo de controlador, Event Grid sigue distintos procedimientos para garantizar la entrega del evento. Si el controlador de eventos de destino es un webhook HTTP, el evento se vuelve a intentar hasta que el controlador devuelve un código de estado de `200 – OK`. Para el centro de Edge se considera que el evento se realizó correctamente si se entrega sin ninguna excepción.

## <a name="security"></a>Seguridad

Event Grid proporciona seguridad para suscribirse a temas y publicarlos. Para más información, vea [Event Grid security and authentication](security-authentication.md) (Seguridad y autenticación de Event Grid).

## <a name="event-delivery"></a>Entrega de eventos

Si Event Grid no puede confirmar que un evento se ha recibido en el punto de conexión del suscriptor, vuelve a entregarlo. Para más información, vea [Entrega y reintento de entrega de mensajes de Event Grid](delivery-retry.md).

## <a name="batching"></a>Lotes

Cuando se usa un tema personalizado, los eventos siempre se deben publicar en una matriz. En escenarios de bajo rendimiento, la matriz solo tendrá un valor. Para los casos de uso de gran volumen, se recomienda procesar varios eventos juntos por publicación para lograr una mayor eficacia. Los lotes pueden tener hasta 1 MB. Cada uno de los eventos no debe superar el 1 MB (versión preliminar).