---
title: Implementación de Traffic Manager para equilibrar las cargas de trabajo de Azure VMware Solution
description: Aprenda a integrar Traffic Manager con Azure VMware Solution para equilibrar las cargas de trabajo de aplicaciones en varios puntos de conexión de distintas regiones.
ms.topic: how-to
ms.date: 02/08/2021
ms.openlocfilehash: d00f699c094995dad15d4ab5558fcd838ed33128
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110087011"
---
# <a name="deploy-traffic-manager-to-balance-azure-vmware-solution-workloads"></a>Implementación de Traffic Manager para equilibrar las cargas de trabajo de Azure VMware Solution

En este artículo se le guía por los pasos para integrar [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) con Azure VMware Solution. La integración equilibra las cargas de trabajo aplicaciones en varios puntos de conexión. En este artículo también se le guía por los pasos para configurar Traffic Manager para dirigir el tráfico entre tres instancias de [Azure Application Gateway](../application-gateway/overview.md) que abarcan varias regiones de Azure VMware Solution. 

Las puertas de enlace tienen máquinas virtuales (VM) de Azure VMware Solution configuradas como miembros del grupo de back-end para equilibrar la carga de las solicitudes de capa 7 entrantes. Para obtener más información, consulte [Uso de Azure Application Gateway para proteger aplicaciones web en Azure VMware Solution](protect-azure-vmware-solution-with-application-gateway.md)

En el diagrama se muestra cómo Traffic Manager proporciona equilibrio de carga para las aplicaciones en el nivel de DNS entre los puntos de conexión regionales. Las puertas de enlace tienen miembros del grupo de back-end configurados como servidores IIS y se hace referencia a ellos como puntos de conexión externos de Azure VMware Solution. La conexión a través de la red virtual entre las dos regiones de la nube privada usa una puerta de enlace ExpressRoute.   

:::image type="content" source="media/traffic-manager/traffic-manager-topology.png" alt-text="Diagrama de la arquitectura de la integración de Traffic Manager con Azure VMware Solution" lightbox="media/traffic-manager/traffic-manager-topology.png" border="false":::

Antes de empezar, revise primero los [requisitos previos](#prerequisites) y, a continuación, analizaremos los procedimientos para lograr lo siguiente:

> [!div class="checklist"]
> * Comprobación de la configuración de las puertas de enlace de aplicación y el segmento NSX-T
> * Creación de un perfil de Traffic Manager
> * Incorporación de puntos de conexión externos en el perfil de Traffic Manager

## <a name="prerequisites"></a>Requisitos previos

- Tres máquinas virtuales configuradas como servidores de Microsoft IIS que se ejecutan en regiones de Azure VMware Solution distintas: 
   - Oeste de EE. UU.
   - Oeste de Europa
   - Este de EE. UU. (local) 

- Una puerta de enlace de aplicación con puntos de conexión externos en las regiones de Azure VMware Solution mencionadas anteriormente.

- Hospede con conectividad de Internet para la comprobación. 

- Un [segmento de red de NSX-T creado en Azure VMware Solution](tutorial-nsx-t-network-segment.md).

## <a name="verify-your-application-gateways-configuration"></a>Comprobación de la configuración de las puertas de enlace de aplicación

En los pasos siguientes se comprueba la configuración de las puertas de enlace de aplicación.

1. En Azure Portal, seleccione **Puertas de enlace de aplicación** para ver una lista de las puertas de enlace de aplicación actuales:

   - AVS-GW-WUS
   - AVS-GW-EUS (local)
   - AVS-GW-WEU

   :::image type="content" source="media/traffic-manager/app-gateways-list-1.png" alt-text="Captura de pantalla de la página de Application Gateway que muestra la lista de puertas de enlace de aplicaciones configuradas." lightbox="media/traffic-manager/app-gateways-list-1.png":::

1. Seleccione una de las puertas de enlace de aplicación previamente implementadas. 

   Se abre una ventana que muestra diversa información sobre la puerta de enlace de aplicación. 

   :::image type="content" source="media/traffic-manager/backend-pool-config.png" alt-text="Captura de pantalla de la página de Application Gateway que muestra los detalles de la puerta de enlace de aplicaciones seleccionada." lightbox="media/traffic-manager/backend-pool-config.png":::

1. Seleccione **Grupos de back-end** para comprobar la configuración de uno de los grupos de back-end. Verá un miembro de un grupo de back-end de VM configurado como servidor web con una dirección IP de 172.29.1.10.
 
   :::image type="content" source="media/traffic-manager/backend-pool-ip-address.png" alt-text="Captura de pantalla de la página Editar un grupo de back-end con la dirección IP de destino resaltada.":::

1. Compruebe la configuración de las otras puertas de enlace de aplicación y los miembros del grupo de back-end. 

## <a name="verify-the-nsx-t-segment-configuration"></a>Comprobación de la configuración del segmento NSX-T

En los pasos siguientes se comprueba la configuración del segmento NSX-T en el entorno de Azure VMware Solution.

1. Seleccione **Segmentos** para ver los segmentos configurados.  Verá que Contoso-segment1 está conectado a la puerta de enlace Contoso-T01, un enrutador flexible de nivel 1.

   :::image type="content" source="media/traffic-manager/nsx-t-segment-avs.png" alt-text="Captura de pantalla que muestra perfiles de segmento en NSX-T Manager." lightbox="media/traffic-manager/nsx-t-segment-avs.png":::    

1. Seleccione **Tier-1 Gateways** (Puertas de enlace de nivel 1) para ver una lista de las puertas de enlace de nivel 1 con el número de segmentos vinculados. 

   :::image type="content" source="media/traffic-manager/nsx-t-segment-linked-2.png" alt-text="Captura de pantalla que muestra la dirección de puerta de enlace del segmento seleccionado.":::    

1. Seleccione el segmento vinculado a Contoso-T01. Se abre una ventana que muestra la interfaz lógica configurada en el enrutador de nivel 01. Sirve como puerta de enlace a la máquina virtual miembro del grupo de back-end conectada al segmento.

1. En el cliente de vSphere, seleccione la máquina virtual para ver sus detalles. 

   >[!NOTE]
   >Su dirección IP coincide con el miembro del grupo de back-end de VM configurado como servidor web de la sección anterior: 172.29.1.10.

   :::image type="content" source="media/traffic-manager/nsx-t-vm-details.png" alt-text="Captura de pantalla que muestra los detalles de la máquina virtual en el cliente VSphere." lightbox="media/traffic-manager/nsx-t-vm-details.png":::    

4. Seleccione la VM y, a continuación, seleccione **ACCIONES > Editar configuración** para comprobar la conexión con el segmento NSX-T.

## <a name="create-your-traffic-manager-profile"></a>Creación de un perfil de Traffic Manager

1. Inicie sesión en [Azure Portal](https://rc.portal.azure.com/#home). En **Servicios de Azure > Redes**, seleccione **Perfiles de Traffic Manager**.

2. Seleccione **+ Agregar** para crear un perfil nuevo de Traffic Manager.
 
3. Proporcione los siguientes datos y, a continuación, seleccione **Crear**:

   - Nombre del perfil
   - Método de enrutamiento (usar [ponderado](../traffic-manager/traffic-manager-routing-methods.md))
   - Suscripción
   - Grupo de recursos

## <a name="add-external-endpoints-into-the-traffic-manager-profile"></a>Incorporación de puntos de conexión externos en el perfil de Traffic Manager

1. Seleccione el perfil de Traffic Manager en el panel de resultados de la búsqueda, seleccione **Puntos de conexión** y, a continuación, **+ Agregar**.

1. En cada uno de los puntos de conexión externos de las distintas regiones, escriba los detalles necesarios y, a continuación, seleccione **Agregar**: 
   - Tipo
   - Nombre
   - Nombre de dominio completo (FQDN) o IP
   - Peso (asignar un peso de 1 a cada punto de conexión). 

   Una vez creados, los tres se muestran en el perfil de Traffic Manager. El estado del monitor de los tres debe ser **En línea**.

3. Seleccione **Información general** y copie la dirección URL en **Nombre DNS**.

   :::image type="content" source="media/traffic-manager/traffic-manager-endpoints.png" alt-text="Captura de pantalla que muestra información general del punto de conexión de Traffic Manager con el nombre DNS resaltado." lightbox="media/traffic-manager/traffic-manager-endpoints.png"::: 

4. Pegue la dirección URL del nombre DNS en un explorador. En la captura de pantalla se muestra el tráfico que se dirige a la región Oeste de Europa.

   :::image type="content" source="media/traffic-manager/traffic-to-west-europe.png" alt-text="Captura de pantalla de la ventana del explorador que muestra el tráfico enrutado a la región Oeste de Europa."::: 

5. Actualice el explorador. En la captura de pantalla siguiente se muestra el tráfico que se dirige a otro conjunto de miembros del grupo de back-end en la región Oeste de EE. UU.

   :::image type="content" source="media/traffic-manager/traffic-to-west-us.png" alt-text="Captura de pantalla de la ventana del explorador que muestra el tráfico enrutado a la región Oeste de EE. UU."::: 

6. Vuelva a actualizar el explorador. En la captura de pantalla se muestra el tráfico que se dirige al conjunto final de miembros del grupo de back-end en el entorno local.

   :::image type="content" source="media/traffic-manager/traffic-to-on-premises.png" alt-text="Captura de pantalla de la ventana del explorador que muestra el tráfico enrutado a la región local.":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha leído sobre la integración de Azure Traffic Manager con Azure VMware Solution, puede obtener más información acerca de:

- [Uso de Azure Application Gateway en Azure VMware Solution](protect-azure-vmware-solution-with-application-gateway.md)
- [Métodos de enrutamiento de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md)
- [Combinación de servicios de equilibrio de carga en Azure](../traffic-manager/traffic-manager-load-balancing-azure.md)
- [Medición del rendimiento de Traffic Manager](../traffic-manager/traffic-manager-performance-considerations.md)
