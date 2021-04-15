---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 02/09/2021
ms.author: alkohli
ms.openlocfilehash: 400c5f0b2281679c3ec913e94643ffec6082d9e6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100552667"
---
#### <a name="to-create-a-volume"></a>Para crear un volumen
1. En la lista tabular de dispositivos de la hoja **Dispositivos**, seleccione el dispositivo. Haga clic en **+ Agregar volumen**.<!--Note to reviewer. No screen updates. Had to rename screens to be able to correct a formatting error.-->

    ![Adición de un nuevo volumen](./media/storsimple-8000-create-volume-u2/create-volume-01.png)

2. En la hoja **Agregar un volumen**:
   
   1. El campo **Seleccionar dispositivo** se rellena automáticamente con el dispositivo actual.

   2. En la lista desplegable, seleccione el contenedor de volúmenes en el que necesita agregar un volumen. 

   3. Escriba un **Nombre** para el volumen. No se puede cambiar el nombre de un volumen una vez que se crea.

   4. En la lista desplegable, seleccione el **tipo** del volumen. Para cargas de trabajo que requieren garantías locales, latencias bajas y un rendimiento más alto, seleccione un volumen **Localmente anclado** . Para todos los demás datos, seleccione un volumen **En capas** . Si usa este volumen para datos de archivo, seleccione **Usar este volumen para los datos de archivo a los que accede con menos frecuencia**.
      
       Un volumen en capas se aprovisiona de manera fina y se puede crear rápidamente. Si utiliza el volumen en capas para los datos de archivo, al seleccionar **Usar este volumen para los datos de archivo a los que accede con menos frecuencia** se cambiará el tamaño del fragmento de desduplicación del volumen a 512 KB. Si este campo no está activado, el volumen correspondiente en capas utilizará un tamaño del fragmento de 64 KB. Un mayor tamaño de fragmento de desduplicación permite que el dispositivo acelere la transferencia de datos de archivado de gran tamaño a la nube.
       
       Un volumen localmente anclado se aprovisiona de forma intensa y garantiza que los datos principales del volumen continúen siendo locales en el dispositivo y que no se traspasan a la nube.  Si crea un volumen localmente anclado, el dispositivo comprobará el espacio disponible en los niveles locales para aprovisionar el volumen del tamaño solicitado. La operación que consiste en crear un volumen localmente anclado puede implicar traspasar los datos existentes del dispositivo a la nube, por lo que la creación del volumen puede llevar mucho tiempo. El tiempo total depende del tamaño del volumen aprovisionado, el ancho de banda de red disponible y los datos en el dispositivo.

   5. Especifique la **Capacidad aprovisionada** para el volumen. Tome nota de la capacidad que está disponible en función del tipo de volumen seleccionado. El tamaño del volumen especificado no debe superar el espacio disponible.
      
       Puede aprovisionar volúmenes anclados localmente de hasta 8,5 TB o volúmenes en capas de hasta 200 TB en el dispositivo 8100. En el dispositivo 8600 de mayor tamaño, puede aprovisionar volúmenes anclados localmente de hasta 22,5 TB o volúmenes en capas de hasta 500 TB. Como el espacio local del dispositivo es necesario para hospedar el espacio de trabajo de los volúmenes en capas, la creación de volúmenes localmente anclados afectará el espacio disponible para aprovisionar volúmenes en capas. Por lo tanto, si crea un volumen anclado localmente, se reduce el espacio disponible para la creación de volúmenes en capas. De forma similar, si se crea un volumen en capas, se reducirá el espacio disponible para la creación de volúmenes anclados localmente.
      
       Si se aprovisiona un volumen anclado localmente de 8,5 TB (tamaño máximo permitido) en el dispositivo 8100, habrá agotado todo el espacio local disponible en él. No puede crear ningún volumen en capas desde ese punto en adelante, porque no hay ningún espacio local en el dispositivo para hospedar el espacio de trabajo del volumen en capas. Los volúmenes existentes en capas también afectan al espacio disponible. Por ejemplo, si tiene un dispositivo 8100 que ya cuenta con volúmenes en capas de aproximadamente 106 TB, solo 4 TB de espacio estarán disponibles para volúmenes anclados localmente.
 
      1. En el campo **Hosts conectados**, haga clic en la flecha. 

         ![Hosts conectados](./media/storsimple-8000-create-volume-u2/create-volume-02.png)

      1. En la hoja **Hosts conectados**, seleccione un ACR existente o agregue un nuevo ACR realizando los pasos siguientes:

         1. Proporcione un **Nombre** para su ACR.
         2. En **Nombre del iniciador iSCSI**, proporcione el nombre cualificado iSCSI (IQN) de su host de Windows. Si no tiene el IQN, vaya a [Obtener el IQN de un host de Windows Server](#get-the-iqn-of-a-windows-server-host).

      1. Haga clic en **Crear**. Se creará un volumen con la configuración especificada.

         ![Haga clic en Crear.](./media/storsimple-8000-create-volume-u2/create-volume-03.png)

         > [!NOTE]
         > Tenga en cuenta que el volumen que ha creado aquí no está protegido. Debe crear y asociar las directivas de copia de seguridad con este volumen para realizar copias de seguridad programadas. 

