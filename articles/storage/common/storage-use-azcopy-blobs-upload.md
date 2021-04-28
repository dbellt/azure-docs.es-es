---
title: Carga de archivos en Azure Blob Storage con AzCopy v10 | Microsoft Docs
description: Este artículo contiene una colección de comandos de ejemplo de AzCopy que le ayudarán a cargar archivos en Azure Blob Storage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 352497f0f4d23250abe9f84121f358589664002b
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502919"
---
# <a name="upload-files-to-azure-blob-storage-by-using-azcopy"></a>Carga de archivos en Azure Blob Storage con AzCopy

Puede cargar archivos y directorios en Blob Storage mediante la utilidad de línea de comandos AzCopy v10. 

Para ver ejemplos de otros tipos de tareas, como la carga de archivos, la sincronización con Blob Storage o la copia de blobs entre cuentas, consulte los vínculos presentados en la sección [Pasos siguientes](#next-steps) de este artículo.

## <a name="get-started"></a>Introducción

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y obtener información sobre las formas de proporcionar credenciales de autorización para el servicio de almacenamiento.

> [!NOTE] 
> En los ejemplos de este artículo se da por hecho que ha proporcionado credenciales de autorización mediante Azure Active Directory (Azure AD).
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy. Por ejemplo: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="create-a-container"></a>Crear un contenedor

Para crear un contenedor puede usar el comando [azcopy make](storage-ref-azcopy-make.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

Este es un ejemplo de tabla:

**Sintaxis**

`azcopy make 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>'`

**Ejemplo**

```azcopy
https://mystorageaccount.blob.core.windows.net/mycontainer
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
https://mystorageaccount.dfs.core.windows.net/mycontainer
```

Para obtener documentos de referencia detallados, consulte [azcopy make](storage-ref-azcopy-make.md).

## <a name="upload-a-file"></a>Cargar un archivo

Cargue un archivo mediante el comando [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'`

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt'
```

También puede cargar un archivo mediante un símbolo comodín (*) en cualquier lugar de la ruta de acceso o del nombre de archivo. Por ejemplo: `'C:\myDirectory\*.txt'` o `C:\my*\*.txt`.

## <a name="upload-a-directory"></a>Subir un directorio

Cargue un directorio mediante el comando [azcopy copy](storage-ref-azcopy-copy.md). 

En este ejemplo se copia un directorio (y todos sus archivos) en un contenedor de blobs. El resultado es un directorio en el contenedor con el mismo nombre.

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --recursive`

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --recursive
```

Para copiar un directorio dentro el contenedor, simplemente especifique el nombre de ese directorio en la cadena de comandos.

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --recursive
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' --recursive
```

Si especifica el nombre de un directorio que no existe en el contenedor, AzCopy crea un directorio con ese nombre.

## <a name="upload-directory-contents"></a>Carga del contenido del directorio

Cargue el contenido de un directorio mediante el comando [azcopy copy](storage-ref-azcopy-copy.md). Use el símbolo comodín (*) para cargar el contenido sin copiar el propio directorio que lo contiene.

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>'` 

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory'
```

Anexe la marca `--recursive` para cargar archivos en todos los subdirectorios.

## <a name="upload-specific-files"></a>Carga de archivos específicos

Puede cargar archivos específicos mediante nombres de archivo completos, nombres parciales con caracteres comodín (*) o fechas y horas.

> [!TIP]
> En estos ejemplos se delimitan los argumentos de ruta de acceso con comillas (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

### <a name="specify-multiple-complete-file-names"></a>Especificación de varios nombres de archivo completos

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-path`. Separe los nombres de archivo individuales mediante punto y coma (`;`).

**Sintaxis** 

`azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-path <semicolon-separated-file-list>` 

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive'
```

En este ejemplo, AzCopy transfiere el directorio `C:\myDirectory\photos` y el archivo `C:\myDirectory\documents\myFile.txt`. Incluya la opción `--recursive` para transferir todos los archivos del directorio `C:\myDirectory\photos`.

También puede excluir archivos mediante la opción `--exclude-path`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

### <a name="use-wildcard-characters"></a>Uso de caracteres comodín

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-pattern`. Especifique nombres parciales que incluyan los caracteres comodín. Separe los nombres con punto y coma (`;`). 

**Sintaxis**

`azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` 

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'
```

También puede excluir archivos mediante la opción `--exclude-pattern`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

Las opciones `--include-pattern` y `--exclude-pattern` solo se aplican a los nombres de archivo, no a la ruta de acceso.  Si quiere copiar todos los archivos de texto que existen en un árbol de directorios, use la opción `–recursive` para obtener todo el árbol de directorios y, a continuación, use el `–include-pattern` y especifique `*.txt` para obtener todos los archivos de texto.

### <a name="upload-files-that-were-modified-before-or-after-a-date-and-time"></a>Carga de archivos modificados antes o después de una fecha y hora 

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-before` o `--include-after`. Especifique una fecha y hora en formato ISO-8601 (por ejemplo: `2020-08-19T15:04:00Z`). 

En los siguientes ejemplos se cargan archivos que se modificaron en la fecha especificada o después de esta.

**Sintaxis**

`azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>'  --include-after <Date-Time-in-ISO-8601-format>` 

**Ejemplo**

```azcopy
azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory'  --include-after '2020-08-19T15:04:00Z'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory'   --include-after '2020-08-19T15:04:00Z'
```

Para ver una referencia detallada, consulte la documentación de referencia de [azcopy copy](storage-ref-azcopy-copy.md).

## <a name="upload-with-index-tags"></a>Carga con etiquetas de índice

Puede cargar un archivo y agregar [etiquetas de índice de blob (versión preliminar)](../blobs/storage-manage-find-blobs.md) al blob de destino.  

Si usa la autorización de Azure AD, a la entidad de seguridad se le debe asignar el rol [Propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o se le debe conceder permiso para la `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [operación del proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) a través de un rol personalizado de Azure. Si usa un token de firma de acceso compartido (SAS), ese token debe proporcionar acceso a las etiquetas del blob mediante el permiso `t` de SAS.

Para agregar etiquetas, use la opción `--blob-tags` junto con un par clave-valor codificado como dirección URL. Por ejemplo, para agregar la clave `my tag` y un valor `my tag value`, agregaría `--blob-tags='my%20tag=my%20tag%20value'` al parámetro de destino. 

Separe varias etiquetas de índice mediante el carácter de Y comercial (`&`).  Por ejemplo, si quiere agregar una clave `my second tag` y un valor `my second tag value`, la cadena de opción completa sería `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`.

En el siguiente ejemplo, se muestra cómo usar la opción `--blob-tags`.

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Carga de un archivo**

```azcopy
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

**Subir un directorio**

```azcopy
azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

**Carga del contenido del directorio**

```azcopy
azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

> [!NOTE]
> Si especifica un directorio para el origen, todos los blobs que se copien en el destino tendrán las mismas etiquetas que especifique en el comando.

## <a name="upload-with-optional-flags"></a>Carga con marcas opcionales

Puede modificar las operaciones de carga mediante marcas opcionales. Estos son algunos ejemplos.

|Escenario|Marca|
|---|---|
|Cargar los archivos como blobs en anexos o blobs en páginas.|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
|Cargar en un nivel de acceso específico (como el nivel de archivo).|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|

Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Pasos siguientes

Encuentre más ejemplos en estos artículos:

- [Ejemplos: descarga](storage-use-azcopy-blobs-download.md)
- [Ejemplos: Copia entre cuentas](storage-use-azcopy-blobs-copy.md)
- [Ejemplos: Sincronización](storage-use-azcopy-blobs-synchronize.md)
- [Ejemplos: Cubos de Amazon S3](storage-use-azcopy-s3.md)
- [Ejemplos: Google Cloud Storage](storage-use-azcopy-google-cloud.md)
- [Ejemplos: Azure Files](storage-use-azcopy-files.md)
- [Tutorial: Migración de datos locales al almacenamiento en la nube mediante AzCopy](storage-use-azcopy-migrate-on-premises-data.md)

Consulte estos artículos para configurar opciones, optimizar el rendimiento y solucionar problemas:

- [Parámetros de configuración de AzCopy](storage-ref-azcopy-configuration-settings.md)
- [Optimización del rendimiento de AzCopy](storage-use-azcopy-optimize.md)
- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)