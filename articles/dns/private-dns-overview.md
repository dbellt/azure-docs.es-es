---
title: ¿Qué es DNS privado de Azure?
description: Este artículo comienza con información general acerca del servicio de hospedaje de DNS privado en Microsoft Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: overview
ms.date: 04/09/2021
ms.author: rohink
ms.openlocfilehash: eb66facb8d36884205bc9cd8e562ab97f92c3dd8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108745044"
---
# <a name="what-is-azure-private-dns"></a>¿Qué es DNS privado de Azure?

El sistema de nombres de dominio, o DNS, se encarga de traducir o resolver un nombre de servicio en una dirección IP.  Azure DNS es un servicio de hospedaje para dominio y ofrece la resolución de nombres mediante la infraestructura de Microsoft Azure. Azure DNS no solo admite dominios DNS accesibles desde Internet, sino también zonas DNS privadas.

Azure Private DNS proporciona un servicio DNS confiable y seguro para la red virtual. Azure Private DNS administra y resuelve los nombre de dominio en la red virtual sin necesidad de configurar una solución DNS personalizada. Mediante el uso de zonas DNS privadas, puede usar su nombre de dominio personalizado en lugar de los nombres facilitados por Azure durante la implementación. El uso de nombres de dominio personalizados facilita la adaptación de la arquitectura de red virtual a las necesidades de su organización. Proporciona una resolución de nombres para las máquinas virtuales dentro de una red virtual y entre redes virtuales. Además, puede configurar nombres de zonas con una vista de horizonte dividido que permite que una zona DNS privada y otra pública compartan el nombre.

Para resolver los registros de una zona DNS privada de la red virtual, debe vincular esta con la zona. Las redes virtuales vinculadas tienen acceso completo y pueden resolver todos los registros DNS que se publican en la zona privada. También puede habilitar el registro automático en un vínculo de red virtual. Cuando habilite el registro automático en un vínculo de red virtual, los registros DNS para las máquinas virtuales de esa red virtual se registrarán en la zona privada. Cuando el registro automático está habilitado, Azure DNS también actualiza los registros de zona siempre que se cree una máquina virtual, cambie su dirección IP o se elimina.

![Información general de DNS](./media/private-dns-overview/scenario.png)

> [!NOTE]
> Como procedimiento recomendado, no use un dominio *.local* para la zona DNS privada. No todos los sistemas operativos admiten esto.

## <a name="benefits"></a>Ventajas

DNS privado de Azure proporciona las ventajas siguientes:

* **Eliminación de la necesidad de contar con soluciones DNS personalizadas**. Anteriormente, muchos clientes crearon soluciones DNS personalizadas para administrar zonas DNS en su red virtual. Ahora puede administrar las zonas DNS con la infraestructura nativa de Azure, lo que evita tener que crear y administrar soluciones DNS personalizadas.

* **Uso de todos los tipos de registros de DNS comunes**. Azure DNS admite los registros A, AAAA, CNAME, MX, PTR, SOA, SRV y TXT.

* **Administración automática de registros de nombres de host**. Además de hospedar los registros de DNS personalizados, Azure mantiene automáticamente los registros de nombres de host de las máquinas virtuales en las redes virtuales especificadas. En este escenario, puede optimizar los nombres de dominio usados sin necesidad de crear soluciones DNS personalizadas ni de modificar aplicaciones.

* **Resolución de nombres de host entre redes virtuales**. A diferencia de los nombres de host proporcionados por Azure, las zonas DNS privadas pueden compartirse entre redes virtuales. Esta funcionalidad simplifica los escenarios de detección de servicios y de distintas redes, como el emparejamiento de red virtual.

* **Experiencia del usuario y herramientas familiares**. Para reducir la curva de aprendizaje, este servicio usa herramientas de Azure DNS establecidas (Azure Portal, Azure PowerShell, la CLI de Azure, plantillas de Azure Resource Manager y la API REST).

* **Compatibilidad con DNS de horizonte dividido**. Con Azure DNS, puede crear zonas con el mismo nombre que se resuelve en diferentes respuestas desde dentro de una red virtual y desde la red pública de Internet. Un escenario típico de DNS de horizonte dividido es proporcionar una versión dedicada de un servicio para usarla dentro de la red virtual.

* **Disponibilidad en todas las regiones de Azure**. La característica de zonas privadas de Azure DNS está disponible en todas las regiones de Azure de la nube pública de Azure.

## <a name="capabilities"></a>Capacidades

Azure DNS ofrece las funcionalidades siguientes:

* **Registro automático de máquinas virtuales desde una única red virtual vinculada a una zona privada con registro automático habilitado**. Las máquinas virtuales se registran en la zona privada como registros A que apuntan a sus direcciones IP privadas. Cuando se elimina una máquina virtual de un vínculo de red virtual que tiene el registro automático habilitado, Azure DNS también elimina automáticamente el registro DNS correspondiente de la zona privada vinculada.

* **Se admite la resolución DNS de reenvío entre redes virtuales vinculadas a la zona privada**. Para la resolución DNS entre redes virtuales, no hay ninguna dependencia explícita de que las redes virtuales estén emparejadas unas con otras. Sin embargo, es posible que desee emparejar redes virtuales en otras situaciones (por ejemplo, tráfico HTTP).

* **Se admite la búsqueda inversa de DNS dentro del ámbito de la red virtual**. La búsqueda inversa de DNS de una dirección IP privada asociada a una zona privada devuelve un FQDN que incluye el nombre de host/registro y el nombre de zona como sufijo.

## <a name="other-considerations"></a>Otras consideraciones

Azure DNS tiene las siguientes limitaciones:

* Una red virtual concreta se puede vincular a una sola zona privada como si se habilitara el registro automático de los registros DNS de una red virtual. Sin embargo, puede vincular varias redes virtuales a una sola zona DNS.
* El DNS inverso solo funciona en el espacio de direcciones IP privadas de la red virtual vinculada.
* El DNS inverso de una dirección IP privada en una red virtual vinculada devolverá `internal.cloudapp.net` como sufijo predeterminado para la máquina virtual. En el caso de las redes virtuales vinculadas a una zona privada con el registro automático habilitado, el DNS inverso de una dirección IP privada devuelve dos FQDN: uno con el valor predeterminado de sufijo `internal.cloudapp.net` y otro con el sufijo de la zona privada.
* Actualmente no se admite el reenvío condicional de forma nativa. Para habilitar la resolución entre Azure y las redes locales, consulte [Resolución de nombres para las máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).
 
## <a name="pricing"></a>Precios

Para obtener información de precios, consulte [Precios de Azure DNS](https://azure.microsoft.com/pricing/details/dns/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, consulte las [preguntas frecuentes de DNS privado](./dns-faq-private.yml).

* Visite [Información general sobre zonas y registros de DNS](dns-zones-records.md) para obtener información sobre zonas y registros DNS.

* Obtenga información sobre las demás [funcionalidades de red](../networking/fundamentals/networking-overview.md) clave de Azure.