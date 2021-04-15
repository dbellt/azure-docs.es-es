---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: cc42c22579346c272ee5a6f41147e6b5b09643ba
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95558992"
---
#### <a name="to-download-hotfixes"></a>Descargar revisiones

Realice los pasos siguientes para descargar la actualización de software desde el catálogo de Microsoft Update.

1. Inicie Internet Explorer y vaya a [http://catalog.update.microsoft.com](https://catalog.update.microsoft.com).
2. Si esta es la primera vez que utiliza el Catálogo de Microsoft Update en este equipo, haga clic en **Instalar** cuando se le solicite que instale el complemento del Catálogo de Microsoft Update.

    ![Instalación del catálogo](./media/storsimple-install-update2-hotfix/HCS_InstallCatalog-include.png)

3. En el cuadro de búsqueda del Catálogo de Microsoft Update, escriba el número de Knowledge Base (KB) de la revisión que desee descargar (por ejemplo, **4011839**) y haga clic en **Buscar**.
   
    Aparece la lista de revisiones; por ejemplo, la **actualización acumulativa 4.0 para la serie StorSimple 8000**.
   
    ![Catálogo de búsqueda](./media/storsimple-install-update2-hotfix/HCS_SearchCatalog1-include.png)

4. Haga clic en **Descargar**. Especifique o **busque** una ubicación local en la que quiera que aparezcan las descargas. Haga clic en los archivos que va a descargar en la ubicación y carpeta especificadas. La carpeta también se puede copiar en un recurso compartido de red que sea accesible desde el dispositivo.
5. Busque las revisiones adicionales que se enumeran en la tabla anterior (**4011841**) y descargue los archivos correspondientes a las carpetas específicas, como se muestra en dicha tabla.

> [!NOTE]
> Las revisiones deben ser accesibles desde ambos controladores para detectar los posibles mensajes de error desde el controlador del mismo nivel.
>
> Las revisiones deben copiarse en tres carpetas separadas. Por ejemplo, la actualización del software o Cis o agente MDS se puede copiar en la carpeta _FirstOrderUpdate_, las restantes actualizaciones que no provocan interrupciones se pueden copiar en la carpeta _SecondOrderUpdate_ y las actualizaciones del modo de mantenimiento en la carpeta _ThirdOrderUpdate_.

#### <a name="to-install-and-verify-regular-mode-hotfixes"></a>Instalar y comprobar las revisiones de modo normal

Realice los pasos siguientes para instalar y comprobar las revisiones de modo normal. Si ya las ha instalado a través del Portal de Azure clásico, puede ir directamente a la sección [Instalar y comprobar las revisiones del modo de mantenimiento](#to-install-and-verify-maintenance-mode-hotfixes).

1. Para instalar las revisiones, acceda a la interfaz de Windows PowerShell en la consola serie del dispositivo de StorSimple. Siga las instrucciones detalladas de [Use PuTTy to connect to the serial console](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console) (Uso de PuTTy para conectarse a la consola serie). En el símbolo del sistema, presione **Entrar**.
2. Seleccione la opción 1, **Iniciar sesión con acceso completo**. Se recomienda instalar primero la revisión en el controlador pasivo.
3. Para instalar la revisión, en el símbolo del sistema, escriba lo siguiente:
   
    `Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`
   
    Use IP en lugar de DNS en la ruta de acceso de recurso compartido en el comando anterior. El parámetro credential se usa únicamente si tiene acceso a un recurso compartido autenticado.
   
    Se recomienda que use el parámetro de credencial para obtener acceso a los recursos compartidos. Incluso los recursos compartidos que están abiertos para "todos" no suelen estar abiertos a los usuarios no autenticados.
   
    Indique la contraseña cuando se le solicite.
   
    A continuación se muestra una salida de ejemplo para instalar las actualizaciones de primer orden. Para la actualización de primer orden, es preciso que apunte al archivo concreto.
   
    ```output
    Controller0>Start-HcsHotfix -Path \\10.100.100.100\share
    \FirstOrderUpdate\HcsSoftwareUpdate.exe -Credential contoso\John
   
    Confirm
   
    This operation starts the hotfix installation and could reboot one or
    both of the controllers. If the device is serving I/Os, these will not
    be disrupted. Are you sure you want to continue?
    [Y] Yes [N] No [?] Help (default is "Y"): Y

    ```
4. Escriba **Y** cuando se le solicite que confirme la instalación de la revisión.
5. Supervise la actualización mediante el cmdlet `Get-HcsUpdateStatus` . La actualización se completará en primer lugar en el controlador pasivo. Una vez actualizado el controlador pasivo, se producirá una conmutación por error, tras lo cual la actualización se aplicará en el otro controlador. La actualización estará completa cuando ambos controladores se hayan actualizado.
   
    La siguiente salida de ejemplo muestra la actualización en curso. `RunInprogress` será `True` cuando la actualización esté en curso.

    ```
    Controller0>Get-HcsUpdateStatus
    RunInprogress       : True
    LastHotfixTimestamp :
    LastUpdateTimestamp : 02/03/2017 2:04:02 AM
    Controller0Events   :
    Controller1Events   :
    ```
   
     La siguiente salida de ejemplo indica que ha finalizado la actualización. `RunInProgress` será `False` cuando se haya completado la actualización.
   
    ```
    Controller0>Get-HcsUpdateStatus
    RunInprogress       : False
    LastHotfixTimestamp : 02/03/2017 9:15:55 AM
    LastUpdateTimestamp : 02/03/2017 9:06:07 AM
    Controller0Events   :
    Controller1Events   :
    ```

    > [!NOTE]
    > En ocasiones, el cmdlet notifica `False` cuando la actualización está todavía en curso. Para garantizar que la revisión está completada, espere unos minutos, vuelva a ejecutar este comando y compruebe que `RunInProgress` es `False`. Si es así, se habrá completado la revisión.

6. Cuando se complete la actualización del software, compruebe las versiones de software del sistema. Escriba:
   
    `Get-HcsSystem`
   
    Verá las versiones siguientes:
   
   * `FriendlySoftwareVersion: StorSimple 8000 Series Update 4.0`
   * `HcsSoftwareVersion: 6.3.9600.17820`
   
     Si el número de versión no cambia después de aplicar la actualización, indica que la revisión no se ha podido aplicar. Si ve esto, póngase en contacto con [Soporte técnico de Microsoft](../articles/storsimple/storsimple-8000-contact-microsoft-support.md) para obtener más ayuda.
     
     > [!IMPORTANT]
     > Antes de aplicar la siguiente actualización, debe reiniciar el controlador activo a través del cmdlet `Restart-HcsController`.
     
7. Repita los pasos 3 a 5 para instalar el agente de Cis/MDS descargado en la carpeta _FirstOrderUpdate_. 
8. Repita los pasos 3 a 5 para instalar las actualizaciones de segundo orden. **En el caso de las actualizaciones de segundo orden, para instalar varias actualizaciones solo hay que ejecutar `Start-HcsHotfix cmdlet` y señalar a la carpeta en que se encuentran dichas actualizaciones. El cmdlet ejecutará todas las actualizaciones disponibles en la carpeta.** Si una actualización ya está instalada, la lógica de la actualización la detectará y no aplicará esa actualización. 

Después de instalar las revisiones, use el cmdlet `Get-HcsSystem`. Las versiones deben ser:

   * `CisAgentVersion:  1.0.9441.0`
   * `MdsAgentVersion: 35.2.2.0`
   * `Lsisas2Version: 2.0.78.00`


#### <a name="to-install-and-verify-maintenance-mode-hotfixes"></a>Instalar y comprobar las revisiones del modo de mantenimiento
Use KB4011837 para instalar las actualizaciones de firmware del disco. Se trata de actualizaciones potencialmente perjudiciales y tardan unos 30 minutos en completarse. Puede elegir instalarlas en una ventana de mantenimiento planificado mediante la conexión a la consola serie del dispositivo.

Tenga en cuenta que, si el firmware del disco ya está actualizado, no será necesario instalar estas actualizaciones. Ejecute el cmdlet `Get-HcsUpdateAvailability` desde la consola serie del dispositivo para ver si hay actualizaciones disponibles y si provocan interrupciones (modo de mantenimiento) o no (modo normal).

Para instalar las actualizaciones de firmware de disco, siga las instrucciones a continuación.

1. Active el modo de mantenimiento del dispositivo. **Tenga en cuenta que no debe usar la comunicación remota de Windows PowerShell al conectarse a un dispositivo en modo de mantenimiento. En mejor que ejecute este cmdlet en el controlador del dispositivo cuando esté conectado a través de la consola serie del dispositivo.** Escriba:
   
    `Enter-HcsMaintenanceMode`
   
    A continuación se muestra una salida de ejemplo.

    ```output
    Controller0>Enter-HcsMaintenanceMode
    Checking device state...
   
    In maintenance mode, your device will not service IOs and will be disconnected from the Microsoft Azure StorSimple Manager service. Entering maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to enter maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y
   
    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8600
    Name: Update4-8600-mystorsimple
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Passive
    ---------------------------------------------------------------
   
    Serial Console Menu
    [1] Log in with full access
    [2] Log into peer controller with full access
    [3] Connect with limited access
    [4] Change language
    Please enter your choice>
    ```
   
    A continuación, se reiniciarán ambos controladores en modo de mantenimiento.
2. Para instalar la actualización de firmware del disco, escriba:
   
    `Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`
   
    A continuación se muestra una salida de ejemplo.

    ```output
    Controller1>Start-HcsHotfix -Path \\10.100.100.100\share\ThirdOrderUpdates\ -Credential contoso\john
    Enter Password:
    WARNING: In maintenance mode, hotfixes should be installed on each controller sequentially. After the hotfix is installed on this controller, install it on the peer controller.
    Confirm
    This operation starts a hotfix installation and could reboot one or both of the controllers. By installing new updates you agree to, and accept any additional terms associated with, the new functionality listed in the release notes (https://go.microsoft.com/fwLink/?LinkID=613790). Are you sure you want to continue?
    [Y] Yes [N] No (Default is "Y"): Y
    WARNING: Installation is currently in progress. This operation can take several minutes to complete.
    ```

3. Supervise el progreso de la instalación con el comando `Get-HcsUpdateStatus` . La actualización se habrá completado cuando `RunInProgress` cambie a `False`.
4. Una vez completada la instalación, se reiniciará el controlador en el que se haya instalado la revisión de modo de mantenimiento. Inicie sesión con la opción 1, **Iniciar sesión con acceso completo**, y compruebe la versión de firmware del disco. Escriba:
   
   `Get-HcsFirmwareVersion`
   
   Las versiones de firmware del disco esperadas son:
   
   `XMGJ, XGEG, KZ50, F6C2, VR08, N002, 0106`
   
   A continuación se muestra una salida de ejemplo.

    ```output
    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8600
    Name: Update4-8600-mystorsimple
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller1
    ---------------------------------------------------------------
   
    Controller1>Get-HcsFirmwareVersion
   
    Controller0 : TalladegaFirmware
        ActiveBIOS:0.45.0010
           BackupBIOS:0.45.0006
           MainCPLD:17.0.000b
           ActiveBMCRoot:2.0.001F
           BackupBMCRoot:2.0.001F
           BMCBoot:2.0.0002
           LsiFirmware:20.00.04.00
           LsiBios:07.37.00.00
           Battery1Firmware:06.2C
           Battery2Firmware:06.2C
           DomFirmware:X231600
           CanisterFirmware:3.5.0.56
           CanisterBootloader:5.03
           CanisterConfigCRC:0x9134777A
           CanisterVPDStructure:0x06
           CanisterGEMCPLD:0x19
           CanisterVPDCRC:0x142F7DC2
           MidplaneVPDStructure:0x0C
           MidplaneVPDCRC:0xA6BD4F64
           MidplaneCPLD:0x10
           PCM1Firmware:1.00|1.05
           PCM1VPDStructure:0x05
           PCM1VPDCRC:0x41BEF99C
           PCM2Firmware:1.00|1.05
           PCM2VPDStructure:0x05
           PCM2VPDCRC:0x41BEF99C

        EbodFirmware
           CanisterFirmware:3.5.0.56
           CanisterBootloader:5.03
           CanisterConfigCRC:0xB23150F8
           CanisterVPDStructure:0x06
           CanisterGEMCPLD:0x14
           CanisterVPDCRC:0xBAA55828
           MidplaneVPDStructure:0x0C
           MidplaneVPDCRC:0xA6BD4F64
           MidplaneCPLD:0x10
           PCM1Firmware:3.11
           PCM1VPDStructure:0x03
           PCM1VPDCRC:0x6B58AD13
           PCM2Firmware:3.11
           PCM2VPDStructure:0x03
           PCM2VPDCRC:0x6B58AD13

        DisksFirmware
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
    ```
   
    Ejecute el comando `Get-HcsFirmwareVersion` en el segundo controlador para comprobar que se actualizó la versión de software. A continuación, puede salir del modo de mantenimiento. Para ello, escriba el comando siguiente para cada controlador de dispositivo:
   
   `Exit-HcsMaintenanceMode`

5. Los controladores se reiniciarán al salir del modo de mantenimiento. Cuando las actualizaciones de firmware de disco se apliquen correctamente y el dispositivo haya salido del modo de mantenimiento, regrese al Portal de Azure clásico. Tenga en cuenta que, durante 24 horas, es posible que no aparezca en el portal la instalación de las actualizaciones en modo de mantenimiento.