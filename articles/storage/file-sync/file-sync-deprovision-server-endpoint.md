---
title: Desaprovisionamiento de un punto de conexión de servidor de Azure File Sync | Microsoft Docs
description: Instrucciones sobre cómo desaprovisionar un punto de conexión de servidor de Azure File Sync en función del caso de uso
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 4/23/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: c6286c22234f7f9a32b58245835dc2df8467c725
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992780"
---
# <a name="deprovision-your-azure-file-sync-server-endpoint"></a>Desaprovisionamiento de un punto de conexión de servidor de Azure File Sync

Antes de desaprovisionar el punto de conexión de servidor, hay algunos pasos que debe seguir para mantener la integridad y disponibilidad de los datos. En este artículo se tratan varios métodos de desaprovisionamiento y las instrucciones adecuadas ordenadas por escenario. Siga los pasos para el caso de uso que mejor se le aplique.

Si no hay problema en perder permanentemente los datos que está sincronizando actualmente, puede pasar directamente a desaprovisionar el punto de conexión de servidor.

> [!Important]
> No intente resolver problemas de sincronización mediante el desaprovisionamiento de un punto de conexión de servidor. Para obtener ayuda para solucionar problemas, consulte [Solución de problemas de Azure File Sync](./file-sync-troubleshoot.md). La pérdida de datos permanente puede producirse si elimina el punto de conexión de servidor sin tener el lado servidor o de la nube totalmente sincronizados entre ellos. 

## <a name="scenario-1-you-intend-to-delete-your-server-endpoint-and-stop-using-your-local-servervm"></a>Escenario 1: tiene previsto eliminar el punto de conexión de servidor y dejar de usar el servidor local o la máquina virtual

El objetivo aquí es asegurarse de que los datos están actualizados en el punto de conexión de nube. Para tener el conjunto completo de archivos actualizados en los puntos de conexión de servidor en su lugar, consulte [Escenario 2: tiene previsto eliminar el punto de conexión de servidor y dejar de usar este recurso compartido de archivos de Azure específico](#scenario-2-you-intend-to-delete-your-server-endpoint-and-stop-using-this-specific-azure-file-share).

Algunos casos de uso que se incluyen en esta categoría son:
-   Migración a un recurso compartido de archivos de Azure
-   Uso sin servidor
-   Interrupción del uso de una ruta de acceso de punto de conexión de servidor específica mientras se mantiene intacto el resto del grupo de sincronización

En este escenario, hay tres pasos que debe realizar antes de eliminar el punto de conexión de servidor: quitar el acceso de usuario, iniciar una sesión de carga de VSS especial y esperar a que se complete una sesión de sincronización final.

### <a name="remove-user-access-to-your-server-endpoint"></a>Eliminación del acceso de usuario al punto de conexión de servidor

Antes de desaprovisionar el punto de conexión de servidor, debe asegurarse de que todos los cambios del servidor se pueden sincronizar con la nube. El primer paso para permitir que la nube pueda ponerse al día es quitar la oportunidad de realizar cambios adicionales en los archivos y carpetas en el punto de conexión de servidor. 

Quitar el acceso significa tiempo de inactividad. Para reducir el tiempo de inactividad, también puede considerar la posibilidad de redirigir el acceso de usuario al punto de conexión de nube. 

Registre la fecha y hora en que quitó el acceso de usuario para sus propios registros y, a continuación, pase a la sección siguiente.

### <a name="initiate-a-special-volume-snapshot-service-vss-upload-session"></a>Inicio de una sesión de carga especial del Servicio de instantáneas de volumen (VSS)

Azure File Sync crea una instantánea VSS temporal todos los días en el servidor para sincronizar archivos con identificadores abiertos. Para asegurarse de que la sesión de sincronización final carga los datos más recientes y para reducir los errores por elemento, inicie una sesión especial para la carga de VSS. Esto también desencadenará una sesión de carga de sincronización especial que comienza una vez que se toma la instantánea.  

Para ello, abra el **Programador de tareas** en el servidor local, vaya a **Microsoft\StorageSync**, haga clic con el botón derecho en la tarea **VssSyncScheduledTask** y seleccione **Ejecutar**.

> [!Important]
> Anote la fecha y hora en que complete este paso. Lo necesitará en la siguiente sección.

![Captura de pantalla de la programación de una sesión de carga de VSS.](media/storage-sync-deprovision-server-endpoint/vss-task-scheduler.png)

### <a name="wait-for-a-final-sync-upload-session-to-complete"></a>Espera hasta que se complete una sesión de carga de sincronización final

Para asegurarse de que los datos más recientes están en la nube, debe esperar a que se complete la sesión de carga de sincronización final. 

Para comprobar el estado de la sesión de sincronización, abra el **Visor de eventos** en el servidor local. Vaya al registro de eventos de telemetría **(Applications and Services\Microsoft\FileSync\Agent)** . Asegúrese de que ve un evento 9102 con "sync direction" = upload, "HResult" = 0 y "PerItemErrorCount" = 0 que se produjo después de iniciar manualmente una sesión de carga de VSS.

![Captura de pantalla de la comprobación de si se ha completado una sesión de sincronización final.](media/storage-sync-deprovision-server-endpoint/event-viewer.png)

Si "PerItemErrorCount" es mayor que 0, los archivos no se pueden sincronizar. Use **FileSyncErrorsReport.ps1** para ver los archivos que no se pueden sincronizar. Este script de PowerShell normalmente se encuentra en esta ruta de acceso en un servidor con un agente de Azure File Sync instalado: **C:\Program Files\Azure\StorageSyncAgent\FileSyncErrorsReport.ps1**.

Si estos archivos no son importantes, puede eliminar el punto de conexión de servidor. Si son importantes, corrija sus errores y espere a que se produzca otro evento 9102 con "sync direction" = upload, "HResult" = 0 y "PerItemErrorCount" = 0 antes de eliminar el punto de conexión de servidor.

## <a name="scenario-2-you-intend-to-delete-your-server-endpoint-and-stop-using-this-specific-azure-file-share"></a>Escenario 2: tiene previsto eliminar el punto de conexión de servidor y dejar de usar este recurso compartido de archivos de Azure específico

El objetivo aquí es asegurarse de que los datos están actualizados en el servidor local o la máquina virtual. Para tener el conjunto completo de archivos actualizados en los puntos de conexión de nube en su lugar, consulte [Escenario 1: tiene previsto eliminar el punto de conexión de servidor y dejar de usar el servidor local o la máquina virtual](#scenario-1-you-intend-to-delete-your-server-endpoint-and-stop-using-your-local-servervm).

En este escenario, hay cuatro pasos que debe seguir antes de eliminar el punto de conexión de servidor: deshabilitar la nube por niveles, recuperar archivos en capas, iniciar la detección de cambios en la nube y esperar a que se complete una sesión de sincronización final.

### <a name="disable-cloud-tiering"></a>Deshabilitación de la nube por niveles
Vaya a la sección Nube por niveles en **Propiedades del punto de conexión de servidor** del punto de conexión de servidor que le gustaría desaprovisionar y para el que quiere deshabilitar la nube por niveles.

### <a name="recall-all-tiered-files"></a>Recuperación de todos los archivos en capas
Incluso si la nube por niveles está deshabilitada, debe recuperar todos los archivos en capas para asegurarse de que todos los archivos se almacenan localmente.

Antes de recuperar los archivos, asegúrese de que tiene suficiente espacio disponible localmente para almacenar todos los archivos. El espacio disponible debe ser aproximadamente el tamaño del recurso compartido de archivos de Azure en la nube menos el tamaño almacenado en caché en el servidor.

Use el cmdlet **Invoke-StorageSyncFileRecall** de PowerShell y especifique el parámetro **SyncGroupName** para recuperar todos los archivos. 
```powershell
Invoke-StorageSyncFileRecall  -SyncGroupName “samplesyncgroupname”
```
Una vez que este cmdlet haya terminado de ejecutarse, puede pasar a la sección siguiente.

### <a name="initiate-cloud-change-detection"></a>Inicio de la detección de cambios en la nube
Iniciar la detección de cambios en la nube garantiza que se han sincronizado los cambios más recientes.

Puede iniciar la detección de cambios con el cmdlet Invoke-AzStorageSyncChangeDetection: 

```powershell
Invoke-AzStorageSyncChangeDetection -ResourceGroupName "myResourceGroup" -StorageSyncServiceName "myStorageSyncServiceName" -SyncGroupName "mySyncGroupName" -Path "Data","Reporting\Templates" 
```

Este paso puede tardar un tiempo en completarse. 

> [!Important]
> Una vez completado este examen de detección de cambios en la nube iniciado, anote la fecha y hora en que se completó. Necesitará esta información en la sección siguiente.

### <a name="wait-for-a-final-sync-session-to-complete"></a>Espera hasta que se complete una sesión de sincronización final
Para asegurarse de que los datos están actualizados en el servidor local, debe esperar a que se complete una sesión de carga de sincronización final. 

Para comprobarlo, vaya a **Visor de eventos** en el servidor local. Vaya al registro de eventos de telemetría **(Applications and Services\Microsoft\FileSync\Agent)** . Asegúrese de que ve un evento 9102 con "sync direction" = download, "HResult" = 0 y "PerItemErrorCount" = 0 que se produjo una vez finalizada la detección de cambios en la nube de fecha y hora.

![Captura de pantalla de la comprobación de si se ha completado una sesión de sincronización final.](media/storage-sync-deprovision-server-endpoint/event-viewer.png)

Si "PerItemErrorCount" es mayor que 0, los archivos no se pueden sincronizar. Use **FileSyncErrorsReport.ps1** para ver los archivos que no se pueden sincronizar. Este script de PowerShell normalmente se encuentra en esta ruta de acceso en un servidor con un agente de Azure File Sync instalado: **C:\Program Files\Azure\StorageSyncAgent\FileSyncErrorsReport.ps1**.

Si estos archivos no son importantes, puede eliminar el punto de conexión de servidor. Si son importantes, corrija sus errores y espere a que se produzca otro evento 9102 con "sync direction" = download, "HResult" = 0 y "PerItemErrorCount" = 0 antes de eliminar el punto de conexión de servidor.

## <a name="next-steps"></a>Pasos siguientes
* [Modificación de la topología de Azure File Sync](./file-sync-modify-sync-topology.md)







