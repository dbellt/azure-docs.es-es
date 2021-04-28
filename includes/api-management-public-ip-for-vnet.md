---
author: dlepow
ms.service: api-management
ms.topic: include
ms.date: 04/12/2021
ms.author: danlep
ms.openlocfilehash: 6ed30ed2333393e9d8c0222500aaf3f17da6d33d
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531619"
---
* **Una dirección [IPv4 pública de SKU estándar](../articles/virtual-network/public-ip-addresses.md#standard)** , si el cliente usa la versión de API 2021-01-01-preview o posterior. El recurso de la dirección IP pública es necesario al configurar la red virtual para el acceso externo o interno. Con una red virtual interna, la dirección IP pública solo se usa para las operaciones de administración. Obtenga más información sobre las [direcciones IP de API Management](../articles/api-management/api-management-howto-ip-addresses.md).

  * La dirección IP debe estar en la misma región y suscripción que la instancia de API Management y la red virtual.

  * El valor de la dirección IP se asigna como la dirección IPv4 pública virtual de la instancia de API Management en esa región. 

  * Al cambiar de una red virtual externa a otra interna (o viceversa), o al cambiar las subredes de la red o actualizar las zonas de disponibilidad para la instancia de API Management, debe configurar una dirección IP pública diferente. 

  > [!IMPORTANT]
  > Actualmente, Azure Portal usa la versión preliminar de API 2021-01-01 al crear o actualizar una instancia de API Management personalizada. Puede especificar esta versión de API mediante una plantilla de Azure Resource Manager o la API de REST de API Management. Las CLI de Azure y Azure PowerShell admiten actualmente la versión de API 2020-12-01.
