---
title: Control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure mediante Azure Portal
description: Aprenda a controlar cuándo se implementan las actualizaciones automáticas de las imágenes del sistema operativo en los conjuntos de escalado de máquinas virtuales de Azure mediante el control de mantenimiento y Azure Portal.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/01/2021
ms.author: jushiman
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6537728feb2145520ee49457b00d9944d5967c9f
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073194"
---
# <a name="maintenance-control-for-os-image-upgrades-on-azure-virtual-machine-scale-sets-using-azure-portal"></a>Control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure mediante Azure Portal

El control de mantenimiento le permite decidir cuándo aplicar las actualizaciones automáticas de imágenes del sistema operativo invitado a los conjuntos de escalado de máquinas virtuales. En este tema se tratan las opciones de Azure Portal para el control de mantenimiento. Para más información sobre el uso del control de mantenimiento, consulte [Control de mantenimiento para conjuntos de escalado de máquinas virtuales de Azure](virtual-machine-scale-sets-maintenance-control.md).


## <a name="create-a-maintenance-configuration"></a>Creación de una configuración de mantenimiento

1. Inicie sesión en Azure Portal.

1. Busque **Configuraciones de mantenimiento**.
    
    :::image type="content" source="media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-search-bar.png" alt-text="Captura de pantalla en la que se muestra cómo abrir Configuraciones de mantenimiento":::

1. Seleccione **Agregar**.

    :::image type="content" source="media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-add.png" alt-text="Captura de pantalla en la que se muestra cómo agregar una configuración de mantenimiento":::

1. En la pestaña Aspectos básicos, elija una suscripción y un grupo de recursos, proporcione un nombre para la configuración, elija una región y seleccione *Actualización de la imagen del sistema operativo* para el ámbito. Seleccione **Siguiente**.
    
    :::image type="content" source="media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-basics-tab.png" alt-text="Captura de pantalla en la que se muestran los elementos básicos de las configuraciones de mantenimiento":::

1. En la pestaña Programar, declare una ventana programada cuando Azure vaya a aplicar las actualizaciones en los recursos. Establezca una fecha de inicio, una ventana de mantenimiento y una periodicidad. Una vez que cree una ventana programada, ya no tendrá que aplicar las actualizaciones manualmente. Seleccione **Siguiente**. 

    > [!IMPORTANT]
    > La **duración** de la ventana de mantenimiento debe ser de *5 horas* o más. La **periodicidad** de mantenimiento debe establecerse para repetirse al menos una vez al día. 

    :::image type="content" source="media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-schedule-tab.png" alt-text="Captura de pantalla en la que se muestra la programación de la configuración de mantenimiento":::

1. En la pestaña Asignación, asigne recursos ahora u omita este paso y asigne recursos después de la implementación de la configuración de mantenimiento. Seleccione **Siguiente**.

1. Agregue las etiquetas y los valores. Seleccione **Siguiente**.
    
    :::image type="content" source="media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-tags-tab.png" alt-text="Captura de pantalla en la que se muestra cómo agregar etiquetas a una configuración de mantenimiento":::

1. Revise el resumen. Seleccione **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.


## <a name="assign-the-configuration"></a>Asignación de la configuración

En la página de detalles de la configuración de mantenimiento, seleccione **Asignaciones** y, después, **Asignar recurso**. 

![Captura de pantalla en la que muestra cómo asignar un recurso](media/virtual-machine-scale-sets-maintenance-control-portal/maintenance-configurations-add-assignment.png)

Seleccione los recursos a los que desea asignar la configuración de mantenimiento y elija **Aceptar**. En la columna **Tipo** se muestra si el recurso es una máquina virtual aislada o un host dedicado de Azure. Para asignar la configuración, es preciso que la máquina virtual esté en ejecución. Si se intenta asignar una configuración a una máquina virtual que está detenida, se produce un error. 


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información sobre el mantenimiento y las actualizaciones de las máquinas virtuales que se ejecutan en Azure](maintenance-and-updates.md)
