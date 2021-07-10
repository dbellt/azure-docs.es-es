---
title: DHCP y DNS en la descripción de Azure VMware Solution
description: Descripción de Azure VMware Solution, DHCP y DNS.
ms.topic: include
ms.date: 05/28/2021
ms.openlocfilehash: 585f195c36dd02e28cb744142080bee7dab91f6a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110788098"
---
<!-- Used in tutorial-network-checklist.md and configure-dhcp-azure-vmware-solution.md -->

Las aplicaciones y cargas de trabajo que se ejecutan en un entorno de nube privada requieren servicios de resolución de nombres y DHCP para la búsqueda y las asignaciones de direcciones IP. Para proporcionar estos servicios se requiere una infraestructura de DHCP y DNS adecuada. Puede configurar una máquina virtual para proporcionar estos servicios en el entorno de nube privada.  

Utilice el servicio DHCP integrado en NSX o un servidor DHCP local en la nube privada en lugar de enrutar el tráfico DHCP de difusión sobre la WAN de nuevo al entorno local.
