---
title: Introducción a las cuentas de almacenamiento
titleSuffix: Azure Storage
description: Obtenga información sobre los distintos tipos de cuentas de almacenamiento de Azure Storage. Revise la nomenclatura de las cuentas, los niveles de rendimiento, los niveles de acceso, la redundancia, el cifrado, los puntos de conexión y mucho más.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/02/2021
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: d7eca7d8f3cd40f4a3961f0ac478fba290be3041
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279670"
---
# <a name="storage-account-overview"></a>Introducción a las cuentas de almacenamiento

Una cuenta de Azure Storage contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas, tablas y discos. La cuenta de almacenamiento proporciona un espacio de nombres único para los datos de Azure Storage que es accesible desde cualquier lugar del mundo a través de HTTP o HTTPS. Los datos de la cuenta de Azure Storage son duraderos y altamente disponibles, seguros y escalables a gran escala.

Para obtener información sobre cómo crear una cuenta de Azure Storage, consulte [Crear una cuenta de almacenamiento](storage-account-create.md).

## <a name="types-of-storage-accounts"></a>Tipos de cuentas de almacenamiento

Azure Storage ofrece varios tipos de cuentas de almacenamiento. Cada tipo admite diferentes características y tiene su propio modelo de precios. Tenga en cuenta estas diferencias antes de crear una cuenta de almacenamiento para determinar el tipo de cuenta más adecuada para sus aplicaciones. Estos son los tipos de cuentas de almacenamiento:

- **Cuentas de uso general v2**: tipo de cuenta de almacenamiento básico para blobs, archivos, colas y tablas. Se recomienda para la mayoría de los escenarios con Azure Storage.
- **Cuentas de uso general v1**: tipo de cuenta heredada para blobs, archivos, colas y tablas. Cuando sea posible, utilice en su lugar las cuentas de uso general v2.
- **Cuentas BlockBlobStorage**: cuentas de almacenamiento con características de rendimiento prémium para blobs en bloques y en anexos. Recomendado para escenarios con altas tasas de transacciones, que utilizan objetos más pequeños o que requieren una latencia de almacenamiento constantemente baja.
- **Cuentas FileStorage**: cuentas de almacenamiento solo para archivos con características de rendimiento superiores. Se recomienda para empresas y aplicaciones de escalado de alto rendimiento.
- **Cuentas BlobStorage**: cuentas de almacenamiento de solo blobs heredadas. Cuando sea posible, utilice en su lugar las cuentas de uso general v2.

En la tabla siguiente se describen los tipos de cuentas de almacenamiento, los servicios que admiten y los modelos de implementación admitidos en cada tipo de cuenta:

| Tipo de cuenta de almacenamiento | Servicios admitidos | Opciones de redundancia | Modelo de implementación<sup>1</sup> |
|--|--|--|--|
| Uso general v2 | Blob, Archivo, Cola, Tabla, Disco y Data Lake Gen2<sup>2</sup> | LRS, GRS, RA-GRS, ZRS, GZRS, RA-GZRS<sup>3</sup> | Resource Manager |
| Uso general v1 | Blob, archivo, cola, tabla y disco | LRS, GRS, RA-GRS | Resource Manager, clásico |
| BlockBlobStorage | Blob (solo blobs en bloques y blobs en anexos) | LRS, ZRS<sup>3</sup> | Resource Manager |
| FileStorage | Solo archivo | LRS, ZRS<sup>3</sup> | Resource Manager |
| BlobStorage | Blob (solo blobs en bloques y blobs en anexos) | LRS, GRS, RA-GRS | Resource Manager |

<sup>1</sup>Se recomienda usar el modelo de implementación de Azure Resource Manager. Las cuentas de almacenamiento que usan el modelo de implementación clásico aún se pueden crear en algunas ubicaciones; asimismo, se seguirán admitiendo las cuentas clásicas ya existentes. Para obtener más información, consulte [La implementación de Azure Resource Manager frente a la implementación clásica: los modelos de implementación y el estado de los recursos](../../azure-resource-manager/management/deployment-models.md).

<sup>2</sup>Azure Data Lake Storage Gen2 es un conjunto de capacidades dedicadas al análisis de macrodatos basado en Azure Blob Storage. Data Lake Storage Gen2 solo se admite en las cuentas de almacenamiento de uso general V2 con un espacio de nombres jerárquico habilitado. Para más información sobre Data Lake Storage Gen2, consulte [Introducción a Azure Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md).

<sup>3</sup>El almacenamiento con redundancia de zona (ZRS) y el almacenamiento con redundancia de zona geográfica (GZRS/RA-GZRS) solo están disponibles en las cuentas estándar de uso general V2, BlockBlobStorage y FileStorage en determinadas regiones. Para más información sobre las opciones de redundancia de Azure Storage, consulte [Redundancia de Azure Storage](storage-redundancy.md).

### <a name="storage-account-redundancy"></a>Redundancia de cuenta de almacenamiento

Entre las opciones de redundancia para una cuenta de almacenamiento se incluyen las siguientes:

- **Almacenamiento con redundancia local (LRS)** : una estrategia de redundancia simple y de bajo costo. Los datos se copian de forma sincrónica tres veces dentro de una sola ubicación física en la región primaria.
- **Almacenamiento con redundancia de zona (ZRS)** : redundancia para escenarios que requieren alta disponibilidad. Los datos se copian de forma sincrónica en tres zonas de disponibilidad de Azure en la región primaria.
- **Almacenamiento con redundancia geográfica (GRS)** : redundancia entre regiones para protegerse frente a las interrupciones regionales. Los datos se copian sincrónicamente tres veces en la región primaria y, luego, se replican de forma asincrónica en la región secundaria. Para obtener acceso de lectura a los datos de la región secundaria, habilite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).
- **Almacenamiento con redundancia de zona geográfica (GZRS)** : redundancia para escenarios que requieren alta disponibilidad y máxima durabilidad. Los datos se copian sincrónicamente en tres zonas de disponibilidad de Azure en la región primaria y, luego, se copian de forma asincrónica en la región secundaria. Para obtener acceso de lectura a los datos de la región secundaria, habilite el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

Para más información sobre las opciones de redundancia en Azure Storage, consulte [Redundancia de Azure Storage](storage-redundancy.md).

### <a name="general-purpose-v2-accounts"></a>Cuentas de uso general v2.

Las cuentas de almacenamiento de uso general v2 son compatibles con las últimas características de Azure Storage, e incorporan todas las funcionalidades de las cuentas de Blob Storage y de uso general v1. Las cuentas de uso general v2 ofrecen los precios de capacidad por gigabyte más bajos para Azure Storage, así como los precios de transacción más competitivos del sector. Las cuentas de almacenamiento de uso general v2 son compatibles con estos servicios de Azure Storage:

- Blobs (todos los tipos: en bloques, adjuntos, en páginas)
- Data Lake Gen2
- Archivos
- Discos
- Colas
- Tablas

> [!NOTE]
> Microsoft recomienda usar cuentas de almacenamiento de uso general v2 en la mayoría de los escenarios. Puede actualizar fácilmente una cuenta de Blob Storage o de uso general v1 a una cuenta de uso general v2 sin tiempo de inactividad y sin la necesidad de copiar datos. Sin embargo, la actualización no se puede deshacer.
>
> Para obtener más información sobre la actualización a una cuenta de uso general v2, consulte [Upgrade to a general-purpose v2 storage account](storage-account-upgrade.md) (Actualizar a una cuenta de almacenamiento de uso general v2).

Las cuentas de almacenamiento de uso general v2 ofrecen varios niveles de acceso para almacenar datos según sus patrones de uso. Para obtener más información, consulte [Access tiers for block blob data](#access-tiers-for-block-blob-data) (Niveles de acceso para los datos del blob en bloques).

### <a name="general-purpose-v1-accounts"></a>Cuentas de uso general v1

Las cuentas de almacenamiento de uso general v1 proporcionan acceso a todos los servicios de Azure Storage, pero puede que no incluyan las características más recientes o que no tengan los precios más bajos por gigabyte. Las cuentas de almacenamiento de uso general v1 son compatibles con estos servicios de Azure Storage:

- Blobs (todos los tipos)
- Archivos
- Discos
- Colas
- Tablas

Puede usar cuentas de uso general v1 en estos escenarios:

- Sus aplicaciones requieren el modelo de implementación clásico de Azure. Las cuentas de uso general v2 y las cuentas de Blob Storage solo admiten el modelo de implementación de Azure Resource Manager.

- Sus aplicaciones realizan una gran cantidad de transacciones o utilizan un ancho de banda significativo de replicación geográfica, pero no es necesario que tengan una gran capacidad. En este caso, las cuentas de uso general v1 pueden ser la opción más económica.

- Usa una versión de la [API de REST de servicios de almacenamiento](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) anterior al 14-02-2014 o una biblioteca de cliente con una versión inferior a la 4.x. No puede actualizar la aplicación.

> [!NOTE]
> Aunque Microsoft recomienda cuentas de uso general v2 para la mayoría de los escenarios, seguirá admitiendo cuentas de uso general v1 para clientes nuevos y existentes. Puede crear cuentas de almacenamiento de uso general v1 en nuevas regiones siempre que Azure Storage esté disponible en esas regiones. Microsoft no tiene actualmente ningún plan para dejar de ofrecer soporte técnico para las cuentas de uso general v1 y proporcionará un aviso previo de al menos un año antes de dejar de usar cualquier característica de Azure Storage. Microsoft seguirá proporcionando actualizaciones de seguridad para las cuentas de uso general v1, pero no se espera el desarrollo de ninguna característica nueva para este tipo de cuenta.
>
> A partir del 1 de octubre de 2020, el precio de las cuentas de uso general v1 en nuevas regiones de Azure Storage es equivalente a los precios de las cuentas de uso general v2 de esas regiones. Los precios en las regiones existentes de Azure Storage no han cambiado. Para más información sobre los precios de las cuentas de uso general v1 en una región específica, consulte la página de precios de Azure Storage. Elija su región y, junto a **Ofertas de precios**, seleccione **Otros**.

### <a name="blockblobstorage-accounts"></a>Cuentas BlockBlobStorage

Una cuenta BlockBlobStorage es una cuenta de almacenamiento especializada del nivel de rendimiento Premium que se usa para almacenar datos de objetos no estructurados como blobs en bloques o blobs en anexos. En comparación con las cuentas de uso general v2 y BlobStorage, las cuentas BlockBlobStorage ofrecen latencia baja y coherente y mayores tasas de transacción.

Las cuentas BlockBlobStorage no admiten actualmente los niveles de acceso frecuente, esporádico ni de archivo. Este tipo de cuenta de almacenamiento no admite blobs en páginas, tablas o colas.

### <a name="filestorage-accounts"></a>Cuentas FileStorage

Una cuenta FileStorage es una cuenta de almacenamiento especializada que se usa para almacenar y crear recursos compartidos de archivos premium. Este tipo de cuenta de almacenamiento admite archivos, pero no de blobs en bloques, blobs en anexos, blobs en páginas, tablas ni colas.

Las cuentas FileStorage ofrecen características dedicadas de rendimiento único, como la ampliación de IOPS. Para más información sobre estas características, consulte la sección sobre las [capas de almacenamiento de recursos compartidos de archivo](../files/storage-files-planning.md#storage-tiers) en la guía de planeación de archivos.

## <a name="naming-storage-accounts"></a>Nomenclatura de las cuentas de almacenamiento

Cuando especifique un nombre para la cuenta de almacenamiento, tenga en cuenta estas reglas:

- Los nombres de las cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y solo pueden incluir números y letras en minúscula.
- El nombre de la cuenta de almacenamiento debe ser único dentro de Azure. No puede haber dos cuentas de almacenamiento con el mismo nombre.

## <a name="performance-tiers"></a>Niveles de rendimiento

En función del tipo de cuenta de almacenamiento que cree, puede elegir entre los niveles de rendimiento Estándar y Premium. En la tabla siguiente se resumen los niveles de rendimiento disponibles para cada tipo de cuenta de almacenamiento.

| Tipo de cuenta de almacenamiento | Niveles de rendimiento admitidos |
|--|--|
| Uso general v2 | Estándar, Premium<sup>1</sup> |
| Uso general v1 | Estándar, Premium<sup>1</sup> |
| BlockBlobStorage | Premium |
| FileStorage | Premium |
| BlobStorage | Estándar |

<sup>1</sup>El rendimiento Premium de las cuentas de uso general v2 y v1 está disponible solo para blobs en páginas y en disco. El rendimiento Premium para blobs en bloque o en anexos solo está disponible en las cuentas de BlockBlobStorage. El rendimiento Premium para archivos solo está disponible en las cuentas de FileStorage.

### <a name="general-purpose-storage-accounts"></a>Cuentas de almacenamiento de uso general

Las cuentas de almacenamiento de uso general pueden configurarse para cada uno de los niveles de rendimiento siguientes:

- Un nivel de rendimiento estándar para almacenar blobs, archivos, tablas, colas y discos de máquina virtual de Azure. Para obtener más información sobre los objetivos de escalabilidad de cuentas de almacenamiento estándar, consulte [Objetivos de escalabilidad para cuentas de almacenamiento estándar](scalability-targets-standard-account.md).
- Un nivel de rendimiento Premium para almacenar los discos de máquina virtual no administrados. Microsoft recomienda usar discos administrados en lugar de discos no administrados con máquinas virtuales de Azure. Para obtener más información sobre los objetivos de escalabilidad del nivel de rendimiento premium, consulte [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas premium](../blobs/scalability-targets-premium-page-blobs.md).

### <a name="blockblobstorage-storage-accounts"></a>Cuentas de almacenamiento BlockBlobStorage

Las cuentas de almacenamiento BlockBlobStorage proporcionan un nivel de rendimiento premium para almacenar blobs en bloques y blobs en anexos. Para obtener más información, consulte [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en bloques premium](../blobs/scalability-targets-premium-block-blobs.md).

### <a name="filestorage-storage-accounts"></a>Cuentas de almacenamiento FileStorage

Las cuentas de almacenamiento FileStorage proporcionan un nivel de rendimiento prémium para los recursos compartidos de archivos de Azure. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Files](../files/storage-files-scale-targets.md).

## <a name="access-tiers-for-block-blob-data"></a>Niveles de acceso a datos de blob en bloques

Azure Storage ofrece diferentes opciones para obtener acceso a los datos de blob en bloques, en función de los patrones de uso. Cada nivel de acceso en Azure Storage está optimizado para un patrón particular de uso de datos. Al seleccionar el nivel de acceso adecuado a sus necesidades, puede almacenar sus datos de blob en bloques de la manera más rentable.

Los niveles de acceso disponibles son:

- Nivel de acceso **frecuente**. Este nivel está optimizado para el acceso frecuente de objetos en la cuenta de almacenamiento. Obtener acceso a los datos en el nivel de acceso frecuente es más rentable, aunque los costos de almacenamiento son más altos. Las nuevas cuentas de almacenamiento se crean en el nivel de acceso frecuente de forma predeterminada.
- Nivel de acceso **esporádico**. Este nivel está optimizado para almacenar grandes cantidades de datos a los que se accede con poca frecuencia y que llevan guardados al menos 30 días. El almacenamiento de datos en el nivel de acceso esporádico es más rentable, pero el acceso a esos datos puede ser algo más costoso que a los del nivel de acceso frecuente.
- Nivel de **archivo**. Este nivel está disponible solo para blobs en bloques individuales. Este nivel está optimizado para los datos que pueden tolerar varias horas de latencia de recuperación y que permanecerán en el nivel de almacenamiento de archivo durante un mínimo de 180 días. El nivel de archivo es la opción más rentable para almacenar datos. Sin embargo, el acceso a los datos es más caro que en el caso de los niveles de acceso frecuente o esporádico.

Si hay un cambio en el patrón de uso de datos, también se puede cambiar de nivel de acceso en cualquier momento. Para más información acerca de los niveles de acceso, consulte [Azure Blob Storage: niveles de almacenamiento de archivo, esporádico y frecuente](../blobs/storage-blob-storage-tiers.md).

En la tabla siguiente se muestran los niveles de acceso que están disponibles para los blobs en cada tipo de cuenta de almacenamiento.

| Tipo de cuenta de almacenamiento | Niveles de acceso admitidos |
|--|--|
| Uso general v2 | Frecuente, esporádico, archivo<sup>1</sup> |
| Uso general v1 | N/D |
| BlockBlobStorage | N/D |
| FileStorage | N/D |
| BlobStorage | Frecuente, esporádico, archivo<sup>1</sup> |

<sup>1</sup> El almacenamiento de archivo y el almacenamiento por niveles de blob solo admiten blobs en bloques. El nivel de archivo está disponible solo en el nivel de un blob individual y no en el nivel de cuenta de almacenamiento. Para obtener más información, vea [Niveles de acceso de Azure Blob Storage: frecuente, esporádico y archivo](../blobs/storage-blob-storage-tiers.md).

> [!IMPORTANT]
> Si cambia el nivel de acceso de una cuenta de almacenamiento o blob existente es posible que deba pagar cargos adicionales. Para obtener más información, vea [Facturación de la cuenta de almacenamiento](#storage-account-billing).

## <a name="storage-account-endpoints"></a>Puntos de conexión de cuenta de almacenamiento

Una cuenta de almacenamiento le proporciona un espacio de nombres único en Azure para sus datos. Cada objeto que almacena en Azure Storage tiene una dirección que incluye su nombre de cuenta único. La combinación del nombre de la cuenta y el punto de conexión del servicio de Azure Storage forma los puntos de conexión de su cuenta de almacenamiento.

En la tabla siguiente se enumeran los puntos de conexión para cada uno de los servicios de Azure Storage.

| Servicio de Storage | Punto de conexión |
|--|--|
| Blob Storage | `https://<storage-account>.blob.core.windows.net` |
| Azure Data Lake Storage Gen2 | `https://<storage-account>.dfs.core.windows.net` |
| Azure Files | `https://<storage-account>.file.core.windows.net` |
| Queue Storage | `https://<storage-account>.queue.core.windows.net` |
| Table Storage | `https://<storage-account>.table.core.windows.net` |

> [!NOTE]
> Las cuentas de almacenamiento de blob y de blob en bloques solo exponen el punto de conexión de Blob service.

Construya la dirección URL para acceder a un objeto de una cuenta de almacenamiento anexando la ubicación de este al punto de conexión. Por ejemplo, una dirección de blob podría tener este formato: http://*mystorageaccount*.blob.core.windows.net/*mycontainer*/*myblob*.

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

## <a name="copying-data-into-a-storage-account"></a>Copiar los datos en una cuenta de almacenamiento

Microsoft proporciona herramientas y bibliotecas para importar datos desde dispositivos de almacenamiento local o proveedores de almacenamiento en la nube de terceros. La solución que use dependerá de la cantidad de datos que vaya a transferir.

Al actualizar a una cuenta de uso general v1 a otra de uso general v2 o a una cuenta Blob Storage, los datos se migran automáticamente. Microsoft recomienda que se usen estos pasos para actualizar la cuenta. Sin embargo, si decide mover datos desde una cuenta de uso general v1 a una cuenta de Blob Storage, deberá migrar manualmente los datos mediante las herramientas y bibliotecas que se describen a continuación.

### <a name="azcopy"></a>AzCopy

AzCopy es una utilidad de línea de comandos de Windows diseñada para la copia de datos de alto rendimiento a y desde Azure Storage. AzCopy se puede usar para copiar datos en una cuenta de Blob Storage desde cuentas de almacenamiento de uso general ya existentes, o para cargar datos desde dispositivos de almacenamiento local. Para más información, vea [Transferencia de datos con la utilidad en línea de comandos AzCopy](./storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

### <a name="data-movement-library"></a>Data Movement Library

Azure Storage Data Movement Library for .NET se basa en el marco de movimiento de datos principal que se utiliza con AzCopy. La biblioteca está diseñada para operaciones de transferencia de datos de alto rendimiento, confiables y fáciles similares a AzCopy. Puede usar la biblioteca de movimiento de datos para aprovechar las características de AzCopy de forma nativa. Para más información, consulte [Azure Storage Data Movement Library for .NET](https://github.com/Azure/azure-storage-net-data-movement)

### <a name="rest-api-or-client-library"></a>API de REST o biblioteca de cliente

Puede crear una aplicación personalizada para migrar los datos desde una cuenta de almacenamiento de uso general v1 a una cuenta de Blob Storage. Use una de las bibliotecas de cliente de Azure o la API REST de servicios de Azure Storage. Azure Storage proporciona bibliotecas de cliente enriquecidas para varios lenguajes y aplicaciones como .NET, Java, C++, Node.JS, PHP, Ruby y Python. Las bibliotecas de cliente ofrecen capacidades avanzadas, como lógica de reintentos, registro y cargas paralelas. También se puede desarrollar directamente en la API de REST, a la que se puede llamar con cualquier lenguaje que genere solicitudes HTTP/HTTPS.

Para obtener más información sobre la API REST de Azure Storage, consulte [Azure Storage Services REST API Reference](/rest/api/storageservices/) (Referencia de la API REST de Azure Storage).

> [!IMPORTANT]
> Los blobs que usan el cifrado de cliente almacenan metadatos relacionados con el cifrado junto con el blob. Si copia un blob en el que se usa un cifrado de cliente, asegúrese de que la operación de copia conserva los metadatos del blob y, especialmente, los metadatos relacionados con el cifrado. Si copia un blob sin los metadatos de cifrado, su contenido no puede volver a recuperarse. Para más información acerca de los metadatos relacionados con el cifrado, consulte [Cifrado del lado de cliente y Almacén de claves de Azure para Microsoft Azure Storage](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="encryption"></a>Cifrado

Todos los datos de su cuenta de almacenamiento se cifran en el mismo servicio. Para obtener más información, consulte [Azure Storage Service Encryption para datos en reposo](storage-service-encryption.md).

## <a name="storage-account-billing"></a>Facturación de la cuenta de almacenamiento

Azure Storage se factura en función del uso que se haga de la cuenta de almacenamiento. Todos los objetos de una cuenta de almacenamiento se facturan juntos como un grupo. Para calcular los costos de almacenamiento se tienen en cuenta los siguientes factores:

- **Región** hace referencia a la región geográfica en la que radica la cuenta.
- **Tipo de cuenta** hace referencia al tipo de cuenta de almacenamiento que se usa.
- **Nivel de acceso** hace referencia al patrón de uso de datos que ha especificado para la cuenta de uso general v2 o la cuenta de Blob Storage.
- **Capacidad** hace referencia a la cuantificación de la asignación correspondiente a la cuenta de almacenamiento que se usa para almacenar datos.
- La **replicación** determina cuántas copias de los datos se conservan al mismo tiempo, y en qué ubicaciones.
- Las **transacciones** se refieren a todas las operaciones de lectura y escritura en Azure Storage.
- La **salida de los datos** se refiere a los datos transferidos fuera de una región de Azure. Cuando una aplicación que no se ejecuta en la misma región que su cuenta de almacenamiento accede a los datos de esta, se le cobra por la salida de los datos. Para obtener información sobre el uso de grupos de recursos para agrupar los datos y servicios en la misma región a fin de limitar los cargos de salida, vea [¿Qué es un grupo de recursos de Azure?](/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management#what-is-an-azure-resource-group).

La página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/) proporciona información detallada sobre los precios en función del tipo de cuenta, de la capacidad de almacenamiento, de la replicación y de las transacciones. La página [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/) proporciona información detallada sobre los precios para la salida de datos. Puede usar la [Calculadora de precios de Azure Storage](https://azure.microsoft.com/pricing/calculator/?scenario=data-management) para ayudarle a calcular los costes.

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Cree una cuenta de almacenamiento](storage-account-create.md)
- [Creación de una cuenta de almacenamiento de blobs en bloques](../blobs/storage-blob-create-account-block-blob.md)
- [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md)
- [recuperar una cuenta de almacenamiento eliminada](storage-account-recover.md)