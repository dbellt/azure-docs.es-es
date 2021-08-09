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
ms.openlocfilehash: 5b47f21ba6597380ca79bfce03feb8edd62aa5ae
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/22/2021
ms.locfileid: "114439736"
---
1. En **Buscar recursos, servicios y documentos (G+/)** , escriba **puerta de enlace de red virtual**. Busque la **puerta de enlace de red virtual** en los resultados de la búsqueda y selecciónela.

   :::image type="content" source="./media/vpn-gateway-add-gw-portal/search.png" alt-text="Captura de pantalla del campo Búsqueda." lightbox="./media/vpn-gateway-add-gw-portal/search-expand.png":::

1. En la página **Puertas de enlace de red virtual**, seleccione **+ Crear**. Se abre la página **Crear puerta de enlace de red virtual**.

   :::image type="content" source="./media/vpn-gateway-add-gw-portal/create.png" alt-text="Captura de pantalla de la página de puertas de enlace de red virtual con la opción Crear resaltada.":::
1. En la pestaña **Aspectos básicos**, rellene los valores de **Detalles del proyecto** y **Detalles de la instancia**.

   :::image type="content" source="./media/vpn-gateway-add-gw-portal/instance-details.png" alt-text="Captura de pantalla de los campos de Instancia.":::

   * **Suscripción**: seleccione la suscripción que desea usar en la lista desplegable.
   * **Grupo de recursos**: esta configuración se rellena automáticamente cuando selecciona la red virtual en esta página.
   * **Name**: Asigne un nombre a la puerta de enlace. Asignar nombre a la puerta de enlace no es lo mismo que asignar nombre a una subred de puerta de enlace. Este es el nombre del objeto de puerta de enlace que va a crear.
   * **Región**: Seleccione la región en la que quiere crear este recurso. La región de la puerta de enlace debe ser la misma que la red virtual.
   * **Tipo de puerta de enlace**: Seleccione **VPN**. Las puertas de enlace VPN usan el tipo de puerta de enlace de red virtual **VPN**.
   * **Tipo de VPN**: seleccione el tipo de VPN que se especifica para la configuración. La mayoría de las configuraciones requieren un tipo de VPN basada en enrutamiento.
   * **SKU:** seleccione la SKU de puerta de enlace que desea usar en la lista desplegable. Las SKU que aparecen en la lista desplegable dependen del tipo de VPN que seleccione. Asegúrese de seleccionar una SKU que admita las características que desea usar. Para más información acerca de las SKU de puerta de enlace, consulte [SKU de puerta de enlace](../articles/vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku).
   * **Generación**: seleccione la generación que desea usar. Consulte [SKU de puertas de enlace](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md#gwsku) para más información.
   * **Red virtual**: En el menú desplegable, seleccione la red virtual a la que quiera agregar esta puerta de enlace.
   * **Intervalo de direcciones de subred de puerta de enlace**: Este campo solo aparece si la red virtual no tiene una subred de puerta de enlace. Es mejor especificar /27 u otro superior (/26, /25, etc.). Esto permite suficientes direcciones IP para futuros cambios, como agregar una puerta de enlace de ExpressRoute. No se recomienda crear un intervalo inferior a /28. Si ya tiene una subred de puerta de enlace y desea ver los detalles de GatewaySubnet, vaya a la red virtual. Haga clic en **Subnets** (Subredes) para ver el intervalo. Si desea cambiar el intervalo, puede eliminar y volver a crear GatewaySubnet.