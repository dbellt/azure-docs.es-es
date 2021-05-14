---
author: asudbring
ms.service: virtual-network
ms.topic: include
ms.date: 04/08/2021
ms.author: allensu
ms.openlocfilehash: 062cdb7c442ee5556aa454c978bd4aa47058aaf5
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107529565"
---
> [!NOTE]
> Azure proporciona una dirección IP efímera para las instancias de Azure Virtual Machines que no tienen asignada una dirección IP pública o que se encuentran en el grupo de back-end de una instancia de Azure Load Balancer del nivel Básico. El mecanismo de dirección IP efímera proporciona una dirección IP de salida que no se puede configurar. 
>
>La dirección IP efímera se deshabilita cuando se asigna una dirección IP pública a la máquina virtual o cuando se coloca la máquina virtual en el grupo de back-end de una instancia de Load Balancer Estándar con o sin reglas de salida. Si se asigna un recurso de puerta de enlace de [Azure Virtual Network NAT](../articles/virtual-network/nat-overview.md) a la subred de la máquina virtual, la dirección IP efímera se deshabilita.
>
> Para más información sobre las conexiones salientes en Azure, consulte [Uso de la Traducción de direcciones de red de origen (SNAT) para conexiones salientes](../articles/load-balancer/load-balancer-outbound-connections.md).