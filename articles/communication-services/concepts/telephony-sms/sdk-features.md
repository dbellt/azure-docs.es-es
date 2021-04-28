---
title: Introducción al SDK de SMS de Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Proporciona información general sobre el SDK de SMS y sus ofertas.
author: mikben
manager: jken
services: azure-communication-services
ms.author: prakulka
ms.date: 03/26/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 0fe6f43811c3a7e990646a75e8db11d93fa01f3c
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108016462"
---
# <a name="sms-sdk-overview"></a>Introducción al SDK de SMS

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Los SDK de SMS de Azure Communication Services se pueden usar para agregar mensajería SMS a las aplicaciones.

## <a name="sms-sdk-capabilities"></a>Funcionalidades del SDK de SMS

En la lista siguiente se presenta el conjunto de características que están disponibles actualmente en nuestros SDK.

| Grupo de características | Capacidad                                                                            | JS  | Java | .NET | Python |
| ----------------- | ------------------------------------------------------------------------------------- | --- | ---- | ---- | ------ |
| Funcionalidades principales | Enviar y recibir mensajes SMS                                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Habilitar informes de entrega para los mensajes enviados                                             | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Todos los juegos de caracteres (compatibilidad con lenguaje/Unicode)                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Compatibilidad con mensajes largos (hasta 2048 bytes)                                          | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Concatenación automática de mensajes largos                                                   | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Envío de mensajes a varios destinatarios a la vez                                        | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Compatibilidad con la idempotencia                                                               | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Etiquetas personalizadas para los mensajes.                                                             | ✔️   | ✔️    | ✔️    | ✔️      |
| Events            | Usar Event Grid para configurar webhooks para recibir mensajes entrantes e informes de entrega | ✔️   | ✔️    | ✔️    | ✔️      |
| Número de teléfono      | Números de teléfono gratuitos                                                                     | ✔️   | ✔️    | ✔️    | ✔️      |
| Llamadas RTC      | Agregar funcionalidades de llamadas RTC al número de teléfono gratuito habilitado para SMS                    | ✔️   | ✔️    | ✔️    | ✔️      |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al envío de SMS](../../quickstarts/telephony-sms/send.md)

Puede que los siguientes documentos le resulten interesantes:

- Familiarización con los [conceptos de SMS](../telephony-sms/concepts.md) generales
- Obtención de un [número de teléfono](../../quickstarts/telephony-sms/get-phone-number.md) compatible con SMS
- [Tipos de número de teléfono en Azure Communication Services](../telephony-sms/plan-solution.md)
