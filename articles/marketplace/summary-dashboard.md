---
title: Panel Resumen para los análisis del Centro de partners en Marketplace comercial
description: Aprenda a acceder a gráficos, tendencias y valores de datos agregados que resumen la actividad de Marketplace desde el panel Resumen en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 8b76469697da62ed379d6854d2977e841080781d
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111539885"
---
# <a name="summary-dashboard-in-commercial-marketplace-analytics"></a>Panel Resumen de los análisis de marketplace comercial

En este artículo se proporciona información sobre el panel Resumen del Centro de partners. Este panel muestra gráficos, tendencias y valores de datos agregados que resumen la actividad de Marketplace para las ofertas.

Para acceder al panel de resumen en el Centro de partners, vaya a **Marketplace comercial** y seleccione **[Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Resumen**.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Terminología de análisis y preguntas comunes de Marketplace comercial](./analytics-faq.md).

## <a name="summary-dashboard"></a>Panel Resumen

El panel Resumen presenta información general del rendimiento comercial de las ofertas de Microsoft AppSource y de Azure Marketplace. El panel proporciona información general extensa de lo siguiente:

- Pedidos de los clientes
- Clientes
- Uso de las ofertas por parte de los clientes
- Visitas a las páginas de Azure Marketplace y AppSource por parte de los clientes

## <a name="elements-of-the-summary-dashboard"></a>Elementos del panel Resumen

En las secciones siguientes se describe cómo usar el panel Resumen y cómo leer los datos.

### <a name="month-range"></a>Intervalo mensual

Puede encontrar una selección del intervalo mensual en la esquina superior derecha de cada página. Puede personalizar salida de los gráficos de la página **Resumen** si selecciona un intervalo mensual basado en los últimos 3, 6 o 12 meses, o bien si selecciona un intervalo mensual personalizado con una duración máxima de 12 meses. El intervalo mensual predeterminado (período de cálculo) es de seis meses.

:::image type="content" source="./media/summary-dashboard/summary-dashboard.png" alt-text="Muestra las opciones de intervalo mensual en el panel de resumen.":::

> [!NOTE]
> Todas las métricas de los widgets de visualización y los informes de exportación respetan el período de cálculo seleccionado por el usuario.

### <a name="orders-widget"></a>Widget Pedidos

El widget Pedidos del panel Resumen** muestra los pedidos actuales para todas las ofertas basadas en transacciones. El widget Pedidos muestra un contador y la tendencia de todos los pedidos de compra (excepto los pedidos cancelados) correspondientes al período de cálculo seleccionado. El valor porcentual de **Pedidos** representa el crecimiento durante el período de cálculo seleccionado.

[![Muestra el widget Pedidos en el panel de resumen.](./media/summary-dashboard/orders-widget.png)](./media/summary-dashboard/orders-widget.png#lightbox)


Para ir al informe Pedidos, también puede seleccionar el vínculo **Panel Pedidos** en la esquina inferior izquierda del widget.

### <a name="customers-widget"></a>Widget Clientes

El widget **Clientes** del panel **Resumen** muestra la cantidad total de clientes que han adquirido sus ofertas para el período de cálculo seleccionado. El widget Clientes muestra un contador y la tendencia del número total de clientes activos (incluidos clientes nuevos y existentes, excepto los clientes perdidos) correspondientes al período de cálculo seleccionado. El valor porcentual debajo de **Clientes** representa el crecimiento durante el período de cálculo seleccionado.

[![Muestra el widget Clientes en el panel de resumen.](./media/summary-dashboard/customers-widget.png)](./media/summary-dashboard/customers-widget.png#lightbox)

Para ir al informe Clientes, también puede seleccionar el vínculo **Panel Clientes** en la esquina inferior izquierda del widget.

### <a name="usage-widget"></a>Widget Uso

El widget **Uso** del panel **Resumen** representa el total de horas de uso normalizadas y sin procesar para todas las ofertas de máquina virtual (VM) de Azure. El widget Uso muestra un contador y la tendencia del total de horas de uso para el período de cálculo seleccionado.

En la tabla de resumen de uso se muestran las horas de uso del cliente para todas las ofertas que haya comprado.

- Las horas de uso normalizadas se definen para contabilizar el número de núcleos de VM ([número de núcleos de VM] x [horas de uso sin procesar]). Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de máquina virtual].
- Las horas de uso sin procesar se definen como la cantidad de tiempo que las máquinas virtuales se han ejecutado en términos de horas.

El valor porcentual debajo del total de horas de uso representa el crecimiento de las horas de uso durante el período de cálculo seleccionado.

[![Muestra el widget Uso en el panel de resumen.](./media/summary-dashboard/usage-widget.png)](./media/summary-dashboard/usage-widget.png#lightbox)

Para ir al informe Uso, también puede seleccionar el vínculo **Panel Uso** en la esquina inferior izquierda del widget.

### <a name="marketplace-insights"></a>Información de Marketplace

Información de Marketplace muestra los usuarios que han visitado las páginas de las ofertas en Azure Marketplace y AppSource. El **Contador de visitas a las páginas** muestra un resumen de los análisis web de Marketplace comercial que permite a los editores medir la participación del cliente en sus respectivas páginas de detalles de productos en las tiendas en línea de Marketplace comercial: Microsoft AppSource y Azure Marketplace. Este widget muestra un contador y la tendencia del total de visitas a las páginas durante el período de cálculo seleccionado.

[![Muestra el widget Contador de visitas a las páginas en el panel de resumen.](./media/summary-dashboard/page-visit-count.png)](./media/summary-dashboard/page-visit-count.png#lightbox)

Para ir al informe Información de Marketplace, también puede seleccionar el vínculo **Panel Información de Marketplace** en la esquina inferior izquierda del widget.

### <a name="geographical-spread"></a>Diseminación geográfica

En el período de cálculo seleccionado, el mapa térmico muestra el número total de clientes, pedidos y horas de uso normalizadas con respecto a una dimensión geográfica.

:::image type="content" source="./media/summary-dashboard/geo-spread.png" alt-text="Muestra el widget Diseminación geográfica en el panel de resumen.":::

Tenga en cuenta lo siguiente:

- Puede mover el mapa para ver la ubicación exacta.
- Puede hacer zoom en una ubicación específica.
- El mapa térmico tiene una cuadrícula complementaria para ver los detalles del recuento de clientes, el recuento de pedidos y las horas de uso normalizadas para la ubicación específica.
- Puede buscar y seleccionar un país o región en la cuadrícula para ampliar la ubicación en el mapa. Vuelva a la vista original presionando el botón **Inicio** en el mapa.

> [!TIP]
> Puede usar el icono de descarga que está en la esquina superior derecha de cualquier widget para descargar los datos. Igualmente, para proporcionar comentarios sobre cada uno de los widgets, seleccione el icono "pulgar hacia arriba" o "pulgar hacia abajo".

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre los informes de análisis disponibles en marketplace comercial, consulte [Acceso a los informes de análisis de marketplace comercial en el Centro de partners](analytics.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](usage-dashboard.md).
- Para información detallada sobre los clientes, incluidas las tendencias de crecimiento, consulte [Panel Cliente de los análisis de marketplace comercial](customer-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](downloads-dashboard.md).
- Para una vista consolidada con los comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel de análisis de calificaciones y opiniones en el Centro de partners](ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./analytics-faq.md).