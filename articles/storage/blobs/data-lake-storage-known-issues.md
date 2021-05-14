---
title: Problemas conocidos con Azure Data Lake Storage Gen2 | Microsoft Docs
description: Más información sobre las limitaciones y problemas conocidos con Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 9a7fb228763870792765c954589b347a18c560b5
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758976"
---
# <a name="known-issues-with-azure-data-lake-storage-gen2"></a>Problemas conocidos con Azure Data Lake Storage Gen2

En este artículo se describen las limitaciones y problemas conocidos con Azure Data Lake Storage Gen2.

## <a name="supported-blob-storage-features"></a>Características de Blob Storage compatibles

Ahora, un número creciente de características de Blob Storage funciona con cuentas que tienen un espacio de nombres jerárquico. Para obtener una lista completa, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="supported-azure-service-integrations"></a>Integraciones de servicios de Azure admitidos

Azure Data Lake Storage Gen2 admite varios servicios de Azure que se pueden usar para ingerir datos, realizar análisis y crear representaciones visuales. Para ver una lista de los servicios de Azure admitidos, consulte el documento en que se indica cuáles son los [servicios de Azure que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

Consulte [Servicios de Azure que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

## <a name="supported-open-source-platforms"></a>Plataformas de código abierto compatibles

Varias plataformas de código abierto son compatibles con Data Lake Storage Gen2. Para ver una lista completa, consulte [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

Consulte [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

## <a name="blob-storage-apis"></a>API de Blob Storage

Las API de Blob y Data Lake Storage Gen2 API pueden operar en los mismos datos.

En esta sección se describen los problemas y las limitaciones del uso de las API de blob y de las API de Data Lake Storage Gen2 para operar en los mismos datos.

* No puede usar la API de blob y las API de Data Lake Storage para escribir en la misma instancia de un archivo. Si escribe en un archivo con la API de Data Lake Storage Gen2, los bloques de ese archivo no estarán visibles para las llamadas a la API de blob [Get Block List](/rest/api/storageservices/get-block-list). La única excepción es cuando se usa para sobrescribir algún elemento. Puede sobrescribir un archivo o blob mediante cualquier API.

* Cuando use la operación [List Blobs](/rest/api/storageservices/list-blobs) (Enumerar blobs) sin especificar un delimitador, los resultados incluirán tanto los directorios como los blobs. Si decide usar un delimitador, use solo una barra diagonal (`/`). Este es el único delimitador admitido.

* Si usa la API [Delete Blob](/rest/api/storageservices/delete-blob) para eliminar un directorio, solo se eliminará si está vacío. Esto significa que no puede utilizar la API de blob para eliminar los directorios de forma recursiva.

Las siguientes API REST de blob no se admiten:

* [Put Blob (Page)](/rest/api/storageservices/put-blob) [Poner blob (página)]
* [Put Page](/rest/api/storageservices/put-page)
* [Get Page Ranges](/rest/api/storageservices/get-page-ranges) (Conseguir intervalos de páginas)
* [Incremental Copy Blob](/rest/api/storageservices/incremental-copy-blob) (Copia incremental del blob)
* [Put Page from URL](/rest/api/storageservices/put-page-from-url) (Poner página de dirección URL)

No se admiten discos de máquina virtual no administrados en cuentas que tengan un espacio de nombres jerárquico. Si desea habilitar un espacio de nombres jerárquico en una cuenta de almacenamiento, coloque los discos de máquina virtual no administrados en una cuenta de almacenamiento que no tenga habilitada la característica de espacio de nombres jerárquico.

<a id="api-scope-data-lake-client-library"></a>

## <a name="support-for-setting-access-control-lists-acls-recursively"></a>Compatibilidad con la configuración de listas de control de acceso (ACL) de forma recursiva

La capacidad de aplicar cambios de ACL de forma recursiva desde el directorio principal a los elementos secundarios está disponible con carácter general. En la versión actual de esta funcionalidad, puede aplicar cambios de ACL mediante PowerShell, la CLI de Azure y el SDK de .NET, Java y Python. Todavía no está disponible la compatibilidad con Azure Portal o el Explorador de Azure Storage.

<a id="known-issues-tools"></a>

## <a name="azcopy"></a>AzCopy

Use solo la versión más reciente de AzCopy ([AzCopy v10](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json)). No se admiten versiones anteriores de AzCopy, como AzCopy v8.1.

<a id="storage-explorer"></a>

## <a name="azure-storage-explorer"></a>Explorador de Azure Storage

Use solo las versiones `1.6.0` o superiores.

<a id="explorer-in-portal"></a>

## <a name="storage-explorer-in-the-azure-portal"></a>Explorador de Storage en Azure Portal

Las ACL aún no se admiten.

<a id="third-party-apps"></a>

## <a name="third-party-applications"></a>Aplicaciones de terceros

Las aplicaciones de terceros que usan las API REST para funcionar seguirán funcionando si se usan con Data Lake Storage Gen2. Lo más probable es que las aplicaciones que llaman a las API de Blob funcionen.

## <a name="access-control-lists-acl-and-anonymous-read-access"></a>Listas de control de acceso (ACL) y acceso de lectura anónimo

Si se ha concedido [acceso de lectura anónimo](./anonymous-read-access-configure.md) a un contenedor, las ACL no tendrán ningún efecto en ese contenedor o los archivos de ese contenedor.

## <a name="diagnostic-logs"></a>Registros de diagnóstico

Todavía no se admite la configuración de los días de retención, pero puede eliminar los registros manualmente mediante cualquier herramienta compatible como Explorador de Azure Storage, REST o un SDK.

## <a name="lifecycle-management-policies-with-premium-tier-for-azure-data-lake-storage"></a>Directivas de administración del ciclo de vida con nivel prémium para Azure Data Lake Storage

Los datos que se almacenan en el nivel prémium no se pueden mover entre los niveles de acceso frecuente, esporádico y de archivo. Sin embargo, puede copiar los datos del nivel prémium al nivel de acceso frecuente en otra cuenta.

## <a name="windows-azure-storage-blob-wasb-driver-unsupported-with-data-lake-storage-gen2"></a>Controlador Windows Azure Storage Blob (WASB) (no compatible con Data Lake Storage Gen2)

Actualmente, el controlador WASB, que se diseñó para funcionar solo con la API Blob, experimenta problemas en algunos escenarios comunes. En concreto, cuando es un cliente en una cuenta de almacenamiento habilitada para espacios de nombres jerárquicos. El acceso multiprotocolo en Data Lake Storage no mitigará estos problemas. 

Por el momento (y probablemente en un futuro inmediato), no se ofrecerá compatibilidad para el controlador WASB como cliente en una cuenta de almacenamiento habilitada para espacios de nombres jerárquicos. En su lugar, se recomienda que utilice el controlador de [Azure Blob File System (ABFS)](data-lake-storage-abfs-driver.md) en su entorno de Hadoop. Si está intentando migrar de un entorno de Hadoop local con una versión anterior a la rama de Hadoop 3, abra una incidencia de soporte técnico de Azure para que podamos ponernos en contacto con usted para indicarle el camino correcto para usted y su organización.