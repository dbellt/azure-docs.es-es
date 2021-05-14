---
title: 'Almacenamiento: procedimientos recomendados y guías de rendimiento'
description: En este artículo se proporcionan los procedimientos recomendados y guías de almacenamiento para optimizar el rendimiento de SQL Server en la máquina virtual (VM) de Azure.
services: virtual-machines-windows
documentationcenter: na
author: dplessMSFT
editor: ''
tags: azure-service-management
ms.assetid: a0c85092-2113-4982-b73a-4e80160bac36
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/25/2021
ms.author: dpless
ms.reviewer: jroth
ms.openlocfilehash: 23e006c637285ad484e98b23b2a9f506156f519c
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107389730"
---
# <a name="storage-performance-best-practices-for-sql-server-on-azure-vms"></a>Almacenamiento: procedimientos recomendados de rendimiento de SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporcionan los procedimientos recomendados y guías de almacenamiento para optimizar el rendimiento de SQL Server en las máquinas virtuales (VM) de Azure.

Por lo general, existe un equilibrio entre la optimización de los costos y la optimización del rendimiento. Esta serie de procedimientos recomendados de rendimiento se centra en cómo obtener el *mejor* rendimiento de SQL Server en las máquinas virtuales de Azure. Si su carga de trabajo es menos exigente, podría no necesitar todas las optimizaciones recomendadas. Tenga en cuenta sus necesidades de rendimiento, costos y patrones de carga de trabajo a medida que evalúa estas recomendaciones.

Para obtener más información, consulte otros artículos de esta serie: [Lista de comprobación de rendimiento](performance-guidelines-best-practices-checklist.md), [Tamaño de VM ](performance-guidelines-best-practices-vm-size.md)y [Recopilación de la línea base](performance-guidelines-best-practices-collect-baseline.md). 

## <a name="checklist"></a>Lista de comprobación

Revise la siguiente lista de comprobación para obtener una breve descripción de los procedimientos recomendados de almacenamiento que se describen en el resto del artículo con mayor detalle: 

- Supervise la aplicación y [determine los requisitos de ancho de banda y latencia de almacenamiento](../../../virtual-machines/premium-storage-performance.md#counters-to-measure-application-performance-requirements) para los archivos de registros, de datos y de tipo tempdb de SQL Server antes de elegir el tipo de disco. 
- Para optimizar el rendimiento del almacenamiento, planee el uso más alto de IOPS disponible sin almacenamiento en caché, y use el almacenamiento en caché de datos como una característica de rendimiento para realizar lecturas de datos al tiempo que evita el [límite de máquinas virtuales y discos](../../../virtual-machines/premium-storage-performance.md#throttling).
- Coloque los archivos de datos, de registros y de tipo tempdb en unidades independientes.
    - Para la unidad de datos, use solo [discos prémium de P30 y P40](../../../virtual-machines/disks-types.md#premium-ssd) para garantizar la disponibilidad del soporte técnico de la caché.
    - En cuanto a la unidad de registro, planee la capacidad y las pruebas de rendimiento junto al costo mientras se evalúan los [discos prémium P30-P80](../../../virtual-machines/disks-types.md#premium-ssd).
      - Si se requiere latencia de almacenamiento de submilisegundos, use los [discos Ultra de Azure](../../../virtual-machines/disks-types.md#ultra-disk) para el registro de transacciones. 
      - En el caso de las implementaciones de máquinas virtuales de la serie M, considere la posibilidad de usar el [acelerador de escritura](../../../virtual-machines/how-to-enable-write-accelerator.md) en discos Ultra de Azure.
    - Coloque el archivo [tempdb](/sql/relational-databases/databases/tempdb-database) en la unidad SSD local efímera `D:\` para la mayoría de las cargas de trabajo de SQL Server después de elegir el tamaño de VM óptimo. 
      - Si la capacidad de la unidad local no es suficiente para tempdb, considere la posibilidad de cambiar el tamaño de la VM. Consulte las [directivas de almacenamiento en caché de los archivos de datos](#data-file-caching-policies) para obtener más información.
- Divida varios discos de datos de Azure mediante los [espacios de almacenamiento](/windows-server/storage/storage-spaces/overview) para aumentar el ancho de banda de E/S hasta los límites de rendimiento e IOPS de la máquina virtual de destino.
- Establezca el [almacenamiento en caché del host](../../../virtual-machines/disks-performance.md#virtual-machine-uncached-vs-cached-limits) en Solo lectura para los discos de archivos de datos.
- Establezca el [almacenamiento en caché del host](../../../virtual-machines/disks-performance.md#virtual-machine-uncached-vs-cached-limits) en Ninguno para los discos de archivos de datos.
    - No habilite el almacenamiento en caché de lectura y escritura en discos que contengan archivos de SQL Server. 
    - Detenga siempre el servicio de SQL Server antes de cambiar la configuración de la memoria caché del disco.
- En el caso de cargas de trabajo de desarrollo y pruebas y el archivo de copia de seguridad a largo plazo, considere la posibilidad de usar el almacenamiento estándar. No se recomienda usar HDD o SDD estándar para las cargas de trabajo de producción.
- La [expansión de disco basada en crédito](../../../virtual-machines/disk-bursting.md#credit-based-bursting) (P1-P20) solo se debe tener en cuenta para cargas de trabajo de desarrollo y pruebas más pequeñas y sistemas departamentales.
- Aprovisione la cuenta de almacenamiento en la misma región que la VM de SQL Server. 
- Deshabilite el almacenamiento con redundancia geográfica de Azure (replicación geográfica) y use LRS (almacenamiento con redundancia local) en la cuenta de almacenamiento.
- Formatee el disco de datos para que use un tamaño de bloque (tamaño de unidad de asignación) de 64 KB para todos los archivos de datos ubicados en una unidad que no sea la unidad temporal `D:\` (que tiene un valor predeterminado de 4 KB). Las VM con SQL Server implementadas mediante Azure Marketplace se ofrecen con discos de datos formateados con un tamaño de bloque y una intercalación del bloque de almacenamiento establecidos en 64 KB. 

Para comparar la lista de comprobación de almacenamiento con las demás, consulte la [Lista de comprobación de procedimientos recomendados de rendimiento](performance-guidelines-best-practices-checklist.md). 

## <a name="overview"></a>Información general

Para encontrar la configuración más eficaz para las cargas de trabajo de SQL Server en una VM de Azure, empiece por [medir el rendimiento de almacenamiento de la aplicación empresarial](performance-guidelines-best-practices-collect-baseline.md#storage). Una vez que se conocen los requisitos de almacenamiento, seleccione una máquina virtual que admita el IOPS y el rendimiento necesarios con la relación de memoria a núcleo virtual adecuada. 

Elija un tamaño de VM con suficiente escalabilidad de almacenamiento para la carga de trabajo y una mezcla de discos (normalmente en un grupo de almacenamiento) que cumplan los requisitos de capacidad y rendimiento de su negocio. 

El tipo de disco depende del tipo de archivo que se hospeda en el disco y de los requisitos de rendimiento máximo.

> [!TIP]
> El aprovisionamiento de una VM con SQL Server a través de Azure Portal le guiará a través del proceso de configuración del almacenamiento e implementa la mayoría de los procedimientos recomendados de almacenamiento, como la creación de grupos de almacenamiento independientes para los archivos de registro, de datos y de tipo tempdb de destino en la unidad `D:\` y la habilitación de la directiva de almacenamiento en caché óptima. Para obtener más información sobre el aprovisionamiento y la configuración del almacenamiento, consulte [Configuración del almacenamiento de VM de SQL](storage-configuration.md). 

## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual

Puede elegir el nivel de rendimiento de los discos. Los tipos de discos administrados disponibles como almacenamiento subyacente (que se enumeran al aumentar las capacidades de rendimiento) son las unidades de disco duro (HDD) estándar, las SSD estándar, las unidades de estado sólido (SSD) prémium y los discos Ultra. 

El rendimiento del disco aumenta con la capacidad, agrupada por [etiquetas de disco prémium](../../../virtual-machines/disks-types.md#premium-ssd) como P1 con 4 GiB de espacio y 120 IOPS en P80 con 32 TiB de almacenamiento y 20 000 IOPS. Premium Storage admite una caché de almacenamiento que le permite mejorar el rendimiento de lectura y escritura de algunas cargas de trabajo. Para obtener más información, consulte [Introducción a Managed Disks](../../../virtual-machines/managed-disks-overview.md). 

También hay tres [tipos de discos](../../../virtual-machines/managed-disks-overview.md#disk-roles) principales que se deben tener en cuenta para la instancia de SQL Server en la VM de Azure: un disco del sistema operativo, un disco temporal y los discos de datos. Elija cuidadosamente lo que se almacena en la unidad del sistema operativo `(C:\)` y en la unidad temporal efímera `(D:\)`. 

### <a name="operating-system-disk"></a>Disco del sistema operativo

Un disco del sistema operativo es un VHD que se puede arrancar y montar como la versión en ejecución de un sistema operativo y está etiquetado como la unidad `C:\`. Al crear una máquina virtual de Azure, la plataforma conectará al menos un disco a la VM para su disco del sistema operativo. La unidad `C:\` es la ubicación predeterminada para las instalaciones de aplicaciones y la configuración de archivos. 

En entornos de SQL Server de producción, no use el disco del sistema operativo para los archivos de datos, los archivos de registro y los registros de errores. 

### <a name="temporary-disk"></a>Disco temporal

Muchas máquinas virtuales de Azure contienen otro disco denominado disco temporal (etiquetado como unidad `D:\`). Dependiendo de la serie y el tamaño de la máquina virtual, variará la capacidad de este disco. El disco temporal es efímero, lo que significa que se vuelve a crear el almacenamiento en disco (esto es, se desasigna y se asigna de nuevo) cuando la máquina virtual se reinicia o se mueve a otro host (por ejemplo, para realizar la [recuperación del servicio](/troubleshoot/azure/virtual-machines/understand-vm-reboot)). 

La unidad de almacenamiento temporal no se conserva en el almacenamiento remoto y, por lo tanto, no debe almacenar los archivos de base de datos de usuario, los archivos de registro de transacciones ni cualquier elemento que deba conservarse. 

Coloque el archivo tempdb en la unidad local de SSD `D:\` para las cargas de trabajo de SQL Server a menos que el consumo de la memoria caché local sea un problema. Si usa una máquina virtual que [no tiene un disco temporal](../../../virtual-machines/azure-vms-no-temp-disk.md), se recomienda colocar el archivo tempdb en su propio disco aislado o grupo de almacenamiento con la opción de almacenamiento en caché establecida en solo lectura. Para obtener más información, consulte las [directivas de almacenamiento en caché de datos del archivo tempdb](performance-guidelines-best-practices-storage.md#data-file-caching-policies).

### <a name="data-disks"></a>Discos de datos.

Los discos de datos son discos de almacenamiento remoto que se suelen crear en [grupos de almacenamiento](/windows-server/storage/storage-spaces/overview) con el fin de superar la capacidad y el rendimiento que puede ofrecer cualquier disco individual a la máquina virtual.

Adjunte el número mínimo de discos que satisface los requisitos de IOPS, de rendimiento y de capacidad de la carga de trabajo. No supere el número máximo de discos de datos de la máquina virtual más pequeña a la que va a cambiar el tamaño.

Coloque los archivos de datos y de registro en los discos de datos aprovisionados para satisfacer mejor los requisitos de rendimiento. 

Formatee el disco de datos para que use un tamaño de unidad de asignación de 64 KB para todos los archivos de datos ubicados en una unidad que no sea la unidad temporal `D:\` (que tiene un valor predeterminado de 4 KB). Las VM de SQL Server implementadas a través de Azure Marketplace se ofrecen con discos de datos formateados con el tamaño de la unidad de asignación y la intercalación para el bloque de almacenamiento establecido en 64 KB. 

## <a name="premium-disks"></a>Discos premium

Use discos SSD prémium para los archivos de datos y de registro de las cargas de trabajo de SQL Server de producción. El IOPS, SSD prémium y el ancho de banda de varían según el [tamaño y el tipo de disco](../../../virtual-machines/disks-types.md). 

En el caso de las cargas de trabajo de producción, use los discos P30 y P40 para los archivos de datos de SQL Server con el fin de garantizar la compatibilidad con el almacenamiento en caché y usar del tipo P30 hasta el P80 para los archivos de registro de transacciones de SQL Server.  Para obtener el mejor costo total de propiedad, empiece con los tipos P30 (5000 IOPS/200 MBPS) para los archivos de datos y de registro, y elija solo las capacidades más altas cuando necesite controlar el número de discos de máquinas virtuales.

En el caso de las cargas de trabajo de OLTP, haga coincidir los IOPS de destino por disco (o por grupo de almacenamiento) con los requisitos de rendimiento mediante las cargas de trabajo de las horas punta y los contadores de rendimiento `Disk Reads/sec` + `Disk Writes/sec`. En el caso de las cargas de trabajo de informes y almacenamiento de datos, haga coincidir el rendimiento de destino mediante las cargas de trabajo de las horas punta y `Disk Read Bytes/sec` + `Disk Write Bytes/sec`. 

Use los espacios de almacenamiento para lograr un rendimiento óptimo y configure dos grupos, uno para los archivos de registro y el otro para los archivos de datos. Si no usa el seccionamiento de discos, utilice dos discos SSD prémium asignados a unidades individuales: uno para el archivo de registro y el otro para guardar los datos.

El [IOPS y el rendimiento aprovisionados](../../../virtual-machines/disks-types.md#premium-ssd) por disco se usan como parte del bloque de almacenamiento. Las capacidades de IOPS y rendimiento combinadas de los discos son la capacidad máxima hasta los límites de rendimiento de la máquina virtual.

El procedimiento recomendado es usar el menor número posible de discos y cumplir los requisitos mínimos de IOPS (y rendimiento) y de capacidad. Sin embargo, el equilibrio entre el precio y el rendimiento tiende a ser mejor con un gran número de discos pequeños en lugar de un número pequeño de discos grandes.

### <a name="scaling-premium-disks"></a>Escalado de discos prémium

Cuando se implementa por primera vez un disco administrado de Azure, el nivel de rendimiento de ese disco se basa en el tamaño del disco aprovisionado. El nivel de rendimiento se puede designar en la implementación o cambiarlo posteriormente, sin tener que cambiar el tamaño del disco. Si la demanda aumenta, puede aumentar el nivel de rendimiento para satisfacer las necesidades de su empresa. 

Cambiar el nivel de rendimiento permite a los administradores prepararse y satisfacer una demanda más alta sin tener que depender de la [expansión de disco](../../../virtual-machines/disk-bursting.md#credit-based-bursting). 

Use el alto rendimiento durante el tiempo que sea necesario, donde la facturación está diseñada para satisfacer el nivel de rendimiento del almacenamiento. Actualice el nivel para que coincida con los requisitos de rendimiento sin aumentar la capacidad. Vuelva al nivel original cuando ya no se necesite el rendimiento adicional.

Esta expansión rentable y temporal del rendimiento es un caso de uso sólido para eventos dirigidos como compras, pruebas de rendimiento, eventos de entrenamiento y otras breves ventanas en las que solo se necesita un rendimiento aún mayor para un breve período. 

Para obtener más información, consulte los [Niveles de rendimiento de los discos administrados](../../../virtual-machines/disks-change-performance.md). 

## <a name="azure-ultra-disk"></a>Disco Ultra de Azure

Si hay una necesidad de tiempos de respuesta de submilisegundos con una latencia reducida, considere la posibilidad de usar el [disco Ultra de Azure](../../../virtual-machines/disks-types.md#ultra-disk) para la unidad de registro de SQL Server, o incluso la unidad de datos para las aplicaciones que son extremadamente sensibles a la latencia de E/S. 

Se puede configurar un disco Ultra en el que la capacidad e IOPS se pueden escalar de manera independiente. Con los administradores de discos Ultra puede aprovisionar un disco con los requisitos de capacidad, IOPS y rendimiento en función de las necesidades de la aplicación. 

El disco Ultra no es compatible con todas las series de VM y tiene otras limitaciones, como la disponibilidad de regiones, la redundancia y la compatibilidad con Azure Backup. Para obtener más información, consulte [Uso de discos Ultra de Azure](../../../virtual-machines/disks-enable-ultra-ssd.md) para obtener una lista completa de limitaciones. 

## <a name="standard-hdds-and-ssds"></a>HDD y SSD estándar

Los [HDD estándar](../../../virtual-machines/disks-types.md#standard-hdd) y las unidades de estado sólido (SSD) presentan el ancho de banda y las latencias variables, de modo que solo se recomiendan para realizar cargas de trabajo de desarrollo o de prueba. En las cargas de trabajo de producción conviene usar SSD prémium. Si no usa las unidades SSD estándar (escenarios de desarrollo y pruebas), se recomienda agregar el número máximo de discos de datos que admita el [tamaño de la VM](../../../virtual-machines/sizes.md?toc=/azure/virtual-machines/windows/toc.json) y usar el seccionamiento de discos con los espacios de almacenamiento para obtener el mejor rendimiento.

## <a name="caching"></a>Almacenamiento en memoria caché

Las máquinas virtuales que admiten el almacenamiento en caché de Premium Storage pueden aprovechar las ventajas de una característica adicional denominada Azure BlobCache o el almacenamiento en caché de host para ampliar las capacidades de IOPS y el rendimiento de una máquina virtual. Las máquinas virtuales habilitadas tanto para Premium Storage como para el almacenamiento en caché de Premium Storage, tienen estos dos límites de ancho de banda de almacenamiento que se pueden usar de forma conjunta para mejorar el rendimiento del almacenamiento.

El IOPS y el rendimiento de MBps sin almacenamiento en caché cuenta con los límites de rendimiento de disco no almacenado en caché de una máquina virtual. Los límites máximos almacenados en la caché proporcionan un búfer adicional para las lecturas que le permite abordar el crecimiento y los picos inesperados.

Habilite el almacenamiento en caché prémium siempre que se admita la opción para mejorar significativamente el rendimiento de las lecturas en la unidad de datos sin costo adicional. 

Las operaciones de lectura y escritura en Azure BlobCache (IOPS y rendimiento en caché) no se tienen en cuentan en los límites de rendimiento e IOPS que no estén en el almacenamiento en caché de la máquina virtual.

> [!NOTE]
> No se admite el almacenamiento en caché de disco para discos de 4 TiB o más (P50 y superiores). Si varios discos están conectados a la máquina virtual, cada disco de menos de 4 TiB será compatible con el almacenamiento en caché. Para obtener más información, consulte el [Almacenamiento en caché del disco](../../../virtual-machines/premium-storage-performance.md#disk-caching). 

### <a name="uncached-throughput"></a>Rendimiento no almacenado en caché

El rendimiento de disco máximo no almacenado en caché para IOPS y el rendimiento es el límite máximo de almacenamiento remoto que la máquina virtual puede controlar. Este límite se define en la máquina virtual y no es un límite del almacenamiento en disco subyacente. Este límite se aplica solo a la E/S en las unidades de datos conectadas de forma remota a la VM, no a la E/S local en la unidad temporal (unidad `D:\`) o la unidad del sistema operativo.

La cantidad de IOPS y rendimiento sin almacenamiento en caché que está disponible para una VM puede comprobarse en la documentación de la máquina virtual.

Por ejemplo, la documentación de la [serie M](../../../virtual-machines/m-series.md) muestra que el rendimiento máximo no almacenado en caché de la VM Standard_M8ms es 5000 IOPS y 125 Mbps de rendimiento de disco sin almacenamiento en caché. 

![Captura de pantalla que muestra la documentación sobre el rendimiento de disco no almacenado en caché de la serie M.](./media/performance-guidelines-best-practices/m-series-table.png)

Del mismo modo, puede ver que la VM Standard_M32ts admite 20 000 IOPS de disco sin almacenamiento en caché y un rendimiento de disco no almacenado en caché de 500 MBps. Este límite se rige en función del nivel de máquina virtual, independientemente del almacenamiento en disco prémium subyacente.

Para obtener más información, consulte los [límites almacenados y no almacenados en caché](../../../virtual-machines/linux/disk-performance-linux.md#virtual-machine-uncached-vs-cached-limits).


### <a name="cached-and-temp-storage-throughput"></a>Rendimiento de almacenamiento temporal y en caché

El límite de rendimiento máximo de almacenamiento temporal y en caché es un límite independiente del límite de rendimiento no almacenado en caché de la máquina virtual. Azure BlobCache consta de una combinación de la memoria de acceso aleatorio del host de la máquina virtual y el SSD conectado localmente. La unidad temporal (unidad `D:\`) dentro de la máquina virtual también se hospeda en este SSD local.

El límite de rendimiento máximo de almacenamiento temporal y en caché rige la E/S en la unidad temporal local (unidad `D:\`) y la instancia de Azure BlobCache **solo si** está habilitado el almacenamiento en caché del host. 

Cuando el almacenamiento en caché está habilitado en Premium Storage, las máquinas virtuales pueden escalar más allá de las limitaciones de los límites de rendimiento e IOPS de VM no almacenados en caché de almacenamiento remoto.  

Solo algunas máquinas virtuales admiten Premium Storage y el almacenamiento en caché de Premium Storage (que debe comprobarse en la documentación de la máquina virtual). Por ejemplo, la documentación de la [serie M](../../../virtual-machines/m-series.md) indica que se admite tanto Premium Storage como el almacenamiento en caché de Premium Storage: 

![Captura de pantalla que muestra la compatibilidad con Premium Storage de la serie M.](./media/performance-guidelines-best-practices/m-series-table-premium-support.png)

Los límites de la memoria caché variarán en función del tamaño de la máquina virtual. Por ejemplo, la máquina virtual Standard_M8ms admite 10 000 IOPS de disco en caché y un rendimiento de disco en caché de 1000 MBps con un tamaño total de caché de 793 GiB. De forma similar, la VM Standard_M32ts admite 40 000 IOPS de disco en caché y un rendimiento de disco en caché de 400 MBps con un tamaño total de caché de 3174 GiB. 

![Captura de pantalla que muestra la documentación sobre el rendimiento de disco de caché de la serie M.](./media/performance-guidelines-best-practices/m-series-table-cached-temp.png)

Puede habilitar manualmente el almacenamiento en caché del host en una VM existente. Detenga todas las cargas de trabajo de la aplicación y los servicios de SQL Server antes de realizar cambios en la directiva de almacenamiento en caché de la máquina virtual. Al cambiar la configuración de la memoria caché de la máquina virtual, el disco de destino se desasocia y se vuelve a adjuntar después de aplicar la configuración.

### <a name="data-file-caching-policies"></a>Directivas de almacenamiento en caché de los archivos de datos

La directiva de almacenamiento en caché varía en función del tipo de los archivos de datos de SQL Server que se hospedan en la unidad. 

En la tabla siguiente se proporciona un resumen de las directivas de almacenamiento en caché recomendadas según el tipo de datos de SQL Server: 

|Disco de SQL Server |Recomendación |
|---------|---------|
| **Disco de datos** | Habilite el almacenamiento en caché `Read-only` para los discos que hospedan archivos de datos de SQL Server. <br/> Las lecturas de la caché serán más rápidas que las lecturas no almacenadas en caché del disco de datos. <br/> El IOPS y el rendimiento sin almacenamiento en caché más el rendimiento y el IOPS en caché producirán el rendimiento total posible disponible desde la máquina virtual dentro de los límites de las VM, pero el rendimiento real variará en función de la capacidad de la carga de trabajo para usar la memoria caché (proporción de aciertos de la caché). <br/>|
|**Disco del registro de transacciones**|Establezca la directiva de almacenamiento en caché en `None` para los discos que hospedan el registro de transacciones.  La habilitación del almacenamiento en caché para el disco del registro de transacciones no supone ninguna ventaja de rendimiento y, de hecho, tener el almacenamiento en caché de `Read-only` o `Read/Write` habilitado en la unidad de registro puede reducir el rendimiento de las opciones de escritura en la unidad y reducir la cantidad de almacenamiento en caché disponible para las lecturas en la unidad de datos.  |
|**Disco del SO operativo** | La directiva de almacenamiento en caché predeterminada podría ser `Read-only` o `Read/write` para la unidad del sistema operativo. <br/> No se recomienda cambiar el nivel de almacenamiento en caché de la unidad del sistema operativo.  |
| **tempdb**| Si tempdb no se puede colocar en la unidad efímera `D:\` debido a razones de capacidad, cambie el tamaño de la máquina virtual para obtener una unidad efímera más grande o coloque el archivo tempdb en una unidad de datos independiente con el almacenamiento en caché de `Read-only` configurado. <br/> La memoria caché de la máquina virtual y la unidad efímera usan el SSD local, por lo que debe tener esto en cuenta al ajustar la E/S de tempdb en los límites de la máquina virtual de IOPS y rendimiento almacenados en la memoria caché cuando se hospedan en la unidad efímera.| 
| | | 


Para obtener más información, consulte [Almacenamiento en caché de discos](../../../virtual-machines/premium-storage-performance.md#disk-caching). 


## <a name="disk-striping"></a>Seccionamiento del disco

Analice el rendimiento y el ancho de banda necesarios para los archivos de datos de SQL con el fin de determinar el número de discos de datos, incluido el archivo de registro y de tipo tempdb. Los límites de rendimiento y ancho de banda varían según el tamaño de la VM. Para obtener más información, consulte el [Tamaño de VM](../../../virtual-machines/sizes.md).

Para disfrutar de un mayor rendimiento, puede agregar más discos de datos y usar la segmentación de discos. Por ejemplo, una aplicación que necesita un rendimiento de 12 000 IOPS y 180 MB/s puede usar tres discos P30 segmentados para ofrecer un rendimiento de 15 000 IOPS y 600 MB/s. 

Para configurar la segmentación de discos, consulte la [segmentación de discos](storage-configuration.md#disk-striping). 

## <a name="disk-capping"></a>Límite del disco 

Hay límites de rendimiento en el nivel de disco y de máquina virtual. Los límites máximos de IOPS por VM y por disco son diferentes e independientes entre sí.

Las aplicaciones que consumen recursos más allá de estos límites se limitarán. Seleccione una máquina virtual y un tamaño de disco en una franja de disco que cumpla los requisitos de la aplicación y no se aplicarán limitaciones. Para administrar el límite, use el almacenamiento en caché o ajuste la aplicación para que se requiera menos rendimiento.

Por ejemplo, una aplicación que necesita 12 000 IOPS y 180 MB/s puede: 
- Usar el tipo [Standard_M32ms](../../../virtual-machines/m-series.md) que tiene un rendimiento de disco no almacenado en caché máximo de 20 000 IOPS y 500 Mbps.
- Segmentar tres discos P30 para ofrecer un rendimiento de 15 000 IOPS y 600 MB/s.
- Usar una máquina virtual [Standard_M16ms](../../../virtual-machines/m-series.md) y use el almacenamiento en caché de host para usar la memoria caché local durante el consumo de rendimiento. 

Las máquinas virtuales configuradas para escalar verticalmente durante los períodos de uso elevado deben aprovisionar el almacenamiento con suficientes IOPS y rendimiento para admitir el tamaño máximo de la VM y garantizar que el número total de los discos sea inferior o igual al número máximo que admite la SKU de VM más pequeña que se va a usar.

Para obtener más información sobre las limitaciones del disco y el uso del almacenamiento en caché para evitar el límite, consulte [Limitación de E/S del disco](../../../virtual-machines/disks-performance.md).

> [!NOTE] 
> Algunos límites de disco pueden dar lugar a un rendimiento satisfactorio para los usuarios; ajuste y mantenga las cargas de trabajo en lugar de cambiar su tamaño a una VM más grande para equilibrar la administración del costo y el rendimiento de la empresa. 


## <a name="write-acceleration"></a>Aceleración de escritura

La Aceleración de escritura es una característica de disco que solo está disponible para las máquinas virtuales de (VM) [serie M](https://docs.microsoft.com/azure/virtual-machines/m-series). El propósito de la Aceleración de escritura es mejorar la latencia de E/S de las operaciones de escritura en Azure Premium Storage cuando se necesita una latencia de E/S de un solo dígito, debido a las grandes cargas de trabajo de OLTP o a los entornos de almacenamiento de datos. 

Use la Aceleración de escritura para mejorar la latencia de las operaciones de escritura en la unidad que hospeda los archivos de registro. No use la Aceleración de escritura para archivos de datos de SQL Server. 

Los discos del Acelerador de escritura comparten el mismo límite de IOPS que la máquina virtual. En una VM, los discos conectados no pueden superar el límite de IOPS del Acelerador de escritura.  

En la tabla siguiente se describe el número de discos de datos y IOPS admitidos por máquina virtual: 

| SKU de la máquina virtual  | Número de discos del Acelerador de escritura  | IOPS de discos del Acelerador de escritura por VM  |
|---|---|---|
| M416ms_v2, M416s_v2  | 16  | 20000  |
| M128ms, M128s  | 16  | 20000  |
| M208ms_v2, M208s_v2  | 8  | 10000  |
| M64ms, M64ls, M64s  |  8 | 10000 |
| M32ms, M32ls, M32ts, M32s  | 4  | 5000  |
| M16ms, M16s  | 2 | 2.500 |
| M8ms, M8s  | 1 | 1250 |

Hay una serie de restricciones a la hora de usar la Aceleración de escritura. Para obtener más información, consulte [Restricciones al usar el Acelerador de escritura](../../../virtual-machines/how-to-enable-write-accelerator.md#restrictions-when-using-write-accelerator).


### <a name="comparing-to-azure-ultra-disk"></a>Comparación con el disco Ultra de Azure

La diferencia más importante entre la Aceleración de escritura y los discos Ultra de Azure es que la Aceleración de escritura es una característica de máquina virtual que solo está disponible para la serie M y los discos Ultra de Azure son una opción de almacenamiento. La Aceleración de escritura es una caché optimizada para la escritura, que cuenta sus propias limitaciones en función del tamaño de la máquina virtual. En cambio, los discos Ultra de Azure son una opción de almacenamiento en disco de latencia baja para Azure Virtual Machines. 

Si es posible, use la Aceleración de escritura en lugar de los discos Ultra para el disco del registro de transacciones. En el caso de las máquinas virtuales que no admiten la Aceleración de escritura pero que requieren una latencia baja en el registro de transacciones, use los discos Ultra de Azure. 

## <a name="monitor-storage-performance"></a>Supervisión del rendimiento del almacenamiento

Para evaluar las necesidades de almacenamiento y determinar cómo funciona el mismo, debe comprender qué se debe medir y qué significan esos indicadores. 

[IOPS (Entrada/Salida por segundo)](../../../virtual-machines/premium-storage-performance.md#iops) es el número de solicitudes que la aplicación realiza en el almacenamiento por segundo. Mida el número IOPS mediante los contadores `Disk Reads/sec` y `Disk Writes/sec` del Monitor de rendimiento. Las aplicaciones [OLTP (Procesamiento de transacciones en línea)](/azure/architecture/data-guide/relational-data/online-transaction-processing) necesitan impulsar más IOPS para lograr un rendimiento óptimo. Asimismo, las aplicaciones como los sistemas de procesamiento de pagos, la compra en línea y los sistemas de punto de venta minorista son ejemplos de aplicaciones OLTP.

El [Rendimiento](../../../virtual-machines/premium-storage-performance.md#throughput) es el volumen de datos que se envía al almacenamiento subyacente, y a menudo se mide en megabytes por segundo. Mida el rendimiento con los contadores `Disk Read Bytes/sec` y `Disk Write Bytes/sec` del Monitor de rendimiento. El [Almacenamiento de datos](/azure/architecture/data-guide/relational-data/data-warehousing) está optimizado para maximizar el rendimiento a través de IOPS. Las aplicaciones como los almacenes de datos para el análisis, los informes, las secuencias de trabajo ETL y otros destinos de inteligencia empresarial son ejemplos de aplicaciones de almacenamiento de datos.

Los tamaños de unidad de E/S influyen en el IOPS y en las capacidades de rendimiento, ya que los tamaños de E/S más pequeños producen una mayor velocidad de E/S y un mayor rendimiento. SQL Server elige automáticamente el tamaño de E/S óptimo. Para obtener más información acerca de esto, consulte [Optimización de IOPS, el rendimiento y la latencia de las aplicaciones](../../../virtual-machines/premium-storage-performance.md#optimize-iops-throughput-and-latency-at-a-glance). 

Existen métricas de Azure Monitor específicas que son útiles para detectar el límite en el nivel de la máquina virtual y el disco, así como el consumo y el estado de la caché de AzureBlob. Para identificar los contadores clave que se van a agregar a la solución de supervisión y al panel de Azure Portal, consulte [Métricas de uso de almacenamiento](../../../virtual-machines/disks-metrics.md#storage-io-utilization-metrics). 

> [!NOTE]
> Azure Monitor no ofrece actualmente métricas de nivel de disco para la unidad temporal efímera `(D:\)`. El porcentaje de uso de IOPS en caché de la VM y el porcentaje de ancho de banda de VM consumido reflejarán los IOPS y el rendimiento de la unidad temporal efímera `(D:\)` y del almacenamiento en caché de host.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los procedimientos recomendados de rendimiento, consulte otros artículos de esta serie:
- [Lista de comprobación rápida](performance-guidelines-best-practices-checklist.md)
- [Tamaño de VM](performance-guidelines-best-practices-vm-size.md)
- [Recopilación de la línea base](performance-guidelines-best-practices-collect-baseline.md)

Para ver los procedimientos recomendados de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Para realizar pruebas detalladas del rendimiento de SQL Server en máquinas virtuales de Azure con las pruebas comparativas TPC-E y TPC_C, consulte el blog en el que se explica cómo [optimizar el rendimiento de OLTP](https://techcommunity.microsoft.com/t5/sql-server/optimize-oltp-performance-with-sql-server-on-azure-vm/ba-p/916794).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
