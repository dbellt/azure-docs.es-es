---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 8cc5dbb907c342b766cebe6da36cf580ddac5e2c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186347"
---
#### <a name="to-install-regular-hotfixes-via-windows-powershell-for-storsimple"></a>Para instalar revisiones regulares a través de Windows PowerShell para StorSimple
1. Conéctese a la consola serie del dispositivo. Para más información, consulte [Paso 1: Conexión a la consola serie](../articles/storsimple/storsimple-update-device.md#step1).
2. En el menú de la consola serie, seleccione la opción 1, **iniciar sesión con acceso completo**. Escriba la contraseña. La contraseña predeterminada es **Password1**.
3. En el símbolo del sistema, escriba:
   
    ```
    Start-HcsHotfix
    ```
   
    > [!IMPORTANT]
    >
    > Este comando solo se aplica a las revisiones regulares. Este comando se ejecuta en un solo controlador, pero se actualizarán ambos controladores.
    > Es posible que observe una conmutación por error del controlador durante el proceso de actualización. Sin embargo, la conmutación por error no afectará a la disponibilidad o el funcionamiento del sistema.

4. Cuando se le solicite, proporcione la ruta de acceso a la carpeta compartida de red que contiene los archivos de la revisión.
5. Se le pedirá confirmación. Escriba **Y** para continuar con la instalación de la revisión.

