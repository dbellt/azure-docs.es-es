---
title: archivo de inclusión
titleSuffix: Azure
description: archivo de inclusión
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 632490498b8dd13414657edb9518cd543ac07af6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81678855"
---
1. En la página **Create a Peering** (Crear un emparejamiento), en la pestaña **Configuration** (Configuración), rellene los cuadros como se muestra aquí.

    > [!div class="mx-imgBorder"]
    > ![Pestaña Configuration (Configuración) de la página Create a Peering (Crear un emparejamiento)](../media/setup-direct-conf-tab.png)

    * En **Tipo de emparejamiento**, seleccione **Directo**.
    * En **Microsoft network** (Red de Microsoft), seleccione **AS8075**. No seleccione ASN 8069. Está reservada para aplicaciones especiales y solo la usa el [servicio de emparejamiento de Microsoft](mailto:peering@microsoft.com).
    * Seleccione **SKU** como **Gratis Básico**. No seleccione Premium Free (Gratis Premium) porque está reservada para aplicaciones especiales.
    * Seleccione la ubicación **Metro** (Área metropolitana) donde quiere convertir el emparejamiento en un recurso de Azure. Si tiene conexiones de emparejamiento con Microsoft en la ubicación **Metro** (Área metropolitana) seleccionada que no se han convertido en un recurso de Azure, dichas conexiones aparecerán en la sección **Peering connections** (Conexiones de emparejamiento), como se muestra a continuación. Ahora puede convertir estas conexiones de emparejamiento en un recurso de Azure.

        > [!div class="mx-imgBorder"]
        > ![Lista de conexiones de emparejamiento](../media/setup-directlegacy-conf-tab.png)

1. Si necesita actualizar el ancho de banda, seleccione el botón Edit (Editar) de una línea para modificar la configuración de la conexión.

    > [!div class="mx-imgBorder"]
    > ![Botón Edit (Editar)](../media/setup-directlegacy-conf-tab-edit.png)

    > [!NOTE]
    > Si quiere agregar conexiones de emparejamiento adicionales con Microsoft en la ubicación **Metro** (Área metropolitana) seleccionada, seleccione **Create new** (Crear nuevo). Para más información, consulte [Creación o modificación de un emparejamiento directo mediante el portal](../howto-direct-portal.md).
    >

1. Seleccione **Revisar + crear**. Observe que el portal ejecuta una validación básica de la información especificada. En la cinta de opciones de la parte superior se muestra el mensaje *Running final validation...* (Ejecutando la validación final...).

    > [!div class="mx-imgBorder"]
    > ![Pestaña Peering Validation (Validación del emparejamiento)](../media/setup-direct-review-tab-validation.png)

1. Cuando el mensaje cambie a *Validation passed* (Validación superada), compruebe la información. Para enviar la solicitud, seleccione **Create** (Crear). Para modificar la solicitud, seleccione **Previous** (Anterior) y repita los pasos.

    > [!div class="mx-imgBorder"]
    > ![Envío del emparejamiento](../media/setup-direct-review-tab-submit.png)

1. Después de enviar la solicitud, espere a que finalice la implementación. Si se produce un error en la implementación, póngase en contacto con el [equipo de emparejamiento de Microsoft](mailto:peering@microsoft.com). Una implementación correcta aparece como se muestra aquí.

    > [!div class="mx-imgBorder"]
    > ![Emparejamiento correcto](../media/setup-direct-success.png)
