---
title: Métricas de supervisión para Azure Front Door Estándar y Premium
description: En este artículo se describen las métricas de supervisión de Azure Front Door Estándar y Premium.
services: frontdoor
author: duongau
manager: KumudD
ms.service: frontdoor
ms.topic: how-to
ms.date: 02/18/2021
ms.author: yuajia
ms.openlocfilehash: a22e3825764f8b727802a1d78f2e02011cf7660a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110793531"
---
# <a name="real-time-monitoring-in-azure-front-door-standardpremium"></a>Supervisión en tiempo real en Azure Front Door Estándar y Premium

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

Azure Front Door Estándar y Premium se integra con Azure Monitor y tiene 11 métricas que ayudan a supervisar Azure Front Door Estándar y Premium en tiempo real para realizar un seguimiento de los problemas, así como para solucionarlos y depurarlos.  

Azure Front Door Estándar y Premium mide y envía sus métricas a intervalos de 60 segundos. Las métricas pueden tardar hasta 3 minutos en aparecer en el portal. Las métricas se pueden mostrar en los gráficos o en la cuadrícula que elija y son accesibles mediante el portal, PowerShell, la CLI y la API. Para más información, consulte  [Información general sobre las métricas en Microsoft Azure](../../azure-monitor/essentials/data-platform-metrics.md).  

Las métricas predeterminadas son gratuitas. Puede habilitar otras métricas con un costo adicional. 

Puede configurar alertas para cada métrica, como un umbral para 4XXErrorRate o 5XXErrorRate. Cuando la tasa de errores supere el umbral, se desencadenará una alerta tal y como se ha configurado. Para más información, vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../../azure-monitor/alerts/alerts-metric.md). 

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="metrics-supported-in-azure-front-door-standardpremium"></a>Métricas admitidas en Azure Front Door Estándar y Premium

| Métricas  | Descripción | Dimensions |
| ------------- | ------------- | ------------- |
| Proporción de aciertos de bytes | Porcentaje de salida de la memoria caché de AFD calculado con respecto a la salida total. </br> **Proporción de aciertos de bytes** = (salida desde el perímetro - salida desde el origen)/salida desde el perímetro. </br> **Escenarios excluidos en el cálculo de la proporción de aciertos de bytes**:</br> 1. No se configura ninguna caché explícitamente a través del motor de reglas o del comportamiento del almacenamiento en caché de cadenas de consulta. </br> 2. Se configura la directiva de control de caché con una caché privada o sin almacén. </br>3. La proporción de aciertos de bytes puede ser baja si la mayor parte del tráfico se reenvía al origen en lugar de servirse desde el almacenamiento en caché en función de sus configuraciones o escenarios. | Punto de conexión |
| RequestCount | El número de solicitudes de cliente que atiende CDN. | Punto de conexión, país del cliente, región del cliente, estado HTTP, grupo de estado HTTP |
| ResponseSize | Número de bytes enviados como respuestas de Front Door a los clientes. |Punto de conexión, país del cliente, región del cliente, estado HTTP, grupo de estado HTTP |
| TotalLatency | El tiempo total desde la solicitud de cliente recibida por CDN **hasta el último byte de respuesta enviado desde CDN al cliente**. |Punto de conexión, país del cliente, región del cliente, estado HTTP, grupo de estado HTTP |
| RequestSize | Número de bytes enviados como solicitudes de clientes a AFD. | Punto de conexión, país del cliente, región del cliente, estado HTTP, grupo de estado HTTP |
| 4XX % ErrorRate | Porcentaje de todas las solicitudes de cliente para las que el código de estado de la respuesta es 4XX. | Punto de conexión, país del cliente, región del cliente |
| 5XX % ErrorRate | Porcentaje de todas las solicitudes de cliente para las que el código de estado de la respuesta es 5XX. | Punto de conexión, país del cliente, región del cliente |
| OriginRequestCount  | Número de solicitudes enviadas desde AFD al origen. | Punto de conexión, origen, estado HTTP, grupo de estado HTTP |
| OriginLatency | Tiempo calculado desde que el perímetro de AFD envía la solicitud al servidor back-end hasta que AFD recibe el último byte de la respuesta del servidor back-end. | Punto de conexión, origen |
| OriginHealth% | Porcentaje de sondeos de estado correctos desde AFD al origen.| Origen, grupo de origen |
| Recuento de solicitudes de WAF | Solicitudes de WAF coincidentes. | Acción, nombre de regla, nombre de directiva |

## <a name="access-metrics-in-azure-portal"></a>Acceso a las métricas en Azure Portal

1. En el menú de Azure Portal, seleccione **Todos los recursos** >>  **\<your-AFD Standard/Premium (Preview) -profile>** .

2. En **Supervisión**, seleccione **Métricas**:

3. En **Métricas**, seleccione la métrica que desea agregar:

   :::image type="content" source="../media/how-to-monitoring-metrics/front-door-metrics-1.png" alt-text="Captura de pantalla de la página de métricas." lightbox="../media/how-to-monitoring-metrics/front-door-metrics-1-expanded.png":::

4. Seleccione **Agregar filtro** para agregar un filtro:

    :::image type="content" source="../media/how-to-monitoring-metrics/front-door-metrics-2.png" alt-text="Captura de pantalla de la adición de filtros a las métricas." lightbox="../media/how-to-monitoring-metrics/front-door-metrics-2-expanded.png":::
    
5. Seleccione **Apply splitting** (Aplicar división) para dividir los datos según las distintas dimensiones:

   :::image type="content" source="../media/how-to-monitoring-metrics/front-door-metrics-4.png" alt-text="Captura de pantalla de la adición de dimensiones a las métricas." lightbox="../media/how-to-monitoring-metrics/front-door-metrics-4-expanded.png":::

6. Seleccione **Nuevo gráfico** para agregar un nuevo gráfico:

## <a name="configure-alerts-in-azure-portal"></a>Configuración de alertas en Azure Portal

1. Para configurar las alertas de Azure Front Door Estándar y Premium (versión preliminar), seleccione **Supervisión** >> **Alertas**.

1. Seleccione **Nueva regla de alertas** para las métricas que aparecen en la sección Métricas.

La alerta se cobrará en función de Azure Monitor. Para más información sobre alertas, consulte [Alertas de Azure Monitor](../../azure-monitor/alerts/alerts-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Informes de Azure Front Door Estándar y Premium (versión preliminar)](how-to-reports.md).
- Más información sobre [Registro de Azure Front Door Estándar y Premium (versión preliminar)](how-to-logs.md).
