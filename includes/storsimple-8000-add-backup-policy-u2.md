---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 02/09/2021
ms.author: alkohli
ms.openlocfilehash: 23ce17844a0113f63931c6ece7d36bfefedc2de5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100552131"
---
#### <a name="to-add-a-storsimple-backup-policy"></a>Para agregar una directiva de copia de seguridad de StorSimple

1. Vaya al dispositivo StorSimple y haga clic en **Directiva de copia de seguridad**.

2. En la hoja **Directiva de copia de seguridad**, haga clic en **+ Agregar directiva** en la barra de comandos.
   
    ![Agregar una directiva de copia de seguridad](./media/storsimple-8000-add-backup-policy-u2/add-backup-policy-01.png)

3. En la hoja **Crear directiva de copia de seguridad**, realice los siguientes pasos:
   
   1. El campo **Seleccionar dispositivo** se rellena automáticamente según el dispositivo seleccionado.
   
   2. Especifique un **Nombre de directiva** de copia de seguridad que tenga entre 3 y 150 caracteres. Una vez creada la directiva, no se puede cambiar el nombre de la misma.
       
   3. Para asignar volúmenes a esta directiva de copia de seguridad, seleccione **Agregar volúmenes** y, en la lista tabular de volúmenes, haga clic en las casillas para asignar uno o más volúmenes.

       ![Incorporación de una directiva de copia de seguridad 2](./media/storsimple-8000-add-backup-policy-u2/add-backup-policy-02.png)<!--Replacement screen source: create-backup-policy-addvolumes.png-->

   4. Para definir una programación para esta directiva de copia de seguridad, haga clic en la **primera programación** y luego modifique los parámetros siguientes:<!--Do the substeps remain the same? Can they follow without a screenshot?-->

       ![Incorporación de una directiva de copia de seguridad 3](./media/storsimple-8000-add-backup-policy-u2/add-backup-policy-03.png)<!--Replacement screen source: create-backup-policy-first-schedule.png-->

       1. En **Tipo de instantánea**, seleccione **Nube** o **Local**.

       2. Indique la frecuencia de las copias de seguridad (especifique un número y elija **Días** o **Semanas** en la lista desplegable).

       3. Especifique una programación de retención.

       4. Escriba una fecha y hora para que comience la directiva de copia de seguridad.

       5. Haga clic en **Aceptar** para definir la programación.

   5. Haga clic en **Crear** para crear una directiva de copia de seguridad.
   
   6. Cuando la directiva de copia de seguridad se haya creado, recibirá una notificación. La directiva recién agregada se muestra en la vista tabular en la hoja **Directiva de copia de seguridad**.

       ![Incorporación de una directiva de copia de seguridad 5](./media/storsimple-8000-add-backup-policy-u2/add-backup-policy-07.png)
