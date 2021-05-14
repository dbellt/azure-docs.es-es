---
title: Aplicación del descuento por reserva a Azure Storage | Microsoft Docs
description: Aprenda cómo se aplican los descuentos de capacidad reservada a los recursos de Azure Blob Storage, Azure Files y Azure Data Lake Storage Gen2.
author: tamram
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 03/08/2021
ms.author: tamram
ms.openlocfilehash: f18097f0da658b3a485b79f5279e9ecc54a41ea0
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/24/2021
ms.locfileid: "105024037"
---
# <a name="understand-how-reservation-discounts-are-applied-to-azure-storage-services"></a>Aplicación del descuento por reserva a Azure Storage 
Los servicios de almacenamiento de Azure permiten ahorrar con los costos de almacenamiento al reservar capacidad. Azure Blob Storage, Azure Files y Azure Data Lake Storage Gen2 admiten las instancias reservadas. Después de comprar capacidad reservada, el descuento por reserva se aplica automáticamente a los recursos de almacenamiento que coincidan con los términos de la reserva. El descuento por reserva se aplica solo a la capacidad de almacenamiento. El ancho de banda y la tasa de solicitudes se cobran según las tarifas de pago por uso.

Para más información acerca de la capacidad reservada de Azure Blob Storage y Azure Data Lake Storage Gen2, consulte [Optimización de los costos de almacenamiento de blobs con capacidad reservada](../../storage/blobs/storage-blob-reserved-capacity.md). Para más información acerca de la capacidad reservada de Azure Files, consulte [Optimización de costos para Azure Files con capacidad reservada](../../storage/files/files-reserve-capacity.md).

Para información sobre los precios de reserva de Azure Blob Storage, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/) y [Precios de Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/). Para información sobre los precios de la reserva de almacenamiento de Azure Files, consulte los [Precios de Azure Files](https://azure.microsoft.com/pricing/details/storage/files).

## <a name="how-the-reservation-discount-is-applied"></a>Aplicación del descuento por reserva
El descuento por la capacidad reservada se aplica por hora a los recursos de almacenamiento admitidos.

El descuento por la capacidad reservada es un descuento para "usarlo o perderlo". Si no tiene recursos de blob en bloques, recursos compartidos de archivos de Azure, ni recursos de Azure Data Lake Storage Gen2 que cumplan los términos de la reserva para una hora determinada, perderá una cantidad de reserva durante esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al eliminar un recurso, el descuento por reserva se aplica automáticamente a otro que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se pierden.

## <a name="discount-examples"></a>Ejemplos de descuento
En los ejemplos siguientes se muestra cómo se aplica el descuento por la capacidad reservada en función de las implementaciones.

Supongamos que ha adquirido 100 TiB de capacidad reservada en la región Oeste de EE. UU. 2 durante un período de un año. La reserva es para el almacenamiento con redundancia local (LRS) en el nivel de acceso frecuente del almacenamiento de blobs.

Supongamos que el costo de esta reserva de ejemplo es de 18 540 $. Puede optar por pagar la cantidad completa por adelantado o pagar las instalaciones mensuales fijas de 1.545 $ al mes durante los próximos 12 meses.

En estos ejemplos, se supone que se ha registrado para un plan de pago de reserva mensual. En los siguientes escenarios se describe lo que sucede si el uso excede o no llega a la capacidad reservada.

### <a name="underusing-your-capacity"></a>Uso por debajo de la capacidad
Supongamos que en una hora determinada dentro del período de reserva, solo usó 80 TiB de la capacidad reservada de 100 TiB. Los 20 TiB restantes no se aplican durante esa hora y no se usan.

### <a name="overusing-your-capacity"></a>Uso por encima de la capacidad
Supongamos que en una hora determinada dentro del período de reserva, usó 101 TiB de la capacidad de almacenamiento. El descuento por reserva se aplica a los 100 TiB de datos y el TiB restante se cobra según las tarifas de pago por uso durante esa hora. Si, en la siguiente hora, el uso cambia a 100 TiB, todo el uso está incluido en la reserva.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros
Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- [Optimización de los costos de almacenamiento de blobs con capacidad reservada](../../storage/blobs/storage-blob-reserved-capacity.md)
- [Optimización de costos para Azure Files con capacidad reservada](../../storage/files/files-reserve-capacity.md)
- [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
