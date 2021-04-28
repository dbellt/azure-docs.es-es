---
title: Configuración del espacio de nombres agregado de Azure HPC Cache
description: Creación de rutas de acceso orientadas al cliente para el almacenamiento de back-end con Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 04/22/2021
ms.author: v-erkel
ms.openlocfilehash: 3eddd53d8b1d4ff59f27535a070924cf0e86c5a7
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904422"
---
# <a name="set-up-the-aggregated-namespace"></a>Configuración del espacio de nombres agregado

Después de crear destinos de almacenamiento, también debe crear rutas de acceso de espacio de nombres para ellos. Las máquinas cliente usan estas rutas de acceso virtuales para acceder a los archivos mediante la memoria caché en lugar de conectarse directamente al almacenamiento de back-end. Este sistema permite que los administradores de caché cambien los sistemas de almacenamiento de back-end sin tener que volver a escribir las instrucciones del cliente.

Lea [Planeamiento del espacio de nombres agregado](hpc-cache-namespace.md) para más información acerca de esta característica.

La página **Namespace** (Espacio de nombres) de Azure Portal muestra las rutas de acceso que los clientes usan para acceder a los datos a través de la caché. Use esta página para crear, quitar o cambiar las rutas de acceso del espacio de nombres. También puede configurar estas rutas mediante la CLI de Azure.

Todas las rutas de acceso orientadas al cliente que se han definido para esta caché aparecen en la página **Espacio de nombres**. Los destinos de almacenamiento que no tienen todavía ninguna ruta de acceso del espacio de nombres definida no aparecen en la tabla.

Puede ordenar las columnas de la tabla para entender mejor el espacio de nombres agregado de la caché. Haga clic en las flechas de los encabezados de columna para ordenar las rutas de acceso.

[![Captura de pantalla de la página de espacio de nombres del portal con dos rutas de acceso en una tabla. Encabezados de columna: Namespace path, Storage target, Export path, Export subdirectory y Client Access Policy (Ruta de acceso del espacio de nombres, Destino de almacenamiento, Ruta de exportación, Subdirectorio de exportación y Directiva de acceso de cliente). Los nombres de rutas de acceso de la primera columna son vínculos en los que se puede hacer clic. Botones superiores: Add namespace path, Refresh, Delete (Agregar ruta de espacio de nombres, Actualizar, Eliminar)](media/namespace-page.png) ](media/namespace-page.png#lightbox)

## <a name="add-or-edit-namespace-paths"></a>Incorporación o edición de rutas de acceso de espacio de nombres

Debe crear al menos una ruta de acceso de espacio de nombres para que los clientes puedan acceder al destino de almacenamiento. Lea [Montaje de la instancia de Azure HPC Cache](hpc-cache-mount.md) para obtener más información sobre el acceso de los clientes.

Si agregó recientemente un destino de almacenamiento o personalizó una directiva de acceso, es posible que tarde un minuto o dos antes de poder crear una ruta de acceso de espacio de nombres.

### <a name="blob-namespace-paths"></a>Rutas de acceso del espacio de nombres de blobs

Un destino de almacenamiento de Azure Blob Storage solo puede tener una ruta de acceso de espacio de nombres.

Siga las instrucciones que se incluyen a continuación para establecer o cambiar la ruta de acceso con Azure Portal o la CLI de Azure.

### <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, cargue la página de configuración **Namespace** (Espacio de nombres). Puede agregar, cambiar o eliminar rutas de acceso de espacio de nombres desde esta página.

* **Agregar una nueva ruta de acceso**: haga clic en el botón **+ Add** (Agregar) en la parte superior y rellene la información en el panel de edición.

  ![Captura de pantalla de los campos de edición de Agregar espacio de nombres con un destino de almacenamiento de blobs seleccionado. Las rutas de acceso de exportación y subdirectorios están establecidos como "/" y no se pueden editar.](media/namespace-add-blob.png)

  * Escriba la ruta de acceso que usarán los clientes para acceder a este destino de almacenamiento.

  * Seleccione la directiva de acceso que se va a usar para esta ruta. Obtenga más información sobre cómo personalizar el acceso de cliente en [Uso de directivas de acceso de cliente](access-policies.md).

  * Seleccione el destino de almacenamiento en la lista desplegable. Si un destino de almacenamiento de blobs ya tiene una ruta de acceso de espacio de nombres, no se puede seleccionar.

  * En un destino de almacenamiento de Azure Blob Storage, las rutas de acceso de exportación y subdirectorio se establecen automáticamente en ``/``.

* **Cambiar una ruta de acceso existente**: haga clic en la ruta de acceso del espacio de nombres. Se abre el panel de edición. Puede modificar la ruta de acceso y la directiva de acceso, pero no puede cambiar a un destino de almacenamiento diferente.

* **Eliminar un espacio de nombres**: active la casilla situada a la izquierda de la ruta de acceso y haga clic en el botón **Delete** (Eliminar).

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Cuando usa la CLI de Azure, debe agregar una ruta de acceso de espacio de nombres al crear el destino de almacenamiento. Lea [Incorporación de un nuevo destino de almacenamiento de Azure Blob Storage](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-azure-blob-storage-target) para obtener más información.

Para actualizar la ruta de acceso del espacio de nombres del destino, use el comando [az hpc-cache blob-storage-target update](/cli/azure/hpc-cache/blob-storage-target#az_hpc_cache_blob_storage_target_update). Los argumentos del comando "update" son similares a los del comando "create", salvo que no se pasa el nombre del contenedor ni la cuenta de almacenamiento.

No se puede eliminar una ruta de acceso de espacio de nombres de un destino de almacenamiento de blobs con la CLI de Azure, pero se puede sobrescribir con otro valor.

---

### <a name="nfs-namespace-paths"></a>Rutas de acceso del espacio de nombres NFS

Un destino de almacenamiento NFS puede tener varias rutas de acceso virtuales, siempre y cuando cada ruta represente una exportación o un subdirectorio diferente sen el mismo sistema de almacenamiento.

Al planear el espacio de nombres para un destino de almacenamiento NFS, tenga en cuenta que cada ruta de acceso debe ser única y no puede ser un subdirectorio de otra ruta de acceso de espacio de nombres. Por ejemplo, si tiene una ruta de acceso de espacio de nombres que se llama ``/parent-a``, no puede crear además rutas como ``/parent-a/user1`` ni ``/parent-a/user2``. Estas rutas de acceso de directorio ya son accesibles en el espacio de nombres como subdirectorios de ``/parent-a``.

Todas las rutas de acceso de espacio de nombres para un sistema de almacenamiento NFS se crean en un solo destino de almacenamiento.

Para cada ruta de acceso del espacio de nombres de NFS, proporcione la ruta de acceso orientada al cliente, la exportación del sistema de almacenamiento y, opcionalmente, un subdirectorio de exportación.

### <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, cargue la página de configuración **Namespace** (Espacio de nombres). Puede agregar, editar o eliminar rutas de acceso de espacio de nombres desde esta página.

* **Para agregar una nueva ruta de acceso**: haga clic en el botón **+ Add** (Agregar) en la parte superior y rellene la información en el panel de edición.
* **Para cambiar una ruta de acceso existente**: haga clic en la ruta de acceso del espacio de nombres. Se abre el panel de edición, donde puede modificar la ruta de acceso.
* **Para eliminar una ruta de acceso de espacio de nombres**: active la casilla situada a la izquierda de la ruta de acceso y haga clic en el botón **Delete** (Eliminar).

Rellene estos valores para cada ruta de acceso del espacio de nombres:

* **Namespace path** (Ruta de acceso del espacio de nombres): la ruta de acceso del archivo orientada al cliente.

* **Directiva de acceso de cliente**: seleccione la directiva de acceso que se va a usar para esta ruta. Obtenga más información sobre cómo personalizar el acceso de cliente en [Uso de directivas de acceso de cliente](access-policies.md).

* **Storage target** (Destino de almacenamiento): si crea una nueva ruta de acceso de espacio de nombres, seleccione un destino de almacenamiento en el menú desplegable.

* **Export path** (Ruta de acceso de exportación): escriba la ruta de acceso a la exportación de NFS. Asegúrese de escribir el nombre de exportación correctamente, ya que el portal valida la sintaxis de este campo, pero no comprueba la exportación hasta que se envía el cambio.

* **Export subdirectory** (Subdirectorio de exportación): si desea que esta ruta de acceso monte un subdirectorio específico de la exportación, indíquelo aquí. En caso contrario, deje este campo en blanco.

![Captura de pantalla de la página de espacio de nombres del portal con la página de edición abierta a la derecha. El formulario de edición muestra la configuración de la ruta de acceso de un espacio de nombres de destino de almacenamiento de NFS](media/namespace-edit-nfs.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Cuando usa la CLI de Azure, debe agregar al menos una ruta de acceso de espacio de nombres al crear el destino de almacenamiento. Lea [Incorporación de un nuevo destino de almacenamiento NFS](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target) para obtener más información.

Para actualizar la ruta de acceso del espacio de nombres del destino o para agregar rutas adicionales, use el comando [az hpc-cache nfs-storage-target update](/cli/azure/hpc-cache/nfs-storage-target#az_hpc_cache_nfs_storage_target_update). Use la opción ``--junction`` para especificar todas las rutas de acceso de espacio de nombres que desee.

Las opciones que se usan en el comando "update" son similares al comando "create", salvo que no se pasa la información del sistema de almacenamiento (dirección IP o nombre de host) y el modelo de uso es opcional. Lea [Incorporación de un nuevo destino de almacenamiento NFS](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target) para obtener más detalles sobre la sintaxis de la opción ``--junction``.

---

### <a name="adls-nfs-namespace-paths-preview"></a>Rutas de acceso de espacio de nombres de ADLS-NFS (versión preliminar)

Al igual que un destino de almacenamiento de blobs normal, un destino de almacenamiento de ADLS-NFS solo tiene una exportación, por lo que solo puede tener una ruta de acceso de espacio de nombres.

Siga las instrucciones que se incluyen a continuación para establecer o cambiar la ruta de acceso con Azure Portal.

Cargue la página de configuración del **espacio de nombres**.

* **Agregar una nueva ruta de acceso**: haga clic en el botón **+ Add** (Agregar) en la parte superior y rellene la información en el panel de edición.

  ![Captura de pantalla de los campos de edición de Agregar espacio de nombres con un destino de almacenamiento de ADLS-NFS seleccionado. Las rutas de acceso de exportación y subdirectorios están establecidos como "/" y no se pueden editar.](media/namespace-add-adls.png)

  * Escriba la ruta de acceso que usarán los clientes para acceder a este destino de almacenamiento.

  * Seleccione la directiva de acceso que se va a usar para esta ruta. Obtenga más información sobre cómo personalizar el acceso de cliente en [Uso de directivas de acceso de cliente](access-policies.md).

  * Seleccione el destino de almacenamiento en la lista desplegable. Si un destino de almacenamiento de ADLS-NFS ya tiene una ruta de acceso de espacio de nombres, no se puede seleccionar.

  * En un destino de almacenamiento de ADLS-NFS, las rutas de acceso de exportación y subdirectorio se establecen automáticamente en ``/``.

* **Cambiar una ruta de acceso existente**: haga clic en la ruta de acceso del espacio de nombres. Se abre el panel de edición. Puede modificar la ruta de acceso y la directiva de acceso, pero no puede cambiar a un destino de almacenamiento diferente.

* **Eliminar un espacio de nombres**: active la casilla situada a la izquierda de la ruta de acceso y haga clic en el botón **Delete** (Eliminar).

## <a name="next-steps"></a>Pasos siguientes

Después de crear el espacio de nombres agregado para los destinos de almacenamiento, puede montar los clientes en la memoria caché. Para obtener más información, vea estos artículos:

* [Montaje de la instancia de Azure HPC Cache](hpc-cache-mount.md)
* [Traslado de datos a Azure Blob Storage](hpc-cache-ingest.md)
