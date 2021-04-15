---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: db4e27c0972a93d870d0a6565f1bd3212146df62
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96581230"
---
Si arranca en un entorno sin DHCP, siga estos pasos para implementar la máquina virtual para Data Box Gateway.

1. [Conéctese a la interfaz de Windows PowerShell del dispositivo](#connect-to-the-powershell-interface).
2. Use el cmdlet `Get-HcsIpAddress` para enumerar las interfaces de red habilitadas en el dispositivo virtual. Si el dispositivo tiene una única interfaz de red habilitada, el nombre predeterminado asignado a esta interfaz es `Ethernet`.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```
    [10.100.10.10]: PS>Get-HcsIpAddress

    OperationalStatus : Up
    Name              : Ethernet
    UseDhcp           : True
    IpAddress         : 10.100.10.10
    Gateway           : 10.100.10.1
    ```

3. Utilice el cmdlet `Set-HcsIpAddress` para configurar la red. Observe el ejemplo siguiente:

    ```
    Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1
    ```

