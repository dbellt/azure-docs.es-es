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
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: da284bd65483392d60442bec62394c008ca38a92
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720698"
---
El cmdlet **Get-AzPeeringLocation** de PowerShell devuelve una lista de ubicaciones de emparejamiento con el parámetro obligatorio `Kind`, que se usará en pasos posteriores:

```powershell
Get-AzPeeringLocation -Kind "Exchange"
```

Las ubicaciones de emparejamiento de Exchange contienen los siguientes campos:
* ExchangeName
* PeeringLocation
* Country
* PeeringDBFacilityId
* PeeringDBFacilityLink
* MicrosoftIPv4Address
* MicrosoftIPv6Address

Para comprobar que está presente en la instalación de emparejamiento deseada consulte [PeeringDB](https://www.peeringdb.com).

En este ejemplo se muestra cómo usar Seattle como ubicación de emparejamiento para crear un emparejamiento.

```powershell
$exchangeLocations = Get-AzPeeringLocation -Kind Exchange
$exchangeLocation = $exchangeLocations | where {$_.PeeringLocation -eq "Seattle"}

#check the location metadata
$exchangeLocation

ExchangeName          : Columbia IX
PeeringLocation       : Seattle
Country               : US
PeeringDBFacilityId   : 99999
PeeringDBFacilityLink : https://www.peeringdb.com/ix/99999
MicrosoftIPv4Address  : 10.12.97.129
MicrosoftIPv6Address  :

ExchangeName          : Equinix Seattle
PeeringLocation       : Seattle
Country               : US
PeeringDBFacilityId   : 11
PeeringDBFacilityLink : https://www.peeringdb.com/ix/11
MicrosoftIPv4Address  : 198.32.134.152
MicrosoftIPv6Address  : 2001:504:12::15

...

```
