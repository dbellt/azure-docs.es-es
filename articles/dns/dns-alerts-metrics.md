---
title: 'Métricas y alertas: Azure DNS'
description: Con esta ruta de aprendizaje, puede empezar a trabajar con métricas y alertas de Azure DNS.
services: dns
documentationcenter: na
author: rohinkoul
manager: kumudD
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/26/2021
ms.author: rohink
ms.openlocfilehash: f2eaac432673682b075763c6ce9fe9426ed77a70
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108017580"
---
# <a name="azure-dns-metrics-and-alerts"></a>Métricas y alertas de Azure DNS

Azure DNS es un servicio de hospedaje para dominios DNS que ofrece resolución de nombres mediante la infraestructura de Microsoft Azure. En este artículo se describen las métricas y alertas del servicio Azure DNS.

## <a name="azure-dns-metrics"></a>Métricas de Azure DNS

Azure DNS proporciona métricas para supervisar aspectos específicos de las zonas DNS. Con las métricas de Azure DNS, puede configurar las alertas en función de las condiciones que se cumplen. Las métricas proporcionadas usan el [servicio Azure Monitor](../azure-monitor/index.yml) para mostrar los datos. Para obtener más información sobre la experiencia y los gráficos del Explorador de métricas, vea [Explorador de métricas de Azure Monitor](../azure-monitor/essentials/metrics-charts.md). 
 
Azure DNS proporciona las siguientes métricas de las zonas DNS a Azure Monitor:

-   QueryVolume
-   RecordSetCount
-   RecordSetCapacityUtilization

Para más información, consulte la [definición de métricas](../azure-monitor/essentials/metrics-supported.md#microsoftnetworkdnszones).

>[!NOTE]
> En este momento, estas métricas solo están disponibles para las zonas DNS públicas hospedadas en Azure DNS. Si tiene zonas privadas hospedadas en Azure DNS, las métricas no proporcionarán datos sobre esas zonas. Además, la característica de métricas y alertas solo se admite en la nube pública de Azure. Más adelante se hablará de la compatibilidad con nubes soberanas. 

El elemento más específico del que puede ver las métricas es una zona DNS. Actualmente, no puede ver las métricas de registros de recursos individuales de una zona.

### <a name="query-volume"></a>Volumen de consultas

La métrica *Query Volume* (Volumen de consultas) muestra el número de consultas DNS que se reciben para la zona DNS en Azure DNS. La unidad de medida es `Count` y la agregación es el valor de `Sum` de todas las consultas que se reciben a lo largo de un período de tiempo.

Para ver esta métrica, seleccione la experiencia del explorador **Métricas** de la página **Supervisión** de Azure Portal. Limite el ámbito a su zona DNS y, luego, seleccione **Aplicar**. En la lista desplegable *Métricas*, seleccione `Query Volume` y, luego, elija `Sum` en la lista desplegable de *Agregación*.

![Volumen de consultas](./media/dns-alerts-metrics/dns-metrics-query-volume.png)

*Figura: Métrica de volumen de consultas de Azure DNS*

### <a name="record-set-count"></a>Recuento de conjunto de registros

La métrica *Record Set Count* (Número de conjuntos de registros) muestra el número de conjuntos de registros de Azure DNS para la zona DNS. Se cuentan todos los conjuntos de registros definidos en la zona. La unidad de medida es `Count` y la agregación es el valor `Maximum` de todos los conjuntos de registros.

Para ver esta métrica, seleccione la experiencia del explorador **Métricas** de la pestaña **Supervisión** de Azure Portal. Limite el ámbito a su zona DNS y, luego, seleccione **Aplicar**. En la lista desplegable *Métricas*, seleccione `Query Volume` y, luego, elija `Sum` en la lista desplegable de *Agregación*.

Seleccione la zona DNS de la lista desplegable **Recursos**, elija la métrica **Record Set Count** (Número de conjuntos de registros) y luego **Máx.** como **Agregación**. 

![Recuento de conjunto de registros](./media/dns-alerts-metrics/dns-metrics-record-set-count.png)

*Figura: Métricas de número de conjuntos de registros de Azure DNS* (Número de conjuntos de registros)

### <a name="record-set-capacity-utilization"></a>Uso de capacidad de conjunto de registros

La métrica *Record Set Capacity Utilization* (Uso de capacidad de conjunto de registros) muestra el porcentaje de uso de la capacidad de conjuntos de registros para una zona DNS. Cada zona de Azure DNS tiene un límite de conjunto de registros que define el número máximo de conjuntos de registros permitidos para la zona. Para más información, consulte la sección [Límites de DNS](dns-zones-records.md#limits). La unidad de medida es `Percentage` y el tipo de agregación es `Maximum`.

Por ejemplo, si tiene 500 conjuntos de registros configurados en la zona DNS y la zona tiene el límite de conjuntos de registros predeterminado de 5000, la métrica RecordSetCapacityUtilization mostrará el valor del 10 %, que se obtiene dividiendo 500 entre 5000. 

Para ver esta métrica, seleccione la experiencia del explorador **Métricas** de la pestaña **Supervisión** de Azure Portal. Limite el ámbito a su zona DNS y, luego, seleccione **Aplicar**. En la lista desplegable *Métricas*, seleccione `Record Set Capacity Utilization` y, luego, elija `Sum` en la lista desplegable de *Agregación*. 

![Captura de pantalla en la se muestra un ejemplo de cómo ver las métricas.](./media/dns-alerts-metrics/dns-metrics-record-set-capacity-uitlization.png)

*Figura: Métricas de uso de capacidad de conjunto de registros de Azure DNS*

## <a name="alerts-in-azure-dns"></a>Alertas de Azure DNS

Azure Monitor dispone de alertas que se pueden configurar para cada valor de métrica disponible. Para más información, consulte [Alertas de Azure Monitor](../azure-monitor/alerts/alerts-metric.md).

1. Para configurar las alertas de las zonas de Azure DNS, seleccione **Alertas** en la página *Supervisión* de Azure Portal. A continuación, haga clic en **+ Nueva regla de alertas**.

    :::image type="content" source="./media/dns-alerts-metrics/alert-metrics.png" alt-text="Captura de pantalla del botón Alerta de la página Supervisión.":::


1. Haga clic en el vínculo **Seleccionar recurso** en la sección Ámbito para abrir la página *Seleccionar un recurso*. Filtre por **Zonas DNS** y, luego, seleccione la zona de Azure DNS de destino que quiera como recurso de destino. Cuando haya elegido la zona, seleccione **Listo**.

    :::image type="content" source="./media/dns-alerts-metrics/select-resource.png" alt-text="Captura de pantalla de la página de selección de recursos en la configuración de alertas.":::

1. Luego, haga clic en el vínculo **Agregar condición** en la sección Condiciones para abrir la página *Select a signal* (Seleccionar una señal). Seleccione uno de los tres tipos de señal de *métrica* para los que quiera configurar la alerta.

    :::image type="content" source="./media/dns-alerts-metrics/select-signal.png" alt-text="Captura de pantalla de las métricas disponibles en la página Select a signal (Seleccionar una señal).":::

1. En la página *Configurar lógica de señal*, configure el umbral y la frecuencia de evaluación de la métrica seleccionada.

    :::image type="content" source="./media/dns-alerts-metrics/configure-signal-logic.png" alt-text="Captura de pantalla de la página Configurar lógica de señal.":::

1. Para enviar una notificación o invocar una acción desencadenada por la alerta, haga clic en **Add action groups** (Agregar grupos de acciones). En la página *Add action groups* (Agregar grupo de acciones), seleccione **+ Crear grupo de acciones**. Para más información, consulte [Grupo de acciones](../azure-monitor/alerts/action-groups.md).

1. Rellene el campo *Nombre de la regla de alertas* y, luego, elija **Crear regla de alertas** para guardar la configuración.

    :::image type="content" source="./media/dns-alerts-metrics/create-alert-rule.png" alt-text="Captura de pantalla de la página Crear regla de alertas.":::

Para obtener más información sobre cómo configurar alertas para métricas de Azure Monitor, vea [Creación, visualización y administración de alertas mediante Azure Monitor](../azure-monitor/alerts/alerts-metric.md). 

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [Azure DNS](dns-overview.md).
