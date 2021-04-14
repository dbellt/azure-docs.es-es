---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 04/09/2018
ms.author: jdial
ms.custom: include file
ms.openlocfilehash: 68ce927c05f7179cca0aa2833460b9550f0a82d2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96028381"
---
> [!div class="op_single_selector"]
> * [Azure Portal](../articles/virtual-network/virtual-network-multiple-ip-addresses-portal.md)
> * [PowerShell](../articles/virtual-network/virtual-network-multiple-ip-addresses-powershell.md)
> * [CLI de Azure](../articles/virtual-network/virtual-network-multiple-ip-addresses-cli.md)
>

Una máquina virtual (VM) de Azure tiene una o varias interfaces de red (NIC) asociadas a ella. Una NIC puede tener una o varias direcciones IP públicas o privadas estáticas o dinámicas asignadas. La asignación de varias direcciones IP a una VM permite las siguientes capacidades:

* Hospede varios sitios web o servicios con direcciones IP y certificados SSL diferentes en un único servidor.
* Actúe como aplicación de red virtual, como un firewall o equilibrador de carga.
* La capacidad de agregar cualquier dirección IP privada para cualquiera de las NIC a un grupo de servidores de back-end de Azure Load Balancer. En el pasado, solo la dirección IP principal para la NIC principal podía agregarse a un grupo de back-end. Para más información sobre cómo equilibrar la carga de varias configuraciones de IP, lea el artículo [Load balancing multiple IP configurations](../articles/load-balancer/load-balancer-multiple-ip.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Equilibrio de carga de varias configuraciones de IP).

Cada NIC conectada a una VM tiene una o varias configuraciones de IP asociadas. Se asigna a cada configuración una dirección IP privada estática o dinámica. Cada configuración también puede tener un recurso de dirección IP pública asociado. Un recurso de dirección IP pública tiene una dirección IP pública dinámica o estática asignada. Para más información sobre las direcciones IP en Azure, lea el artículo sobre [direcciones IP en Azure](../articles/virtual-network/public-ip-addresses.md). 

Existe un límite para el número de direcciones IP privadas que pueden asignarse a una NIC. También existe un límite para el número de direcciones IP públicas que pueden usarse en una suscripción de Azure. Para más información, consulte el artículo sobre los [límites de Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).