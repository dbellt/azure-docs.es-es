---
title: Copia o transferencia de datos a Azure Storage con AzCopy v10 | Microsoft Docs
description: AzCopy es una utilidad de línea de comandos que puede usar para copiar datos entre cuentas de almacenamiento. En este artículo sirve de ayuda para descargar AzCopy, conectarse a la cuenta de almacenamiento y, a continuación, transferir archivos.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.custom: contperf-fy21q2
ms.openlocfilehash: 34d3bd45d2c0bf0260a4f8524cff6f8ac03b746c
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107501726"
---
# <a name="get-started-with-azcopy"></a>Introducción a AzCopy

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. En este artículo sirve de ayuda para descargar AzCopy, conectarse a la cuenta de almacenamiento y, a continuación, transferir archivos.

> [!NOTE]
> AzCopy **V10** es la versión admitida actualmente de AzCopy.
>
> Si necesita usar una versión anterior de AzCopy, consulte la sección [Uso de la versión anterior de AzCopy](#previous-version) de este artículo.

<a id="download-and-install-azcopy"></a>

## <a name="download-azcopy"></a>Descargar AzCopy

En primer lugar, descargue el archivo ejecutable de AzCopy V10 en cualquier directorio en el equipo. AzCopy V10 es solo un archivo ejecutable, de modo que no hay que instalar nada.

- [Windows 64 bits](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Windows 32 bits](https://aka.ms/downloadazcopy-v10-windows-32bit) (zip)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

Estos archivos se comprimen como un archivo ZIP (Windows y Mac) o un archivo TAR (Linux). Para descargar y descomprimir el archivo tar en Linux, consulte la documentación de su distribución de Linux.

> [!NOTE]
> Si desea copiar datos desde y hacia su servicio de [almacenamiento de Azure Table](../tables/table-storage-overview.md), instale [AzCopy versión 7.3](https://aka.ms/downloadazcopynet).

## <a name="run-azcopy"></a>Ejecución de AzCopy

Para mayor comodidad, considere la posibilidad de agregar la ubicación del directorio del ejecutable de AzCopy a la ruta de acceso del sistema para facilitar su uso. De este modo, puede escribir `azcopy` desde cualquier directorio del sistema.

Si decide no agregar el directorio de AzCopy a la ruta de acceso, tendrá que cambiar los directorios a la ubicación de su archivo ejecutable de AzCopy y escribir `azcopy` o `.\azcopy` en los símbolos del sistema de Windows PowerShell.

Como propietario de la cuenta de Azure Storage, no se le asignan automáticamente permisos para tener acceso a datos. Antes de hacer nada significativo con AzCopy, debe decidir cómo proporcionará las credenciales de autorización al servicio de almacenamiento. 

<a id="choose-how-youll-provide-authorization-credentials"></a>

## <a name="authorize-azcopy"></a>Autorización de AzCopy

Puede proporcionar las credenciales de autorización mediante Azure Active Directory (AD) o mediante un token de firma de acceso compartido (SAS).

Use esta tabla como guía:

| Tipo de almacenamiento | Método de autorización admitido actualmente |
|--|--|
|**Blob Storage** | Azure AD y SAS |
|**Blob Storage (espacio de nombres jerárquico)** | Azure AD y SAS |
|**Almacenamiento de archivos** | SAS solo |

#### <a name="option-1-use-azure-active-directory"></a>Opción 1: Uso de Azure Active Directory

Esta opción solo está disponible para Blob storage. Con Azure Active Directory, puede proporcionar credenciales una vez en lugar de anexar un token de SAS a cada comando.  

> [!NOTE]
> En la versión actual, si tiene previsto copiar blobs entre distintas cuentas de almacenamiento, deberá anexar un token de SAS a cada URL de origen. Puede omitir el token de SAS solo de la dirección URL de destino. Si necesita ejemplos, vea [Copia de blobs entre cuentas de almacenamiento](#transfer-data).

Para autorizar el acceso mediante Azure AD, consulte [Autorización del acceso a blobs con AzCopy y Azure Active Directory (Azure AD)](storage-use-azcopy-authorize-azure-active-directory.md).

#### <a name="option-2-use-a-sas-token"></a>Opción 2: Uso de un token de SAS

Puede anexar un token de SAS a cada dirección URL de origen o destino que use en los comandos de AzCopy.

El comando de este ejemplo copia recursivamente los datos desde un directorio local a un contenedor de blobs. Un token de SAS ficticio se anexa al final de la dirección URL del contenedor.

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

Para más información sobre los tokens de SAS y de cómo obtener uno, consulte [Uso de firmas de acceso compartido (SAS)](./storage-sas-overview.md).

> [!NOTE]
> El parámetro [Se requiere transferencia segura](storage-require-secure-transfer.md) de una cuenta de almacenamiento determina si la conexión a una cuenta de almacenamiento está protegida con Seguridad de la capa de transporte (TLS). Esta opción está habilitada de forma predeterminada.   

<a id="transfer-data"></a>

## <a name="transfer-data"></a>Transferencia de datos

Después de autorizar la identidad o de obtener un token de SAS, puede comenzar la transferencia de datos.

Para obtener ejemplos de comandos, consulte cualquiera de estos artículos.

| Servicio | Artículo |
|--------|-----------|
|Azure Blob Storage|[Carga de archivos en Azure Blob Storage](storage-use-azcopy-blobs-upload.md) |
|Azure Blob Storage|[Descarga de blobs desde Azure Blob Storage](storage-use-azcopy-blobs-download.md)|
|Azure Blob Storage|[Copia de blobs entre cuentas de Azure Storage](storage-use-azcopy-blobs-copy.md)|
|Azure Blob Storage|[Sincronización con Azure Blob Storage](storage-use-azcopy-blobs-synchronize.md)|
|Azure Files |[Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)|
|Amazon S3|[Copia de datos de Amazon S3 a Azure Storage](storage-use-azcopy-s3.md)|
|Google Cloud Storage|[Copia de datos de Google Cloud Storage a Azure Storage (versión preliminar)](storage-use-azcopy-google-cloud.md)|
|Almacenamiento de Azure Stack|[Transferencia de datos con AzCopy y Azure Stack Storage](/azure-stack/user/azure-stack-storage-transfer#azcopy)|

## <a name="get-command-help"></a>Obtención de ayuda sobre los comandos

Para ver una lista de los comandos, escriba `azcopy -h` y, a continuación, presione la tecla ENTRAR.

Para obtener información acerca de un comando específico, basta con que incluya el nombre del comando (por ejemplo: `azcopy list -h`).

> [!div class="mx-imgBorder"]
> ![Ayuda en línea](media/storage-use-azcopy-v10/azcopy-inline-help.png)

### <a name="list-of-commands"></a>Lista de comandos

En la tabla siguiente se enumeran todos los comandos de AzCopy v10. Cada comando está vinculado a un artículo de referencia. 

|Get-Help|Descripción|
|---|---|
|[azcopy bench](storage-ref-azcopy-bench.md?toc=/azure/storage/blobs/toc.json)|Ejecuta una prueba comparativa de rendimiento al cargar o descargar datos de prueba hacia o desde un destino especificado.|
|[azcopy copy](storage-ref-azcopy-copy.md?toc=/azure/storage/blobs/toc.json)|Copia los datos de origen en una ubicación de destino.|
|[azcopy doc](storage-ref-azcopy-doc.md?toc=/azure/storage/blobs/toc.json)|Genera documentación para la herramienta en formato de Markdown.|
|[azcopy env](storage-ref-azcopy-env.md?toc=/azure/storage/blobs/toc.json)|Muestra las variables de entorno que pueden configurar el comportamiento de AzCopy.|
|[azcopy jobs](storage-ref-azcopy-jobs.md?toc=/azure/storage/blobs/toc.json)|Subcomandos relacionados con la administración de trabajos.|
|[azcopy jobs clean](storage-ref-azcopy-jobs-clean.md?toc=/azure/storage/blobs/toc.json)|Elimina todos los archivos de registro y de plan de todos los trabajos.|
|[azcopy jobs list](storage-ref-azcopy-jobs-list.md?toc=/azure/storage/blobs/toc.json)|Muestra información sobre todos los trabajos.|
|[azcopy jobs remove](storage-ref-azcopy-jobs-remove.md?toc=/azure/storage/blobs/toc.json)|Quita todos los archivos asociados al identificador de trabajo especificado.|
|[azcopy jobs resume](storage-ref-azcopy-jobs-resume.md?toc=/azure/storage/blobs/toc.json)|Reanuda el trabajo existente con el identificador de trabajo especificado.|
|[azcopy jobs show](storage-ref-azcopy-jobs-show.md?toc=/azure/storage/blobs/toc.json)|Muestra información detallada del identificador de trabajo especificado.|
|[azcopy load](storage-ref-azcopy-load.md)|Subcomandos relacionados con la transferencia de datos en formatos específicos.|
|[azcopy load clfs](storage-ref-azcopy-load-avere-cloud-file-system.md?toc=/azure/storage/blobs/toc.json)|Transfiere datos locales a un contenedor y los almacena en el formato Avere Cloud FileSystem (CLFS) de Microsoft.|
|[azcopy list](storage-ref-azcopy-list.md?toc=/azure/storage/blobs/toc.json)|Enumera las entidades de un recurso determinado.|
|[azcopy login](storage-ref-azcopy-login.md?toc=/azure/storage/blobs/toc.json)|Inicia sesión en Azure Active Directory para acceder a recursos de Azure Storage.|
|[azcopy logout](storage-ref-azcopy-logout.md?toc=/azure/storage/blobs/toc.json)|Cierra la sesión del usuario y finaliza el acceso a los recursos de Azure Storage.|
|[azcopy make](storage-ref-azcopy-make.md?toc=/azure/storage/blobs/toc.json)|Crea un contenedor o un recurso compartido de archivos.|
|[azcopy remove](storage-ref-azcopy-remove.md?toc=/azure/storage/blobs/toc.json)|Elimine blobs o archivos de una cuenta de almacenamiento de Azure.|
|[azcopy sync](storage-ref-azcopy-sync.md?toc=/azure/storage/blobs/toc.json)|Replica la ubicación de origen en la ubicación de destino.|

## <a name="use-in-a-script"></a>Uso en un script

#### <a name="obtain-a-static-download-link"></a>Obtención de un vínculo de descarga estático

Con el tiempo, el [vínculo de descarga](#download-and-install-azcopy) de AzCopy apuntará a nuevas versiones de AzCopy. Si el script descarga AzCopy, puede que deje de funcionar si una versión más reciente de AzCopy modifica las características de las que depende.

Para evitar estos problemas, obtenga un vínculo estático (sin cambios) a la versión actual de AzCopy. De este modo, el script descarga exactamente la misma versión de AzCopy cada vez que se ejecuta.

Para obtener el vínculo, ejecute este comando:

| Sistema operativo  | Get-Help |
|--------|-----------|
| **Linux** | `curl -s -D- https://aka.ms/downloadazcopy-v10-linux | grep ^Location` |
| **Windows** | `(curl https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction silentlycontinue).headers.location` |

> [!NOTE]
> Para Linux, `--strip-components=1` en el comando `tar` elimina la carpeta de nivel superior que contiene el nombre de la versión y, en su lugar, extrae el archivo binario directamente en la carpeta actual. Esto permite que el script se actualice con una nueva versión de `azcopy` al actualizar únicamente la dirección URL de `wget`.

La dirección URL aparece en la salida de este comando. A continuación, el script puede descargar AzCopy mediante esa dirección URL.

| Sistema operativo  | Get-Help |
|--------|-----------|
| **Linux** | `wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1` |
| **Windows** | `Invoke-WebRequest https://azcopyvnext.azureedge.net/release20190517/azcopy_windows_amd64_10.1.2.zip -OutFile azcopyv10.zip <<Unzip here>>` |

#### <a name="escape-special-characters-in-sas-tokens"></a>Escape de caracteres especiales en tokens de SAS

En los archivos por lotes con la extensión `.cmd`, tendrá que usar una secuencia de escape en los caracteres `%` que aparezcan en los tokens de SAS. Para ello, agregue un carácter `%` adicional junto a los caracteres `%` existentes en la cadena de token de SAS.

#### <a name="run-scripts-by-using-jenkins"></a>Ejecutar scripts mediante Jenkins

Si tiene previsto usar [Jenkins](https://jenkins.io/) para ejecutar scripts, asegúrese de colocar el siguiente comando al principio del script.

```
/usr/bin/keyctl new_session
```

## <a name="use-in-azure-storage-explorer"></a>Uso en el Explorador de Azure Storage

El [Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) usa AzCopy para realizar todas sus operaciones de transferencia de datos. Puede usar el [Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) si desea aprovechar las ventajas de rendimiento de AzCopy, pero prefiere usar una interfaz gráfica de usuario, en lugar de la línea de comandos, para interactuar con los archivos.

El Explorador de Azure Storage utiliza la clave de la cuenta para realizar operaciones, por lo que después de iniciar sesión en el Explorador de Azure Storage, no tendrá que proporcionar credenciales de autorización adicionales.

<a id="previous-version"></a>

## <a name="configure-optimize-and-fix"></a>Configuración, optimización y corrección

Consulte los siguientes recursos:

- [Parámetros de configuración de AzCopy](storage-ref-azcopy-configuration-settings.md)

- [Optimización del rendimiento de AzCopy](storage-use-azcopy-optimize.md)

- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)

## <a name="use-a-previous-version"></a>Uso de una versión anterior

Si necesita usar la versión anterior de AzCopy, consulte alguno de los vínculos siguientes:

- [AzCopy en Windows (v8)](/previous-versions/azure/storage/storage-use-azcopy)

- [AzCopy en Linux (v7)](/previous-versions/azure/storage/storage-use-azcopy-linux)

## <a name="next-steps"></a>Pasos siguientes

Si tiene alguna pregunta, problemas o comentarios generales, envíelos en [la página GitHub](https://github.com/Azure/azure-storage-azcopy).
