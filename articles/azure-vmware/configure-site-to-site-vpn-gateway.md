---
title: Configuración de una VPN de sitio a sitio en vWAN para Azure VMware Solution
description: Obtenga información sobre cómo establecer un túnel de sitio a sitio de VPN (IPsec IKEv1 e IKEv2) en Azure VMware Solution.
ms.topic: how-to
ms.date: 03/23/2021
ms.openlocfilehash: 4d410a94b822db8eeed0ba166908c804a1a6eaaa
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802782"
---
# <a name="configure-a-site-to-site-vpn-in-vwan-for-azure-vmware-solution"></a>Configuración de una VPN de sitio a sitio en vWAN para Azure VMware Solution

En este artículo analizaremos los pasos necesarios para establecer un túnel VPN (IPsec IKEv1 e IKEv2) de sitio a sitio que termine en el centro de Microsoft Azure Virtual WAN. El centro contiene la puerta de enlace de ExpressRoute de Azure VMware Solution y la puerta de enlace de VPN de sitio a sitio. Conecta un dispositivo VPN local con un punto de conexión de Azure VMware Solution.

:::image type="content" source="media/create-ipsec-tunnel/vpn-s2s-tunnel-architecture.png" alt-text="Diagrama que muestra la arquitectura de un túnel VPN de sitio a sitio." border="false":::

En este artículo de procedimientos:
- Va a crear un centro de Azure Virtual WAN y una puerta de enlace de VPN con una dirección IP pública asociada. 
- Va a crear una puerta de enlace de Azure ExpressRoute y establecer un punto de conexión de Azure VMware Solution. 
- Va a habilitar una configuración local de VPN basada en directivas. 

## <a name="prerequisites"></a>Requisitos previos
Debe tener una dirección IP de acceso público que termine en un dispositivo VPN local.

## <a name="step-1-create-an-azure-virtual-wan"></a>Paso 1. Creación de una instancia de Azure Virtual WAN

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="step-2-create-a-virtual-wan-hub-and-gateway"></a>Paso 2. Creación de un centro de conectividad de Virtual WAN y una puerta de enlace

>[!TIP]
>También puede [crear una puerta de enlace en un centro de conectividad ya existente](../virtual-wan/virtual-wan-expressroute-portal.md#existinghub), editándolo.

1. Seleccione la instancia de Virtual WAN que creó en el paso anterior.

1. Seleccione **Crear centro de conectividad virtual**, escriba los campos necesarios y, después, seleccione **Siguiente: Sitio a sitio**. 

   Escriba la subred con un valor `/24` (mínimo).

   :::image type="content" source="media/create-ipsec-tunnel/create-virtual-hub.png" alt-text="Captura de pantalla que muestra la página de creación de un centro virtual.":::

4. Seleccione la pestaña **Sitio a sitio**, defina la puerta de enlace de sitio a sitio estableciendo el rendimiento agregado desde la lista desplegable **Unidades de escalado de puerta de enlace**. 

   >[!TIP]
   >Las unidades de escalado se encuentran en pares de redundancia, cada una de las cuales admite 500 Mbps (una unidad de escalado = 500 Mbps). 
  
   :::image type="content" source="../../includes/media/virtual-wan-tutorial-hub-include/site-to-site.png" alt-text="Captura de pantalla que muestra los detalles de sitio a sitio":::

5. En la pestaña **ExpressRoute**, cree una puerta de enlace de ExpressRoute. 

   :::image type="content" source="../../includes/media/virtual-wan-tutorial-er-hub-include/hub2.png" alt-text="Captura de pantalla de la configuración de ExpressRoute":::

   >[!TIP]
   >Una unidad de escalado equivale a 2 Gbps. 

    Se tardan 30 minutos aproximadamente en crear cada centro. 

## <a name="step-3-create-a-site-to-site-vpn"></a>Paso 3. Crear una VPN de sitio a sitio.

1. En Azure Portal, seleccione la instancia de Virtual WAN que creó anteriormente.

2. En la página **Información general** del centro de conectividad virtual, seleccione **Conectividad** > **VPN (De sitio a sitio)**  > **Crear un nuevo sitio de VPN**.

   :::image type="content" source="media/create-ipsec-tunnel/create-vpn-site-basics.png" alt-text="Captura de pantalla de la página de información general del centro virtual, con las opciones VPN (De sitio a sitio) y Crear un nuevo sitio de VPN seleccionadas.":::  
 
3. En la pestaña **Aspectos básicos**, complete los campos necesarios. 

   :::image type="content" source="media/create-ipsec-tunnel/create-vpn-site-basics2.png" alt-text="Captura de pantalla de la pestaña Aspectos básicos del nuevo sitio VPN":::  

   1. Establezca la opción **Protocolo de puerta de enlace de borde** en **Habilitar**.  Cuando está habilitada, garantiza que tanto Azure VMware Solution como los servidores locales anuncien sus rutas a través del túnel. Si esta opción está deshabilitada, las subredes que tienen que anunciarse deberán mantenerse manualmente. Si faltan subredes, HCX no podrá formar la malla del servicio. Para obtener más información, consulte [Acerca de BGP con Azure VPN Gateway](../vpn-gateway/vpn-gateway-bgp-overview.md).
   
   1. En **Espacio de direcciones privadas**, escriba el bloque CIDR local. Se usa para enrutar todo el tráfico enlazado en el entorno local a través del túnel. El bloque CIDR solo es necesario si no se habilita BGP.

1. Seleccione **Siguiente: Vínculos** y complete los campos obligatorios. La especificación de los nombres del proveedor y el vínculo le permite distinguir entre cualquier número de puertas de enlace que puedan crearse en algún momento como parte del centro. El BGP y el número de sistema autónomo (ASN) deben ser únicos dentro de su organización.

   :::image type="content" source="media/create-ipsec-tunnel/create-vpn-site-links.png" alt-text="Captura de pantalla que muestra los detalles del vínculo":::

1. Seleccione **Revisar + crear**. 

1. Vaya hasta el centro de conectividad virtual que desee y anule la selección de la **Asociación de concentrador** para conectar el sitio VPN al centro.
 
   :::image type="content" source="../../includes/media/virtual-wan-tutorial-site-include/connect.png" alt-text="Captura de pantalla que muestra el panel Sitios conectados del centro de conectividad virtual listo para una clave precompartida y la configuración asociada":::   

## <a name="step-4-optional-create-policy-based-vpn-site-to-site-tunnels"></a>Paso 4. (Opcional) Creación de túneles de sitio a sitio de VPN basados en directivas

>[!IMPORTANT]
>Este es un paso opcional y se aplica únicamente a las VPN basadas en directivas. 

Las configuraciones de VPN basadas en directivas requieren que se especifiquen redes locales y de Azure VMware Solution, incluidos los intervalos del centro de conectividad.  Dichos intervalos del centro de conectividad se usan para especificar el dominio de cifrado del punto de conexión local del túnel VPN basado en directivas.  El lado de Azure VMware Solution solo requiere que se habilite el indicador del selector de tráfico basado en directivas. 

1. En Azure Portal, vaya al sitio central de Virtual WAN. En **Conectividad**, seleccione **VPN (de sitio a sitio)** .

2. Seleccione el nombre del sitio VPN, el botón de puntos suspensivos (…) situado en el extremo derecho y, luego, **Editar la conexión de VPN a este concentrador**.
 
   :::image type="content" source="media/create-ipsec-tunnel/edit-vpn-section-to-this-hub.png" alt-text="Captura de pantalla de la página de Azure para el sitio del centro de WAN virtual que muestra un botón de puntos suspensivos seleccionado para acceder a la opción Editar la conexión de VPN a este concentrador." lightbox="media/create-ipsec-tunnel/edit-vpn-section-to-this-hub.png":::

3. Edite la conexión entre el sitio de VPN y el centro y, luego, seleccione **Guardar**.
   - Para el protocolo de seguridad de Internet (IPSec), seleccione **Personalizado**.
   - Para la opción Usar el selector de tráfico basado en directivas, seleccione **Habilitar**.
   - Especifique los detalles de **Fase 1 IKE** y **Fase 2 IKE (IPsec)** . 
 
   :::image type="content" source="media/create-ipsec-tunnel/edit-vpn-connection.png" alt-text="Captura de pantalla de la página Editar conexión VPN."::: 
 
   Los selectores de tráfico o las subredes que forman parte del dominio de cifrado basado en directivas deben ser los siguientes:
    
   - Centro de centro de conectividad de Virtual WAN `/24`
   - Nube privada de Azure VMware Solution `/22`
   - Red virtual de Azure conectada (si existe)

## <a name="step-5-connect-your-vpn-site-to-the-hub"></a>Paso 5. Conexión del sitio de VPN con el centro

1. Elija el nombre del sitio VPN y, luego, seleccione **Conectar sitios de VPN**. 

1. En el campo **Clave precompartida**, escriba la clave definida previamente para el punto de conexión local. 

   >[!TIP]
   >Si no tiene una clave ya definida, puede dejar este campo en blanco. Se generará una clave de forma automática. 

   :::image type="content" source="../../includes/media/virtual-wan-tutorial-connect-vpn-site-include/connect.png" alt-text="Captura de pantalla que muestra el panel Sitios conectados del centro de conectividad virtual listo para una clave precompartida y la configuración asociada"::: 

1. Si va a implementar un firewall en el centro de conectividad y es el próximo salto, establezca la opción **Propagar la ruta predeterminada** en **Habilitar**. 

   Cuando está habilitada, el centro de conectividad de Virtual WAN se propaga a una conexión solo si dicho centro ya ha aprendido la ruta predeterminada al implementar un firewall en el centro de conectividad o si otro sitio conectado tiene habilitada la tunelización forzada. La ruta predeterminada no se origina en el centro de conectividad de Virtual WAN.  

1. Seleccione **Conectar**. Al cabo de unos minutos, en el sitio aparecerá el estado de la conexión y de la conectividad.

   :::image type="content" source="../../includes/media/virtual-wan-tutorial-connect-vpn-site-include/status.png" alt-text="Captura de pantalla en la que se muestra una conexión de sitio a sitio y el estado de conectividad" lightbox="../../includes/media/virtual-wan-tutorial-connect-vpn-site-include/status.png":::

1. [Descargue el archivo de configuración de VPN](../virtual-wan/virtual-wan-site-to-site-portal.md#device) para el punto de conexión local.  

3. Aplique una revisión de ExpressRoute de Azure VMware Solution en el centro de Virtual WAN. 

   >[!IMPORTANT]
   >En primer lugar, debe crear una nube privada para poder aplicar revisiones a la plataforma. 

   [!INCLUDE [request-authorization-key](includes/request-authorization-key.md)]

4. Vincule Azure VMware Solution y la puerta de enlace de VPN en el centro de Virtual WAN. Usará la clave de autorización y el identificador de ExpressRoute (URI del circuito del mismo nivel) del paso anterior.

   1. Seleccione la puerta de enlace de ExpressRoute y, a continuación, seleccione **Canjear la clave de autorización**.

      :::image type="content" source="media/create-ipsec-tunnel/redeem-authorization-key.png" alt-text="Captura de pantalla que muestra la página de ExpressRoute de la nube privada con la opción Canjear la clave de autorización seleccionada.":::

   1. Pegue la clave de autorización en el campo **Clave de autorización**.
   1. Pegue el identificador de ExpressRoute en el campo **URI de circuito del mismo nivel**. 
   1. Seleccione **Asocie automáticamente este circuito de ExpressRoute con el centro**. 
   1. Seleccione **Agregar** para establecer el vínculo. 

5. Para probar la conexión, [cree un segmento NSX-T](./tutorial-nsx-t-network-segment.md) y aprovisione una máquina virtual en la red. Haga ping a los puntos de conexión locales y de Azure VMware Solution.
