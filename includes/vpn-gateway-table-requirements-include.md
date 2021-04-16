---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: dc018b5d09c9b33c10cd2d54ac6572537e05ed25
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010855"
---
La tabla siguiente enumera los requisitos para las puertas de enlace VPN basadas en enrutamientos y directivas. Esta tabla se aplica a los modelos de implementación del Administrador de recursos y clásico. Para el modelo clásico, las puertas de enlace de VPN basadas en directivas son las mismas que las puertas de enlace estáticas y las basadas en enrutamientos son las mismas que las dinámicas.

|  | **VPN Gateway básica basada en directivas** | **VPN Gateway básica basada en enrutamientos** | **VPN Gateway estándar basada en enrutamientos** | **VPN Gateway de alto rendimiento basada en enrutamientos** |
| --- | --- | --- | --- | --- |
| **Conectividad de sitio a sitio...(S2S)** |Configuración de VPN de PolicyBased |Configuración de VPN de RouteBased |Configuración de VPN de RouteBased |Configuración de VPN de RouteBased |
| **Conectividad de punto a sitio (P2S**) |No compatible |Compatible (puede coexistir con S2S) |Compatible (puede coexistir con S2S) |Compatible (puede coexistir con S2S) |
| **Método de autenticación** |Clave precompartida |Clave precompartida para la conectividad de S2S, Certificados para la conectividad de P2S |Clave precompartida para la conectividad de S2S, Certificados para la conectividad de P2S |Clave precompartida para la conectividad de S2S, Certificados para la conectividad de P2S |
| **Número máximo de conexiones S2S** |1 |10 |10 |30 |
| **Número máximo de conexiones P2S** |No compatible |128 |128 |128 |
| **Compatibilidad con enrutamiento activo (BGP)** (*) |No compatible |No compatible |Compatible |Compatible |

  (*) BGP no es compatible con el modelo de implementación clásica.
