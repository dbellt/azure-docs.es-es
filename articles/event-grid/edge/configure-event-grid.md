---
title: 'Configuración: Azure Event Grid IoT Edge | Microsoft Docs'
description: Configuración en Event Grid en IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 825783faa14c6775f44e5fba8c798988d0e2ef12
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370589"
---
# <a name="event-grid-configuration"></a>Configuración de Event Grid

Event Grid proporciona muchas configuraciones que se pueden modificar en función del entorno. La siguiente sección es una referencia a todas las opciones disponibles y sus valores predeterminados.

## <a name="tls-configuration"></a>Configuración de TLS

Para más información sobre la autenticación de cliente en general, consulte [Seguridad y autenticación](security-authentication.md). Puede encontrar ejemplos de su uso en [este artículo](configure-api-protocol.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
|`inbound__serverAuth__tlsPolicy`| Directiva de TLS del módulo de Event Grid. El valor predeterminado es solo HTTPS.
|`inbound__serverAuth__serverCert__source`| El origen del certificado de servidor utilizado por el módulo de Event Grid para su configuración de TLS. El valor predeterminado es IoT Edge.

## <a name="incoming-client-authentication"></a>Autenticación de cliente entrante

Para más información sobre la autenticación de cliente en general, consulte [Seguridad y autenticación](security-authentication.md). Se pueden encontrar ejemplos [en este artículo](configure-client-auth.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
|`inbound__clientAuth__clientCert__enabled`| Para activar/desactivar la autenticación de cliente basada en certificados. El valor predeterminado es true.
|`inbound__clientAuth__clientCert__source`| Origen para validar los certificados de cliente. El valor predeterminado es IoT Edge.
|`inbound__clientAuth__clientCert__allowUnknownCA`| Directiva para permitir un certificado de cliente autofirmado. El valor predeterminado es true.
|`inbound__clientAuth__sasKeys__enabled`| Para activar/desactivar la autenticación de cliente basada en clave SAS. Está desactivado de manera predeterminada.
|`inbound__clientAuth__sasKeys__key1`| Uno de los valores para validar las solicitudes entrantes.
|`inbound__clientAuth__sasKeys__key2`| Segundo valor opcional para validar las solicitudes entrantes.

## <a name="outgoing-client-authentication"></a>Autenticación de cliente saliente
Para más información sobre la autenticación de cliente en general, consulte [Seguridad y autenticación](security-authentication.md). Se pueden encontrar ejemplos [en este artículo](configure-identity-auth.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
|`outbound__clientAuth__clientCert__enabled`| Para activar o desactivar la asociación de un certificado de identidad para las solicitudes salientes. El valor predeterminado es true.
|`outbound__clientAuth__clientCert__source`| Origen para recuperar el certificado saliente del módulo de Event Grid. El valor predeterminado es IoT Edge.

## <a name="webhook-event-handlers"></a>Controladores de eventos de webhook

Para más información sobre la autenticación de cliente en general, consulte [Seguridad y autenticación](security-authentication.md). Se pueden encontrar ejemplos [en este artículo](configure-webhook-subscriber-auth.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
|`outbound__webhook__httpsOnly`| Directiva para controlar si solo se permitirán suscriptores HTTPS. El valor predeterminado es true (solo HTTPS).
|`outbound__webhook__skipServerCertValidation`| Marca para controlar si se debe validar el certificado del suscriptor. El valor predeterminado es true.
|`outbound__webhook__allowUnknownCA`| Directiva para controlar si un certificado autofirmado puede presentarse por un suscriptor. El valor predeterminado es true. 

## <a name="delivery-and-retry"></a>Entrega y nuevo intento

Para obtener información sobre esta característica en general, vea [Entrega y reintento](delivery-retry.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
| `broker__defaultMaxDeliveryAttempts` | Número máximo de intentos de entrega de un evento. El valor predeterminado es 30.
| `broker__defaultEventTimeToLiveInSeconds` | Período de vida (TTL) del evento en segundos transcurrido el cual se descartará un evento si no se entrega. El valor predeterminado es **7200** segundos.

## <a name="output-batching"></a>Procesamiento por lotes de salida

Para obtener información sobre esta característica en general, vea [Entrega y procesamiento por lotes de salida](delivery-output-batching.md).

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
| `api__deliveryPolicyLimits__maxBatchSizeInBytes` | Valor máximo permitido para el control `ApproxBatchSizeInBytes`. El valor predeterminado es `1_058_576`.
| `api__deliveryPolicyLimits__maxEventsPerBatch` | Valor máximo permitido para el control `MaxEventsPerBatch`. El valor predeterminado es `50`.
| `broker__defaultMaxBatchSizeInBytes` | Tamaño máximo de la solicitud de entrega cuando solo se especifica `MaxEventsPerBatch`. El valor predeterminado es `1_058_576`.
| `broker__defaultMaxEventsPerBatch` | Número máximo de eventos para agregar a un lote cuando solo se especifica `MaxBatchSizeInBytes`. El valor predeterminado es `10`.

## <a name="metrics"></a>Métricas

Para más información sobre el uso de métricas con Event Grid en IoT Edge, consulte [Supervisión de temas y suscripciones](monitor-topics-subscriptions.md)

| Nombre de propiedad | Descripción |
| ---------------- | ------------ |
| `metrics__reporterType` | Tipo de informador del punto de conexión de métricas. El valor predeterminado es `none` y este deshabilita las métricas. Si se establece en `prometheus`, se habilitan las métricas en el formato de exposición Prometheus.