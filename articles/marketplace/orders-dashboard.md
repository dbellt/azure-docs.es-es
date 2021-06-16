---
title: Panel Pedidos del Centro de partners de los análisis del marketplace comercial | Microsoft AppSource y Azure Marketplace
description: Obtenga información sobre cómo acceder a informes analíticos sobre los pedidos de ofertas del marketplace comercial en un formato gráfico y descargable.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 344c7e6c14ca8ecdd91ffb9c148ce01d2b937573
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111542010"
---
# <a name="orders-dashboard-in-commercial-marketplace-analytics"></a>Panel Pedidos de los análisis de marketplace comercial

En este artículo se proporciona información sobre el panel Pedidos del Centro de partners. En este panel se muestra información sobre los pedidos, incluidas las tendencias de crecimiento, presentada en un formato gráfico y descargable.

Para acceder al panel Pedidos en el Centro de partners, vaya a **Marketplace comercial** y seleccione **[Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Pedidos**.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Terminología de análisis y preguntas comunes de Marketplace comercial](./analytics-faq.md).

## <a name="orders-dashboard"></a>Panel Pedidos

En el panel Pedidos se muestran los pedidos actuales de todas las ofertas de software como servicio (SaaS). Puede ver representaciones gráficas de los elementos siguientes:

- Tendencia de pedidos
- Pedidos por tendencia de puesto y sitio
- Pedidos en función de las ofertas y las SKU
- Pedidos por geografía
- Tabla de pedidos detallados
- Filtros de la página de pedidos

> [!NOTE]
> La latencia máxima entre la adquisición de clientes y la generación de informes en el Centro de partners es de 48 horas.

## <a name="elements-of-the-orders-dashboard"></a>Elementos del panel Pedidos

En las secciones siguientes se describe cómo usar el panel Pedidos y cómo leer los datos.

### <a name="month-range"></a>Intervalo mensual

Puede encontrar una selección del intervalo mensual en la esquina superior derecha de cada página. Puede personalizar salida de los gráficos de la página **Pedidos** seleccionando un intervalo mensual basado en los últimos 6 o 12 meses, o seleccionando un intervalo mensual personalizado con una duración máxima de 12 meses. El intervalo mensual predeterminado (período de cálculo) es de seis meses.

:::image type="content" source="./media/orders-dashboard/month-filters.png" alt-text="Ilustra los filtros del mes en el panel Pedidos.":::

> [!NOTE]
> Todas las métricas de los widgets de visualización y los informes de exportación respetan el período de cálculo seleccionado por el usuario.

### <a name="orders-trend"></a>Tendencia de pedidos

En esta sección, encontrará el gráfico **Pedidos** que muestra la tendencia de los pedidos activos y cancelados para el período de cálculo seleccionado. Las métricas y las tendencias de crecimiento se representan mediante un gráfico de líneas y mostrarán el valor de cada mes al mantener el puntero sobre la línea del gráfico. El valor de porcentaje que aparece debajo de las métricas de Pedidos del widget representa la cantidad de crecimiento o disminución durante el período de cálculo seleccionado.

Hay dos contadores de pedidos: _Activos_ y _Cancelados_.

- **Activos** es igual al número de pedidos que los clientes están utilizando actualmente durante el intervalo de fechas seleccionado.
- **Cancelados** es igual al número de pedidos que se compraron previamente y se cancelaron durante el intervalo de fechas seleccionado.

[![Ilustra el widget Pedidos en el panel Pedidos que muestra la tendencia de los pedidos activos y cancelados.](./media/orders-dashboard/orders-trend.png)](./media/orders-dashboard/orders-trend.png#lightbox)

### <a name="orders-by-per-seat-and-site-trend"></a>Pedidos por tendencia de puesto y sitio

El gráfico de líneas **Pedidos basados en puesto y sitio** representa la métrica y la tendencia de los pedidos SaaS por sitio y por puesto que los clientes compran (este gráfico incluye pedidos cancelados).

[![Ilustra el widget Pedidos en el panel Pedidos que muestra los pedidos por tendencia de puesto y sitio.](./media/orders-dashboard/seats-sites.png)](./media/orders-dashboard/seats-sites.png#lightbox)

Las ofertas de SaaS pueden usar dos modelos de precios con cada plan: tarifa plana (basada en el sitio) o por usuario (basada en el puesto).

- **Tarifa plana**: habilite el acceso a la oferta con una única tarifa plana mensual o anual. A veces esto se denomina precios basados en puesto.
- **Por usuario**: habilite el acceso a su oferta con un precio basado en el número de usuarios que pueden acceder a la oferta u ocupar los puestos. Con este modelo basado en el uso, puede establecer el número mínimo y máximo de usuarios admitidos por el plan. Además, puede crear varios planes para configurar diferentes puntos de precio en función del número de usuarios. Estos campos son opcionales. Si no se seleccionan, se interpretará que el número de usuarios no tiene límite (el mínimo es 1 y el máximo es tantos como el sistema pueda admitir). Estos campos se pueden editar como parte de una actualización del plan.
- **Facturación de uso medido**: además de los precios de tarifa plana. Con este modelo de precios, puede definir opcionalmente planes de uso medido que emplean la API del servicio de medición del marketplace para cobrar a los clientes por el uso que no está cubierto por la tarifa plana.

Para más información sobre la facturación basada en puestos, sitios y uso medido, consulte [Planeamiento de una oferta de SaaS en el marketplace comercial](plan-saas-offer.md).

### <a name="orders-by-offers-and-skus"></a>Pedidos en función de las ofertas y las SKU

En el gráfico Pedidos en función de las de ofertas y las SKU se muestran las medidas y tendencias de todas las ofertas:

- Las ofertas principales se muestran en el gráfico y el resto se agrupan como **Restantes**.
- Puede seleccionar las ofertas específicas de la leyenda para mostrar solo esa oferta y las SKU asociadas en el gráfico.
- Al mantener el puntero sobre un sector del gráfico aparece el número de pedidos y el porcentaje de esa oferta en comparación con el número total de pedidos de todas las demás ofertas.
- La **tendencia de los pedidos por ofertas** muestra las tendencias de crecimiento mensualmente. La columna de mes representa el número de pedidos por nombre de la oferta. El gráfico de líneas muestra la tendencia del porcentaje de crecimiento trazada en un eje z.

:::image type="content" source="./media/orders-dashboard/orders-by-offers.png" alt-text="Ilustra el panel Pedidos por ofertas en el panel Pedidos.":::

Puede seleccionar cualquier oferta y un máximo de tres SKU de esa oferta para ver la tendencia mensual de la oferta, las SKU y los puestos.

### <a name="orders-by-geography"></a>Pedidos por geografía

En el período de cálculo seleccionado, el mapa térmico muestra el número total de pedidos y el porcentaje de crecimiento de los pedidos recién agregados en una región geográfica.  El tono del color de claro a oscuro del mapa representa el número de clientes de bajo a alto. Seleccione un registro de la tabla para acercar un país o región específicos.

[![Ilustra el gráfico Diseminación geográfica en el panel Pedidos.](./media/orders-dashboard/geographical-spread.png)](./media/orders-dashboard/geographical-spread.png#lightbox)

Tenga en cuenta lo siguiente:

- Puede mover el mapa para ver la ubicación exacta.
- Puede hacer zoom en una ubicación específica.
- El mapa térmico tiene una cuadrícula complementaria para ver los detalles del recuento de clientes, el recuento de pedidos y las horas de uso normalizadas para la ubicación específica.
- Puede buscar y seleccionar un país o región en la cuadrícula para ampliar la ubicación en el mapa. Vuelva a la vista original presionando el botón **Inicio** en el mapa.

### <a name="orders-details-table"></a>Tabla Detalles del pedido

En la tabla Detalles del pedido se muestra una lista numerada de los 1000 pedidos principales ordenados por fecha de adquisición.

- Cada columna de la cuadrícula se puede ordenar.
- Los datos se pueden extraer a un archivo .csv o .tsv si el recuento de los registros es inferior a 1000.
- Si el número de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas durante los próximos 30 días.
- Aplique filtros a la tabla **Detalles del pedido** a fin de mostrar solo los datos que le interesen. Filtre por país o región, tipo de licencia de Azure, tipo de licencia del marketplace comercial, tipo de oferta, estado de pedido, evaluaciones gratuitas, identificador de suscripción del marketplace comercial, identificador de cliente y nombre de la empresa.
- Cuando un cliente protegido adquiere un pedido, la información de **Customer Detailed Data** (Datos detallados del pedido) se enmascara (************).

***Tabla 1: Diccionario de términos de datos***

| Nombre de la columna en la<br>interfaz de usuario | Nombre del atributo | Definición | Nombre de la columna en informes<br>de acceso mediante programación |
| ------------ | ------------- | ------------- | ------------- |
| Identificador de la suscripción a Marketplace | Identificador de la suscripción a Marketplace | Identificador único asociado a la suscripción a Azure que el cliente usó para comprar la oferta de Marketplace comercial. En el caso de las ofertas de infraestructura, este es el GUID de la suscripción de Azure del cliente. En el caso de las ofertas de SaaS, se muestra como ceros, ya que las compras de SaaS no requieren una suscripción a Azure. | Identificador de la suscripción a Marketplace |
| MonthStartDate | Fecha de inicio del mes | Fecha de inicio del mes representa el mes de la compra. El formato es aaaa-mm-dd. | MonthStartDate |
| Tipo de oferta | Tipo de oferta | Tipo de oferta de Marketplace comercial. | OfferType |
| Azure License Type (Tipo de licencia de Azure) | Azure License Type (Tipo de licencia de Azure) | Tipo de contrato de licencia que los clientes utilizan para comprar Azure. También conocido como Canal. Los valores posibles son:<ul><li>Proveedor de soluciones en la nube</li><li>Enterprise</li><li>Enterprise a través de revendedor</li><li>Pago por uso</li></ul> | AzureLicenseType |
| Marketplace License Type (Tipo de licencia de Marketplace) | Marketplace License Type (Tipo de licencia de Marketplace) | Método de facturación de la oferta de Marketplace comercial. Los distintos valores son:<ul><li>Facturado a través de Azure</li><li>Traiga su propia licencia</li><li>Gratuito</li><li>Microsoft como revendedor</li></ul> | MarketplaceLicenseType |
| SKU | SKU | Plan asociado con la oferta. | SKU |
| Customer Country (País del cliente) | País o región del cliente. | Nombre del país o región proporcionado por el cliente. Podría ser diferente del país o región de la suscripción a Azure del cliente. | CustomerCountry |
| Versión preliminar de SKU | Versión preliminar de SKU | El valor le permitirá saber si ha etiquetado la SKU como "versión preliminar". El valor es "Sí" si la SKU se ha etiquetado en consecuencia y solo las suscripciones de Azure autorizadas por usted pueden implementar y usar esta imagen. El valor es "No" si la SKU no se ha identificado como "versión preliminar". | IsPreviewSKU |
| Identificador del pedido | Identificador del pedido | Identificador único del pedido del cliente para el servicio de Marketplace comercial. Las ofertas basadas en el uso de máquinas virtuales no están asociadas con un pedido. | OrderId |
| Cantidad del pedido | Cantidad del pedido | Número de recursos asociados al identificador del pedido para pedidos activos. | OrderQuantity |
| Nombre de la instancia de nube | Nombre de la instancia de nube | Microsoft Cloud donde se produjo la implementación de una VM. | Nombre de instancia en la nube |
| Cliente nuevo | Cliente nuevo | El valor identifica si un nuevo cliente adquirió una o varias de sus ofertas por primera vez. El valor es "Sí" si es el mismo mes natural de la "Fecha de adquisición". El valor es "No" si el cliente ha comprado cualquiera de las ofertas antes del mes natural notificado. | IsNewCustomer |
| Order Status (Estado del pedido) | Order Status (Estado del pedido) | Estado de un pedido de Marketplace comercial en el momento en que se actualizaron por última vez los datos. | OrderStatus |
| Order Cancel Date (Fecha de cancelación de pedido) | Order Cancel Date (Fecha de cancelación de pedido) | Fecha en que se canceló el pedido de Marketplace comercial. | OrderCancelDate |
| Customer Company Name (Nombre de la empresa del cliente) | Customer Company Name (Nombre de la empresa del cliente) | Nombre de empresa proporcionado por el cliente. El nombre podría ser diferente de la ciudad de la suscripción a Azure del cliente. | CustomerCompanyName |
| Order Purchase Date (Fecha de compra del pedido) | Order Purchase Date (Fecha de compra del pedido) | Fecha en que se creó el pedido de Marketplace comercial. El formato es aaaa-mm-dd. | OrderPurchaseDate |
| Nombre de la oferta | Nombre de la oferta | Nombre de la oferta de Marketplace comercial. | OfferName |
| Trial End Date (Fecha de fin de la prueba) | Trial End Date (Fecha de fin de la prueba) | La fecha del período de prueba de este pedido va a finalizar o ha finalizado. | TrialEndDate |
| Customer ID | Customer ID | Identificador único asignado a un cliente. Un cliente puede no tener ninguna suscripción a Azure Marketplace o bien tener varias. | CustomerId |
| Identificador de la cuenta de facturación | Identificador de la cuenta de facturación | Identificador de la cuenta en la que se genera la facturación. Asigne el **identificador de la cuenta de facturación** a **customerID** para conectar el informe de transacciones de pago con los informes de cliente, pedido y uso. | BillingAccountId |
| AssetCount | Número de recursos | Número de recursos asociados al identificador del pedido. | Obsoleto |
|||||

### <a name="orders-page-filters"></a>Filtros de la página de pedidos

Los filtros de la página **Pedidos** se aplican en el nivel de la página de Pedidos. Puede seleccionar uno o varios filtros para representar el gráfico para los criterios que quiera ver, así como los datos que quiera que se muestren en la cuadrícula o exportación "Datos detallados de los pedidos". Los filtros se aplican a los datos extraídos del intervalo del mes que ha seleccionado en la esquina superior derecha de la página de pedidos.

> [!TIP]
> Puede usar el icono de descarga que está en la esquina superior derecha de cualquier widget para descargar los datos. Igualmente, para proporcionar comentarios sobre cada uno de los widgets, haga clic en el icono "pulgar hacia arriba" o "pulgar hacia abajo".

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre los informes de análisis disponibles en marketplace comercial, consulte [Acceso a los informes de análisis de marketplace comercial en el Centro de partners](analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resumen la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](downloads-dashboard.md).
- Para obtener una vista consolidada con los comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel de análisis de calificaciones y opiniones en el Centro de partners](ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./analytics-faq.md).