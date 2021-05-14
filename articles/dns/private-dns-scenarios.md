---
title: 'Escenarios de zonas privadas: Azure DNS'
description: En este artículo, obtendrá información sobre los escenarios comunes de uso de Azure DNS Private Zones.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 04/27/2021
ms.author: rohink
ms.openlocfilehash: 5a2572af0fd312efeacb8544b653db1b35809244
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108127828"
---
# <a name="azure-dns-private-zones-scenarios"></a>Escenarios de zonas privadas de Azure DNS

Azure DNS Private Zones proporciona la resolución de nombres de una red virtual, así como entre redes virtuales. En este artículo, echaremos un vistazo a algunos escenarios comunes que pueden sacar provecho a esta característica.

## <a name="scenario-name-resolution-scoped-to-a-single-virtual-network"></a>Escenario: Resolución de nombres con ámbito de una única red virtual

En este escenario, tiene una red virtual en Azure que contiene varios recursos, entre ellos máquinas virtuales. Necesita resolver los recursos de la red virtual mediante un nombre de dominio específico (zona DNS). También necesita que la resolución de nomenclatura sea privada y no sea accesible desde Internet. Por último, necesita que Azure registre automáticamente las VM en la zona DNS.

Este escenario se muestra a continuación. La red virtual denominada "A" contiene dos VM (VNETA-VM1 y VNETA-VM2). Cada VM tiene una dirección IP privada asociada. Una vez que haya creado una zona privada, por ejemplo `contoso.com`, y vincule la red virtual "A" como una red virtual de registro, Azure DNS creará automáticamente dos registros "A" en la zona que hace referencia a las dos VM. Las consultas de VNETA-VM1 pueden resolver `VNETA-VM2.contoso.com` y recibirán una respuesta DNS que contiene la dirección IP privada de VNETA-VM2.
También puede realizar una consulta DNS inversa (PTR) para la dirección IP privada de VNETA-VM1 (10.0.0.1) desde VNETA-VM2. La respuesta DNS contendrá el nombre VNETA-VM1, según lo previsto. 

![Resolución de una única red virtual](./media/private-dns-scenarios/single-vnet-resolution.png)

## <a name="scenario-name-resolution-across-virtual-networks"></a>Escenario: Resolución de nombres entre redes virtuales

En este escenario, debe asociar una zona privada a varias redes virtuales. Asimismo, puede implementar esta solución en varias arquitecturas de red, como el modelo radial. Esta configuración se establece cuando se usa una red virtual del centro de conectividad para conectar varias redes virtuales radiales juntas. La red virtual del centro de conectividad central se puede vincular como red virtual de registro, y las redes virtuales radiales se pueden vincular como redes virtuales de resolución. 

En el diagrama siguiente se muestra una versión sencilla de este escenario donde solo hay dos redes virtuales: A y B. La A se ha designado como red virtual de registro y la B como red virtual de resolución. Lo que se pretende es que ambas redes virtuales compartan una zona común `contoso.com`. Cuando se crea la zona, las redes virtuales definidas como registro registrarán automáticamente los registros DNS de las VM de la red virtual (VNETA-VM1 y VNETA-VM2). También puede agregar manualmente registros DNS a la zona de las VM de la red virtual de resolución B. Con esta configuración, verá que se produce el comportamiento siguiente para las consultas DNS inversas y desviadas:
* Una consulta DNS desde VNETB-VM1 en la red virtual de resolución B, de VNETA-VM1.contoso.com, recibirá una respuesta DNS que contiene la dirección IP privada de VNETA-VM1.
* Una consulta DNS inversa (PTR) desde VNETB-VM2 en la red virtual de resolución B, de 10.1.0.1, recibirá una respuesta DNS que contiene el FQDN VNETB-VM1.contoso.com.  
* Una consulta DNS inversa (PTR) desde VNETB-VM3 en la red virtual de resolución B, de 10.0.0.1, recibirá NXDOMAIN. El motivo es que el ámbito de las consultas DNS inversas es únicamente la misma red virtual. 

![Resoluciones de varias redes virtuales](./media/private-dns-scenarios/multi-vnet-resolution.png)

## <a name="scenario-split-horizon-functionality"></a>Escenario: Funcionalidad de horizonte dividido

En este escenario, necesita una resolución de nomenclatura diferente que dependa de dónde se encuentra el cliente para la misma zona DNS. Es posible que tenga una versión privada y una versión pública de la aplicación que tenga diferentes funciones o comportamientos. Tenga en cuenta que es necesario usar el mismo nombre de dominio para ambas versiones. Este escenario se puede lograr mediante la creación de una zona pública y privada en Azure DNS con el mismo nombre. 

En el siguiente diagrama, se ilustra este escenario. La red virtual denominada "A" contiene dos VM (VNETA-VM1 y VNETA-VM2). Ambas tienen configurada una dirección IP privada y una IP pública. Se creará una zona DNS pública denominada `contoso.com` y se registrarán las direcciones IP públicas de estas VM como registros DNS dentro de la zona. También se crea una zona DNS privada denominada `contoso.com`. Ya ha definido la red virtual A como una red virtual de registro. A continuación, Azure registra automáticamente las VM como registros de A en la zona privada y apunta a sus direcciones IP privadas.

Asimismo, cuando un cliente de Internet emite una consulta DNS para buscar `VNETA-VM1.contoso.com`, Azure devolverá el registro de dirección IP pública de la zona pública. Si se emite la misma consulta DNS desde otra máquina virtual (por ejemplo, VNETA-VM2) de la misma red virtual A, Azure devolverá el registro de dirección IP privada de la zona privada. 

![Resolución de cerebro dividido](./media/private-dns-scenarios/split-brain-resolution.png)

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de las zonas DNS privadas, consulte [Using Azure DNS for private domains](private-dns-overview.md) (Uso de Azure DNS para dominios privados).

Más información sobre cómo [crear una zona DNS privada](./private-dns-getstarted-powershell.md) en Azure DNS.

Visite [Información general sobre zonas y registros de DNS](dns-zones-records.md) para obtener más información sobre zonas y registros DNS.

Obtenga información sobre las demás [funcionalidades de red](../networking/fundamentals/networking-overview.md) clave de Azure.
