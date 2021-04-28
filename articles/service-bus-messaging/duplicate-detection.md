---
title: Detección de mensajes duplicados de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo puede detectar duplicados en mensajes de Azure Service Bus. El mensaje duplicado se puede omitir y quitar.
ms.topic: article
ms.date: 04/19/2021
ms.openlocfilehash: baeda3509cb5646c658f79fb11610ecfdd1ffd3d
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751281"
---
# <a name="duplicate-detection"></a>Detección de duplicados

Si una aplicación deja de funcionar debido a un error grave inmediatamente después de enviar un mensaje y la instancia de la aplicación reiniciada cree falsamente que el mensaje anterior no se ha producido, un posterior envío provoca que el mismo mensaje aparezca en el sistema dos veces.

También puede producirse un error en el nivel de cliente o de red un momento antes y que un mensaje enviado se confirme en la cola, sin que la confirmación se devuelva al cliente correctamente. Este escenario deja al cliente en duda sobre el resultado de la operación de envío.

La detección de duplicados elimina la duda en estas situaciones, ya que permite al remitente reenviar el mismo mensaje, y la cola o el tema descartan las copias duplicadas.

> [!NOTE]
> El nivel Básico de Service Bus no admite la detección de duplicados. Los niveles Estándar y Premium admiten la detección de duplicados. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

## <a name="how-it-works"></a>¿Cómo funciona? 
La habilitación de la detección de duplicados ayuda a mantener el seguimiento del valor *MessageId* controlado por la aplicación de todos los mensajes enviados a una cola o un tema durante una ventana de tiempo específica. Si se envía algún mensaje nuevo con *MessageId* que se haya registrado durante el periodo de tiempo, se notifica como aceptado (la operación de envío se realiza correctamente), pero el mensaje recién enviado se ignora y descarta al instante. No se tiene en cuenta ninguna otra parte del mensaje que no sea *MessageId*.

El control de aplicación del identificador es esencial, ya que es lo único que permite que la aplicación enlace el valor *MessageId* a un contexto de proceso empresarial desde el que se pueda reconstruir de manera predecible en caso de error.

En el caso de un proceso empresarial en el que se envían varios mensajes en el transcurso del control de algún contexto de aplicación, el valor *MessageId* puede constar del identificador de contexto de nivel de aplicación, como un número de orden de compra, y el asunto del mensaje, por ejemplo, **12345.2017/payment**.

El valor *MessageId* siempre puede ser algún GUID, pero el anclaje del identificador al proceso de negocio produce una capacidad de repetición predecible, lo que resulta conveniente para usar con efectividad la característica de detección de duplicados.

> [!IMPORTANT]
>- Cuando la **creación de particiones** está **habilitada**, `MessageId+PartitionKey` se usa para determinar la unicidad. Cuando se habilitan las sesiones, la clave de partición y el identificador de sesión deben ser iguales. 
>- Cuando la **creación de particiones** está **deshabilitada** (valor predeterminado), solo se usa `MessageId` para determinar la unicidad.
>- Para información sobre SessionId, PartitionKey y MessageId, consulte [Uso de claves de partición](service-bus-partitioning.md#use-of-partition-keys).
>- El [nivel premier](service-bus-premium-messaging.md) no admite la creación de particiones, por lo que se recomienda usar identificadores de mensaje únicos en las aplicaciones y no depender de claves de partición para la detección de duplicados. 


## <a name="duplicate-detection-window-size"></a>Tamaño de la ventana de detección de duplicados

Además de habilitar la detección de duplicados, también puede configurar el tamaño de la ventana de tiempo del historial de detección de duplicados durante la cual se retienen los id. de mensaje.
Este valor predeterminado es de 10 minutos para colas y temas, con un valor mínimo de 20 segundos y un valor máximo de 7 días.

La habilitación de la detección de duplicados y el tamaño de la ventana afecta directamente al rendimiento de la cola (y del tema), dado que todos los identificadores de mensaje registrados deben coincidir con el identificador de mensaje recién enviado.

Mantener la ventana pequeña significa que menos identificadores de mensaje se deben conservar y coincidir, y el rendimiento resulta afectado en un menor grado. En las entidades de alto rendimiento que requieren la detección de duplicados, debe mantener la ventana lo más pequeña posible.

## <a name="next-steps"></a>Pasos siguientes
Puede habilitar la detección de mensajes duplicados mediante Azure Portal, PowerShell, CLI, la plantilla de Resource Manager, .NET, Java, Python y JavaScript. Para obtener más información, consulte [Habilitar la detección de mensajes duplicados](enable-duplicate-detection.md). 

En escenarios en los que el código de cliente no puede volver a enviar un mensaje con el mismo valor de *MessageId* que antes, es importante diseñar mensajes que se puedan volver a procesar de forma segura. En esta [entrada de blog sobre idempotencia](https://particular.net/blog/what-does-idempotent-mean) se describen diversas técnicas de cómo hacerlo.

Pruebe los ejemplos en el lenguaje que prefiera para explorar las características de Azure Service Bus. 

- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)
- [Ejemplos de Azure.Messaging.ServiceBus para .NET](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/)

A continuación, encontrará ejemplos de las bibliotecas cliente de .NET y Java anteriores:
- [Ejemplos de Microsoft.Azure.ServiceBus para .NET](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/)
- [Ejemplos de azure-servicebus para Java](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/MessageBrowse)

