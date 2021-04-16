---
author: cherylmc
ms.date: 12/13/2019
ms.service: expressroute
ms.topic: include
ms.author: cherylmc
ms.openlocfilehash: 7aee201b0419b65c7ea8ddcb6f2d42ffaff4711e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "101750294"
---
|  | **Emparejamiento privado** | **Emparejamiento de Microsoft** |  **Emparejamiento público** (en desuso para circuitos nuevos) |
| --- | --- | --- | --- |
| **Número máximo de prefijos admitidos por emparejamiento** |4000 de forma predeterminada, 10.000 con ExpressRoute Premium |200 |200 |
| **Intervalos de direcciones IP admitidas** |Cualquier dirección IP válida de la WAN. |Direcciones IP públicas propiedad suya o de su proveedor de conectividad. |Direcciones IP públicas propiedad suya o de su proveedor de conectividad. |
| **Requisitos del número de sistema autónomo (AS)** |Números de sistema autónomo (AS) públicos y privados Debe poseer el número de AS público si opta por utilizar uno. |Números de sistema autónomo (AS) públicos y privados Sin embargo, debe comprobar la titularidad de las direcciones IP públicas. |Números de sistema autónomo (AS) públicos y privados Sin embargo, debe comprobar la titularidad de las direcciones IP públicas. |
| **Protocolos IP admitidos**| IPv4, IPv6 (versión preliminar) |  IPv4, IPv6 | IPv4 |
| **Direcciones IP de la interfaz de enrutamiento** |Direcciones IP públicas y de RFC1918 |Direcciones IP públicas registradas para usted en los registros de enrutamiento. |Direcciones IP públicas registradas para usted en los registros de enrutamiento. |
| **Compatibilidad con Hash MD5** |Sí |Sí |Sí |