---
title: 'Conexión de la red local a una red virtual: VPN de sitio a sitio (clásico): Portal'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo crear una conexión de IPsec entre la red local y una red virtual de Azure clásico a través de la red pública de Internet.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: f440ec76d3085e7a5ef7ec99f1a5a76c9c444ef9
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205348"
---
# <a name="create-a-site-to-site-connection-using-the-azure-portal-classic"></a>Creación de una conexión de sitio a sitio mediante Azure Portal (clásico)

Este artículo muestra cómo usar Azure Portal para crear una conexión de puerta de enlace VPN de sitio a sitio desde la red local a la red virtual. Los pasos de este artículo se aplican al modelo de implementación clásica, no al modelo de implementación actual, Resource Manager. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](./tutorial-site-to-site-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-site-to-site-classic-portal.md)
> 

Se utiliza una conexión de puerta de enlace VPN de sitio a sitio para conectar su red local a una red virtual de Azure a través de un túnel VPN de IPsec/IKE (IKEv1 o IKEv2). Este tipo de conexión requiere un dispositivo VPN local que tenga una dirección IP pública asignada. Para más información acerca de las puertas de enlace VPN, consulte [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).

![Diagrama de la conexión entre locales de VPN Gateway de sitio a sitio](./media/vpn-gateway-howto-site-to-site-classic-portal/site-to-site-diagram.png)

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Compruebe que desea trabajar en el modelo de implementación clásica. Si desea trabajar en el modelo de implementación de Resource Manager, consulte [Creación de una conexión de sitio a sitio (Resource Manager)](./tutorial-site-to-site-portal.md). Se recomienda usar el modelo de implementación de Resource Manager, ya que el modelo clásico es heredado.
* Asegúrese de tener un dispositivo VPN compatible y alguien que pueda configurarlo. Para más información acerca de los dispositivos VPN compatibles y su configuración, consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md).
* Compruebe que tiene una dirección IPv4 pública externa para el dispositivo VPN.
* Si no está familiarizado con los intervalos de direcciones IP ubicados en la red local, necesita trabajar con alguien que pueda proporcionarle estos detalles. Al crear esta configuración, debe especificar los prefijos del intervalo de direcciones IP al que Azure enrutará la ubicación local. Ninguna de las subredes de la red local puede superponerse con las subredes de la red virtual a la que desea conectarse.
* Se requiere PowerShell para especificar la clave compartida y crear la conexión de puerta de enlace VPN. [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

### <a name="sample-configuration-values-for-this-exercise"></a><a name="values"></a>Valores de configuración de ejemplo para este ejercicio

Los ejemplos de este artículo utilizan los valores siguientes. Puede usar estos valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo. Normalmente, cuando se trabaja con valores de dirección IP para el espacio de direcciones, se debe coordinar con el administrador de red para evitar espacios de direcciones superpuestos, lo que puede afectar al enrutamiento. En este caso, reemplace los valores de dirección IP por los suyos propios si quiere crear una conexión operativa.

* **Grupos de recursos:** TestRG1
* **Nombre de la red virtual:** TestVNet1
* **Espacio de direcciones**: 10.11.0.0/16
* **Nombre de subred:** FrontEnd
* **Intervalo de direcciones de subred:** 10.11.0.0/24
* **GatewaySubnet:** 10.11.255.0/27
* **Región:** (EE. UU.) Este de EE. UU.
* **Nombre del sitio local:** Site2
* **Espacio de direcciones de cliente:** el espacio de direcciones que se encuentra en el sitio local.

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Creación de una red virtual

Cuando se crea una red virtual que se usará para una conexión S2S, debe asegurarse de que los espacios de direcciones que especifique no se superponen con ninguno de los espacios de direcciones de cliente de los sitios locales a los que desea conectarse. Si tiene subredes superpuestas, la conexión no funcionará correctamente.

* Si ya dispone de una red virtual, compruebe que la configuración sea compatible con el diseño de la puerta de enlace de VPN. Preste especial atención a las subredes que se pueden superponer con otras redes.

* Si no tiene una red virtual, créela. Las capturas de pantalla se proporcionan a modo de ejemplo. Asegúrese de reemplazar los valores por los suyos.

### <a name="to-create-a-virtual-network"></a>Creación de una red virtual

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="configure-the-site-and-gateway"></a><a name="localsite"></a>Configuración del sitio y la puerta de enlace

### <a name="to-configure-the-site"></a>Configuración del sitio

Normalmente, sitio local suele hacer referencia a la ubicación local. Contiene la dirección IP del dispositivo VPN al que se creará una conexión y los intervalos de direcciones IP que se enrutarán a través de la puerta de enlace VPN en el dispositivo VPN.

1. En la página de la red virtual, en **Configuración**, seleccione **Conexiones de sitio a sitio**.
1. En la página Conexiones de sitio a sitio, seleccione **+ Agregar**.
1. En la página **Configurar una conexión VPN y una puerta de enlace**, para **Tipo de conexión**, deje seleccionado **De sitio a sitio**. Para este ejercicio, debe usar una combinación de los [valores de ejemplo](#values) y sus propios valores.

   * **Dirección IP de la puerta de enlace de VPN:** es la dirección IP pública del dispositivo VPN en la red local. El dispositivo VPN requiere una dirección IP IPv4 pública. Especifique una dirección IP pública válida para el dispositivo VPN al que desea conectarse. Debe ser accesible para Azure. Si no conoce la dirección IP del dispositivo VPN, siempre puede poner un valor de marcador de posición (siempre que tenga el formato de una dirección IP pública válida) y cambiarlo más adelante.

   * **Espacio de direcciones de cliente:** lista de los intervalos de direcciones IP que quiere enrutar a la red local mediante esta puerta de enlace. Puede agregar varios intervalos de espacios de direcciones. Asegúrese de que los intervalos que especifique aquí no se superponen con los intervalos de otras redes a la que se conecta su red virtual, ni con los intervalos de direcciones de la propia red virtual.
1. En la parte inferior de la página, NO seleccione Revisar y crear. En su lugar, seleccione **Siguiente: Puerta de enlace>** .

### <a name="to-configure-the-virtual-network-gateway"></a><a name="sku"></a>Configuración de la puerta de enlace de red virtual

1. En la página **Puerta de enlace**, seleccione los siguientes valores:

   * **Size:** Es la SKU de la puerta de enlace que va a usa para crear la puerta de enlace de red virtual. Las puertas de enlace de VPN clásicas utilizan las SKU antiguas (heredadas). Para más información acerca de las SKU antiguas de puerta de enlace, consulte [Funcionamiento de SKU de puerta de enlace de red virtual (SKU antigua)](vpn-gateway-about-skus-legacy.md). Puede seleccionar **Estándar** para este ejercicio.

   * **Tipo de enrutamiento:** Seleccione el tipo de enrutamiento de la puerta de enlace. Esto también se conoce como tipo de VPN. Es importante seleccionar el tipo correcto porque la puerta de enlace no se puede convertir de un tipo a otro. El dispositivo VPN debe ser compatible con el tipo de enrutamiento que seleccione. Para más información acerca del tipo de enrutamiento, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#vpntype). Verá que muchos artículos hacen referencia a los tipos de VPN 'RouteBased' y 'PolicyBased'. 'Dynamic' corresponde a 'RouteBased' y 'Static' corresponde a 'PolicyBased'. Normalmente, se quiere el enrutamiento dinámico.

   * **Subred de puerta de enlace:** El tamaño de la subred de la puerta de enlace que especifique depende de la configuración de la puerta de enlace VPN que desea crear. Aunque es posible crear una puerta de enlace tan pequeña como /29, le recomendamos que use /27 o /28. Esto crea una subred mayor que incluye más direcciones. El uso de una subred de la puerta de enlace mayor permite suficientes direcciones IP para dar cabida a posibles configuraciones futuras.

1. En la parte inferior de la página, seleccione **Revisar y crear** para validar la configuración. Seleccione **Crear** para realizar la implementación. Según la SKU de puerta de enlace que seleccione, puede tardar hasta 45 minutos en crear una puerta de enlace de red virtual.

## <a name="configure-your-vpn-device"></a><a name="vpndevice"></a>Configuración del dispositivo VPN

Las conexiones de sitio a sitio a una red local requieren un dispositivo VPN. En este paso, se configura el dispositivo VPN. Para configurar el dispositivo VPN, necesita los valores siguientes:

* Una clave compartida. Se trata de la misma clave compartida que se especifica al crear la conexión VPN de sitio a sitio. En estos ejemplos se utiliza una clave compartida básica. Se recomienda que genere y utilice una clave más compleja.
* La dirección IP pública de la puerta de enlace de red virtual. Puede ver la dirección IP pública mediante Azure Portal, PowerShell o la CLI.

[!INCLUDE [vpn-gateway-configure-vpn-device-rm](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

## <a name="retrieve-values"></a><a name="getvalues"></a>Recuperación de valores

[!INCLUDE [retrieve values](../../includes/vpn-gateway-values-classic.md)]

## <a name="create-the-connection"></a><a name="CreateConnection"></a>Creación de la conexión

> [!NOTE]
> En el modelo de implementación clásica, este paso no está disponible en Azure Portal o a través de Azure Cloud Shell. Debe usar la versión para Service Management (SM) de los cdmlets de Azure PowerShell localmente desde el escritorio.
>

En este paso, con los valores de los pasos anteriores, establezca la clave compartida y cree la conexión. La clave que se establezca debe ser la misma que se usó en la configuración del dispositivo VPN.

1. Establezca la clave compartida y cree la conexión.

   * Cambie los valores de -VNetName y -LocalNetworkSiteName. Al especificar el nombre que contiene espacios, escriba el valor entre comillas simples.
   * "SharedKe" es un valor que se puede generar y especificar. En el ejemplo, hemos usado "abc123" pero puede usar algo más complejo. Lo importante es que el valor que especifique aquí debe ser el mismo que el que se especificó al configurar el dispositivo VPN.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group TestRG1 TestVNet1' `
   -LocalNetworkSiteName '6C74F6E6_Site2' -SharedKey abc123
   ```

1. Cuando se crea la conexión, el resultado es: **Estado: Correcto**.

## <a name="verify-your-connection"></a><a name="verify"></a>Comprobación de la conexión

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

Si tiene problemas para conectarse, consulte la sección de **solución de problemas** de la tabla de contenido en el panel izquierdo.

## <a name="how-to-reset-a-vpn-gateway"></a><a name="reset"></a>Procedimientos para restablecer una puerta de enlace de VPN

Restablecer una puerta de enlace de VPN de Azure es útil si se pierde la conectividad VPN entre locales en uno o varios túneles VPN de sitio a sitio. En esta situación, todos tus dispositivos VPN locales funcionan correctamente, pero no pueden establecer túneles IPsec con las Puertas de enlace de VPN de Azure. Para conocer los pasos, consulte [Restablecimiento de una puerta de enlace de VPN](./reset-gateway.md#resetclassic).

## <a name="how-to-change-a-gateway-sku"></a><a name="changesku"></a>Procedimientos para cambiar la SKU de una puerta de enlace

Para que conocer los pasos para cambiar la SKU de puerta de enlace, consulte la sección [Cambio de tamaño de las SKU de una puerta de enlace](vpn-gateway-about-SKUS-legacy.md#classicresize).

## <a name="next-steps"></a>Pasos siguientes

* Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](../index.yml) para más información.
* Para más información acerca de la tunelización forzada, consulte la [información acerca de la tunelización forzada](vpn-gateway-about-forced-tunneling.md).