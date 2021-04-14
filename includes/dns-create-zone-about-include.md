---
author: vhorne
ms.service: dns
ms.topic: include
ms.date: 11/25/2018
ms.author: victorh
ms.openlocfilehash: 74031a8dbc9b64d6a09533789eed1296ff334d47
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186976"
---
Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Para iniciar el hospedaje de su dominio en DNS de Azure, debe crear una zona DNS para ese nombre de dominio. Cada registro DNS del dominio se crea luego en esta zona DNS.

Por ejemplo, puede que el dominio "contoso.com" contenga varios registros DNS, como "mail.contoso.com" (para un servidor de correo) y "www.contoso.com" (para un sitio web).

Al crear una zona DNS de Azure DNS:

* El nombre de la zona debe ser único en el grupo de recursos y la zona no debe existir aún. De lo contrario, la operación presentará un error.
* El mismo nombre de zona podrá reutilizarse en otro grupo de recursos o en otra suscripción de Azure.
* Cuando varias zonas comparten el mismo nombre, se asigna a cada instancia diferentes direcciones del servidor. Solo se puede configurar un único conjunto de direcciones con el registrador de nombres de dominio.

> [!NOTE]
> No tiene que poseer un nombre de dominio para crear una zona DNS con dicho nombre de dominio en DNS de Azure. Sin embargo, es necesario ser propietario del dominio para configurar los servidores de nombres en DNS de Azure como los servidores de nombres correctos para el nombre de dominio con el registrador de nombres de dominio.
> 
> Para más información, vea [Delegación de un dominio en DNS de Azure](../articles/dns/dns-domain-delegation.md).