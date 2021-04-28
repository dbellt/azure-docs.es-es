---
title: Actualización de los destinos de almacenamiento en Azure HPC Cache
description: Procedimientos para editar los destinos de almacenamiento en Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 03/29/2021
ms.author: v-erkel
ms.openlocfilehash: ebf68c1eb06984e2de8114c53e1bb55d52aed70a
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107862640"
---
# <a name="edit-storage-targets"></a>Edición de los destinos de almacenamiento

Puede quitar o modificar destinos de almacenamiento con Azure Portal o la CLI de Azure.

En función del tipo de almacenamiento, puede modificar estos valores de destino de almacenamiento:

* En el caso de los destinos de almacenamiento de blobs, se puede cambiar la ruta del espacio de nombres y la directiva de acceso.

* En el caso de los destinos de almacenamiento de NFS, se pueden cambiar estos valores:

  * Rutas de acceso del espacio de nombres
  * Directiva de acceso
  * El subdirectorio de exportación o la exportación de almacenamiento asociados a una ruta de acceso del espacio de nombres
  * Modelo de uso

* En el caso de los destinos de almacenamiento de ADLS-NFS, puede cambiar la ruta de acceso del espacio de nombres, la directiva de acceso y el modelo de uso.

No se puede editar el nombre, el tipo o el sistema de almacenamiento de back-end de un destino de almacenamiento (contenedor de blobs, o dirección IP o nombre de host de NFS). Si tiene que cambiar estas propiedades, elimine el destino de almacenamiento y cree un reemplazo con el valor nuevo.

> [!TIP]
> En el [vídeo de administración de Azure HPC Cache](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) se muestra cómo editar un destino de almacenamiento en Azure Portal.

## <a name="remove-a-storage-target"></a>Eliminación de un destino de almacenamiento

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para quitar un destino de almacenamiento, abra la página **Destinos de almacenamiento**. Seleccione el destino de almacenamiento en la lista y haga clic en el botón **Eliminar**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Use [az hpc-cache storage-target remove](/cli/azure/hpc-cache/storage-target#az_hpc_cache_storage_target_remove) para eliminar un destino de almacenamiento de la memoria caché.

```azurecli
$ az hpc-cache storage-target remove --resource-group cache-rg --cache-name doc-cache0629 --name blob1

{- Finished ..
  "endTime": "2020-07-09T21:45:06.1631571+00:00",
  "name": "2f95eac1-aded-4860-b19c-3f089531a7ec",
  "startTime": "2020-07-09T21:43:38.5461495+00:00",
  "status": "Succeeded"
}
```

---

Esta acción quita la asociación del destino de almacenamiento con este sistema de Azure HPC Cache, pero no cambia el sistema de almacenamiento de back-end. Por ejemplo, si ha usado un contenedor Azure Blob Storage, el contenedor y su contenido siguen existiendo después de eliminarlo de la memoria caché. Puede agregar el contenedor a otra instancia de Azure HPC Cache, volver a agregarlo a esta caché o eliminarlo con Azure Portal.

Los cambios de archivo almacenados en la caché se escriben en el sistema de almacenamiento de back-end antes de que se quite el destino de almacenamiento. Este proceso puede tardar una hora o más si hay muchos datos cambiados en la caché.

## <a name="change-a-blob-storage-targets-namespace-path"></a>Cambio de la ruta de acceso del espacio de nombres de un destino de almacenamiento de blobs

Las rutas de acceso de espacios de nombres son las rutas de acceso que los clientes usan para montar este destino de almacenamiento. Para obtener más información, lea [Planeamiento del espacio de nombres agregado](hpc-cache-namespace.md) y [Configuración del espacio de nombres agregado](add-namespace-paths.md).

La ruta de acceso del espacio de nombres es la única actualización que se puede hacer en un destino de almacenamiento de blobs de Azure. Puede usar Azure Portal o la CLI de Azure para cambiarla.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Use la página **Espacio de nombres** de su instancia de Azure HPC Cache. La página del espacio de nombres se describe con más detalle en el artículo [Configuración del espacio de nombres agregado](add-namespace-paths.md).

Haga clic en el nombre de la ruta de acceso que desea cambiar y cree la nueva ruta de acceso en la ventana de edición que aparece.

![Captura de pantalla de la página del espacio de nombres después de hacer clic en una ruta de acceso del espacio de nombres del blob: los campos de edición aparecen en un panel a la derecha.](media/update-namespace-blob.png)

Después de realizar los cambios, haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Para cambiar el espacio de nombres de un destino de almacenamiento de blobs con la CLI de Azure, use el comando [az hpc-cache blob-storage-target update](/cli/azure/hpc-cache/blob-storage-target#az_hpc_cache_blob_storage_target_update). Solo se puede cambiar el valor de `--virtual-namespace-path`.

  ```azurecli
  az hpc-cache blob-storage-target update --cache-name cache-name --name target-name \
    --resource-group rg --virtual-namespace-path "/new-path"
  ```

---

## <a name="update-an-nfs-storage-target"></a>Actualización de un destino de almacenamiento de NFS

En el caso de los destinos de almacenamiento de NFS, puede cambiar o agregar rutas de acceso de espacios de nombres virtuales, cambiar los valores de exportación o el subdirectorio de NFS al que apunta una ruta de acceso de espacio de nombres, y cambiar el modelo de uso.

Los destinos de almacenamiento en cachés con algunos tipos de valores de DNS personalizados también tienen un control para actualizar sus direcciones IP. (Este tipo de configuración es poco frecuente).

A continuación se muestran los detalles:

* [Cambio de los valores del espacio de nombres agregado](#change-aggregated-namespace-values) (ruta de acceso del espacio de nombres virtual, de exportación y del subdirectorio de exportación)
* [Cambiar el modelo de uso](#change-the-usage-model)
* [Actualización de DNS](#update-ip-address-custom-dns-configurations-only)

### <a name="change-aggregated-namespace-values"></a>Cambio de los valores del espacio de nombres agregado

Puede usar Azure Portal o la CLI de Azure para cambiar la ruta de acceso del espacio de nombres orientado al cliente, la exportación del almacenamiento y el subdirectorio de exportación (si se usa).

Lea las instrucciones en [Incorporación de rutas de acceso del espacio de nombres de NFS](add-namespace-paths.md#nfs-namespace-paths) si necesita recordar cómo crear varias rutas válidas en un destino de almacenamiento.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Use la página **Espacio de nombres** de su instancia de Azure HPC Cache para actualizar los valores de espacio de nombres. Esta página se describe con más detalle en el artículo [Configuración del espacio de nombres agregado](add-namespace-paths.md).

![Captura de pantalla de la página del espacio de nombres del portal con la página de actualización de NFS abierta a la derecha](media/update-namespace-nfs.png)

1. Haga clic en el nombre de la ruta de acceso que desee cambiar.
1. Utilice la ventana de edición para escribir nuevos valores de ruta de acceso virtual, de exportación o de subdirectorio, o para seleccionar una directiva de acceso diferente.
1. Después de realizar los cambios, haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Use la opción ``--junction`` del comando [az hpc-cache nfs-storage-target update](/cli/azure/hpc-cache/nfs-storage-target) para cambiar la ruta de acceso del espacio de nombres, la exportación de NFS o el subdirectorio de exportación.

El parámetro ``--junction`` usa estos valores:

* ``namespace-path``: Ruta de acceso del archivo virtual orientado al cliente
* ``nfs-export``: Exportación del sistema de almacenamiento que se va a asociar a la ruta de acceso orientada al cliente
* ``target-path`` (opcional): Subdirectorio de la exportación, si es necesario

Ejemplo: ``--junction namespace-path="/nas-1" nfs-export="/datadisk1" target-path="/test"``

Debe proporcionar los tres valores para cada ruta de acceso en la instrucción ``--junction``. Use los valores existentes en los valores que no quiera cambiar.

El nombre de la memoria caché, el nombre del destino de almacenamiento y el grupo de recursos también son obligatorios en todos los comandos de actualización.

Comando de ejemplo:

```azurecli
az hpc-cache nfs-storage-target update --cache-name mycache \
  --name st-name --resource-group doc-rg0619 \
  --junction namespace-path="/new-path" nfs-export="/my-export" target-path="my-subdirectory"
```

---

### <a name="change-the-usage-model"></a>Cambio del modelo de uso

El modelo de uso influye en el modo en que la memoria caché conserva los datos. Consulte [Descripción de los modelos de uso de caché](cache-usage-models.md) para más información.

> [!NOTE]
> Si cambia los modelos de uso, es posible que tenga que volver a montar los clientes para evitar errores de NLM. Consulte [Cuándo volver a montar los clientes para NLM](cache-usage-models.md#know-when-to-remount-clients-for-nlm) para más información.

Para cambiar el modelo de uso de un destino de almacenamiento de NFS, use uno de estos métodos.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Cambie el modelo de uso desde la página **Destinos de almacenamiento** en Azure Portal. Haga clic en el nombre del destino de almacenamiento que se va a cambiar.

![captura de pantalla de la página de edición de un destino de almacenamiento de NFS](media/edit-storage-nfs.png)

Use el selector desplegable para elegir un nuevo modelo de uso. Haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[Configurar la CLI de Azure para Azure HPC Cache](./az-cli-prerequisites.md).

Use el comando [az hpc-cache nfs-storage-target update](/cli/azure/hpc-cache/nfs-storage-target#az_hpc_cache_nfs_storage_target_update).

El comando update es casi idéntico al comando que se usa para agregar un destino de almacenamiento de NFS. Consulte [Creación de un destino de almacenamiento de NFS](hpc-cache-add-storage.md#create-an-nfs-storage-target) para obtener más información y ejemplos.

Para cambiar el modelo de uso, actualice la opción ``--nfs3-usage-model``. Ejemplo: ``--nfs3-usage-model WRITE_WORKLOAD_15``

El nombre de la memoria caché, el nombre del destino de almacenamiento y el grupo de recursos son obligatorios.

Si quiere, compruebe los nombres de los modelos de uso con el comando [az hpc-cache usage-model list](/cli/azure/hpc-cache/usage-model#az_hpc_cache_usage-model-list).

Si la memoria caché se ha detenido o no tiene un estado correcto, la actualización se aplicará después de que la memoria caché vuelva al estado correcto.

---

### <a name="update-ip-address-custom-dns-configurations-only"></a>Actualización de la dirección IP (solo configuraciones DNS personalizadas)

Si la memoria caché usa una configuración de DNS no predeterminada, es posible que la dirección IP de su destino de almacenamiento de NFS cambie debido a los cambios de DNS de back-end. Si el servidor DNS cambia la dirección IP del sistema de almacenamiento de back-end, Azure HPC Cache puede perder el acceso al sistema de almacenamiento.

Idealmente, debe trabajar con el administrador del sistema DNS personalizado de la memoria caché para planear las actualizaciones, ya que estos cambios hacen que el almacenamiento no esté disponible.

Si necesita actualizar la dirección IP proporcionada por DNS de un destino de almacenamiento, hay un botón en la lista de destinos de almacenamiento. Haga clic en **Actualizar DNS** para consultar el servidor DNS personalizado para obtener una nueva dirección IP.

![Captura de pantalla de la lista de destinos de almacenamiento Para un destino de almacenamiento, el menú "..." de la columna de la derecha está abierto y aparecen dos opciones: eliminar y actualizar DNS.](media/refresh-dns.png)

Si se realiza correctamente, la actualización debe tardar menos de dos minutos. Solo puede actualizar un destino de almacenamiento a la vez; Espere a que se complete la operación anterior antes de intentar otra.

## <a name="update-an-adls-nfs-storage-target-preview"></a>Actualización de un destino de almacenamiento de ADLS-NFS (versión preliminar)

De forma similar a los destinos de NFS, puede cambiar la ruta de acceso del espacio de nombres y el modelo de uso para los destinos de almacenamiento de ADLS-NFS.

### <a name="change-an-adls-nfs-namespace-path"></a>Cambio de la ruta de acceso de un espacio de nombres de ADLS-NFS

Use la página **Espacio de nombres** de su instancia de Azure HPC Cache para actualizar los valores de espacio de nombres. Esta página se describe con más detalle en el artículo [Configuración del espacio de nombres agregado](add-namespace-paths.md).

![Captura de pantalla de la página del espacio de nombres del portal con la página de actualización de NFS abierta a la derecha](media/update-namespace-adls.png)

1. Haga clic en el nombre de la ruta de acceso que desee cambiar.
1. Utilice la ventana de edición para escribir nuevas rutas de acceso virtual o para actualizar la directiva de acceso.
1. Después de realizar los cambios, haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="change-adls-nfs-usage-models"></a>Cambio de los modelos de uso de ADLS-NFS

La configuración de los modelos de uso de ADLS-NFS es idéntica a la selección del modelo de uso de NFS. Lea las instrucciones del portal en [Cambio del modelo de uso](#change-the-usage-model) en la sección de NFS anterior. Hay herramientas adicionales para actualizar los destinos de almacenamiento de ADLS-NFS en desarrollo.


## <a name="next-steps"></a>Pasos siguientes

* Lea [Incorporación de destinos de almacenamiento](hpc-cache-add-storage.md) para obtener más información sobre estas opciones.
* Lea [Planeamiento del espacio de nombres agregado](hpc-cache-namespace.md) para obtener más recomendaciones sobre el uso de rutas de acceso virtuales.
