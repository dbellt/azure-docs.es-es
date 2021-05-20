---
title: Desactivación y eliminación de Microsoft Azure StorSimple Virtual Array | Microsoft Docs
description: Describe cómo desactivar y eliminar en primer lugar el dispositivo de StorSimple para quitarlo del servicio.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: a929f5bc-03e2-4b01-b925-973db236f19f
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/27/2021
ms.author: alkohli
ms.openlocfilehash: 9480bc7ddb03ba537ea7f13057492aa05ed66bae
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108770234"
---
# <a name="deactivate-and-delete-a-storsimple-virtual-array"></a>Desactivación y eliminación de una matriz virtual de StorSimple

## <a name="overview"></a>Información general

Cuando se desactiva StorSimple Virtual Array, se interrumpe la conexión entre el dispositivo y el servicio StorSimple Device Manager correspondiente. Este tutorial explica cómo realizar lo siguiente:

* Desactivación de un dispositivo
* Eliminación de un dispositivo desactivado

La información de este artículo solo se aplica a las instancias de StorSimple Virtual Array. Para obtener información sobre la serie 8000, consulte [Desactivación y eliminación de un dispositivo de StorSimple](./storsimple-8000-deactivate-and-delete-device.md).

## <a name="when-to-deactivate"></a>¿Cuándo hay que realizar la desactivación?

La desactivación es una operación permanente y no se puede deshacer. Los dispositivos desactivados no se pueden volver a registrar en el servicio StorSimple Device Manager. Es posible que tenga que desactivar y eliminar una instancia de StorSimple Virtual Array en los escenarios siguientes:

* **Conmutación por error planeada**: el dispositivo está en línea y planea realizar la conmutación por error en su dispositivo. Si piensa actualizar a un dispositivo de mayor tamaño, es posible que necesite realizar la conmutación por error en su dispositivo. Después de que se transfiere la propiedad de los datos y la conmutación por error está completa, se elimina automáticamente el dispositivo de origen.
* **Conmutación por error no planeada**: el dispositivo está en línea y es preciso realizar la conmutación por error en el dispositivo. Esta situación puede producirse durante un desastre, cuando hay una interrupción del centro de datos y el dispositivo principal está inactivo. Planea realizar la conmutación por error del dispositivo en un dispositivo secundario. Después de que se transfiere la propiedad de los datos y la conmutación por error está completa, se elimina automáticamente el dispositivo de origen.
* **Dar de baja**: desea dar de baja el dispositivo. Esto requiere que primero desactive el dispositivo y luego lo elimine. Al desactivar un dispositivo, no podrá acceder a los datos almacenados localmente. Solo podrá acceder y recuperar los datos almacenados en la nube. Si piensa mantener los datos del dispositivo tras la desactivación, debe realizar una instantánea en la nube de todos los datos antes de desactivar un dispositivo. Dicha instantánea le permitirá recuperar todos los datos más adelante.

## <a name="deactivate-a-device"></a>Desactivación de un dispositivo

Para desactivar el dispositivo, realice los pasos siguientes. Un dispositivo debe estar en línea para desactivarse.

#### <a name="to-deactivate-the-device"></a>Para desactivar el dispositivo

1. En el servicio, vaya a **Administración > Dispositivos**. En la hoja **Dispositivos**, haga clic en el dispositivo que desea desactivar para desactivarlo. El estado del dispositivo debe ser **Conectado**.
   
    ![Selección del dispositivo que desactivar](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete7.png)
2. En la hoja **Device dashboard** (Panel del dispositivo), haga clic en **... More** (... más) y en la lista, seleccione **Deactivate** (Desactivar).
   
    ![Clic en Desactivar](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete8.png)
3. En la hoja **Deactivate** (Desactivar), escriba el nombre del dispositivo y haga clic en **Deactivate** (Desactivar). 
   
    ![Confirmación de la desactivación](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete1.png)
   
    El proceso de desactivación se inicia y tarda pocos minutos en completarse.
   
    ![Desactivación en curso](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete2.png)

4. Después de la desactivación la lista de dispositivos se actualiza.
   
    ![Desactivación completa](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete3.png)
   
    Ahora puede eliminar este dispositivo.

## <a name="delete-the-device"></a>Eliminación del dispositivo

Para poder eliminar un dispositivo, es preciso desactivarlo antes. Al eliminar un dispositivo, se quita de la lista de dispositivos conectados al servicio. El servicio ya no podrá administrar el dispositivo eliminado. Sin embargo, los datos asociados al dispositivo permanecerán en la nube. A partir de ahí, estos datos acumulan cargos.

Para eliminar el dispositivo, realice los siguientes pasos.

#### <a name="to-delete-the-device"></a>Para eliminar el dispositivo

1. En el administrador de dispositivos de StorSimple, vaya a **Administración > Dispositivos**. En la hoja **Dispositivos**, seleccione el dispositivo desactivado que desea eliminar.
2. En la hoja **Device dashboard** (Panel del dispositivo), haga clic en **... More** (... más) y, después, en **Delete** (Eliminar).
   
   ![Selección del dispositivo que eliminar](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete4.png)
3. En la hoja **Delete** (Eliminar), escriba el nombre del dispositivo para confirmar la eliminación y, luego, haga clic en **Delete** (Eliminar). Al eliminar el dispositivo no se eliminan los datos en la nube asociados a él. 
   
   ![Confirmar eliminación](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete5.png) 
4. La eliminación se inicia y tarda pocos minutos en completarse.
   
   ![Eliminación en curso](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete6.png)
   
    Después de eliminar el dispositivo puede ver la lista actualizada de los dispositivos.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de cómo realizar la conmutación por error, vaya a [Disaster recovery and device failover for your StorSimple Virtual Array](storsimple-virtual-array-failover-dr.md) (Recuperación ante desastres y conmutación por error de dispositivos en StorSimple Virtual Array).

* Para obtener información acerca de cómo usar el servicio StorSimple Device Manager, vaya a [Use the StorSimple Device Manager service to administer your StorSimple Virtual Array](storsimple-virtual-array-manager-service-administration.md) (Uso del servicio StorSimple Device Manager para administrar StorSimple Virtual Array).