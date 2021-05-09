---
title: Contracargo de costos de Reserva de Azure
description: Aprenda a ver los costos de Reserva de Azure para el contracargo.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 03/10/2021
ms.author: banders
ms.openlocfilehash: 4059318e6b8052f3b0221c87e8a357cfc8679e44
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107532758"
---
# <a name="charge-back-azure-reservation-costs"></a>Contracargo de costos de Reserva de Azure

Los lectores de facturación de Contrato Enterprise y Contrato de cliente de Microsoft pueden ver los datos de los costos amortizados para las reservas. Pueden utilizar los datos de costo para realizar un contracargo del valor monetario de una suscripción, un grupo de recursos, un recurso o una etiqueta a sus asociados. En los datos amortizados, el precio efectivo es el costo de la reserva por hora prorrateada. El costo es el costo total de uso de la reserva por parte del recurso en ese día.

Los usuarios con una suscripción individual pueden obtener los datos de costos amortizados en su archivo de uso. Cuando un recurso obtiene un descuento por reserva, la sección *AdditionalInfo* del archivo de uso contiene los detalles de la reserva. Para más información, consulte [Descarga de uso en Azure Portal](../understand/download-azure-daily-usage.md#download-usage-from-the-azure-portal-csv).

## <a name="see-reservation-usage-data-for-show-back-and-charge-back"></a>Consulte los datos de uso de la reserva para mostrar y realizar un contracargo de costos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a **Administración de costos + facturación**. 
3. Seleccione **Análisis de costos** en el panel de navegación izquierdo. 
4. En **Costo real**, seleccione la métrica **Costo amortizado**.
5. Para ver qué recursos ha usado una reserva, aplique un filtro para **Reserva** y, después, seleccione reservas.
6. Establezca la **Granularidad** en **Mensual** o **Diaria**.
7. Establezca el tipo de gráfico en **Tablas**.
8. En la opción **Agrupar por**, seleccione **Recurso**.

[![Ejemplo que muestra los costos de recursos de reserva que se pueden usar para el contracargo](./media/charge-back-usage/amortized-reservation-costs.png)](./media/charge-back-usage/amortized-reservation-costs.png#lightbox)

Este es un vídeo en el que se muestra cómo ver los costos de uso de reservas en los niveles de suscripción, grupo de recursos y recurso en Azure Portal.

 > [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4sQOw] 

## <a name="get-the-data-for-show-back-and-charge-back"></a>Obtención de los datos para mostrarlos en el fondo y el contracargo
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a **Administración de costos + facturación**. 
3. Haga clic en **Exportar** en el panel de navegación izquierdo. 
4. Haga clic en el botón **Agregar**.
5. Seleccione Costo amortizado como botón de métrica y configure la exportación.

El precio efectivo del uso que obtiene el descuento de la reserva es el costo prorrateado de la reserva (en lugar de ser cero). Esto le ayuda a conocer el valor monetario del consumo de reserva de una suscripción, un grupo de recursos o un recurso, y puede ayudarle realizar cargos al usuario por el uso de la reserva internamente. El conjunto de datos también tiene horas de reserva sin usar. 

## <a name="get-azure-consumption-and-reservation-usage-data-using-api"></a>Obtención de datos de uso de reserva y consumo de Azure mediante API

Puede obtener los datos mediante la API o descargarlos desde Azure Portal.

Para obtener los datos nuevos, llame a [Usage Details API](/rest/api/consumption/usagedetails/list). Para obtener más información acerca de la terminología, consulte los [términos de uso](../understand/understand-usage.md).

Este es un ejemplo de llamada a Usage Details API:

```
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{enrollmentId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodId}/providers/Microsoft.Consumption/usagedetails?metric={metric}&amp;api-version=2019-05-01&amp;$filter={filter}
```

Para obtener más información acerca de {enrollmentId} y {billingPeriodId}, vea el artículo de la API [Usage Details – List](/rest/api/consumption/usagedetails/list) (Detalles de uso: lista).

La información de la tabla siguiente sobre la métrica y los filtros puede ayudar a resolver problemas comunes de reserva.

| **Tipo de datos de API** | Acción de llamada a la API |
| --- | --- |
| **Todos los cargos (uso y compras)** | Reemplace {metric} con ActualCost |
| **Uso que obtuvo el descuento de reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/reservationId%20ne%20 |
| **Uso que no obtuvo el descuento de reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/reservationId%20eq%20 |
| **Cargos amortizados (uso y compras)** | Reemplace {metric} con AmortizedCost |
| **Informe de reserva sin usar** | Reemplace {metric} con AmortizedCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'UnusedReservation' |
| **Compras de la reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'Purchase'  |
| **Reembolsos** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'Refund' |

## <a name="download-the-usage-csv-file-with-new-data"></a>Descarga del archivo CSV de uso con los nuevos datos

Si es administrador de EA, puede descargar el archivo CSV que contiene los nuevos datos de uso de Azure Portal. Estos datos no están disponibles en el portal de EA (ea.azure.com), debe descargar el archivo de uso de Azure Portal (portal.azure.com) para ver los nuevos datos.

En Azure Portal, vaya a [Administración de costos + facturación](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/BillingAccounts).

1. Seleccione el nombre de la cuenta de facturación.
2. Haga clic en **Uso + cargos**.
3. Haga clic en **Descargar**.  
![Ejemplo que muestra dónde debe descargar el archivo de datos de uso CSV en Azure Portal](./media/understand-reserved-instance-usage-ea/portal-download-csv.png)
4. En **Detalles de uso**, seleccione **Amortized usage data** (Datos de uso amortizados).

Los archivos CSV que descarga contienen los costos reales y los costos amortizados.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes
- Para más información acerca de los datos de uso de Azure Reservations, consulte los siguientes artículos:
  - [Costos y uso de reservas del Contrato Enterprise y del Contrato de cliente de Microsoft](understand-reserved-instance-usage-ea.md)
 
