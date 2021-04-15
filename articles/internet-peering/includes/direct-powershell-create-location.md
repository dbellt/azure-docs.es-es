---
title: archivo de inclusión
titleSuffix: Azure
description: archivo de inclusión
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: dbaa0b5fc87cb5393b323b8a9b7a38b72efe9518
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81680745"
---
El cmdlet **Get-AzPeeringLocation** de PowerShell devuelve una lista de ubicaciones de emparejamiento con el parámetro obligatorio `Kind`, que se usará en pasos posteriores:

```powershell
Get-AzPeeringLocation -Kind Direct
```

Las ubicaciones de emparejamiento directo contienen los siguientes campos:
* PeeringLocation 
* Country
* PeeringDBFacilityId
* PeeringDBFacilityLink
* BandwidthOffers

Para comprobar que está presente en la instalación de emparejamiento deseada, consulte [PeeringDB](https://wwww.peeringdb.com).

En este ejemplo se muestra cómo usar Seattle como ubicación de emparejamiento para crear un emparejamiento directo.

```powershell
$peeringLocations = Get-AzPeeringLocation -Kind Direct
$peeringLocation = $peeringLocations | where {$_.PeeringLocation -contains "Seattle"}
$peeringLocation

PeeringLocation       : Seattle
Address               : 2001 Sixth Avenue
Country               : US
PeeringDBFacilityId   : 71
PeeringDBFacilityLink : https://www.peeringdb.com/fac/71
BandwidthOffers       : {10Gbps, 100Gbps}
```