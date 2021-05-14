---
title: Continuidad empresarial de Azure DDoS Protection estándar| Microsoft Docs
description: Obtenga información acerca de qué hacer si se produce una interrupción de un servicio de Azure que afecta a las instancias de Azure DDoS Protection estándar.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/16/2021
ms.author: yitoh
ms.topic: article
ms.openlocfilehash: 5085f1584a737e75adccf4ec23bf709bede28a4b
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107730388"
---
# <a name="azure-ddos-protection-standard--business-continuity"></a>Azure DDoS Protection estándar: continuidad empresarial

La continuidad empresarial y recuperación ante desastres en Azure DDoS Protection estándar permite que la empresa continúe en funcionamiento en caso de que se produzca una interrupción. En este artículo se describe la disponibilidad (dentro de la región) y la recuperación ante desastres.

## <a name="overview"></a>Información general
Azure DDoS Protection estándar protege las direcciones IP públicas en redes virtuales. La protección se puede habilitar fácilmente en cualquier red virtual nueva o existente y no requiere cambios en las aplicaciones ni los recursos.

Una red virtual (VNet) es una representación lógica de su red en la nube. Las redes virtuales sirven como límite de confianza para hospedar los recursos, como Azure Application Gateway, Azure Firewall y Azure Virtual Machines. Una red virtual se crea dentro del ámbito de una región. Puede *crear* redes virtuales con el mismo espacio de direcciones en dos regiones diferentes (por ejemplo, Este de EE. UU. y Oeste de EE. UU.), pero no se pueden conectar entre sí porque tienen el mismo espacio de dirección. 

## <a name="business-continuity"></a>Continuidad empresarial

La aplicación puede sufrir diferentes formas de interrupción. Por ejemplo, es posible que el servicio se corte por completo en una región determinada como consecuencia de un desastre natural o que se produzca un desastre parcial debido a un error en varios dispositivos o servicios. El impacto en las redes virtuales protegidas es diferente en cada una de estas situaciones.

**P: ¿Qué se puede hacer si se produce una interrupción en una región entera? Por ejemplo, ¿y si debido a un desastre natural el servicio se ha cortado por completo en una región? ¿Qué ocurre con las redes virtuales hospedadas en la región?**

A. La red virtual y los recursos de la región afectada permanecen inaccesibles durante el tiempo que dure la interrupción del servicio.

![Diagrama de red virtual simple](../virtual-network/media/virtual-network-disaster-recovery-guidance/vnet.png)

**P: ¿Qué se puede hacer para volver a crear la misma red virtual en una región distinta?**

A. Las redes virtuales son recursos bastante ligeros. Puede invocar las API de Azure para crear una red virtual con el mismo espacio de direcciones en una región distinta. Para volver a crear el mismo entorno que estaba presente en la región afectada, tendrá que realizar llamadas API para volver a implementar los recursos en las redes virtuales que tenía. Si tiene conectividad local, como una implementación híbrida, tendrá que implementar una nueva instancia de VPN Gateway y conectarse a su red local.

Para crear una red virtual, consulte [Creación de una red virtual](../virtual-network/manage-virtual-network.md#create-a-virtual-network).

**P: ¿Es posible volver a crear una réplica de una red virtual de una región determinada en otra región de antemano?**

A. Sí, puede crear dos redes virtuales con el mismo espacio de direcciones IP privadas y los recursos en dos regiones diferentes antes de que suceda algo. Si hospedaba servicios orientados a Internet en la red virtual, podría haber configurado Traffic Manager para distribuir geográficamente el tráfico en la región que está activa. Sin embargo, dos redes virtuales que tengan el mismo espacio de direcciones no se pueden conectar con la red local, ya que se producirían problemas de enrutamiento. Si se produjera un desastre y la pérdida de una red virtual en una región, podría conectar la otra red virtual de la región disponible, con el espacio de direcciones coincidente con la red local.

Para crear una red virtual, consulte [Creación de una red virtual](../virtual-network/manage-virtual-network.md#create-a-virtual-network).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un plan de protección contra DDoS](manage-ddos-protection.md).