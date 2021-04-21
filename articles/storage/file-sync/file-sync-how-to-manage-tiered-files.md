---
title: Administración de archivos almacenados por niveles de Azure File Sync | Microsoft Docs
description: Sugerencias y commandlets de PowerShell para ayudarle a administrar archivos almacenados por niveles
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/13/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: bd740c773998450ef6e8bb95c4df3a1abadaceed
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796267"
---
# <a name="how-to-manage-tiered-files"></a>Administración de archivos en niveles

En este artículo se proporciona una guía para los usuarios que tienen preguntas relacionadas con la administración de archivos por niveles. Para preguntas conceptuales relacionadas con la nube por niveles, consulte [Preguntas frecuentes sobre Azure Files](../files/storage-files-faq.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json).

## <a name="how-to-check-if-your-files-are-being-tiered"></a>Comprobación de que los archivos están almacenados por niveles

El hecho de que los archivos deban organizarse por niveles según directivas establecidas se evalúa una vez a la hora. Al crear un punto de conexión de servidor, pueden darse dos situaciones:

Al agregar por primera vez un nuevo punto de conexión de servidor, a menudo existen archivos en esa ubicación de servidor. Estos archivos deben cargarse para que la funcionalidad de la nube por niveles pueda comenzar. La directiva de espacio disponible en el volumen no comienza a trabajar hasta que termina la carga inicial de todos los archivos. Pero la directiva de fecha opcional empieza a trabajar archivo a archivo, en cuanto se carga un archivo. El intervalo de una hora también se aplica aquí. 

Al agregar un nuevo punto de conexión de servidor, es posible que haya conectado una ubicación de servidor vacía a un recurso compartido de archivos de Azure que contiene los datos. Si decide descargar el espacio de nombres y recuperar el contenido durante la descarga inicial en el servidor, después de que el espacio de nombres quede inactivo, se recuperan los archivos en función de la marca de tiempo de última modificación, hasta alcanzar los límites de la directiva de espacio disponible del volumen y la directiva de fecha opcional.

Hay varias maneras de comprobar si un archivo se ha organizado en niveles en el recurso compartido de archivos de Azure:
    
   *  **Compruebe los atributos de archivo en el archivo.**
     Haga clic con el botón derecho en un archivo, vaya a **Detalles** y desplácese hacia abajo a la propiedad **Atributos**. Un archivo en niveles tendrá los siguientes atributos establecidos:     
        
        | Letra del atributo | Atributo | Definición |
        |:----------------:|-----------|------------|
        | A | Archivar | Indica que se debe realizar la copia de seguridad del archivo con un software de copia de seguridad. Este atributo siempre se establece, independientemente de si el archivo está organizado en niveles o está completamente almacenado en el disco. |
        | P | Archivos dispersos | Indica que el archivo es un archivo disperso. Un archivo disperso es un tipo especializado de archivo que ofrece NTFS para un uso eficaz cuando el archivo en el flujo del disco está en gran parte vacío. Azure File Sync usa archivos dispersos porque un archivo o está completamente organizado en niveles o parcialmente recuperado. En un archivo completamente organizado en niveles, su flujo de archivos se almacena en la nube. En un archivo parcialmente recuperado, esa parte del archivo ya está en el disco. Esto puede ocurrir cuando aplicaciones como reproductores multimedia o utilidades zip leen parcialmente los archivos. Si un archivo está totalmente recuperado en el disco, Azure File Sync lo convierte de un archivo disperso a un archivo normal. Este atributo solo está establecido en Windows Server 2016 y versiones anteriores.|
        | M | Recuperación del acceso a datos | Indica que los datos del archivo no están totalmente presentes en el almacenamiento local. La lectura del archivo hará que se capture al menos parte del contenido del archivo desde un recurso compartido de archivos de Azure al que está conectado el punto de conexión del servidor. Este atributo solo está establecido en Windows Server 2019. |
        | L | Punto de repetición de análisis | Indica que el archivo tiene un punto de repetición de análisis. Un punto de repetición de análisis es un puntero especial para su uso en un filtro del sistema de archivos. Azure File Sync usa puntos de repetición de análisis a fin de definir para el filtro del sistema de archivos de Azure File Sync (StorageSync.sys) la ubicación en la nube donde está almacenado el archivo. Esto permite el acceso sin problemas. Los usuarios no necesitan saber que se está usando Azure File Sync ni cómo obtener acceso al archivo en el recurso compartido de archivos de Azure. Cuando un archivo completo se recupera, Azure File Sync quita el punto de repetición de análisis del archivo. |
        | O | Sin conexión | Indica que parte del contenido del archivo, o la totalidad, no se ha almacenado en el disco. Cuando un archivo completo se recupera, Azure File Sync quita este atributo. |

        ![Cuadro de diálogo Propiedades de un archivo con la pestaña Detalles seleccionada](../files/media/storage-files-faq/azure-file-sync-file-attributes.png)
        
    
        > [!NOTE]
        > Puede ver los atributos de todos los archivos en una carpeta mediante la adición del campo **Atributos** a la presentación de la tabla del Explorador de archivos. Para ello, haga clic con el botón derecho en una columna existente (por ejemplo, **Tamaño**), seleccione **Más** y, después, **Atributos** en la lista desplegable.
        
        > [!NOTE]
        > Todos estos atributos serán visibles también para los archivos recuperados parcialmente.
        
   * **Use `fsutil` para buscar puntos de repetición de análisis en un archivo.**
       Como se indica en la opción anterior, un archivo organizado en niveles siempre tiene establecido un punto de repetición de análisis. Un punto de repetición de análisis permite al controlador de filtro del sistema de archivos de Azure File Sync (StorageSync.sys) recuperar el contenido de los recursos compartidos de archivos de Azure que no se almacenan localmente en el servidor. 

       Para comprobar si un archivo tiene un punto de repetición de análisis, en un símbolo del sistema con privilegios elevados o en una ventana de PowerShell, ejecute la utilidad `fsutil`:

        ```powershell
        fsutil reparsepoint query <your-file-name>
        ```
       Si el archivo tiene un punto de repetición de análisis, debería ver **Valor de la etiqueta de análisis: 0x8000001e**. Este valor hexadecimal es el valor del punto de repetición de análisis que pertenece a Azure File Sync. La salida también contendrá los datos de repetición de análisis que representan la ruta de acceso al archivo en el recurso compartido de archivos de Azure.
        
        > [!WARNING]  
        > El comando de la utilidad `fsutil reparsepoint` también puede eliminar un punto de repetición de análisis. No ejecute este comando a menos que el equipo de ingeniería de Azure File Sync se lo indique. Si lo ejecuta, podría producirse una pérdida de datos. 

## <a name="how-to-exclude-applications-from-cloud-tiering-last-access-time-tracking"></a>Exclusión de las aplicaciones del seguimiento del último acceso de la nube por niveles

Con la versión 11.1 del agente de Azure File Sync, ahora puede excluir las aplicaciones del seguimiento del último acceso. Cuando una aplicación tiene acceso a un archivo, la hora de último acceso para el archivo se actualiza en la base de datos de la nube por niveles. Las aplicaciones que examinan el sistema de archivos, como un antivirus, provocan que todos los archivos tengan la misma hora de último acceso, lo que afecta al momento en que los archivos se almacenan por niveles.

Para excluir las aplicaciones del seguimiento del último acceso, agregue el nombre del proceso al valor del Registro HeatTrackingProcessNameExclusionList, que se encuentra en HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure\StorageSync.

Ejemplo: reg ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure\StorageSync" /v HeatTrackingProcessNameExclusionList /t REG_MULTI_SZ /d "SampleApp.exe\0AnotherApp.exe" /f

> [!NOTE]
> Los procesos Desduplicación de datos y Administrador de recursos del servidor de archivos (FSRM) se excluyen de forma predeterminada. El sistema respeta los cambios en la lista de exclusión de procesos cada 5 minutos.

## <a name="how-to-access-the-heat-store"></a>Acceso al almacén térmico

La nube por niveles usa la hora del último acceso y la frecuencia de acceso de un archivo para determinar qué archivos deben almacenarse por niveles. El controlador de filtro de la nube por niveles (storagesync.sys) realiza un seguimiento de la hora del último acceso y registra la información en el almacén térmico de la nube por niveles. Puede recuperar el almacén térmico y guardarlo en un archivo CSV mediante un cmdlet de PowerShell local de servidor.

Hay un único almacén térmico para todos los archivos del mismo volumen. El almacén térmico puede llegar a ser muy grande. Si solo necesita recuperar el número de elementos a los que se accede con menos frecuencia, use -Limit y un número y también considere la posibilidad de filtrar por una subruta de acceso en lugar de la raíz del volumen.

- Importar el módulo de PowerShell: `Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'`

- ESPACIO DISPONIBLE DEL VOLUMEN: para obtener el orden en que los archivos se almacenan por niveles mediante la directiva de espacio disponible del volumen: `Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName FilesToBeTieredBySpacePolicy`

- DIRECTIVA DE FECHA: para obtener el orden en que los archivos se almacenan por niveles mediante la directiva de fecha: `Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName FilesToBeTieredByDatePolicy`

- Buscar información del almacén térmico para un archivo determinado: `Get-StorageSyncHeatStoreInformation -FilePath '<PathToSpecificFile>'`

- Ver todos los archivos en orden descendente por hora de último acceso: `Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName DescendingLastAccessTime`

- Ver el orden en el que se recuperarán los archivos almacenados por niveles mediante la recuperación en segundo plano o la recuperación a petición a través de PowerShell: `Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName OrderTieredFilesWillBeRecalled`

## <a name="how-to-force-a-file-or-directory-to-be-tiered"></a>Forzar el almacenamiento por niveles de un archivo o directorio

> [!NOTE]
> Cuando se selecciona un directorio para organizarlo por niveles, solo los archivos que se encuentran en ese momento en el directorio se organizan por niveles. Los archivos creados después de ese momento no se organizan por niveles automáticamente.

Cuando se habilita la característica de organización en niveles en la nube, se crean automáticamente niveles en los archivos según la última hora de acceso y de modificación a fin de conseguir el porcentaje de espacio libre en el volumen especificado en el punto de conexión en la nube. Aun así, en ocasiones podría querer forzar un archivo para organizarse en niveles de forma manual. Esto puede resultar útil cuando se guarda un archivo grande que no piensa volver a usar durante mucho tiempo y quiere espacio libre en el volumen ahora para usarlo para otros archivos o carpetas. Puede forzar la organización en niveles con los siguientes comandos de PowerShell:

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncCloudTiering -Path <file-or-directory-to-be-tiered>
```

## <a name="how-to-recall-a-tiered-file-to-disk"></a>Recuperación de un archivo almacenado por niveles en el disco

La manera más fácil de recuperar un archivo en el disco es abrirlo. El filtro del sistema de archivos de Azure File Sync (StorageSync.sys) descarga sin problemas el archivo desde el recurso compartido de archivos de Azure de forma automática. En el caso de los tipos de archivo que se pueden leer o transmitirse parcialmente, como los archivos multimedia o los archivos .zip, solo con abrir el archivo no se garantiza que se descargue entero.

Para que un archivo se descargue completamente en el disco local, debe usar PowerShell para obligar a un archivo a recuperarse por completo. Esta opción puede resultar también útil cuando se quiere recuperar muchos archivos a la vez, como todos los archivos de una carpeta. Abra una sesión de PowerShell en el nodo del servidor donde está instalado Azure File Sync y ejecute los siguientes comandos de PowerShell:
    
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint>
```
Parámetros opcionales:
* `-Order CloudTieringPolicy` recupera primero los archivos modificados o a los que se ha accedido más recientemente permitidos por la directiva de niveles actual. 
    * Si la directiva de espacio disponible en el volumen está configurada, se recuperan archivos hasta que se alcanza el valor de esta. Por ejemplo, si el valor de la directiva es 20 %, la recuperación se detiene una vez que el espacio disponible en el volumen alcanza el 20 %.  
    * Si la directiva de espacio disponible en el volumen y la directiva de fecha están configuradas, se recuperan archivos hasta que se alcanzan los valores de estas. Por ejemplo, si el valor de la directiva de espacio disponible es 20 % y el de la directiva de fecha es 7 días, la recuperación se detiene una vez que el espacio disponible en el volumen alcanza el 20 % o si todos los archivos a los que se ha accedido o que se han modificado en los últimos 7 días son locales.
* `-ThreadCount` determina el número de archivos que se pueden recuperar en paralelo.
* `-PerFileRetryCount` determina la frecuencia con que se intentará recuperar un archivo que está bloqueado actualmente.
* `-PerFileRetryDelaySeconds` determina el tiempo en segundos entre los reintentos de recuperación y siempre se debe usar en combinación con el parámetro anterior.

Ejemplo:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -ThreadCount 8 -Order CloudTieringPolicy -PerFileRetryCount 3 -PerFileRetryDelaySeconds 10
``` 

> [!Note]  
>- Si el volumen local que hospeda el servidor no tiene suficiente espacio disponible para recuperar todos los datos con niveles, el cmdlet `Invoke-StorageSyncFileRecall` dará error.  

> [!Note]
> Para recuperar los archivos que están organizados en niveles, el ancho de banda de red debe ser de al menos 1 Mbps. Si el ancho de banda de red es menor que 1 Mbps, puede ocurrir un error de tiempo de expiración al recuperar los archivos.

## <a name="next-steps"></a>Pasos siguientes

* [Preguntas más frecuentes (P+F) sobre Azure Files](../files/storage-files-faq.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json)