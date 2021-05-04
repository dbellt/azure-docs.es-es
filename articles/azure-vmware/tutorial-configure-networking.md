---
title: 'Tutorial: Configuración de redes para la nube privada de VMware en Azure'
description: Aprenda a crear y configurar las redes necesarias para implementar una nube privada en Azure.
ms.topic: tutorial
ms.custom: contperf-fy21q4
ms.date: 04/23/2021
ms.openlocfilehash: 13f74c6120bd78127a2989218b3d535989bb7861
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945703"
---
# <a name="tutorial-configure-networking-for-your-vmware-private-cloud-in-azure"></a>Tutorial: Configuración de redes para la nube privada de VMware en Azure

Una nube privada de Azure VMware Solution requiere una instancia de Azure Virtual Network. Dado que Azure VMware Solution no admite su instancia de vCenter local, se necesitan pasos adicionales para la integración con el entorno local. También es necesario configurar un circuito de ExpressRoute y una puerta de enlace de red virtual.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una red virtual
> * Creación de una puerta de enlace de red virtual
> * Conectar el circuito ExpressRoute a la puerta de enlace


## <a name="create-a-virtual-network"></a>Creación de una red virtual

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya al grupo de recursos que creó en el [tutorial para crear una nube privada](tutorial-create-private-cloud.md) y seleccione **+ Agregar** para definir un nuevo recurso. 

1. En el campo **Buscar en Marketplace**, escriba **Virtual Network**. Busque el recurso Virtual Network y selecciónelo.

1. En la página **Virtual Network**, seleccione **Crear** para configurar la red virtual para la nube privada.

1. En la página **Crear red virtual**, escriba los detalles de la red virtual.

1. En la pestaña **Aspectos básicos**, escriba el nombre de la red virtual, seleccione la región adecuada y haga clic en **Siguiente: Direcciones IP**.

1. En la pestaña **Direcciones IP**, en **Espacio de direcciones IPv4**, escriba el espacio de direcciones que creó en el tutorial anterior.

   > [!IMPORTANT]
   > Debe usar un espacio de direcciones que **no** se superponga con el que usó al crear la nube privada en el tutorial anterior.

1. Seleccione **+ Agregar subred** y, en la página **Agregar subred**, asigne a la subred un nombre y un intervalo de direcciones adecuados. Cuando haya terminado, seleccione **Agregar**.

1. Seleccione **Revisar + crear**.

   :::image type="content" source="./media/tutorial-configure-networking/create-virtual-network.png" alt-text="Seleccionar Revisar y crear." border="true":::

1. Compruebe la información y seleccione **Crear**. Una vez completada la implementación, verá la red virtual en el grupo de recursos.

## <a name="create-a-virtual-network-gateway"></a>Creación de una puerta de enlace de red virtual

Una vez creada una red virtual, creará una puerta de enlace de red virtual.

1. En el grupo de recursos, seleccione **+ Agregar** para agregar un recurso nuevo.

1. En el campo **Buscar en el Marketplace**, escriba **Puerta de enlace de red virtual**. Busque el recurso Virtual Network y selecciónelo.

1. En la página **Puerta de enlace de red virtual**, seleccione **Crear**.

1. En la pestaña Aspectos básicos de la página **Crear puerta de enlace de red virtual**, proporcione valores para los campos y seleccione **Revisar y crear**. 

   | Campo | Valor |
   | --- | --- |
   | **Suscripción** | Valor rellenado previamente con la suscripción a la que pertenece el grupo de recursos. |
   | **Grupos de recursos** | Valor rellenado previamente para el grupo de recursos actual. El valor debe ser el grupo de recursos que creó en una prueba anterior. |
   | **Nombre** | Escriba un nombre único para la puerta de enlace de red virtual. |
   | **Región** | Seleccione la ubicación geográfica de la puerta de enlace de red virtual. |
   | **Tipo de puerta de enlace** | seleccione **ExpressRoute**. |
   | **SKU** | Deje el valor predeterminado: **standard**. |
   | **Red virtual** | Seleccione la red virtual que ha creado antes. Si no ve la red virtual, asegúrese de que la región de la puerta de enlace se corresponde con la región de la red virtual. |
   | **Intervalo de direcciones de subred de puerta de enlace** | Este valor se rellena cuando se selecciona la red virtual. No cambie el valor predeterminado. |
   | **Dirección IP pública** | Seleccione **Crear nuevo**. |

   :::image type="content" source="./media/tutorial-configure-networking/create-virtual-network-gateway.png" alt-text="Proporcione valores para los campos y, a continuación, seleccione Revisar y crear." border="true":::

1. Compruebe que los detalles son correctos y seleccione **Crear** para iniciar la implementación de la puerta de enlace de red virtual. 
1. Una vez finalizada la implementación, vaya a la siguiente sección para conectar ExpressRoute a la puerta de enlace de red virtual que contiene la nube privada de Azure VMware Solution.

## <a name="connect-expressroute-to-the-virtual-network-gateway"></a>Conexión de ExpressRoute a la puerta de enlace de red virtual

Ahora que ha implementado una puerta de enlace de red virtual, deberá agregar una conexión entre ella y la nube privada de Azure VMware Solution.

[!INCLUDE [connect-expressroute-to-vnet](includes/connect-expressroute-vnet.md)]


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de una red virtual
> * Creación de una puerta de enlace de red virtual
> * Conectar el circuito ExpressRoute a la puerta de enlace


Continúe con el siguiente tutorial para aprender a crear los segmentos de red NSX-T que se usan para las máquinas virtuales en vCenter.

> [!div class="nextstepaction"]
> [Creación de segmentos de red de NSX-T](tutorial-nsx-t-network-segment.md)