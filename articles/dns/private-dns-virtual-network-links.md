---
title: ¿Qué es un subrecurso de vínculo de red virtual de Azure DNS Private Zones?
description: Información general de los subrecursos de vínculo de red virtual de una zona privada de Azure DNS
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 04/26/2021
ms.author: rohink
ms.openlocfilehash: 9e8ed0e21f1a6a98915dd3db1d5b8c2dcc1d6103
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108763296"
---
# <a name="what-is-a-virtual-network-link"></a>¿Qué es un vínculo de red virtual?

Después de crear una zona DNS privada en Azure, deberá vincularle una red virtual. Una vez vinculada, las máquinas virtuales hospedadas en esa red virtual pueden acceder a la zona DNS privada. Cada zona DNS privada tiene una colección de recursos secundarios de vínculo de red virtual. Cada uno de estos recursos representa una conexión a una red virtual. Una red virtual se puede vincular a una zona DNS privada como red virtual de registro o de resolución.

## <a name="registration-virtual-network"></a>Red virtual de registro

Al [crear un vínculo](./private-dns-getstarted-portal.md#link-the-virtual-network) entre una zona DNS privada y una red virtual. Tiene la opción de habilitar [el registro automático](./private-dns-autoregistration.md). Con esta opción habilitada, la red virtual se convierte en una red virtual de registro para la zona DNS privada. Se crea automáticamente un registro DNS para las máquinas virtuales que se implementan en la red. Se crearán también registros DNS para las máquinas virtuales que ya se han implementado en la red virtual.

Desde la perspectiva de la red virtual, la zona DNS privada se convierte en la zona de registro de esa red virtual. Una zona DNS privada puede tener varias redes virtuales de registro. Sin embargo, cada red virtual solo puede tener una zona de registro asociada.

## <a name="resolution-virtual-network"></a>Resolución de una red virtual

Si decide vincular la red virtual con la zona DNS privada sin registro automático. La red virtual solo se trata como una red virtual de resolución. Los registros DNS de las máquinas virtuales implementadas en esta red virtual no se crearán automáticamente en la zona privada. Sin embargo, las máquinas virtuales implementadas en la red virtual pueden consultar correctamente los registros DNS de la zona privada. Estos registros incluyen registros creados manualmente y registrados automáticamente desde otras redes virtuales vinculadas a la zona DNS privada.

Una zona DNS privada puede tener varias redes virtuales de resolución y una red virtual puede tener varias zonas de resolución asociadas.

## <a name="limits"></a>Límites

Para conocer el número de redes de registro y de resolución que puede vincular a zonas DNS privadas, consulte [Límites de Azure DNS](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-dns-limits)

## <a name="other-considerations"></a>Otras consideraciones

* No se admiten las redes virtuales implementadas mediante el modelo de implementación clásica.

* Solo puede crear un vínculo entre una zona DNS privada y una red virtual.

* Cada vínculo de red virtual en una zona DNS privada debe tener un nombre único en el contexto de la zona DNS privada. Puede tener vínculos con el mismo nombre en distintas zonas DNS privadas.

* Después de crear un vínculo de red virtual, compruebe el campo "Estado del vínculo" del recurso de vínculo de red virtual. En función del tamaño de la red virtual, puede tardar unos minutos antes de que el vínculo sea operativo y el estado del vínculo cambie a *Completado*.

* Cuando se elimina una red virtual, se eliminan automáticamente todos los vínculos de red virtual y los registros DNS registrados automáticamente asociados a ella en diferentes zonas DNS privadas.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo vincular una red virtual a una zona DNS privada mediante [Azure Portal](./private-dns-getstarted-portal.md#link-the-virtual-network)

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, como el comportamiento específico que se puede esperar con determinadas clases de operaciones, consulte [Preguntas frecuentes de DNS privado](./dns-faq-private.yml).