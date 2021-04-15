---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 560c9c177bfa693580979101e5b9343fcff7fe40
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186511"
---
### <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>Instalación de actualizaciones en modo de mantenimiento a través de Windows PowerShell para StorSimple

Cuando aplica las actualizaciones en modo de mantenimiento en un dispositivo de StorSimple, todas las solicitudes de E/S quedan en pausa. Se detienen servicios como la memoria de acceso aleatorio no volátil (NVRAM) o el servicio de Cluster Server. Al entrar o salir de este modo, se reinician ambos controladores. Al salir de este modo, todos los servicios se reanudan y deben estar correctos. Esto puede tardar unos minutos.

> [!IMPORTANT]
> * Antes de entrar en el modo de mantenimiento, compruebe que ambos controladores de dispositivos aparecen con un estado correcto en Azure Portal. Si el controlador no es correcto, [póngase en contacto con Soporte técnico de Microsoft](../articles/storsimple/storsimple-8000-contact-microsoft-support.md) para conocer los pasos siguientes.
> * Cuando está en el modo de mantenimiento, primero debe actualizar un controlador y luego el otro.

1. Use PuTTY para conectarse a la consola serie. Siga las instrucciones detalladas de [Use PuTTy to connect to the serial console](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console) (Uso de PuTTy para conectarse a la consola serie). En el símbolo del sistema, presione **Entrar**. Seleccione la opción 1, **Iniciar sesión con acceso completo**.

2. Para colocar el controlador en modo de mantenimiento, escriba:
    
    `Enter-HcsMaintenanceMode`

    Ambos controladores se reinician en modo de mantenimiento.

3. Instale las actualizaciones en modo de mantenimiento. Escriba:

    `Start-HcsUpdate`

    Se le pedirá confirmación. Después de confirmar las actualizaciones, se instalan en el controlador al que está accediendo en este momento. Una vez instaladas las actualizaciones, se reinicia el controlador.

4. Supervise el estado de las actualizaciones. Inicie sesión en el controlador del mismo nivel cuando el controlador actual se está actualizando y no puede procesar ningún otro comando. Escriba:

    `Get-HcsUpdateStatus`

    Si `RunInProgress` es `True`, la actualización está en curso. Si `RunInProgress` es `False`, indica que se ha completado la actualización.

5. Una vez que las actualizaciones de firmware del disco se aplican correctamente y el controlador actualizado se reinicie, compruebe la versión de firmware del disco. En el controlador actualizado, escriba:

    `Get-HcsFirmwareVersion`
   
    Las versiones de firmware del disco esperadas son: `XMGJ, XGEG, KZ50, F6C2, VR08, N003, 0107`

6. Salga del modo de mantenimiento. Escriba el comando siguiente para cada controlador de dispositivo:

    `Exit-HcsMaintenanceMode`

    Los controladores se reiniciarán al salir del modo de mantenimiento.

7. Vuelva a Azure Portal. Es posible que el portal no muestre que se instalaron las actualizaciones en modo de mantenimiento durante 24 horas.