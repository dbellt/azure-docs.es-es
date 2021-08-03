---
title: Configuración y administración de DHCP en Azure VMware Solution
description: Aprenda a crear y administrar DHCP para la nube privada de Azure VMware Solution.
ms.topic: how-to
ms.custom: contperf-fy21q2
ms.date: 05/17/2021
ms.openlocfilehash: 28fb995cbc91d6e350e8d295c1f57d62c706e59e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110105397"
---
# <a name="configure-and-manage-dhcp-in-azure-vmware-solution"></a>Configuración y administración de DHCP en Azure VMware Solution

Las aplicaciones y cargas de trabajo que se ejecutan en un entorno de nube privada requieren servicios de DHCP para las asignaciones de direcciones IP.  En este artículo se muestra cómo crear y administrar DHCP en Azure VMware Solution de dos maneras:

- Si utiliza NSX-T para hospedar el servidor DHCP, deberá [crear un servidor DHCP](#create-a-dhcp-server) y [retransmitir a ese servidor](#create-dhcp-relay-service). Al crear el servidor DHCP, también agregará un segmento de red y especificará el intervalo de direcciones IP de DHCP.   

- Si usa un servidor DHCP externo de terceros en la red, tendrá que [crear un servicio de retransmisión DHCP](#create-dhcp-relay-service). Al crear una retransmisión a un servidor DHCP, ya sea mediante NSX-T o un tercero para hospedar el servidor DHCP, deberá especificar el intervalo de direcciones IP de DHCP.

>[!IMPORTANT]
>DHCP no funciona para las máquinas virtuales (VM) de la red elástica L2 de VMware HCX cuando el servidor DHCP está en el centro de datos local.  De forma predeterminada, NSX impide que todas las solicitudes DHCP atraviesen la red elástica L2. Para ver la solución, consulte el procedimiento [Configuración de DHCP en redes VMware HCX extendidas L2](configure-l2-stretched-vmware-hcx-networks.md).


## <a name="create-a-dhcp-server"></a>Creación de un servidor DHCP

Si desea usar NSX-T para hospedar el servidor DHCP, tendrá que crear un servidor DHCP. A continuación, agregará un segmento de red y especificará el intervalo de direcciones IP de DHCP.

1. En NSX-T Manager, seleccione **Networking** > **DHCP** (Redes > DHCP) y, a continuación, seleccione **Add Server** (Agregar servidor).

1. Seleccione **DHCP** para **Server Type** (Tipo de servidor), proporcione el nombre del servidor y la dirección IP y, a continuación, seleccione **Save** (Guardar).

   :::image type="content" source="./media/manage-dhcp/dhcp-server-settings.png" alt-text="agregar un servidor DHCP" border="true":::

1. Seleccione **Tier 1 Gateways** (Puertas de enlace de nivel 1), seleccione los puntos suspensivos verticales en la puerta de enlace de nivel 1 y seleccione **Edit** (Editar).

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway.png" alt-text="seleccionar la puerta de enlace que se va a usar" border="true":::

1. Para agregar una subred, seleccione **No IP Allocation Set** (No hay conjuntos de asignación de direcciones IP).

   :::image type="content" source="./media/manage-dhcp/add-subnet.png" alt-text="agregar una subred" border="true":::

1. En **Type** (Tipo), seleccione **DHCP Local Server** (Servidor local DHCP). 
   
1. Para **DHCP Server** (Servidor DHCP), seleccione **Default DHCP** (DHCP predeterminado) y luego **Save** (Guardar).

1. Seleccione **Save** (Guardar) de nuevo y luego **Close Editing** (Cerrar edición).

### <a name="add-a-network-segment"></a>Incorporación de un segmento de red

[!INCLUDE [add-network-segment-steps](includes/add-network-segment-steps.md)]


## <a name="create-dhcp-relay-service"></a>Creación de un servicio de retransmisión DHCP

Si desea usar un servidor DHCP externo de terceros, deberá crear un servicio de retransmisión DHCP. También especificará el intervalo de direcciones IP de DHCP en NSX-T Manager. 

1. En NSX-T Manager, seleccione **Networking** > **DHCP** (Redes > DHCP) y, a continuación, seleccione **Add Server** (Agregar servidor).

1. Seleccione **DHCP Relay** (Retransmisión de DHCP) para **Server Type** (Tipo de servidor), proporcione el nombre del servidor y la dirección IP y, a continuación, seleccione **Save** (Guardar).

   :::image type="content" source="./media/manage-dhcp/create-dhcp-relay.png" alt-text="creación de un servicio de retransmisión DHCP" border="true":::

1. Seleccione **Tier 1 Gateways** (Puertas de enlace de nivel 1), seleccione los puntos suspensivos verticales en la puerta de enlace de nivel 1 y seleccione **Edit** (Editar).

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway-relay.png" alt-text="editar puerta de enlace de nivel 1" border="true":::

1. Seleccione **No IP Allocation Set** (No hay conjuntos de asignación de direcciones IP) para definir la asignación de direcciones IP.

   :::image type="content" source="./media/manage-dhcp/edit-ip-address-allocation.png" alt-text="editar asignación de direcciones IP" border="true":::

1. En **Type** (Tipo), seleccione **DHCP Server** (Servidor DHCP). 
   
1. Para **DHCP Server** (Servidor DHCP), seleccione **DHCP Relay** (Retransmisión DHCP) y luego **Save** (Guardar).

1. Seleccione **Save** (Guardar) de nuevo y luego **Close Editing** (Cerrar edición).


## <a name="specify-the-dhcp-ip-address-range"></a>Especifique el intervalo de direcciones IP de DHCP.

1. En NSX-T Manager, seleccione **Networking** > **Segments** (Redes > Segmentos). 
   
1. Seleccione los puntos suspensivos verticales en el nombre del segmento y seleccione **Edit** (Editar).
   
1. Seleccione **Set Subnets** (Establecer subredes) para especificar la dirección IP de DHCP para la subred. 
   
   :::image type="content" source="./media/manage-dhcp/network-segments.png" alt-text="segmentos de red" border="true":::
      
1. Modifique la dirección IP de puerta de enlace si es necesario y escriba la dirección IP del intervalo de DHCP. 
      
   :::image type="content" source="./media/manage-dhcp/edit-subnet.png" alt-text="editar subredes" border="true":::
      
1. Seleccione **Apply** (Aplicar) y, a continuación, **Save** (Guardar). Se asigna un grupo de servidores DHCP al segmento.
      
   :::image type="content" source="./media/manage-dhcp/assigned-to-segment.png" alt-text="grupo de servidores DHCP asignado al segmento" border="true":::



## <a name="next-steps"></a>Pasos siguientes

Si quiere enviar solicitudes DHCP desde las máquinas virtuales de Azure VMware Solution a un servidor DHCP que no es de NSX-T, creará un perfil de segmento de seguridad. Para ver la solución, consulte el procedimiento [Configuración de DHCP en redes VMware HCX extendidas L2](configure-l2-stretched-vmware-hcx-networks.md).

