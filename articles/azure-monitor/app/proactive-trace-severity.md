---
title: Degradación en la relación de gravedad de seguimiento - Azure Application Insights
description: Supervise los seguimientos de las aplicaciones con Azure Application Insights para detectar patrones poco habituales de telemetría de seguimiento con detección inteligente.
ms.topic: conceptual
ms.date: 11/27/2017
ms.openlocfilehash: 776469f5e5cdab7796df1edccb680ab7151bbd18
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111951530"
---
# <a name="degradation-in-trace-severity-ratio-preview"></a>Degradación en la relación de gravedad de seguimiento (versión preliminar)

>[!NOTE]
>Puede migrar los recursos de Application Insights a la detección inteligente basada en alertas (versión preliminar). La migración crea reglas de alerta para los distintos módulos de detección inteligente. Una vez creadas, puede administrar y configurar dichas reglas como cualquier otra regla de Azure Monitor. También puede configurar grupos de acciones para estas reglas, lo que habilita varios métodos para realizar acciones o desencadenar notificaciones en nuevas detecciones.
>
> Para más información, vea el artículo sobre la [migración de alertas de detección inteligente](../alerts/alerts-smart-detections-migration.md).
> 

Los seguimientos se usan ampliamente en aplicaciones y ayudan a describir lo que sucede en segundo plano. Cuando algo va mal, los seguimientos proporcionan visibilidad fundamental en la secuencia de eventos que conducen al estado no deseado. Aunque, en su mayoría, los seguimientos no están estructurados, su nivel de gravedad todavía puede proporcionar información valiosa. En el estado estable de una aplicación, se podría esperar que la relación entre los seguimientos "correctos" (*Información* y *Detallado*) y los seguimientos "incorrectos" (*Advertencia*, *Error* y *Crítico*) permanecieran estables. 

Es normal esperar algún nivel de seguimientos "Incorrecto" por cualquier motivo, como problemas transitorios de red. Pero cuando un problema real comienza a crecer, normalmente se manifiesta como un aumento en la proporción relativa de los seguimientos "incorrectos" y "correctos". La detección inteligente analiza automáticamente la telemetría de seguimiento que registra la aplicación y puede advertirle sobre patrones inusuales en su gravedad.

Esta característica no requiere ninguna configuración especial, excepto la configuración del registro de seguimiento de la aplicación. Vea cómo configurar un agente de escucha de registro de seguimiento para [.NET](./asp-net-trace-logs.md) o [Java](./java-in-process-agent.md). Se activa cuando la aplicación genera suficientes datos de telemetría de seguimiento.

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>¿Cuándo recibiré este tipo de notificación de detección inteligente?
Obtendrá este tipo de notificación si la relación entre los seguimientos "correctos" (seguimientos iniciados con el nivel *Información* o *Detallado*) y los seguimientos "incorrectos" (seguimientos iniciados con el nivel *Advertencia*, *Error* o *Irrecuperable*) se degrada en un día determinado, en comparación con una base de referencia calculada durante los últimos siete días.

## <a name="does-my-app-definitely-have-a-problem"></a>Entonces, ¿mi aplicación tiene un problema?
Una notificación no significa que la aplicación tenga un problema. Aunque una degradación en la relación entre los seguimientos "correctos" e "incorrectos" podría indicar un problema en la aplicación, también puede ser favorable. Por ejemplo, el aumento puede deberse a un nuevo flujo en la aplicación que emite más seguimientos "incorrectos" que los flujos de existentes.

## <a name="how-do-i-fix-it"></a>¿Cómo puedo corregirlo?
Las notificaciones incluyen información de diagnóstico para facilitar el proceso de diagnóstico:
1. **Evaluación de prioridades**. La notificación muestra el número de operaciones afectadas. Esta información puede ayudarle a asignar una prioridad al problema.
2. **Ámbito.** ¿El problema afecta a todo el tráfico o solo a alguna operación? Esta información puede obtenerse de la notificación.
3. **Diagnóstico.** Puede usar los elementos relacionados y los vínculos de informes para obtener información que puede ayudarle a efectuar un diagnóstico más exhaustivo del problema.