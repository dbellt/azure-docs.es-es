---
title: Establezca la audiencia preliminar de una oferta de contenedor de Azure en Microsoft AppSource.
description: Establezca la audiencia preliminar de una oferta de contenedor de Azure en Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 03/30/2021
ms.openlocfilehash: 0d8bd02e247a3db0429f227e396abbfbc7cdfd91
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107892548"
---
# <a name="set-the-preview-audience-for-an-azure-container-offer"></a>Establecimiento de la audiencia preliminar de una oferta de contenedor de Azure

En este artículo se describe cómo configurar una audiencia preliminar para una oferta de contenedor de Azure en el marketplace comercial mediante el Centro de partners. La audiencia preliminar puede revisar la oferta antes de que se publique.

## <a name="define-a-preview-audience"></a>Definición de una audiencia preliminar

En la página **Vista previa de la audiencia**, defina una audiencia limitada que pueda revisar la oferta de contenedor antes de publicarla para la audiencia más amplia del marketplace. Especifique la audiencia preliminar mediante los identificadores de suscripción de Azure, junto con una descripción opcional de cada uno. Los clientes no pueden ver ninguno de estos campos. Puede encontrar el identificador de la suscripción en la página **Suscripciones** de Azure Portal.

Agregue al menos un identificador de suscripción de Azure, bien de forma individual (hasta 10) o mediante la carga de un archivo CSV (hasta 100) para definir quién podrá obtener una versión preliminar de la oferta. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar las actualizaciones de esta.

## <a name="add-email-addresses-manually"></a>Agregar direcciones de correo electrónico manualmente

1. En la página **Preview audience** (Audiencia preliminar), agregue un único identificador de suscripción de Azure y una descripción opcional en los cuadros proporcionados.
1. Para agregar otra dirección de correo electrónico, seleccione el vínculo **Add ID (Max 10)** (Agregar identificador [10 como máximo]).
1. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña para configurar los planes.

## <a name="add-email-addresses-using-a-csv-file"></a>Adición de direcciones de correo electrónico mediante un archivo CSV

1. En la página **Preview audience** (Audiencia preliminar), seleccione el vínculo **Export Audience (csv)** (Exportar audiencia [csv]).
1. Abra el archivo CSV. En la columna **Id.** , escriba los identificadores de la suscripción de Azure que quiere agregar a la audiencia preliminar.
1. En la columna **Descripción**, tiene la opción de agregar una descripción para cada entrada.
1. En la columna **Tipo**, agregue **SubscriptionID** a cada fila que tenga un identificador.
1. Guarde el archivo como un archivo CSV.
1. En la página **Preview audience** (Audiencia preliminar), seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
1. En el cuadro de diálogo **Confirmar**, seleccione **Sí** y, luego, cargue el archivo CSV.
1. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña para configurar los planes.

> [!IMPORTANT]
> Después de ver la oferta en Versión preliminar, debe seleccionar **Transmitir** para publicar su oferta para el público.

## <a name="next-steps"></a>Pasos siguientes

- [Creación y administración de planes](azure-container-plan-overview.md)
