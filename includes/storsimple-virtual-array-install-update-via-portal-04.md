---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 48ce594fa5f4b736e69b5b4ef7a10011fe8031fa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96027543"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>Para instalar actualizaciones mediante Azure Portal

1. Vaya a su Administrador de dispositivos de StorSimple y seleccione **Dispositivos**. En la lista de dispositivos conectados al servicio, seleccione y haga clic en el dispositivo que desea actualizar. 

    ![En Recientes, el Administrador de dispositivos MySSDevManager está resaltado y seleccionado, Dispositivos está resaltado y seleccionado y el dispositivo MYSSIS1103 está resaltado y seleccionado.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate1m.png) 

2. En la hoja **Configuración**, haga clic en **Actualizaciones del dispositivo**. 

    ![Se muestra la información del dispositivo MYSSIS1103. En Configuración, está resaltada la opción las Actualizaciones del dispositivo.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate2m.png)  

3. Verá un mensaje si hay actualizaciones de software disponibles. Para buscar actualizaciones, también puede hacer clic en **Examinar**.

    ![En el panel Actualizaciones del dispositivo se indica "Nuevas actualizaciones disponibles" y "Último análisis/18/01/2017 2:42 PM/Versión del software/10.0.10288.0". La versión aparece resaltada.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate3m1.png)

    Se le notificará cuando se inicie el examen y se complete correctamente.

    ![La notificación indica "Examinando actualizaciones para el dispositivo MySSIS1103…. 2:12 PM/La operación se completó correctamente."](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate5m.png)

4. Una vez examinadas las actualizaciones, haga clic en **Descargar actualizaciones**. 

    ![En el panel Actualizaciones del dispositivo aparece el mensaje "New updates are available"(Hay nuevas actualizaciones). Aparece resaltado el botón Descargar actualizaciones.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate6m.png)

5. En la hoja **Nuevas actualizaciones**, revise la información y, una vez descargadas las actualizaciones, debe confirmar la instalación. Haga clic en **OK**.

    ![En el panel Actualizaciones nuevas se indica "After the updates are downloaded, you will need to confirm the installation" (Una vez descargadas las actualizaciones, deberá confirmar la instalación). El botón Aceptar está resaltado.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate7m.png)

6. Se le notificará cuando se inicie la carga y se complete correctamente.

     ![La notificación indica "Descargando actualizaciones para el dispositivo MySSIS1… 2:13 PM".](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate8m.png)

5. En la hoja **Actualizaciones del dispositivo**, haga clic en **Instalar**.

     ![En Actualizaciones del dispositivo aparece el mensaje "Confirm installing updates" (Confirme la instalación de las actualizaciones). Hay un botón Instalar.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate11m1.png)   

6. En la hoja **Nuevas actualizaciones**, se le advertirá de que la actualización puede ser perjudicial. Puesto que la matriz virtual es un dispositivo de un solo nodo, el dispositivo se reiniciará una vez actualizado. Se interrumpirá cualquier E/S en curso. Haga clic en **Aceptar** para instalar las actualizaciones. 

    ![El mensaje del panel Nuevas actualizaciones es "Your device will have a downtime when these updates are installed" (Durante la instalación de estas actualizaciones, el dispositivo estará sin actividad). El botón Aceptar está resaltado.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate12m.png) 

7. Se le notificará cuando se inicie el trabajo de instalación. 

    ![La notificación indica "Instalando actualizaciones para el dispositivo MySSIS1103. 2:15 PM".](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate13m.png)

8.  Cuando el trabajo de instalación finalice correctamente, haga clic en el vínculo **Ver trabajo**, en la hoja **Actualizaciones del dispositivo**, para supervisar la instalación. 

    ![En el panel Actualizaciones de dispositivos, hay un vínculo con la etiqueta "Installing updates. View Job" (Instalando actualizaciones. Ver trabajo).](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate15m1.png)

    Esto le lleva a la hoja **Instalar actualizaciones**. Puede ver información detallada sobre el trabajo aquí.

    ![El panel Instalar actualizaciones tiene información sobre la instalación, incluidos el dispositivo, el estado, la duración, la hora de inicio y la hora de detención.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate16m1.png)

9. Una vez instaladas correctamente las actualizaciones, verá un mensaje que lo indica en la hoja **Actualizaciones del dispositivo**. 
