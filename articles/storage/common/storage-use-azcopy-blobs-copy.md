---
title: Copia de blobs entre cuentas de almacenamiento de Azure con AzCopy v10 | Microsoft Docs
description: Este artículo contiene una colección de comandos de ejemplo de AzCopy que le ayudan a copiar blobs entre cuentas de almacenamiento.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 980c4da25e681b8bb2fb7a608481160a2c0857f2
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854565"
---
# <a name="copy-blobs-between-azure-storage-accounts-by-using-azcopy"></a>Copia de blobs entre cuentas de almacenamiento de Azure mediante AzCopy

Puede copiar blobs, directorios y contenedores entre cuentas de almacenamiento mediante la utilidad de línea de comandos AzCopy v10. 

Para ver ejemplos de otros tipos de tareas, como la carga de archivos, la descarga de blobs y la sincronización con Blob Storage, consulte los vínculos presentados en la sección [Pasos siguientes](#next-steps) de este artículo.

AzCopy usa interfaces [API](/rest/api/storageservices/put-page-from-url)[de servidor a servidor](/rest/api/storageservices/put-block-from-url), por lo que los datos se copian directamente entre servidores de almacenamiento. En estas operaciones de copia no se usa el ancho de banda de red del equipo.

Para descargar AzCopy y conocer las formas en que puede proporcionar credenciales de autorización al servicio de almacenamiento, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md). 

## <a name="guidelines"></a>Directrices

Aplique las siguientes directrices a los comandos AzCopy. 

- El cliente debe tener acceso a redes tanto en las cuentas de almacenamiento de origen y de destino. Para aprender a configurar la red de cada cuenta de almacenamiento, consulte [Configuración de redes virtuales y firewalls de Azure Storage](storage-network-security.md?toc=/azure/storage/blobs/toc.json).

- Anexe un token de SAS a cada dirección URL de origen. 

  Si proporciona credenciales de autorización mediante Azure Active Directory (AD), puede omitir el token de SAS solo de la dirección URL de destino. Asegúrese de que ha configurado los roles adecuados en su cuenta de destino. Consulte [Opción 1: Uso de Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory). 

  En los ejemplos de este artículo se da por hecho que ha autenticado su identidad mediante Azure AD, por tanto, se omiten los tokens de SAS de la dirección URL de destino.

-  Si realiza la copia en una cuenta de almacenamiento de blobs en bloques premium, omita el nivel de acceso de un blob de la operación de copia estableciendo `s2s-preserve-access-tier` en `false` (por ejemplo: `--s2s-preserve-access-tier=false`). Las cuentas de almacenamiento de blobs en bloques Premium no admiten niveles de acceso. 

- Si realiza la copia en una cuenta que tiene un espacio de nombres jerárquico o desde allí, use `blob.core.windows.net` en lugar de `dfs.core.windows.net` en la sintaxis de la dirección URL. El [acceso multiprotocolo en Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) le permite usar `blob.core.windows.net` y es la única sintaxis admitida en escenarios de copia entre cuentas. 

- Para aumentar el rendimiento de las operaciones, puede establecer el valor de la variable de entorno `AZCOPY_CONCURRENCY_VALUE`. Para más información, consulte [Aumento de la simultaneidad](storage-use-azcopy-optimize.md#increase-concurrency). 

- Si los blobs de origen tienen etiquetas de índice y quiere conservar esas etiquetas, tendrá que volver a aplicarlas a los blobs de destino. Para información sobre cómo establecer etiquetas de índice, consulte la sección [Copia de blobs en otra cuenta de almacenamiento con etiquetas de índice](#copy-between-accounts-and-add-index-tags) de este artículo.

## <a name="copy-a-blob"></a>Copia de un blob

Copie un blob en otra cuenta de almacenamiento mediante el comando [azcopy copy](storage-ref-azcopy-copy.md). 

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path>'`

**Ejemplo**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos. 

## <a name="copy-a-directory"></a>Copia de un directorio

Copie un directorio en otra cuenta de almacenamiento mediante el comando [azcopy copy](storage-ref-azcopy-copy.md). 

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive`

**Ejemplo**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive
```

La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos.

## <a name="copy-a-container"></a>Copia de un contenedor

Copie un contenedor en otra cuenta de almacenamiento mediante el comando [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive`

**Ejemplo**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive
```

La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos.

## <a name="copy-containers-directories-and-blobs"></a>Copia de contenedores, directorios y blobs

Copie todos los contenedores, directorios y blobs en otra cuenta de almacenamiento mediante el comando [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/' --recursive`

**Ejemplo**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive
```

La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos.

<a id="copy-between-accounts-and-add-index-tags"></a>

## <a name="copy-blobs-and-add-index-tags"></a>Copia de blobs e incorporación de etiquetas de índice

Copie blobs en otra cuenta de almacenamiento y agregue [etiquetas de índice de blob (versión preliminar)](../blobs/storage-manage-find-blobs.md) al blob de destino.

Si usa la autorización de Azure AD, a la entidad de seguridad se le debe asignar el rol [Propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o se le debe conceder permiso para la `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [operación del proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) a través de un rol personalizado de Azure. Si usa un token de firma de acceso compartido (SAS), ese token debe proporcionar acceso a las etiquetas del blob mediante el permiso `t` de SAS.

Para agregar etiquetas, use la opción `--blob-tags` junto con un par clave-valor codificado como dirección URL. 

Por ejemplo, para agregar la clave `my tag` y un valor `my tag value`, agregaría `--blob-tags='my%20tag=my%20tag%20value'` al parámetro de destino. 

Separe varias etiquetas de índice mediante el carácter de Y comercial (`&`).  Por ejemplo, si quiere agregar una clave `my second tag` y un valor `my second tag value`, la cadena de opción completa sería `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`.

En el siguiente ejemplo, se muestra cómo usar la opción `--blob-tags`.

> [!TIP]
> En estos ejemplos se delimitan los argumentos de ruta de acceso con comillas (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Ejemplo de blob**

```azcopy

azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

**Ejemplo de directorio**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

 **Ejemplo de contenedor**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

**Ejemplo de cuenta**

```azcopy
azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'
```

La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos.

> [!NOTE]
> Si especifica un directorio, un contenedor o una cuenta como origen, todos los blobs que se copian en el destino tendrán las mismas etiquetas que especifique en el comando.

## <a name="copy-with-optional-flags"></a>Copia con marcas opcionales

Puede modificar las operaciones de copia mediante marcas opcionales. Estos son algunos ejemplos.

|Escenario|Marca|
|---|---|
|Copie blobs como blobs en bloques, en páginas o en anexos.|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
|Copiar en un nivel de acceso específico (como el nivel de archivo).|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|
|Descomprimir archivos automáticamente.|**--decompress**=\[gzip\|deflate\]|

Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options). 

## <a name="next-steps"></a>Pasos siguientes

Encuentre más ejemplos en estos artículos:

- [Ejemplos: Carga](storage-use-azcopy-blobs-upload.md)
- [Ejemplos: descarga](storage-use-azcopy-blobs-download.md)
- [Ejemplos: Sincronización](storage-use-azcopy-blobs-synchronize.md)
- [Ejemplos: Cubos de Amazon S3](storage-use-azcopy-s3.md)
- [Ejemplos: Google Cloud Storage](storage-use-azcopy-google-cloud.md)
- [Ejemplos: Azure Files](storage-use-azcopy-files.md)
- [Tutorial: Migración de datos locales al almacenamiento en la nube mediante AzCopy](storage-use-azcopy-migrate-on-premises-data.md)

Consulte estos artículos para configurar opciones, optimizar el rendimiento y solucionar problemas:

- [Parámetros de configuración de AzCopy](storage-ref-azcopy-configuration-settings.md)
- [Optimización del rendimiento de AzCopy](storage-use-azcopy-optimize.md)
- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)
