---
title: 'Tutorial: Creación de conexiones de ExpressRoute mediante Azure Virtual WAN'
description: En este tutorial, aprenderá a usar Azure Virtual WAN para crear conexiones de ExpressRoute con Azure y con entornos locales.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: a7c213abccef29f0c34c019d5ecf7a1c889b64d8
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164028"
---
# <a name="tutorial-create-an-expressroute-association-using-azure-virtual-wan"></a>Tutorial: Creación de una asociación de ExpressRoute mediante Azure Virtual WAN

En este tutorial se muestra cómo usar Virtual WAN para conectarse a los recursos de Azure mediante un circuito de ExpressRoute. Para más información sobre Virtual WAN y sus recursos, consulte el artículo de [introducción a Virtual WAN](virtual-wan-about.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una instancia de Virtual WAN
> * Crear un centro de conectividad y una puerta de enlace
> * Conectar una red virtual a un concentrador
> * Conectar un circuito a una puerta de enlace de centro de conectividad
> * Comprobación de la conectividad
> * Cambiar el tamaño de una puerta de enlace
> * Anunciar una ruta predeterminada

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Tiene una red virtual a la que quiere conectarse. Compruebe que ninguna de las subredes de sus redes locales se superpone a las redes virtuales a las que quiere conectarse. Para crear una red virtual en Azure Portal consulte este [Inicio rápido](../virtual-network/quick-create-portal.md).

* Su red virtual no tiene ninguna puerta de enlace de red virtual. Si la red virtual tiene alguna puerta de enlace (ya sea VPN o ExpressRoute), tiene que quitarla. Esta configuración requiere que las redes virtuales estén conectadas a la puerta de enlace del centro de conectividad de Virtual WAN.

* Obtenga un intervalo de direcciones IP para la región del concentrador. El centro de conectividad es una red virtual que Virtual WAN crea y usa. El intervalo de direcciones que especifique para el centro de conectividad no se puede superponer a ninguna de las redes virtuales existentes a las que ya esté conectado. Tampoco puede superponerse con los intervalos de direcciones a los que se conecta en el entorno local. Si no está familiarizado con los intervalos de direcciones IP ubicados en la configuración de la red local, póngase de acuerdo con alguien que pueda proporcionarle estos detalles.

* El circuito ExpressRoute debe ser un circuito Premium o Standard para conectarse a la puerta de enlace del centro de conectividad.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Creación de una instancia de Virtual WAN

Desde un explorador, navegue al [Portal de Azure](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

1. Vaya a la página de Virtual WAN. En el portal, haga clic en **+Crear un recurso**. Escriba **Virtual WAN** en el cuadro de búsqueda y seleccione ENTRAR.
2. Seleccione **Virtual WAN** en los resultados. En la página Virtual WAN, haga clic en **Crear** para abrir la página Crear una red WAN.
3. Dentro de la página **Crear una red WAN**, en la pestaña **Aspectos básicos**, rellene los campos siguientes:

   ![Creación de una red WAN](./media/virtual-wan-expressroute-portal/createwan.png)

   * **Suscripción**: seleccione la suscripción que quiere usar.
   * **Grupo de recursos**: cree uno nuevo o utilice uno ya existente.
   * **Ubicación del grupo de recursos**: elija una ubicación para los recursos en la lista desplegable. Una red WAN es un recurso global y no reside en una región determinada. De todas formas, tiene que seleccionar una región con el fin de administrar y ubicar más fácilmente el recurso WAN que cree.
   * **Nombre**: escriba el nombre que desea dar a la WAN.
   * **Tipo**: seleccione **Estándar**. No se puede crear una puerta de enlace de ExpressRoute con la SKU básica.
4. Cuando termine de rellenar los campos, haga clic en **Revisión y creación**.
5. Una vez que se haya superado la validación, seleccione **Crear** para crear la WAN virtual.

## <a name="create-a-virtual-hub-and-gateway"></a><a name="hub"></a>Creación de un centro de conectividad virtual y una puerta de enlace

Un centro de conectividad virtual es una red virtual que Virtual WAN crea y usa. Puede contener varias puertas de enlace, como VPN y ExpressRoute. En esta sección, creará una puerta de enlace de ExpressRoute para el centro de conectividad virtual. Puede crear la puerta de enlace cuando [cree un nuevo centro de conectividad virtual](#newhub), o puede crear la puerta de enlace en un [centro de conectividad ya existente](#existinghub) editándolo. 

Las puertas de enlace de ExpressRoute se aprovisionan en unidades de 2 Gbps. 1 unidad de escalado = 2 Gbps con capacidad para hasta 10 unidades de escalado = 20 Gbps. El proceso completo de creación de un centro de conectividad virtual y una puerta de enlace lleva aproximadamente 30 minutos.

### <a name="to-create-a-new-virtual-hub-and-a-gateway"></a><a name="newhub"></a>Para crear un nuevo centro de conectividad virtual y una puerta de enlace

Cree un nuevo centro de conectividad virtual. Una vez que haya creado el centro de conectividad, se le cobrará por él, aunque no lo conecte a ningún sitio.

[!INCLUDE [Create a hub](../../includes/virtual-wan-tutorial-er-hub-include.md)]

### <a name="to-create-a-gateway-in-an-existing-hub"></a><a name="existinghub"></a>Para crear una puerta de enlace en un centro de conectividad existente

También puede crear una puerta de enlace en un centro de conectividad ya existente, editándolo.

1. Vaya hasta el centro de conectividad virtual que desea editar y selecciónelo.
2. En la página **Editar centro de conectividad virtual**, seleccione la casilla **Incluir la puerta de enlace de ExpressRoute**.
3. Seleccione **Confirmar** para confirmar los cambios. El proceso completo de creación del centro de conectividad y de los recursos del mismo lleva aproximadamente 30 minutos.

   ![centro de conectividad existente](./media/virtual-wan-expressroute-portal/edithub.png "editar un centro de conectividad")

### <a name="to-view-a-gateway"></a>Para ver una puerta de enlace

Una vez que haya creado una puerta de enlace de ExpressRoute, puede ver los detalles de la misma. Vaya al centro de conectividad, seleccione **ExpressRoute** y verá la puerta de enlace.

![Ver puerta de enlace](./media/virtual-wan-expressroute-portal/viewgw.png "ver la puerta de enlace")

## <a name="connect-your-vnet-to-the-hub"></a><a name="connectvnet"></a>Conexión de la red virtual al centro de conectividad

En esta sección, creará la conexión de emparejamiento entre el centro de conectividad y una red virtual. Repita estos pasos para cada red virtual que desee conectar.

1. En la página de la red virtual WAN, haga clic en **Conexión de red virtual**.
2. En la página de conexión de red virtual, haga clic en **+ Agregar conexión**.
3. En la página **Agregar conexión**, rellene los campos siguientes:

    * **Nombre de la conexión**: asigne un nombre a la conexión.
    * **Centros**: seleccione el concentrador que desea asociar a esta conexión.
    * **Suscripción**: compruebe la suscripción.
    * **Red virtual**: seleccione la red virtual que quiere conectar con este concentrador. La red virtual no puede tener una puerta de enlace de red virtual ya existente (ni VPN, ni ExpressRoute).

## <a name="connect-your-circuit-to-the-hub-gateway"></a><a name="connectcircuit"></a>Conexión del circuito a la puerta de enlace del centro de conectividad

Una vez que se crea la puerta de enlace, puede conectarle un [circuito ExpressRoute](../expressroute/expressroute-howto-circuit-portal-resource-manager.md). Los circuitos Estándar o Premium de ExpressRoute que se encuentran en las ubicaciones compatibles con ExpressRoute Global Reach se pueden conectar a una puerta de enlace Virtual WAN ExpressRoute y disfrutar de todas las funcionalidades de tránsito de Virtual WAN (tránsito de VPN a VPN, de VPN y de ExpressRoute). Los circuitos Estándar o Premium de ExpressRoute que se encuentren en ubicaciones que no sean de Global Reach pueden conectarse a los recursos de Azure, pero no podrán usar las funcionalidades de tránsito de Virtual WAN. ExpressRoute Local también es compatible con los centros de conectividad de Azure Virtual WAN.

### <a name="to-connect-the-circuit-to-the-hub-gateway"></a>Para conectar el circuito a la puerta de enlace del centro de conectividad

En el portal, vaya a la página **Centro virtual-> Conectividad-> ExpressRoute**. Si tiene acceso a un circuito ExpressRoute en su suscripción, verá el circuito que desea usar en la lista de circuitos. Si no ve ningún circuito, pero se le ha proporcionado una clave de autorización y un URI de circuito del mismo nivel, puede canjear y conectar un circuito. Consulte [Para conectar canjeando una clave de autorización](#authkey).

1. Seleccione el circuito.
2. Seleccione **Conectar los circuitos**.

   ![conectar los circuitos](./media/virtual-wan-expressroute-portal/cktconnect.png "conectar los circuitos")

### <a name="to-connect-by-redeeming-an-authorization-key"></a><a name="authkey"></a>Para conectar canjeando una clave de autorización

Para conectar use la clave de autorización y el URI del circuito que se le proporcionaron.

1. En la página ExpressRoute, haga clic en **Canjear la clave de autorización**

   ![Captura de pantalla que muestra ExpressRoute en un centro de conectividad virtual con la opción Redeem authorization key (Canjear clave de autorización) seleccionada](./media/virtual-wan-expressroute-portal/redeem.png "canjear")
2. En la página Canjear l clave de autorización, rellene los valores.

   ![canjear los valores de clave](./media/virtual-wan-expressroute-portal/redeemkey2.png "canjear los valores de clave")
3. Seleccione **Agregar** para agregar la clave.
4. Vea el circuito. Un circuito canjeado solo muestra el nombre (sin tipo, proveedor, ni más información) porque está en una suscripción diferente a la del usuario.

## <a name="to-test-connectivity"></a>Para probar la conectividad

Una vez establecida la conexión del circuito, el estado de la conexión del centro de conectividad indicará "este centro", lo que implica que la conexión está establecida en la puerta de enlace de ExpressRoute del centro de conectividad. Espere unos 5 minutos antes de probar la conectividad de un cliente que se encuentre detrás del circuito ExpressRoute, por ejemplo, una máquina virtual en la red virtual que haya creado anteriormente.

Si tiene sitios conectados a una puerta de enlace de VPN Virtual WAN en el mismo centro de conectividad que la puerta de enlace de ExpressRoute, puede tener conectividad bidireccional entre los puntos de conexión de VPN y ExpressRoute. Se admite el enrutamiento dinámico (BGP). El ASN de las puertas de enlace en el centro de conectividad es fijo y en este momento no se puede editar.

## <a name="to-change-the-size-of-a-gateway"></a>Para cambiar el tamaño de una puerta de enlace

Si desea cambiar el tamaño de la puerta de enlace de ExpressRoute, busque la puerta de enlace de ExpressRoute dentro del centro de conectividad y seleccione las unidades de escalado en la lista desplegable. Guarde el cambio. Se tardará aproximadamente unos 30 minutos en actualizar la puerta de enlace del centro de conectividad.

![cambiar el tamaño de una puerta de enlace](./media/virtual-wan-expressroute-portal/changescale.png "cambiar el tamaño de una puerta de enlace")

## <a name="to-advertise-default-route-00000-to-endpoints"></a>Para anunciar la ruta predeterminada 0.0.0.0/0 a los puntos de conexión

Si desea que el centro de conectividad virtual de Azure anuncie la ruta predeterminada 0.0.0.0/0 a los puntos de conexión de ExpressRoute, tendrá que habilitar "Propagar la ruta predeterminada".

1. Seleccione su **Circuito >...-> Editar conexión**.

   ![Editar la conexión](./media/virtual-wan-expressroute-portal/defaultroute1.png "Edición de la conexión")

2. Seleccione **Habilitar** para propagar la ruta predeterminada.

   ![Propagar la ruta predeterminada](./media/virtual-wan-expressroute-portal/defaultroute2.png "Propagar la ruta predeterminada")

## <a name="clean-up-resources"></a><a name="cleanup"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, elimínelos. Algunos recursos de Virtual WAN deben eliminarse en un orden determinado debido a las dependencias. La eliminación puede tardar unos 30 minutos.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Virtual WAN en:

> [!div class="nextstepaction"]
> * [Preguntas más frecuentes sobre Virtual WAN](virtual-wan-faq.md)
