---
title: Guía de migración de Azure Storage
description: La guía de introducción sobre la migración del almacenamiento describe instrucciones básicas para su realización.
author: dukicn
ms.author: nikoduki
ms.topic: conceptual
ms.date: 03/31/2021
ms.service: storage
ms.subservice: common
ms.openlocfilehash: 3e9baedafb436bc92f734bf39519918686cec58d
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124480"
---
# <a name="azure-storage-migration-overview"></a>Introducción a la migración del almacenamiento

Este artículo se centra en las migraciones del almacenamiento a Azure y proporciona instrucciones sobre los siguientes escenarios de migración del almacenamiento:

- Migración de datos no estructurados, como archivos y objetos
- Migración de dispositivos basados en bloques, como discos y redes de área de almacenamiento (SAN)

## <a name="migration-of-unstructured-data"></a>Migración de datos no estructurados

La migración de datos no estructurados incluye los siguientes escenarios:

- Migración de archivos del almacenamiento conectado a la red (NAS) a una de las ofertas de archivos de Azure:
  - [Archivos de Azure](https://azure.microsoft.com/services/storage/files/)
  - [Azure NetApp Files](https://azure.microsoft.com/services/netapp/)
  - [Soluciones de proveedores de software independientes (ISV)](../solution-integration/validated-partners/primary-secondary-storage/partner-overview.md).
- Migración de objetos desde las soluciones de almacenamiento de objetos a la plataforma de almacenamiento de objetos de Azure:
  - [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/)
  - [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/).

### <a name="migration-phases"></a>Fases de migración

Una migración completa consta de varias fases diferentes: detección, evaluación y migración.

![Diagrama que muestra las fases de la migración: detección, evaluación y migración.](./media/storage-migration-overview/migration-phases.png)

#### <a name="discovery-phase"></a>Fase de detección

En la fase de detección, se determinan todos los orígenes que deben migrarse como recursos compartidos de SMB, exportaciones NFS o espacios de nombres de objetos. Puede realizar esta fase manualmente o usar herramientas automatizadas.

#### <a name="assessment-phase"></a>Fase de evaluación

La fase de evaluación es fundamental para comprender las opciones disponibles para la migración. Para reducir el riesgo durante la migración y evitar errores comunes, siga estos tres pasos:

| Pasos de la fase de evaluación                     | Opciones                                                                          |
|--------------------------------------------|----------------------------------------------------------------------------------|
| **Elección de un servicio de almacenamiento de destino**            | - Azure Blob Storage y Data Lake Storage<br>- Azure Files<br>- Azure NetApp Files<br>- Soluciones de proveedores de software independientes |
| **Selección de un método de migración**                  | - En línea<br>- Sin conexión<br> - Combinación de ambos                                  |
| **Elección de la mejor herramienta de migración para el trabajo** | - Herramientas comerciales (Azure y proveedores de software independientes)<br> - Código abierto                             


Hay varias herramientas comerciales (ISV) que pueden ayudar en la fase de evaluación. Consulte la [tabla comparativa](../solution-integration/validated-partners/data-management/migration-tools-comparison.md).

##### <a name="choose-a-target-storage-service"></a>Elección de un servicio de almacenamiento de destino

La elección de un servicio de almacenamiento de destino depende de la aplicación o de los usuarios que acceden a los datos. La elección correcta depende de aspectos técnicos y financieros. En primer lugar, realice una evaluación técnica para evaluar los posibles destinos y determinar qué servicios cumplen los requisitos. A continuación, realice una evaluación financiera para determinar cuál es la mejor opción.

Para ayudar a seleccionar el servicio de almacenamiento de destino para la migración, evalúe los siguientes aspectos de cada servicio:

- Compatibilidad con protocolos
- Características de rendimiento
- Límites del servicio de almacenamiento de destino

El diagrama siguiente es un árbol de decisión simplificado que le ayudará con los servicios de archivos de Azure recomendados. Si los servicios nativos de Azure no satisfacen los requisitos, hay una variedad de [soluciones de proveedores de software independientes](../solution-integration/validated-partners/primary-secondary-storage/partner-overview.md) que sí pueden hacerlo.

Después de finalizar la evaluación técnica y seleccionar el destino adecuado, realice una evaluación de costos para determinar la opción más rentable.

![Árbol de decisión básico para elegir el servicio de archivos correcto](./media/storage-migration-overview/files-decision-tree.png)

Para simplificar el árbol de decisión, los límites del servicio de almacenamiento de destino no se han incorporado al diagrama. Para más información sobre los límites actuales y para determinar si es necesario modificar las opciones en función de ellos, consulte:

- [Límites de la cuenta de almacenamiento](../../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits)
- [Límites de Blob Storage](../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-blob-storage-limits)
- [Objetivos de escalabilidad y rendimiento de Azure Files](../files/storage-files-scale-targets.md)
- [Límites de recursos para Azure NetApp Files](../../azure-netapp-files/azure-netapp-files-resource-limits.md)

Si alguno de los límites constituye un bloqueo para usar un servicio, Azure admite varios proveedores de almacenamiento que ofrecen sus soluciones en Azure Marketplace. Para obtener información acerca de los ISV asociados validados que proporcionan servicios de archivos, consulte [Asociados de Azure Storage para el almacenamiento principal y secundario](../solution-integration/validated-partners/primary-secondary-storage/partner-overview.md).

##### <a name="select-the-migration-method"></a>Selección del método de migración

Existen dos métodos de migración básicos para las migraciones de almacenamiento.

- **En línea**. El método en línea usa la red para la migración de datos. Se puede usar la red pública de Internet o [Azure ExpressRoute](../../expressroute/expressroute-introduction.md). Si el servicio no tiene un punto de conexión público, debe usar una VPN con Internet público.
- **Sin conexión.** El método sin conexión usa uno de los dispositivos de [Azure Data Box](https://azure.microsoft.com/services/databox/).

La decisión de usar un método en línea en lugar de un método sin conexión depende del ancho de banda de red disponible. Se prefiere el método en línea en los casos en los que haya suficiente ancho de banda de red para realizar una migración dentro de la escala de tiempo necesaria.

Es posible usar una combinación de ambos métodos, el método sin conexión para la migración masiva inicial y un método en línea para la migración incremental de los cambios. El uso simultáneo de ambos métodos requiere un alto nivel de coordinación y no se recomienda por este motivo. Si decide usar ambos métodos, aísle los conjuntos de datos que se migrarán en línea de los conjuntos de datos que se migrarán sin conexión.

Para más información sobre los diferentes métodos de migración e instrucciones, consulte [Elección de la solución de Azure para la transferencia de datos](./storage-choose-data-transfer-solution.md) y [Migración a recursos compartidos de archivos de Azure](../files/storage-files-migration-overview.md).

##### <a name="choose-the-best-migration-tool-for-the-job"></a>Elección de la mejor herramienta de migración para el trabajo

Hay varias herramientas de migración que puede usar para realizar la migración. Algunas son de código abierto como AzCopy, Robocopy, xcopy y rsync, otras en cambio son comerciales. La lista de herramientas comerciales disponibles y la comparación entre ellas está disponible en nuestra [tabla comparativa](../solution-integration/validated-partners/data-management/migration-tools-comparison.md).

Las herramientas de código abierto son adecuadas para migraciones a pequeña escala. Para la migración desde servidores de archivos de Windows a Azure Files, Microsoft recomienda comenzar con la funcionalidad nativa de Azure Files y usar [Azure File Sync](/windows-server/manage/windows-admin-center/azure/azure-file-sync). Para migraciones más complejas que se componen de diferentes orígenes, gran capacidad o requisitos especiales, como la limitación o la generación de informes detallados con funcionalidades de auditoría, las herramientas comerciales son la mejor opción. Estas herramientas facilitan la migración y reducen el riesgo de forma significativa. La mayoría de las herramientas comerciales también pueden realizar la detección, lo que proporciona una valiosa información para la evaluación.

#### <a name="migration-phase"></a>Fase de migración

La fase de migración es el paso final del proceso que realiza el movimiento de datos y la migración. Normalmente, se ejecutará la fase de migración varias veces para lograr un intercambio más sencillo. La fase de migración consta de los siguientes pasos:

1. **Migración inicial.** En el paso de migración inicial se migran todos los datos del origen al destino. Este paso migra el grueso de los datos.
2. **Resincronización.** La operación de resincronización migra todos los datos que se cambiaron después del paso de migración inicial. Puede repetir este paso varias veces si hay muchos cambios. El objetivo de ejecutar varias operaciones de resincronización es reducir el tiempo que tarda el paso final. Para los datos inactivos o que no tienen cambios (como los datos de copia de seguridad o archivo), puede omitir este paso.
3. **Intercambio final**. El paso de intercambio final cambia el uso activo de los datos del origen al destino y retira el origen.

La duración de la migración de datos no estructurados depende de varios aspectos. Aparte del método elegido, los factores fundamentales son el tamaño total de los datos y la distribución del tamaño de los archivos. Cuanto mayor sea el conjunto de datos total, mayor será el tiempo de migración. Cuanto menor sea el tamaño medio de los archivos, mayor será el tiempo de migración. Si tiene un gran número de archivos pequeños, considere la posibilidad de archivarlos en archivos más grandes (por ejemplo, un archivo .tar o .zip), si procede, para reducir el tiempo total de migración.

## <a name="migration-of-block-based-devices"></a>Migración de dispositivos basados en bloques

La migración de dispositivos basados en bloques se realiza normalmente como parte de la migración de máquinas virtuales o hosts físicos. Es una idea equivocada habitual el hecho de retrasar las decisiones de almacenamiento en bloques hasta después de la migración. Tomar estas decisiones con anterioridad con las consideraciones adecuadas para los requisitos de las cargas de trabajo conduce a una migración más fluida a la nube.

Para explorar las cargas de trabajo que se van a migrar y decidir el enfoque que se va a adoptar, consulte la [documentación de Azure Disk Storage](../../virtual-machines/disks-types.md), y para conocer los recursos, consulte la [página de producto de Disk Storage](https://azure.microsoft.com/services/storage/disks/#resources). Puede obtener información sobre qué discos se ajustan a sus requisitos y sobre las funcionalidades más recientes, como la [expansión de disco](../../virtual-machines/disk-bursting.md). Para más información sobre cómo migrar las máquinas virtuales junto con los dispositivos basados en bloques subyacentes, consulte la documentación de [Azure Migrate](../../migrate/index.yml).

## <a name="see-also"></a>Consulte también

- [Elección de la solución de Azure para la transferencia de datos](./storage-choose-data-transfer-solution.md)
- [Comparación entre herramientas de migración comerciales](../solution-integration/validated-partners/data-management/migration-tools-comparison.md)
- [Migración a recursos compartidos de archivos de Azure](../files/storage-files-migration-overview.md)
- [Migración a Data Lake Storage con WANdisco LiveData Platform para Azure](../blobs/migrate-gen2-wandisco-live-data-platform.md)
- [Copia o movimiento de datos a Azure Storage con AzCopy](./storage-use-azcopy-v10.md)
- [Migración de grandes conjuntos de datos a Azure Blob Storage con AzReplicate](https://github.com/Azure/AzReplicate/tree/master/)