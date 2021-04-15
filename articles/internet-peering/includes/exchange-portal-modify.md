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
ms.openlocfilehash: e06b5261ca6923e158c818d236a30cf6ebff189b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81680920"
---
En esta sección se describe cómo realizar las siguientes operaciones de modificación para el emparejamiento directo.

### <a name="add-exchange-peering-connections"></a>Adición de conexiones de emparejamiento de Exchange

1. Seleccione el botón **+ Add connections** (+ Agregar conexiones) y configure una nueva conexión de emparejamiento.
    > [!div class="mx-imgBorder"]
    > ![Vista de recursos de emparejamiento](../media/setup-exchange-modify-addconnection.png)
1. Rellene el formulario de **conexión de emparejamiento de Exchange** y seleccione **Guardar**. Para obtener ayuda con la configuración de una conexión de emparejamiento, revise los pasos descritos en la sección anterior "Creación y aprovisionamiento de un emparejamiento directo".
    > [!div class="mx-imgBorder"]
    > ![Formulario de conexiones de emparejamiento de Exchange](../media/setup-exchange-modify-savenewconnection.png)

### <a name="remove-exchange-peering-connections"></a>Eliminación de conexiones de emparejamiento de Exchange

1. Seleccione una conexión de emparejamiento que quiera eliminar y, luego, elija **...**  > **Delete connection** (Eliminar conexión).
    > [!div class="mx-imgBorder"]
    > ![Botón Delete connection (Eliminar conexión)](../media/setup-exchange-modify-deleteconnection.png)
1. Escriba el identificador de recurso en el cuadro **Confirm Delete** (Confirmar eliminación) y seleccione **Delete** (Eliminar).
    > [!div class="mx-imgBorder"]
    > ![Confirmación de eliminación](../media/setup-exchange-modify-deleteconnectionconfirm.png)

### <a name="add-an-ipv4-or-ipv6-session-on-active-connections"></a>Adición de una sesión IPv4 o IPv6 en conexiones activas

1. Seleccione una conexión de emparejamiento que quiera modificar y, luego, elija **...**  > **Edit connection** (Editar conexión).
    > [!div class="mx-imgBorder"]
    > ![Botón Edit connection (Editar conexión)](../media/setup-exchange-modify-editconnection.png)
1. Agregue la información de **IPv4 address** (Dirección IPv4) o **IPv6 address** (Dirección IPv6) y seleccione **Save** (Guardar).
    > [!div class="mx-imgBorder"]
    > ![Modificación de la conexión de emparejamiento](../media/setup-exchange-modify-editconnectionsettings.png)

### <a name="remove-an-ipv4-or-ipv6-session-on-active-connections"></a>Eliminación de una sesión IPv4 o IPv6 en conexiones activas

La eliminación de una sesión IPv4 o IPv6 de una conexión existente no se admite actualmente en el portal. Para más información, póngase en contacto con el servicio de [emparejamiento de Microsoft](mailto:peeringexperience@microsoft.com).