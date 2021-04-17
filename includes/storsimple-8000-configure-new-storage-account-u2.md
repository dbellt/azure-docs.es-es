---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 9f1c30b303bd1fe02e0685c7d848be92073ca2f6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96016883"
---
#### <a name="to-add-a-storage-account-credential-in-the-same-azure-subscription-as-the-storsimple-device-manager-service"></a>Para agregar una credencial de cuenta de almacenamiento en la misma suscripción que el servicio StorSimple Device Manager, siga estos pasos:

1. Vaya al servicio StorSimple Device Manager. En la sección **Configuración**, haga clic en **Credenciales de cuenta de almacenamiento**.

    ![Ir a Credenciales de la cuenta de almacenamiento](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct1.png)

2. En la hoja **Credenciales de cuenta de almacenamiento**, haga clic en **+ Agregar**.

    ![Incorporación de una credencial de cuenta de almacenamiento](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct2.png)

3. En la hoja **Agregar credencial de cuenta de almacenamiento**, realice los siguientes pasos:

    1. Como va a agregar una credencial de cuenta de almacenamiento en la misma suscripción de Azure que su servicio, asegúrese de que esté seleccionada la opción **Actual**.

    2. En la lista desplegable **Cuenta de almacenamiento**, seleccione una cuenta de almacenamiento existente.

    3. Según la cuenta de almacenamiento seleccionada, se mostrará la **ubicación** (aparece atenuada y no se puede cambiar aquí).

    4. Seleccione **Habilitar modo SSL** para crear un canal seguro para la comunicación de red entre su dispositivo y la nube. Seleccione **Habilitar SSL** solo si trabaja en una nube privada.

        ![Hoja Agregar credenciales de cuenta de almacenamiento](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct3.png)

    5. Haga clic en **Agregar** para iniciar la creación del trabajo de la credencial de cuenta de almacenamiento. Cuando la credencial de cuenta de almacenamiento se haya creado correctamente, recibirá una notificación.

        ![Notificación de que las credenciales de la cuenta de almacenamiento se han creado correctamente](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct5.png)

La credencial de cuenta de almacenamiento recién creada se mostrará en la lista de **credenciales de cuenta de almacenamiento**.

![Lista de las credenciales de la cuenta de almacenamiento](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct6.png)

