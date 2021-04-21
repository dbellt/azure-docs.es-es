---
title: Autenticación de la entrega de eventos en los controladores de eventos (Azure Event Grid).
description: En este artículo se describen diferentes formas de autenticar los controladores de eventos en Azure Event Grid.
ms.topic: conceptual
ms.date: 01/07/2021
ms.openlocfilehash: 7db258ee152e4b1c46362e74e0246b80513ca9f2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777264"
---
# <a name="authenticate-event-delivery-to-event-handlers-azure-event-grid"></a>Autenticación de la entrega de eventos en los controladores de eventos (Azure Event Grid).
En este artículo se proporciona información sobre cómo autenticar la entrega de eventos a los controladores de eventos. También muestra como proteger los puntos de conexión de webhook que se usan para recibir eventos de Event Grid mediante Azure Active Directory (Azure AD) o un secreto compartido.

## <a name="use-system-assigned-identities-for-event-delivery"></a>Usar identidades asignadas por el sistema para la entrega de eventos.
Puede habilitar una identidad administrada asignada por el sistema para un tema o dominio y usar la identidad para reenviar eventos a destinos compatibles, como colas y temas de Service Bus, centros de eventos y cuentas de almacenamiento.

He aquí los pasos: 

1. Cree un tema o un dominio con una identidad asignada por el sistema, o bien actualice un tema o dominio existente para habilitar la identidad. 
1. Agregue la identidad a un rol adecuado (por ejemplo, Remitente de los datos de Service Bus) en el destino (por ejemplo, una cola de Service Bus).
1. Al crear suscripciones de eventos, habilite el uso de la identidad para enviar eventos al destino. 

Para obtener instrucciones paso a paso detalladas, consulte [Entrega de eventos con una identidad administrada](managed-service-identity.md).


## <a name="authenticate-event-delivery-to-webhook-endpoints"></a>Autenticación de la entrega de eventos a los puntos de conexión de webhook
En las secciones siguientes se describe cómo autenticar la entrega de eventos a los puntos de conexión de webhook. Debe usar un mecanismo de protocolo de enlace de validación con independencia del método que use. Consulte [Entrega de eventos de webhook](webhook-event-delivery.md) para detalles. 


### <a name="using-azure-active-directory-azure-ad"></a>Uso de Azure Active Directory (Azure AD)
Puede proteger el punto de conexión de webhook que se usa para recibir eventos de Event Grid mediante Azure AD. Tendrá que crear una aplicación de Azure AD, crear un rol y una entidad de servicio en la aplicación que autorice a Event Grid, y configurar la suscripción de eventos para usar la aplicación de Azure AD. Obtenga información sobre cómo [configurar Azure Active Directory con Event Grid](secure-webhook-delivery.md).

### <a name="using-client-secret-as-a-query-parameter"></a>Uso del secreto de cliente como parámetro de consulta
Además, puede proteger el punto de conexión del webhook mediante la incorporación de parámetros de consulta a la dirección URL de destino del webhook, especificada como parte de la creación de una suscripción de eventos. Establezca uno de los parámetros de consulta para que sea un secreto de cliente, como un [token de acceso](https://en.wikipedia.org/wiki/Access_token) o un secreto compartido. El servicio Event Grid incluye todos los parámetros de consulta en cada solicitud de entrega de eventos al webhook. El servicio de webhook puede recuperar y validar el secreto. Si se actualiza el secreto de cliente, también debe actualizarse la suscripción de eventos. Para evitar errores de entrega durante la rotación del secreto, haga que el webhook acepte tanto los secretos antiguos como los nuevos durante un plazo limitado antes de actualizar la suscripción de eventos con el nuevo secreto. 

Como los parámetros de consulta pueden contener secretos de cliente, se controlan con más cuidado. Se almacenan como cifrados y no son accesibles para los operadores del servicio. No se registran como parte de los registros o seguimientos de servicio. Al recuperar las propiedades de la suscripción de eventos, los parámetros de la consulta de destino no se devuelven de forma predeterminada. Por ejemplo: el parámetro [--include-full-endpoint-url](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_show) se usará en la [CLI](/cli/azure) de Azure.

Para más información sobre cómo entregar eventos a webhooks, consulte [Entrega de eventos de webhook](webhook-event-delivery.md).

> [!IMPORTANT]
> Azure Event Grid solo admite puntos de conexión de webhook **HTTPS**. 

## <a name="endpoint-validation-with-cloudevents-v10"></a>Validación de puntos de conexión con CloudEvents v1.0
Si ya está familiarizado con Event Grid, es posible que conozca el protocolo de enlace de validación de punto de conexión para evitar el uso inapropiado. CloudEvents v1.0 implementa su propia [semántica de protección contra abusos](webhook-event-delivery.md) mediante el método **HTTP OPTIONS**. Para más información al respecto, consulte [Webhooks de HTTP 1.1 para la entrega de eventos: versión 1.0](https://github.com/cloudevents/spec/blob/v1.0/http-webhook.md#4-abuse-protection). Cuando usa el esquema de CloudEvents para la salida, Event Grid usa la protección contra abusos de CloudEvents v1.0 en lugar del mecanismo de eventos de validación de Event Grid. Para obtener más información, vea [Uso del esquema CloudEvents v1.0 con Event Grid](cloudevents-schema.md). 


## <a name="next-steps"></a>Pasos siguientes
Consulte [autenticación de publicación de clientes](security-authenticate-publishing-clients.md) para obtener información sobre la autenticación de clientes que publican eventos en temas o dominios. 
