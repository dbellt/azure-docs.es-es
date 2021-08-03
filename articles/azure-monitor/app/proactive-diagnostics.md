---
title: Detección inteligente en Azure Application Insights | Microsoft Docs
description: Application Insights realiza un análisis profundo automático de la telemetría de la aplicación y le advierte de los posibles problemas.
ms.topic: conceptual
ms.date: 02/07/2019
ms.openlocfilehash: 79fe723a7972d265621a1a52ea589bf7dcf6ec62
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111536735"
---
# <a name="smart-detection-in-application-insights"></a>Detección inteligente en Application Insights

>[!NOTE]
>Puede migrar la detección inteligente del recurso de Application Insights para que se base en alertas. La migración crea reglas de alerta para los distintos módulos de detección inteligente. Una vez creadas, puede administrar y configurar dichas reglas como cualquier otra regla de Azure Monitor. También puede configurar grupos de acciones para estas reglas, lo que habilita varios métodos para realizar acciones o desencadenar notificaciones en nuevas detecciones.
>
> Para obtener más información, consulte el artículo sobre la [migración de alertas de detección inteligente](../alerts/alerts-smart-detections-migration.md).

La detección inteligente avisa automáticamente de posibles problemas de rendimiento y anomalías de error en su aplicación web. Realiza un análisis proactivo de la telemetría que su aplicación envía a [Application Insights](./app-insights-overview.md). Si hay un aumento repentino de las tasas de error o patrones de rendimiento anormales en el cliente o el servidor, recibirá una alerta. Esta característica no necesita ninguna configuración. Funciona si la aplicación envía suficiente telemetría.

Puede acceder a las detecciones emitidas por la detección inteligente tanto desde los correos electrónicos que recibe, como desde la hoja de detección inteligente.

## <a name="review-your-smart-detections"></a>Revisión de las detecciones inteligentes
Puede detectar las detecciones de dos maneras:

* **Recibirá un correo electrónico** de Application Insights. Este es un ejemplo típico:
  
    ![Alerta de correo electrónico](./media/proactive-diagnostics/03.png)
  
    Haga clic en el botón grande para abrir más detalles en el portal.
* **La hoja de detección inteligente** en Application Insights. Seleccione **Detección inteligente** en el menú **Investigar** para ver una lista de las detecciones recientes.

![Visualización de detecciones recientes](./media/proactive-diagnostics/04.png)

Seleccione una detección para ver sus detalles.

## <a name="what-problems-are-detected"></a>¿Qué problemas se detectan?

La detección inteligente detecta y notifica diversos problemas, como:

* [Detección inteligente: anomalías de errores](./proactive-failure-diagnostics.md). Usamos el aprendizaje automático para establecer la tasa esperada de solicitudes con error para su aplicación, que se correlaciona con la carga y otros factores. Notifica si la tasa de error queda fuera del rango esperado.
* [Detección inteligente: anomalías de rendimiento](./proactive-performance-diagnostics.md). Notifica si el tiempo de respuesta de una operación o la duración de la dependencia se está ralentizando, en comparación con la base de referencia histórica. También notifica si se identifica algún patrón anómalo en el tiempo de respuesta o en el tiempo de carga de la página.   
* Problemas y degradaciones generales, como [Degradación de seguimiento](./proactive-trace-severity.md), [Fuga de memoria](./proactive-potential-memory-leak.md), [Aumento anómalo del volumen de excepciones](./proactive-exception-volume.md) y [Antipatrones de seguridad](./proactive-application-security-detection-pack.md).

(Los vínculos a la Ayuda de cada notificación llevan a los artículos pertinentes).

## <a name="smart-detection-email-notifications"></a>Notificaciones por correo electrónico de detección inteligente

Todas las reglas de detección inteligente, excepto las marcadas como _versión preliminar_, se configuran de forma predeterminada para enviar notificaciones por correo electrónico cuando se encuentran las detecciones.

La configuración de las notificaciones por correo electrónico para una regla de detección inteligente específica puede hacerse si se abre la hoja **Configuración** de detección inteligente y se selecciona la regla, lo que abrirá la hoja **Editar regla**.

Como alternativa, puede cambiar la configuración mediante plantillas de Azure Resource Manager. Para obtener más información, consulte [Administración de reglas de detección inteligente de Application Insights con plantillas de Azure Resource Manager](./proactive-arm-config.md).

## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]



## <a name="next-steps"></a>Pasos siguientes
Estas herramientas de diagnóstico lo ayudarán a inspeccionar los datos de telemetría de su aplicación:

* [Explorador de métricas](../essentials/metrics-charts.md)
* [Explorador de búsqueda](./diagnostic-search.md)
* [Analytics: Lenguaje de consulta eficaz](../logs/log-analytics-tutorial.md)

La detección inteligente es automática. Pero ¿quizás le gustaría configurar algunas alertas más?

* [Alertas de métricas configuradas manualmente](../alerts/alerts-log.md)
* [Pruebas web de disponibilidad](./monitor-web-app-availability.md)
