---
title: Degradación en la relación de gravedad de seguimiento - Azure Application Insights
description: Supervise los seguimientos de las aplicaciones con Azure Application Insights para detectar patrones poco habituales de telemetría de seguimiento con Detección inteligente.
ms.topic: conceptual
ms.date: 11/27/2017
ms.openlocfilehash: 8432ce53a4d630839a62d29833e3863fff4e8afa
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110072431"
---
# <a name="degradation-in-trace-severity-ratio-preview"></a>Degradación en la relación de gravedad de seguimiento (versión preliminar)

Los seguimientos se usan ampliamente en aplicaciones, ya que ayudan a describir lo que sucede en segundo plano. Cuando algo va mal, los seguimientos proporcionan visibilidad fundamental en la secuencia de eventos que conducen al estado no deseado. Aunque los seguimientos no están estructurados generalmente, hay algo que puede aprender de ellos en concreto: su nivel de gravedad. En el estado estable de una aplicación, se podría esperar que la relación entre los seguimientos "correctos" (*Información* y *Detallado*) y los seguimientos "incorrectos" (*Advertencia*, *Error* y *Crítico*) permanecieran estables. La suposición es que los seguimientos "incorrectos" pueden reflejarse de forma periódica hasta cierto punto debido a una serie de motivos (por ejemplo, problemas de red transitorios). Pero cuando un problema real comienza a crecer, normalmente se manifiesta como un aumento en la proporción relativa de los seguimientos "incorrectos" y "correctos". La detección inteligente de Application Insights analiza automáticamente los seguimientos registrados por la aplicación y puede avisarle sobre los patrones poco habituales en la gravedad de la telemetría de seguimiento.

Esta característica no requiere ninguna configuración especial, que no sea la configuración del registro de seguimiento de la aplicación (consulte cómo configurar un agente de escucha de registro de seguimiento para [.NET](./asp-net-trace-logs.md) o [Java](java-2x-trace-logs.md)). Se activará cuando la aplicación genere suficiente telemetría de excepciones.

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>¿Cuándo recibiré este tipo de notificación de detección inteligente?
Es posible que obtenga este tipo de notificación si la relación entre los seguimientos “correctos” (seguimientos iniciados con el nivel *Información* o *Detallado*) y los seguimientos “incorrectos” (seguimientos iniciados con el nivel *Advertencia*, *Error* o *Irrecuperable*) se degrada en un día determinado, en comparación con una base de referencia calculada durante los últimos siete días.

## <a name="does-my-app-definitely-have-a-problem"></a>Entonces, ¿mi aplicación tiene un problema?
No, una notificación no significa que la aplicación tenga un problema. Aunque una degradación en la relación entre los seguimientos "correctos" e "incorrectos" podría indicar un problema de aplicación, este cambio en la relación también puede ser benigno. Por ejemplo, el aumento podría ser debido a un nuevo flujo en la aplicación que emita más seguimientos "incorrectos" que los flujos de existentes.

## <a name="how-do-i-fix-it"></a>¿Cómo puedo corregirlo?
Las notificaciones incluyen información de diagnóstico para facilitar el proceso de diagnóstico:
1. **Evaluación de prioridades**. La notificación muestra el número de operaciones afectadas. Esto puede ayudarlo a asignar una prioridad al problema.
2. **Ámbito.** ¿El problema afecta a todo el tráfico o solo a alguna operación? Esta información puede obtenerse de la notificación.
3. **Diagnóstico.** Puede usar los elementos relacionados y los vínculos de informes para obtener información que puede ayudarle a efectuar un diagnóstico más exhaustivo del problema.
