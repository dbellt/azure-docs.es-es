---
title: Restauración de blobs de Azure
description: Obtenga información sobre cómo restaurar blobs de Azure.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: aaca425e2ea13948fbd52bf60ed30d64c29c27d3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108767282"
---
# <a name="restore-azure-blobs"></a>Restauración de blobs de Azure

Los blobs en bloques en cuentas de almacenamiento que tengan configurada la copia de seguridad operativa se pueden restaurar a cualquier punto en el tiempo dentro de la duración de la retención. Además, puede determinar que se restauren todos los blobs en bloques de la cuenta de almacenamiento o solo un subconjunto.

## <a name="before-you-start"></a>Antes de comenzar

- Los blobs se restaurarán en la misma cuenta de almacenamiento. Por lo tanto, se sobrescribirán los blobs que hayan sufrido cambios desde el momento que se está restaurando.
- Solo los blobs en bloques de una cuenta de almacenamiento estándar de uso general v2 pueden restaurarse como parte de una operación de restauración. Los blobs en anexos, blobs en páginas y blobs en bloques prémium no se restauran.
- Mientras haya un trabajo de restauración en curso, no se puede leer ni escribir en los blobs del almacenamiento.
- No se puede restaurar un blob con una concesión activa. Si se incluye un blob con una concesión activa en el intervalo de blobs para restaurar, la operación de restauración producirá un error automáticamente. Interrumpa las concesiones activas antes de iniciar la operación de restauración.
- Las instantáneas no se crean ni se eliminan como parte de una operación de restauración. Solo el blob base se restaura a su estado anterior.
- Si elimina un contenedor de la cuenta de almacenamiento llamando a la operación **Eliminar contenedor**, dicho contenedor no se puede restaurar con una operación de restauración. En lugar de eliminar un contenedor completo, elimine blobs individuales, por si desea restaurarlos más adelante. Además, Microsoft recomienda habilitar la eliminación temporal de los contenedores, además de la copia de seguridad operativa, como protección contra la eliminación accidental de contenedores.
- Consulte la [tabla de compatibilidad](blob-backup-support-matrix.md) para ver todas las limitaciones y los escenarios admitidos.

## <a name="restore-blobs"></a>Restauración de blobs

Puede iniciar una restauración en el Centro de copias de seguridad.

1. En el Centro de copias de seguridad, vaya a **Restaurar** en la barra superior.

    ![Restauración en el Centro de copias de seguridad](./media/blob-restore/backup-center-restore.png)

1. En la pestaña **Iniciar restauración**, elija **Blobs de Azure (Azure Storage)** como tipo de origen de datos, y seleccione la **Instancia de copia de seguridad** que desea restaurar. Aquí, la instancia de copia de seguridad es la cuenta de almacenamiento que contiene los blobs que desea restaurar.

     ![Selección de una instancia de copia de seguridad](./media/blob-restore/select-backup-instance.png)

1. En la pestaña **Seleccione un punto de recuperación**, elija de qué fecha y hora desea restaurar los datos. También puede usar el control deslizante para elegir el momento que se va a restaurar. La burbuja de información que se encuentra junto a la fecha muestra la duración válida desde la cual puede restaurar sus datos. Como la copia de seguridad operativa de los blobs es continua, ofrece un control granular sobre los puntos de los cuales se pueden recuperar datos.

    >[!NOTE]
    > La hora indicada es la hora local.

    ![Fecha y hora de la restauración](./media/blob-restore/date-and-time.png)

1. En la pestaña **Parámetros de restauración**, elija si quiere restaurar todos los blobs de la cuenta de almacenamiento, contenedores específicos o un subconjunto de blobs cuyos prefijos coincidan. Con la coincidencia de prefijo puede especificar hasta 10 intervalos de prefijos o rutas de archivo. [Aquí](#use-prefix-match-for-restoring-blobs) encontrará más información sobre cómo usar la coincidencia de prefijos.

    ![Restauración de parámetros](./media/blob-restore/restore-parameters.png)

    Elija una de estas opciones:

    - **Restore all blobs in the storage account** (Restaurar todos los blobs de la cuenta de almacenamiento): con esta opción se revierten todos los blobs en bloques de la cuenta de almacenamiento al momento del tiempo seleccionado para restaurarlos. Las cuentas de almacenamiento que contienen grandes cantidades de datos o con una alta tasa de renovación pueden tardar más tiempo en restaurarse.

    - **Browse and select containers to restore** (Examinar y seleccionar los contenedores para restaurar): con esta opción podrá examinar y seleccionar hasta 10 contenedores para restaurar. Debe tener los permisos necesarios para ver los contenedores de la cuenta de almacenamiento; de lo contrario, es posible que no pueda ver el contenido de la cuenta de almacenamiento.

    - **Prefix match** (Coincidencia de prefijo): esta opción permite restaurar un subconjunto de blobs que coincidan con un prefijo. Puede restaurar hasta 10 intervalos lexicográficos de blobs dentro de un solo contenedor o en varios contenedores para devolver esos blobs a su estado anterior en un momento dado. Hay algunos aspectos que se deben tener en cuenta:

        - Puede usar una barra diagonal (/) para delimitar el nombre del contenedor del prefijo de blob.
        - El inicio del intervalo especificado es inclusivo, pero el intervalo especificado es exclusivo.

    Para obtener más información sobre el uso de prefijos para restaurar intervalos de blobs, consulte [esta sección](#use-prefix-match-for-restoring-blobs).

1. Una vez que termine de especificar los blobs que desea restaurar, continúe en la pestaña **Revisar y restaurar** y seleccione **Restaurar** para iniciar la restauración.

1. **Seguimiento de la restauración**: use la vista **Trabajos de copia de seguridad** para hacer un seguimiento de los detalles y el estado de las restauraciones. Para ello, vaya a **Centro de copias de seguridad** > **Trabajos de copia de seguridad**. El estado mostrará **En curso** mientras se realiza la restauración.

    ![Pestaña Trabajos de copia de seguridad](./media/blob-restore/backup-jobs.png)

    Cuando la operación de restauración se complete correctamente, el estado cambiará a **Completado**. Una vez que la restauración finalice correctamente, podrá volver a leer y escribir blobs en la cuenta de almacenamiento.

## <a name="additional-topics"></a>Otros temas

### <a name="use-prefix-match-for-restoring-blobs"></a>Uso de coincidencia de prefijo para restaurar blobs

Considere el ejemplo siguiente:

![Restauración con coincidencia de prefijo](./media/blob-restore/prefix-match.png)

La operación de restauración que se muestra en la imagen realiza las acciones siguientes:

- Restaura todo el contenido de *container1*.
- Restaura los blobs del intervalo lexicográfico *blob1* a *blob5* de *container2*. Este rango restaura blobs con nombres como *blob1*, *blob11*, *blob100*, *blob2*, etc. Como el final del intervalo es exclusivo, restaura los blobs cuyos nombres empiezan con *blob4*, pero no los blobs cuyos nombres empiezan con *blob5*.
- Restaura todos los blobs de *container3* y *container4*. Como el final del intervalo es exclusivo, este intervalo no restaura *container5*.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a la copia de seguridad operativa para blobs de Azure](blob-backup-overview.md)
