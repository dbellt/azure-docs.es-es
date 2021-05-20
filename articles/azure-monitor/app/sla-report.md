---
title: 'Libro sobre tiempo de inactividad, Acuerdo de Nivel de Servicio e interrupciones: Application Insights'
description: Calcule e informe sobre el Acuerdo de Nivel de Servicio para la prueba web a través de un panel único para todos los recursos de Application Insights y las suscripciones a Azure.
ms.topic: conceptual
ms.date: 05/4/2021
ms.openlocfilehash: 4063cd28111dd0ea3c3d0cabe44f884f6605c063
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108750549"
---
# <a name="downtime-sla-and-outages-workbook"></a>Libro sobre tiempo de inactividad, Acuerdo de Nivel de Servicio e interrupciones

Presentamos una manera simple de calcular e informar sobre el Acuerdo de Nivel de Servicio para las pruebas web a través de un panel único para todos los recursos de Application Insights y las suscripciones a Azure. El informe Tiempo de inactividad e interrupciones proporciona eficaces consultas y visualizaciones de datos pregeneradas para mejorar su comprensión de la conectividad del cliente, el tiempo de respuesta típico de la aplicación y el tiempo de inactividad ocurrido.

Puede acceder a la plantilla del libro de Acuerdo de Nivel de Servicio a través de la galería de libros en el recurso de Application Insights o a través de la pestaña Disponibilidad si selecciona **SLA Reports** (informes de Acuerdo de Nivel de Servicio) en la parte superior.
:::image type="content" source="./media/sla-report/availability.png" alt-text="Captura de pantalla de la pestaña Disponibilidad con la opción SLA Reports (informes de Acuerdo de Nivel de Servicio) resaltada." lightbox="./media/sla-report/availability.png":::

:::image type="content" source="./media/sla-report/workbook-gallery.png" alt-text="Captura de pantalla de la galería de libros con el libro Tiempo de inactividad e interrupciones resaltado." lightbox ="./media/sla-report/workbook-gallery.png":::

## <a name="parameter-flexibility"></a>Flexibilidad de parámetros

Los parámetros establecidos en el libro influyen en el resto del informe.

:::image type="content" source="./media/sla-report/parameters.png" alt-text=" Captura de pantalla de parámetros." lightbox= "./media/sla-report/parameters.png":::

Los parámetros `Subscriptions`,  `App Insights Resources` y  `Web Test`  determinan las opciones de recursos generales disponibles. Estos parámetros se basan en consultas de Log Analytics y se usan en todas las consultas de informe.

`Failure Threshold` y `Outage Window`  permiten definir criterios propios para una interrupción del servicio; por ejemplo, los criterios de una alerta de disponibilidad de App Insights según el contador de ubicación con errores en un período seleccionado. El umbral típico es de tres ubicaciones en un período de cinco minutos.

`Maintenance Period`  permite seleccionar la frecuencia de mantenimiento normal y  `Maintenance Window`  es un selector de fecha y hora para un período de mantenimiento de ejemplo. Todos los datos que se produzcan durante el período identificado se omitirán en los resultados.

`Availability Target %`  especifica el objetivo de destino y los valores personalizados.

## <a name="overview-page"></a>Página de información general

La página de información general contiene información generalizada sobre el Acuerdo de Nivel de Servicio total (excluidos los períodos de mantenimiento, si están definidos), las instancias de interrupción de un extremo a otro y el tiempo de inactividad de la aplicación. Las instancias de interrupción están definida entre cuando una prueba comienza a devolver errores hasta que se realiza correctamente, en función de los parámetros de interrupción. Si se produce un error en una prueba a las 8:00 h y se realiza de nuevo a las 10:00 h, todo el período de datos se considera como la misma interrupción.

:::image type="content" source="./media/sla-report/overview.gif" alt-text=" GIF de la página de información general que muestra la tabla de información general ordenada por pruebas." lightbox="./media/sla-report/overview.gif":::

También puede investigar la interrupción más larga que se produjo durante el período de informe.

Algunas pruebas se pueden vincular al recurso de Application Insights correspondiente para una mejor inspección, pero esto solo es posible en el [recurso de Application Insights basado en áreas de trabajo](create-workspace-resource.md).

## <a name="downtime-outages-and-failures"></a>Tiempo de inactividad, interrupciones y errores

La pestaña **Interrupciones y tiempo de inactividad** tiene información sobre las instancias de interrupción y el tiempo de inactividad totales desglosados por prueba. La pestaña **Errores por ubicación** incluye un mapa geográfico de ubicaciones de prueba con errores para ayudarle a identificar las posibles áreas de conexión problemáticas.

:::image type="content" source="./media/sla-report/outages-failures.gif" alt-text=" GIF de la pestaña Interrupciones y tiempo de inactividad, y de la pestaña Errores por ubicación en el libro Tiempo de inactividad e interrupciones." lightbox="./media/sla-report/outages-failures.gif":::

## <a name="edit-the-report"></a>Editar el informe

Puede editar el informe como cualquier otro [libro de Azure Monitor](../visualize/workbooks-overview.md). Puede personalizar las consultas o visualizaciones en función de las necesidades de su equipo.

:::image type="content" source="./media/sla-report/edit.gif" alt-text=" GIF que muestra la selección del botón Editar para cambiar la visualización a un gráfico circular." lightbox="./media/sla-report/edit.gif":::

### <a name="log-analytics"></a>Log Analytics

Todas las consultas se pueden ejecutar en [Log Analytics](../logs/log-analytics-overview.md) y usarse en otros informes o paneles. Quite la restricción de parámetros y reutilice la consulta principal.

:::image type="content" source="./media/sla-report/logs.gif" alt-text=" GIF de la consulta de registros." lightbox="./media/sla-report/logs.gif":::

## <a name="access-and-sharing"></a>Acceso y uso compartido

El informe se puede compartir con sus equipos, con la dirección o anclarse a un panel para su uso posterior. El usuario debe tener permisos de lectura o acceso al recurso de Application Insights en el que se almacena el libro real.

:::image type="content" source="./media/sla-report/share.png" alt-text=" Captura de pantalla de Compartir esta plantilla." lightbox= "./media/sla-report/share.png":::

## <a name="next-steps"></a>Pasos siguientes

- [Sugerencias para la optimización de consultas de Log Analytics](../logs/query-optimization.md).
- Aprenda a [crear un gráfico en los libros](../visualize/workbooks-chart-visualizations.md).
- Obtenga información sobre cómo supervisar su sitio web con las [pruebas de disponibilidad](monitor-web-app-availability.md).