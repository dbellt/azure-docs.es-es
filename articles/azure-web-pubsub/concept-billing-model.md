---
title: Modelo de facturación del servicio Azure Web PubSub
description: Información general sobre conceptos clave del modelo de facturación del servicio Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: conceptual
ms.date: 03/29/2021
ms.openlocfilehash: 0d1ba3de5686e161fbd118f78bd4734034dbc1bb
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167124"
---
# <a name="billing-model-of-azure-web-pubsub-service"></a>Modelo de facturación del servicio Azure Web PubSub

El modelo de facturación del servicio Azure Web PubSub se basa en el número de unidades y el volumen de tráfico saliente (recuento de mensajes). En este artículo se explica cómo se definen y contabilizan las unidades y el tráfico saliente (recuento de mensajes) a efectos de facturación.

## <a name="how-units-are-counted-with-billing-model"></a>Cómo se contabilizan las unidades con el modelo de facturación

### <a name="what-is-the-connection"></a>Qué es una conexión

Una conexión tiene lugar entre el cliente y el servidor. Puede supervisar el número de conexiones en Azure Portal. En la vista "Conexión (máx.)" se muestra el número máximo de conexiones en un período específico. 

### <a name="what-is-the-unit"></a>Qué es una unidad

La unidad es un concepto abstracto de la funcionalidad del servicio Azure Web PubSub. Cada unidad admite 1000 conexiones simultáneas como máximo. Cada instancia del servicio Azure Web PubSub puede tener 1, 2, 5, 10, 20, 50 o 100 unidades. Por lo tanto, el recuento de unidades especifica cuántas conexiones puede aceptar la instancia del servicio Web PubSub.

###  <a name="how-units-are-counted-with-billing-model"></a>Cómo se contabilizan las unidades con el modelo de facturación

Las unidades se contabilizan en función del número y del tiempo de uso (segundos) y se facturan diariamente. 

Por ejemplo, imagine que tiene una instancia del servicio Azure Web PubSub con 5 unidades, la amplía a 10 de 10:00 a 16:00 y, luego, vuelve a reducirla a 5 después de las 16:00. El resultado son 5 unidades durante 18 horas y 10 durante 6 horas de un día específico.

> Uso de las unidades a efectos de facturación = (5 unidades * 18 horas + 10 unidades * 6 horas) / 24 horas = 6,25 unidades/día

## <a name="how-outbound-traffic-is-counted-with-billing-model"></a>Cómo se contabiliza el tráfico saliente con el modelo de facturación

### <a name="what-is-inboundoutbound-traffic"></a>Qué es el tráfico entrante o saliente 

El tráfico saliente son los mensajes que se envían fuera del servicio Azure Web PubSub. El tráfico saliente puede supervisarse mediante Azure Portal. En la vista "Tráfico saliente (total)" se muestra el volumen agregado de mensajes salientes (bytes) en un período específico:

- Los mensajes transmitidos desde el servicio a los receptores
- Los mensajes enviados desde el servicio a los webhooks ascendentes
- Los registros de diagnóstico con la [herramienta de seguimiento en directo](./howto-troubleshoot-diagnostic-logs.md#capture-diagnostic-logs-with-azure-web-pubsub-service-live-trace-tool) 

El tráfico entrante son los mensajes que se envían al servicio Azure Web PubSub. 

- Los mensajes enviados desde clientes al servicio
- Los mensajes enviados desde el servidor o las funciones al servicio

### <a name="what-is-message-count"></a>Qué es el recuento de mensajes

A efectos de facturación, el recuento de mensajes es un concepto abstracto que se define como el volumen de tráfico saliente (bytes) dividido entre 2 KB. Por ejemplo, un tráfico de 100 KB se contabiliza como 50 mensajes.  

### <a name="how-traffic-is-counted-with-billing-model"></a>Cómo se contabiliza el tráfico con el modelo de facturación

A efectos de facturación, solo se contabiliza el tráfico saliente. 

Por ejemplo, imagine que tiene una aplicación con el servicio Azure Web PubSub y Azure Functions. Un usuario ha difundido 4 KB de datos a 10 conexiones de un grupo. El resultado son 4 KB para la conexión ascendente del servicio a la función y 40 KB de la transmisión del servicio a 10 conexiones.

> Tráfico saliente a efectos de facturación = 4 KB + 40 KB = 44 KB

> Recuento de mensajes equivalente = 44 KB / 2 KB = 22

El servicio Azure Web PubSub también ofrece una cuota gratuita diaria de tráfico saliente (recuento de mensajes) en función del uso de las unidades. El tráfico saliente (recuento de mensajes) que exceda la cuota gratuita constituye el tráfico saliente adicional (mensajes adicionales). Tomando como ejemplo el plan estándar, la cuota gratuita es de 2 000 000 KB de tráfico saliente (1 000 000 mensajes) por unidad y día.

Con el ejemplo anterior de uso de unidades, la aplicación usa 6,25 unidades al día, que garantiza una cuota gratuita diaria como tráfico saliente de 12 500 000 KB (6,25 millones de mensajes). Si suponemos un tráfico saliente diario de 30 000 000 KB (15 millones de mensajes), los mensajes adicionales representarán un tráfico saliente de 17 500 000 KB (8,75 millones de mensajes). Como resultado, se le facturarán 6,25 unidades estándar y 8,75 unidades de mensajes adicionales con relación al día.

## <a name="pricing"></a>Precios 

El servicio Azure Web PubSub ofrece varios planes con diferentes precios. Una vez que entienda cómo se contabilizan el número de unidades y el volumen de tráfico saliente (recuento de mensajes) con el modelo de facturación, puede obtener más información sobre los [precios del servicio Azure Web PubSub](https://azure.microsoft.com/pricing/details/web-pubsub).





