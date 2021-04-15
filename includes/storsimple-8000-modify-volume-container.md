---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 02/09/2021
ms.author: alkohli
ms.openlocfilehash: 71f18cf8448060385ea38be9b2719b1ed545c5d2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100545425"
---
> [!NOTE] 
> No se puede modificar la configuración de cifrado ni las credenciales de la cuenta de almacenamiento asociadas a un contenedor de volúmenes después de crearlo.

#### <a name="to-modify-a-volume-container"></a>Para modificar un contenedor de volúmenes

1. Vaya al servicio StorSimple Device Manager y, después, a **Administración > Contenedores de volúmenes**.

2. En la lista tabular de contenedores de volúmenes, seleccione el contenedor que desea modificar. En la página **Dispositivos**, seleccione el dispositivo, haga doble clic en él y, a continuación, haga clic en la pestaña **Contenedores de volúmenes**.

3. En la lista tabular de contenedores de volúmenes, seleccione el contenedor que desea modificar. En la hoja que se abre, haga clic en **Modificar** en la barra de comandos.

    ![Modificar contenedor de volúmenes](./media/storsimple-8000-modify-volume-container/modify-volume-container-01.png)

4. En la hoja **Modificar contenedor de volúmenes**, realice los siguientes pasos:
   
   1. No se pueden cambiar el nombre, la clave de cifrado ni la cuenta de almacenamiento asociados con el contenedor de volúmenes una vez especificados. Cambie la configuración de ancho de banda asociada.<!--STEPS NEED WORK. Updated screen doesn't show alternative to Unlimited or subsequent steps if they customize bandwidth. Can we talk them through this (briefly)?-->
      
       ![Cambiar la configuración de ancho de banda](./media/storsimple-8000-modify-volume-container/modify-volume-container-02.png)<!--New graphic based on: modify-volume-container-bw-setting.png-->

   1.  Haga clic en **OK**.<!--If they choose Custom, do they still click OK, or are there more steps?-->

5. En la página siguiente del cuadro de diálogo **Modificar contenedor de volúmenes** :<!--This step happens only if they choose Custom bandwidth? Are the steps similar to those in "Add volume container," step 3f, above?"-->
   
   1. En la lista desplegable, elija una plantilla de ancho de banda existente.
   1. Revise la configuración de programación para la plantilla de ancho de banda especificada.
   1. Haga clic en **Guardar** y confirme los cambios.
      
       ![Confirmar cambios](./media/storsimple-8000-modify-volume-container/modify-volume-container-03.png)

      La hoja **Contenedores de volúmenes** se actualiza para reflejar los cambios.
