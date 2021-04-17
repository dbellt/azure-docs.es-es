---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 867cdc97ff91d5932230b733dee4d7660d499c39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026471"
---
#### <a name="to-complete-the-minimum-storsimple-device-setup"></a>Para completar la instalación mínima del dispositivo StorSimple

   > [!NOTE]
   > No se puede cambiar el nombre del dispositivo cuando se haya completado la configuración mínima del dispositivo.
   
1. En la lista tabular de dispositivos de la hoja **Dispositivos**, seleccione su dispositivo y haga clic en él. El dispositivo está en estado **Listo para configurar**. Se abre la hoja **Configurar dispositivo**.

     ![Interfaces de red de la instalación mínima del dispositivo StorSimple](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig1.png)

2. En la hoja **Configurar dispositivo**:
   
   1. Proporcione un **nombre descriptivo** para el dispositivo. El nombre de dispositivo predeterminado refleja información como el modelo de dispositivo y el número de serie. Puede asignar un nombre descriptivo de hasta 64 caracteres para administrar el dispositivo.
   2. Establezca la **zona horaria** según la ubicación geográfica en la que se va a implementar el dispositivo. El dispositivo usa esta zona horaria para todas las operaciones programadas.
   3. En la **configuración de DATOS 0**:

       1. La interfaz de red de DATOS 0 se muestra como habilitada con la configuración de red (IP, subred, puerta de enlace) definida mediante el Asistente para instalación. DATOS 0 también se habilita automáticamente para la nube y para iSCSI.

       2. Proporcione las direcciones IP fijas de Controlador 0 y Controlador 1. **Las direcciones IP fijas del controlador deben ser direcciones IP libres dentro de la subred y accesibles mediante la dirección IP del dispositivo.** Si la interfaz DATA 0 estaba configurada para IPv4, las direcciones IP fijas deben suministrarse en formato IPv4. Si proporcionó un prefijo para la configuración de IPv6, las direcciones IP fijas se rellenan automáticamente en estos campos.

            ![Interfaces de red de la instalación mínima del dispositivo StorSimple 2](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig2.png)

            Las direcciones IP fijas del controlador se usan para el mantenimiento de las actualizaciones del dispositivo y para la recolección de elementos no utilizados. Por lo tanto, las direcciones IP fijas deben ser enrutables y disponer de capacidad de conexión a Internet. Para comprobar que sus IP de controlador fijas son enrutables, use el cmdlet [Test-HcsmConnection][Test]. En el ejemplo siguiente se muestra que las direcciones IP fijas del controlador se enrutan a Internet y pueden tener acceso a los servidores de Microsoft Update.

            ![Test-HcsmConnection que muestra direcciones IP enrutables](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig3.png)

1. Haga clic en **OK**. Se inicia la configuración del dispositivo. Cuando finalice, recibirá una notificación. El estado del dispositivo cambia a **En línea** en la hoja **Dispositivos**.

    ![Interfaces de red de la instalación mínima del dispositivo StorSimple 3](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig4.png)

<!--Link reference-->
[Test]: /previous-versions/windows/powershell-scripting/dn715782(v=wps.630)