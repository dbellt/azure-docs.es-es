---
title: archivo de inclusión
description: archivo de inclusión
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 02/17/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: e160b21581bc7b5fa38b12309bd9deb90bfbbe51
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107251434"
---
En la siguiente tabla se muestra la información de cuotas específica de la mensajería de Azure Service Bus. Para obtener información sobre los precios y otras cuotas de Service Bus, vea [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

| Nombre de cuota | Ámbito | Notas | Value |
| --- | --- | --- | --- |
| Número máximo de espacios de nombres Básico o Estándar por suscripción de Azure |Espacio de nombres |Azure Portal rechaza las solicitudes posteriores de espacios de nombres Básico o Estándar adicionales. | El valor predeterminado es 100. <br/> El valor máximo es 1000. <br/><br/> Para aumentar el límite, póngase en contacto con el equipo de soporte técnico de Azure. |
| Número máximo de espacios de nombres Premium por suscripción de Azure |Espacio de nombres |El portal rechaza las solicitudes posteriores de espacios de nombres Premium adicionales. | El valor predeterminado es 100. <br/> El valor máximo es 1000. <br/><br/> Para aumentar el límite, póngase en contacto con el equipo de soporte técnico de Azure. |
| Tamaño de cola o tema |Entidad |Se define al crear la cola o el tema. <br/><br/> Los sucesivos mensajes entrantes se rechazan y el código que llama recibe una excepción. |1, 2, 3, 4 o 5 GB.<br /><br />En la SKU Premium y en la SKU Estándar con [particiones](../articles/service-bus-messaging/service-bus-partitioning.md) habilitadas, el tamaño máximo de cola o tema es de 80 GB. |
| Número de conexiones simultáneas en un espacio de nombres |Espacio de nombres |Las solicitudes posteriores de conexiones adicionales se rechazan y el código que llama recibe una excepción. Las operaciones REST no cuentan para las conexiones de TCP simultáneas. |Número neto de mensajes: 1000.<br /><br />AMQP: 5000. |
| Número de solicitudes de recepción simultáneas en una cola, un tema o una entidad de suscripción |Entidad |Las solicitudes de recepción posteriores se rechazan y el código que llama recibe una excepción. Esta cuota se aplica a un número combinado de operaciones de recepción simultáneas en todas las suscripciones de un tema. |5\.000 |
| Número de temas o colas por espacio de nombres |Espacio de nombres |Se rechazan las posteriores solicitudes de creación colas o temas nuevos en el espacio de nombres. Como resultado, si se configuran mediante [Azure Portal][Azure portal], se genera un mensaje de error. Si se realiza una llamada desde la API de administración, el código de llamada recibe una excepción. |10 000 para el nivel Básico o Estándar. El número total de temas y colas de un espacio de nombres debe ser menor o igual a 10 000. <br/><br/>En el nivel Premium, 1000 por unidad de mensajería (MU). |
| Número de [temas o colas con particiones](../articles/service-bus-messaging/service-bus-partitioning.md) por espacio de nombres |Espacio de nombres |Se rechazan las posteriores solicitudes de creación colas o temas con particiones nuevos en el espacio de nombres. Como resultado, si se configuran mediante [Azure Portal][Azure portal], se genera un mensaje de error. Si se realiza una llamada desde la API de administración, el código que llama recibe una excepción **QuotaExceededException**. |Niveles Básico y Estándar: 100.<br/><br/>No se admiten entidades con particiones en el nivel [Premium](../articles/service-bus-messaging/service-bus-premium-messaging.md).<br/><br />Cada cola o tema con particiones cuenta para la cuota de 1000 entidades por espacio de nombres. |
| Tamaño máximo de cualquier ruta de entidad de mensajería: cola o tema |Entidad |- |260 caracteres. |
| Tamaño máximo de cualquier nombre de entidad de mensajería: espacio de nombres, suscripción o regla de suscripción |Entidad |- |50 caracteres. |
| Tamaño máximo de un identificador de mensaje | Entidad |- | 128 |
| Tamaño máximo de un identificador de sesión de mensaje | Entidad |- | 128 |
| Tamaño de mensaje de una cola, un tema o una entidad de suscripción |Entidad |Los mensajes entrantes que superan estas cuotas se rechazan y el código que llama recibe una excepción. |Tamaño de mensaje máximo: 256 KB para el [nivel Estándar](../articles/service-bus-messaging/service-bus-premium-messaging.md), 1 MB para el [nivel Premium](../articles/service-bus-messaging/service-bus-premium-messaging.md). <br /><br />Debido a la sobrecarga del sistema, este límite es menor que estos valores.<br /><br />Tamaño de encabezado máximo: 64 KB.<br /><br />Número máximo de propiedades de encabezado en el contenedor de propiedades: **byte/int.MaxValue**.<br /><br />Tamaño máximo de la propiedad en el contenedor de propiedades: sin límite explícito. Limitado por tamaño de encabezado máximo. |
| Tamaño de propiedad de mensaje para una cola, un tema o una entidad de suscripción |Entidad | Se genera la excepción `SerializationException`. |El tamaño máximo de propiedad de mensaje para cada propiedad es 32 000. El tamaño acumulado de todas las propiedades no puede superar 64 000. Este límite se aplica a todo el encabezado del mensaje asincrónico, que contiene tanto las propiedades de usuario como las propiedades del sistema, como el número de secuencia, la etiqueta y el identificador del mensaje. |
| Número de suscripciones por tema |Entidad |Se rechazan las posteriores solicitudes de creación de suscripciones adicionales para el tema. Como resultado, si se configura a través del portal, se muestra un mensaje de error. Si se realiza una llamada desde la API de administración, el código de llamada recibe una excepción. |2000 por tema para el nivel Estándar y el nivel Premium. |
| Número de filtros SQL por tema |Entidad |Se rechazan las posteriores solicitudes de creación de filtros adicionales en el tema y el código que realiza la llamada recibe una excepción. |2\.000 |
| Número de filtros de correlación por tema |Entidad |Se rechazan las posteriores solicitudes de creación de filtros adicionales en el tema y el código que realiza la llamada recibe una excepción. |100 000 |
| Tamaño de filtros o acciones SQL |Espacio de nombres |Se rechazan las posteriores solicitudes de creación de filtros adicionales y el código que realiza la llamada recibe una excepción. |Longitud máxima de la cadena de condición de filtro: 1024 (1 K).<br /><br />Longitud máxima de la cadena de acción de regla: 1024 (1 K).<br /><br />Número máximo de expresiones por acción de regla: 32. |
| Número de reglas de autorización de acceso compartido por espacio de nombres, cola o tema |Entidad, espacio de nombres |Se rechazan las posteriores solicitudes de creación de reglas adicionales en el tema y el código que llama recibe una excepción. |Número máximo de reglas por tipo de entidad: 12. <br /><br /> Las reglas que se configuran en un espacio de nombres de Service Bus se aplican a todos los tipos: colas, temas. |
| Número de mensajes por transacción | Transacción | Los mensajes entrantes adicionales se rechazan y el código que llama recibe una excepción que indica "Cannot send more than 100 messages in a single transaction" (No se pueden enviar más de 100 mensajes en una sola transacción). | 100 <br /><br /> Para ambas operaciones **Send()** y **SendAsync()** . |
| Número de reglas de filtro de dirección IP y red virtual | Espacio de nombres | &nbsp; | 128 |

[Azure portal]: https://portal.azure.com
