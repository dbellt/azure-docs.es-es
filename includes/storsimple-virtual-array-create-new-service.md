---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: alkohli
ms.service: storage
ms.topic: include
ms.date: 09/15/2018
ms.author: alkohli
ms.custom: include file
ms.openlocfilehash: 1be6a654962b513cfcf755d45e562b86067e7b25
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95995031"
---
#### <a name="to-create-a-new-service"></a>Para crear un nuevo servicio

1.  Con las credenciales de su cuenta de Microsoft, inicie sesión en Azure Portal en esta dirección URL: <https://portal.azure.com/>. Si va a implementar el dispositivo en el Portal de Government, inicie sesión en: <https://portal.azure.us/>.

2.  En Azure Portal, haga clic en **+ Crear un recurso**&gt;**Almacenamiento** &gt; **StorSimple Virtual Series** (Serie virtual de StorSimple).

    ![Creación de un servicio](./media/storsimple-virtual-array-create-new-service/createnewservice2.png) 

3.  En la hoja **Administrador de dispositivos de StorSimple** que se abre, haga lo siguiente:

    1.  Proporcione un valor único en **Nombre de recurso** para el servicio. El nombre de recurso es un nombre descriptivo que sirve para identificar el servicio. El nombre puede tener entre 2 y 50 caracteres que pueden ser letras, números y guiones. El nombre debe empezar y terminar con una letra o un número.

    2.  Elija una **Suscripción** en la lista desplegable. La suscripción está vinculada a la cuenta de facturación. Este campo no está presente si tiene una sola suscripción.

    3.  Para **Grupo de recursos**, seleccione un grupo de recursos existente o cree uno. Para más información, consulte [Grupos de recursos en Azure](../articles/azure-resource-manager/management/manage-resource-groups-portal.md).

    4.  Proporcione una **Ubicación** para el servicio. Consulte [Regiones de Azure](https://azure.microsoft.com/regions/#services) para más información acerca de qué servicios están disponibles en cada región. En general, elija una **ubicación** más cercana a la región geográfica donde desee implementar el dispositivo. También puedes querer tener en cuenta lo siguiente:

        -   Si tiene cargas de trabajo existentes en Azure que también va a implementar con el dispositivo de StorSimple, se recomienda usar ese centro de datos.

        -   El Administrador de dispositivos de StorSimple y el almacenamiento de Azure pueden encontrarse en dos ubicaciones independientes. En ese caso, se le pedirá que cree la cuenta del Administrador de dispositivos de StorSimple y la de almacenamiento de Azure por separado. Para crear una cuenta de almacenamiento de Azure, vaya a Azure Storage en Azure Portal y siga los pasos descritos en la sección [Crear una cuenta de almacenamiento](../articles/storage/common/storage-account-create.md). Después de crear esta cuenta, agréguela al servicio Administrador de dispositivos de StorSimple mediante los pasos indicados en la sección [Configurar una nueva cuenta de almacenamiento para el servicio](../articles/storsimple/storsimple-virtual-array-manage-storage-accounts.md#add-a-storage-account-credential).

        -   Si está implementando el dispositivo virtual en el Portal de Government, el servicio Administrador de dispositivos de StorSimple se encuentra disponible en las ubicaciones EE. UU. (Iowa) y EE. UU. (Virginia).

    5.  Seleccione **Crear una nueva cuenta de almacenamiento de Azure** para crear automáticamente una cuenta de almacenamiento con el servicio. Especifique un **nombre de cuenta de almacenamiento**. Si necesitas tener tus datos en una ubicación diferente, desactiva esta casilla.

    6.  Active **Anclar al panel** si desea un vínculo rápido a este servicio en el panel.

    7.  Haga clic en **Crear** para crear el Administrador de dispositivos de StorSimple.

        ![Creación de un nuevo servicio 2](./media/storsimple-virtual-array-create-new-service/createnewservice4.png)  

Se le dirige a la página de destino del **Servicio**. Se tarda unos minutos en crear el servicio. Después de que el servicio se cree correctamente, se le notificará de forma adecuada y el estado del servicio cambiará a **Activo**.