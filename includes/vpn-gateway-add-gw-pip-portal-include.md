---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 07/16/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e259525bc8215b4542dba86cbe66aaa9ce6d9fbf
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/22/2021
ms.locfileid: "114439721"
---
4. Especifique en los valores de **Dirección IP pública**. estas opciones de configuración especifican el objeto de dirección IP pública que se asocia a la puerta de enlace de VPN. La dirección IP pública se asigna dinámicamente a este objeto cuando se crea la puerta de enlace de VPN. La única vez que la dirección IP pública cambia es cuando la puerta de enlace se elimina y se vuelve a crear. No cambia cuando se cambia el tamaño, se restablece o se realizan actualizaciones u otras operaciones de mantenimiento interno de una puerta de enlace VPN.

   :::image type="content" source="./media/vpn-gateway-add-gw-pip-portal/pip-details.png" alt-text="Captura de pantalla del campo Dirección IP pública.":::

     * **Dirección IP pública**: Mantenga la opción **Crear nueva** seleccionada.
     * **Nombre de dirección IP pública**: En el cuadro de texto, escriba un nombre para la dirección IP pública.
     * **Asignación**: VPN Gateway solo admite Dinámica.
     * **Habilitar el modo activo/activo**: Seleccione **Habilitar el modo activo/activo** solo si va a crear una configuración de puerta de enlace activa/activa. En caso contrario, deje este valor **Deshabilitado**.
     * Mantenga **Configurar BGP** en **Deshabilitado**, a menos que su configuración requiera específicamente este valor. Si necesita esta configuración, el valor predeterminado del ASN es 65515, aunque esto se puede cambiar.
5. Seleccione **Revisar y crear** para ejecutar la validación.
6. Una vez superada la validación, seleccione **Crear** para implementar VPN Gateway.