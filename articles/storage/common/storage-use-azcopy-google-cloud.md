---
title: Copia desde Google Cloud Storage a Azure Storage con AzCopy | Microsoft Docs
description: Use AzCopy para copiar datos de Google Cloud Storage a Azure Storage. AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: 3b2ad11abb7d1a3e64deef1ca49d9f84f03e5879
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107498346"
---
# <a name="copy-data-from-google-cloud-storage-to-azure-storage-by-using-azcopy-preview"></a>Copia de datos de Google Cloud Storage a Azure Storage con AzCopy (versión preliminar)

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. Este artículo le ayuda a copiar objetos, directorios y cubos de Google Cloud Storage a Azure Blob Storage mediante AzCopy.

> [!IMPORTANT]
> La copia de datos de Google Cloud Storage a Azure Storage se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="choose-how-youll-provide-authorization-credentials"></a>Elección del modo de proporcionar las credenciales de autorización

* Para la autorización con Azure Storage, use Azure Active Directory (AD) o un token de firma de acceso compartido (SAS).

* Para la autorización con Google Cloud Storage, use una clave de cuenta de servicio.

### <a name="authorize-with-azure-storage"></a>Autorización con Azure Storage

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y obtener información sobre las formas de proporcionar credenciales de autorización para el servicio de almacenamiento.

> [!NOTE] 
> En los ejemplos de este artículo se da por hecho que ha proporcionado credenciales de autorización mediante Azure Active Directory (Azure AD).
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy. Por ejemplo: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

### <a name="authorize-with-google-cloud-storage"></a>Autorización con Google Cloud Storage

Para la autorización con Google Cloud Storage, usará una clave de cuenta de servicio. Para obtener información acerca de cómo crear una clave de cuenta de servicio, consulte [Crea y administra claves de cuentas de servicio](https://cloud.google.com/iam/docs/creating-managing-service-account-keys).

Después de obtener una clave de servicio, establezca la variable de entorno `GOOGLE_APPLICATION_CREDENTIALS` en una ruta de acceso absoluta al archivo de clave de cuenta de servicio:

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |
| **Linux** | `export GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |
| **macOS** | `export GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |

## <a name="copy-objects-directories-and-buckets"></a>Copia de objetos, directorios y cubos

AzCopy usa la API [Put Block From URL](/rest/api/storageservices/put-block-from-url), por lo que los datos se copian directamente entre Google Cloud Storage y los servidores de almacenamiento. En estas operaciones de copia no se usa el ancho de banda de red del equipo.

> [!TIP]
> En los ejemplos de esta sección se delimitan los argumentos de ruta de acceso con comillas simples (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

 Estos ejemplos también funcionan con las cuentas que tienen un espacio de nombres jerárquico. El [acceso multiprotocolo en Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) le permite usar la misma sintaxis de URL (`blob.core.windows.net`) en esas cuentas. 

### <a name="copy-an-object"></a>Copia de un objeto

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'
```


### <a name="copy-a-directory"></a>Copia de un directorio

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true
```

> [!NOTE]
> En este ejemplo se anexa la marca `--recursive` para copiar archivos en todos los subdirectorios.

### <a name="copy-the-contents-of-a-directory"></a>Copia de los contenidos de un directorio

Puede copiar el contenido de un directorio sin copiar el propio directorio contenedor mediante el carácter comodín (*).

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<directory-name>/*' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true
```

### <a name="copy-a-cloud-storage-bucket"></a>Copia de un cubo de Cloud Storage

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/mybucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true
```

### <a name="copy-all-buckets-in-a-google-cloud-project"></a>Copia de todos los cubos en un proyecto de Google Cloud 

En primer lugar, establezca el valor de `GOOGLE_CLOUD_PROJECT` en el identificador del proyecto de Google Cloud.

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/' 'https://mystorageaccount.blob.core.windows.net' --recursive=true
```

### <a name="copy-a-subset-of-buckets-in-a-google-cloud-project"></a>Copia de un subconjunto de cubos en un proyecto de Google Cloud 

En primer lugar, establezca el valor de `GOOGLE_CLOUD_PROJECT` en el identificador del proyecto de Google Cloud.

Copia de un subconjunto de cubos mediante un símbolo comodín (*) en el nombre del cubo. Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

**Sintaxis**

`azcopy copy 'https://storage.cloud.google.com/<bucket*name>' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true`

**Ejemplo**

```azcopy
azcopy copy 'https://storage.cloud.google.com/my*bucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true
```

## <a name="handle-differences-in-bucket-naming-rules"></a>Control de las diferencias en las reglas de nomenclatura de cubos

Google Cloud Storage tiene otro conjunto de convenciones de nomenclatura para los nombres de cubo, en comparación con los contenedores de blobs de Azure. [Aquí](https://cloud.google.com/storage/docs/naming-buckets) puede leer más al respecto. Si elige copiar un grupo de cubos en una cuenta de Azure Storage, es posible que se produzca un error en la operación de copia debido a las diferencias de nomenclatura.

AzCopy se encarga de tres de los problemas más comunes que pueden surgir; cubos que contienen puntos, cubos que contienen guiones consecutivos y cubos que contienen guiones bajos. Los nombres de cubo de Google Cloud Storage pueden contener puntos y guiones consecutivos, pero un contenedor de Azure no. AzCopy reemplaza los puntos con guiones y los guiones consecutivos con un número que representa el número de guiones consecutivos (por ejemplo: un cubo denominado `my----bucket` se convierte en `my-4-bucket`). Si el nombre del cubo tiene un guion bajo (`_`), AzCopy lo reemplaza por un guion (por ejemplo: un depósito denominado `my_bucket` se convierte en `my-bucket`). 

## <a name="handle-differences-in-object-naming-rules"></a>Control de las diferencias en las reglas de nomenclatura de objetos

Google Cloud Storage tiene otro conjunto de convenciones de nomenclatura para los nombres de objeto, en comparación con los blobs de Azure. [Aquí](https://cloud.google.com/storage/docs/naming-objects) puede leer más al respecto.

Azure Storage no permite que los nombres de objeto (o cualquier segmento de la ruta de acceso del directorio virtual) terminen con puntos finales (por ejemplo, `my-bucket...`). Los puntos finales se eliminan cuando se realiza la operación de copia. 

## <a name="handle-differences-in-object-metadata"></a>Control de las diferencias en los metadatos de objetos

Google Cloud Storage y Azure permiten distintos conjuntos de caracteres en los nombres de las claves de objeto. [Aquí](https://cloud.google.com/storage/docs/metadata) puede leer sobre los metadatos en Google Cloud Storage. En Azure, las claves de objeto de blob se adhieren a las reglas de nomenclatura para los [ identificadores de C#](/dotnet/csharp/language-reference/).

Como parte de un comando `copy` de AzCopy, puede proporcionar un valor opcional para la marca `s2s-handle-invalid-metadata` que especifique cómo quiere controlar los archivos donde los metadatos del archivo contienen nombres de clave no compatibles. En la tabla siguiente se describe cada valor de marca.

| Valor de marca | Descripción  |
|--------|-----------|
| **ExcludeIfInvalid** | (Opción predeterminada) Los metadatos no se incluyen en el objeto transferido. AzCopy registra una advertencia. |
| **FailIfInvalid** | Los objetos no se copian. AzCopy registra un error y lo incluye en el recuento de errores que aparece en el resumen de la transferencia.  |
| **RenameIfInvalid**  | AzCopy resuelve la clave de metadatos no válida y copia el objeto en Azure mediante el par clave-valor de metadatos resuelto. Para saber exactamente qué pasos realiza AzCopy para cambiar el nombre de las claves de objeto, vea la sección [Cómo cambia AzCopy el nombre de las claves de objeto](#rename-logic) más adelante. Si AzCopy no puede cambiar el nombre de la clave, el objeto no se copiará. |

<a id="rename-logic"></a>

### <a name="how-azcopy-renames-object-keys"></a>Cómo cambia AzCopy el nombre de las claves de objeto

AzCopy realiza estos pasos:

1. Reemplaza los caracteres no válidos con "_".

2. Agrega la cadena `rename_` al principio de una nueva clave válida.

   Esta clave se usará para guardar el **valor** de los metadatos originales.

3. Agrega la cadena `rename_key_` al principio de una nueva clave válida.
   Esta clave se usará para guardar la **clave** no válida de los metadatos originales.
   Puede usar esta clave para intentar recuperar los metadatos en Azure, ya que la clave de metadatos se conserva como un valor en el servicio Blob Storage.

## <a name="next-steps"></a>Pasos siguientes

Encuentre más ejemplos en estos artículos:

- [Ejemplos: Carga](storage-use-azcopy-blobs-upload.md)
- [Ejemplos: descarga](storage-use-azcopy-blobs-download.md)
- [Ejemplos: Copia entre cuentas](storage-use-azcopy-blobs-copy.md)
- [Ejemplos: Sincronización](storage-use-azcopy-blobs-synchronize.md)
- [Ejemplos: Cubos de Amazon S3](storage-use-azcopy-s3.md)
- [Ejemplos: Azure Files](storage-use-azcopy-files.md)
- [Tutorial: Migración de datos locales al almacenamiento en la nube mediante AzCopy](storage-use-azcopy-migrate-on-premises-data.md)

Consulte estos artículos para configurar opciones, optimizar el rendimiento y solucionar problemas:

- [Parámetros de configuración de AzCopy](storage-ref-azcopy-configuration-settings.md)
- [Optimización del rendimiento de AzCopy](storage-use-azcopy-optimize.md)
- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)