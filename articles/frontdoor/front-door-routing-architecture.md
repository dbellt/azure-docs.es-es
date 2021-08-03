---
title: 'Azure Front Door: arquitectura de enrutamiento | Microsoft Docs'
description: Este artículo le ayudará a comprender el aspecto de la visión general de la arquitectura de Front Door.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: f4e493b0cbc52c7f8f6165a8198dd19acf352378
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110494341"
---
# <a name="routing-architecture-overview"></a>Introducción a la arquitectura de enrutamiento

Cuando Azure Front Door reciba las solicitudes de cliente, realizará una de estas dos cosas. O bien, responderá a ellas (si habilita el almacenamiento en caché), o las reenviará al servidor back-end de la aplicación adecuado como proxy inverso.

## <a name="selecting-the-front-door-environment-for-traffic-routing-anycast"></a><a name = "anycast"></a>Selección del entorno de Front Door para el enrutamiento del tráfico (Anycast)

El tráfico enrutado en entornos de Azure Front Door aprovecha las ventajas de [Anycast](https://en.wikipedia.org/wiki/Anycast) para DNS (Sistema de nombres de dominio) y para el tráfico HTTP (Protocolo de transferencia de hipertexto), lo cual permite que las solicitudes de usuario lleguen al entorno más cercanos en el mínimo número de saltos de red. Esta arquitectura ofrece mejores tiempos de ida y vuelta para los usuarios finales al maximizar las ventajas de Split TCP (División TCP). Front Door organiza sus entornos en "anillos" principales y de reserva. El anillo exterior tiene entornos que están más próximos a los usuarios, ofreciendo latencias más bajas.  El anillo interior tiene entornos que pueden controlar la conmutación por error para el entorno del anillo exterior en caso de que se produzca algún problema. El anillo exterior es el destino preferido para todo el tráfico y el anillo interior controla el desbordamiento de tráfico desde el anillo exterior. A cada host o dominio de front-end atendido por Front Door se le asigna una IP virtual principal (direcciones de protocolo de Internet virtual), que se anuncia por los entornos de los anillos interno y externo. Solo los entornos del anillo interno anuncian una IP virtual de reserva. 

Esta arquitectura garantiza que las solicitudes de los usuarios finales alcanzan siempre el entorno de Front Door más cercano. Aunque el entorno de Front Door preferido sea incorrecto, todo el tráfico se mueve automáticamente al siguiente entorno más cercano.

## <a name="connecting-to-front-door-environment-split-tcp"></a><a name = "splittcp"></a>Conexión al entorno de Front Door (División TCP)

La [División TCP](https://en.wikipedia.org/wiki/Performance-enhancing_proxy) es una técnica para reducir las latencias y los problemas de TCP mediante la división de una conexión que podría incurrir en unos tiempos de ida y vuelta altos en partes más pequeñas. Con los entornos de Front Door más cercanos a los usuarios finales, las conexiones TCP finalizan dentro del entorno de Front Door. Una conexión TCP con un tiempo de ida y vuelta (RTT) prolongado al servidor back-end de la aplicación se divide en dos conexiones independientes. Una "conexión breve" entre el usuario final y el entorno de Front Door significa que la conexión se establece mediante tres recorridos de ida y vuelta breves en lugar de tres recorridos de ida y vuelta prolongados, lo que mejora la latencia. La "conexión prolongada" entre el entorno de Front Door y el servidor back-end se puede establecer previamente y reutilizar en varias solicitudes de usuario final, lo cual mejora el tiempo de conectividad. El efecto de Split TCP (División TCP) se multiplica al establecer una conexión SSL/TLS (Seguridad de la capa de transporte), ya que hay más viajes de ida y vuelta para proteger la conexión.

## <a name="processing-request-to-match-a-routing-rule"></a>Procesamiento de la solicitud para que coincida con una regla de enrutamiento
Cuando una solicitud llega a un entorno de Front Door, después de establecer una conexión y realizar un protocolo de enlace TLS, el primer paso es hacer que coincida con una regla de enrutamiento. Esto se determina mediante configuraciones en Front Door con las que deben coincidir la solicitud y la regla de enrutamiento en concreto. Obtenga información sobre cómo realiza Front Door la [coincidencia de rutas](front-door-route-matching.md).

## <a name="identifying-available-backends-in-the-backend-pool-for-the-routing-rule"></a>Identificación de los back-end disponibles en el grupo de back-end para la regla de enrutamiento
Cuando Front Door tiene una regla de enrutamiento que coincide con una solicitud entrante, el siguiente paso es conseguir el estado de sondeo de estado del grupo de servidores back-end asociado a esta regla de enrutamiento si no existe almacenamiento en caché. Obtenga información sobre cómo supervisa Front Door el mantenimiento del back-end mediante [Sondeos de estado](front-door-health-probes.md).

## <a name="forwarding-the-request-to-your-application-backend"></a>Reenvío de la solicitud al back-end de aplicación
Por último, suponiendo que no se ha configurado el almacenamiento en caché, se reenvía la solicitud del usuario al "mejor" servidor back-end según la configuración del [método de enrutamiento](front-door-routing-methods.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
