---
title: Descripción de una factura para el contrato de cliente de Microsoft en Azure
description: Aprenda a leer y entender la factura para el contrato de cliente de Microsoft en Azure.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: banders
ms.openlocfilehash: ff53131f3078b33b7e7d853c1fca891b0b86d792
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484611"
---
# <a name="terms-in-your-microsoft-customer-agreement-invoice"></a>Términos en la factura del contrato de cliente de Microsoft

Este artículo se aplica a una cuenta de facturación de Azure de un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

La factura proporciona un resumen de los gastos e instrucciones de pago. Se encuentra disponible en Portable Document Format (.pdf) desde [Azure Portal](https://portal.azure.com/) o puede enviarse por correo electrónico. Para más información, consulte [Visualización y descarga de la factura de Microsoft Azure](download-azure-invoice.md).

Vea este vídeo con [información sobre la factura del Contrato de cliente de Microsoft](https://www.youtube.com/watch?v=e2LGZZ7GubA) para conocer los detalles de la factura y analizar los cargos que contiene.

>[!VIDEO https://www.youtube.com/embed/e2LGZZ7GubA]

## <a name="billing-period"></a>Período de facturación

Recibe facturas mensuales. Puede averiguar qué día del mes recibe las facturas si revisa la *fecha de la factura* en las propiedades del perfil de facturación en [Azure Portal](https://portal.azure.com/). Los cargos que se producen entre el final del período de facturación y la fecha de la factura se incluyen en la factura del próximo mes, puesto que se encuentran en el período de facturación	 siguiente. Las fechas de inicio y finalización del período de facturación para cada factura aparecen en el archivo PDF de la factura en **Resumen de facturación**.

Si va a migrar de un Contrato Enterprise a un Contrato de cliente de Microsoft, seguirá recibiendo facturas de su Contrato Enterprise hasta el momento de la migración. La nueva factura del Contrato de cliente de Microsoft se genera el quinto día del mes después de la migración. La primera factura muestra un cargo parcial a partir de la fecha de migración. Las facturas posteriores se generan cada mes y muestran todos los cargos de cada mes.

### <a name="changes-for-pay-as-you-go-subscriptions"></a>Cambios en las suscripciones de pago por uso

Cuando se traslada, transfiere o cancela una suscripción, la última factura generada contiene los cargos del ciclo de facturación anterior y el nuevo ciclo de facturación incompleto.

Por ejemplo:

Suponga que el ciclo de facturación de la suscripción de pago por uso es todos los meses del día 8 al día 7 del mes siguiente. La suscripción se transfirió a un Contrato de cliente de Microsoft el 16 de noviembre. La última factura de pago por uso incluye los cargos del 8 de octubre de 2020 al 7 de noviembre de 2020. También incluye los cargos del nuevo ciclo de facturación parcial del Contrato de cliente de Microsoft del 8 de noviembre de 2020 al 16 de noviembre de 2020. A continuación se muestra una imagen de ejemplo.

:::image type="content" source="./media/mca-understand-your-invoice/last-invoice-billing-cycle.png" alt-text="Imagen de ejemplo de una factura que muestra el último ciclo de facturación." lightbox="./media/mca-understand-your-invoice/last-invoice-billing-cycle.png" :::

## <a name="invoice-terms-and-descriptions"></a>Términos y descripciones de la factura

Las siguientes secciones enumeran los términos importantes que ve en la factura y proporciona las descripciones de cada uno.

### <a name="invoice-summary"></a>Resumen de factura

El **Resumen de factura** se encuentra en la parte superior de la primera página y muestra información sobre el perfil de facturación y la forma de pago.

![Sección Resumen de factura](./media/mca-understand-your-invoice/invoicesummary.png)

| Término | Descripción |
| --- | --- |
| Dirección de venta |Dirección de la persona jurídica; se encuentra en las propiedades de la cuenta de facturación|
| Dirección de facturación |Dirección de facturación del perfil de facturación que recibe la factura; se encuentra en las propiedades del perfil de facturación|
| Perfil de facturación |El nombre del perfil de facturación que recibe la factura |
| Pedido de compra number |Un número de pedido de compra opcional asignado por usted para el seguimiento |
| Número de factura |Un número de factura único generado por Microsoft usado para fines de seguimiento |
| Fecha de la factura |Fecha en la que se generó la factura, normalmente entre 5 y 12 días después del final del ciclo de facturación. Puede consultar la fecha de la factura en las propiedades del perfil de facturación.|
| Condiciones de pago |Forma de pago de su factura de Microsoft. *30 días naturales* significa que se paga en el plazo de 30 días desde la fecha de la factura. |

### <a name="billing-summary"></a>Resumen de facturación

El **Resumen de facturación** muestra los cargos en el perfil de facturación desde el período de facturación anterior, los créditos aplicados, los impuestos y el importe total debido.

![Sección Resumen de facturación](./media/mca-understand-your-invoice/billingsummary.png)

| Término | Descripción |
| --- | --- |
| Charges|Número total de cargos de Microsoft para este perfil de facturación desde el último período de facturación |
| Créditos |Créditos que ha recibido de devoluciones |
| Crédito de Azure aplicados | Créditos de Azure que se aplican automáticamente a cargos de Azure cada período de facturación |
| Subtotal |Importe debido antes de impuestos |
| Impuesto |El tipo y el importe del impuesto que pagar, según el país o región de su perfil de facturación. Si no tiene que pagar impuestos, no verá impuestos en la factura. |
| Ahorro total estimado |La cantidad total estimada que ahorra con descuentos efectivos. Si procede, las tarifas de descuento efectivas aparecen debajo de los elementos de línea de compra en detalles de la sección de factura. |

### <a name="invoice-sections"></a>Secciones de la factura

Para cada sección de la factura en su perfil de facturación, verá los cargos, la cantidad de créditos de Azure aplicados, los impuestos y el importe total que pagar.

`Total = Charges - Azure Credit + Tax`

### <a name="details-by-invoice-section"></a>Detalles por sección de la factura

Los detalles muestran el costo para cada sección de la factura desglosado por pedido de producto. Dentro de cada pedido de producto, el costo se desglosa por tipo de servicio. Puede encontrar los cargos diarios de sus productos y servicios en Azure Portal y en el archivo CSV de cargos y uso de Azure. Para más información, consulte [Descripción de los cargos en la factura del contrato de cliente de Microsoft](review-customer-agreement-bill.md).

El importe total debido de cada familia de servicios se calcula restando *créditos de Azure* de *créditos/cargos* y sumando *impuestos*:


![Detalles por sección de la factura](./media/mca-understand-your-invoice/invoicesectiondetails.png)

| Término |Descripción |
| --- | --- |
| Precio unitario | El precio por unidad efectivo del servicio (en la moneda de determinación del precio) que se usa para tasar el uso. Este precio es único para cada producto, familia de servicios, medidor y oferta. |
| Cantidad | Cantidad comprada o consumida durante el período de facturación |
| Cargos/créditos | Importe neto de cargos después de aplicar créditos y reembolsos |
| Crédito de Azure | La cantidad de créditos de Azure aplicados a los cargos/créditos|
| Tipo impositivo | Tipos impositivos según el país o región |
| Importe de impuestos | Importe de impuesto aplicado a la compra según el tipo impositivo |
| Total | El importe total debido de la compra |

### <a name="how-to-pay"></a>Forma de pago

En la parte inferior de la factura, se incluyen instrucciones para pagar la factura. Puede pagar mediante cheque, transferencia o en línea. Si paga en línea, puede usar una tarjeta de crédito o créditos de Azure, si procede.

### <a name="publisher-information"></a>Información del anunciante

Si tiene servicios de terceros en su factura, el nombre y la dirección de cada editor se muestra en la parte inferior de la factura.

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los cargos en la factura del perfil de facturación](review-customer-agreement-bill.md)
- [Obtención de los datos de uso diario y la factura de Azure](../manage/download-azure-invoice-daily-usage-date.md)
- [Visualización de los precios de Azure de su organización](../manage/ea-pricing.md)
- [Visualización de los documentos fiscales de su perfil de facturación](mca-download-tax-document.md)
