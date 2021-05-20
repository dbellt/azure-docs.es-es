---
title: Sincronización con Azure Blob Storage mediante AzCopy v10 | Microsoft Docs
description: Este artículo contiene una colección de comandos de ejemplo de AzCopy que le ayudarán a realizar la sincronización con Azure Blob Storage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 4dc2dd06128c373439229b5e649c37caa25b727e
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715023"
---
# <a name="synchronize-with-azure-blob-storage-by-using-azcopy"></a>Sincronización con Azure Blob Storage mediante AzCopy

Puede sincronizar el almacenamiento local con Azure Blob Storage mediante la utilidad de línea de comandos AzCopy v10. 

Puede sincronizar el contenido de un sistema de archivos local con un contenedor de blobs. También puede sincronizar los contenedores y directorios virtuales entre sí. La sincronización es unidireccional. En otras palabras, tendrá que elegir cuál de estos dos puntos de conexión es el origen y cuál es el destino. La sincronización también usa las API de servidor a servidor. Los ejemplos que se presentan en esta sección también funcionan con las cuentas que tienen un espacio de nombres jerárquico. 

> [!NOTE]
> La versión actual de AzCopy no sincroniza entre otros orígenes y destinos (por ejemplo: Almacenamiento de archivos o cubos de Amazon Web Services (AWS) S3).

Para ver ejemplos de otros tipos de tareas, como cargar archivos, descargar blobs o copiar blobs entre cuentas, consulte los vínculos presentados en la sección [Pasos siguientes](#next-steps) de este artículo.

## <a name="get-started"></a>Introducción

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y obtener información sobre las formas de proporcionar credenciales de autorización para el servicio de almacenamiento.

> [!NOTE] 
> En los ejemplos de este artículo se da por hecho que ha proporcionado credenciales de autorización mediante Azure Active Directory (Azure AD).
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy. Por ejemplo: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="guidelines"></a>Directrices

- El comando [sync](storage-ref-azcopy-sync.md) compara nombres de archivo y las marcas de tiempo de la última modificación. Establezca la marca opcional `--delete-destination` en un valor de `true` o `prompt` para eliminar archivos en el directorio de destino si esos archivos ya no existen en el directorio de origen.

- Si establece la marca `--delete-destination` en `true`, AzCopy elimina los archivos sin proporcionar un aviso. Si quiere que aparezca un mensaje antes de que AzCopy elimine un archivo, establezca la marca `--delete-destination` en `prompt`.

- Si tiene previsto establecer la marca `--delete-destination` en `prompt` o `false`, considere la posibilidad de usar el comando [copy](storage-ref-azcopy-copy.md) en lugar del comando [sync](storage-ref-azcopy-sync.md) y establezca el parámetro `--overwrite` en `ifSourceNewer`. El comando [copy](storage-ref-azcopy-copy.md) consume menos memoria y genera menos costos de facturación porque una operación de copia no tiene que indexar el origen o el destino antes de mover archivos. 

- Para evitar eliminaciones accidentales, asegúrese de habilitar la característica de [eliminación temporal](../blobs/soft-delete-blob-overview.md) antes de usar la marca `--delete-destination=prompt|true`.

- La máquina en la que ejecute el comando de sincronización debe tener un reloj del sistema preciso porque las últimas horas modificadas son fundamentales para determinar si se debe transferir un archivo. Si el sistema tiene un sesgo de reloj importante, evite modificar los archivos en el destino demasiado cerca de la hora en que planea ejecutar un comando de sincronización.

## <a name="update-a-container-with-changes-to-a-local-file-system"></a>Actualización de un contenedor con los cambios realizados en un sistema de archivos local

En este caso, el contenedor es el destino y el sistema de archivos local es el origen. 

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy sync '<local-directory-path>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive`

**Ejemplo**

```azcopy
azcopy sync 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive
```

## <a name="update-a-local-file-system-with-changes-to-a-container"></a>Actualización de un sistema de archivos local con los cambios realizados en un contenedor

En este caso, el sistema de archivos local es el destino y el contenedor es el origen.

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy sync 'https://<storage-account-name>.blob.core.windows.net/<container-name>' 'C:\myDirectory' --recursive`

**Ejemplo**

```azcopy
azcopy sync 'https://mystorageaccount.blob.core.windows.net/mycontainer' 'C:\myDirectory' --recursive
```

## <a name="update-a-container-with-changes-in-another-container"></a>Actualización de un contenedor con cambios en otro contenedor

El primer contenedor que aparece en este comando es el contenedor origen. El segundo es el destino. Asegúrese de anexar un token de SAS a cada URL de origen.  

Si proporciona credenciales de autorización mediante Azure Active Directory (AD), puede omitir el token de SAS solo de la dirección URL de destino. Asegúrese de que ha configurado los roles adecuados en su cuenta de destino. Consulte [Opción 1: Uso de Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive`

**Ejemplo**

```azcopy
azcopy sync 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive
```

## <a name="update-a-directory-with-changes-to-a-directory-in-another-container"></a>Actualización de un directorio con cambios en un directorio de otro contenedor

El primer directorio que aparece en este comando es el origen. El segundo es el destino. Asegúrese de anexar un token de SAS a cada URL de origen.  

Si proporciona credenciales de autorización mediante Azure Active Directory (AD), puede omitir el token de SAS solo de la dirección URL de destino. Asegúrese de que ha configurado los roles adecuados en su cuenta de destino. Consulte [Opción 1: Uso de Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory).

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

**Sintaxis**

`azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive`

**Ejemplo**

```azcopy
azcopy sync 'https://mysourceaccount.blob.core.windows.net/<container-name>/myDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myDirectory' --recursive
```

## <a name="synchronize-with-optional-flags"></a>Sincronización con marcas opcionales

Puede modificar las operaciones de sincronización mediante marcas opcionales. Estos son algunos ejemplos.

|Escenario|Marca|
|---|---|
|Especificar cómo de estrictamente se deben validar los hashes MD5 al descargarse.|**--check-md5**=\[NoCheck\|LogOnly\|FailIfDifferent\|FailIfDifferentOrMissing\]|
|Excluir archivos en función de un patrón.|**--exclude-path**|
|Especifique el grado de detalles que quiere que sean las entradas de registro relacionadas con la sincronización.|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|

Para obtener una lista completa de marcas, vea las [opciones](storage-ref-azcopy-sync.md#options).

> [!NOTE]
> La marca `--recursive` se establece en `true` de manera predeterminada. Las marcas `--exclude-pattern` y `--include-pattern` solo afectan a los nombres de archivo y no a otros elementos de la ruta de acceso del archivo. 

## <a name="next-steps"></a>Pasos siguientes

Encuentre más ejemplos en estos artículos:

- [Ejemplos: Carga](storage-use-azcopy-blobs-upload.md)
- [Ejemplos: descarga](storage-use-azcopy-blobs-download.md)
- [Ejemplos: Copia entre cuentas](storage-use-azcopy-blobs-copy.md)
- [Ejemplos: Cubos de Amazon S3](storage-use-azcopy-s3.md)
- [Ejemplos: Google Cloud Storage](storage-use-azcopy-google-cloud.md)
- [Ejemplos: Azure Files](storage-use-azcopy-files.md)
- [Tutorial: Migración de datos locales al almacenamiento en la nube mediante AzCopy](storage-use-azcopy-migrate-on-premises-data.md)

Consulte estos artículos para configurar opciones, optimizar el rendimiento y solucionar problemas:

- [Parámetros de configuración de AzCopy](storage-ref-azcopy-configuration-settings.md)
- [Optimización del rendimiento de AzCopy](storage-use-azcopy-optimize.md)
- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)

