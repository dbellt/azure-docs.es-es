---
title: Implementación y configuración de Azure VMware Solution
description: Aprenda a usar la información recopilada en la fase de planeación para implementar y configurar la nube privada de Azure VMware Solution.
ms.topic: tutorial
ms.custom: contperf-fy21q4
ms.date: 04/23/2021
ms.openlocfilehash: fe5e34ecb0ed3d8984432d44bbed5678c4592553
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108175863"
---
# <a name="deploy-and-configure-azure-vmware-solution"></a>Implementación y configuración de Azure VMware Solution

En este artículo, usará la información de la [sección de planeación](production-ready-deployment-steps.md) para implementar y configurar Azure VMware Solution. 

>[!IMPORTANT]
>Es información que ha usado en la [sección de planeamiento](production-ready-deployment-steps.md) antes de continuar.

## <a name="step-1-register-the-microsoftavs-resource-provider"></a>Paso 1. Registre el proveedor de recursos **Microsoft.AVS**.

[!INCLUDE [register-resource-provider-steps](includes/register-resource-provider-steps.md)]

## <a name="step-2-create-an-azure-vmware-solution-private-cloud"></a>Paso 2. Creación de una nube privada de Azure VMware Solution

[!INCLUDE [create-private-cloud-azure-portal-steps](includes/create-private-cloud-azure-portal-steps.md)]

>[!NOTE]
>Puede encontrar información completa de este paso en el vídeo [Azure VMware Solution: Implementación](https://www.youtube.com/embed/gng7JjxgayI).


## <a name="step-3-connect-to-azure-virtual-network-with-expressroute"></a>Paso 3. Conexión a Azure Virtual Network con ExpressRoute

En la fase de planeamiento, ha definido si va a usar una puerta de enlace de red virtual de ExpressRoute *nueva* o *existente*.  

### <a name="use-a-new-expressroute-virtual-network-gateway"></a>Uso de una nueva puerta de enlace de red virtual de ExpressRoute

>[!IMPORTANT]
>Debe tener una red virtual con una subred de puerta de enlace que aún **no** tenga una puerta de enlace de red virtual.

| Si | Entonces  |
| --- | --- |
| Si no tiene una red virtual...     |  Cree los siguientes elementos:<ul><li><a href="tutorial-configure-networking.md#create-a-virtual-network">Red virtual</a></li><li><a href="../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet">GatewaySubnet</a></li><li><a href="tutorial-configure-networking.md#create-a-virtual-network-gateway">Puerta de enlace de red virtual</a></li></ul>        |
| Ya tiene una red virtual **sin** subred de puerta de enlace...   | Cree los siguientes elementos: <ul><li><a href="../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet">GatewaySubnet</a></li><li><a href="tutorial-configure-networking.md#create-a-virtual-network-gateway">Puerta de enlace de red virtual</a></li></ul>          |
| Ya tiene una red virtual **con** una subred de puerta de enlace... | [Creación de una puerta de enlace de red virtual](tutorial-configure-networking.md#create-a-virtual-network-gateway)   |


### <a name="use-an-existing-virtual-network-gateway"></a>Uso de una puerta de enlace de red virtual existente

[!INCLUDE [connect-expressroute-to-vnet](includes/connect-expressroute-vnet.md)]


## <a name="step-4-validate-the-connection"></a>Paso 4. Validación de la conexión

Debe tener conectividad entre la instancia de Azure Virtual Network donde finaliza ExpressRoute y la nube privada de Azure VMware Solution. 

1. Use una [máquina virtual](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine) dentro de la instancia de Azure Virtual Network en la que finaliza la instancia de ExpressRoute de Azure VMware Solution (consulte [Paso 3. Conexión a Azure Virtual Network con ExpressRoute](#step-3-connect-to-azure-virtual-network-with-expressroute)).  

   1. Inicie sesión en [Azure Portal](https://portal.azure.com).
   2. Vaya a una máquina virtual que se encuentre en estado de ejecución y, en **Configuración**, seleccione **Redes** y seleccione el recurso de la interfaz de red.
      ![Visualización de las interfaces de red](../virtual-network/media/diagnose-network-routing-problem/view-nics.png)
   4. A la izquierda, seleccione **Rutas eficaces**. Verá una lista de prefijos de dirección que se encuentran dentro del bloque CIDR `/22` que especificó durante la fase de implementación.

1. Si desea iniciar sesión en vCenter y NSX-T Manager, abra un explorador web e inicie sesión en la misma máquina virtual que se usa para la validación de rutas de red.  

   Puede identificar las direcciones IP y las credenciales de la consola de NSX-T Manager y vCenter en Azure Portal.  Seleccione una nube privada y, después, **Administrar** > **Identidad**.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Capturas de pantalla de las direcciones URL y las credenciales de NSX Manager y vCenter de la nube privada." border="true":::


## <a name="next-steps"></a>Pasos siguientes

En la siguiente sección, conectará Azure VMware Solution a la red local a través de ExpressRoute.

> [!div class="nextstepaction"]
> [Tutorial: Emparejamiento de entornos locales con una nube privada](tutorial-expressroute-global-reach-private-cloud.md)