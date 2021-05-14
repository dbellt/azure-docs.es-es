---
title: 'Creación de una conexión entre redes virtuales: clásico: Portal de Azure'
description: Obtenga información sobre cómo conectar redes virtuales de Azure clásico simultáneamente mediante PowerShell y Azure Portal.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/15/2020
ms.author: cherylmc
ms.openlocfilehash: aee40b4d642a43f0d4261bae7c42f92049daa4de
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289523"
---
# <a name="configure-a-vnet-to-vnet-connection-classic"></a>Configuración de una conexión de red virtual a red virtual (clásico)

Este artículo lo ayuda a crear una conexión de puerta de enlace de VPN entre las redes virtuales. Las redes virtuales pueden estar en la misma región o en distintas, así como pertenecer a una única suscripción o a varias.

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-portal-classic/v2vclassic.png" alt-text="Diagrama que muestra la arquitectura de red virtual a red virtual clásica":::

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

Los pasos de este artículo se corresponden al modelo de implementación clásica y a Azure Portal. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:

> [!div class="op_single_selector"]
> * [Clásico](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Resource Manager](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [Conexión de redes virtuales en diferentes modelos de implementación](vpn-gateway-connect-different-deployment-models-portal.md)
>
>

## <a name="about-vnet-to-vnet-connections"></a>Acerca de conexiones de red virtual a red virtual

La conexión de una red virtual a otra (de red virtual a red virtual) en el modelo de implementación clásica con una VPN Gateway es parecida a la conexión de una red virtual a la ubicación de un sitio local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro con IPsec/IKE.

Las redes virtuales que se conecten pueden estar en suscripciones y regiones distintas. Puede combinar la comunicación entre redes virtuales con configuraciones de varios sitios. Esto permite establecer topologías de red que combinen la conectividad entre entornos con la conectividad entre redes virtuales.

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-portal-classic/aboutconnections.png" alt-text="Diagrama que muestra las conexiones":::

### <a name="why-connect-virtual-networks"></a><a name="why"></a>¿Por qué debería conectarse a redes virtuales?

Puede que desee conectar redes virtuales por las siguientes razones:

* **Presencia geográfica y redundancia geográfica entre regiones**

  * Puede configurar su propia replicación geográfica o sincronización con conectividad segura sin recurrir a los puntos de conexión a Internet.
  * Con Azure Load Balancer y Microsoft, o con una tecnología de agrupación en clústeres de otros fabricantes, puede configurar cargas de trabajo de alta disponibilidad y redundancia geográfica en varias regiones de Azure. Por ejemplo, puede configurar AlwaysOn de SQL con grupos de disponibilidad distribuidos en varias regiones de Azure.
* **Aplicaciones regionales de varios niveles con límite de aislamiento sólido**

  * En la misma región se pueden configurar aplicaciones de niveles múltiples con varias redes virtuales conectadas entre sí, con un aislamiento sólido y una comunicación entre niveles segura.
* **Comunicación entre suscripciones y entre organizaciones en Azure**

  * Si tiene varias suscripciones a Azure, puede conectar cargas de trabajo de distintas suscripciones simultáneamente entre redes virtuales de forma segura.
  * Asimismo, tanto las empresas como los proveedores de servicios pueden habilitar la comunicación entre organizaciones con tecnología VPN segura en Azure.

Para más información acerca de las conexiones de red virtual a red virtual, consulte [Consideraciones de red virtual a red virtual](#faq) al final de este artículo.

## <a name="prerequisites"></a>Requisitos previos

Usamos el portal para la mayoría de los pasos, pero debe usar PowerShell para crear las conexiones entre las redes virtuales. No puede crear las conexiones mediante Azure Portal porque no hay ninguna manera de especificar la clave compartida en el portal. [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="planning"></a><a name="planning"></a>Planificación

Es importante decidir los intervalos que usará para configurar las redes virtuales. Para esta configuración, debe asegurarse de que ninguno de los intervalos de red virtual se superpongan entre sí o con cualquiera de las redes locales a las que se conectan.

### <a name="vnets"></a><a name="vnet"></a>Redes virtuales

Para este ejercicio, usamos los valores de ejemplo siguientes:

**Valores para TestVNet1**

Nombre: TestVNet1<br>
Espacio de direcciones: 10.11.0.0/16, 10.12.0.0/16 (opcional)<br>
Nombre de subred: predeterminado<br>
Intervalo de direcciones de subred: 10.11.0.0/24<br>
Grupo de recursos: ClassicRG<br>
Ubicación: Este de EE. UU.<br>
GatewaySubnet: 10.11.1.0/27

**Valores para TestVNet4**

Nombre: TestVNet4<br>
Espacio de direcciones: 10.41.0.0/16, 10.42.0.0/16 (opcional)<br>
Nombre de subred: predeterminado<br>
Intervalo de direcciones de subred: 10.41.0.0/24<br>
Grupo de recursos: ClassicRG<br>
Ubicación: Oeste de EE. UU.<br>
GatewaySubnet: 10.41.1.0/27

### <a name="connections"></a><a name="plan"></a>Conexiones

En la tabla siguiente se muestra un ejemplo de cómo conectará las redes virtuales. Use los intervalos solo como referencia. Escriba los intervalos para las redes virtuales. Necesitará esta información en pasos posteriores.

En este ejemplo, TestVNet1 se conecta a un sitio de red local que crea con el nombre "VNet4Local". La configuración de VNet4Local contiene los prefijos de dirección para TestVNet4.
El sitio local para cada red virtual es la otra red virtual. Los valores de ejemplo siguientes se usan para esta configuración:

**Ejemplo**

| Virtual Network | Espacio de direcciones | Location | Se conecta a un sitio de red local |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |Este de EE. UU. |SiteVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |Oeste de EE. UU. |SiteVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

## <a name="create-virtual-networks"></a><a name="vnetvalues"></a>Creación de redes virtuales

En este paso, crea dos redes virtuales clásicas, TestVNet1 y TestVNet4. Si utiliza este artículo como ejercicio, use los [valores de ejemplo](#vnet).

**Cuando cree las redes virtuales, recuerde la siguiente configuración:**

* **Espacios de direcciones de la red virtual**: en la página Espacios de direcciones de la red virtual, especifique el intervalo de direcciones que desea usar para la red virtual. Estas son las direcciones IP dinámicas que se asignarán a las máquinas virtuales y a las demás instancias de rol implementadas en esta red virtual.<br>Los espacios de direcciones que selecciona no se pueden superponer con los espacios de direcciones de ninguna de las otras redes virtuales o las ubicaciones locales a las que se conectará esta red virtual.

* **Ubicación**: al crear una red virtual, la debe asociar a una ubicación de Azure (región). Por ejemplo, Por ejemplo, si desea que las máquinas virtuales que implemente en la red virtual se encuentren físicamente en Oeste de EE.UU., seleccione esa ubicación. No se puede cambiar la ubicación asociada a la red virtual después de crearla.

**Después de crear las redes virtuales, puede agregar la configuración siguiente:**

* **Espacio de direcciones**: no se requiere espacio de direcciones adicional para esta configuración, pero puede agregar espacio de direcciones adicional después de crear la red virtual.

* **Subredes**: no se requieren subredes adicionales para esta configuración, pero puede que quiera que las máquinas virtuales estén en una subred independiente de las otras instancias de rol.

* **Servidores DNS**: escriba el nombre del servidor DNS y la dirección IP. Mediante este valor no se crea un servidor DNS. Le permite especificar el servidor DNS que desea usar para la resolución de nombres para esta red virtual.

### <a name="to-create-a-classic-virtual-network"></a>Para crear una red virtual clásica

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="configure-sites-and-gateways"></a><a name="localsite"></a>Configuración de sitios y puertas de enlace

Azure usa la configuración especificada en cada sitio de red local para determinar cómo enrutar el tráfico entre las redes virtuales. Cada red virtual debe señalar a la red local correspondiente a la que se desea redirigir el tráfico. Puede determinar el nombre que quiere utilizar para hacer referencia a cada sitio de red local. Se recomienda usar un nombre descriptivo.

Por ejemplo, TestVNet1 se conecta a un sitio de red local que cree con el nombre "VNet4Local". La configuración de VNet4Local contiene los prefijos de dirección para TestVNet4.

Tenga en cuenta que el sitio local para cada red virtual es la otra red virtual.

| Virtual Network | Espacio de direcciones | Location | Se conecta a un sitio de red local |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |Este de EE. UU. |SiteVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |Oeste de EE. UU. |SiteVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

### <a name="to-configure-a-site"></a><a name="site"></a>Para configurar un sitio

Normalmente, sitio local suele hacer referencia a la ubicación local. Contiene la dirección IP del dispositivo VPN al que se creará una conexión y los intervalos de direcciones IP que se enrutarán a través de la puerta de enlace VPN en el dispositivo VPN.

1. En la página de la red virtual, en **Configuración**, seleccione **Conexiones de sitio a sitio**.
1. En la página Conexiones de sitio a sitio, seleccione **+ Agregar**.
1. En la página **Configurar una conexión VPN y una puerta de enlace**, para **Tipo de conexión**, deje seleccionado **De sitio a sitio**.

   * **Dirección IP de la puerta de enlace de VPN:** es la dirección IP pública del dispositivo VPN en la red local. Para este ejercicio, puede incluir una dirección ficticia porque todavía no tiene la dirección IP de VPN Gateway del otro sitio. Por ejemplo: 5.4.3.2. Más adelante, una vez configurada la puerta de enlace de la otra red virtual, puede ajustar este valor.

   * **Espacio de direcciones de cliente:** lista de los intervalos de direcciones IP que quiere enrutar a la otra red virtual mediante esta puerta de enlace. Puede agregar varios intervalos de espacios de direcciones. Asegúrese de que los intervalos que especifique aquí no se superponen con los intervalos de otras redes a la que se conecta su red virtual, ni con los intervalos de direcciones de la propia red virtual.
1. En la parte inferior de la página, NO seleccione Revisar y crear. En su lugar, seleccione **Siguiente: Puerta de enlace>** .

### <a name="to-configure-a-virtual-network-gateway"></a><a name="sku"></a>Configuración de una puerta de enlace de red virtual

1. En la página **Puerta de enlace**, seleccione los siguientes valores:

   * **Size:** Es la SKU de la puerta de enlace que va a usa para crear la puerta de enlace de red virtual. Las puertas de enlace de VPN clásicas utilizan las SKU antiguas (heredadas). Para más información acerca de las SKU antiguas de puerta de enlace, consulte [Funcionamiento de SKU de puerta de enlace de red virtual (SKU antigua)](vpn-gateway-about-skus-legacy.md). Puede seleccionar **Estándar** para este ejercicio.

   * **Tipo de enrutamiento:** Seleccione el tipo de enrutamiento de la puerta de enlace. Esto también se conoce como tipo de VPN. Es importante seleccionar el tipo correcto porque la puerta de enlace no se puede convertir de un tipo a otro. El dispositivo VPN debe ser compatible con el tipo de enrutamiento que seleccione. Para más información acerca del tipo de enrutamiento, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#vpntype). Verá que muchos artículos hacen referencia a los tipos de VPN 'RouteBased' y 'PolicyBased'. 'Dynamic' corresponde a 'RouteBased' y 'Static' corresponde a 'PolicyBased'. Para esta configuración, seleccione **Dynamic**.

   * **Subred de puerta de enlace:** El tamaño de la subred de la puerta de enlace que especifique depende de la configuración de la puerta de enlace VPN que desea crear. Aunque es posible crear una puerta de enlace tan pequeña como /29, le recomendamos que use /27 o /28. Esto crea una subred mayor que incluye más direcciones. El uso de una subred de la puerta de enlace mayor permite suficientes direcciones IP para dar cabida a posibles configuraciones futuras.

1. En la parte inferior de la página, seleccione **Revisar y crear** para validar la configuración. Seleccione **Crear** para realizar la implementación. Según la SKU de puerta de enlace que seleccione, puede tardar hasta 45 minutos en crear una puerta de enlace de red virtual.
1. Puede continuar en el paso siguiente mientras se crea esta puerta de enlace.

### <a name="configure-testvnet4-settings"></a>Configuración de las opciones de TestVNet4

Repita los pasos de [Creación de un sitio y una puerta de enlace](#localsite) para configurar TestVNet4 y reemplace los valores cuando sea necesario. Si hace esto como ejercicio, use los [valores de ejemplo](#planning).

## <a name="update-local-sites"></a><a name="updatelocal"></a>Actualización de sitios locales

Una vez que se crean las puertas de enlace de red virtual para ambas redes virtuales, debe ajustar las propiedades de **Dirección IP de VPN Gateway** de los sitios locales.

|Nombre de red virtual|Sitio conectado|Dirección IP de la puerta de enlace|
|:--- |:--- |:--- |
|TestVNet1|VNet4Local|Dirección IP de VPN Gateway para TestVNet4|
|TestVNet4|VNet1Local|Dirección IP de VPN Gateway para TestVNet1|

### <a name="part-1---get-the-virtual-network-gateway-public-ip-address"></a>Parte 1: Obtención de la dirección IP pública de puerta de enlace de red virtual

1. Para navegar a su virtual, diríjase a **Grupo de recursos** y selecciónela.
1. En la página de la red virtual, en el panel **Essentials** de la derecha, busque la **Dirección IP de la puerta de enlace** y cópiela en el portapapeles.

### <a name="part-2---modify-the-local-site-properties"></a>Parte 2: Modificación de las propiedades de sitios locales

1. En Conexiones de sitio a sitio, seleccione la conexión. Por ejemplo, SiteVNet4.
1. En la página **Propiedades** de Conexión de sitio a sitio, seleccione **Editar el sitio local**.
1. En el campo **Dirección IP de la puerta de enlace de VPN**, pegue la dirección IP de la puerta de enlace de VPN que copió en la sección anterior.
1. Seleccione **Aceptar**.
1. El campo se actualiza en el sistema. También puede usar este método para agregar la dirección IP adicional que quiere enrutar a este sitio.

### <a name="part-3---repeat-steps-for-the-other-vnet"></a>Parte 3: Repetición de los pasos para la otra red virtual

Repita los pasos para TestVNet4.

## <a name="retrieve-configuration-values"></a><a name="getvalues"></a>Recuperar los valores de configuración

[!INCLUDE [retrieve values](../../includes/vpn-gateway-values-classic.md)]

## <a name="create-connections"></a><a name="createconnections"></a>Crear conexiones

Una vez completados todos los pasos anteriores, puede establecer las claves compartidas previamente de IPsec/IKE y cree la conexión. Este conjunto de pasos usa PowerShell. Las conexiones de red virtual a red virtual del modelo de implementación clásica no se pueden configurar en Azure Portal porque la clave compartida no se puede especificar en el portal.

En los ejemplos, verá que la clave compartida es exactamente la misma. Siempre debe coincidir con la clave compartida. Asegúrese de reemplazar los valores de estos ejemplos por los nombres exactos de las redes locales y los sitios de red local.

1. Cree la conexión de TestVNet1 a TestVNet4. Asegúrese de cambiar los valores.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet1' `
   -LocalNetworkSiteName 'value for _VNet4Local' -SharedKey A1b2C3D4
   ```
2. Cree la conexión de TestVNet4 a TestVNet1.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet4' `
   -LocalNetworkSiteName 'value for _VNet1Local' -SharedKey A1b2C3D4
   ```
3. Espere a que se inicialicen las conexiones. Una vez que se inicializa la puerta de enlace, el estado pasa a ser "Correcto".

   ```
   Error          :
   HttpStatusCode : OK
   Id             :
   Status         : Successful
   RequestId      :
   StatusCode     : OK
   ```

## <a name="faq-and-considerations"></a><a name="faq"></a>Preguntas más frecuentes y consideraciones

Estas consideraciones se aplican a las redes virtuales clásicas y a las puertas de enlace de red virtual clásicas.

* Las redes virtuales pueden estar en la misma suscripción o en suscripciones distintas.
* Las redes virtuales pueden estar en la misma región de Azure o en regiones distintas (ubicaciones).
* Un servicio en la nube o un punto de conexión de equilibrio de carga no puede abarcar varias redes virtuales, aunque estas estén conectadas entre sí.
* La conexión simultánea de varias redes virtuales no requiere dispositivos VPN locales.
* VNet a VNet admite la conexión a Azure Virtual Networks. No admite la conexión de máquinas virtuales o servicios en la nube que no estén implementados en una red virtual.
* Red virtual a red virtual requiere puertas de enlace de enrutamiento dinámico. No se admiten puertas de enlace de enrutamiento estático de Azure.
* La conectividad de red virtual se puede usar de forma simultánea con VPN de varios sitios. Existe un máximo de 10 túneles de VPN para una VPN Gateway de red virtual conectada a otras redes virtuales o sitios locales.
* Los espacios de direcciones de las redes virtuales y de los sitios de red local no se pueden solapar. Los espacios de direcciones solapados provocarán un error al crear redes virtuales o al cargar archivos de configuración netcfg.
* No se admiten los túneles redundantes entre un par de redes virtuales.
* Todos los túneles de VPN para la red virtual, incluidas las VPN de punto a sitio (P2S), comparten el ancho de banda disponible para la VPN Gateway y el mismo Acuerdo de Nivel de Servicio de tiempo de actividad de VPN Gateway de Azure.
* El tráfico VNet a VNet viaja a través de la red troncal de Azure.

## <a name="next-steps"></a>Pasos siguientes

Compruebe las conexiones. Consulte [Comprobación de una conexión de VPN Gateway](vpn-gateway-verify-connection-resource-manager.md).
