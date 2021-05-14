---
title: Entidades de mensajería de Azure Service Bus con reenvío automático
description: En este artículo se describe cómo encadenar una cola o suscripción de Azure Service Bus a otra cola u otro tema.
ms.topic: article
ms.date: 04/23/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 78fb7478e0584d7c6dc79829d4bb242a448d43bd
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988701"
---
# <a name="chaining-service-bus-entities-with-autoforwarding"></a>Encadenamiento de entidades de Service Bus con reenvío automático

La característica de *reenvío automático* de Service Bus permite encadenar una cola o suscripción a otra cola u otro tema que forman parte del mismo espacio de nombres. Cuando el reenvío automático está habilitado, Service Bus elimina automáticamente los mensajes que se colocan en la primera cola o suscripción (origen) y los coloca en la segunda cola o en el segundo tema (destino). Todavía se puede enviar un mensaje a la entidad de destino directamente.

> [!NOTE]
> El nivel Básico de Service Bus no admite la característica de reenvío automático. Los niveles Estándar y Prémium admiten la característica. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

La entidad de destino debe existir en el momento en que se creó la entidad de origen. Si la entidad de destino no existe, Service Bus devuelve una excepción cuando se le pide que cree la entidad de origen.

## <a name="scenarios"></a>Escenarios

### <a name="scale-out-an-individual-topic"></a>Escalado horizontal de un tema individual
El reenvío automático se puede usar para escalar horizontalmente un tema individual. Service Bus limita el [número de suscripciones de un tema dado](service-bus-quotas.md) a 2000. Para alojar suscripciones adicionales, cree temas de segundo nivel. Aunque no tenga la limitación de Service Bus sobre el número de suscripciones, el hecho de agregar un segundo nivel de temas puede mejorar el rendimiento general del tema.

![Diagrama de un escenario de reenvío automático que muestra un mensaje procesado a través de un tema de pedidos que se puede bifurcar a cualquiera de los tres temas de pedidos de segundo nivel.][0]

### <a name="decouple-message-senders-from-receivers"></a>Desacoplar remitentes de mensajes de receptores
También puede usarlo para desacoplar los remitentes de los destinatarios. Por ejemplo, suponga que un sistema ERP consta de tres módulos: procesamiento de pedidos, administración de inventario y administración de relaciones con clientes. Cada uno de estos módulos genera mensajes que se ponen en cola en el tema correspondiente. Alice y Bob son representantes de ventas que están interesados en todos los mensajes relacionados con sus clientes. Para recibir dichos mensajes, Alice y Bob crean una cola personal y una suscripción en cada uno de los temas de ERP que reenvían automáticamente todos los mensajes a su cola.

![Diagrama de un escenario de reenvío automático que muestra tres módulos de procesamiento que envían mensajes a través de tres temas correspondientes a dos colas independientes.][1]

Si Alice se va de vacaciones, se llena su cola personal, en lugar del tema de ERP. En este escenario, como un representante de ventas no ha recibido ningún mensaje, ninguno de los temas de ERP alcanza la cuota.

> [!NOTE]
> Cuando el reenvío automático está configurado, el valor de AutoDeleteOnIdle en **el origen y el destino** se establece automáticamente en el valor máximo del tipo de datos.
> 
>   - En el lado de origen, el reenvío automático actúa como una operación de recepción. Por lo tanto, el origen que tiene configurado el reenvío automático nunca está realmente "inactivo".
>   - En el lado de destino, esto se hace para garantizar que siempre haya un destino al que reenviar el mensaje.

## <a name="autoforwarding-considerations"></a>Consideraciones del reenvío automático

Si la entidad de destino acumula muchos mensajes y supera la cuota (o la entidad de destino está deshabilitada), la entidad de origen agrega los mensajes a su [cola de mensajes fallidos](service-bus-dead-letter-queues.md) hasta que haya espacio en el destino (o hasta que se vuelva a habilitar la entidad). Esos mensajes siguen estando en la cola de mensajes fallidos, por lo que debe recibirlos y procesarlos explícitamente desde dicha cola.

Al encadenar temas individuales para obtener un tema compuesto con muchas suscripciones, se recomienda tener un número moderado de suscripciones en el tema de primer nivel y muchas suscripciones en los temas de segundo nivel. Por ejemplo, un tema de primer nivel con 20 suscripciones, cada una de ellas encadenada a un tema de segundo nivel con 200 suscripciones, ofrece un mayor rendimiento que un tema de primer nivel con 200 suscripciones, cada una de ellas encadenada a un tema de segundo nivel con 20 suscripciones.

Service Bus factura una operación por cada mensaje reenviado. Por ejemplo, el envío de un mensaje a un tema con 20 suscripciones, cada una de ellas configurada para reenviar automáticamente mensajes a otra cola, o a otro tema, se factura como 21 operaciones si todas las suscripciones del primer nivel reciben una copia del mensaje.

Para crear una suscripción encadenada a otra cola o a otro tema, el creador debe tener permisos de **administración** tanto en la entidad de origen como en la de destino. Para enviar mensajes al tema de origen, solo se requieren permisos de **envío** en el tema de origen.

No cree una cadena que supere los 4 saltos. Los mensajes que superan los 4 saltos se colocan en la cola de mensajes fallidos.

## <a name="next-steps"></a>Pasos siguientes
Para aprender a habilitar o deshabilitar el reenvío automático de maneras diferentes (Azure Portal, PowerShell, CLI, plantilla de Azure Resource Management, etc.), consulte [Habilitación del reenvío automático para colas y suscripciones](enable-auto-forward.md).


[0]: ./media/service-bus-auto-forwarding/IC628631.gif
[1]: ./media/service-bus-auto-forwarding/IC628632.gif
[Partitioned messaging entities]: service-bus-partitioning.md
