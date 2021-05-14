---
title: 'Configure BGP para VPN Gateway: Portal'
titleSuffix: Azure VPN Gateway
description: Aprenda cómo configurar BGP para Azure VPN Gateway.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 03/22/2021
ms.author: yushwang
ms.openlocfilehash: df42925b50cba4d32ea554ae54dbcb09a5e04377
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205960"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways"></a>Configuración de BGP en instancias de Azure VPN Gateway

Este artículo le guiará por los pasos para habilitar BGP en una conexión de VPN de sitio a sitio (S2S) entre locales y una conexión de red virtual a red virtual mediante Azure Portal.

## <a name="about-bgp"></a><a name="about"></a>Información acerca de BGP

BGP es el protocolo de enrutamiento estándar usado habitualmente en Internet para intercambiar información de enrutamiento y disponibilidad entre dos o más redes. BGP permite que instancias de Azure VPN Gateway y los dispositivos VPN locales, denominados vecinos o pares BGP, intercambien "rutas" que comunicarán a ambas puertas de enlace la disponibilidad y la posibilidad de que dichos prefijos pasen a través de las puertas de enlace o los enrutadores implicados. BGP también puede permitir el enrutamiento del tránsito entre varias redes mediante la propagación de las rutas que una puerta de enlace de BGP aprende de un par BGP a todos los demás pares BGP.

Para obtener más información sobre las ventajas de BGP y conocer los requisitos técnicos y las consideraciones sobre el uso de BGP, consulte [Información general de BGP con Azure VPN Gateway](vpn-gateway-bgp-overview.md).

## <a name="getting-started"></a>Introducción

Cada parte de este artículo le ayuda a constituir un bloque de creación básico para habilitar BGP en la conectividad de red. Si completa las tres partes, podrá crear la topología tal como se muestra en el diagrama 1.

**Diagrama 1**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises-v2v.png" alt-text="Diagrama que muestra la configuración y la arquitectura de red" border="false":::

Puede combinar estos elementos juntos para crear una red de tránsito más compleja y de saltos múltiples que satisfaga sus necesidades.

### <a name="prerequisites"></a>Requisitos previos

Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).

## <a name="part-1-configure-bgp-on-the-virtual-network-gateway"></a><a name ="config"></a>Parte 1: Configuración de BGP en la puerta de enlace de red virtual

En esta sección, creará y configurará una red virtual, creará y configurará una puerta de enlace de red virtual con parámetros BGP, y obtendrá la dirección IP del par BGP de Azure. El diagrama 2 muestra los parámetros de configuración que se deben usar al trabajar con los pasos de esta sección.

**Diagrama 2**

:::image type="content" source="./media/bgp-howto/bgp-gateway.png" alt-text="Diagrama que muestra la configuración de la puerta de enlace de red virtual" border="false":::

### <a name="1-create-and-configure-testvnet1"></a>1. Creación y configuración de TestVNet1

En este paso, creará y configurará TestVNet1. Use los pasos del [tutorial de creación de una puerta de enlace](./tutorial-create-gateway-portal.md) para crear y configurar la red virtual de Azure y VPN Gateway. Use la configuración de referencia en las siguientes capturas de pantalla.

* Red virtual:

   :::image type="content" source="./media/bgp-howto/testvnet-1.png" alt-text="TestVNet1 con los prefijos de dirección correspondientes":::

* Subredes:

   :::image type="content" source="./media/bgp-howto/testvnet-1-subnets.png" alt-text="Subredes TestVNet1":::

### <a name="2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>2. Creación de VPN Gateway para TestVNet1 con parámetros BGP

En este paso, creará una instancia de VPN Gateway con los parámetros BGP correspondientes.

1. En Azure Portal, vaya al recurso de **puerta de enlace de red virtual** de Marketplace y seleccione **Crear**.

1. Rellene los parámetros como se muestra a continuación:

   :::image type="content" source="./media/bgp-howto/create-gateway-1.png" alt-text="Creación de VNG1":::

1. En la sección **Configuración de BGP** de la página, establezca la siguiente configuración:

   :::image type="content" source="./media/bgp-howto/create-gateway-1-bgp.png" alt-text="Configuración de BGP":::

   * Seleccione **Configuración de BGP** - **Habilitada** para mostrar la sección de configuración de BGP.

   * Rellene el ASN (número de sistema autónomo).

   * El campo de **dirección IP de BGP APIPA de Azure** es opcional. Si sus dispositivos VPN locales usan la dirección APIPA para BGP, debe seleccionar una dirección en el intervalo de direcciones APIPA reservado de Azure para VPN, que se sitúa entre **169.254.21.0** y **169.254.22.255**. En este ejemplo se usa 169.254.21.11.

   * Si crea una instancia de VPN Gateway activo-activo, en la sección de BGP se mostrará una **segunda dirección IP de BGP APIPA de Azure personalizada** . En el intervalo de APIPA permitido, (de **169.254.21.0** a **169.254.22.255**), seleccione otra dirección IP. La segunda dirección IP debe ser distinta de la primera.

   > [!IMPORTANT]
   >
   > * De forma predeterminada, Azure asigna una dirección IP privada desde el intervalo del prefijo GatewaySubnet automáticamente como dirección IP de BGP de Azure en la instancia de Azure VPN Gateway. La dirección de BGP APIPA de Azure es necesaria cuando los dispositivos VPN locales usan una dirección APIPA (169.254.0.1 a 169.254.255.254) como dirección IP de BGP. Azure VPN Gateway elegirá la dirección APIPA personalizada si el recurso de puerta de enlace de red local correspondiente (red local) tiene una dirección APIPA como dirección IP del par BGP. Si la puerta de enlace de red local usa una dirección IP normal (no APIPA), Azure VPN Gateway revertirá a la dirección IP privada del intervalo GatewaySubnet.
   >
   > * Las direcciones de BGP APIPA no deben superponerse entre los dispositivos VPN locales y todas las instancias de Azure VPN Gateway conectadas.
   >
   > * Cuando se usan direcciones de APIPA en puertas de enlace de VPN de Azure, estas últimas no inician sesiones de emparejamiento de BGP con direcciones IP de origen APIPA. El dispositivo VPN local debe iniciar conexiones de emparejamiento de BGP.
   >

1. Seleccione **Revisar y crear** para ejecutar la validación. Una vez superada la validación, seleccione **Crear** para implementar VPN Gateway. Una puerta de enlace puede tardar hasta 45 minutos en crearse e implementarse completamente. Puede ver el estado de implementación en la página Información general de la puerta de enlace.

### <a name="3-obtain-the-azure-bgp-peer-ip-addresses"></a>3. Obtenga las direcciones IP del par BGP de Azure

Una vez creada la puerta de enlace, puede obtener las direcciones IP del par BGP en la puerta de enlace de VPN de Azure. Estas direcciones son necesarias para configurar los dispositivos VPN locales para establecer sesiones de BGP con la puerta de enlace de VPN de Azure.

1. Vaya al recurso de puerta de enlace de red virtual y seleccione la página **Configuración** para ver la información de configuración de BGP como se muestra en la siguiente captura de pantalla. En esta página, puede ver toda la información de configuración de BGP en la instancia de Azure VPN Gateway: ASN, una dirección IP pública y las direcciones IP del par BGP correspondientes en Azure (predeterminadas y APIPA).

   :::image type="content" source="./media/bgp-howto/vnet-1-gw-bgp.png" alt-text="Puerta de enlace de BGP":::

1. En la página **Configuración**, puede realizar los siguientes cambios de configuración:

   * Puede actualizar el ASN o la dirección IP de BGP APIPA si es necesario.
   * Si tiene una instancia de VPN Gateway activo-activo, se mostrará en esta página la dirección IP pública, la predeterminada y direcciones IP de BGP APIPA de la segunda instancia de Azure VPN Gateway.

1. Si realizó algún cambio, seleccione **Guardar** para confirmar los cambios en la instancia de Azure VPN Gateway.

## <a name="part-2-configure-bgp-on-cross-premises-s2s-connections"></a><a name ="crosspremises"></a>Parte 2: Configuración de BGP en conexiones S2S entre locales

Para establecer una conexión entre locales, debe crear una *puerta de enlace de red local* para representar el dispositivo VPN local y una *conexión* para conectarse a la instancia de VPN Gateway con la puerta de enlace de red local como se explica en [Creación de una conexión de sitio a sitio](./tutorial-site-to-site-portal.md). En este artículo se incluyen las propiedades adicionales necesarias para especificar los parámetros de configuración de BGP.

**Diagrama 3**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises.png" alt-text="Diagrama que muestra IPsec" border="false":::

### <a name="1-configure-bgp-on-the-local-network-gateway"></a>1. Configuración de BGP en la puerta de enlace de red local

En este paso, configurará BGP en la puerta de enlace de red local. Use la siguiente captura de pantalla como ejemplo. En la captura de pantalla se muestra la puerta de enlace de red local (Site5) con los parámetros especificados en el diagrama 3.

:::image type="content" source="./media/bgp-howto/create-local-bgp.png" alt-text="Configuración de BGP para la puerta de enlace de red local":::

#### <a name="important-configuration-considerations"></a>Consideraciones importantes sobre la configuración

* El ASN y la dirección IP del par BGP deben coincidir con la configuración del enrutador VPN local.
* Puede dejar el **especio de direcciones** vacío solo si usa BGP para conectarse a esta red. Azure VPN Gateway agregará de forma interna una ruta de la dirección IP del par BGP al túnel IPsec correspondiente. Si **NO** usa BGP entre la instancia de Azure VPN Gateway y esta red en particular, **debe** proporcionar una lista de prefijos de dirección válidos para el **espacio de direcciones**.
* Opcionalmente, puede usar una **dirección IP APIPA** (169.254.x.x) como dirección IP del par BGP local si es necesario. Sin embargo, también tendrá que especificar una dirección IP APIPA como se ha descrito anteriormente en este artículo para la instancia de Azure VPN Gateway; de lo contrario, la sesión de BGP no podrá establecerse para esta conexión.
* Puede especificar la información de configuración de BGP durante la creación de la puerta de enlace de red local, o bien puede agregar o cambiar la configuración de BGP en la página **Configuración** del recurso de puerta de enlace de red local.

**Ejemplo**

En este ejemplo de usa una dirección APIPA (169.254.100.1) como dirección IP del par BGP local:

:::image type="content" source="./media/bgp-howto/local-apipa.png" alt-text="APIPA y BGP de puerta de enlace de red local":::

### <a name="2-configure-a-s2s-connection-with-bgp-enabled"></a>2. Configuración de una conexión S2S con BGP habilitado

En este paso, creará una nueva conexión con BGP habilitado. Si ya tiene una conexión y desea habilitar BGP en ella, puede [actualizar una conexión existente](#update).

#### <a name="to-create-a-connection-with-bgp-enabled"></a>Para crear una conexión con BGP habilitado

Para crear una nueva conexión con BGP habilitado, en la página **Agregar conexión**, rellene los valores y, a continuación, active la opción **Habilitar BGP** para habilitar BGP en esta conexión. Seleccione **Aceptar** para crear la conexión.

:::image type="content" source="./media/bgp-howto/ipsec-connection-bgp.png" alt-text="Conexión entre locales de IPsec con BGP":::

#### <a name="to-update-an-existing-connection"></a><a name ="update"></a>Para actualizar una conexión existente

Si desea cambiar la opción de BGP en una conexión, vaya a la página **Configuración** del recurso de conexión y, a continuación, alterne la opción de **BGP** como se resalta en el siguiente ejemplo. Para guardar los cambios, seleccione **Guardar**.

:::image type="content" source="./media/bgp-howto/update-bgp.png" alt-text="Actualización de BGP para una conexión":::

## <a name="part-3-configure-bgp-on-vnet-to-vnet-connections"></a><a name ="v2v"></a>Parte 3: Configuración de BGP en conexiones de red virtual a red virtual

Los pasos para habilitar o deshabilitar BGP en una conexión de red virtual a red virtual son los mismos que los pasos de sitio a sitio de la [parte 2](#crosspremises). Puede habilitar BGP al crear la conexión o actualizar la configuración en una conexión de red virtual a red virtual existente.

>[!NOTE] 
>Una conexión de red virtual a red virtual sin BGP limitará la comunicación solo a las dos redes virtuales conectadas. Habilite BGP para permitir la funcionalidad de enrutamiento a otras conexiones de sitio a sitio o de red virtual a red virtual de estas dos redes virtuales.
>

Para dar contexto, en referencia al **diagrama 4**, si BGP fuera a deshabilitarse entre TestVNet2 y TestVNet1, TestVNet2 no aprendería las rutas para la red local, Site5, y, por tanto, no podría comunicarse con el Site5. Una vez que habilite BGP, como se muestra en el diagrama 4, las tres redes podrán comunicarse a través de las conexiones de IPsec y de red virtual a red virtual.

**Diagrama 4**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises-v2v.png" alt-text="Diagrama que muestra la red completa" border="false":::

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md) para ver los pasos.