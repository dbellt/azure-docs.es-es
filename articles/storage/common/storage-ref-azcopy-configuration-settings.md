---
title: Configuración de AzCopy v10 (Azure Storage) | Microsoft Docs
description: En este artículo se proporciona información de referencia para los valores de configuración de AzCopy v10.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 4fcbcf145dc417d2a7f78913e70429c3929cd902
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107508993"
---
# <a name="azcopy-v10-configuration-settings-azure-storage"></a>Opciones de configuración de AzCopy v10 (Azure Storage)

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. Este artículo contiene una lista de variables de entorno que se pueden usar para configurar AzCopy v10.

> [!NOTE]
> Si busca contenido que le ayude a empezar a trabajar con AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

## <a name="azcopy-v10-environment-variables"></a>Variables de entorno de AzCopy v10

La tabla siguiente describe cada variable de entorno e incluye vínculos al contenido que pueden ayudarle a usar la variable.

| Variable de entorno | Descripción |
|--|--|
| AWS_ACCESS_KEY_ID | Clave de acceso de Amazon Web Services. Proporciona una clave para autorizar el acceso con Amazon Web Services. [Copia de datos de Amazon S3 a Azure Storage mediante AzCopy](storage-use-azcopy-s3.md) |
| AWS_SECRET_ACCESS_KEY | La clave de acceso secreta de Amazon Web Services proporciona una clave secreta para autorizar el acceso con Amazon Web Services. [Copia de datos desde Amazon S3 a Azure Storage con AzCopy](storage-use-azcopy-s3.md) |
| AZCOPY_ACTIVE_DIRECTORY_ENDPOINT | El punto de conexión de Azure Active Directory que se usará. Esta variable solo se usa para el inicio de sesión automático. Utilice la marca de línea de comandos en su lugar para invocar el comando de inicio de sesión. |
| AZCOPY_AUTO_LOGIN_TYPE | Establezca esta variable en `DEVICE`, `MSI` o `SPN`. Esta variable permite autorizar el acceso sin usar el comando `azcopy login`. Este mecanismo es útil en los casos en los que el sistema operativo no tiene un almacén de secretos, como el *conjunto de claves* de Linux. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_BUFFER_GB | Establezca la cantidad máxima de memoria del sistema que desea que AzCopy utilice para descargar y cargar archivos. Exprese este valor en gigabytes (GB). Consulte [Optimización del uso de memoria](storage-use-azcopy-optimize.md#optimize-memory-use). |
| AZCOPY_CACHE_PROXY_LOOKUP | De forma predeterminada, AzCopy en Windows almacenará en caché las búsquedas del servidor proxy en el nivel de nombre de host, sin tener en cuenta la dirección URL. Establezca en cualquier otro valor que no sea "true" para deshabilitar la memoria caché. |
| AZCOPY_CONCURRENCY_VALUE | Especifica el número de solicitudes simultáneas que pueden producirse. Puede usar esta variable para aumentar el rendimiento. Si el equipo tiene menos de cinco CPU, el valor de esta variable se establece en `32`. En caso contrario, el valor predeterminado es igual a 16 multiplicado por el número de CPU. El valor máximo predeterminado de esta variable es `3000`, pero puede establecerlo manualmente en un valor superior o inferior. Consulte [Aumento de la simultaneidad](storage-use-azcopy-optimize.md#increase-concurrency). |
| AZCOPY_CONCURRENT_FILES | Invalida el número aproximado de archivos que están en curso en un momento dado al controlar el número de archivos para los que se inician simultáneamente las transferencias. |
| AZCOPY_CONCURRENT_SCAN | Determina el grado máximo de paralelismo empleado en un examen. Solo afecta a los enumeradores paralelizados, como Azure Files blobs y los sistemas de archivos locales. |
| AZCOPY_DEFAULT_SERVICE_API_VERSION | Invalida la versión de la API de servicio para que AzCopy pueda dar cabida a entornos personalizados como Azure Stack. |
| AZCOPY_DISABLE_HIERARCHICAL_SCAN | Solo se aplica cuando el origen es Azure Blobs. El examen simultáneo es más rápido, pero emplea la API de lista jerárquica, lo cual puede dar lugar a un aumento de las operaciones de E/S y el costo. Especifique "true" para ahorrar en costos, sacrificando el rendimiento. |
| AZCOPY_JOB_PLAN_LOCATION | Invalida la ubicación en la que se almacenan los archivos del plan de trabajo (que se emplean para hacer un seguimiento del progreso y reanudar la actividad) con el fin de evitar que un disco se llene. |
| AZCOPY_LOG_LOCATION | Invalida la ubicación en la que se almacenan los archivos de registro con el fin de evitar que un disco se llene. |
| AZCOPY_MSI_CLIENT_ID | La identidad de cliente para una identidad administrada asignada por el usuario. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `MSI`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_MSI_OBJECT_ID | El identificador de objeto de la identidad administrada asignada por el usuario. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `MSI`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_MSI_RESOURCE_STRING | El identificador del recurso de la identidad administrada asignada por el usuario. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_PACE_PAGE_BLOBS | ¿Se debería ajustar automáticamente el rendimiento de los blobs en páginas para que coincidan con los límites del servicio? El valor predeterminado es true. Para deshabilitarlo, establezca el valor en "false". |
| AZCOPY_PARALLEL_STAT_FILES | Hace que AzCopy busque propiedades de archivo en "subprocesos" simultáneos al examinar el sistema de archivos local.  Los subprocesos proceden del grupo definido por AZCOPY_CONCURRENT_SCAN.  Si el valor se establece en "true", puede mejorar el rendimiento del examen en Linux.  No es necesario ni recomendable hacerlo en Windows. |
| AZCOPY_SHOW_PERF_STATES | Si se establece cualquier valor, la salida en pantalla incluirá el número de fragmentos en función del estado. |
| AZCOPY_SPA_APPLICATION_ID | El identificador de aplicación del registro de aplicación de la entidad de servicio. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `SPN`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_SPA_CERT_PASSWORD | La contraseña de un certificado. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `SPN`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_SPA_CERT_PATH | La ruta de acceso relativa o completa de un archivo de certificado. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `SPN`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_SPA_CLIENT_SECRET | Secreto del cliente. Se usa cuando `AZCOPY_AUTO_LOGIN_TYPE` se establece en `SPN`. Consulte [Autorización sin un almacén secreto](storage-use-azcopy-authorize-azure-active-directory.md#authorize-without-a-secret-store). |
| AZCOPY_TENANT_ID | El identificador de inquilino de Azure Active Directory que se usa para el inicio de sesión interactivo de dispositivos compatibles con OAuth. Esta variable solo se usa para el inicio de sesión automático. Utilice la marca de línea de comandos en su lugar para invocar el comando de inicio de sesión. |
| AZCOPY_TUNE_TO_CPU | Establezca el valor en "false" para evitar que AzCopy tenga en cuenta el uso de CPU al ajustar automáticamente el nivel de simultaneidad (por ejemplo, en el comando de punto de referencia). |
| AZCOPY_USER_AGENT_PREFIX | Agregue un prefijo al agente de usuario de AzCopy predeterminado, que se usa con fines de telemetría. Se insertará un espacio automáticamente. |
| GOOGLE_APPLICATION_CREDENTIALS | La ruta de acceso absoluta al archivo de clave de la cuenta de servicio proporciona una clave para autorizar el acceso con Google Cloud Storage. [Copia de datos de Google Cloud Storage a Azure Storage con AzCopy (versión preliminar)](storage-ref-azcopy-configuration-settings.md) |
| HTTPS_PROXY | Configura los valores de proxy para AzCopy. Establezca esta variable en la dirección IP y el número de puerto del proxy. Por ejemplo, `xx.xxx.xx.xxx:xx`. Si ejecuta AzCopy en Windows, AzCopy detecta automáticamente la configuración de proxy, por lo que no tiene que usar esta opción en Windows. Si decide usar esta opción en Windows, invalidará la detección automática. Consulte [Configuración de proxy](#configure-proxy-settings). |


## <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Para configurar las opciones de proxy para AzCopy, establezca la variable de entorno `HTTPS_PROXY`. Si ejecuta AzCopy en Windows, AzCopy detecta automáticamente la configuración de proxy, por lo que no tiene que usar esta opción en Windows. Si decide usar esta opción en Windows, invalidará la detección automática.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | En un símbolo del sistema, use: `set HTTPS_PROXY=<proxy IP>:<proxy port>`<br> En PowerShell, use: `$env:HTTPS_PROXY="<proxy IP>:<proxy port>"`|
| **Linux** | `export HTTPS_PROXY=<proxy IP>:<proxy port>` |
| **macOS** | `export HTTPS_PROXY=<proxy IP>:<proxy port>` |

En la actualidad, AzCopy no admite servidores proxy que requieren autenticación con NTLM o Kerberos.

### <a name="bypassing-a-proxy"></a>Omisión de un proxy

Si está ejecutando AzCopy en Windows y desea indicarle que _no_ use ningún proxy en absoluto (en lugar de detectar automáticamente la configuración), use estos comandos. Con esta configuración, AzCopy no buscará ni intentará usar ningún proxy.

| Sistema operativo | Entorno | Comandos:  |
|--------|-----------|----------|
| **Windows** | Símbolo del sistema (CMD) | `set HTTPS_PROXY=dummy.invalid` <br>`set NO_PROXY=*`|
| **Windows** | PowerShell | `$env:HTTPS_PROXY="dummy.invalid"` <br>`$env:NO_PROXY="*"`<br>|

En otros sistemas operativos, simplemente deje la variable HTTPS_PROXY sin establecer si no desea usar ningún proxy.

## <a name="see-also"></a>Consulte también

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Optimización del rendimiento de AzCopy v10 con Azure Storage](storage-use-azcopy-optimize.md)
- [Solución de problemas de AzCopy v10 en Azure Storage mediante el uso de archivos de registro](storage-use-azcopy-configure.md)
