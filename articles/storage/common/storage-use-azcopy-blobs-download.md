---
title: Descarga de blobs de Azure Blob Storage mediante AzCopy v10 | Microsoft Docs
description: Este artículo contiene una colección de comandos de ejemplo de AzCopy que le ayudarán a descargar blobs de Azure Blob Storage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 56bb36cfda9d0cf1a8882950c862a73ad1e77898
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502953"
---
# <a name="download-blobs-from-azure-blob-storage-by-using-azcopy"></a>Descarga de blobs de Azure Blob Storage mediante AzCopy

Puede descargar blobs y directorios de Blob Storage mediante la utilidad de línea de comandos AzCopy v10. 

Para ver ejemplos de otros tipos de tareas, como la carga de archivos, la sincronización con Blob Storage o la copia de blobs entre cuentas, consulte los vínculos presentados en la sección [Pasos siguientes](#next-steps) de este artículo.

## <a name="get-started"></a>Introducción

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y obtener información sobre las formas de proporcionar credenciales de autorización para el servicio de almacenamiento.

> [!NOTE] 
> En los ejemplos de este artículo se da por hecho que ha proporcionado credenciales de autorización mediante Azure Active Directory (Azure AD).
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy. Por ejemplo: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="download-a-blob"></a>Descarga de un blob

Descargue un blob mediante el comando [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

``azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'``

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'
```

> [!NOTE]
> Si el valor de la propiedad `Content-md5` de un blob contiene un hash, AzCopy calcula un hash MD5 para los datos descargados y comprueba que el hash MD5 almacenado en la propiedad `Content-md5` del blob coincide con el hash calculado. Si estos valores no coinciden, se produce un error en la descarga a menos que invalide este comportamiento mediante la anexión de `--check-md5=NoCheck` o `--check-md5=LogOnly` al comando de copia.

## <a name="download-a-directory"></a>Descargar un directorio

Descargue un directorio mediante el comando [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>' '<local-directory-path>' --recursive`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive
```

En este ejemplo se crea un directorio denominado `C:\myDirectory\myBlobDirectory` que contiene todos los blobs descargados.

## <a name="download-directory-contents"></a>Descarga del contenido del directorio

Puede descargar el contenido de un directorio sin copiar el propio directorio contenedor mediante el carácter comodín (*).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

> [!NOTE]
> En la actualidad, este escenario solo se admite para las cuentas que no tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.blob.core.windows.net/<container-name>/*' '<local-directory-path>/'`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory/*' 'C:\myDirectory'
```

Anexe la marca `--recursive` para descargar archivos en todos los subdirectorios.

## <a name="download-specific-blobs"></a>Descarga de blobs específicos

Puede descargar blobs específicos mediante nombres de archivo completos, nombres parciales con caracteres comodín (*) o fechas y horas. 

> [!TIP]
> En estos ejemplos se delimitan los argumentos de ruta de acceso con comillas (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

#### <a name="specify-multiple-complete-blob-names"></a>Especificación de varios nombres de blob completos

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-path`. Separe los nombres de blob individuales con punto y coma (`;`).

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt'--recursive
```

En este ejemplo, AzCopy transfiere el directorio `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` y el archivo `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/documents/myFile.txt`. Incluya la opción `--recursive` para transferir todos los blobs del directorio `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos`.

También puede excluir blobs mediante la opción `--exclude-path`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

#### <a name="use-wildcard-characters"></a>Uso de caracteres comodín

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-pattern`. Especifique nombres parciales que incluyan los caracteres comodín. Separe los nombres con punto y coma (`;`).

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'
```

También puede excluir blobs mediante la opción `--exclude-pattern`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

Las opciones `--include-pattern` y `--exclude-pattern` solo se aplican a los nombres de blob, no a la ruta de acceso.  Si quiere copiar todos los archivos de texto (blobs) que existen en un árbol de directorios, use la opción `–recursive` para obtener todo el árbol de directorios y, luego, use `–include-pattern` y especifique `*.txt` para obtener todos los archivos de texto.

#### <a name="download-blobs-that-were-modified-before-or-after-a-date-and-time"></a>Descarga de blobs modificados antes o después de una fecha y hora 

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-before` o `--include-after`. Especifique una fecha y hora en formato ISO-8601 (por ejemplo: `2020-08-19T15:04:00Z`). 

En los siguientes ejemplos se descargan archivos que se modificaron en la fecha especificada o después de esta.

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>/*' '<local-directory-path>' --include-after <Date-Time-in-ISO-8601-format>`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'
```

Para ver una referencia detallada, consulte la documentación de referencia de [azcopy copy](storage-ref-azcopy-copy.md).

#### <a name="download-previous-versions-of-a-blob"></a>Descarga de versiones anteriores de un blob

Si ha habilitado el [control de versiones de blobs](../blobs/versioning-enable.md), puede descargar una o más versiones anteriores de un blob. 

En primer lugar, cree un archivo de texto que contenga una lista de [identificadores de versión](../blobs/versioning-overview.md). Cada identificador de versión debe aparecer en una línea independiente. Por ejemplo: 

```
2020-08-17T05:50:34.2199403Z
2020-08-17T05:50:34.5041365Z
2020-08-17T05:50:36.7607103Z
```

Después, utilice el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--list-of-versions`. Especifique la ubicación del archivo de texto que contiene la lista de versiones (por ejemplo: `D:\\list-of-versions.txt`).  

#### <a name="download-a-blob-snapshot"></a>Descarga de una instantánea de un blob

Puede descargar una [instantánea de un blob](../blobs/snapshots-overview.md) haciendo referencia al valor **DateTime** de la misma. 

**Sintaxis**

`azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>?sharesnapshot=<DateTime-of-snapshot>' '<local-file-path>'`

**Ejemplo**

```azcopy
azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'
```

**Ejemplo (espacio de nombres jerárquico)**

```azcopy
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'
```

> [!NOTE]
> Si usa un token de SAS para autorizar el acceso a los datos de un blob, anexe el valor **DateTime** de la instantánea después del token de SAS. Por ejemplo: `'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z'`.

## <a name="download-with-optional-flags"></a>Descarga con marcas opcionales

Puede modificar las operaciones de descarga mediante marcas opcionales. Estos son algunos ejemplos.

|Escenario|Marca|
|---|---|
|Descomprimir archivos automáticamente.|**DECOMPRESS**|
|Especifique el grado de detalles que quiere que sean las entradas de registro relacionadas con la copia.|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|
|Especifique si desea sobrescribir los archivos y blobs en conflicto en el destino, y cómo hacerlo.|**--overwrite**=\[true\|false\|ifSourceNewer\|prompt\]|

Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Pasos siguientes

Encuentre más ejemplos en estos artículos:

- [Ejemplos: Carga](storage-use-azcopy-blobs-upload.md)
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
