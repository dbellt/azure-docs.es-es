---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 473bc0a58fe49c7f454c81402b57ddce7fc745b2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186376"
---
#### <a name="to-configure-remote-management-on-cloud-appliance"></a>Para configurar la administración remota en el dispositivo de nube, siga estos pasos:

1. En el servicio StorSimple Device Manager, haga clic en **Dispositivos**. Seleccione el dispositivo de nube y haga clic en él en la lista de dispositivos conectados al servicio.
    ![Selección de dispositivo de nube de StorSimple](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage1.png)

2. Vaya a **Configuración > Seguridad** para abrir la hoja **Configuración de seguridad**.

     ![Configuración de seguridad de StorSimple](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage2.png)

3. Vaya a la sección **Administración remota**. Haga clic en el cuadro **Administración remota**.
     ![Administración remota de StorSimple](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage3.png)

4. En la hoja **Administración remota**:

    1. Asegúrese de que la opción **Enable remote administration** (Habilitar la administración remota) esté habilitada.
    2. El valor predeterminado es conectarse a través de HTTPS. Puede elegir conectarse mediante HTTP. La conexión a través de HTTP solo es aceptable en redes de confianza. Asegúrese de que HTTP esté habilitado.
    3. En la barra de comandos de la parte superior de la hoja, haga clic en **...Más** y, a continuación, haga clic en **Descargar certificado** para descargar un certificado de administración remota. Puede especificar la ubicación en la que quiere guardar este archivo. Este certificado debe estar instalado en la máquina cliente o host que usa para conectarse al dispositivo de nube.

        ![Hoja Administración remota](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage4.png)
5. Haga clic en **Guardar** y, cuando se le pida, confirme los cambios.