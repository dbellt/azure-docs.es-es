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
ms.openlocfilehash: fda22346a44388248e37473bc7891b8a130569c4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81681058"
---
1. En la página **Create a Peering** (Crear un emparejamiento), en la pestaña **Configuration** (Configuración), rellene los cuadros como se muestra aquí.

    * En **Tipo de emparejamiento**, seleccione **Directo**.
    * En **Microsoft network** (Red de Microsoft), seleccione **AS8075**. No cree emparejamiento con ASN 8069. Está reservada para aplicaciones especiales y solo la usa el [servicio de emparejamiento de Microsoft](mailto:peering@microsoft.com).
    * Seleccione **SKU** como **Gratis Básico**. No seleccione Premium Free (Gratis Premium) porque está reservada para aplicaciones especiales.
    * Seleccione la ubicación **Metro** (Área metropolitana) donde quiere configurar el emparejamiento.

        > [!NOTE]
        > Si ya tiene conexiones de emparejamiento con Microsoft en la ubicación **Metro** (Área metropolitana) seleccionada y es la primera vez que usa Azure Portal para configurar el emparejamiento en esa ubicación, las conexiones de emparejamiento existentes se enumerarán en la sección **Peering connections** (Conexiones de emparejamiento), como se muestra a continuación. Microsoft convertirá automáticamente estas conexiones de emparejamiento en un recurso de Azure para que pueda administrarlas todas, junto con las nuevas conexiones, en un solo lugar. Para más información, consulte [Conversión de un emparejamiento directo heredado en un recurso de Azure mediante el portal](../howto-legacy-direct-portal.md).
        >

1. En **Peering connections** (Conexiones de emparejamiento), seleccione **Create new** (Crear nuevo) para agregar una línea para cada nueva conexión que quiera configurar.

    * Para configurar o modificar la configuración de conexión, seleccione el botón Edit (Editar) de una línea.

        > [!div class="mx-imgBorder"]
        > ![Botón Edit (Editar)](../media/setup-direct-conf-tab-edit.png)
    
    * Para eliminar una línea, seleccione **...**  > **Delete** (Eliminar).

        > [!div class="mx-imgBorder"]
        > ![Botón Delete (Eliminar)](../media/setup-direct-conf-tab-delete.png)

    * Se le pedirá que proporcione toda la configuración de una conexión, como se muestra a continuación.

         > [!div class="mx-imgBorder"]
         > ![Página Direct Peering Connection (Conexión de emparejamiento directo)](../media/setup-direct-conf-tab-connection.png)

        1. Seleccione la **instalación de emparejamiento** en la que debe configurarse la conexión.
        1. El **proveedor de direcciones de sesión** se usa para determinar quién proporciona la subred necesaria para configurar la sesión de BGP entre la red y Microsoft. Si puede proporcionar la subred, seleccione **Peer** (Del mismo nivel). De lo contrario, elija **Microsoft** y el [servicio de emparejamiento de Microsoft](mailto:peering@microsoft.com) se pondrá en contacto con usted. Si elige esta opción, Microsoft tardará más tiempo en procesar la solicitud de emparejamiento. En algunos casos, es posible que Microsoft no pueda proporcionar subredes, lo que producirá una denegación de la solicitud.
        1. Si ha seleccionado la opción **Session Address Provider** (Proveedor de direcciones de sesión) en **Peer** (Del mismo nivel), escriba las direcciones IPv4 e IPv6 junto con las máscaras de prefijo en los cuadros **Session IPv4 prefix** (Prefijo IPv4 de sesión) y **Session IPv6 prefix** (Prefijo IPv6 de sesión), respectivamente.
        1. Escriba el número de prefijos IPv4 e IPv6 que anunciará en los cuadros **Maximum advertised IPv4 addresses** (Máximo de direcciones IPv4 anunciadas) y **Maximum advertised IPv6 addresses** (Máximo de direcciones IPv6 anunciadas), respectivamente.
        1. Ajuste el control deslizante **Total bandwidth** (Ancho de banda total) para reflejar el ancho de banda de la conexión.
        1. Seleccione **Save** (Guardar) para guardar la configuración de la conexión.

1. Repita el paso anterior para agregar más conexiones en cualquier instalación donde Microsoft esté colocado con la red, dentro de **Metro** (Área metropolitana) seleccionada previamente.

1. Después de agregar todas las conexiones necesarias, seleccione **Review + create** (Revisar + crear).

    > [!div class="mx-imgBorder"]
    > ![Pestaña Peering Configuration (Configuración de emparejamiento) final](../media/setup-direct-conf-tab-final.png)

1. Observe que el portal ejecuta una validación básica de la información especificada. En la cinta de opciones de la parte superior se muestra el mensaje *Running final validation...* (Ejecutando la validación final...).

    > [!div class="mx-imgBorder"]
    > ![Pestaña Peering Validation (Validación del emparejamiento)](../media/setup-direct-review-tab-validation.png)

1. Cuando el mensaje cambie a *Validation passed* (Validación superada), compruebe la información. Para enviar la solicitud, seleccione **Create** (Crear). Para modificar la solicitud, seleccione **Previous** (Anterior) y repita los pasos.

    > [!div class="mx-imgBorder"]
    > ![Envío del emparejamiento](../media/setup-direct-review-tab-submit.png)

1. Después de enviar la solicitud, espere a que finalice la implementación. Si se produce un error en la implementación, póngase en contacto con el [equipo de emparejamiento de Microsoft](mailto:peering@microsoft.com). Una implementación correcta aparece como se muestra aquí.

    > [!div class="mx-imgBorder"]
    > ![Emparejamiento correcto](../media/setup-direct-success.png)
