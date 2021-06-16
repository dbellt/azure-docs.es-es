---
title: Adición de una audiencia preliminar para la oferta de servicio administrado
description: Aprenda a agregar una audiencia preliminar para la oferta de servicio administrado en el Centro de partners de Microsoft.
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.reviewer: anbene
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 12/23/2020
ms.openlocfilehash: a30265f335dd279470e6de421f83e682a304788f
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060514"
---
# <a name="how-to-add-a-preview-audience-for-your-managed-service-offer"></a>Adición de una audiencia preliminar para la oferta de servicio administrado

En este artículo se describe cómo configurar una audiencia preliminar para una oferta de servicio administrado en el marketplace comercial mediante el Centro de partners. La audiencia preliminar puede revisar la oferta antes de que se publique.

## <a name="define-a-preview-audience"></a>Definición de una audiencia preliminar

En la pestaña **Preview audience** (Audiencia preliminar), puede definir una audiencia limitada que revise la oferta de servicio administrado antes de publicarla para la audiencia más amplia del marketplace. Especifique la audiencia preliminar mediante los identificadores de suscripción de Azure, junto con una descripción opcional de cada uno. Los clientes no pueden ver ninguno de estos campos. Puede encontrar el identificador de la suscripción en la página **Suscripciones** de Azure Portal.

Agregue al menos un identificador de suscripción de Azure, bien de forma individual (hasta 10) o mediante la carga de un archivo CSV (hasta 100) para definir quién recibirá una versión preliminar de la oferta antes de que se publique. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar las actualizaciones de esta.

> [!NOTE]
> Una audiencia *preliminar* no es lo mismo que una audiencia *privada*. A una audiencia preliminar se le permite el acceso a la oferta antes de que se publique en las tiendas en línea. Estas personas podrán ver y validar todos los planes, incluidos aquellos que estarán disponibles solo para una audiencia privada después de que su oferta se publique completamente en el marketplace. Puede hacer que un plan esté disponible solo para una audiencia privada. Una audiencia privada (definida en la pestaña Disponibilidad del plan) tiene acceso exclusivo a un plan determinado.

## <a name="add-email-addresses-manually"></a>Agregar direcciones de correo electrónico manualmente

1. En la página **Preview audience** (Audiencia preliminar), agregue un único identificador de suscripción de Azure y una descripción opcional en los cuadros proporcionados.
2. Para agregar otra dirección de correo electrónico, seleccione el vínculo **Add ID (Max 10)** (Agregar identificador [10 como máximo]).
3. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña.

## <a name="add-email-addresses-using-a-csv-file"></a>Adición de direcciones de correo electrónico mediante un archivo CSV

1. En la página **Preview audience** (Audiencia preliminar), seleccione el vínculo **Export Audience (csv)** (Exportar audiencia [csv]).
2. Abra el archivo CSV. En la columna **Id.** , escriba los identificadores de la suscripción de Azure que quiere agregar a la audiencia preliminar.
3. En la columna **Descripción**, tiene la opción de agregar una descripción para cada entrada.
4. En la columna **Tipo**, agregue **SubscriptionID** a cada fila que tenga un identificador.
5. Guarde el archivo como un archivo CSV.
6. En la página **Preview audience** (Audiencia preliminar), seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
7. En el cuadro de diálogo **Confirmar**, seleccione **Sí** y, luego, cargue el archivo CSV.
8. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un plan en Azure Stack Hub](create-managed-service-offer-plans.md)
