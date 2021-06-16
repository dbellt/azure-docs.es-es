---
title: Envío de comentarios sobre anomalías al servicio Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo enviar comentarios sobre las anomalías que encuentre la instancia de Metrics Advisor y ajustar los resultados.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: mbullwin
ms.openlocfilehash: 2d63457bddbe1758a172abef4073419a218acf75
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111892605"
---
# <a name="provide-anomaly-feedback"></a>Envío de comentarios sobre anomalías

Los comentarios de los usuarios son uno de los métodos más importantes para detectar defectos en el sistema de detección de anomalías. Aquí proporcionamos a los usuarios una manera de marcar los resultados de detección incorrectos directamente en una serie temporal y aplicar los comentarios de forma inmediata. De esta manera, cualquier usuario puede enseñar al sistema de detección de anomalías cómo realizar la detección de anomalías para una serie temporal específica a través de interacciones activas. 

> [!NOTE]
> Actualmente, los comentarios solo afectarán a los resultados de la detección de anomalías por **Detección inteligente**, pero no **Umbral saturado** y **Cambiar umbral**.

## <a name="how-to-give-time-series-feedback"></a>Proporcionar comentarios de series temporales

Puede proporcionar comentarios de la página de detalles de la métrica en cualquier serie. Seleccione cualquier punto y verá el siguiente cuadro de diálogo de comentarios. Muestra las dimensiones de la serie que ha elegido. Puede volver a seleccionar valores de dimensión o incluso quitar algunos de ellos para obtener un lote de datos de series temporales. Después de elegir una serie temporal, seleccione el botón **Agregar** para agregar los comentarios. Hay cuatro tipos de comentarios que se pueden proporcionar. Para anexar varios elementos de los comentarios, seleccione el botón **Guardar** tras completar las anotaciones.

:::image type="content" source="../media/feedback/click-on-any-point.png" alt-text="Grafo con datos de una serie temporal con una línea azul y puntos rojos en varios lugares. Cuadro rojo que rodea un punto con el texto: Seleccione cualquier punto":::

:::image type="content" source="../media/feedback/select-or-remove.png" alt-text="Cuadro de diálogo para agregar comentarios con dos dimensiones y la opción para seleccionar o quitar dimensiones y agregar comentarios.":::

### <a name="mark-the-anomaly-point-type"></a>Marca del tipo de punto de la anomalía

Como se muestra en la imagen siguiente, el cuadro de diálogo de comentarios rellenará automáticamente la marca de tiempo del punto elegido automáticamente, aunque este valor se puede editar. A continuación, seleccione si desea identificar este elemento como `Anomaly`, `NotAnomaly` o `AutoDetect`.

:::image type="content" source="../media/feedback/anomaly-value.png" alt-text="Menú desplegable con las opciones Anomaly, NotAnomaly y AutoDetect":::

La selección aplicará sus comentarios al futuro procesamiento de la detección de anomalías de la misma serie. Los puntos procesados no se volverán a calcular, lo que significa que si marcó una anomalía como NotAnomaly, se suprimirán las anomalías similares en el futuro y si marcó un punto de `NotAnomaly` como `Anomaly`, en el futuro la tendencia será que se detecten puntos similares como `Anomaly`. Si se elige `AutoDetect`, todos los comentarios anteriores que haya en el mismo punto se omitirán en el futuro.

## <a name="provide-feedback-for-multiple-continuous-points"></a>Envío de comentarios en varios puntos continuos 

Si desea proporcionar comentarios sobre anomalías en varios puntos continuos al mismo tiempo, seleccione el grupo de puntos que desea anotar. Cuando especifique los comentarios sobre anomalías verá que el intervalo de tiempo elegido se rellenará automáticamente.

:::image type="content" source="../media/feedback/continuous-anomaly-feedback.png" alt-text="Menú de comentarios sobre anomalías con una anomalía seleccionada y un intervalo de tiempo específico":::

Para ver si los comentarios sobre anomalías afectan a algún punto individual, cuando examine una serie temporal seleccione uno de sus puntos. Si los comentarios han cambiado el resultado de su detección de anomalías, la información sobre herramientas mostrará **Affected by feedback: true** (Afectado por comentarios: true). Si muestra **Affected by feedback: false** (Afectado por comentarios: false), significa que se ha realizado algún cálculo de comentarios sobre anomalías para ese punto, pero que la detección de anomalías no se debe cambiar.

:::image type="content" source="../media/feedback/affected-by-feedback.png" alt-text="Pantalla de información sobre herramientas con las palabras: Affected by feedback: true (Afectado por comentarios: true), resaltadas en rojo":::

Hay algunas situaciones en las que no se recomienda aportar comentarios:

- La anomalía la ha causado alguna festividad. Para resolver este tipo de falsa alarma se recomienda usar un evento preestablecido, ya que será más preciso.
- La anomalía la provoca un cambio en un origen de datos conocido. Por ejemplo, en un momento dado se ha producido un cambio en el sistema ascendente. En esta situación, se espera que se produzca una alerta por alguna anomalía, ya que el sistema no sabe qué ha provocado el cambio de valor y cuándo se volverán a producir cambios de valor similares. Por tanto, no es aconsejable anotar este tipo de problema como `NotAnomaly`.

## <a name="change-points"></a>Cambio de puntos

A veces, el cambio de tendencia de los datos afectará a los resultados de la detección de anomalías. Cuando se tome la decisión de si un punto es una anomalía o no, se tendrá en cuenta la última ventana de datos del historial. Cuando la serie temporal tiene un cambio de tendencia, puede marcar el punto exacto del cambio, lo que ayudará a nuestro detector de anomalías en el análisis futuro.

Como se muestra en la siguiente ilustración, puede seleccionar `ChangePoint` como tipo de comentario y `ChangePoint`, `NotChangePoint` o `AutoDetect` en la lista desplegable.

:::image type="content" source="../media/feedback/changepoint.png" alt-text="Menú de punto de cambio con una lista desplegable que contiene las opciones de ChangePoint, NotChangePoint y AutoDetect":::

> [!NOTE]
> Si los datos siguen cambiando, solo tendrá que marcar un punto como `ChangePoint`, por lo que si marcó un `timerange`, se rellenará automáticamente la marca de tiempo y la hora del último punto. En este caso, la anotación solo afectará a los resultados de detección de anomalías después de 12 puntos.

## <a name="seasonality"></a>Estacionalidad

En el caso de los datos estacionales, cuando se realiza la detección de anomalías, uno de los pasos es calcular el período (estacionalidad) de la serie temporal y aplicarlo a la fase de detección de anomalías. A veces, es difícil identificar un período preciso y el período también puede cambiar. Un período definido incorrectamente puede tener efectos secundarios en los resultados de la detección de anomalías. El período actual se puede encontrar en una información sobre herramientas, su nombre es `Min Period`.

:::image type="content" source="../media/feedback/min-period.png" alt-text="Superposición de la información sobre herramientas con la palabra Period (Periodo) y el número siete resaltados en rojo.":::

Puede proporcionar comentarios sobre el período para corregir este tipo de error de detección de anomalías. Como muestra la ilustración, puede establecer un valor de período. La unidad `interval` significa una granularidad. Aquí, cero intervalos significa que los datos no son estacionales. También puede seleccionar `AutoDetect` si desea cancelar los comentarios anteriores y dejar que la canalización detecte el período de forma automática. 
 
> [!NOTE]
> Al establecer el período no es necesario asignar una marca de tiempo o un intervalo, el período afectará a la futura detección de anomalías en toda la serie temporal desde el momento en el que envíe comentarios.


:::image type="content" source="../media/feedback/period-feedback.png" alt-text="Menú con el periodo de detección automática establecido en 28 y el intervalo establecido en 0, lo que indica que no es estacional.":::

## <a name="provide-comment-feedback"></a>Aportación de comentarios sobre el contenido

También puede agregar comentarios para anotar y proporcionar contexto a los datos. Para agregar comentarios, seleccione un intervalo de tiempo y agregue el texto del comentario.

:::image type="content" source="../media/feedback/feedback-comment.png" alt-text="Menú con la posibilidad de establecer un intervalo de tiempo y un cuadro para agregar un comentario de texto":::

## <a name="time-series-batch-feedback"></a>Comentarios por lotes de series temporales

Como ya se ha descrito, el modal de comentarios permite volver a seleccionar o quitar valores de dimensión, con el fin de obtener un lote de series temporales definidas mediante un filtro de dimensión. Este modal también se puede abrir haciendo clic en el botón "+" para obtener comentarios en el panel izquierdo y seleccionar las dimensiones y los valores de dimensión.

:::image type="content" source="../media/feedback/feedback-time-series.png" alt-text="Menú con un signo más (+) azul resaltado en rojo junto a la palabra Feedback (Comentarios)":::

:::image type="content" source="../media/feedback/feedback-dimension.png" alt-text="Agregar el menú de comentarios con dos dimensiones, indicadas por Dim1 y Dim2":::

## <a name="how-to-view-feedback-history"></a>Visualización del historial de comentarios

Hay dos formas de ver el historial de comentarios. Puede seleccionar el botón de historial de comentarios en el panel izquierdo para ver un modal de la lista de comentarios. Enumera todos los comentarios que se han proporcionado antes para los filtros de una sola serie o de una dimensión.

:::image type="content" source="../media/feedback/feedback-history-options.png" alt-text="Menú de lista de comentarios`":::

Otra forma de ver el historial de comentarios es desde una serie. Verá varios botones en la esquina superior derecha de cada serie. Seleccione el botón de mostrar comentarios y la línea pasará de mostrar puntos de anomalías a entradas de comentarios. La marca verde representa un punto de cambio y los puntos azules son otros puntos de comentarios. También puede seleccionarlos para obtener un modal de la lista de comentarios que enumera los detalles de los comentarios proporcionados para este punto.

:::image type="content" source="../media/feedback/feedback-history-graph.png" alt-text="Grafo del historial de comentarios":::

:::image type="content" source="../media/feedback/feedback-list.png" alt-text="Menú de lista de comentarios con dos dimensiones":::

> [!NOTE]
> Cualquier persona que tenga acceso a la métrica tiene permiso para proporcionar comentarios, por lo que se pueden ver los comentarios proporcionados por otros propietarios de la fuente de alimentación. Si edita el mismo punto que otra persona, sus comentarios sobrescribirán la entrada anterior.       

## <a name="next-steps"></a>Pasos siguientes
- [Diagnóstico de incidentes](diagnose-incident.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)
- [Configuración de alertas y obtención de notificaciones mediante enlaces](../how-tos/alerts.md)