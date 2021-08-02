---
title: Detección de fugas de memoria - Detección inteligente de Azure Application Insights
description: Supervise las aplicaciones con Azure Application Insights para detectar posibles fugas de memoria.
ms.topic: conceptual
ms.date: 12/12/2017
ms.openlocfilehash: f2b13fc97baac2e8fcfb27475e6e018fdc0c5392
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111536623"
---
# <a name="memory-leak-detection-preview"></a>Detección de fugas de memoria (versión preliminar)

>[!NOTE]
>Puede migrar los recursos de Application Insights a la detección inteligente basada en alertas (versión preliminar). La migración crea reglas de alerta para los distintos módulos de detección inteligente. Una vez creadas, puede administrar y configurar dichas reglas como cualquier otra regla de Azure Monitor. También puede configurar grupos de acciones para estas reglas, lo que habilita varios métodos para realizar acciones o desencadenar notificaciones en nuevas detecciones.
>
> Para obtener más información, consulte el artículo sobre la [migración de alertas de detección inteligente](../alerts/alerts-smart-detections-migration.md).

La detección inteligente analiza automáticamente el consumo de memoria de cada uno de los procesos de la aplicación y le avisará sobre posibles fugas de memoria o de un aumento en el consumo de memoria.

Esta característica no requiere ninguna configuración especial, excepto la [configuración de los contadores de rendimiento](./performance-counters.md) de la aplicación. Se activa cuando la aplicación genera suficiente telemetría en los contadores de rendimiento de memoria (por ejemplo, bytes privados).

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>¿Cuándo recibiré este tipo de notificación de detección inteligente?
Una notificación típica suele generarse por un aumento uniforme del consumo de memoria durante un largo período de tiempo, en uno o más procesos o máquinas, que forman parte de la aplicación. Los algoritmos de aprendizaje automático se utilizan para detectar un consumo de memoria aumentado que coincida con un patrón de fuga de memoria.

## <a name="does-my-app-really-have-a-problem"></a>Entonces, ¿mi aplicación tiene realmente un problema?
Una notificación no significa que la aplicación tenga un problema. Aunque los patrones de fuga de memoria indican muchas veces un problema de la aplicación, estos patrones pueden ser típicos de su proceso específico o pueden tener una justificación comercial natural. En tal caso, puede ignorarse la notificación.

## <a name="how-do-i-fix-it"></a>¿Cómo puedo corregirlo?
Las notificaciones incluyen información de diagnóstico para facilitar el proceso de análisis de diagnóstico:
1. **Evaluación de prioridades**. La notificación muestra cuánto aumentó la memoria (en GB) y el intervalo de tiempo en el que se produjo ese aumento. Esta información puede ayudarle a asignar una prioridad al problema.
2. **Ámbito.** ¿Cuántas máquinas mostraron el patrón de fuga de memoria? ¿Cuántas excepciones se desencadenaron durante la posible fuga de memoria? Esta información puede obtenerse de la notificación.
3. **Diagnóstico.** La detección contiene el patrón de fuga de memoria, y muestra el consumo de memoria del proceso a lo largo del tiempo. También puede usar los elementos relacionados y los vínculos de informes para obtener información que puede ayudarle a efectuar un diagnóstico más exhaustivo del problema.
