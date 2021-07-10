---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/27/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 1894f38a92fe839dcf44cc3265aa1ca25c05d6f6
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111564633"
---
1. En [Azure Portal](https://portal.azure.com), en **Buscar recursos, servicios y documentos (G+/)** escriba **puerta de enlace de red virtual**. Busque la **puerta de enlace de red virtual** en los resultados de la búsqueda y selecciónela.

   :::image type="content" source="./media/vpn-gateway-add-gw-rm-portal-include/search.png" alt-text="Campo Buscar" lightbox="./media/vpn-gateway-add-gw-rm-portal-include/search-expand.png":::

1. En la página **Puerta de enlace de red virtual**, seleccione **+ Agregar**. Se abre la página **Crear puerta de enlace de red virtual**.

   :::image type="content" source="./media/vpn-gateway-add-gw-rm-portal-include/add.png" alt-text="página de las puerta de enlace de red virtual":::
1. En la pestaña **Aspectos básicos**, rellene los valores de la puerta de enlace de red virtual.

   :::image type="content" source="./media/vpn-gateway-add-gw-rm-portal-include/gateway.png" alt-text="Campos de puerta de enlace":::

   :::image type="content" source="./media/vpn-gateway-add-gw-rm-portal-include/gateway-vnet.png" alt-text="Campos de puerta de enlace adicionales":::

   * **Suscripción**: seleccione la suscripción que desea usar en la lista desplegable.
   * **Grupo de recursos**: esta configuración se rellena automáticamente cuando selecciona la red virtual en esta página.

   **Detalles de instancia**

   * **Name**: Asigne un nombre a la puerta de enlace. Asignar nombre a la puerta de enlace no es lo mismo que asignar nombre a una subred de puerta de enlace. Este es el nombre del objeto de puerta de enlace que va a crear.
   * **Región**: Seleccione la región en la que quiere crear este recurso. La región de la puerta de enlace debe ser la misma que la red virtual.
   * **Tipo de puerta de enlace**: Seleccione **VPN**. Las puertas de enlace VPN usan el tipo de puerta de enlace de red virtual **VPN**.
   * **Tipo de VPN**: seleccione el tipo de VPN que se especifica para la configuración. La mayoría de las configuraciones requieren un tipo de VPN basada en enrutamiento.
   * **SKU**: seleccione la SKU de puerta de enlace en la lista desplegable. Las SKU que aparecen en la lista desplegable dependen del tipo de VPN que seleccione. Asegúrese de seleccionar una SKU que admita las características que desea usar. Para más información acerca de las SKU de puerta de enlace, consulte [SKU de puerta de enlace](../articles/vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku).
   * **Generación**: para obtener información sobre la generación de VPN Gateway, consulte [SKU de puerta de enlace](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md#gwsku).
   * **Red virtual**: En el menú desplegable, seleccione la red virtual a la que quiera agregar esta puerta de enlace.
   * **Intervalo de direcciones de subred de puerta de enlace**: Este campo solo aparece si la red virtual no tiene una subred de puerta de enlace. Si es posible, intente que el intervalo sea /27, o incluso mayor (/26, /25, etc.). No se recomienda crear un intervalo inferior a /28. Si ya tiene una subred de puerta de enlace y desea ver los detalles de GatewaySubnet, vaya a la red virtual. Haga clic en **Subnets** (Subredes) para ver el intervalo. Si desea cambiar el intervalo, puede eliminar y volver a crear GatewaySubnet.

   **Dirección IP pública**

   esta configuración especifica el objeto de dirección IP pública que se asocia a la puerta de enlace de VPN. La dirección IP pública se asigna dinámicamente a este objeto cuando se crea la puerta de enlace de VPN. La única vez que la dirección IP pública cambia es cuando la puerta de enlace se elimina y se vuelve a crear. No cambia cuando se cambia el tamaño, se restablece o se realizan actualizaciones u otras operaciones de mantenimiento interno de una puerta de enlace VPN.

     * **Dirección IP pública**: Mantenga la opción **Crear nueva** seleccionada.
     * **Nombre de dirección IP pública**: En el cuadro de texto, escriba un nombre para la dirección IP pública.
     * **Asignación**: VPN Gateway solo admite Dinámica.
     * **Habilitar el modo activo/activo**: Seleccione **Habilitar el modo activo/activo** solo si va a crear una configuración de puerta de enlace activa/activa. En caso contrario, deje este valor **Deshabilitado**.
     * Mantenga **Configurar BGP** en **Deshabilitado**, a menos que su configuración requiera específicamente este valor. Si necesita esta configuración, el valor predeterminado del ASN es 65515, aunque esto se puede cambiar.
1. Seleccione **Revisar y crear** para ejecutar la validación.
1. Una vez superada la validación, seleccione **Crear** para implementar VPN Gateway.

Una puerta de enlace puede tardar hasta 45 minutos en crearse e implementarse completamente. Puede ver el estado de implementación en la página Información general de la puerta de enlace. Una vez creada la puerta de enlace, puede ver la dirección IP que se le ha asignado consultando la red virtual en el portal. La puerta de enlace aparece como un dispositivo conectado.
