---
title: Información general sobre la protección de datos
titleSuffix: Azure Storage
description: Las opciones de protección de datos que están disponibles para los datos de Blob Storage y Azure Data Lake Storage Gen2 permiten proteger los datos contra la eliminación y sobrescritura. Si necesita recuperar datos que se eliminaron o sobrescribieron, esta guía puede ayudarle a elegir la opción de recuperación más adecuada para su escenario.
services: storage
author: tamram
ms.service: storage
ms.date: 04/09/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: prishet
ms.subservice: common
ms.openlocfilehash: 90c83397089b77d30694041a37debc0731ea2a38
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304263"
---
# <a name="data-protection-overview"></a>Información general sobre la protección de datos

Azure Storage proporciona protección de datos para Blob Storage y Azure Data Lake Storage Gen2 que le ayudarán a prepararse para escenarios en los que necesite recuperar datos que se han eliminado o sobrescrito. Es importante pensar en la mejor manera de proteger los datos antes de que se produzca un incidente que podría ponerlos en peligro. Esta guía puede ayudarle a decidir por adelantado qué características de protección de datos requiere su escenario y cómo implementarlas. Si necesita recuperar datos que se eliminaron o sobrescribieron, esta introducción también incluye instrucciones sobre cómo proceder en función de su escenario.

En la documentación de Azure Storage, *protección de datos* trata sobre las estrategias para proteger la cuenta de almacenamiento y los datos que contiene contra la eliminación o modificación, o para restaurar los datos después de eliminarlos o modificarlos. Azure Storage también ofrece opciones para la *recuperación ante desastres*, incluidos varios niveles de redundancia para proteger los datos contra a interrupciones del servicio debido a problemas de hardware o desastres naturales y conmutación por error administrada por el cliente en caso de que el centro de datos de la región principal deje de estar disponible. Para obtener más información sobre cómo se protegen los datos frente a las interrupciones del servicio, consulte [Recuperación ante desastres](#disaster-recovery).

## <a name="recommendations-for-basic-data-protection"></a>Recomendaciones para la protección de datos básica

Si está buscando cobertura básica de protección de datos para su cuenta de almacenamiento y los datos que contiene, Microsoft recomienda realizar los siguientes pasos para comenzar:

- Configure un bloqueo de Azure Resource Manager en la cuenta de almacenamiento para protegerla de la eliminación o de los cambios de configuración. [Más información...](../common/lock-account-resource.md)
- Habilite la eliminación temporal de contenedores para la cuenta de almacenamiento con el fin de recuperar un contenedor eliminado y su contenido. [Más información...](soft-delete-container-enable.md)
- Guarde el estado de un blob a intervalos regulares:
  - En el caso de las cargas de trabajo de Blob Storage, habilite el control de versiones de blobs para guardar automáticamente el estado de los datos cada vez que se sobrescriba un blob. [Más información...](versioning-enable.md)
  - En el caso de las cargas de trabajo de Azure Data Lake Storage, realice instantáneas manuales para guardar el estado de los datos en un momento determinado. [Más información...](snapshots-overview.md)

Estas opciones de protección de datos se describen con más detalle en la sección siguiente, junto con otras para otros escenarios.

Para una introducción a los costos que conllevan estas características, consulte el [Resumen de las consideraciones de costos](#summary-of-cost-considerations).

## <a name="overview-of-data-protection-options"></a>Introducción a las opciones de protección de datos

En la tabla siguiente se resumen las opciones disponibles en Azure Storage para los escenarios comunes de protección de datos. Elija los escenarios correspondientes a su situación para obtener más información sobre las opciones disponibles. Tenga en cuenta que no todas las características están disponibles en este momento para las cuentas de almacenamiento que tienen habilitado un espacio de nombres jerárquico.

| Escenario | Opción de protección de datos | Recomendaciones | Ventajas de protección | Disponible para Data Lake Storage |
|--|--|--|--|--|
| Evitar que se elimine o modifique una cuenta de almacenamiento. | Bloqueo de Azure Resource Manager<br />[Más información...](../common/lock-account-resource.md) | Bloquee todas las cuentas de almacenamiento con un bloqueo de Azure Resource Manager a fin de impedir su eliminación. | Protege la cuenta de almacenamiento contra la eliminación o los cambios de configuración.<br /><br />No protege los contenedores o blobs de la cuenta contra la eliminación o sobrescritura. | Sí |
| Impedir que un contenedor y sus blobs se eliminen o modifiquen durante un intervalo determinado por el usuario. | Directiva de inmutabilidad de un contenedor<br />[Más información...](storage-blob-immutable-storage.md) | Establezca una directiva de inmutabilidad en un contenedor para proteger documentos críticos para la empresa; por ejemplo, con el fin de cumplir los requisitos de cumplimiento normativo o legal. | Protege un contenedor y sus blobs de cualquier eliminación y sobrescritura.<br /><br />Cuando está vigente una suspensión legal o una directiva de retención de duración limitada bloqueada, la cuenta de almacenamiento también está protegida contra la eliminación. Los contenedores para los que no se ha establecido ninguna directiva de inmutabilidad no están protegidos contra la eliminación. | Sí, en versión preliminar |
| Restaurar un contenedor eliminado en un intervalo específico. | Eliminación temporal de contenedores (versión preliminar)<br />[Más información...](soft-delete-container-overview.md) | Habilite la eliminación temporal de contenedores para todas las cuentas de almacenamiento, con un intervalo de retención mínimo de siete días.<br /><br />Habilite el control de versiones de blobs y la eliminación temporal de blobs junto con la eliminación temporal de contenedores para proteger blobs concretos de un contenedor.<br /><br />Almacene contenedores que requieran diferentes períodos de retención en cuentas de almacenamiento independientes. | Se puede restaurar un contenedor eliminado y su contenido dentro del período de retención.<br /><br />Solo se pueden restaurar las operaciones de nivel de contenedor (por ejemplo, [Eliminar contenedor](/rest/api/storageservices/delete-container)). La eliminación temporal de contenedores no permite restaurar un blob individual en el contenedor si dicho blob se eliminó. | Sí, en versión preliminar |
| Guardar automáticamente el estado de un blob en una versión anterior cuando se sobrescribe. | Control de versiones de blobs<br />[Más información...](versioning-overview.md) | Habilite el control de versiones de blobs, junto con la eliminación temporal de contenedores y la eliminación temporal de blobs, para las cuentas de almacenamiento en las que necesita protección óptima para los datos de blobs.<br /><br />Almacene datos de blobs que no requieran el control de versiones en una cuenta independiente para limitar los costos. | Cada operación de escritura de blobs crea una versión. La versión actual de un blob se puede restaurar a partir de una versión anterior si se elimina o sobrescribe la versión actual. | No |
| Restaurar una versión de blob o un blob eliminado en un intervalo especificado. | Eliminación temporal de blobs<br />[Más información...](soft-delete-blob-overview.md) | Habilite la eliminación temporal de blobs para todas las cuentas de almacenamiento, con un intervalo de retención mínimo de siete días.<br /><br />Habilite el control de versiones de blobs y la eliminación temporal de contenedores junto con la eliminación temporal de blobs para una protección óptima de los datos del blob.<br /><br />Almacene blobs que requieran diferentes períodos de retención en cuentas de almacenamiento independientes. | Se puede restaurar una versión de blob o un blob eliminado en el período de retención. | No |
| Restaurar un conjunto de blobs en bloques a un momento dado anterior. | Restauración a un momento dado<br />[Más información...](point-in-time-restore-overview.md) | Si quiere restaurar a un momento dado para revertir a un estado anterior, diseñe la aplicación a fin de que elimine los blobs en bloques individuales en lugar de eliminar los contenedores. | Un conjunto de blobs en bloques se puede revertir al estado que tenía en un punto concreto del pasado.<br /><br />Solo se revierten las operaciones realizadas en los blobs en bloques. No se revierten las operaciones realizadas en contenedores, blobs en páginas o blobs en anexos. | No |
| Guardar manualmente el estado de un blob en un momento dado. | Instantánea de blob<br />[Más información...](snapshots-overview.md) | Se recomienda como alternativa al control de versiones de blobs cuando el control de versiones no es adecuado para su escenario debido a los costos u otras consideraciones, o cuando la cuenta de almacenamiento tiene habilitado un espacio de nombres jerárquico. | Un blob se puede restaurar a partir de una instantánea si se sobrescribe el blob. Si se elimina el blob, también se eliminan las instantáneas. | Sí, en versión preliminar |
| Un blob se puede eliminar o sobrescribir, pero los datos se copian periódicamente en una segunda cuenta de almacenamiento. | Implemente su propia solución para copiar datos a una segunda cuenta mediante la replicación de objetos de Azure Storage o una herramienta, como AzCopy o Azure Data Factory. | Recomendado para una protección óptima contra acciones intencionadas inesperadas o escenarios impredecibles.<br /><br />Cree la segunda cuenta de almacenamiento en la misma región que la cuenta principal para evitar la generación de cargos de salida. | Los datos se pueden restaurar a partir de la segunda cuenta de almacenamiento si la cuenta principal se ve comprometida de alguna manera. | Se admiten AzCopy y Azure Data Factory.<br /><br />No se admite la replicación de objetos. |

## <a name="data-protection-by-resource-type"></a>Protección de datos por tipo de recurso

En la tabla siguiente se resumen las opciones de protección de datos de Azure Storage según los recursos protegidos.

| Opción de protección de datos | Protege una cuenta contra la eliminación | Protege un contenedor contra la eliminación | Protege un blob contra la eliminación | Protege un blob contra las sobrescrituras |
|--|--|--|--|--|
| Bloqueo de Azure Resource Manager | Sí | No<sup>1</sup> | No | No |
| Directiva de inmutabilidad de un contenedor | Sí<sup>2</sup> | Sí | Sí | Sí |
| Eliminación temporal de contenedores | No | Sí | No | No |
| Control de versiones de blobs<sup>3</sup> | No | No | Sí | Sí |
| Eliminación temporal de blobs<sup>3</sup> | No | No | Sí | Sí |
| Restauración a un momento dado<sup>3</sup> | No | No | Sí | Sí |
| Instantánea de blob | No | No | No | Sí |
| Implementación de una solución propia para la copia de datos a una segunda cuenta<sup>4</sup> | No | Sí | Sí | Sí |

<sup>1</sup> Un bloqueo de Azure Resource Manager no protege un contenedor de la eliminación.<br />
<sup>2</sup> Mientras está vigente una suspensión legal o una directiva de retención de duración limitada bloqueada para un contenedor, la cuenta de almacenamiento también está protegida contra la eliminación.<br />
<sup>3</sup> No se admite actualmente para las cargas de trabajo de Data Lake Storage.<br />
<sup>4</sup> AzCopy y Azure Data Factory son opciones compatibles con las cargas de trabajo de Blob Storage y Data Lake Storage. La replicación de objetos solo se admite para las cargas de trabajo de Blob Storage.<br />

## <a name="recover-deleted-or-overwritten-data"></a>Recuperación de datos eliminados o sobrescritos

Si necesita recuperar los datos que se han sobrescrito o eliminado, la forma de proceder dependerá de las opciones de protección de datos que haya habilitado y de los recursos afectados. En la tabla siguiente se describen las acciones que puede realizar para recuperar los datos.

| Recurso eliminado o sobrescrito | Posibles acciones de recuperación | Requisitos para la recuperación |
|--|--|--|
| Cuenta de almacenamiento | Tratar de recuperar la cuenta de almacenamiento eliminada.<br />[Más información...](../common/storage-account-recover.md) | La cuenta de almacenamiento se creó originalmente con el modelo de implementación de Azure Resource Manager y se eliminó en los últimos 14 días. No se ha creado una cuenta de almacenamiento con el mismo nombre desde que se eliminó la cuenta original. |
| Contenedor | Recuperar el contenedor eliminado temporalmente y su contenido.<br />[Más información...](soft-delete-container-enable.md) | La eliminación temporal del contenedor está habilitada y el período de retención de la eliminación temporal del contenedor todavía no ha expirado. |
| Contenedores y blobs | Restaurar datos a partir de una segunda cuenta de almacenamiento. | Todas las operaciones de contenedores y blobs se han replicado eficazmente en una segunda cuenta de almacenamiento. |
| Blob (cualquier tipo) | Restaurar un blob a partir de una versión anterior<sup>1</sup><br />[Más información...](versioning-enable.md) | El control de versiones de blobs está habilitado y el blob tiene una o más versiones anteriores. |
| Blob (cualquier tipo) | Recuperar un blob eliminado temporalmente<sup>1</sup><br />[Más información...](soft-delete-blob-enable.md) | La eliminación temporal de blobs está habilitada y el intervalo de retención de la eliminación temporal no ha expirado. |
| Blob (cualquier tipo) | Restaurar un blob a partir de una instantánea<br />[Más información...](snapshots-manage-dotnet.md) | El blob tiene una o varias instantáneas. |
| Conjunto de blobs en bloques | Recuperar un conjunto de blobs en bloques al estado en que estaban en un momento anterior en el tiempo<sup>1</sup><br />[Más información...](point-in-time-restore-manage.md) | La restauración a un momento dado está habilitada y el punto de restauración se encuentra en el intervalo de retención. La cuenta de almacenamiento no se ha puesto en peligro ni está dañada. |
| Versión de un blob | Recuperar una versión eliminada temporalmente<sup>1</sup><br />[Más información...](soft-delete-blob-enable.md) | La eliminación temporal de blobs está habilitada y el intervalo de retención de la eliminación temporal no ha expirado. |

<sup>1</sup> No se admite actualmente para las cargas de trabajo de Data Lake Storage.

## <a name="summary-of-cost-considerations"></a>Resumen de las consideraciones de costos

En la tabla siguiente se resumen las consideraciones de costos para las distintas opciones de protección de datos descritas en esta guía.

| Opción de protección de datos | Consideraciones sobre el costo |
|-|-|
| Bloqueo de Azure Resource Manager para una cuenta de almacenamiento | No se cobra por configurar un bloqueo en una cuenta de almacenamiento. |
| Directiva de inmutabilidad de un contenedor | No se cobra por configurar una directiva de inmutabilidad en un contenedor. |
| Eliminación temporal de contenedores | No se cobra por habilitar la eliminación temporal de contenedores para una cuenta de almacenamiento. Los datos de un contenedor eliminado temporalmente se facturan a la misma tarifa que los datos activos hasta que el contenedor se elimina permanentemente. |
| Control de versiones de blobs | No se cobra por habilitar el control de versiones de blobs para una cuenta de almacenamiento. Una vez habilitado el control de versiones de blobs, cada operación de escritura o eliminación que se realice en un blob de la cuenta creará una nueva versión, lo que puede generar mayores costos de capacidad.<br /><br />Una versión de blob se factura en función de los bloques o páginas únicos. Por lo tanto, los costos aumentan a medida que el blob base difiere de una versión determinada. Cambiar el nivel de un blob o de una versión de blob puede tener un impacto en la facturación. Para obtener más información, consulte la sección [Precios y facturación](versioning-overview.md#pricing-and-billing).<br /><br />Use la administración del ciclo de vida para eliminar versiones anteriores según sea necesario para controlar los costos. Para más información, consulte [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](storage-lifecycle-management-concepts.md). |
| Eliminación temporal de blobs | No se cobra por habilitar la eliminación temporal de blobs para una cuenta de almacenamiento. Los datos de un blob eliminado temporalmente se facturan a la misma tarifa que los datos activos hasta que el blob se elimina permanentemente. |
| Restauración a un momento dado | No se aplica ningún cargo por habilitar la restauración a un momento dado para una cuenta de almacenamiento; sin embargo, habilitar la restauración a un momento dado también habilita el control de versiones de blobs, la eliminación temporal y la fuente de cambios, cada uno de los cuales puede generar cargos adicionales.<br /><br />Se le facturará por la restauración a un momento dado cuando realice operaciones de restauración. El costo de una operación de restauración depende de la cantidad de datos que se restaurarán. Para obtener más información, consulte la sección [Precios y facturación](point-in-time-restore-overview.md#pricing-and-billing). |
| Instantáneas de blob | Los datos de una instantánea se facturan en función de los bloques o páginas únicos. Por lo tanto, los costos aumentan a medida que el blob base difiere de la instantánea. Cambiar el nivel de un blob o una instantánea puede tener un impacto en la facturación. Para obtener más información, consulte la sección [Precios y facturación](snapshots-overview.md#pricing-and-billing).<br /><br />Use la administración del ciclo de vida para eliminar instantáneas anteriores según sea necesario para controlar los costos. Para más información, consulte [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](storage-lifecycle-management-concepts.md). |
| Copia de datos en una segunda cuenta de almacenamiento | El mantenimiento de los datos en una segunda cuenta de almacenamiento generará costos de capacidad y de transacción. Si la segunda cuenta de almacenamiento se encuentra en una región diferente de la cuenta de origen, la copia de datos a esa segunda cuenta también generará en cargos de salida. |

## <a name="disaster-recovery"></a>Recuperación ante desastres

Azure Storage siempre mantiene varias copias de los datos, con el fin de protegerlos de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red, y desastres naturales masivos. La redundancia garantiza que la cuenta de almacenamiento cumple sus objetivos de disponibilidad y durabilidad, aunque se produzcan errores. Para obtener más información sobre cómo configurar la cuenta de almacenamiento para la alta disponibilidad, consulte [Redundancia de Azure Storage](../common/storage-redundancy.md).

En caso de que se produzca un error en un centro de datos, si la cuenta de almacenamiento es redundante en dos regiones geográficas (con redundancia geográfica), tendrá la opción de conmutar por error la cuenta de la región primaria a la secundaria. Para más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../common/storage-disaster-recovery-guidance.md).

La conmutación por error administrada por el cliente no se admite actualmente para las cuentas de almacenamiento con un espacio de nombres jerárquico habilitado. Para más información, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="next-steps"></a>Pasos siguientes

- [Redundancia de Azure Storage](../common/storage-redundancy.md)
- [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../common/storage-disaster-recovery-guidance.md)
