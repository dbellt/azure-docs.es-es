---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: ca060e75e50a3e2327fc0516c3cfc9550afbf90f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96581239"
---
1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-HcsApplianceInfo` para obtener la información del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```
    [10.100.10.10]: PS>Get-HcsApplianceInfo
    
    Id                            : b2044bdb-56fd-4561-a90b-407b2a67bdfc
    FriendlyName                  : DBE-NBSVFQR94S6
    Name                          : DBE-NBSVFQR94S6
    SerialNumber                  : HCS-NBSVFQR94S6
    DeviceId                      : 40d7288d-cd28-481d-a1ea-87ba9e71ca6b
    Model                         : Virtual
    FriendlySoftwareVersion       : Data Box Gateway 1902
    HcsVersion                    : 1.4.771.324
    IsClustered                   : False
    IsVirtual                     : True
    LocalCapacityInMb             : 1964992
    SystemState                   : Initialized
    SystemStatus                  : Normal
    Type                          : DataBoxGateway
    CloudReadRateBytesPerSec      : 0
    CloudWriteRateBytesPerSec     : 0
    IsInitialPasswordSet          : True
    FriendlySoftwareVersionNumber : 1902
    UploadPolicy                  : All
    DataDiskResiliencySettingName : Simple
    ApplianceTypeFriendlyName     : Data Box Gateway
    IsRegistered                  : False
    ```

    Esta es una tabla que resume alguna de la información importante del dispositivo:

    | Parámetro | Descripción |
    |-----------|-------------|
    | FriendlyName                   | El nombre descriptivo del dispositivo, tal y como se configuró mediante la interfaz de usuario web local durante la implementación del dispositivo. El nombre descriptivo predeterminado es el número de serie del dispositivo.  |
    | SerialNumber                   | El número de serie del dispositivo es un número único asignado en la fábrica.                                                                             |
    | Modelo                          | El modelo del dispositivo. El modelo es virtual en el caso de Data Box Gateway.                   |
    | FriendlySoftwareVersion        | La cadena descriptiva que corresponde a la versión de software del dispositivo. Para un sistema que ejecuta la versión preliminar, la versión de software descriptiva sería Data Box Edge 1902. |
    | HcsVersion                     | La versión de software HCS que se ejecuta en el dispositivo. Por ejemplo, la versión de software HCS correspondiente a Data Box Edge 1902 es 1.4.771.324.            |
    | LocalCapacityInMb              | La capacidad total local del dispositivo en megabits.                                                                                                        |
    | IsRegistered                   | Este valor indica si el dispositivo se activa con el servicio.                                                                                         |


