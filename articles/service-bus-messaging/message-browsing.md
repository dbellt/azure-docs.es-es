---
title: Exploración de mensajes de Azure Service Bus
description: Explorar e inspeccionar los mensajes de Service Bus permite a un cliente de Azure Service Bus enumerar todos los mensajes de una cola o suscripción.
ms.topic: article
ms.date: 03/29/2021
ms.openlocfilehash: deafe9e6ddeeebf233922aade36823ddaaede864
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107520129"
---
# <a name="message-browsing"></a>Exploración de mensajes
La exploración de mensajes, o la inspección, permite a un cliente de Service Bus enumerar todos los mensajes de una cola o una suscripción con fines de diagnóstico o depuración.

La operación de inspección de una cola o una suscripción devuelve como máximo el número solicitado de mensajes. En la tabla siguiente se muestran los tipos de mensajes devueltos por la operación de inspección. 

| Tipo de mensajes | ¿Se incluye? | 
| ---------------- | ----- | 
| Mensajes activos | Sí |
| Mensajes fallidos | No | 
| Mensajes bloqueados | Sí |
| Mensajes expirados |  Puede ser (antes de que se les haya enviado un mensaje fallido). |
| Mensajes programados | Sí, para las colas. No, para suscripciones. |

## <a name="dead-lettered-messages"></a>Mensajes fallidos
Para ver los mensajes **fallidos** de una cola o suscripción, la operación de inspección debe ejecutarse en la cola de mensajes no enviados asociada a la cola o la suscripción. Para obtener más información, vea [Ruta de acceso para la cola de mensajes fallidos](service-bus-dead-letter-queues.md#path-to-the-dead-letter-queue).

## <a name="expired-messages"></a>Mensajes expirados
Los mensajes que han expirado se pueden incluir en los resultados devueltos por la operación de inspección. Los mensajes consumidos y expirados se limpian mediante una ejecución de "recolección de elementos no utilizados" asincrónica. Este paso no se produce necesariamente inmediatamente después de que expiren los mensajes. Por eso, una operación interactiva puede devolver mensajes que ya han expirado. Estos mensajes se quitarán o se pondrán en la cola de mensajes fallidos cuando se invoque una operación de recepción en la cola o la suscripción la próxima vez. Tenga en cuenta este comportamiento al intentar recuperar los mensajes aplazados de la cola. 

Un mensaje expirado ya no es apto para la recuperación normal mediante ningún otro medio, incluso cuando se devuelve con Peek. Estos mensajes se devuelven por diseño dado que Peek es una herramienta de diagnóstico que refleja el estado actual del registro.

## <a name="locked-messages"></a>Mensajes bloqueados
La operación de inspección también devuelve los mensajes que estaban **bloqueados** y que actualmente son procesados por otros receptores. Sin embargo, dado que Peek devuelve una instantánea desconectada, el estado de bloqueo de un mensaje no se puede observar en los mensajes inspeccionados.

## <a name="peek-apis"></a>API de Peek
La operación de inspección es compatible con las colas y suscripciones y con las colas correspondientes de mensajes fallidos. 

Cuando se llama repetidamente a la operación de inspección, se enumeran todos los mensajes del registro de la cola o la suscripción, por orden, desde el número de secuencia más bajo disponible hasta el más alto. Este es el orden en que los mensajes se ponen en cola; no es el orden en que finalmente se podrían recuperar.

También se puede pasar un valor de SequenceNumber a una operación de inspección. Este valor se usará para determinar por dónde empezar la operación de inspección. Posteriormente, puede llamar a la operación de inspección sin especificar el parámetro para que la enumeración sea más amplia.

## <a name="next-steps"></a>Pasos siguientes
Pruebe los ejemplos en el idioma de su elección para explorar la característica de inspección o exploración de mensajes:

- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java](/samples/azure/azure-sdk-for-java/servicebus-samples/) - Ejemplo de **Peak at a message**
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/) - Ejemplo de **receive_peek.py**
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/) - Ejemplo de **browseMessages.js**
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/) - Ejemplo de **browseMessages.js**
- [Ejemplos de Azure.Messaging.ServiceBus para .NET](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/). Vea los métodos de inspección de clases receptoras en la [documentación de referencia](/dotnet/api/azure.messaging.servicebus).

Encontrará ejemplos de las bibliotecas cliente de .NET y Java a continuación:
- [Ejemplos de Microsoft.Azure.ServiceBus para .NET](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/) - Ejemplo de **Message Browsing (Peek)** 
- [Ejemplos de azure-servicebus para Java](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/MessageBrowse) -  Ejemplo de **Message Browse** 
