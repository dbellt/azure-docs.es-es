---
title: Redes y conectividad de Azure VMware Solution
description: Descripción de redes y conectividad de Azure VMware Solution.
ms.topic: include
ms.date: 05/28/2021
ms.openlocfilehash: 66ea12565aa84661a4019e5685a5544792591b0c
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111350779"
---
<!-- Used in introduction.md and concepts-networking.md -->

Azure VMware Solution ofrece un entorno de nube privada accesible desde recursos locales y basados en Azure. Servicios como Azure ExpressRoute y las conexiones VPN o Azure Virtual WAN proporcionan la conectividad. Estos servicios requieren intervalos de direcciones de red y puertos de firewall específicos para habilitar los servicios.

Cuando se implementa una nube privada, se crean redes privadas para la administración, el aprovisionamiento y vMotion. Use estas redes privadas para acceder a vCenter y NSX-T Manager, así como para la implementación o vMotion de máquinas virtuales.  

Global Reach de ExpressRoute se usa para conectar las nubes privadas a entornos locales. La conexión requiere una red virtual con un circuito ExpressRoute en el entorno local en su suscripción.

Las máquinas virtuales (VM) implementadas en la nube privada son accesibles a través de Internet mediante la funcionalidad de IP pública de Azure Virtual WAN.  De forma predeterminada, el acceso a Internet está deshabilitado para las nuevas nubes privadas. Para más información, consulte [Habilitación del acceso a la red pública de Internet en Azure VMware Solution](../enable-public-internet-access.md).



