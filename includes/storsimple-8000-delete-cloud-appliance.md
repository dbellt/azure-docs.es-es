---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: ac708eb2ac79a74b8f4e09a7306a42665b3aca94
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186372"
---
#### <a name="to-delete-a-cloud-appliance"></a>Eliminación de una aplicación en la nube

1. Inicie sesión en Azure Portal.
2. Solo se puede eliminar un dispositivo desactivado que no contiene datos. Elimine primero los datos en el dispositivo o puede [conmutar por error los datos](../articles/storsimple/storsimple-8000-device-failover-cloud-appliance.md) en contenedores de volúmenes a otro dispositivo. Una vez que se eliminan los datos, está preparado para desactivar el dispositivo.
3. En la página del servicio Administrador de dispositivos de StorSimple, haga clic en **Devices** (Dispositivos) y, luego, seleccione el dispositivo. Haga clic con el botón derecho y seleccione **Desactivar**.
4. Una vez que el dispositivo está desactivado, haga clic en el dispositivo y seleccione **Eliminar**.

    ![Seleccione el dispositivo desactivado y haga clic en eliminar](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance1.png)

5. Escriba el nombre del dispositivo para confirmar la eliminación del dispositivo. Después de que el dispositivo se elimina, se actualiza la lista de dispositivos.

    ![Confirmar eliminación](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance2.png)

6. Recibirá una notificación cuando el dispositivo se haya eliminado.

    ![Notificación de eliminación de dispositivo correcta](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance4.png)

7. La lista de dispositivos se actualiza para indicar que el dispositivo se eliminó.

    ![Lista de dispositivos actualizada](./media/storsimple-8000-delete-cloud-appliance/delete-cloud-appliance5.png)
