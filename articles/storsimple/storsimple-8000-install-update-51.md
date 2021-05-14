---
title: Instalación de Update 5.1 en un dispositivo de la serie StorSimple 8000 | Microsoft Docs
description: Explica cómo instalar Update 5.1 de la serie StorSimple 8000 en un dispositivo de la serie StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 04/21/2021
ms.author: alkohli
ms.openlocfilehash: 289ffbbd3dc23be0060ee5dbe488bf80323214e2
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897224"
---
# <a name="install-update-51-on-your-storsimple-device"></a>Instalación de Update 5.1 en un dispositivo StorSimple

## <a name="overview"></a>Información general

En este tutorial se explica cómo instalar Update 5.1 en un dispositivo StorSimple que ejecute una versión anterior del software a través de Azure Portal y mediante el método de revisión.

Update 5.1 incluye actualizaciones de seguridad sin interrupciones que se pueden aplicar a través de Azure Portal o mediante el método de revisión.

Si aplica Update 5.1 desde Azure Portal, se llevan a cabo comprobaciones previas para determinar el estado del dispositivo en cuanto a la conectividad de red y el estado del hardware. Estas comprobaciones previas solo se realizan si aplica las actualizaciones desde Azure Portal.

Si usa el método de revisión, se recomienda encarecidamente instalar Update 5 primero siguiendo las instrucciones de [Instalación de Update 5 en el dispositivo StorSimple](storsimple-8000-install-update-5.md). A continuación, siga los pasos de [Instalación de Update 5.1 como una revisión](#install-update-51-as-a-hotfix) a continuación para instalar Update 5.1.

Las actualizaciones de seguridad de Update 5.1 tardan unos treinta minutos en instalarse.

> [!IMPORTANT]
> * Update 5.1 es una actualización obligatoria y debe instalarse inmediatamente. Para obtener más información, consulte [Notas de la versión de Update 5.1](storsimple-update51-release-notes.md).
> * Update 5 es una versión mínimamente compatible.

> [!NOTE]
> * Se recomienda instalar las actualizaciones de software y otras actualizaciones normales a través de Azure portal. <!--You should only go to the Windows PowerShell interface of the device (to install updates) if the pre-update gateway check fails in the portal. Depending upon the version you are updating from, the updates may take 4 hours (or greater) to install. The maintenance mode updates must be installed through the Windows PowerShell interface of the device. As maintenance mode updates are disruptive updates, these result in a down time for your device. - N/A FOR 5.1? No maintenance mode updates, and the security updates go quickly.-->
> * Si tiene previsto realizar la instalación mediante el método de revisión, debe ponerse en contacto con el [Soporte técnico de Microsoft](mailto:support@microsoft.com) antes de comenzar la instalación.

[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-51-through-the-azure-portal"></a>Instalación de Update 5.1 mediante Azure Portal

Siga los pasos a continuación para actualizar el dispositivo a [Update 5.1](storsimple-update51-release-notes.md).

> [!NOTE]
> Microsoft extrae información de diagnóstico adicional del dispositivo. Como consecuencia, cuando nuestro equipo de operaciones identifica dispositivos que están teniendo problemas, estamos mejor equipados para recopilar información del dispositivo y diagnosticar problemas.

#### <a name="to-install-an-update-from-the-azure-portal"></a>Instalar una actualización desde el Portal de Azure

1. En la página de servicio de StorSimple, seleccione el dispositivo.

    ![Selección del dispositivo](./media/storsimple-8000-install-update-51/update1.png)

2. Vaya a **Configuración del dispositivo** > **Actualizaciones del dispositivo**.

    ![Captura de pantalla de la hoja Configuración con la opción Actualizaciones del dispositivo seleccionada.](./media/storsimple-8000-install-update-51/update2.png)

3. Aparecerá una notificación si hay nuevas actualizaciones disponibles. O bien, en la hoja **Actualizaciones del dispositivo**, haga clic en **Buscar actualizaciones**. Se crea un trabajo para buscar las actualizaciones disponibles. Se le notificará cuando el trabajo se complete correctamente.

    ![Captura de pantalla de la hoja Configuración con la opción Actualizaciones del dispositivo seleccionada y la hoja Actualizaciones del dispositivo con las nuevas actualizaciones periódicas están disponibles como mensaje.](./media/storsimple-8000-install-update-51/update3.png)

4. Se recomienda revisar las notas de la versión antes de aplicar una actualización en el dispositivo. Para aplicar las actualizaciones, haga clic en **Instalar actualizaciones**. En la hoja **Confirmar actualizaciones normales**, revise los requisitos previos que debe completar antes de aplicar las actualizaciones. Seleccione la casilla para indicar que está listo para actualizar el dispositivo y, a continuación, haga clic en **Instalar**.

    ![Captura de pantalla de la hoja Actualizaciones del dispositivo con la opción Instalar actualizaciones seleccionada y la opción Confirmar actualizaciones periódicas con la opción de aceptación y la opción de instalación seleccionadas.](./media/storsimple-8000-install-update-51/update4.png)

5. Ahora se inicia un conjunto de comprobaciones previas. Estas comprobaciones incluyen:
   
   * **Comprobaciones del estado del controlador** para comprobar que los controladores del dispositivo están en buen estado y en línea.
   * **Comprobaciones de mantenimiento de componentes de hardware** para comprobar que todos los componentes de hardware del dispositivo StorSimple están en buen estado.
   * **Comprobaciones de DATA 0** para comprobar DATA 0 está habilitado en el dispositivo. Si la interfaz no está habilitada, tendrá que habilitarla y, después, volver a intentarlo.

     La actualización se descargará e instalará solo si todas las comprobaciones se completan correctamente. Recibirá una notificación cuando las comprobaciones están en curso. Si se produce un error en la comprobación previa, se le indicarán las causas del error. Resuelva estos problemas e intente realizar de nuevo la operación. Puede que necesite ponerse en contacto con el soporte técnico de Microsoft si no puede resolver estos problemas por sí mismo.

6. Una vez realizadas correctamente las comprobaciones previas, se crea un trabajo de actualización. Recibirá una notificación cuando el trabajo de actualización esté correctamente creado.
   
    ![Captura de pantalla de la notificación que dice "Iniciando el trabajo de actualizaciones de software".](./media/storsimple-8000-install-update-51/update6.png)
   
    La actualización se aplica en el dispositivo.

7. La actualización tarda unas horas en completarse. Seleccione el trabajo de actualización y haga clic en **Detalles** para ver los detalles del trabajo en cualquier momento.

    ![Captura de pantalla de la hoja Actualizaciones del dispositivo con la opción Descargar e instalar las actualizaciones de software en curso seleccionada y la hoja Instalar actualizaciones.](./media/storsimple-8000-install-update-51/update8.png)

     También puede supervisar el progreso del trabajo de actualización desde **Configuración del dispositivo > Trabajos**. En la hoja **Trabajos**, puede ver el progreso de la actualización.

     ![Captura de pantalla de la hoja Configuración con la opción Trabajos seleccionada y la hoja Trabajos que muestra el progreso de la actualización.](./media/storsimple-8000-install-update-51/update7.png)

8. Una vez completado el trabajo, vaya a **Configuración del dispositivo > Actualizaciones del dispositivo**. La versión del software ya debería estar actualizada.

   Compruebe que el dispositivo ejecuta **StorSimple 8000 Series Update 5.1 (6.3.9600.17885)** y que **Fecha de la última actualización** indica la fecha actual.

## <a name="install-update-51-as-a-hotfix"></a>Instalación de Update 5.1 como una revisión

Si desea instalar Update 5.1 como una revisión, realice estos pasos antes de comenzar la instalación:

* Instale Update 5 antes de instalar Update 5.1. Para obtener instrucciones, consulte [Instalación de Update 5 en el dispositivo StorSimple](storsimple-8000-install-update-5.md).
* Antes de comenzar la instalación de la revisión, póngase en contacto con el [Soporte técnico de Microsoft](mailto:support@microsoft.com).

El método de revisión implica los pasos siguientes:

1. Descargar la revisión desde el catálogo de Microsoft Update.
2. Instalar y comprobar la revisión de modo normal.
3. Instalar y comprobar la revisión del modo de mantenimiento.

#### <a name="download-updates-for-your-device"></a>Descargar las actualizaciones para el dispositivo

Debe descargar e instalar las revisiones siguientes en las carpetas sugeridas y en el orden indicado.

| Pedido | KB       | Descripción | Tipo de actualización | Hora de instalación |Carpeta de instalación|
|-------|----------|------------ |-------------|--------------|----- |
|1.     |KB4542887|Actualización de software<br>Descargar _HcsSoftwareUpdate.exe_ y _CisMSDAgent.exe_ |Normal <br></br>Sin interrupciones |~ 25 min |FirstOrderUpdate|
|3.<sup>1, 2</sup>     |KB4037263|Firmware del disco|Mantenimiento <br></br>Perjudicial|~30 min|ThirdOrderUpdate|

<sup>1</sup> No hay actualizaciones de segundo orden en Update 5.1.

<sup>2</sup> Instale las actualizaciones de tercer orden si no instaló las actualizaciones de firmware de disco sobre las actualizaciones de revisiones de Update 5.

Realice los siguientes pasos para descargar e instalar las revisiones.

#### <a name="download-hotfixes"></a>Descarga de revisiones

Para descargar las revisiones, consulte [Descargar revisiones](storsimple-8000-install-update-5.md#to-download-hotfixes).

#### <a name="install-and-verify-device-updates"></a>Instalación y comprobación de actualizaciones del dispositivo

Instale las actualizaciones del dispositivo de KB4542887 siguiendo los pasos que se describen en [Instalar y comprobar las revisiones de modo normal](storsimple-8000-install-update-5.md#to-install-and-verify-regular-mode-hotfixes) en **Instalación de Update 5 en el dispositivo StorSimple**.

> [!IMPORTANT]
> Si aún no se puso en contacto con el [Soporte técnico de Microsoft](mailto:support@microsoft.com), debe hacerlo ahora, antes de instalar las revisiones.

Siga los pasos para instalar las actualizaciones de primer orden. No hay actualizaciones de segundo orden en Update 5.1.

En el caso de Update 5.1, compruebe estas versiones de software después de la instalación:

 * FriendlySoftwareVersion: StorSimple 8000 Series Update 5.1
 * HcsSoftwareVersion: 6.3.9600.17885
 * CisAgentVersion: 1.0.9777.0
 * MdsAgentVersion: 35.2.2.0
 * Lsisas2Version: 2.0.78.00

#### <a name="install-and-verify-disk-firmware-updates"></a>Instalación y comprobación de actualizaciones de firmware de disco

Si no instaló las actualizaciones de firmware de disco al instalar Update 5, instale las actualizaciones de firmware de disco de KB4037263 siguiendo los pasos que se describen en [Instalar y comprobar las revisiones de modo normal](storsimple-8000-install-update-5.md#to-install-and-verify-maintenance-mode-hotfixes) en **Instalación de Update 5 en el dispositivo StorSimple**.

No es necesario instalar actualizaciones de firmware de disco si ejecuta estas versiones de firmware: `XMGJ`, `XGEG`, `KZ50`, `F6C2`, `VR08`, `N003`, `0107`.

Para comprobar si necesita las actualizaciones de firmware de disco, puede ejecutar el cmdlet `Get-HcsFirmwareVersion`.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la [versión de Update 5.1](storsimple-update51-release-notes.md).
