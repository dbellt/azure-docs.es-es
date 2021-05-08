---
title: 'Azure VMware Solution by CloudSimple: servicio'
description: Obtenga información sobre el servicio CloudSimple con esta introducción. La creación del servicio permite adquirir nodos, reservarlos y crear nubes privadas.
author: shortpatti
ms.author: v-patsho
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: acf5180e438d244f2eab791f4a2ebff77df6eaa4
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108182289"
---
# <a name="cloudsimple-service-overview"></a>Información general del servicio CloudSimple

El servicio CloudSimple le permite consumir Azure VMware Solution by CloudSimple.  La creación del servicio permite adquirir nodos, reservarlos y crear nubes privadas.  El servicio CloudSimple se crea en cada región de Azure donde está disponible. El servicio define la red perimetral de Azure VMware Solution by CloudSimple. La red perimetral admite servicios que incluyen VPN, ExpressRoute y conectividad de Internet a las nubes privadas.

## <a name="gateway-subnet"></a>Subred de puerta de enlace

Se requiere una subred de puerta de enlace por cada servicio CloudSimple y es única para la región en la que se crea. La subred de puerta de enlace se usa al crear la red perimetral y requiere un bloque CIDR /28.  El espacio de direcciones de subred de puerta de enlace debe ser único. No se debe superponer con otras redes que se comuniquen con el entorno de CloudSimple. Las redes que se comunican con CloudSimple incluyen redes locales y redes virtuales de Azure.  Después de crear una subred de puerta de enlace no se puede eliminar.  La subred de puerta de enlace se quita cuando se elimina el servicio.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre cómo [crear un servicio CloudSimple en Azure](quickstart-create-cloudsimple-service.md).
