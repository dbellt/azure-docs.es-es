---
title: Tipos de ataques que mitiga Azure DDoS Protection Estándar
description: Obtenga información sobre los tipos de ataques contra los que Azure DDoS Protection Estándar protege.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 8eac365fc5146f315db8f09a67960aa892fa1c85
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107106061"
---
# <a name="types-of-ddos-attacks-overview"></a>Información general sobre tipos de ataques DDoS

El servicio Protección contra DDoS estándar puede mitigar los tipos de ataque siguientes:

- **Ataques volumétricos**: estos ataques desbordan la capa de la red con una gran cantidad de tráfico aparentemente legítimo. Incluyen ataques de tipo "flood" de UDP, de amplificación y otros ataques de tipo "flood" de paquetes falsificados. DDoS Protection Standard mitiga estos posibles ataques de varios gigabytes, ya que los absorbe y los limpia automáticamente aprovechando la escala de red global de Azure.
- **Ataques a protocolos**: estos ataques representan un destino inaccesible al aprovechar una vulnerabilidad en la pila de protocolo en los niveles 3 y 4. Incluyen ataques de tipo "flood" de congestión del servidor de SYN, ataques de reflejo y otros ataques de protocolo. El servicio Protección contra DDoS estándar mitiga estos ataques al diferenciar entre el tráfico malintencionado y el legítimo. Para ello, interactúa con el cliente y bloquea el tráfico malintencionado. 
- **Ataques de nivel de recurso (aplicación)** : estos ataques van dirigidos a paquetes de aplicaciones web y su objetivo es interrumpir la transmisión de datos entre hosts. Incluyen las infracciones de protocolo HTTP, inyección de código SQL, scripts de sitios y otros ataques de nivel 7. Use un firewall de aplicaciones web, por ejemplo el [firewall de aplicaciones web de Azure Application Gateway](../web-application-firewall/ag/ag-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json), así como DDoS Protection Standard, para la defensa frente a estos ataques. También existen ofertas de firewall de aplicaciones web de terceros disponibles en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?page=1&search=web%20application%20firewall).

## <a name="azure-ddos-protection-standard"></a>Protección contra DDoS de Azure estándar

Protección contra DDoS estándar protege los recursos de las redes virtuales, incluidas las direcciones IP públicas asociadas a máquinas virtuales, los equilibradores de carga y las puertas de enlace de aplicaciones. Cuando se combina con el firewall de aplicaciones web de Application Gateway o un firewall de aplicaciones web de terceros implementado en una red virtual con una dirección IP pública, DDoS Protection Standard puede proporcionar una capacidad de mitigación completa de capa 3 a capa 7.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un plan de protección contra DDoS](manage-ddos-protection.md).