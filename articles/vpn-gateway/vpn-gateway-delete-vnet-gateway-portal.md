---
title: 'Eliminación de una puerta de enlace de red virtual: portal'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo eliminar una puerta de enlace de red virtual mediante Azure Portal.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 02/10/2021
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 0510292b27d61c9142edd710e903ebad74b3d244
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228935"
---
# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>Eliminación de una puerta de enlace de red virtual mediante el portal

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (clásico)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)

Este artículo lo ayuda a eliminar una puerta de enlace de red virtual. Hay un par de enfoques que puede adoptar cuando desee eliminar una puerta de enlace correspondiente a una configuración de puerta de enlace VPN.

* Si quiere eliminar todo el contenido y volver a empezar, como en el caso de un entorno de prueba, puede eliminar el grupo de recursos. Cuando se elimina un grupo de recursos, se quitan todos los recursos de él. Este método solo se recomienda si no desea conservar los recursos del grupo en cuestión. Con este enfoque no podrá eliminar de forma selectiva solo unos pocos recursos.

* Si desea mantener algunos de los recursos en el grupo de recursos, será algo más complicado eliminar una puerta de enlace de red virtual. Antes de poder eliminar la puerta de enlace de red virtual, primero debe quitar todos los recursos que dependen de la puerta de enlace. Los pasos que seguirá dependerán del tipo de conexiones que creó y de los recursos dependientes de cada conexión.

> [!IMPORTANT]
> Los pasos descritos en este artículo se aplican al modelo de implementación de Resource Manager. Para eliminar una puerta de enlace VPN implementada mediante el modelo de implementación clásica, use los pasos que aparecen en el artículo sobre la [eliminación clásica de una puerta de enlace](vpn-gateway-delete-vnet-gateway-classic-powershell.md).

## <a name="delete-a-vpn-gateway"></a>Eliminación de una instancia de VPN Gateway

Para eliminar una puerta de enlace de red virtual, primero debe eliminar cada recurso que pertenece a la puerta de enlace de red virtual. Los recursos deben eliminarse en un orden determinado debido a las dependencias.

[!INCLUDE [delete gateway](../../includes/vpn-gateway-delete-vnet-gateway-portal-include.md)]

En este punto, se elimina la puerta de enlace de red virtual.

### <a name="to-delete-additional-resources"></a>Para eliminar recursos adicionales

Los pasos siguientes lo ayudan a eliminar todos los recursos que ya no se usan.

#### <a name="to-delete-the-local-network-gateway"></a>Para eliminar la puerta de enlace de red local

1. En **Todos los recursos**, busque las puertas de enlace de red local asociadas a cada conexión.
1. En la hoja **Información general** de la puerta de enlace de red local, haga clic en **Eliminar**.

#### <a name="to-delete-the-public-ip-address-resource-for-the-gateway"></a>Para eliminar el recurso de dirección IP pública para la puerta de enlace

1. En **Todos los recursos**, busque el recurso de dirección IP pública que se asoció a la puerta de enlace. Si la puerta de enlace de red virtual era activa-activa, verá dos direcciones IP públicas.
1. En la página **Introducción general** de la dirección IP pública, haga clic en **Eliminar** y después en **Sí** para confirmar.

#### <a name="to-delete-the-gateway-subnet"></a>Para eliminar la subred de la puerta de enlace

1. En **Todos los recursos**, busque la red virtual. 
1. En la hoja **Subredes**, haga clic en el elemento **GatewaySubnet** y después en **Eliminar**. 
1. Haga clic en **Sí** para confirmar que desea eliminar la subred de la puerta de enlace.

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="deleterg"></a>Eliminación de una puerta de enlace VPN mediante la eliminación del grupo de recursos

Si no desea mantener ninguno de los recursos del grupo, sino que desea empezar de nuevo, puede eliminar dicho grupo al completo. Se trata de una forma rápida de quitarlos todos. Los siguientes pasos se aplican solo al modelo de implementación de Resource Manager.

1. En **Todos los recursos**, busque el grupo de recursos y haga clic para abrir la hoja.
1. Haga clic en **Eliminar**. En la hoja Eliminar, revise los recursos afectados. Asegúrese de que desea eliminar todos estos recursos. Si no es así, siga los pasos de Eliminación de una puerta de enlace VPN en la parte superior de este artículo.
1. Para continuar, escriba el nombre del grupo de recursos que quiere eliminar y, después, haga clic en **Eliminar**.
