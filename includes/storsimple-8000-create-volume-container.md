---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 02/09/2021
ms.author: alkohli
ms.openlocfilehash: fd9b3b501d6efbe6a74d350a678494e8254dbb32
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100545407"
---
#### <a name="to-create-a-volume-container"></a>Para crear un contenedor de volúmenes

1. Vaya al servicio Administrador de dispositivos de StorSimple y haga clic en **Dispositivos**. En la lista tabular de dispositivos, seleccione un dispositivo y haga clic en él. 

    ![Hoja Contenedor de volúmenes](./media/storsimple-8000-create-volume-container/create-volume-container-01.png)

2. En el panel del dispositivo, haga clic en **+ Agregar contenedor de volúmenes**

    ![Hoja Contenedor de volúmenes 2](./media/storsimple-8000-create-volume-container/create-volume-container-02.png)

3. En la hoja **Agregar contenedor de volúmenes**:
   
   1. El dispositivo se selecciona automáticamente.
   2. Proporcione un **Nombre** para el contenedor de volúmenes. El nombre debe tener una longitud de entre 3 y 32 caracteres. No se puede cambiar el nombre de un contenedor de volúmenes una vez que se crea.
   3. Seleccione **Habilitar el cifrado de almacenamiento en la nube** para habilitar el cifrado de los datos enviados del dispositivo a la nube.
   4. Proporcione y confirme una **Clave de cifrado de almacenamiento en la nube** con una longitud de entre 8 y 32 caracteres. El dispositivo usa esta clave para acceder a los datos cifrados.
   5. Seleccione una **Cuenta de almacenamiento** para asociarla a este contenedor de volúmenes. Puede elegir una cuenta de almacenamiento ya existente o la cuenta predeterminada que se genera en el momento en que se crea el servicio. También puede usar la opción **Agregar nuevo** para especificar una cuenta de almacenamiento que no esté vinculada a la suscripción al servicio.
   6. Seleccione **Ilimitado** en la lista desplegable **Especificar el ancho de banda** si desea consumir todo el ancho de banda disponible. También puede establecer esta opción en **Personalizado** para usar los controles de ancho de banda y especificar un valor de entre 1 y 1000 Mbps.
   
      Si tiene a su disposición información sobre el uso del ancho de banda, es posible que pueda asignar ancho de banda en función de una programación especificando **Seleccionar una plantilla de ancho de banda**. Para conocer el procedimiento paso a paso, vaya a [Agregar una plantilla de ancho de banda](../articles/storsimple/storsimple-8000-manage-bandwidth-templates.md#add-a-bandwidth-template).

      ![Hoja Contenedor de volúmenes 3](./media/storsimple-8000-create-volume-container/create-volume-container-06-b.png)<!--New graphic. Source: add-volume-container-bw-setting.-->

   7. Haga clic en **Crear**.

        <!--![Volume container blade 4](./media/storsimple-8000-create-volume-container/create-volume-container-06.png)-->
   
       Recibirá una notificación cuando el contenedor de volúmenes se haya creado correctamente.

       ![Notificación de creación del contenedor de volúmenes](./media/storsimple-8000-create-volume-container/create-volume-container-08.png)

   El contenedor de volúmenes recién creado aparece en la lista de contenedores de volúmenes del dispositivo.

   ![Hoja Agregar contenedor de volúmenes](./media/storsimple-8000-create-volume-container/create-volume-container-09.png)
