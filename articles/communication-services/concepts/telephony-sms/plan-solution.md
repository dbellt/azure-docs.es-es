---
title: Tipos de número de teléfono en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Aprenda a usar eficazmente distintos tipos de números de teléfono para SMS y telefonía.
author: prakulka
manager: nmurav
services: azure-communication-services
ms.author: prakulka
ms.date: 06/30/2021
ms.topic: overview
ms.custom: references_regions
ms.service: azure-communication-services
ms.openlocfilehash: 150daaa834cefbb3f74e6711fea500138774f2de
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113111094"
---
# <a name="phone-number-types-in-azure-communication-services"></a>Tipos de número de teléfono en Azure Communication Services

> [!IMPORTANT]
> La disponibilidad de números de teléfono se limita actualmente a las suscripciones de Azure de pago que tienen una dirección de facturación en Estados Unidos y recursos de servicios de comunicación cuyos datos están ubicados en EE. UU. No se pueden obtener números de teléfono para cuentas de prueba ni mediante créditos gratuitos de Azure. Para obtener más información, consulte la sección sobre [elegibilidad de las suscripciones](#azure-subscriptions-eligibility) de este documento.


Azure Communication Services le permite usar números de teléfono para realizar llamadas de voz y enviar mensajes SMS con la red telefónica conmutada pública (RTC). En este documento revisaremos los tipos de número de teléfono, las opciones de configuración y la disponibilidad de regiones para planear la solución de telefonía y SMS con Communication Services.

## <a name="azure-subscriptions-eligibility"></a>Elegibilidad de las suscripciones de Azure

Para adquirir un número de teléfono, debe disponer de una suscripción de Azure de pago. No se pueden adquirir números de teléfono en cuentas de evaluación gratuita ni mediante créditos gratuitos de Azure.

La disponibilidad de números de teléfono se limita actualmente a las suscripciones de Azure que tienen una dirección de facturación en Estados Unidos y recursos de servicios de comunicación cuyos datos están ubicados en EE. UU.


## <a name="number-types-and-features"></a>Tipos de números y características
Communication Services ofrece dos tipos de número de teléfono: **local** y **gratuito**.

### <a name="local-numbers"></a>Números locales
Los números locales (geográficos) son números de teléfono de 10 dígitos que se componen de los códigos de área locales de Estados Unidos. Por ejemplo, `+1 (206) XXX-XXXX` es un número local con el código de área `206`. Este código de área está asignado a la ciudad de Seattle. Estos números de teléfono suelen ser usados por personas y empresas locales. Azure Communication Services ofrece números locales de Estados Unidos. Estos números se pueden usar para realizar llamadas telefónicas, pero no para enviar mensajes SMS.

### <a name="toll-free-numbers"></a>Números de teléfono gratuitos
Los números gratuitos son números de teléfono de 10 dígitos con códigos de área distintos, a los que se puede llamar desde cualquier número de teléfono de forma gratuita. Por ejemplo, `+1 (800) XXX-XXXX` es un número gratuito en la región de Estados Unidos. Estos números de teléfono se suelen usar con fines de servicio al cliente. Azure Communication Services ofrece números de teléfono gratuitos de Estados Unidos. Estos números se pueden usar para realizar llamadas telefónicas y para enviar mensajes SMS. Los números de teléfono gratuitos no se pueden usar para personas y solo se pueden asignar a aplicaciones.

#### <a name="choosing-a-phone-number-type"></a>Elección de un tipo de número de teléfono

Si la aplicación va a usar el número de teléfono (por ejemplo, para realizar llamadas o enviar mensajes en nombre del servicio), puede seleccionar un número de teléfono gratuito o local (geográfico). Puede seleccionar un número de teléfono gratuito si la aplicación envía mensajes SMS o realiza llamadas.

Si el número de teléfono lo utiliza una persona (por ejemplo, un usuario de la aplicación que realiza las llamadas), debe usar un número de teléfono local (geográfico).

En la tabla siguiente se resumen estos tipos de números de teléfono:

| Tipo de número de teléfono | Ejemplo                              | Disponibilidad de país    | Funcionalidad del número de teléfono |Caso de uso común                                                                                                     |
| ----------------- | ------------------------------------ | ----------------------- | ------------------------|------------------------------------------------------------------------------------------------------------------- |
| Local (geográfico)        | +1 (código de área local) XXX XX XX  | US                      | Llamadas (salientes) | Asignación de números de teléfono a los usuarios de las aplicaciones  |
| Gratuito         | \+ 1 (*código* de área gratuito) XXX XX XX | US                      | Llamadas (salientes), SMS (entrantes y salientes)| Asignación de números de teléfono a sistemas o bots de respuesta interactiva de voz (IVR) y aplicaciones de SMS                                        |


### <a name="phone-number-capabilities-in-azure-communication-services"></a>Funcionalidades de los números de teléfono en Azure Communication Services

[!INCLUDE [Emergency Calling Notice](../../includes/emergency-calling-notice-include.md)]

Para la mayoría de los números de teléfono, puede configurar un conjunto de funcionalidades "a la carta". Estas funcionalidades se pueden seleccionar a medida que alquila los números de teléfono en Azure Communication Services.

Las funcionalidades disponibles dependen del país en el que opera, el caso de uso y el tipo de número de teléfono que haya seleccionado. Estas funcionalidades varían según el país debido a los requisitos normativos. Azure Communication Services ofrece las siguientes funcionalidades para los números de teléfono:

- **One-way outbound SMS** (SMS de salida unidireccional): esta opción permite enviar mensajes SMS a los usuarios. Resulta útil para escenarios de autenticación en dos fases y de notificación.
- **Two-way inbound and outbound SMS** (SMS de entrada y salida bidireccional): esta opción permite enviar y recibir mensajes de los usuarios mediante números de teléfono. Resulta útil en escenarios de atención al cliente.
- **One-way outbound telephone calling** (Llamada telefónica de salida unidireccional): esta opción permite realizar llamadas a los usuarios y configurar el identificador del autor de la llamada para las llamadas salientes que realice el servicio. Resulta útil en escenarios de atención al cliente y de notificación mediante voz.

## <a name="countryregion-availability"></a>Disponibilidad de país/región

En la tabla siguiente se muestra dónde puede adquirir los diferentes tipos de números de teléfono junto con las características de llamada y SMS entrantes y salientes asociadas a estos tipos de números de teléfono.

|Tipo de número de teléfono| Adquirir números en | Realizar llamadas a                                        | Recibir llamadas de                                    |Enviar mensajes a       | Recibir mensajes de |
|-----------| ------------------ | ---------------------------------------------------  |-------------------------------------------------------|-----------------------|--------|
| Local (geográfico)  | US                 | Estados Unidos, Canadá, Reino Unido, Alemania, Francia. +más*| Estados Unidos, Canadá, Reino Unido, Alemania, Francia. +más* |No disponible| No disponible |
| Gratuito | US                 | US                                                   | US                                                    |US                | US |

*Para más información sobre los destinos de llamada y los precios, consulte la [página de precios](../pricing.md).


## <a name="next-steps"></a>Pasos siguientes

### <a name="quickstarts"></a>Guías de inicio rápido

- [Obtención de un número de teléfono](../../quickstarts/telephony-sms/get-phone-number.md)
- [Realización de una llamada](../../quickstarts/voice-video-calling/calling-client-samples.md)
- [Enviar un mensaje de texto.](../../quickstarts/telephony-sms/send.md)

### <a name="conceptual-documentation"></a>Documentación conceptual

- [Conceptos de voz y vídeo](../voice-video-calling/about-call-types.md)
- [Conceptos de telefonía](./telephony-concept.md)
- [Flujos de llamadas](../call-flows.md)
- [Precios](../pricing.md)
