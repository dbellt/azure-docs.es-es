---
title: Filtrado de FQDN de Azure Firewall en reglas de red
description: Cómo usar el filtrado de FQDN de Azure Firewall en reglas de red
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 06/11/2021
ms.author: victorh
ms.openlocfilehash: f28092c050e58c6ef90a3faf07034be21198008a
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112017206"
---
# <a name="use-fqdn-filtering-in-network-rules"></a>Uso del filtrado de FQDN en reglas de red

Un nombre de dominio completo (FQDN) representa un nombre de dominio de un host o una o varias direcciones IP. Puede usar FQDN en reglas de red basadas en la resolución DNS en la directiva de firewall y Azure Firewall. Esta funcionalidad permite filtrar el tráfico saliente con cualquier protocolo TCP/UDP (incluidos NTP, SSH, RDP y mucho más). Debe habilitar el proxy DNS para usar FQDN en sus reglas de red. Para obtener más información, consulte [Configuración DNS de Azure Firewall](dns-settings.md).

> [!NOTE]
> Por diseño, el filtrado de FQDN no admite caracteres comodín.

## <a name="how-it-works"></a>Funcionamiento

Una vez que defina qué servidor DNS necesita su organización (Azure DNS o su propio DNS personalizado), Azure Firewall traduce el FQDN a una o varias direcciones IP basadas en el servidor DNS seleccionado. Esta traducción se produce para el procesamiento de reglas de red y de aplicación.

Cuando se produce una nueva resolución DNS, se agregan nuevas direcciones IP a las reglas de firewall. Las direcciones IP antiguas que el servidor DNS ya no devuelve expiran en 15 minutos. Las reglas de Azure Firewall se actualizan cada 15 segundos a partir de la resolución DNS de los FQDN en reglas de red.

### <a name="differences-in-application-rules-vs-network-rules"></a>Diferencias en las reglas de aplicación frente a las reglas de red

- El filtrado de FQDN en reglas de aplicación para HTTP/S y MSSQL se basa en un proxy transparente de nivel de aplicación y el encabezado SNI. Como tal, puede distinguir entre dos FQDN que se resuelven en la misma dirección IP. Este no es el caso del filtrado de FQDN en reglas de red. 

   Use siempre reglas de aplicación cuando sea posible:
  - Si el protocolo es HTTP/S o MSSQL, use reglas de aplicación para el filtrado de FQDN.
  - Para servicios como AzureBackup, HDInsight, etc., use reglas de aplicación con etiquetas FQDN.
  - Para cualquier otro protocol, puede usar reglas de aplicación o de red para el filtrado de FQDN.

## <a name="next-steps"></a>Pasos siguientes

[Configuración DNS de Azure Firewall](dns-settings.md)
