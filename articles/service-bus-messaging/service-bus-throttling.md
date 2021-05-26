---
title: Introducción a la limitación de Azure Service Bus | Microsoft Docs
description: 'Introducción a la limitación de Service Bus: niveles Estándar y Premium.'
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 95b03859b26d97c1089b6112b41511618abdcffe
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110191107"
---
# <a name="throttling-operations-on-azure-service-bus"></a>Operaciones de limitación en Azure Service Bus

Las soluciones nativas en la nube ofrecen una apariencia de recursos ilimitados que se pueden escalar con la carga de trabajo. Aunque esta apariencia es más real en la nube que en los sistemas locales, existen límites en la nube.

Estos límites pueden provocar la limitación de las solicitudes de las aplicaciones cliente en los niveles Estándar y Premium, como se analiza en este artículo. 

## <a name="throttling-in-azure-service-bus-standard-tier"></a>Limitación en el nivel Estándar de Azure Service Bus

El nivel Estándar de Azure Service Bus funciona como una instalación multiinquilino con un modelo de precios de pago por uso. Aquí hay varios espacios de nombres en el mismo clúster que comparten los recursos asignados. El nivel Estándar es la opción recomendada para entornos de desarrollo, pruebas y control de calidad al igual que los sistemas de producción de bajo rendimiento.

En el pasado, Azure Service Bus tenía límites amplios que dependían estrictamente del uso de los recursos. Sin embargo, existe la oportunidad de refinar la lógica de limitación y proporcionar un comportamiento de limitación predecible para todos los espacios de nombres que comparten estos recursos.

En un intento de garantizar el uso y la distribución equitativos de los recursos en todos los espacios de nombres estándar de Azure Service Bus que usan los mismos recursos, la lógica de las limitaciones se ha modificado para que se base en el crédito.

> [!NOTE]
> Es importante tener en cuenta que la limitación ***no es algo nuevo*** para Azure Service Bus o cualquier otro servicio nativo en la nube.
>
> La limitación basada en créditos simplemente está refinando la manera en que varios espacios de nombres comparten recursos en un entorno multiinquilino de nivel Estándar permitiendo así un uso equitativo por parte de todos los espacios de nombres que comparten los recursos.

### <a name="what-is-credit-based-throttling"></a>¿Qué son las limitaciones basadas en créditos?

Las limitaciones basadas en créditos limitan el número de operaciones que se pueden realizar en un espacio de nombres determinado en un período de tiempo concreto. 

A continuación, se muestra el flujo de trabajo de una limitación basada en créditos 

  * Al comienzo de cada período de tiempo, se proporciona un número determinado de créditos a cada espacio de nombres.
  * Todas las operaciones realizadas por las aplicaciones cliente del emisor o del receptor se contarán con estos créditos (y se restarán de los créditos disponibles).
  * Si se agotan los créditos, las operaciones subsiguientes se limitarán hasta el inicio del período de tiempo siguiente.
  * Los créditos se reponen al principio del siguiente período de tiempo.

### <a name="what-are-the-credit-limits"></a>¿Cuáles son los límites de créditos?

Los límites de créditos se establecen actualmente en "1000" créditos cada segundo (por espacio de nombres).

No todas las operaciones se crean igual. Estos son los costos en créditos de cada una de las operaciones: 

| Operación | Costo en créditos|
|-----------|-----------|
| Operaciones con datos (Send, SendAsync, Receive, ReceiveAsync, Peek) |1 crédito por mensaje |
| Operaciones de administración (Create, Read, Update, Delete on Queues, Topics, Subscriptions, Filters) | 10 créditos |

> [!NOTE]
> Tenga en cuenta que, al realizar el envío a un tema, cada mensaje se evalúa con respecto a los filtros antes de que esté disponible en la suscripción.
> Cada evaluación mediante filtros también cuenta con el límite de crédito (es decir, un crédito por evaluación de filtro).
>

### <a name="how-will-i-know-that-im-being-throttled"></a>¿Cómo sé que estoy limitado?

Cuando las solicitudes de las aplicaciones cliente se limitan, la aplicación recibe la siguiente respuesta del servidor y la registra.

```
The request was terminated because the entity is being throttled. Error code: 50009. Please wait 2 seconds and try again.
```

### <a name="how-can-i-avoid-being-throttled"></a>¿Cómo puedo evitar la limitación?

Con los recursos compartidos, es importante aplicar algún tipo de uso equitativo en varios espacios de nombres de Service Bus que compartan esos recursos. La limitación garantiza que cualquier pico de una carga de trabajo no provoque que se limiten otras cargas de trabajo en los mismos recursos.

Como se menciona más adelante en el artículo, no hay ningún riesgo en experimentar una limitación porque los SDK de cliente (y otras ofertas de PaaS de Azure) tienen la directiva de reintentos predeterminada integrada. Cualquier solicitud limitada se volverá a intentar con retroceso exponencial y se terminará de realizar cuando se repongan los créditos.

Es comprensible que algunas aplicaciones sean más sensibles a las limitaciones. En ese caso, se recomienda [migrar el espacio de nombres actual de Service Bus de Estándar a Premium](service-bus-migrate-standard-premium.md). 

Tras la migración, puede asignar recursos dedicados al espacio de nombres de Service Bus y escalar verticalmente los recursos de manera adecuada si hay un pico en la carga de trabajo y reducir así la probabilidad de que se limiten. Además, cuando la carga de trabajo se reduce a niveles normales, puede reducir verticalmente los recursos asignados al espacio de nombres.

## <a name="throttling-in-azure-service-bus-premium-tier"></a>Limitación en el nivel Prémium de Azure Service Bus

El nivel [Prémium de Azure Service Bus](service-bus-premium-messaging.md) asigna recursos dedicados, en términos de unidades de mensajería, a cada espacio de nombres configurado por el cliente. Estos recursos dedicados proporcionan una latencia y un rendimiento predecibles y se recomiendan para sistemas de producción de alto rendimiento o confidenciales.

Además, el nivel Premium también permite a los clientes escalar verticalmente su capacidad de rendimiento si experimentan picos en la carga de trabajo.

### <a name="how-does-throttling-work-in-service-bus-premium"></a>¿Cómo funciona la limitación en el nivel Prémium de Service Bus?

Con la asignación de recursos exclusiva del nivel Prémium de Service Bus, la limitación se rige únicamente por las limitaciones de los recursos asignados al espacio de nombres.

Si el número de solicitudes es mayor de lo que los recursos actuales pueden atender, se limitarán las solicitudes.

### <a name="how-will-i-know-that-im-being-throttled"></a>¿Cómo sé que estoy limitado?

Hay varias maneras de identificar una limitación en el nivel Prémium de Azure Service Bus: 
  * Las **solicitudes limitadas** aparecen en las [métricas de solicitudes de Azure Monitor](monitor-service-bus-reference.md#request-metrics) para identificar el número de solicitudes que se han limitado.
  * Un **uso elevado de la CPU** indica que la asignación de recursos actual es alta y que se pueden limitar las solicitudes si la carga de trabajo actual no se reduce.
  * Un **uso elevado de la memoria** indica que la asignación de recursos actual es alta y que se pueden limitar las solicitudes si la carga de trabajo actual no se reduce.

### <a name="how-can-i-avoid-being-throttled"></a>¿Cómo puedo evitar la limitación?

Como el espacio de nombres del nivel Premium de Service Bus ya tiene recursos dedicados, puede reducir la posibilidad de experimentar una limitación mediante el escalado vertical del número de unidades de mensajería asignadas al espacio de nombres en caso de (o en previsión de) un pico en la carga de trabajo.

El escalado o reducción vertical se puede lograr mediante la creación de [runbooks](../automation/automation-create-alert-triggered-runbook.md) que se pueden desencadenar mediante cambios en las métricas anteriores.

## <a name="faqs"></a>Preguntas más frecuentes

### <a name="how-does-throttling-affect-my-application"></a>¿Cómo afecta la limitación a mi aplicación?

Cuando una solicitud está limitada, eso implica que el servicio está ocupado porque está recibiendo más solicitudes de las que permiten los recursos. Si se vuelve a intentar la misma operación transcurridos unos minutos, una vez que el servicio haya finalizado las operaciones con la carga de trabajo actual, se podrá atender la solicitud.

Dado que la limitación es un comportamiento previsible en cualquier servicio nativo en la nube, hemos integrado la lógica de reintento en el propio SDK de Azure Service Bus. El valor predeterminado se establece en el reintento automático con retroceso exponencial para asegurarse de que no se limita siempre la misma solicitud.

La lógica de reintento predeterminada se aplicará a todas las operaciones.

### <a name="does-throttling-result-in-data-loss"></a>¿La limitación provoca la pérdida de datos?

Azure Service Bus está optimizado para la persistencia, por lo que se garantiza que todos los datos enviados a Service Bus se almacenan antes de que el servicio confirme la correcta finalización de la solicitud.

Una vez que Service Bus confirma correctamente la solicitud (se devuelve "ACK"), eso significa que la ha procesado correctamente. Si Service Bus devuelve "NACK" (error), eso significa que Service Bus no ha podido procesar la solicitud y que la aplicación cliente debe reintentar la solicitud.

Sin embargo, cuando se limita una solicitud, el servicio entiende que no puede aceptar y procesar la solicitud en este momento debido a las limitaciones de los recursos. Esto **no** conlleva ningún tipo de pérdida de datos porque Service Bus simplemente no ha examinado la solicitud. En este caso, confiar en la directiva de reintentos predeterminada del SDK de Service Bus garantiza que la solicitud se procesará realmente.

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos de cómo usar la mensajería de Service Bus, consulte estos temas avanzados:

* [Introducción a la mensajería de Service Bus](service-bus-messaging-overview.md)
* [Inicio rápido: Envío y recepción de mensajes mediante Azure Portal y .NET](service-bus-quickstart-portal.md)
* [Tutorial: Actualización del inventario mediante Azure Portal y temas y suscripciones](service-bus-tutorial-topics-subscriptions-portal.md)

