---
title: Opciones de aislamiento de red de Azure Cache for Redis
description: En este artículo, obtendrá información sobre cómo determinar la mejor solución de aislamiento de red para sus necesidades. Analizaremos los aspectos básicos de Azure Private Link, la inserción de Azure Virtual Network (VNet) y las reglas de Azure Firewall con sus ventajas y limitaciones.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: b47462a60e35200908592274a9e1c424eca5ab62
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110655373"
---
# <a name="azure-cache-for-redis-network-isolation-options"></a>Opciones de aislamiento de red de Azure Cache for Redis

En este artículo, obtendrá información sobre cómo determinar la mejor solución de aislamiento de red para sus necesidades. Trataremos los aspectos básicos de Azure Private Link, la inserción de Azure Virtual Network (VNet) y las reglas de Azure Firewall. Analizaremos sus ventajas y limitaciones.  

## <a name="azure-private-link"></a>Azure Private Link

Azure Private Link proporciona conectividad privada desde una red virtual a los servicios PaaS de Azure. Simplifica la arquitectura de red y protege la conexión entre los puntos de conexión de Azure. Para ello, elimina la exposición de los datos a la red pública de Internet.

### <a name="advantages"></a>Ventajas

* Se admite en las instancias Básica, Estándar y Premium de Azure Cache for Redis.
* Mediante [Azure Private Link](../private-link/private-link-overview.md), puede conectarse a una instancia de Azure Cache desde la red virtual a través de un punto de conexión privado. Al punto de conexión se le asigna una dirección IP privada de una subred dentro de la red virtual. Con este vínculo privado, las instancias de caché están disponibles tanto dentro de la red virtual como de forma pública.  
* Una vez creado un punto de conexión privado, el acceso a la red pública se puede restringir a través de la marca `publicNetworkAccess`. Esta marca se establece en `Disabled` de forma predeterminada, lo que solo permitirá el acceso de vínculos privados. Puede establecer el valor en `Enabled` o `Disabled` con una solicitud PATCH. Para más información, consulte Azure Cache for Redis con Azure Private Link (cache-private-link.md).
* Ninguna dependencia de caché externa afectará a las reglas de NSG de VNet.

### <a name="limitations"></a>Limitaciones

* Los grupos de seguridad de red (NSG) se deshabilitan para los puntos de conexión privados. Sin embargo, si hay otros recursos en la subred, la obligatoriedad de NSG se aplicará a esos recursos.
* No se admite todavía la replicación geográfica, las reglas de firewall, la compatibilidad con la consola del portal, varios puntos de conexión por caché en clúster, la persistencia en el firewall ni las cachés insertadas en la red virtual.
* Para conectarse a una caché en clúster, `publicNetworkAccess` debe establecerse en `Disabled`, y solo puede haber una conexión de punto de conexión privado.

> [!NOTE]
> Al agregar un punto de conexión privado a una instancia de caché, todo el tráfico de Redis se mueve al punto de conexión privado debido al DNS.
> Asegúrese de que se ajustan antes las reglas de firewall anteriores.  

## <a name="azure-virtual-network-injection"></a>Inserción de Azure Virtual Network

VNet es el bloque de creación básico de una red privada en Azure. VNet permite que muchos recursos de Azure se puedan comunicar de forma segura entre ellos, con Internet y con redes en el entorno local. Una red virtual es como una red tradicional que usaría en su propio centro de datos. Sin embargo, la red virtual también tiene las ventajas de infraestructura, escalabilidad, disponibilidad y aislamiento de Azure.

### <a name="advantages"></a>Ventajas

* Si una instancia de Azure Cache for Redis se configura con una red virtual, no se puede acceder a ella de forma pública. Solo se puede acceder desde máquinas virtuales y aplicaciones que se encuentren dentro de la red virtual.  
* Cuando se combina VNet con directivas de NSG restringidas, ayuda a reducir el riesgo de la exfiltración de datos.
* La implementación de una red virtual proporciona mayor seguridad y aislamiento para la instancia de Azure Cache for Redis. Las subredes, las directivas de control de acceso y otras características restringen aún más el acceso.
* Se admite la replicación geográfica.

### <a name="limitations"></a>Limitaciones

* Las cachés insertadas en VNet solo están disponibles para Azure Cache for Redis Premium.
* Al usar una caché insertada en una red virtual, tendrá que cambiar esta para almacenar en caché las dependencias, como las CRL/PKI, AKV, Azure Storage, Azure Monitor, etc.  

## <a name="azure-firewall-rules"></a>Reglas de Azure Firewall

[Azure Firewall](../firewall/overview.md) es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure VNet. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad en la nube sin restricciones. Puede crear, aplicar y registrar directivas de aplicaciones y de conectividad de red a nivel central en suscripciones y redes virtuales.  

### <a name="advantages"></a>Ventajas

* Cuando se configuran las reglas de firewall, solo las conexiones de cliente de los intervalos de direcciones IP especificados pueden conectarse a la memoria caché. Siempre se permiten las conexiones de los sistemas de supervisión de Azure Cache for Redis, incluso si se configuran reglas de firewall. También se permiten las reglas de NSG que defina.  

### <a name="limitations"></a>Limitaciones

* Se pueden usar reglas de firewall junto con memorias caché insertadas en una red virtual, pero actualmente no con puntos de conexión privados.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo configurar una [memoria caché insertada de VNet para una instancia Premium de Azure Cache for Redis](cache-how-to-premium-vnet.md).
* Obtenga información sobre cómo [configurar reglas de firewall para todos los niveles de Azure Cache for Redis](cache-configure.md#firewall).
* Obtenga información sobre cómo [configurar puntos de conexión privados para todos los niveles de Azure Cache for Redis](cache-private-link.md).
