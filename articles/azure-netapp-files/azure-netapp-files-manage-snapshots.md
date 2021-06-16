---
title: Administración de instantáneas mediante Azure NetApp Files | Microsoft Docs
description: Describe cómo crear, administrar y usar instantáneas mediante Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/14/2021
ms.author: b-juche
ms.openlocfilehash: b0cc4deebac60fcf0ac4df20b112b8f69da05414
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112072116"
---
# <a name="manage-snapshots-by-using-azure-netapp-files"></a>Administración de instantáneas mediante Azure NetApp Files

Azure NetApp Files admite la creación de instantáneas a petición y el uso de directivas de instantáneas para programar la creación automática de instantáneas. También puede restaurar una instantánea en un nuevo volumen, restaurar un único archivo mediante un cliente o revertir un volumen existente mediante una instantánea.

> [!NOTE] 
> Para conocer las consideraciones sobre la administración de instantáneas en la replicación entre regiones, consulte [Requisitos y consideraciones del uso de la replicación entre regiones](cross-region-replication-requirements-considerations.md).

## <a name="create-an-on-demand-snapshot-for-a-volume"></a>Creación de una instantánea a petición para un volumen

Puede crear instantáneas de volumen a petición. 

1.  Vaya al volumen para el que desee crear una instantánea. Haga clic en **Instantáneas**.

    ![Desplazamiento hasta las instantáneas](../media/azure-netapp-files/azure-netapp-files-navigate-to-snapshots.png)

2.  Haga clic en **+ Add snapshot** (+Agregar instantánea) para crear una instantánea a petición para un volumen.

    ![Agregar instantánea](../media/azure-netapp-files/azure-netapp-files-add-snapshot.png)

3.  En la ventana Nueva instantánea, proporcione un nombre para la nueva instantánea que va a crear.   

    ![Nueva instantánea](../media/azure-netapp-files/azure-netapp-files-new-snapshot.png)

4. Haga clic en **OK**. 

## <a name="manage-snapshot-policies"></a>Administración de directivas de instantánea

Puede programar la realización automática de instantáneas de volumen mediante el uso de directivas de instantánea. También puede modificar una directiva de instantánea según sea necesario o eliminar una directiva de instantánea que ya no necesite.  

### <a name="create-a-snapshot-policy"></a>Creación de una directiva de instantánea 

Una directiva de instantánea le permite especificar la frecuencia de creación de instantáneas en ciclos de horas, diarios, semanales o mensuales. También debe especificar el número máximo de instantáneas que se conservarán para el volumen.  

1.  En la vista Cuenta de NetApp, haga clic en **Directiva de instantánea**.

    ![Navegación por la directiva de instantánea](../media/azure-netapp-files/snapshot-policy-navigation.png)

2.  En la ventana Directiva de instantánea, establezca el estado de la directiva en **Habilitado**. 

3.  Haga clic en la pestaña **Por hora**, **Cada día**, **Semanalmente** o **Mensualmente** para crear directivas de instantánea por hora, diarias, semanales o mensuales. Seleccione el **número de instantáneas que desea mantener**.  

    Consulte [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md) acerca del número máximo de instantáneas permitido para un volumen. 

    En el ejemplo siguiente se muestra la configuración de la directiva de instantánea por hora. 

    ![Directiva de instantánea por hora](../media/azure-netapp-files/snapshot-policy-hourly.png)

    En el ejemplo siguiente se muestra la configuración de la directiva de instantánea diaria.

    ![Directiva de instantánea diaria](../media/azure-netapp-files/snapshot-policy-daily.png)

    En el ejemplo siguiente se muestra la configuración de la directiva de instantánea semanal.

    ![Directiva de instantánea semanal](../media/azure-netapp-files/snapshot-policy-weekly.png)

    En el ejemplo siguiente se muestra la configuración de la directiva de instantánea mensual.

    ![Directiva de instantánea mensual](../media/azure-netapp-files/snapshot-policy-monthly.png) 

4.  Haga clic en **Save**(Guardar).  

Si tiene que crear directivas de instantánea adicionales, repita el paso 3.
Las directivas que ha creado aparecen en la página Directiva de instantánea.

Si desea que un volumen use la directiva de instantánea, debe [aplicar la directiva al volumen](azure-netapp-files-manage-snapshots.md#apply-a-snapshot-policy-to-a-volume). 

### <a name="apply-a-snapshot-policy-to-a-volume"></a>Aplicación de una directiva de instantánea a un volumen

Si desea que un volumen use una directiva de instantánea que haya creado, debe aplicarla al volumen. 

No se puede aplicar una directiva de instantánea a un volumen de destino en la replicación entre regiones.  

1.  Vaya a la página **Volúmenes**, haga clic con el botón derecho en el volumen al que desee aplicar una directiva de instantánea y seleccione **Editar**.

    ![Menú contextual de volúmenes](../media/azure-netapp-files/volume-right-cick-menu.png) 

2.  En la ventana de edición, en **Directiva de instantánea**, seleccione la directiva que se vaya a usar para el volumen.  Haga clic en **Aceptar** para aplicar la directiva.  

    ![Edición de directiva de instantánea](../media/azure-netapp-files/snapshot-policy-edit.png) 

### <a name="modify-a-snapshot-policy"></a>Modificación de una directiva de instantánea 

Puede modificar una directiva de instantánea existente para cambiar su estado, la frecuencia (cada hora, diariamente, semanalmente o mensualmente) o el número de instantáneas que se deben conservar.  
 
1.  En la vista Cuenta de NetApp, haga clic en **Directiva de instantánea**.

2.  Haga clic con el botón derecho en la directiva de instantánea que desee modificar y, a continuación, seleccione **Editar**.

    ![Menú contextual de directiva de instantánea](../media/azure-netapp-files/snapshot-policy-right-click-menu.png) 

3.  Realice los cambios en la ventana Directiva de instantánea que aparezca y, a continuación, haga clic en **Guardar**. 

### <a name="delete-a-snapshot-policy"></a>Eliminación de una directiva de instantánea 

Puede eliminar una directiva de instantánea que ya no desee conservar.   

1.  En la vista Cuenta de NetApp, haga clic en **Directiva de instantánea**.

2.  Haga clic con el botón derecho en la directiva de instantánea que desee modificar y, a continuación, seleccione **Eliminar**.

    ![Menú contextual de directiva de instantánea](../media/azure-netapp-files/snapshot-policy-right-click-menu.png) 

3.  Haga clic en **Sí** para confirmar que desea eliminar la directiva de instantánea.   

    ![Confirmación de la eliminación de una directiva de instantánea](../media/azure-netapp-files/snapshot-policy-delete-confirm.png) 

## <a name="edit-the-hide-snapshot-path-option"></a>Edición de la opción Ocultar la ruta de acceso de la instantánea
La opción Ocultar la ruta de acceso de la instantánea controla si está visible la ruta de acceso de la instantánea de un volumen. Durante la creación de un volumen de [NFS](azure-netapp-files-create-volumes.md#create-an-nfs-volume) o [SMB](azure-netapp-files-create-volumes-smb.md#add-an-smb-volume), puede especificar si se debe ocultar la ruta de acceso de la instantánea. Posteriormente, puede editar esta opción según proceda.  

> [!NOTE]
> En el caso de un [volumen de destino](cross-region-replication-create-peering.md#create-the-data-replication-volume-the-destination-volume) en la replicación entre regiones, la opción Ocultar la ruta de acceso de la instantánea está habilitada de forma predeterminada y no se puede modificar. 

1. Para ver la opción Ocultar la ruta de acceso de la instantánea de un volumen, seleccione el volumen. En el campo **Ocultar la ruta de acceso de la instantánea** se muestra si la opción está habilitada.   
    ![Captura de pantalla que describe el campo Ocultar la ruta de acceso de la instantánea.](../media/azure-netapp-files/hide-snapshot-path-field.png) 
2. Para editar la opción Ocultar la ruta de acceso de la instantánea, haga clic en **Editar** en la página del volumen y modifique la opción **Ocultar la ruta de acceso de la instantánea** como proceda.   
    ![Captura de pantalla que describe la opción de edición de la instantánea de volumen.](../media/azure-netapp-files/volume-edit-snapshot-options.png) 

## <a name="restore-a-snapshot-to-a-new-volume"></a>Restauración de una instantánea a un nuevo volumen

Actualmente, puede restaurar una instantánea solo a un nuevo volumen. 
1. Seleccione **Instantáneas** en la hoja Volumen para mostrar la lista de instantáneas. 
2. Haga clic con el botón derecho en la instantánea que desee restaurar y seleccione **Restaurar al nuevo volumen** en la opción de menú.  

    ![Restauración de la instantánea al nuevo volumen](../media/azure-netapp-files/azure-netapp-files-snapshot-restore-to-new-volume.png)

3. En la ventana Crear un volumen, especifique la información del nuevo volumen:  
    * **Nombre**   
        Especifique el nombre para el volumen que va a crear.  
        
        El nombre debe ser único dentro de un grupo de recursos. Debe tener tres caracteres de longitud, como mínimo.  Puede usar cualquier carácter alfanumérico.

    * **Cuota**  
        Especifique la cantidad de almacenamiento lógico que desee asignar al volumen.  

    ![Restauración en el nuevo volumen](../media/azure-netapp-files/snapshot-restore-new-volume.png) 

4. Haga clic en **Revisar y crear**.  Haga clic en **Crear**.   
    El nuevo volumen utiliza el mismo protocolo que la instantánea.   
    El nuevo volumen al que se restaura la instantánea aparece en la hoja Volúmenes.

## <a name="restore-a-file-from-a-snapshot-using-a-client"></a>Restauración de un archivo desde una instantánea mediante un cliente

Si no desea [restaurar la instantánea completa en un volumen](#restore-a-snapshot-to-a-new-volume), tiene la opción de restaurar un archivo desde una instantánea mediante un cliente que tenga el volumen montado.  

El volumen montado contiene un directorio de instantáneas denominado `.snapshot` (en clientes NFS) o `~snapshot` (en clientes SMB) que es accesible para el cliente. El directorio de instantáneas contiene subdirectorios correspondientes a las instantáneas del volumen. Cada subdirectorio contiene los archivos de la instantánea. Si elimina o sobrescribe accidentalmente un archivo, puede restaurarlo en el directorio principal de lectura y escritura copiando el archivo de un subdirectorio de instantánea en el directorio de lectura y escritura. 

Puede controlar el acceso a los directorios de instantáneas mediante la opción [Ocultar la ruta de acceso de la instantánea](#edit-the-hide-snapshot-path-option). Esta opción controla si el directorio debe ocultarse para los clientes. Por lo tanto, también controla el acceso a los archivos y carpetas de las instantáneas.  

NFSv 4.1 no muestra el directorio `.snapshot` (`ls -la`). Sin embargo, aunque no establezca la opción Ocultar la ruta de acceso de la instantánea, todavía puede acceder al directorio `.snapshot` a través de NFSv 4.1 mediante el comando `cd <snapshot-path>` desde la línea de comandos del cliente. 

### <a name="restore-a-file-by-using-a-linux-nfs-client"></a>Restauración de un archivo mediante un cliente NFS de Linux 

1. Utilice el comando de Linux `ls` para mostrar el archivo que desea restaurar desde el directorio `.snapshot`. 

    Por ejemplo:

    `$ ls my.txt`   
    `ls: my.txt: No such file or directory`   

    `$ ls .snapshot`   
    `daily.2020-05-14_0013/              hourly.2020-05-15_1106/`   
    `daily.2020-05-15_0012/              hourly.2020-05-15_1206/`   
    `hourly.2020-05-15_1006/             hourly.2020-05-15_1306/`   

    `$ ls .snapshot/hourly.2020-05-15_1306/my.txt`   
    `my.txt`

2. Utilice el comando `cp` para copiar el archivo en el directorio principal.  

    Por ejemplo: 

    `$ cp .snapshot/hourly.2020-05-15_1306/my.txt .`   

    `$ ls my.txt`   
    `my.txt`   

### <a name="restore-a-file-by-using-a-windows-client"></a>Restauración un archivo mediante un cliente Windows 

1. Si el directorio `~snapshot` del volumen está oculto, [muestre los elementos ocultos](https://support.microsoft.com/help/4028316/windows-view-hidden-files-and-folders-in-windows-10) en el directorio principal para ver `~snapshot`.

    ![Mostrar elementos ocultos](../media/azure-netapp-files/snapshot-show-hidden.png) 

2. Vaya al subdirectorio de `~snapshot` para buscar el archivo que desea restaurar.  Haga clic con el botón derecho en el archivo. Seleccione **Copiar**.  

    ![Copia de archivo para restaurar](../media/azure-netapp-files/snapshot-copy-file-restore.png) 

3. Vuelva al directorio principal. Haga clic con el botón derecho en el directorio principal y seleccione `Paste` para pegar el archivo en el directorio.

    ![Pegado de archivo para restaurar](../media/azure-netapp-files/snapshot-paste-file-restore.png) 

4. También puede hacer clic con el botón derecho en el directorio principal, seleccionar **Propiedades**, hacer clic en la pestaña **Versiones anteriores** para ver la lista de instantáneas y seleccionar **Restaurar** para restaurar un archivo.  

    ![Versiones anteriores de Propiedades](../media/azure-netapp-files/snapshot-properties-previous-version.png) 

## <a name="revert-a-volume-using-snapshot-revert"></a>Reversión de un volumen mediante la reversión de instantáneas

La funcionalidad de reversión de instantáneas le permite revertir rápidamente un volumen al estado en que se encontraba cuando se tomó una instantánea determinada. En la mayoría de los casos, la reversión de un volumen es mucho más rápida que la restauración de archivos individuales a partir de una instantánea en el sistema de archivos activo. También requiere menos espacio que la restauración de una instantánea en un nuevo volumen. 

Puede encontrar la opción Revertir volumen en el menú Instantáneas de un volumen. Después de seleccionar una instantánea para la reversión, Azure NetApp Files revierte el volumen a los datos y las marcas de tiempo que contenía cuando se tomó la instantánea seleccionada. 

> [!IMPORTANT]
> Los datos del sistema de archivos activo y las instantáneas que se tomaron después de tomar la instantánea seleccionada se perderán. La operación de reversión de instantáneas reemplazará *todos* los datos del volumen de destino por los datos de la instantánea seleccionada. Debe prestar atención al contenido y a la fecha de creación de la instantánea al seleccionar una instantánea. No se puede deshacer la operación de reversión de instantáneas.

1. Vaya al menú **Instantáneas** de un volumen.  Haga clic con el botón derecho en la instantánea que quiere utilizar para la operación de reversión. Seleccione **Revertir volumen**. 

    ![Captura de pantalla que describe el menú contextual de una instantánea](../media/azure-netapp-files/snapshot-right-click-menu.png) 

2. En la ventana Revertir el volumen a la instantánea, escriba el nombre del volumen y haga clic en **Revertir**.   

    El volumen ahora se restaura al momento en que se tomó la instantánea seleccionada.

    ![Captura de pantalla que muestra la ventana Revertir el volumen a la instantánea](../media/azure-netapp-files/snapshot-revert-volume.png) 

## <a name="delete-snapshots"></a>Eliminar instantáneas  

Puede eliminar las instantáneas que ya no necesita conservar. 

> [!IMPORTANT]
> La operación de eliminación de instantáneas no se puede deshacer. No se puede recuperar una instantánea eliminada. 

1. Vaya al menú **Instantáneas** de un volumen. Haga clic con el botón derecho en la instantánea que desea eliminar. Seleccione **Eliminar**.

    ![Captura de pantalla que describe el menú contextual de una instantánea](../media/azure-netapp-files/snapshot-right-click-menu.png) 

2. En la ventana Eliminar instantánea, haga clic en **Sí** para confirmar que quiere eliminar la instantánea. 

    ![Captura de pantalla que confirma la eliminación de la instantánea](../media/azure-netapp-files/snapshot-confirm-delete.png)  

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de directivas de instantáneas](troubleshoot-snapshot-policies.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Vídeo Instantáneas de Azure NetApp Files 101](https://www.youtube.com/watch?v=uxbTXhtXCkw&feature=youtu.be)
* [¿Qué es la herramienta Azure Application Consistent Snapshot?](azacsnap-introduction.md)
