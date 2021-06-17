---
title: Panel de uso en el análisis de Marketplace comercial | Azure Marketplace
description: Obtenga información sobre cómo obtener acceso a todas las métricas de facturación de uso y uso medido relacionadas con las ofertas publicadas en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 04/28/2021
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: fca763f307c8ae1437009b282e460fc249c40b60
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111540966"
---
# <a name="usage-dashboard-in-commercial-marketplace-analytics"></a>Panel de uso de los análisis de marketplace comercial

En este artículo se proporciona información sobre el panel de uso del Centro de partners. En este panel se muestran todas las máquinas virtuales (VM) que ofrecen métricas de facturación de uso normalizado, de uso sin procesar y de uso medido en tres pestañas diferentes: Uso normalizado de VM, uso sin procesar de VM y uso medido de facturación.

Para obtener acceso al panel de uso en el centro de partners, vaya a **Marketplace comercial** y seleccione **[Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Uso**.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Terminología de análisis y preguntas comunes de Marketplace comercial](./analytics-faq.md).

## <a name="usage-dashboard"></a>Panel de uso

En el panel **Uso** del menú **Analizar** se muestran los pedidos actuales de todas las ofertas de software como servicio (SaaS). Puede ver representaciones gráficas de los elementos siguientes:

- Tendencia de uso
- Uso normalizado en función de las ofertas
- Uso normalizado en función de otras dimensiones: Tamaño de VM, canales de ventas y tipos de oferta
- Uso por geografía
- Tabla de uso detallada
- Filtros de la página de pedidos

> [!NOTE]
> La latencia máxima entre la generación del evento de uso y el informe en el Centro de partners es de 48 horas.

## <a name="elements-of-the-usage-dashboard"></a>Elementos del panel de uso

En las secciones siguientes se describe cómo usar el panel de Uso y cómo leer los datos.

### <a name="month-range"></a>Intervalo mensual

Puede encontrar una selección del intervalo mensual en la esquina superior derecha de cada página. Puede personalizar salida de los gráficos de la página **Uso** seleccionando un intervalo mensual basado en los últimos 6 o 12 meses, o seleccionando un intervalo mensual personalizado con una duración máxima de 12 meses. El intervalo mensual predeterminado (período de cálculo) es de seis meses.

:::image type="content" source="./media/usage-dashboard/month-filter.png" alt-text="Muestra los filtros del mes en el panel de Uso.":::

### <a name="usage-trend"></a>Tendencia de uso

En esta sección, encontrará las horas de uso totales y la tendencia de todas las ofertas que consumen los clientes durante el período de cálculo seleccionado. Las métricas y las tendencias de crecimiento se representan mediante un gráfico de líneas. Muestre el valor de cada mes desplazando el puntero sobre la línea del gráfico. El valor de porcentaje que aparece debajo de las métricas de uso del widget representa la cantidad de crecimiento o disminución durante el período de cálculo seleccionado.

Existen dos representaciones de las horas de uso: uso normalizado de VM y uso sin procesar de VM.

- Las horas de uso normalizadas se definen para contabilizar el número de núcleos de VM ([número de núcleos de VM] x [horas de uso sin procesar]). Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de máquina virtual].
- Las horas de uso sin procesar se definen como la cantidad de tiempo que las máquinas virtuales se han ejecutado en términos de horas.

[![Ilustra el uso normalizado y los datos de uso sin procesar en el panel de Uso.](./media/usage-dashboard/normalized-usage.png)](./media/usage-dashboard/normalized-usage.png#lightbox)

### <a name="normalized-usage-by-offers"></a>Uso normalizado en función de las ofertas

En esta sección se proporcionan las horas de uso totales y la tendencia de las ofertas basadas en el uso en Azure Marketplace. El gráfico de uso normalizado de las ofertas se describe a continuación.

- El gráfico de columnas apiladas de **uso normalizado de las ofertas** muestra un desglose de las horas de uso normalizado correspondientes a las cinco ofertas principales según el período de cálculo seleccionado. En el gráfico se muestran las cinco ofertas principales y el resto se agrupan en la categoría **Restantes**.
- En el gráfico de barras apiladas se muestra una tendencia de crecimiento mensual para el intervalo de fechas seleccionado. Las columnas de mes representan las horas de uso de las ofertas, con las horas de mayor uso para el mes correspondiente. En el gráfico de líneas se muestra la tendencia del porcentaje de crecimiento trazada en el eje Y secundario.
- Puede seleccionar las ofertas específicas de la leyenda para mostrar solo esas ofertas en el gráfico.

:::image type="content" source="./media/usage-dashboard/normalized-usage-offers.png" alt-text="Muestra los datos del uso normalizado de las ofertas en el panel de Uso.":::

Puede seleccionar cualquier oferta y un máximo de tres SKU de esa oferta para ver la tendencia de uso mensual de la oferta y las SKU seleccionadas.

:::image type="content" source="./media/usage-dashboard/normalized-usage-offers-sku.png" alt-text="Muestra los datos del uso normalizado de las ofertas y las SKU en el panel de Uso.":::

### <a name="orders-by-offers-and-skus"></a>Pedidos en función de las ofertas y las SKU

En el gráfico de **Pedidos en función de las de ofertas y las SKU** se muestran las medidas y tendencias de todas las ofertas. Tenga en cuenta lo siguiente:

- Las ofertas principales se muestran en el gráfico y el resto se agrupan como **Restantes**.
- Puede seleccionar las ofertas específicas de la leyenda para mostrar solo esas ofertas en el gráfico.
- Al mantener el puntero sobre un sector del gráfico aparece el número de pedidos y el porcentaje de esa oferta en comparación con el número total de pedidos de todas las demás ofertas.
- La **tendencia de los pedidos por ofertas** muestra las tendencias de crecimiento mensualmente. La columna de mes representa el número de pedidos por nombre de la oferta. El gráfico de líneas muestra la tendencia del porcentaje de crecimiento trazada en un eje z.

:::image type="content" source="./media/usage-dashboard/orders-by-offers-and-skus.png" alt-text="Muestra el gráfico de pedidos en función de las ofertas y las SKU en el panel de Uso.":::

Puede seleccionar cualquier oferta y un máximo de tres SKU de esa oferta para ver la tendencia mensual de la oferta, las SKU y los puestos.

:::image type="content" source="./media/usage-dashboard/orders-by-offers-and-skus-2.png" alt-text="Muestra el gráfico de pedidos en función de las ofertas y las SKU en el panel de Uso. Se muestran la tendencia de la oferta, de las SKU y de los puestos.":::

#### <a name="normalized-usage-by-other-dimensions-vm-size-sales-channels-and-offer-type"></a>Uso normalizado en función de otras dimensiones: Tamaño de VM, canales de ventas y tipos de oferta

Hay tres pestañas para las dimensiones: tamaño de VM, canales de ventas y tipos de oferta. Puede ver las métricas de uso y la tendencia mensual en cada una de estas dimensiones.

:::image type="content" source="./media/usage-dashboard/normalized-usage-other-dimensions.png" alt-text="Muestra el gráfico de uso normalizado de otras dimensiones en el panel de Uso.":::

### <a name="usage-by-geography"></a>Uso por geografía

En el período de cálculo seleccionado, el mapa térmico muestra el uso total en la dimensión geográfica. El tono del color de claro a oscuro del mapa representa el número de clientes de bajo a alto. Seleccione un registro de la tabla para acercar un país o región.

:::image type="content" source="./media/usage-dashboard/normalized-usage-country.png" alt-text="Muestra el gráfico con el país que cuenta con el uso normalizado en el panel de Uso.":::

Tenga en cuenta lo siguiente:

- Puede mover el mapa para ver la ubicación exacta.
- Puede hacer zoom en una ubicación específica.
- El mapa térmico tiene una cuadrícula complementaria para ver los detalles del recuento de clientes, el recuento de pedidos y las horas de uso normalizado en la ubicación específica.
- Puede buscar y seleccionar un país o región en la cuadrícula para ampliar la ubicación en el mapa. Vuelva a la vista original presionando el botón **Inicio** en el mapa.

### <a name="usage-details-table"></a>Tabla de detalles de uso

En la **tabla de detalles de uso** se muestra una lista numerada de los 1000 registros de uso principales ordenados por uso. Tenga en cuenta lo siguiente:

- Cada columna de la cuadrícula se puede ordenar.
- Los datos se pueden extraer en un archivo .TSV o .CSV si el recuento de los registros es inferior a 1000.
- Si el recuento de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas que estará disponible durante los próximos 30 días.
- Aplique filtros a los **datos de uso detallados** para mostrar solo los datos que le interesen. Filtre los datos por país o región, canal de ventas, tipo de licencia de marketplace, tipo de uso, tipo y nombre de la oferta, evaluaciones gratuitas, identificador de la suscripción a marketplace, identificador de cliente y nombre de la empresa.

_**Tabla 1: diccionario de términos de datos**_

| Nombre de la columna en la<br>interfaz de usuario | Nombre del atributo | Definición | Nombre de la columna en informes<br>de acceso mediante programación |
| ------------ | ------------- | ------------- | ------------- |
| Identificador de la suscripción a Marketplace | Identificador de la suscripción a Marketplace | Identificador único asociado a la suscripción a Azure que el cliente usó para comprar la oferta de Marketplace comercial. Antes era el GUID de la suscripción a Azure. | MarketplaceSubscriptionId |
| MonthStartDate | Fecha de inicio del mes | Fecha de inicio del mes representa el mes de la compra. | MonthStartDate |
| Tipo de oferta | Tipo de oferta | Tipo de oferta de Marketplace comercial. | OfferType |
| Azure License Type (Tipo de licencia de Azure) | Azure License Type (Tipo de licencia de Azure) | Tipo de contrato de licencia que los clientes utilizan para comprar Azure. También se conoce como Canal. Los valores posibles son:<ui><li>Proveedor de soluciones en la nube</li><li>Enterprise</li><li>Enterprise a través de revendedor</li><li>Pago por uso</li></ul> | AzureLicenseType |
| Marketplace License Type (Tipo de licencia de Marketplace) | Marketplace License Type (Tipo de licencia de Marketplace) | Método de facturación de la oferta de Marketplace comercial. Los valores posibles son:<ul><li>Facturado a través de Azure</li><li>Traiga su propia licencia</li><li>Gratuito</li><li>Microsoft como revendedor</li></ul> | MarketplaceLicenseType |
| SKU | SKU | Plan asociado con la oferta. | SKU |
| Customer Country (País del cliente) | País o región del cliente. | Nombre del país o región proporcionado por el cliente. Podría ser diferente del país o región de la suscripción a Azure del cliente. | CustomerCountry |
| Versión preliminar de SKU | Versión preliminar de SKU | Valor que le mostrará si ha etiquetado la SKU como "versión preliminar". El valor es "Sí" si la SKU se ha etiquetado en consecuencia y solo las suscripciones de Azure autorizadas por usted pueden implementar y usar esta imagen. El valor es "No" si la SKU no se ha identificado como "versión preliminar". | IsPreviewSKU |
| SKU Billing Type (Tipo de facturación de la SKU) | SKU Billing Type (Tipo de facturación de la SKU) | Tipo de facturación asociado a cada SKU de la oferta. Los valores posibles son:<ul><li>Gratuito</li><li>De pago</li></ul> | SKUBillingType |
| IsInternal | Obsoleto | Obsoleto | Obsoleto |
| Tamaño de máquina virtual | Tamaño de la máquina virtual | En el caso de los tipos de oferta basados en VM, esta entidad indica el tamaño de la VM asociada a la SKU de la oferta. | VMSize |
| Nombre de la instancia de nube | Nombre de la instancia de nube | Microsoft Cloud donde se produjo la implementación de una VM. | Nombre de instancia en la nube |
| ServicePlanName | Obsoleto | En desuso (es la misma definición que la SKU). | ServicePlanName |
| Nombre de la oferta | Nombre de la oferta | Nombre de la oferta de Marketplace comercial. | OfferName |
| DeploymentMethod | Obsoleto | En desuso (es la misma definición que el tipo de oferta). | DeploymentMethod |
| Customer Company Name (Nombre de la empresa del cliente) | Customer Company Name (Nombre de la empresa del cliente) | Nombre de empresa proporcionado por el cliente. El nombre podría ser diferente del nombre de la suscripción a Azure del cliente. | CustomerCompanyName |
| Fecha de uso | Fecha de uso | Fecha de generación de eventos de uso para recursos basados en el uso. | UsageDate |
| IsMultisolution | Es multisolución | Indica si la oferta es un tipo de oferta multisolución. | IsMultisolution |
| Cliente nuevo | Obsoleto | Obsoleto | IsNewCustomer |
| Tamaño de núcleo | Tamaño de núcleo | Número de núcleos asociados con la oferta basada en la VM. | CoreSize |
| Usage Type (Tipo de uso) | Usage Type (Tipo de uso) | Indica si el evento de uso asociado a la oferta es uno de los siguientes:<ul><li>Uso normalizado</li><li>Uso sin procesar</li><li>Uso medido</li></ul> | UsageType |
| Trial End Date (Fecha de fin de la prueba) | Trial End Date (Fecha de fin de la prueba) | La fecha del período de prueba de este pedido va a finalizar o ha finalizado. | TrialEndDate |
| Customer Currency (CC) [Moneda del cliente (CC)] | Moneda del cliente | Moneda que usa el cliente para las transacciones en Marketplace comercial. | CustomerCurrencyCC |
| Price (CC) [Precio (CC)] | Price | Precio unitario de la SKU que se muestra en la moneda del cliente. | PriceCC |
| Payout Currency (PC) [Moneda de pago (PC)] | Moneda de pago | Se paga al publicador en función los eventos de uso asociados al recurso en la moneda que haya configurado ese publicador. | PayoutCurrencyPC |
| Estimated Price (PC) [Precio estimado (PC)] | Precio estimado | Precio unitario de la SKU en la moneda que haya configurado el publicador. | EstimatedPricePC |
| Usage Reference (Referencia de uso) | Usage Reference (Referencia de uso) | GUID concatenado que se usa para conectar el informe de uso (en el análisis de Marketplace comercial) con el informe de transacciones de pago. La referencia de uso está conectada con OrderId y LineItemId en el informe de transacciones de pago. | UsageReference |
| Unidad de uso | Unidad de uso | Unidad de consumo asociada a la SKU. | UsageUnit |
| Customer ID | Customer ID | Identificador único asignado a un cliente. Un cliente puede no tener ninguna suscripción a Azure Marketplace o bien tener varias. | CustomerId |
| Identificador de la cuenta de facturación | Identificador de la cuenta de facturación | Identificador de la cuenta en la que se genera la facturación. Asigne el **identificador de la cuenta de facturación** a **customerID** para conectar el informe de transacciones de pago con los informes de cliente, pedido y uso. | BillingAccountId |
| Cantidad de uso | Cantidad de uso | Unidades de uso totales que usa el recurso que implementa el cliente.<br>Estas se basan en el elemento de Tipo de uso. Por ejemplo, si el Tipo de uso es de Uso normalizado, entonces la Cantidad de uso es para el Uso normalizado. | UsageQuantity |
| NormalizedUsage | Uso normalizado | Unidades de uso normalizadas totales que usa el recurso que implementa el cliente.<br>Las horas de uso normalizadas se definen para contabilizar el número de núcleos de VM ([número de núcleos de VM] x [horas de uso sin procesar]). Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de máquina virtual]. | NormalizedUsage |
| MeteredUsage | Uso medido | Unidades de uso totales que consumen los medidores configurados con la oferta que implementó el cliente. | MeteredUsage |
| RawUsage | Raw Usage (Uso sin procesar) | Unidades de uso sin procesar totales que usa el recurso que implementa el cliente.<br>Las horas de uso sin procesar se definen como la cantidad de tiempo que las VM se han ejecutado en términos de unidades de uso. | RawUsage |
| Estimated Extended Charge (CC) [Cargo total estimado (CC)] | Cargo total estimado en la moneda del cliente | Indica los cargos asociados al uso. La columna es el producto de precio (CC) y la cantidad de uso. | EstimatedExtendedChargeCC |
| Estimated Extended Charge (PC) [Cargo total estimado (PC)] | Cargo total estimado en la moneda de pago | Indica los cargos asociados al uso. La columna es el producto del precio estimado (PC) y la cantidad de uso. | EstimatedExtended ChargePC |
| Id. de medidor | Id. de medidor | Indica el identificador de medidor de la oferta. | MeterId |
| Anomalía detectada del Centro de partners | Anomalía detectada del Centro de partners | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Indica si el publicador ha notificado un uso por encima del límite para la dimensión de medidor personalizado de la oferta que el Centro de partners ha marcado como una anomalía. Los valores posibles son: <ul><li>0 (no es una anomalía)</li><li>1 (es una anomalía)</li></ul>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna a través del acceso mediante programación, el valor será NULL._ | PartnerCenterDetectedAnomaly |
| Anomalía marcada por el publicador | Anomalía marcada por el publicador | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Indica si el publicador ha reconocido el uso por encima del límite por parte del cliente para la dimensión de medidor personalizado de la oferta como genuino o falso. Los valores posibles son:<ul><li>0 (el publicador lo ha marcado como que no es una anomalía)</li><li>1 (el publicador lo ha marcado como que es una anomalía)</li></ul>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna a través del acceso mediante programación, el valor será NULL._ | PublisherMarkedAnomaly |
| Nuevo uso notificado | Nuevo uso notificado | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Para el uso por encima del límite por parte del cliente para la dimensión de medidor personalizado de la oferta identificada como anómala por el publicador. Este campo especifica el nuevo uso por encima del límite notificado por el publicador.<br>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna a través del acceso mediante programación, el valor será NULL._ | NewReportedUsage |
| Acción realizada a las | Acción realizada a las | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Indica la hora en que el publicador ha reconocido el uso por encima del límite por parte del cliente para la dimensión de medidor personalizado de la oferta como genuino o falso.<br>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna a través del acceso mediante programación, el valor será NULL._ | ActionTakenAt |
| Acción realizada por | Acción realizada por | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Indica la persona que ha reconocido el uso por encima del límite por parte del cliente para la dimensión de medidor personalizado de la oferta como genuino o falso.<br>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna a través del acceso mediante programación, el valor será NULL._ | ActionTakenBy |
| Impacto financiero estimado (USD) | Impacto financiero estimado en USD | **Aplicable a las ofertas con dimensiones de medidores personalizados**.<br>Cuando el Centro de partners marca como anómalo un uso por encima del límite por parte del cliente para la dimensión de medidor personalizado de la oferta, el campo especifica el impacto financiero estimado (en USD) del uso anómalo por encima del límite.<br>_Si el publicador no tiene ofertas con dimensiones de medidores personalizados y exporta esta columna mediante programación, el valor será NULL._ | EstimatedFinancialImpactUSD |
|||||

### <a name="usage-page-filters"></a>Filtros de la página de uso

Los filtros de la página de **Uso** se aplican en el nivel de la página de Pedidos. Puede seleccionar varios filtros para representar el gráfico para los criterios que quiera ver, así como los datos que quiera que se muestren en la cuadrícula o exportación "Datos de los pedidos de uso". Los filtros se aplican a los datos extraídos del intervalo mensual que haya seleccionado en la esquina superior derecha de la página de uso.

Los widgets y el informe de exportación para el uso sin procesar de la VM son similares al uso normalizado de las VM, pero cuentan con las siguientes distinciones:

- Las horas de uso normalizadas se definen para contabilizar el número de núcleos de VM ([número de núcleos de VM] x [horas de uso sin procesar]). Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de máquina virtual].
- Las horas de uso sin procesar se definen como la cantidad de tiempo que las VM se han ejecutado en términos de unidades de uso.

### <a name="metered-billing-usage"></a>Facturación de uso medido

En la pestaña **Uso medido** se muestra la información de uso de los tipos de oferta, en la cual el uso se mide en función de la dimensión del medidor. Actualmente, se presenta el uso por encima del límite del tipo de oferta de SaaS. En la pestaña se muestran representaciones gráficas de las tendencias de uso por encima del límite del uso de facturación medido de SaaS:

- **Tendencia superior al límite por dimensión de medidor**: muestra la tendencia superior al límite mensual para la dimensión de medidor seleccionada de una oferta. El eje X representa el mes y el eje Y representa la cantidad de uso por encima del límite. La unidad de medida del medidor personalizado también se muestra en el eje Y.

    :::image type="content" source="./media/usage-dashboard/metered-usage.png" alt-text="Muestra el gráfico del uso normalizado de la VM en el panel de Uso.":::

- **Tendencia superior al límite por SKU**: representa la tendencia de la cantidad de uso de la dimensión de medidor que haya seleccionado la SKU o los planes. Se muestran los cinco planes principales con la mayor cantidad de uso de la oferta seleccionada.

- **Tendencia de uso por encima del límite de los clientes**: El panel de líderes del cliente representa una lista apilada de clientes con las horas de uso más frecuentes que se muestran en una _tabla de clasificación_ ordenada a partir del uso más alto del medidor personalizado. Seleccione un cliente de la tabla de clasificación para ver la tendencia de uso por encima del límite de una dimensión de medidor seleccionada.

    :::image type="content" source="./media/usage-dashboard/metered-usage-by-customers.png" alt-text="Muestra el gráfico de uso medido por clientes en el panel de Uso.":::

Si tiene varias ofertas que usan medidores personalizados, el informe de uso de facturación medido muestra información de uso de todas las ofertas, de acuerdo con sus dimensiones de medidor personalizadas.

> [!TIP]
> Puede usar el icono de descarga que está en la esquina superior derecha de cualquier widget para descargar los datos. Igualmente, para proporcionar comentarios sobre cada uno de los widgets, haga clic en el icono "pulgar hacia arriba" o "pulgar hacia abajo".

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre los informes de análisis disponibles en marketplace comercial, consulte [Acceso a los informes de análisis de marketplace comercial en el Centro de partners](analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resuman la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para obtener información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel de pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](usage-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](downloads-dashboard.md).
- Para obtener una vista consolidada con los comentarios de los clientes sobre las ofertas de Azure Marketplace y Microsoft AppSource, consulte [Panel de análisis de calificaciones y opiniones en el Centro de partners](ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./analytics-faq.md).
