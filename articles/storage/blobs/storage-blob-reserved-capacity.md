---
title: Optimización de los costos de almacenamiento de blobs con capacidad reservada
titleSuffix: Azure Storage
description: Obtenga información sobre cómo comprar la capacidad reservada de Azure Storage para ahorrar costos en blobs en bloques y recursos de Azure Data Lake Storage Gen2.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/17/2021
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 5f130e2367ed2fa4c9095479e9ff9d9722b010e5
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110098549"
---
# <a name="optimize-costs-for-blob-storage-with-reserved-capacity"></a>Optimización de los costos de almacenamiento de blobs con capacidad reservada

Puede ahorrar dinero en costos de almacenamiento de datos de blobs con la capacidad reservada de Azure Storage. La capacidad reservada de Azure Storage ofrece un descuento en la capacidad para los blobs en bloques y los datos de Azure Data Lake Storage Gen2 en las cuentas de almacenamiento estándar cuando se compromete a una reserva durante un año o tres años. Una reserva proporciona una cantidad fija de capacidad de almacenamiento para el plazo de la reserva.

La capacidad reservada de Azure Storage puede disminuir considerablemente los costos de capacidad de los blobs en bloques y los datos de Azure Data Lake Storage Gen2. El ahorro de costos logrado depende de la duración de la reserva, la capacidad total que elija reservar y el nivel de acceso y el tipo de redundancia que haya elegido para la cuenta de almacenamiento. La capacidad reservada ofrece un descuento en la facturación y no afecta el estado de sus recursos de Azure Storage.

Para información sobre los precios de reserva de Azure Storage, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/) y [Precios de Azure Data Lake Storage Gen 2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

## <a name="reservation-terms-for-azure-storage"></a>Términos de la reserva de Azure Storage

En las secciones siguientes se describen los términos de una reserva de Azure Storage.

### <a name="reservation-capacity"></a>Capacidad de reserva

Puede comprar capacidad reservada de Azure Storage en unidades de 100 TiB y 1 PiB al mes durante un período de uno o tres años.

### <a name="reservation-scope"></a>Ámbito de la reserva

La capacidad reservada de Azure Storage está disponible en una suscripción única o para varias suscripciones (ámbito compartido). Cuando el ámbito es de una suscripción única, el descuento por reserva se aplica a la suscripción seleccionada únicamente. Cuando el ámbito es de varias suscripciones, el descuento de la reserva se comparte entre esas suscripciones según el contexto de facturación del cliente.

Al comprar la capacidad reservada de Azure Storage, puede usar la reserva para los datos de blobs en bloques y de Azure Data Lake Storage Gen2. Una reserva se aplica al uso dentro del ámbito adquirido y no se puede limitar a una cuenta de almacenamiento, un contenedor o un objeto específico de la suscripción.

Una reserva de Azure Storage solo cubre la cantidad de datos que se almacenan en una suscripción o un grupo de recursos compartidos. Los cargos iniciales por transferencia de datos, ancho de banda, operaciones y eliminación no se incluyen en la reserva. En cuanto se compra una reserva, los cargos de capacidad que coincidan con los atributos de reserva se cobran según las tarifas de descuento en lugar de las tarifas de pago por uso. Para más información sobre las reservas de Azure, consulte [¿Qué es Azure Reservations?](../../cost-management-billing/reservations/save-compute-costs-reservations.md).

### <a name="supported-account-types-tiers-and-redundancy-options"></a>Opciones de redundancia, niveles y tipos de cuentas compatibles

La capacidad reservada de Azure Storage está disponible para los recursos de las cuentas de almacenamiento estándar, incluidas las cuentas de uso general v2 (GPv2) y de almacenamiento de blobs.

Todos los niveles de acceso (frecuente, esporádicoy de archivo) se admiten para las reservas. Para más información sobre los niveles de acceso, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

Se admiten todos los tipos de redundancia para las reservas. Para más información sobre las opciones de redundancia, consulte [Redundancia de Azure Storage](../common/storage-redundancy.md).

> [!NOTE]
> La capacidad reservada de Azure Storage no está disponible para cuentas de almacenamiento prémium, cuentas de almacenamiento de uso general v1 (GPv1), Azure Data Lake Storage Gen1, blobs en páginas, Azure Queue Storage o Azure Table. Para más información sobre la capacidad reservada de Azure Files, consulte [Optimización de costos para Azure Files con capacidad reservada](../files/files-reserve-capacity.md).  

### <a name="security-requirements-for-purchase"></a>Requisitos de seguridad para la compra

Para adquirir capacidad reservada:

- Debe tener el rol **Propietario** al menos en una suscripción Enterprise o individual con tarifas de pago por uso.
- En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el portal de EA. O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso del programa del Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden comprar capacidad reservada de Azure Blob Storage.

## <a name="determine-required-capacity-before-purchase"></a>Determinación de la capacidad necesaria antes de la compra

Cuando compra una reserva de Azure Storage, debe elegir la opción de región, nivel de acceso y redundancia para la reserva. La reserva solo es válida para los datos almacenados en esa región, el nivel de acceso y el nivel de redundancia. Por ejemplo, supongamos que adquiere una reserva de datos en la región Oeste de EE. UU. para el nivel de acceso frecuente mediante el almacenamiento con redundancia de zona (ZRS). No se puede usar la misma reserva para datos en la región Este de EE. UU., datos en el nivel de archivo o datos en almacenamiento con redundancia geográfica (GRS). Sin embargo, puede adquirir otra reserva para sus necesidades adicionales.  

Las reservas están disponibles hoy para bloques de 100 TiB o 1 PiB, con descuentos superiores para los bloques de 1 PiB. Cuando se adquiere una reserva en Azure Portal, Microsoft puede proporcionarle recomendaciones basadas en el uso anterior para ayudar a determinar qué reserva debe comprar.

## <a name="purchase-azure-storage-reserved-capacity"></a>Compra de capacidad reservada de Azure Storage

Puede adquirir capacidad reservada de Azure Storage a través de [Azure Portal](https://portal.azure.com). Pague la reserva por adelantado o de manera mensual. Para más información sobre cómo comprar con pagos mensuales, consulte [Compra de reservas de Azure con pagos por adelantado o mensuales](../../cost-management-billing/reservations/prepare-buy-reservation.md).

Para ayudar a identificar los términos de reserva adecuados para su escenario, consulte el artículo sobre cómo [entender el descuento para la capacidad reservada de Azure Storage](../../cost-management-billing/reservations/understand-storage-charges.md).

Siga estos pasos para adquirir la capacidad reservada:

1. Vaya al panel [Comprar reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/Browse_AddCommand) en Azure Portal.  
1. Seleccione **Azure Blob Storage** para comprar una reserva nueva.  
1. Rellene los campos obligatorios tal como se describe en la tabla siguiente:

    ![Captura de pantalla que muestra cómo comprar capacidad reservada](media/storage-blob-reserved-capacity/select-reserved-capacity.png)

   |Campo  |Descripción  |
   |---------|---------|
   |**Ámbito**   |  Indica el número de suscripciones que pueden usar la ventaja de facturación asociada con la reserva. También controla cómo se aplica la reserva a suscripciones concretas. <br/><br/> Si selecciona **Compartido**, el descuento de la reserva se aplica a la capacidad de Azure Storage en cualquier suscripción en el contexto de facturación. El contexto de facturación se basa en cómo se haya suscrito a Azure. Para los clientes Enterprise, el ámbito compartido es la inscripción e incluye todas las suscripciones que esta contiene. Para los clientes de pago por uso, el ámbito compartido incluye todas las suscripciones con tarifas de pago por uso creadas por el administrador de la cuenta.  <br/><br/>  Si selecciona **Suscripción única**, el descuento de reserva se aplica a la capacidad de Azure Storage de la suscripción seleccionada. <br/><br/> Si selecciona **Grupo de recursos único**, el descuento de reserva se aplica a la capacidad de Azure Storage de la suscripción seleccionada y al grupo de recursos seleccionado dentro de esa suscripción. <br/><br/> Puede cambiar el ámbito de reserva después de comprar la reserva.  |
   |**Suscripción**  | La suscripción que se usa para pagar la reserva de Azure Storage. El método de pago en la suscripción seleccionada se usa al cargar los costos. La suscripción debe ser uno de los tipos siguientes: <br/><br/>  Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P): En el caso de una suscripción Enterprise, los cargos se deducirán del saldo de pago por adelantado de la inscripción de Azure (anteriormente llamado compromiso monetario) o se cobrarán como parte del uso por encima del límite. <br/><br/> Suscripción individual con tarifas de pago por uso (números de la oferta: MS-AZR-0003P o MS-AZR-0023P): en una suscripción individual con tarifas de pago por uso, los cargos se cobran en el método de pago de tarjeta de crédito o factura de la suscripción.    |
   | **Región** | La región donde está en vigor la reserva. |
   | **Nivel de acceso** | El nivel de acceso donde está en vigor la reserva. Entre las opciones se incluyen *Frecuente*, *Esporádico* o *De archivo*. Para más información acerca de los niveles de acceso, consulte [Azure Blob Storage: niveles de almacenamiento de archivo, esporádico y frecuente](storage-blob-storage-tiers.md). |
   | **Redundancia** | La opción de redundancia para la reserva. Entre las opciones se incluyen *LRS*, *ZRS*, *GRS*, *GZRS*, *RA-GRS* y *RA-GZRS*. Para más información sobre las opciones de redundancia, consulte [Redundancia de Azure Storage](../common/storage-redundancy.md). |
   | **Frecuencia de facturación** | Indica la frecuencia con la que se factura la cuenta para la reserva. Entre las opciones se incluyen *Mensual* o *Por adelantado*. |
   | **Tamaño** | Cantidad de capacidad que se va a reservar. |
   |**Término**  | Un año o tres años.   |

1. Después de seleccionar los parámetros para la reserva, Azure Portal muestra el costo. El portal también muestra el porcentaje de descuento en la facturación de pago por uso.

1. En el panel **Compra de reservas**, revise el costo total de la reserva. También puede proporcionar un nombre para la reserva.

    ![Captura de pantalla que muestra cómo adquirir una reserva](media/storage-blob-reserved-capacity/purchase-reservations.png)

Después de comprar una reserva, se aplica automáticamente a cualquier blob en bloques de Azure Storage existente o recursos de Azure Data Lake Storage Gen2 que coincidan con los términos de la reserva. Si aún no ha creado ningún recurso de Azure Storage, se aplicará la reserva siempre que cree un recurso que coincida con los términos de la reserva. En cualquier caso, el período de la reserva comienza inmediatamente después de una compra correcta.

## <a name="exchange-or-refund-a-reservation"></a>Intercambio o reembolso de una reserva

Puede intercambiar o reembolsar una reserva, con ciertas limitaciones que se describen en las secciones siguientes.

Para intercambiar o reembolsar una reserva, vaya a los detalles de la reserva en Azure Portal. Seleccione **Intercambiar** o **Reembolsar** y siga las instrucciones para enviar una solicitud de soporte técnico. Una vez procesada la solicitud, Microsoft le enviará un correo electrónico para confirmar que se completó la solicitud.

Para más información sobre las directivas de reservas de Azure, consulte [Autoservicio de cambios y reembolsos de reservas de Azure](../../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

### <a name="exchange-a-reservation"></a>Intercambio de una reserva

El intercambio de una reserva le permite recibir un reembolso prorrateado en función de la parte no utilizada de la reserva. Después, puede aplicar el reembolso al precio de compra de una nueva reserva de Azure Storage.

No hay límite en el número de intercambios que puede hacer. Además, los intercambios no conllevan ninguna tarifa. La reserva nueva que compre debe ser igual o mayor que el crédito prorrateado de la reserva original. Una reserva de Azure Storage se puede intercambiar solo para otra reserva de Azure Storage y no para una reserva de ningún otro servicio de Azure.

### <a name="refund-a-reservation"></a>Reembolso de una reserva

Puede cancelar una reserva de Azure Storage en cualquier momento. Cuando lo haga, recibirá un reembolso prorrateado según el período restante de la reserva, menos una cuota de finalización temprana del 12 %. El reembolso máximo por año es de USD 50 000.

Al cancelar una reserva, esta se finaliza inmediatamente y se devuelven los meses restantes a Microsoft. El saldo prorrateado restante, menos la cuota, se reembolsará en la forma de compra original.

## <a name="expiration-of-a-reservation"></a>Expiración de una reserva

Cuando expira una reserva, cualquier capacidad de Azure Storage que esté usando en virtud de esa reserva se factura según la tarifa de pago por uso. Las reservas no se renuevan automáticamente.

Recibirá una notificación por correo electrónico 30 días antes de la expiración de la reserva y nuevamente en la fecha de expiración. Para seguir aprovechando el ahorro de costos que proporciona una reserva, debe renovarla a más tardar en la fecha de expiración.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure Reservations?](../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Aplicación del descuento por reserva a Azure Storage](../../cost-management-billing/reservations/understand-storage-charges.md)
