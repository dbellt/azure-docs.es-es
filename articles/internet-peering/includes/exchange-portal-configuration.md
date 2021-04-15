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
ms.openlocfilehash: 268703081a2a40e8bcc665889eaeaf8edd673bfd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81680938"
---
1. En la página **Create a Peering** (Crear un emparejamiento), en la pestaña **Configuration** (Configuración), rellene los cuadros como se muestra.

    > [!div class="mx-imgBorder"]
    > ![Página Create a Peering (Crear un emparejamiento): tipo de emparejamiento de Exchange](../media/setup-exchange-conf-tab.png)

    * En **Tipo de emparejamiento**, seleccione **Exchange**.
    * Seleccione **SKU** como **Gratis Básico**.
    * Seleccione la ubicación **Metro** (Área metropolitana) donde quiere configurar el emparejamiento.

        > [!NOTE]
        > Si ya tiene conexiones de emparejamiento con Microsoft en la ubicación **Metro** (Área metropolitana) seleccionada y es la primera vez que usa el portal para configurar el emparejamiento en esa ubicación, las conexiones de emparejamiento existentes se enumerarán en la sección **Peering connections** (Conexiones de emparejamiento), como se muestra a continuación. Microsoft convertirá automáticamente estas conexiones de emparejamiento en un recurso de Azure para que pueda administrarlas todas, junto con las nuevas conexiones, en un solo lugar. Para más información, consulte [Conversión de un emparejamiento de Exchange heredado a un recurso de Azure mediante el portal](../howto-legacy-exchange-portal.md).
        >

1. En **Peering connections** (Conexiones de emparejamiento), seleccione **Create new** (Crear nuevo) para agregar una línea para cada nueva conexión que quiera configurar.

    * Para configurar o modificar la configuración de conexión, seleccione el botón Edit (Editar) de una línea.

        > [!div class="mx-imgBorder"]
        > ![Botón Edit (Editar)](../media/setup-exchange-conf-tab-edit.png)

    * Para eliminar una línea, seleccione **...**  > **Delete** (Eliminar).

        > [!div class="mx-imgBorder"]
        > ![Botón Delete (Eliminar)](../media/setup-exchange-conf-tab-delete.png)

    * Se le pedirá que proporcione toda la configuración de una conexión, como se muestra a continuación.

         > [!div class="mx-imgBorder"]
         > ![Página Exchange Peering Connection (Conexión de emparejamiento de Exchange)](../media/setup-exchange-conf-tab-connection.png)

        1. Seleccione la **instalación de emparejamiento** en la que debe configurarse la conexión.
        1. En los cuadros **IPv4 address** (Dirección IPv4) y **IPv6 address** (Dirección IPv6), escriba las direcciones IPv4 e IPv6, respectivamente, que se configurarían en los enrutadores de Microsoft con el comando vecino.
        1. Escriba el número de prefijos IPv4 e IPv6 que anunciará en los cuadros **Maximum advertised IPv4 addresses** (Máximo de direcciones IPv4 anunciadas) y **Maximum advertised IPv6 addresses** (Máximo de direcciones IPv6 anunciadas), respectivamente.
        1. Haga clic en **OK** (Aceptar) para guardar la configuración de la conexión.

1. Repita el paso anterior para agregar más conexiones en cualquier instalación donde Microsoft colocado con la red, dentro de la ubicación **Metro** (Área metropolitana) seleccionada anteriormente.

1. Después de agregar todas las conexiones necesarias, seleccione **Review + create** (Revisar + crear).

    > [!div class="mx-imgBorder"]
    > ![Pestaña Peering Configuration (Configuración de emparejamiento)](../media/setup-exchange-conf-tab-final.png)

1. Observe que el portal ejecuta una validación básica de la información especificada. En la cinta de opciones de la parte superior se muestra el mensaje *Running final validation...* (Ejecutando la validación final...).

    > [!div class="mx-imgBorder"]
    > ![Pestaña Peering Validation (Validación del emparejamiento)](../media/setup-direct-review-tab-validation.png)

1. Cuando el mensaje cambie a *Validation passed* (Validación superada), compruebe la información. Para enviar la solicitud, seleccione **Create** (Crear). Para modificar la solicitud, seleccione **Previous** (Anterior) y repita los pasos.

    > [!div class="mx-imgBorder"]
    > ![Envío del emparejamiento](../media/setup-exchange-review-tab-submit.png)

1. Después de enviar la solicitud, espere a que finalice la implementación. Si se produce un error en la implementación, póngase en contacto con el [equipo de emparejamiento de Microsoft](mailto:peering@microsoft.com). Una implementación correcta aparece como se muestra aquí.

    > [!div class="mx-imgBorder"]
    > ![Emparejamiento correcto](../media/setup-direct-success.png)
