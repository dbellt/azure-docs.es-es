---
title: 'Azure VMware Solution by CloudSimple: conexión de una nube privada a la red de Azure mediante ExpressRoute'
description: Describe cómo establecer la conexión del entorno de nube privada en CloudSimple a la red virtual de Azure mediante ExpressRoute
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0ebf9e16f5ddaf38d1427342106dffe6ca4d3648
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176943"
---
# <a name="connect-your-cloudsimple-private-cloud-environment-to-the-azure-virtual-network-using-expressroute"></a>Conexión del entorno de nube privada de CloudSimple a la red virtual de Azure mediante ExpressRoute

Su nube privada de CloudSimple se puede conectar a la red virtual de Azure mediante Azure ExpressRoute.  Esta conexión de alto ancho de banda y baja latencia permite acceder a los servicios que se ejecutan en la suscripción de Azure desde el entorno de nube privada.

La conexión de red virtual le permite:

* Uso de Azure como destino de copia de seguridad para máquinas virtuales en su nube privada.
* Implementar servidores KMS en su suscripción de Azure para cifrar el almacén de datos vSAN de la nube privada.
* Usar aplicaciones híbridas donde el nivel de web de la aplicación se ejecuta en la nube pública mientras la aplicación y los niveles de base de datos se ejecutan en su nube privada.

![Conexión de Azure ExpressRoute a la red virtual](media/cloudsimple-azure-network-connection.png)

## <a name="set-up-a-virtual-network-connection"></a>Configurar una conexión de red virtual

Para configurar la conexión de red virtual a una nube privada, necesitará la clave de autorización, el URI del circuito del mismo nivel y acceso a su suscripción de Azure. Esta información está disponible en la página Virtual Network Connection en el portal de CloudSimple. Para instrucciones, consulte [Obtención de información de emparejamiento para una conexión de red virtual de Azure a CloudSimple](virtual-network-connection.md). Si tiene algún problema para obtener la información, envíe una <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">solicitud de soporte técnico</a>.

> [!TIP]
> Si ya tiene una red virtual de Azure, una subred de puerta de enlace y una puerta de enlace de red virtual, puede ir directamente al paso 4.

1. Cree una red virtual en la suscripción de Azure y compruebe que el espacio de direcciones que seleccione es diferente del de la nube privada.  Si ya tiene una red virtual de Azure, puede usarla.  Para información detallada, consulte [Creación de una red virtual mediante Azure Portal](../virtual-network/quick-create-portal.md).
2. Cree la subred de puerta de enlace en la red virtual de Azure.  Si ya tiene una subred virtual de puerta de enlace en su red virtual de Azure, puede usarla. Para información detallada, consulte [Creación de la subred de puerta de enlace](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet).
3. Cree la puerta de enlace de red virtual en su red virtual.  Si ya tiene una puerta de enlace de red virtual, puede usarla. Para información detallada, consulte [Creación de la puerta de enlace de red virtual](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-virtual-network-gateway).
4. Cree la conexión entre la red virtual y la nube privada; para ello, rescate la clave de autorización, tal como se describe en [Conexión de una red virtual con un circuito: otra suscripción](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription).

> [!WARNING]
> Si usa una puerta de enlace de red virtual existente y tiene una conexión ExpressRoute a la misma ubicación que el circuito ExpressRoute de CloudSimple, la conexión no se establecerá.  Cree una nueva red virtual y siga los pasos anteriores.

## <a name="test-the-virtual-network-connection"></a>Prueba de la conexión de red virtual

Una vez creada la conexión, puede comprobar el estado de la conexión si selecciona **Propiedades** en **Configuración**.  El estado y el estado de aprovisionamiento deberían ser **correctos**.

![Estado de la conexión](media/azure-expressroute-connection.png)

Para probar la conexión de red virtual:

1. Cree una máquina virtual en su suscripción de Azure.
2. Busque la dirección IP del centro privado de la nube privada (consulte el correo electrónico de bienvenida).
3. Haga ping a su centro virtual de Cloud desde la máquina virtual creada en la red virtual de Azure.
4. Haga ping a la máquina virtual de Azure desde una máquina virtual que se ejecute en su centro virtual de la nube privada.

Si tiene algún problema para establecer la conexión, envíe una <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">solicitud de soporte técnico</a>.
