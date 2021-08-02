---
title: 'Recepción de notificaciones sobre problemas mediante alertas: Azure IoT Edge'
description: Uso de reglas de alertas de Azure Monitor para la supervisión a gran escala
author: veyalla
manager: philmea
ms.author: veyalla
ms.date: 06/08/2021
ms.topic: conceptual
ms.reviewer: kgremban
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 14deb9a8a8ecaf67306ab8e29b2dfea7fa130c00
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111904577"
---
# <a name="get-notified-about-issues-using-alerts-preview"></a>Recepción de notificaciones sobre problemas mediante alertas (versión preliminar)

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Use [alertas de registro de Azure Monitor](../azure-monitor/alerts/alerts-unified-log.md) para supervisar los dispositivos IoT Edge a gran escala. Como se resalta en la [arquitectura de la solución](how-to-collect-and-transport-metrics.md#architecture), Log Analytics de Azure Monitor se usa como base de datos de métricas. Esta integración desbloquea funcionalidades de alerta eficaces y flexibles mediante alertas de registro centradas en los recursos.

## <a name="create-an-alert-rule"></a>Crear una regla de alerta

Puede [crear una regla de alertas de registro](../azure-monitor/alerts/alerts-log.md) para supervisar una amplia variedad de condiciones en toda la flota de dispositivos.

Se proporcionan consultas de alerta de [KQL](https://aka.ms/kql) de ejemplo en el recurso de IoT Hub. Las consultas que funcionan sobre datos de métricas de dispositivos perimetrales van precedidas de *IoT Edge* en su título. Use estos ejemplos tal y como están o modifíquelos según sea necesario para crear una consulta para su necesidad exacta.

Para acceder a las consultas de alerta de ejemplo, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.
1. Seleccione **Registros** en la sección **Supervisión** del menú.
1. Seleccione **Consultas** para abrir el explorador de consultas de ejemplo.

:::image type="content" source="./media/how-to-create-alerts/example-alerts.png" alt-text="Acceso a consultas de alerta de ejemplo." lightbox="./media/how-to-create-alerts/example-alerts.png":::

El [módulo metrics-collector](how-to-collect-and-transport-metrics.md#metrics-collector-module) ingiere todos los datos en la tabla [insightsMetrics](/azure/azure-monitor/reference/tables/insightsmetrics) estándar. Puede crear reglas de alertas basadas en datos de métricas de módulos personalizados consultando la misma tabla.

### <a name="split-by-device-dimension"></a>División por dimensión de dispositivo

Todas las reglas de alertas de ejemplo consultan los valores agregados por identificador de dispositivo. Esta agrupación es necesaria para determinar qué dispositivo provocó que se disparara la alerta. Puede seleccionar dispositivos específicos en los que habilitar la regla de alertas o habilitarla en todos los dispositivos. Use el gráfico de vista previa para explorar la tendencia por dispositivo antes de establecer la lógica de alerta.

### <a name="choose-notification-preferences"></a>Selección de las preferencias de notificación

Configure las preferencias de notificación en un [grupo de acciones](../azure-monitor/alerts/action-groups.md) y asócielo a una regla de alertas cuando cree una.

## <a name="select-alert-rule-scope"></a>Selección del ámbito de la regla de alertas

Con las instrucciones de la sección anterior se crea una regla de alertas dirigida a un único centro de IoT. Sin embargo, es posible que quiera crear la misma regla para varios centros de IoT. Cambie el ámbito a un grupo de recursos o a una suscripción completa para habilitar la regla de alertas en todos los centros dentro de ese ámbito.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.
1. Seleccione **Registros** en la sección **Supervisión** del menú.
1. Elija **Seleccionar ámbito** para cambiar el ámbito de una regla de alertas.

:::image type="content" source="./media/how-to-create-alerts/change-scope.png" alt-text="Cambio del ámbito de las alertas" lightbox="./media/how-to-create-alerts/change-scope.png":::

Agregue valores por el campo `_ResourceId` y elíjalo como *columna de Id. de recurso* al crear la regla de alertas. Este enfoque asociará una alerta con el recurso correcto para mayor comodidad.

## <a name="viewing-alerts"></a>Visualización de alertas

Consulte las alertas generadas para dispositivos en varios centros de IoT en la pestaña **Alertas** del [libro de vista de flotas de IoT Edge](how-to-explore-curated-visualizations.md#iot-edge-fleet-view-workbook).

Haga clic en el nombre de la regla de alertas para ver más contexto sobre la alerta. Al hacer clic en el vínculo de nombre del dispositivo, se mostrarán las métricas detalladas del dispositivo en torno al momento en que se disparó la alerta.

## <a name="next-steps"></a>Pasos siguientes

Mejore la solución de supervisión con [métricas de módulos personalizados](how-to-add-custom-metrics.md). 