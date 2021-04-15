---
title: Entrega de eventos mediante el servicio Private Link
description: En este artículo se describe cómo solucionar la limitación de no poder enviar eventos mediante el servicio Private Link.
ms.topic: how-to
ms.date: 02/12/2021
ms.openlocfilehash: 75ae8ff2c341048c39156e43da3615788e9d0f0a
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107309438"
---
# <a name="deliver-events-using-private-link-service"></a>Entrega de eventos mediante el servicio Private Link
Actualmente, no es posible enviar eventos mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md). Es decir, no hay compatibilidad si tiene requisitos de aislamiento de red estrictos por los que el tráfico de eventos entregados no debe abandonar el espacio de la IP privada. 

## <a name="use-managed-identity"></a>Uso de identidad administrada
Sin embargo, si sus requisitos exigen una forma segura de enviar eventos mediante un canal cifrado y una identidad conocida del remitente (en este caso, Event Grid) por medio de un espacio de IP pública, puede enviar eventos a los servicios Event Hubs, Service Bus o Azure Storage mediante un tema o un dominio personalizados de Azure Event Grid con una identidad administrada por el sistema. Para más información sobre cómo entregar eventos mediante identidad administrada, consulte [Entrega de eventos con una identidad administrada](managed-service-identity.md). 

Después, puede usar un vínculo privado configurado en Azure Functions o en el webhook implementado en la red virtual para extraer eventos. Vea el siguiente ejemplo: [Conexión a puntos de conexión privados con Azure Functions](/samples/azure-samples/azure-functions-private-endpoints/connect-to-private-endpoints-with-azure-functions/).


:::image type="content" source="./media/consume-private-endpoints/deliver-private-link-service.svg" alt-text="Entrega a través del servicio Private Link":::


En esta configuración, el tráfico pasa por la dirección IP pública/Internet desde Event Grid hasta Event Hubs, Service Bus o Azure Storage, pero el canal se puede cifrar y se usa una identidad administrada de Event Grid. Si configura Azure Functions o el webhook implementado en la red virtual para usar una instancia de Event Hubs, Service Bus o Azure Storage a través de un vínculo privado, esa sección del tráfico permanecerá en Azure de forma evidente.

## <a name="deliver-events-to-event-hubs-using-managed-identity"></a>Entrega de eventos a Event Hubs mediante identidad administrada
Para enviar eventos a Event Hubs en el espacio de nombres de Event Hubs mediante identidad administrada, siga estos pasos:

1. Habilite la identidad asignada por el sistema: [temas del sistema](enable-identity-system-topics.md), [temas y dominios personalizados](enable-identity-custom-topics-domains.md).  
1. [Agregue la identidad al rol **Remitente de datos de Azure Event Hubs** en el espacio de nombres de Event Hubs](../event-hubs/authenticate-managed-identity.md#to-assign-azure-roles-using-the-azure-portal).
1. [Habilite el valor **Allow trusted Microsoft services to bypass this firewall** (Permitir que servicios de Microsoft de confianza omitan este firewall) en el espacio de nombres de Event Hubs](../event-hubs/event-hubs-service-endpoints.md#trusted-microsoft-services). 
1. [Configure la suscripción de eventos](managed-service-identity.md#create-event-subscriptions-that-use-an-identity) que emplea un centro de eventos como punto de conexión para usar la identidad asignada por el sistema.

## <a name="deliver-events-to-service-bus-using-managed-identity"></a>Entrega de eventos a Service Bus mediante identidad administrada
Para enviar eventos a Service Bus colas o temas del espacio de nombres de Service Bus mediante identidad administrada, siga estos pasos:

1. Habilite la identidad asignada por el sistema: [temas del sistema](enable-identity-system-topics.md), [temas y dominios personalizados](enable-identity-custom-topics-domains.md). 
1. [Agregue la identidad al rol **Remitente de datos de Azure Service Bus**](../service-bus-messaging/service-bus-managed-service-identity.md#azure-built-in-roles-for-azure-service-bus) en el espacio de nombres de Service Bus.
1. [Habilite el valor **Allow trusted Microsoft services to bypass this firewall** (Permitir que servicios de Microsoft de confianza omitan este firewall) en el espacio de nombres de Service Bus](../service-bus-messaging/service-bus-service-endpoints.md#trusted-microsoft-services). 
1. [Configure la suscripción de eventos](managed-service-identity.md) que emplea una cola o tema de Service Bus como punto de conexión para usar la identidad asignada por el sistema.

## <a name="deliver-events-to-storage"></a>Entrega de eventos a Storage 
Para enviar eventos a las colas de almacenamiento mediante identidad administrada, siga estos pasos:

1. Habilite la identidad asignada por el sistema: [temas del sistema](enable-identity-system-topics.md), [temas y dominios personalizados](enable-identity-custom-topics-domains.md). 
1. [Agregue la identidad al rol **Remitente de mensajes de datos de Queue Storage**](../storage/common/storage-auth-aad-rbac-portal.md) en la cola de Azure Storage.
1. [Configure la suscripción de eventos](managed-service-identity.md#create-event-subscriptions-that-use-an-identity) que emplea una cola o tema de Service Bus como punto de conexión para usar la identidad asignada por el sistema.


## <a name="next-steps"></a>Pasos siguientes
Para más información sobre la entrega de eventos mediante identidad administrada, consulte [Entrega de eventos con una identidad administrada](managed-service-identity.md). 
