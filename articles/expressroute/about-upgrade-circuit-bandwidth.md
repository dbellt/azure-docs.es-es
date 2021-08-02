---
title: Acerca de la actualización del ancho de banda del circuito | Azure ExpressRoute
description: En este artículo, conocerá los procedimientos recomendados para actualizar el ancho de banda del circuito ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 06/08/2021
ms.author: duau
ms.openlocfilehash: 75570beb243fca802ecebebf34edf02b3d8ca9a5
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006498"
---
# <a name="about-upgrading-expressroute-circuit-bandwidth"></a>Acerca de la actualización del ancho de banda del circuito ExpressRoute

ExpressRoute es una conectividad dedicada y privada a la red global de Microsoft. La conectividad se facilita mediante la red de un asociado de ExpressRoute o una conexión directa a los dispositivos de Microsoft Enterprise Edge (MSEE). Una vez que se ha configurado y probado la conectividad física, puede habilitar la conectividad de capa 2 y capa 3 creando un circuito ExpressRoute y configurando el emparejamiento.

## <a name="upgrade-circuit-bandwidth"></a><a name="upgrade"></a>Actualización del ancho de banda del circuito

Para actualizar el ancho de banda del circuito, el asociado de ExpressRoute Direct o de ExpressRoute debe tener [suficiente ancho de banda disponible](#considerations) para que la actualización se realice correctamente.

Si la capacidad está disponible, puede actualizar el circuito mediante los métodos siguientes:

* [Azure Portal](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [PowerShell](expressroute-howto-circuit-arm.md#modify)
* [CLI de Azure](howto-circuit-cli.md#modify)

## <a name="capacity-considerations"></a><a name="considerations"></a>Consideraciones sobre la capacidad

### <a name="insufficient-capacity-for-physical-connection"></a>Capacidad insuficiente para la conexión física

Se crea un circuito ExpressRoute en una conexión física entre Microsoft y un asociado de ExpressRoute. La conexión física tiene una capacidad fija. Si no puede aumentar el tamaño del circuito, significa que la conexión física subyacente del circuito existente no tiene capacidad para la actualización. Deberá crear un circuito si desea cambiar el tamaño del circuito.

Después de crear correctamente el nuevo circuito ExpressRoute, debe vincular las redes virtuales existentes a este circuito. A continuación, puede probar y validar la conectividad del nuevo circuito ExpressRoute antes de desaprovisionar el circuito antiguo. Estos son los pasos de migración recomendados para minimizar el tiempo de desconexión y la interrupción de la carga de trabajo de producción.

### <a name="insufficient-expressroute-partner-bandwidth"></a><a name="bandwidth"></a>Ancho de banda insuficiente del asociado de ExpressRoute

Si no puede crear un nuevo circuito ExpressRoute debido a un error de capacidad, significa que este asociado de ExpressRoute no tiene capacidad para conectarse a Microsoft en esta ubicación de emparejamiento. Póngase en contacto con su asociado de ExpressRoute para solicitar más capacidad.

Una vez aprovisionada la nueva capacidad, puede usar los pasos que se incluyen en la sección [Actualización del ancho de banda del circuito](#upgrade) para crear un circuito, configurar la conectividad y eliminar el circuito anterior.


### <a name="insufficient-expressroute-direct-bandwidth"></a><a name="bandwidth"></a>Ancho de banda insuficiente de ExpressRoute Direct

Si el recurso de ExpressRoute Direct no tiene capacidad suficiente, tiene dos opciones. Puede eliminar los circuitos asociados al recurso de ExpressRoute Direct que ya no son necesarios, o bien crear otro recurso de ExpressRoute Direct. Para más instrucciones sobre cómo administrar el recurso de ExpressRoute Direct, consulte [Configuración de ExpressRoute Direct](how-to-expressroute-direct-portal.md).

## <a name="next-steps"></a>Pasos siguientes

* [Creación y modificación de un circuito](expressroute-howto-circuit-portal-resource-manager.md)
* [Creación y modificación de la configuración de emparejamiento](expressroute-howto-routing-portal-resource-manager.md)
* [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
