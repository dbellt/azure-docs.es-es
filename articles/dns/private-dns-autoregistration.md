---
title: ¿Qué es la característica de registro automático de zonas privadas de Azure DNS?
description: Información general de la característica de registro automático de zonas privadas de Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 04/26/2021
ms.author: rohink
ms.openlocfilehash: c9ed45d452f2a6ed89ee71826fbee5e107a395ae
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108761640"
---
# <a name="what-is-the-auto-registration-feature-in-azure-dns-private-zones"></a>¿Qué es la característica de registro automático de zonas privadas de Azure DNS?

La característica de registro automático de zonas privadas de Azure DNS administra registros DNS para las máquinas virtuales implementadas en una red virtual. Al [vincular una red virtual con una zona DNS privada](./private-dns-virtual-network-links.md) con esta configuración habilitada. Se crea un registro DNS para cada máquina virtual implementada en la red virtual. 

Para cada máquina virtual, se crean un registro D y un registro PTR. Los registros DNS de las máquinas virtuales recién implementadas también se crean automáticamente en la zona DNS privada vinculada. Cuando se elimina una máquina virtual, los registros DNS asociados también se eliminan de la zona DNS privada.

Para habilitar el registro automático, active la casilla "Habilitar registro automático" al crear el vínculo de red virtual.

:::image type="content" source="./media/privatedns-concepts/enable-autoregistration.png" alt-text="Captura de pantalla de habilitación del registro automático en la página para agregar vínculo de red virtual.":::

## <a name="restrictions"></a>Restricciones

* El registro automático solo funciona para las máquinas virtuales. En el caso de todos los demás recursos, como los equilibradores de carga internos, etc., puede crear registros DNS manualmente en la zona DNS privada vinculada a la red virtual.
* Los registros DNS se crean automáticamente solo para la NIC de la máquina virtual principal. Si las máquinas virtuales tienen más de una NIC, puede crear manualmente los registros DNS para las demás interfaces de red.
* Los registros DNS se crean automáticamente solo si la NIC de la máquina virtual principal usa DHCP. Si usa direcciones IP estáticas, como una configuración con [varias direcciones IP en Azure](../virtual-network/virtual-network-multiple-ip-addresses-portal.md#os-config),el registro automático no creará registros para esa máquina virtual.
* No se admite el registro automático para IPv6 (registros AAAA).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, como el comportamiento específico que se puede esperar con determinadas clases de operaciones, consulte [Preguntas frecuentes de DNS privado](./dns-faq-private.yml).