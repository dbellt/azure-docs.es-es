---
title: Introducción a las cuentas de almacenamiento
titleSuffix: Azure Storage
description: Obtenga información sobre los distintos tipos de cuentas de almacenamiento de Azure Storage. Revise la nomenclatura de las cuentas, los niveles de rendimiento, los niveles de acceso, la redundancia, el cifrado, los puntos de conexión y mucho más.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/19/2021
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: d4874ad6688fa85f0c511632498938817bb218f7
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714207"
---
# <a name="storage-account-overview"></a>Introducción a las cuentas de almacenamiento

Una cuenta de Azure Storage contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas, tablas y discos. La cuenta de almacenamiento proporciona un espacio de nombres único para los datos de Azure Storage que es accesible desde cualquier lugar del mundo a través de HTTP o HTTPS. Los datos de la cuenta de Azure Storage son duraderos y altamente disponibles, seguros y escalables a gran escala.

Para obtener información sobre cómo crear una cuenta de Azure Storage, consulte [Crear una cuenta de almacenamiento](storage-account-create.md).

## <a name="types-of-storage-accounts"></a>Tipos de cuentas de almacenamiento

Azure Storage ofrece varios tipos de cuentas de almacenamiento. Cada tipo admite diferentes características y tiene su propio modelo de precios. Tenga en cuenta estas diferencias antes de crear una cuenta de almacenamiento para determinar el tipo de cuenta más adecuada para sus aplicaciones.

En la tabla siguiente se describen los tipos de cuentas de almacenamiento recomendados por Microsoft para la mayoría de los escenarios:

| Tipo de cuenta de almacenamiento | Servicios admitidos | Opciones de redundancia | Modelo de implementación | Uso |
|--|--|--|--|--|
| De uso general, estándar, v2 | Blob, File, Queue, Table y Data Lake Storage<sup>1</sup> | LRS, GRS, RA-GRS<br /><br />ZRS, GZRS, RA-GZRS<sup>2</sup> | Resource Manager<sup>3</sup> | tipo de cuenta de almacenamiento básico para blobs, archivos, colas y tablas. Se recomienda para la mayoría de los escenarios con Azure Storage. |
| Blobs en bloques prémium<sup>4</sup> | Solo blobs en bloques | LRS<br /><br />ZRS<sup>2</sup> | Resource Manager<sup>3</sup> | Cuentas de almacenamiento con características de rendimiento Premium para blobs en bloques y en anexos. Recomendado para escenarios con altas tasas de transacciones, que utilizan objetos más pequeños o que requieren una latencia de almacenamiento constantemente baja.<br />[Más información...](../blobs/storage-blob-performance-tiers.md) |
| Recursos compartido de archivos prémium<sup>4</sup> | Recursos compartidos de archivos únicamente | LRS<br /><br />ZRS<sup>2</sup> | Resource Manager<sup>3</sup> | Cuentas de almacenamiento solo para archivos con características de rendimiento superiores. Se recomienda para empresas y aplicaciones de escalado de alto rendimiento.<br />[Más información...](../files/storage-files-planning.md#management-concepts) |
| Blobs en páginas prémium<sup>4</sup> | Solo blobs en páginas | LRS | Resource Manager<sup>3</sup> | Tipo de cuenta de almacenamiento prémium solo para blobs en páginas.<br />[Más información...](../blobs/storage-blob-pageblob-overview.md) |

<sup>1</sup> Azure Data Lake Storage es un conjunto de funcionalidades dedicadas al análisis de macrodatos basado en Azure Blob Storage. Data Lake Storage solo se admite en las cuentas de almacenamiento de uso general v2 con un espacio de nombres jerárquico habilitado. Para obtener más información sobre Data Lake Storage Gen2, consulte [Introducción a Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md).

<sup>2</sup> El almacenamiento con redundancia de zona (ZRS) y el almacenamiento con redundancia de zona geográfica (GZRS/RA-GZRS) solo están disponibles en cuentas de uso general estándar v2, de blobs en bloques prémium y de recursos compartido de archivos prémium en determinadas regiones. Para más información sobre las opciones de redundancia de Azure Storage, consulte [Redundancia de Azure Storage](storage-redundancy.md).

<sup>3</sup> Azure Resource Manager es el modelo de implementación recomendado para los recursos de Azure, incluidas las cuentas de almacenamiento. Para obtener más información, consulte [Información general de Resource Manager](../../azure-resource-manager/management/overview.md).

<sup>4</sup> Las cuentas de almacenamiento de un nivel de rendimiento prémium usan discos de estado sólido (SSD) para una baja latencia baja y un alto rendimiento.

También se admiten cuentas de almacenamiento heredadas. Para obtener más información, consulte [Tipos de cuentas de almacenamiento heredadas](#legacy-storage-account-types).

## <a name="storage-account-endpoints"></a>Puntos de conexión de cuenta de almacenamiento

Una cuenta de almacenamiento le proporciona un espacio de nombres único en Azure para sus datos. Cada objeto que almacena en Azure Storage tiene una dirección que incluye su nombre de cuenta único. La combinación del nombre de la cuenta y el punto de conexión del servicio de Azure Storage forma los puntos de conexión de su cuenta de almacenamiento.

Cuando especifique un nombre para la cuenta de almacenamiento, tenga en cuenta estas reglas:

- Los nombres de las cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y solo pueden incluir números y letras en minúscula.
- El nombre de la cuenta de almacenamiento debe ser único dentro de Azure. No puede haber dos cuentas de almacenamiento con el mismo nombre.

En la tabla siguiente se enumeran los formatos de los puntos de conexión para cada uno de los servicios de Azure Storage.

| Servicio de Storage | Punto de conexión |
|--|--|
| Blob Storage | `https://<storage-account>.blob.core.windows.net` |
| Data Lake Storage Gen2 | `https://<storage-account>.dfs.core.windows.net` |
| Azure Files | `https://<storage-account>.file.core.windows.net` |
| Queue Storage | `https://<storage-account>.queue.core.windows.net` |
| Table Storage | `https://<storage-account>.table.core.windows.net` |

Construya la dirección URL para acceder a un objeto de una cuenta de almacenamiento anexando la ubicación de este al punto de conexión. Por ejemplo, la dirección URL de un blob será similar a la siguiente:

`http://*mystorageaccount*.blob.core.windows.net/*mycontainer*/*myblob*`

También puede configurar una cuenta de almacenamiento para usar un dominio personalizado para los blobs. Para obtener más información, consulte [Configuración de un nombre de dominio personalizado para una cuenta de Azure Storage](../blobs/storage-custom-domain-name.md).  

## <a name="migrating-a-storage-account"></a>Migración de una cuenta de almacenamiento

En la tabla siguiente se resumen y se proporcionan instrucciones para mover, actualizar o migrar una cuenta de almacenamiento:

| Escenario de migración | Detalles |
|--|--|
| Traslado de una cuenta de almacenamiento a otra suscripción | Azure Resource Manager brinda opciones para cambiar de suscripción cualquier recurso. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md). |
| Traslado de una cuenta de almacenamiento a una otro grupo de recursos | Azure Resource Manager brinda opciones para cambiar de grupo de recursos cualquier recurso. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md). |
| Traslado de una cuenta de almacenamiento a otra región | Para mover una cuenta de almacenamiento, cree una copia de esta en otra región. Luego, mueva los datos a esa cuenta mediante AzCopy u otra herramienta de su elección. Para más información, consulte [Traslado de una cuenta de Azure Storage a otra región](storage-account-move.md). |
| Actualización a una cuenta de almacenamiento de uso general v2 | Todas las cuentas de almacenamiento de uso general v1 o las cuentas de almacenamiento de blobs se pueden actualizar y convertirse en cuentas de almacenamiento de uso general v2. Tenga en cuenta que esta acción no se puede deshacer. Para más información, consulte [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md). |
| Migración de una cuenta de almacenamiento clásica a una implementada con Azure Resource Manager | El modelo de implementación de Azure Resource Manager es superior al modelo de implementación clásica en cuanto a funcionalidad, escalabilidad y seguridad. Para más información sobre cómo migrar una cuenta de almacenamiento de implementación clásica a una de implementación con Azure Resource Manager, consulte la sección [Migración de cuentas de almacenamiento](../../virtual-machines/migration-classic-resource-manager-overview.md#migration-of-storage-accounts) del artículo **Migración compatible con la plataforma de recursos de IaaS del modelo clásico al de Azure Resource Manager**. |

## <a name="transferring-data-into-a-storage-account"></a>Transferencia de datos a una cuenta de almacenamiento

Microsoft proporciona servicios y utilidades para importar datos desde dispositivos de almacenamiento local o proveedores de almacenamiento en la nube de terceros. La solución que use dependerá de la cantidad de datos que vaya a transferir. Para obtener más información, consulte [Introducción a la migración de Azure Storage](storage-migration-overview.md).

## <a name="storage-account-encryption"></a>Cifrado de la cuenta de almacenamiento

Todos los datos de su cuenta de almacenamiento se cifran automáticamente en el servicio. Para obtener más información sobre cifrado y la administración de claves, consulte [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md).

## <a name="storage-account-billing"></a>Facturación de la cuenta de almacenamiento

Azure Storage se factura en función del uso que se haga de la cuenta de almacenamiento. Todos los objetos de una cuenta de almacenamiento se facturan juntos como un grupo. Para calcular los costos de almacenamiento se tienen en cuenta los siguientes factores:

- **Región** hace referencia a la región geográfica en la que radica la cuenta.
- **Tipo de cuenta** hace referencia al tipo de cuenta de almacenamiento que se usa.
- **Nivel de acceso** hace referencia al patrón de uso de datos que ha especificado para la cuenta de uso general v2 o la cuenta de Blob Storage.
- **Capacidad** hace referencia a la cuantificación de la asignación correspondiente a la cuenta de almacenamiento que se usa para almacenar datos.
- La **redundancia** determina cuántas copias de los datos se conservan al mismo tiempo, y en qué ubicaciones.
- Las **transacciones** se refieren a todas las operaciones de lectura y escritura en Azure Storage.
- La **salida de los datos** se refiere a los datos transferidos fuera de una región de Azure. Cuando una aplicación que no se ejecuta en la misma región que su cuenta de almacenamiento accede a los datos de esta, se le cobra por la salida de los datos. Para obtener información sobre el uso de grupos de recursos para agrupar los datos y servicios en la misma región a fin de limitar los cargos de salida, vea [¿Qué es un grupo de recursos de Azure?](/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management#what-is-an-azure-resource-group).

La página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/) proporciona información detallada sobre los precios en función del tipo de cuenta, de la capacidad de almacenamiento, de la replicación y de las transacciones. La página [Detalles de precios de transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/) proporciona información detallada sobre los precios para la salida de datos. Puede usar la [Calculadora de precios de Azure Storage](https://azure.microsoft.com/pricing/calculator/?scenario=data-management) para ayudarle a calcular los costos.

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="legacy-storage-account-types"></a>Tipos de cuentas de almacenamiento heredadas

En la tabla siguiente se describen los tipos de cuentas de almacenamiento heredadas. Microsoft no recomienda estos tipos de cuenta, pero se pueden usar en determinados escenarios:

| Tipo de cuenta de almacenamiento heredada | Servicios admitidos | Opciones de redundancia | Modelo de implementación | Uso |
|--|--|--|--|--|
| De uso general, estándar, v1 | Blob, File, Queue, Table y Data Lake Storage | LRS, GRS, RA-GRS | Resource Manager, clásico | Es posible que las cuentas de uso general v1 no incluyan las características más recientes o que no tengan los precios más bajos por gigabyte. Considere el uso para estos escenarios:<br /><ul><li>Sus aplicaciones requieren el modelo de implementación clásico de Azure.</li><li>Sus aplicaciones realizan una gran cantidad de transacciones o utilizan un ancho de banda significativo de replicación geográfica, pero no es necesario que tengan una gran capacidad. En este caso, las cuentas de uso general v1 pueden ser la opción más económica.</li><li>Usa una versión de la API REST de Azure Storage que es anterior a 14-02-2014 o una biblioteca cliente con una versión inferior a 4.x, y no puede actualizar la aplicación.</li></ul> |
| Almacenamiento de blobs estándar | Blob (solo blobs en bloques y blobs en anexos) | LRS, GRS, RA-GRS | Resource Manager | Microsoft recomienda usar cuentas estándar de uso general v2 en su lugar cuando sea posible. |

## <a name="next-steps"></a>Pasos siguientes

- [Cree una cuenta de almacenamiento](storage-account-create.md)
- [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md)
- [recuperar una cuenta de almacenamiento eliminada](storage-account-recover.md)