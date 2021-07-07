---
title: Definiciones de métricas para Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: En este documento se incluyen las definiciones de las métricas disponibles en Azure Portal.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: f144b3710e0af570289be1981214687c176cbc71
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113111472"
---
# <a name="metrics-overview"></a>Información general de las métricas

Actualmente, Azure Communication Services ofrece métricas de chat y SMS. El [Explorador de métricas de Azure](../../azure-monitor/essentials/metrics-getting-started.md) puede usarse para trazar gráficos propios, investigar anomalías en los valores de las métricas y comprender el tráfico de las API mediante el uso de los datos de métricas que emiten las solicitudes de Chat y SMS.

## <a name="where-to-find-metrics"></a>Dónde se encuentran las métricas

Los servicios Chat y SMS de Azure Communication Services emiten métricas sobre las solicitudes de API. Estas métricas se pueden encontrar en la hoja Métricas del recurso de Communication Services. También puede crear paneles permanentes mediante la hoja de libros del recurso de Communication Services.

## <a name="metric-definitions"></a>Definiciones de métricas

Hay dos tipos de solicitudes que se representan en las métricas de Communication Services: **Solicitudes de API de Chat** y **solicitudes de API de SMS**.

Las métricas de las solicitudes de API de SMS y de Chat contienen tres dimensiones que sirven para filtrar los datos de las métricas. Estas dimensiones se pueden agregar juntas mediante el tipo de agregación `Count`, y admiten todas las series temporales de agregación estándares de Azure, como `Sum`, `Average`, `Min` y `Max`.

Más información sobre los tipos de agregación admitidos y las agregaciones de series temporales en [Características avanzadas del Explorador de métricas de Azure](../../azure-monitor/essentials/metrics-charts.md#aggregation).

- **Operation**: todas las operaciones o rutas a las que se puede llamar en la puerta de enlace de chat de ACS.
- **Status Code**: respuesta del código de estado enviada después de la solicitud.
- **StatusSubClass**: la serie de código de estado enviada después de la respuesta. 


### <a name="chat-api-request-metric-operations"></a>Operaciones de las métricas de solicitudes de API de Chat

Las siguientes operaciones están disponibles en las métricas de solicitudes de API de Chat:

| Operación/ruta    | Descripción                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| GetChatMessage       | Obtiene un mensaje por su identificador. |
| ListChatMessages     | Obtiene una lista de mensajes de chat de una conversación. |
| SendChatMessage      | Envía un mensaje de chat a una conversación. |
| UpdateChatMessage    | Actualiza un mensaje de chat. |
| DeleteChatMessage    | Elimina un mensaje de chat. |
| GetChatThread        | Obtiene una conversación de chat. |
| ListChatThreads      | Obtiene la lista de conversaciones de chat de un usuario. |
| UpdateChatThread     | Actualiza las propiedades de la conversación de chat. |
| CreateChatThread     | Crea una conversación de chat. |
| DeleteChatThread     | Elimina una conversación. |
| GetReadReceipts      | Obtiene confirmaciones de lectura de una conversación. |
| SendReadReceipt      | Envía un evento de confirmación de lectura a una conversación en nombre de un usuario. |
| SendTypingIndicator           | Publica un evento de escritura a una conversación en nombre de un usuario. |
| ListChatThreadParticipants    | Obtiene los miembros de una conversación. |
| AddChatThreadParticipants     | Agrega miembros a una conversación. Si ya existen, no se produce ningún cambio. |
| RemoveChatThreadParticipant   | Elimina un miembro de una conversación. |

:::image type="content" source="./media/chat-metric.png" alt-text="Métrica de solicitudes de API de Chat.":::

Si se realiza una solicitud a una operación que no se reconoce, recibirá una respuesta con valor de "ruta incorrecta".

### <a name="sms-api-requests"></a>Solicitudes de API de SMS

Las siguientes operaciones están disponibles en las métricas de solicitud de API de SMS:

| Operación/ruta    | Descripción                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| SMSMessageSent       | Envía un mensaje SMS. |
| SMSDeliveryReportsReceived     | Obtiene informes de entrega de SMS. |
| SMSMessagesReceived      | Obtiene mensajes SMS. |


:::image type="content" source="./media/sms-metric.png" alt-text="Métrica de solicitud de API de SMS.":::

### <a name="authentication-api-requests"></a>Solicitudes de API de autenticación

Las siguientes operaciones están disponibles en las métricas de solicitudes de API de autenticación:

| Operación/ruta    | Descripción                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| CreateIdentity       | Crea una identidad que representa a un solo usuario. |
| DeleteIdentity       | Elimina una identidad. |
| CreateToken          | Crea un token de acceso. |
| RevokeToken          | Revoca todos los tokens de acceso creados para una identidad antes de un tiempo dado. |

:::image type="content" source="./media/acs-auth-metrics.png" alt-text="Métrica de solicitud de autenticación.":::

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [Métricas de la plataforma de datos](../../azure-monitor/essentials/data-platform-metrics.md).
