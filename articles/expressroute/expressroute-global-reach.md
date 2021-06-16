---
title: 'Azure ExpressRoute: Conexión a Microsoft Cloud mediante Global Reach'
description: Aprenda el modo en que Global Reach de ExpressRoute puede vincular los circuitos de ExpressRoute para crear una red privada entre las redes locales.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 06/04/2021
ms.author: duau
ms.custom: references_regions
ms.openlocfilehash: 512f2529e5426e97187f5cb2840dcab27a32c1d6
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111541606"
---
# <a name="expressroute-global-reach"></a>Global Reach de ExpressRoute
ExpressRoute es una forma privada y resistente de conectar las redes locales a la nube de Microsoft. Puede acceder a muchos servicios en la nube de Microsoft, como Azure y Microsoft 365, desde su centro de datos privado o la red corporativa. Por ejemplo, puede tener una sucursal en San Francisco con un circuito ExpressRoute en Silicon Valley y otra sucursal en Londres con un circuito ExpressRoute en la misma ciudad. Ambas sucursales tienen conectividad de alta velocidad a recursos de Azure en el sur de Reino Unido y el oeste de EE. UU. Sin embargo, las sucursales no pueden conectarse ni enviarse datos directamente entre sí. En otras palabras, 10.0.1.0/24 puede enviar datos a 10.0.3.0/24 y 10.0.4.0/24, pero NO a 10.0.2.0/24.

![Diagrama que muestra circuitos no vinculados junto con Global Reach de ExpressRoute.][1]

Con **Global Reach de ExpressRoute**, puede vincular los circuitos ExpressRoute para crear una red privada entre las redes locales. En el ejemplo anterior, con la adición de Global Reach de ExpressRoute, la oficina de San Francisco (10.0.1.0/24) puede intercambiar datos directamente con la oficina de Londres (10.0.2.0/24) mediante los circuitos ExpressRoute existentes y a través de la red global de Microsoft. 

![Diagrama que muestra circuitos vinculados junto con Global Reach de ExpressRoute.][2]

## <a name="use-case"></a>Caso de uso
Global Reach de ExpressRoute está diseñado para complementar la implementación de WAN de su proveedor de servicios y conectar sucursales en todo el mundo. Por ejemplo, si su proveedor de servicios opera principalmente en los Estados Unidos y ha vinculado todas sus sucursales en Estados Unidos, pero el proveedor no opera en Japón ni Hong Kong, con Global Reach de ExpressRoute puede trabajar con un proveedor de servicios local y Microsoft conectará sus sucursales a las de los Estados Unidos mediante ExpressRoute y nuestra red global.

![Diagrama que muestra un caso de uso de Global Reach de ExpressRoute.][3]

## <a name="availability"></a>Disponibilidad 
Global Reach de ExpressRoute se admite en los siguientes lugares. 

> [!NOTE] 
> Para habilitar Global Reach de ExpressRoute entre [diferentes regiones geopolíticas](expressroute-locations-providers.md#locations), los circuitos deben ser **SKU Premium**.

* Australia
* Canadá
* Dinamarca
* Francia
* Alemania
* RAE de Hong Kong
* India
* Irlanda
* Japón
* Corea
* Países Bajos
* Nueva Zelanda
* Noruega
* Singapur
* Sudáfrica (solo Johannesburgo)
* Suecia
* Suiza
* Reino Unido
* Estados Unidos

## <a name="next-steps"></a>Pasos siguientes
- Consulte las [Preguntas más frecuentes sobre Global Reach](expressroute-faqs.md#globalreach).
- Aprenda a [habilitar Global Reach](expressroute-howto-set-global-reach.md).
- Aprenda a [vincular un circuito de ExpressRoute con una red virtual](expressroute-howto-linkvnet-arm.md).

<!--Image References-->
[1]: ./media/expressroute-global-reach/1.png "diagrama sin Global Reach"
[2]: ./media/expressroute-global-reach/2.png "diagrama con Global Reach"
[3]: ./media/expressroute-global-reach/3.png "caso de uso de global reach"
