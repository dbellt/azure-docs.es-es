---
title: Almacenamiento y movimiento de datos para la representación
description: Conozca las distintas opciones de almacenamiento y movimiento de datos para representar cargas de trabajo de archivos de recursos y salida.
services: batch
ms.service: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: 0a18ee6961cb601b0fa9db7213eb6115afa20096
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765204"
---
# <a name="storage-and-data-movement-options-for-rendering-asset-and-output-files"></a>Opciones de almacenamiento y movimiento de datos para representar archivos de recursos y de salida

Existen varias opciones para que las aplicaciones de representación del grupo de máquinas virtuales puedan usar los archivos de escena y de recursos:

* [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md):
  * Tanto los archivos de escena como los de recursos se cargan en Blob Storage desde un sistema de archivos local. Cuando es una tarea la que ejecuta la aplicación, los archivos necesarios se copian de Blob Storage a la máquina virtual, con el fin de que la aplicación que realiza la representación pueda acceder a ellos. Los archivos de salida los escribe la aplicación que realiza la representación en el disco de la máquina virtual y, después, se copian en Blob Storage.  Si es necesario, los archivos de salida se pueden descargar de Blob Storage a un sistema de archivos local.
  * Azure Blob Storage es una opción sencilla y rentable para proyectos pequeños.  Dado que todos los archivos de recursos son necesarios en cada máquina virtual del grupo, una vez que aumenta el número y tamaño de los archivos de recursos es preciso prestar más atención para asegurarse de que las transferencias de archivos sean tan eficaces como sea posible.  
* Azure Storage como sistema de archivos que usa [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md):
  * En el caso de las máquinas virtuales Linux, una cuenta de almacenamiento se puede exponer y usar como sistema de archivos cuando se utiliza el controlador del sistema de archivos virtual blobfuse.
  * Esta opción tiene la ventaja de ser muy rentable, ya que no se necesitan máquinas virtuales para el sistema de archivos, además de que el almacenamiento en caché de blobfuse en las máquinas virtuales evita tener que descargar en repetidas ocasiones los mismos archivos para varios trabajos y tareas.  El movimiento de datos también es sencillo, ya que los archivos son simplemente blobs y se pueden usar API y herramientas estándar, como azcopy, para copiar cualquier archivo entre un sistema de archivos local y Azure Storage.
* Sistema de archivos o recurso compartido de archivos:
  * En función del sistema operativo y de los requisitos de rendimiento y escalado de la máquina virtual, las opciones incluyen [Azure Files](../storage/files/storage-files-introduction.md), el uso de una máquina virtual con discos conectados para NFS, el uso de varias máquinas virtuales con discos conectados para un sistema de archivos distribuido como GlusterFS o el uso de una oferta de terceros.
  * [Avere Systems](https://www.averesystems.com/) ahora forma parte de Microsoft y en un futuro próximo tendrá soluciones ideales para la representación a gran escala y de alto rendimiento.  La solución de Avere permitirá crear una caché un NFS o SMB basada en Azure que funcione en conjunción con Blob Storage o con dispositivos NAS locales.
  * Con un sistema de archivos, los archivos se pueden leer o escribir directamente en el sistema de archivos, o bien se pueden copiar entre este y las máquinas virtuales del grupo.
  * Un sistema de archivos compartido permite utilizar un gran número de recursos compartidos entre proyectos y trabajos, y que las tareas de representación solo accedan a los que se requieren.

## <a name="using-azure-blob-storage"></a>Uso de Azure Blob Storage

Se deben usar una cuenta de Blob Storage o una cuenta de almacenamiento de uso general v2.  Estos dos tipos de cuenta de almacenamiento se pueden configurar con límites considerablemente mayores, en comparación con una cuenta de almacenamiento de uso general v1, como se detalla en [esta entrada del blog](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/).  Al configurarlas, los límites superiores mejorarán mucho el rendimiento y la escalabilidad, sobre todo cuando hay muchas máquinas virtuales del grupo que acceden a la cuenta de almacenamiento.

### <a name="copying-files-between-client-and-blob-storage"></a>Copia de archivos entre el cliente y Blob Storage

Para copiar archivos a Azure Storage, y desde él, se pueden usar diversos mecanismos, entre los que se incluyen la API de Blob Storage, la [Biblioteca de movimiento de datos de Azure Storage](https://github.com/Azure/azure-storage-net-data-movement), la herramienta de línea de comandos azcopy para [Windows](../storage/common/storage-use-azcopy-v10.md) o [Linux](../storage/common/storage-use-azcopy-v10.md), el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/), y [Azure Batch Explorer](https://azure.github.io/BatchExplorer/).

Por ejemplo, mediante azcopy, todos los recursos de una carpeta se pueden transferir como se indica a continuación:


`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /Y`

Para copiar solo los archivos modificados, se puede usar el parámetro /XO:

`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /XO /Y`

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copia de archivos de recursos de entrada desde Blob Storage a las máquinas virtuales del grupo de Batch

Hay un par de enfoques distintos para copiar archivos, y el mejor lo determina el tamaño de los recursos del trabajo.
El enfoque más sencillo es copiar todos los archivos de recursos a las máquinas virtuales del grupo para cada trabajo:

* Cuando haya archivos únicos para un trabajo, pero sean necesarios para todas las tareas de un trabajo, se puede especificar una [tarea de preparación del trabajo](/rest/api/batchservice/job/add#jobpreparationtask) para copiar todos los archivos.  La tarea de preparación del trabajo se ejecuta una vez cuando la primera tarea del trabajo se ejecuta en una máquina virtual, pero no en las tareas posteriores del trabajo.
* Se debe especificar una [tarea de liberación del trabajo](/rest/api/batchservice/job/add#jobreleasetask) para quitar los archivos por trabajo una vez que este se haya completado; así se evita que los archivos de recursos del trabajo llenen el disco de la máquina virtual.
* Cuando varios trabajos usen los mismos recursos y solo haya cambios incrementales en los recursos de cada trabajo se copiarán todos los archivos de recursos, aunque se haya actualizado solo un subconjunto.  Esto es ineficaz cuando hay muchos archivos de recursos de gran tamaño.

Si los archivos de recursos se reutilizan en distintos trabajos y solo se realizan cambios incrementales entre los trabajos, un enfoque más eficaz, aunque un poco más complicado, es almacenar los recursos en la carpeta compartida en la máquina virtual y sincronizar los archivos que han cambiado.

* La tarea de preparación del trabajo realizaría la copia mediante azcopy con el parámetro /XO en la carpeta compartida de la máquina virtual especificada por la variable de entorno AZ_BATCH_NODE_SHARED_DIR.  Esto solo copiará los archivos cambiados en cada máquina virtual.
* Habrá que pensar en el tamaño de todos los recursos para asegurarse de que encajan en la unidad temporal de las máquinas virtuales del grupo.

Azure Batch tiene compatibilidad integrada para copiar archivos entre una cuenta de almacenamiento y las máquinas virtuales del grupo de Batch.  Los [archivos de recursos](/rest/api/batchservice/job/add#resourcefile) de la tarea copian archivos de almacenamiento a las máquinas virtuales del grupo y se pueden especificar para la tarea de preparación del trabajo.  Lamentablemente, cuando hay cientos de archivos es posible alcanzar un límite y que ello produzca errores en las tareas.  Cuando hay un gran número de recursos se recomienda usar la herramienta de línea de comandos azcopy en la tarea de preparación del trabajo, ya que puede usar caracteres comodín y no tiene ningún límite.

### <a name="copying-output-files-to-blob-storage-from-batch-pool-vms"></a>Copia de archivos de salida en Blob Storage desde las máquinas virtuales del grupo de Batch

Los [archivos de salida](/rest/api/batchservice/task/add#outputfile) se pueden usar para copiar archivos de una máquina virtual de un grupo en el almacenamiento.  Uno o varios archivos se pueden copiar desde la máquina virtual a una cuenta de almacenamiento especificada cuando se haya completado la tarea.  Se debe copiar la salida representada, pero también se pueden almacenar los archivos de registro.

## <a name="using-a-blobfuse-virtual-file-system-for-linux-vm-pools"></a>Uso de un sistema de archivos virtual blobfuse para los grupos de máquinas virtuales Linux

Blobfuse es un controlador del sistema de archivos virtual para Azure Blob Storage, que permite acceder a los archivos almacenados como blobs en una cuenta de almacenamiento a través del sistema de archivos Linux.

Los nodos del grupo pueden montar el sistema de archivos al iniciarse o el montaje puede ocurrir como parte de una tarea de preparación del trabajo (una tarea que se ejecuta solo cuando la primera tarea de un trabajo se ejecuta en un nodo).  Blobfuse se puede configurar para sacar provecho tanto de un disco RAM como de la unidad de estado sólido local de las máquinas virtuales para almacenar archivos en la caché, lo que aumentarán considerablemente el rendimiento si varias tareas de un nodo acceden a algunos de los mismos archivos.

[Hay plantillas de ejemplo disponibles](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/vray/render-linux-with-blobfuse-mount) para ejecutar representaciones de V-Ray independientes mediante un sistema de archivos de blobfuse y puede usarse como base para las plantillas para otras aplicaciones.

### <a name="accessing-files"></a>Acceso a archivos

Las tareas del trabajo especifican las rutas de acceso a los archivos de entrada y de salida mediante el sistema de archivos montado.

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copia de archivos de recursos de entrada desde Blob Storage a las máquinas virtuales del grupo de Batch

Como los archivos son simplemente blobs en Azure Storage, las API de blob estándar, las herramientas y las interfaces de usuario se pueden utilizar para copiar los archivos entre un sistema de archivos local y Blob Storage; por ejemplo, azcopy, Explorador de Storage, Batch Explorer, etc.

## <a name="using-azure-files-with-windows-vms"></a>Uso de Azure Files con máquinas virtuales Windows

[Azure Files](../storage/files/storage-files-introduction.md) ofrece recursos compartidos de archivos en la nube totalmente administrados a los que se puede acceder mediante el protocolo SMB.  Se basa en Azure Blob Storage; es [rentable](https://azure.microsoft.com/pricing/details/storage/files/) y se puede configurar con la replicación de datos en otra región, así que es redundante de forma global.  [Escalar destinos](../storage/files/storage-files-scale-targets.md#azure-files-scale-targets) deben examinarse para determinar si se debe usar Azure Files dado el tamaño del grupo de previsión y el número de archivos de recursos.

Hay [documentación](../storage/files/storage-how-to-use-files-windows.md) que trata sobre cómo montar un recurso compartido de Azure Files.

### <a name="mounting-an-azure-files-share"></a>Montaje de un recurso compartido de Azure Files

Para usarlo en Batch, es preciso realizar una operación de montaje cada vez que se ejecuta una tarea, ya que no es posible conservar la conexión entre tareas.  La forma más fácil de hacerlo es utilizar cmdkey para conservar las credenciales mediante la tarea de inicio en la configuración del grupo y, después, montar el recurso compartido antes de cada tarea.

Ejemplo de uso de cmdkey en una plantilla de grupo (carácter de escape para su uso en un archivo JSON): tenga en cuenta que al separar la llamada cmdkey de la llamada de net use, el contexto de usuario para la tarea de inicio debe ser el mismo que se utiliza para ejecutar las tareas:

```
"startTask": {
  "commandLine": "cmdkey /add:storageaccountname.file.core.windows.net
    /user:AZURE\\markscuscusbatch /pass:storage_account_key",
  "userIdentity":{
    "autoUser": {
      "elevationLevel": "nonadmin",
      "scope": "pool"
    }
}
```

Línea de comandos de tarea de trabajo de ejemplo:
```
"commandLine":"net use S:
  \\\\storageaccountname.file.core.windows.net\\rendering &
3dsmaxcmdio.exe -v:5 -rfw:0 -10 -end:10
  -bitmapPath:\"s:\\3dsMax\\Dragon\\Assets\"
  -outputName:\"s:\\3dsMax\\Dragon\\RenderOutput\\dragon.jpg\"
  -w:1280 -h:720
  \"s:\\3dsMax\\Dragon\\Assets\\Dragon_Character_Rig.max\""
```

### <a name="accessing-files"></a>Acceso a archivos

Las tareas del trabajo especifican las rutas de acceso a los archivos de entrada y de salida mediante el sistema de archivos montado, mediante el uso de una asignada o de una ruta de acceso UNC.

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copia de archivos de recursos de entrada desde Blob Storage a las máquinas virtuales del grupo de Batch

Azure Files es compatibles con las principales API y herramientas que tienen compatibilidad con Azure Storage; por ejemplo, azcopy, CLI de Azure, el Explorador de Storage, Azure PowerShell, Batch Explorer, etc.

[Azure File Sync](../storage/file-sync/file-sync-planning.md) está disponible para sincronizar automáticamente archivos entre un sistema de archivos local y un recurso compartido de Azure Files.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las opciones de almacenamiento, consulte la documentación detallada:

* [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md)
* [Blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md)
* [Archivos de Azure](../storage/files/storage-files-introduction.md)
