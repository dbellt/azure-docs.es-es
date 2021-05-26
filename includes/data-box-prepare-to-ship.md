---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 05/10/2021
ms.author: alkohli
ms.openlocfilehash: 70dfc7e9f20c4331a8c007d1053eca39bd2aa2af
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736250"
---
El paso final consiste en preparar el dispositivo para el envío. En este paso, se desconectan todos los recursos compartidos de dispositivo. Una vez que empieza este proceso, no se puede acceder a los recursos compartidos.

> [!IMPORTANT]
> La preparación para el envío es necesario, ya que marca datos que no cumplen las convenciones de nomenclatura de Azure. Este paso evita errores de carga en datos por falta de cumplimiento.

1. Vaya a **Prepare to ship** (Preparación para el envío) y haga clic en **Start preparation** (Iniciar preparación). De forma predeterminada, las sumas de comprobación se calculan mientras se copian los datos. Preparar para enviar completa el cálculo de la suma de comprobación y crea la lista de archivos (*Archivos -BOM*). El proceso de la suma de comprobación puede tardar de horas a días, dependiendo del tamaño de los datos. 
   
    ![Preparación para el envío 1](media/data-box-prepare-to-ship/prepare-to-ship-1.png)

    Si por algún motivo desea detener la preparación del dispositivo, haga clic en **Detener preparación**. Puede reanudar la preparación para el envío más adelante.
        
    ![Preparación para el envío 2](media/data-box-prepare-to-ship/prepare-to-ship-2.png)
    
2. La preparación para el envío se inicia y se desconectan los recursos compartidos del dispositivo. <!--You see a reminder to download the shipping label once the device is ready.--> Una vez completada la preparación del dispositivo, su estado se actualiza a *Ready to ship* (Listo para enviar) y el dispositivo se bloquea.
        
    ![Preparación para el envío 3](media/data-box-prepare-to-ship/prepare-to-ship-3.png)

    Si desea copiar más datos en el dispositivo, puede desbloquearlo, copiar más datos y ejecutar la preparación para el envío de nuevo.

    Si hay errores en este paso, el estado se actualiza a *Exploración completada con errores*. Desbloquee el dispositivo y vaya a la página **Conectar y copiar**, descargue la lista de problemas y resuelva los errores.

    ![Preparación para el envío 4](media/data-box-prepare-to-ship/prepare-to-ship-4.png)

    Una vez hecho esto, ejecute **Preparación para el envío**.

4. Una vez que la preparación para el envío haya finalizado correctamente (sin errores), siga estos pasos:

    1. Anote el número de referencia de finalización. En función del país en el que se encuentre, este número puede ser necesario para distintas operaciones.
    2. Descargue la lista de archivos (conocido también como manifiesto) que se han copiado en este proceso. Más adelante, puede usar esta lista para comprobar los archivos se cargaron en Azure. Para más información, consulte [Inspeccionar BOM durante la preparación para el envío](../articles/databox/data-box-logs.md#inspect-bom-during-prepare-to-ship).
        
        ![Preparación para el envío 5](media/data-box-prepare-to-ship/prepare-to-ship-5.png)
    3. Seleccione y descargue las instrucciones de envío para el dispositivo. Las instrucciones de envío varían en función del país en el que se encuentre.
    4. Si E-ink no muestra la etiqueta de envío, puede descargar aquí la etiqueta de envío inverso aquí. 

5. Apague el dispositivo. Vaya a la página **Shut down or restart** (Apagar o reiniciar) y haga clic en **Shut down** (Apagar). Cuando se le pida confirmación, haga clic en **OK** (Aceptar) para continuar.

6. Retire los cables. El siguiente paso es enviar el dispositivo a Microsoft.
