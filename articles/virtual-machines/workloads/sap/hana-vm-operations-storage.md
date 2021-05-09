---
title: Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA | Microsoft Docs
description: Recomendaciones de almacenamiento para máquinas virtuales con SAP HANA implementado en ellas.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP, Azure HANA, disco Ultra Storage, Premium Storage
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/27/2021
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4350f60029673af04ad263c9e9f25d7a74bc532b
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131032"
---
# <a name="sap-hana-azure-virtual-machine-storage-configurations"></a>Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA

Azure proporciona distintos tipos de almacenamiento adecuados para máquinas virtuales de Azure que ejecutan SAP HANA. Los **tipos de almacenamiento de Azure con certificación de SAP HANA** que pueden considerarse para las implementaciones de SAP HANA, como: 

- SSD Premium o Premium Storage de Azure 
- [Disco Ultra](../../disks-enable-ultra-ssd.md)
- [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) 

Para obtener información sobre estos tipos de disco, consulte el artículo [Tipos de Azure Storage para la carga de trabajo de SAP](./planning-guide-storage.md) y [Seleccionar un tipo de disco](../../disks-types.md).

Azure ofrece dos métodos de implementación de VHD en Azure Standard Storage y Azure Premium Storage. Esperamos que aproveche el [disco administrado de Azure](https://azure.microsoft.com/services/managed-disks/) para implementaciones de almacenamiento en bloque de Azure. 

Para obtener una lista de tipos de almacenamiento y sus Acuerdos de Nivel de Servicio sobre IOPS y rendimiento del almacenamiento, revise la [documentación de Azure para Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).

> [!IMPORTANT]
> Independientemente del tipo de almacenamiento de Azure elegido, el sistema de archivos que se usa en ese almacenamiento debe ser compatible con SAP para el sistema operativo y el DBMS específicos. La [nota de compatibilidad 2972496 de SAP](https://launchpad.support.sap.com/#/notes/2972496) enumera los sistemas de archivos admitidos para los diferentes sistemas operativos y bases de datos, incluido SAP HANA. Esto se aplica a todos los volúmenes SAP HANA con acceso de lectura y escritura para cualquier tarea. Especialmente si usa NFS en Azure para SAP HANA, se aplican restricciones adicionales de versiones de NFS como se indica más adelante en este artículo. 


Las condiciones de certificación mínimas de SAP HANA para los diferentes tipos de almacenamiento son: 

- Azure Premium Storage: el [Acelerador de escritura](../../how-to-enable-write-accelerator.md) de Azure debe admitir **/hana/log**. El volumen **/hana/data** puede colocarse en Premium Storage sin el Acelerador de escritura de Azure o bien en un disco Ultra.
- Disco Ultra de Azure al menos para el volumen **/hana/log**. El volumen **/hana/data** se puede colocar en el Premium Storage sin el Acelerador de escritura de Azure o en un disco Ultra para obtener un tiempo de reinicio más rápido.
- Volúmenes de **NFS v4.1** sobre Azure NetApp Files para **/hana/log y /hana/data**. El volumen de /hana/shared puede usar el protocolo NFS v3 o NFS v 4.1.

Algunos de los tipos de almacenamiento se pueden combinar. Por ejemplo, es posible colocar **/hana/data** en Premium Storage y **/hana/log** se puede colocar en un almacenamiento en disco Ultra para obtener la baja latencia necesaria. Si usa un volumen basado en ANF para **/hana/data**, el volumen **/hana/log** se debe basar también en NFS, además de en ANF. **No se admite** el uso de NFS sobre ANF para uno de los volúmenes (como /hana/data) y Azure Premium Storage o almacenamiento en disco Ultra para el otro volumen (como **/hana/log**).

En el mundo local, casi nunca ha tenido que preocuparse sobre los subsistemas de E/S y sus funcionalidades. El motivo era que el proveedor de las aplicaciones tenía que asegurarse de que se cumplieran los requisitos de almacenamiento mínimo para SAP HANA. Mientras crea la infraestructura de Azure, debe tener en cuenta algunos de estos requisitos que emite SAP. Algunas de las características de rendimiento mínimas que recomienda SAP son:

- Lectura o escritura en **/hana/log** de 250 MB/s con tamaños de E/S de 1 MB
- Actividad de lectura de al menos 400 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB
- Actividad de escritura de al menos 250 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB

Debido a que la baja latencia de almacenamiento es fundamental para los sistemas DBMS, incluso estos DBMS, como SAP HANA, mantienen los datos en memoria. La ruta crítica en el almacenamiento normalmente está relacionada con las escrituras del registro de transacciones de los sistemas DBMS. Sin embargo, también las operaciones como la escritura de puntos de retorno o la carga de datos en memoria después de la recuperación tras un bloqueo pueden ser críticas. Por ello, es **obligatorio** usar Azure Premium Storage, discos Ultra o ANF para los volúmenes **/hana/data** y **/hana/log**. 


Estos son algunos principios de la selección de la configuración de almacenamiento para HANA:

- Decida el tipo de almacenamiento según [Tipos de Azure Storage para la carga de trabajo de SAP](./planning-guide-storage.md) y [Seleccionar un tipo de disco](../../disks-types.md).
- El rendimiento global de E/S de la VM y los límites de IOPS al elegir una VM o determinar su tamaño. El rendimiento general del almacenamiento de VM está documentado en el artículo [Tamaños de máquina virtual optimizada para memoria](../../sizes-memory.md).
- A la hora de decidirse por la configuración del almacenamiento, intente permanecer por debajo del rendimiento general de la VM con la configuración del volumen **/hana/data**. En cuanto a los puntos de retorno, SAP HANA puede presentar cierta agresividad al emitir E/S. Es fácil alcanzar los límites de rendimiento del volumen **/hana/data** al escribir un punto de retorno. Si los discos que compilan el volumen **/hana/data** tienen un rendimiento superior al que permite la VM, puede que se produzcan situaciones en que el rendimiento usado por la escritura del punto de retorno interfiera con las demandas de rendimiento de las escrituras de registros de fase de puesta al día. Esta situación puede afectar al rendimiento de la aplicación.
- Si está pensando en usar la replicación del sistema HANA, debe usar exactamente el mismo tipo de almacenamiento de Azure para **/hana/data** y **/hana/log** para todas las VM que participan en la configuración de replicación del sistema HANA. Por ejemplo, no se admite el uso de Azure Premium Storage para **/hana/data** con una VM y un disco Ultra de Azure para **/hana/log** en otra VM dentro de la misma configuración de replicación del sistema HANA.


> [!IMPORTANT]
> Las sugerencias para las configuraciones de almacenamiento se han diseñado como indicaciones con las que empezar. Al ejecutar la carga de trabajo y analizar los patrones de uso del almacenamiento, es posible que se dé cuenta de que no está usando todo el ancho de banda de almacenamiento o IOPS proporcionado. En ese caso, puede considerar la posibilidad de reducir el tamaño del almacenamiento. También puede que, por el contrario, la carga de trabajo necesite más rendimiento de almacenamiento del sugerido con estas configuraciones. Como resultado, es posible que tenga que implementar más capacidad, IOPS o rendimiento. En el campo de la tensión entre la capacidad de almacenamiento necesaria, la latencia de almacenamiento necesaria, el rendimiento de almacenamiento y las IOPS necesarias y la configuración menos costosa, Azure ofrece suficientes tipos de almacenamiento diferentes con distintas funcionalidades y precios de venta para determinar el compromiso adecuado para usted y su carga de trabajo de HANA y adaptarse a este.


## <a name="stripe-sets-versus-sap-hana-data-volume-partitioning"></a>Conjuntos de franjas frente a creación de particiones de volumen de datos de SAP HANA
Con Azure Premium Storage, puede conseguir la mejor relación precio/rendimiento al seccionar el volumen **/hana/data** o **/hana/log** en varios discos de Azure, en lugar de implementar volúmenes de disco mayores que proporcionan más información sobre IOPS o el rendimiento necesario. Hasta ahora esto se lograba con los administradores de volúmenes LVM y MDADM que forman parte de Linux. El método de seccionamiento de discos tiene décadas de antigüedad y es perfectamente conocido. Por muy beneficiosos que sean esos volúmenes seccionados para conseguir las IOPS o las capacidades de rendimiento que pueda necesitar, este método agrega complejidades en torno a la administración de dichos volúmenes, especialmente en los casos en los que es necesario ampliar la capacidad de los volúmenes. Al menos para **/hana/data**, SAP presentó un método alternativo que logra el mismo objetivo que el seccionamiento en varios discos de Azure. A partir de SAP HANA 2.0 SPS03, el servidor de indexación de HANA puede seccionar su actividad de E/S en varios archivos de datos de HANA que se encuentran en diferentes discos de Azure. La ventaja es que no tiene que encargarse de crear y administrar un volumen seccionado en varios discos de Azure. La funcionalidad SAP HANA de la creación de particiones de volúmenes de datos se describe en detalle en:

- [Guía para administradores de HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.05/en-US/40b2b2a880ec4df7bac16eae3daef756.html?q=hana%20data%20volume%20partitioning)
- [Blog sobre SAP HANA: creación de particiones de volúmenes de datos](https://blogs.sap.com/2020/10/07/sap-hana-partitioning-data-volumes/)
- [Nota de SAP n.º 2400005](https://launchpad.support.sap.com/#/notes/2400005)
- [Nota de SAP n.º 2700123](https://launchpad.support.sap.com/#/notes/2700123)

Al leer los detalles, es evidente que aprovechar esta funcionalidad elimina las complejidades de los conjuntos de franjas basados en el administrador de volúmenes. También puede observar que la creación de particiones de volúmenes de datos de HANA no solo funciona para el almacenamiento en bloques de Azure, como Azure Premium Storage. También puede usar esta funcionalidad para seccionar en los recursos compartidos de NFS en caso de que estos recursos compartidos tengan limitaciones de IOPS o rendimiento.  


## <a name="linux-io-scheduler-mode"></a>Modo de programador de E/S de Linux
Linux tiene varios modos diferentes de programación de E/S. Una recomendación común de los proveedores de Linux y SAP consiste en reconfigurar el modo de programador de E/S para los volúmenes de disco del modo **mq-deadline** o **kyber** en el modo **noop** (no multicola) o **none** (multicola) si todavía no lo han hecho los perfiles saptune de SLES. Los detalles se incluyen en: 

- [Nota de SAP n.º 1984787](https://launchpad.support.sap.com/#/notes/1984787)
- [Nota de SAP n.º 2578899](https://launchpad.support.sap.com/#/notes/2578899) 
- [Problema con la configuración de noop en SLES 12 SP4](https://www.suse.com/support/kb/doc/?id=000019547)

En Red Hat, deje la configuración establecida por los perfiles de ajuste específicos para las diferentes aplicaciones SAP.


## <a name="solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines"></a>Soluciones con Premium Storage y el Acelerador de escritura de Azure para máquinas virtuales de la serie M de Azure
El Acelerador de escritura de Azure es una funcionalidad que está disponible exclusivamente para las máquinas virtuales de la serie M de Azure. Como el nombre indica, el propósito de la funcionalidad es mejorar la latencia de E/S de las escrituras en Azure Premium Storage. Para SAP HANA, el Acelerador de escritura se supone que debe usarse solo en el volumen **/hana/log**. Por lo tanto, **/hana/data** y **/hana/log** son volúmenes independientes y el Acelerador de escritura de Azure admite solo el volumen **/hana/log**. 

> [!IMPORTANT]
> Cuando se usa Azure Premium Storage, es obligatorio usar el [Acelerador de escritura](../../how-to-enable-write-accelerator.md) para el volumen **/hana/log**. El Acelerador de escritura solo está disponible para Premium Storage y para las VM de las series M y Mv2. El Acelerador de escritura no funciona en combinación con otras familias de VM de Azure, como Esv3 o Edsv4.

Las recomendaciones de almacenamiento en caché para los siguientes discos Premium de Azure suponen que se dan las siguientes características de E/S para SAP HANA:

- Apenas hay carga de trabajo de lectura en los archivos de datos de HANA. Las excepciones son las operaciones de E/S de tamaño grande después del reinicio de la instancia HANA o cuando se cargan datos en HANA. Otro caso de operaciones de E/S de lectura de mayor tamaño con archivos de datos pueden ser las copias de seguridad de base de datos de HANA. Como resultado, el almacenamiento en caché de lectura no tiene sentido ya que, en la mayoría de los casos, todos los volúmenes de archivos de datos tienen que leerse por completo. 
- La escritura en los archivos de datos se produce en ráfagas según los puntos de retorno de HANA y la recuperación tras bloqueos de HANA. La escritura de los puntos de retorno es asincrónica y no bloquea las transacciones de usuario. La escritura de datos durante la recuperación tras bloqueo es esencial en el rendimiento a fin de lograr que el sistema responda de nuevo rápidamente. Sin embargo, la recuperación tras bloqueo debería constituir una situación excepcional.
- Apenas hay lecturas de los archivos de puesta al día de HANA. Las operaciones de E/S grandes al realizar copias de seguridad del registro de transacciones, la recuperación tras bloqueo o en la fase de reinicio de una instancia de HANA constituyen excepciones.  
- La carga principal en el caso del archivo de registro de puesta al día SAP HANA la constituyen las escrituras. Según la naturaleza de la carga de trabajo, puede tener operaciones de E/S de solo 4 KB o llegar a superar 1 MB. La latencia de las escrituras en relación con el registro de puesta al día SAP HANA es esencial para el rendimiento.
- Todas las operaciones de escritura se deben conservar en el disco de forma confiable.

**Recomendación: Como resultado de estos patrones de E/S observados por SAP HANA, debe establecer el almacenamiento en caché de los diferentes volúmenes con Azure Premium Storage como sigue:**

- **/hana/data**: sin almacenamiento en caché ni almacenamiento en caché de lectura
- **/hana/log**: Sin almacenamiento en caché. Excepción para las VM de las series M y Mv2, en las que el Acelerador de escritura de Azure debe estar habilitado. 
- **/hana/shared**: almacenamiento en caché de lecturas
- **Disco de SO**: no cambie el almacenamiento en caché predeterminado establecido por Azure en el momento de la creación de la VM.


Si usa LVM o mdadm para crear conjuntos de franjas en varios discos Premium de Azure, debe definir tamaños de franja. Estos tamaños difieren entre **/hana/data** y **/hana/log**. **Recomendación: En cuanto a tamaños de franja, la recomendación es usar:**

- 256 KB para **/hana/data**
- 64 KB para **/hana/log**

> [!NOTE]
> El tamaño de franja de **/hana/data** ha cambiado con respecto a las recomendaciones anteriores que precisan de 64 KB o 128 KB a 256 KB según las experiencias del cliente con versiones más recientes de Linux. El tamaño de 256 KB proporciona un rendimiento ligeramente superior. También hemos cambiado la recomendación para los tamaños de franja de **/hana/log** de 32 KB a 64 KB para obtener un rendimiento suficiente con tamaños de E/S superiores.

> [!NOTE]
> No es necesario configurar ningún nivel de redundancia con volúmenes RAID, ya que el almacenamiento en bloque de Azure conserva tres imágenes de un VHD. El uso de un conjunto de franjas con discos Premium de Azure es, simplemente, configurar volúmenes que proporcionen suficientes IOPS o rendimiento de E/S.

La acumulación de un número de VHD de Azure por debajo de un conjunto de franjas es acumulativo por parte de las IOPS y del rendimiento de almacenamiento. Por lo tanto, si coloca un conjunto de franjas en 3 x P30 discos de Azure Premium Storage, debe proporcionarle tres veces las IOPS y tres veces el rendimiento de almacenamiento de un solo disco P30 de Azure Premium Storage.

> [!IMPORTANT]
> Si usa LVM o mdadm como administrador de volúmenes para crear conjuntos de bandas en varios discos de Azure Premium, los tres sistemas de archivos de SAP HANA /data, /log y /shared no se deben colocar en un grupo de volúmenes raíz o predeterminado. Se recomienda seguir la guía de los proveedores de Linux, que normalmente indica crear grupos de volúmenes individuales para /data, /log y /shared.


### <a name="azure-burst-functionality-for-premium-storage"></a>Funcionalidad de ráfaga de Azure para Premium Storage
En el caso de los discos de Azure Premium Storage con una capacidad de 512 GiB o inferior, se ofrece funcionalidad de ráfaga. La forma exacta en que funciona la ráfaga de disco se describe en el artículo [Seguridad de disco](../../disk-bursting.md). Al leer el artículo, comprenderá el concepto de acumulación de IOPS y rendimiento en los casos en que la carga de trabajo de E/S esté por debajo del valor de IOPS y de rendimiento de los discos nominal (para obtener más información sobre el rendimiento nominal, consulte [Precios del disco administrado](https://azure.microsoft.com/pricing/details/managed-disks/)). Acumulará la diferencia de IOPS y rendimiento entre el uso actual y los valores nominales del disco. Las ráfagas están limitadas a un máximo de 30 minutos.

Los casos ideales en los que se puede planear esta funcionalidad de ráfaga serán, probablemente, los volúmenes o discos que contengan archivos de datos para distintos DBMS. Se espera que la carga de trabajo de E/S para esos volúmenes, especialmente con sistemas de gama pequeña o mediana, tenga el aspecto siguiente:

- Carga de trabajo de lectura baja a moderada, ya que los datos, idealmente, se almacenan en la memoria caché, o, en el caso de HANA, deben estar completamente en la memoria.
- Ráfagas de escritura desencadenadas por puntos de control o puntos de retorno de la base de datos que se emiten de forma regular
- Carga de trabajo de copia de seguridad que se lee en un flujo continuo en los casos en que las copias de seguridad no se ejecutan mediante instantáneas de almacenamiento
- En el caso de SAP HANA, carga de los datos en memoria después de reiniciar una instancia

Especialmente en sistemas DBMS más pequeños en los que la carga de trabajo controla solo unos cientos de transacciones por segundo, esta funcionalidad de ráfaga puede tener sentido para los discos o volúmenes que almacenan la transacción o el registro de fase de puesta al día. La carga de trabajo esperada en este disco o volúmenes tiene el siguiente aspecto:

- Las escrituras regulares en el disco que dependen de la carga de trabajo y la naturaleza de la carga de trabajo, ya que es probable que todas las confirmaciones emitidas por la aplicación desencadenen una operación de E/S
- Mayor carga de trabajo en el rendimiento para los casos de tareas operativas, como crear o recompilar índices
- Ráfagas de lectura al realizar copias de seguridad de registros de transacciones o fase de puesta al día


### <a name="production-recommended-storage-solution-based-on-azure-premium-storage"></a>Solución de almacenamiento recomendada para producción basada en Azure Premium Storage

> [!IMPORTANT]
> La certificación de SAP HANA para las máquinas virtuales de serie M de Azure es exclusivamente con el Acelerador de escritura de Azure para el volumen **/hana/log**. Como resultado, las implementaciones de SAP HANA en un escenario de producción en máquinas virtuales de la serie M de Azure se espera que estén configuradas con el Acelerador de escritura de Azure para el volumen **/hana/log**.  

> [!NOTE]
> En escenarios que implican Azure Premium Storage, estamos implementando funcionalidades de ráfaga en la configuración. A medida que use herramientas de prueba de almacenamiento de cualquier forma, tenga en cuenta el [funcionamiento de la expansión de discos Premium de Azure](../../disk-bursting.md). Al ejecutar las pruebas de almacenamiento que proporciona la herramienta HWCCT o HCMT de SAP, no se espera que todas las pruebas cumplan los criterios, ya que algunas pruebas superarán los créditos de expansión que se pueden acumular. Especialmente, cuando todas las pruebas se ejecutan secuencialmente sin interrupción.

> [!NOTE]
> Con las VM M32ts y M32ls, puede ocurrir que el rendimiento del disco sea menor de lo esperado mediante las pruebas de disco HCMT/HWCCT. Esto puede ocurrir incluso con la expansión de disco o con un rendimiento de E/S suficientemente aprovisionado de los discos subyacentes. La causa principal del comportamiento observado era que los archivos de prueba de almacenamiento HCMT/HWCCT estaban completamente almacenados en la caché de lectura de los discos de datos de almacenamiento prémium. Esta caché se encuentra en el host de proceso que hospeda la máquina virtual y que puede almacenar por completo en caché los archivos de prueba de HCMT/HWCCT. En tal caso, las cuotas enumeradas en la columna **Rendimiento máximo de almacenamiento temporal y en caché: IOPS/MBps (tamaño de caché en GiB)** del artículo [Serie M](../../m-series.md) son relevantes. Específicamente para M32ts y M32ls, la cuota de rendimiento en función de la caché de lectura es de solo 400 MB/seg. Como resultado, los archivos de prueba están completamente almacenados en la caché, y es posible que, a pesar de la expansión del disco o de obtener un mayor rendimiento de E/S aprovisionado, las pruebas sean ligeramente inferiores a 400 MB/seg del rendimiento máximo. Como alternativa, puede realizar pruebas sin la caché de lectura habilitada en los discos de datos de Azure Premium Storage.


> [!NOTE]
> Para escenarios de producción, compruebe si un determinado tipo de máquina virtual es compatible con SAP HANA de SAP en la [documentación de SAP para IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html).

**Recomendación: Las configuraciones recomendadas para escenarios de producción de Azure Premium Storage tienen el siguiente aspecto:**

Configuración para el volumen **/hana/data** de SAP:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/data | Rendimiento aprovisionado | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 GiB | 500 MBps | 4 x P6 | 200 MBps | 680 Mbps | 960 | 14 000 |
| M32ls | 256 GiB | 500 MBps | 4 x P6 | 200 MBps | 680 Mbps | 960 | 14 000 |
| M64ls | 512 GB | 1000 MBps | 4 x P10 | 400 MBps | 680 Mbps | 2\.000 | 14 000 |
| M32dms_v2, M32ms_v2 | 875 GiB  | 500 MBps | 4 x P15 | 500 MBps | 680 Mbps | 4400 | 14 000 |
| M64s, M64ds_v2, M64s_v2 | 1024 GiB | 1000 MBps | 4 x P15 | 500 MBps | 680 Mbps | 4400 | 14 000 |
| M64ms, M64dms_v2, M64ms_v2 | 1792 GiB | 1000 MBps | 4 x P20 | 600 MBps | 680 Mbps | 9200 | 14 000 |  
| M128s, M128ds_v2, M128s_v2 | 2048 GiB | 2000 Mbps | 4 x P20 | 600 MBps | 680 Mbps | 9200| 14 000 | 
| M192ds_v2, M192s_v2 | 2048 GiB | 2000 Mbps | 4 x P20 | 600 MBps | 680 Mbps | 9200| 14 000 | 
| M128ms, M128dms_v2, M128ms_v2 | 3892 GiB | 2000 Mbps | 4 x P30 | 800 MBps | sin expansión | 20.000 | sin expansión | 
| M192ms, M192dms_v2, M128ms_v2 | 4096 GiB | 2000 Mbps | 4 x P30 | 800 MBps | sin expansión | 20.000 | sin expansión | 
| M208s_v2 | 2850 GiB | 1000 MBps | 4 x P30 | 800 MBps | sin expansión | 20.000| sin expansión | 
| M208ms_v2 | 5700 GiB | 1000 MBps | 4 x P40 | 1000 MBps | sin expansión | 30,000 | sin expansión |
| M416s_v2 | 5700 GiB | 2000 Mbps | 4 x P40 | 1000 MBps | sin expansión | 30,000 | sin expansión |
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 4 x P50 | 2000 Mbps | sin expansión | 30,000 | sin expansión |


Para el volumen **/hana/log**. La configuración sería similar a la siguiente:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen **/hana/log** | Rendimiento aprovisionado | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 GiB | 500 MBps | 3 x P10 | 300 MBps | 510 Mbps | 1500 | 10 500 | 
| M32ls | 256 GiB | 500 MBps | 3 x P10 | 300 MBps | 510 Mbps | 1500 | 10 500 | 
| M64ls | 512 GB | 1000 MBps | 3 x P10 | 300 MBps | 510 Mbps | 1500 | 10 500 | 
| M32dms_v2, M32ms_v2 | 875 GiB | 500 MBps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 | 
| M64s, M64ds_v2, M64s_v2 | 1024 GiB | 1000 MBps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 | 
| M64ms, M64dms_v2, M64ms_v2 | 1792 GiB | 1000 MBps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 |  
| M128s, M128ds_v2, M128s_v2 | 2048 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500| 
| M192ds_v2, M192s_v2 | 2048 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500| 
| M128ms, M128dms_v2, M128ms_v2 | 3892 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 |
| M192dms_v2, M192ms_v2 | 4096 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 | 
| M208s_v2 | 2850 GiB | 1000 MBps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 |  
| M208ms_v2 | 5700 GiB | 1000 MBps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 |  
| M416s_v2 | 5700 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 |  
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 3 x P15 | 375 MBps | 510 Mbps | 3300 | 10 500 | 


For the other volumes, the configuration would look like:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| M32ts | 192 GiB | 500 MBps | 1 x P15 | 1 x P6 | 1 x P6 |
| M32ls | 256 GiB | 500 MBps |  1 x P15 | 1 x P6 | 1 x P6 |
| M64ls | 512 GB | 1000 MBps | 1 x P20 | 1 x P6 | 1 x P6 |
| M32dms_v2, M32ms_v2 | 875 GiB | 500 MBps | 1 x P30 | 1 x P6 | 1 x P6 |
| M64s, M64ds_v2, M64s_v2 | 1024 GiB | 1000 MBps | 1 x P30 | 1 x P6 | 1 x P6 |
| M64ms, M64dms_v2, M64ms_v2 | 1792 GiB | 1000 MBps | 1 x P30 | 1 x P6 | 1 x P6 | 
| M128s, M128ds_v2, M128s_v2 | 2048 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 | 
| M192ds_v2, M192s_v2  | 2048 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 | 
| M128ms, M128dms_v2, M128ms_v2 | 3892 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 |
| M192dms_v2, M192ms_v2  | 4096 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 |
| M208s_v2 | 2850 GiB | 1000 MBps |  1 x P30 | 1 x P10 | 1 x P6 |
| M208ms_v2 | 5700 GiB | 1000 MBps | 1 x P30 | 1 x P10 | 1 x P6 | 
| M416s_v2 | 5700 GiB | 2000 Mbps |  1 x P30 | 1 x P10 | 1 x P6 | 
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 | 


Compruebe si el rendimiento del almacenamiento para los diferentes volúmenes sugeridos se adapta a la carga de trabajo que se va a ejecutar. Si la carga de trabajo requiere volúmenes mayores para **/hana/data** y **/hana/log**, debe aumentar el número de VHD de Azure Premium Storage. Ajustar el tamaño de un volumen con más discos duros virtuales de los que se enumeran, aumenta el IOPS y el rendimiento de E/S dentro de los límites del tipo de máquina virtual de Azure.

El Acelerador de escritura de Azure solo funciona con [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/). Así pues, al menos los discos de Azure Premium Storage que forman el volumen **/hana/log** deben implementarse como discos administrados. Puede encontrar instrucciones y restricciones del Acelerador de escritura de Azure en el artículo [Acelerador de escritura](../../how-to-enable-write-accelerator.md).

En el caso de las VM certificadas para HANA de la familia de [Esv3](../../ev3-esv3-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#esv3-series) y [Edsv4](../../edv4-edsv4-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#edsv4-series), debe aplicar ANF para los volúmenes **/hana/data** y **/hana/log**. O bien debe aprovechar el almacenamiento en disco Ultra de Azure en lugar de Azure Premium Storage solo para el volumen **/hana/log**. Como resultado, las configuraciones del volumen **/hana/data** en Azure Premium Storage podrían tener este aspecto:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/data | Rendimiento aprovisionado | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- |
| E20ds_v4 | 160 GiB | 480 Mbps | 3 x P10 | 300 MBps | 510 Mbps | 1500 | 10 500 |
| E32ds_v4 | 256 GiB | 768 MBps | 3 x P10 |  300 MBps | 510 Mbps | 1500 | 10 500|
| E48ds_v4 | 384 GiB | 1152 Mbps | 3 x P15 |  375 MBps |510 Mbps | 3300  | 10 500 | 
| E64ds_v4 | 504 GiB | 1200 MBps | 3 x P15 |  375 MBps | 510 Mbps | 3300 | 10 500 | 
| E64s_v3 | 432 GiB | 1200 MB/s | 3 x P15 |  375 MBps | 510 Mbps | 3300 | 10 500 | 

En el caso del resto de volúmenes, incluido **/hana/log** del disco Ultra, la configuración podría tener este aspecto:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen /hana/log | Rendimiento de E/S de /hana/log | Operaciones de E/S por segundo en /hana/log | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 GiB | 480 Mbps | 80 GB | 250 MBps | 1800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E32ds_v4 | 256 GiB | 768 MBps | 128 GB | 250 MBps | 1800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E48ds_v4 | 384 GiB | 1152 Mbps | 192 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64ds_v4 | 504 GiB | 1200 MBps | 256 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64s_v3 | 432 GiB | 1200 MBps | 220 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |


## <a name="azure-ultra-disk-storage-configuration-for-sap-hana"></a>Configuración de almacenamiento en el disco Ultra de Azure para SAP HANA
Otro tipo de almacenamiento de Azure se denomina [disco Ultra de Azure](../../disks-types.md#ultra-disk). Una diferencia considerable entre el almacenamiento de Azure que se ofrecía hasta ahora y el disco Ultra es que las funcionalidades del disco ya no están asociadas a su tamaño. Como un cliente, para el disco Ultra puede definir estas funcionalidades:

- El tamaño de disco, de 4 GiB a 65 536 GiB
- Intervalo de 100 operaciones de E/S por segundo a 160 000 operaciones de E/S por segundo (el máximo depende también de los tipos de máquina virtual)
- Rendimiento del almacenamiento de 300 MB/s a 2000 MB por segundo

El disco Ultra ofrece la posibilidad de definir un único disco que responda a su rango de necesidades de tamaño, IOPS y rendimiento de disco. En lugar de usar administradores de volúmenes lógicos como LVM o MDADM además de Azure Premium Storage para construir los volúmenes que cumplan los requisitos de rendimiento de almacenamiento y de IOPS. Puede ejecutar una configuración combinada de disco Ultra y Premium Storage. Como resultado, puede limitar el uso del disco Ultra a los volúmenes **/hana/data** y **/hana/log** críticos para el rendimiento y cubrir el resto de volúmenes con Azure Premium Storage.

Otra ventaja del disco Ultra puede ser una mejor latencia de lectura en comparación con Premium Storage. Una latencia de lectura más rápida puede tener ventajas si desea reducir los tiempos de inicio de HANA y la carga posterior de los datos en la memoria. También se pueden apreciar las ventajas del almacenamiento en disco Ultra cuando HANA está escribiendo puntos de retorno. 

> [!NOTE]
> El disco Ultra todavía no está presente en todas las regiones de Azure y aún no es compatible con todos los tipos de máquinas virtuales que se indican a continuación. Para obtener información detallada sobre dónde está disponible el disco Ultra y qué familias de máquinas virtuales se admiten, consulte el artículo [¿Qué tipos de disco están disponibles en Azure?](../../disks-types.md#ultra-disk).

### <a name="production-recommended-storage-solution-with-pure-ultra-disk-configuration"></a>Solución de almacenamiento recomendada para producción con configuración de disco Ultra pura
En esta configuración, puede conservar los volúmenes **/hana/data** y **/hana/log** por separado. Los valores sugeridos se derivan de los KPI que SAP tiene para certificar los tipos de máquina virtual de SAP HANA y las configuraciones de almacenamiento como se recomienda en las [notas del producto SAP TDI Storage](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html).

A menudo, las recomendaciones superan los requisitos mínimos de SAP como se indicó anteriormente en este artículo. Las recomendaciones enumeradas son un compromiso entre las recomendaciones de tamaño de SAP y el rendimiento de almacenamiento máximo que proporcionan los diferentes tipos de máquinas virtuales.

> [!NOTE]
> El disco Ultra de Azure exige un mínimo de 2 IOPS por capacidad de Gigabyte de un disco


| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen /hana/data | Rendimiento de E/S de /hana/data | Operaciones de E/S por segundo en /hana/data | Volumen /hana/log | Rendimiento de E/S de /hana/log | Operaciones de E/S por segundo en /hana/log |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 GiB | 480 MB/s | 200 GB | 400 MBps | 2,500 | 80 GB | 250 MB | 1800 |
| E32ds_v4 | 256 GiB | 768 MB/s | 300 GB | 400 MBps | 2,500 | 128 GB | 250 MBps | 1800 |
| E48ds_v4 | 384 GiB | 1152 MB/s | 460 GB | 400 MBps | 3,000 | 192 GB | 250 MBps | 1800 |
| E64ds_v4 | 504 GiB | 1200 MB/s | 610 GB | 400 MBps | 3500 |  256 GB | 250 MBps | 1800 |
| E64s_v3 | 432 GiB | 1200 MB/s | 610 GB | 400 MBps | 3500 | 220 GB | 250 MB | 1800 |
| M32ts | 192 GiB | 500 MB/s | 250 GB | 400 MBps | 2,500 | 96 GB | 250 MBps  | 1800 |
| M32ls | 256 GiB | 500 MB/s | 300 GB | 400 MBps | 2,500 | 256 GB | 250 MBps  | 1800 |
| M64ls | 512 GB | 1000 MB/s | 620 GB | 400 MBps | 3500 | 256 GB | 250 MBps  | 1800 |
| M32dms_v2, M32ms_v2 | 875 GiB | 500 MB/s |  1200 GB | 600 MBps | 5\.000 | 512 GB | 250 MBps  | 2,500 |
| M64s, M64ds_v2, M64s_v2 | 1024 GiB | 1000 MB/s |  1200 GB | 600 MBps | 5\.000 | 512 GB | 250 MBps  | 2,500 |
| M64ms, M64dms_v2, M64ms_v2 | 1792 GiB | 1000 MB/s | 2100 GB | 600 MBps | 5\.000 | 512 GB | 250 MBps  | 2,500 |
| M128s, M128ds_v2, M128s_v2 | 2048 GiB | 2000 MB/s |2400 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2,500 |
| M192ds_v2, M192s_v2 | 2048 GiB | 2000 MB/s |2400 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2,500 | 
| M128ms, M128dms_v2, M128ms_v2 | 3892 GiB | 2000 MB/s | 4800 GB | 750 Mbps |9 600 | 512 GB | 250 MBps  | 2,500 | 
| M192dms_v2, M192ms_v2 | 4096 GiB | 2000 MB/s | 4800 GB | 750 Mbps |9 600 | 512 GB | 250 MBps  | 2,500 | 
| M208s_v2 | 2850 GiB | 1000 MB/s | 3500 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2,500 | 
| M208ms_v2 | 5700 GiB | 1000 MB/s | 7200 GB | 750 Mbps | 14 400 | 512 GB | 250 MBps  | 2,500 | 
| M416s_v2 | 5700 GiB | 2000 MB/s | 7200 GB | 1000 MBps | 14 400 | 512 GB | 400 MBps  | 4\.000 | 
| M416ms_v2 | 11 400 GiB | 2000 MB/s | 14 400 GB | 1500 MBps | 28,800 | 512 GB | 400 MBps  | 4\.000 |   

**Los valores de la lista están diseñados como punto de partida y deben valorarse según las demandas reales.** La ventaja del disco Ultra de Azure es que los valores de IOPS y de rendimiento se pueden adaptar sin necesidad de apagar la máquina virtual o detener la carga de trabajo que se aplica al sistema.   

> [!NOTE]
> Por ahora las instantáneas de almacenamiento con el disco Ultra no están disponibles. Esto bloquea el uso de las instantáneas de máquina virtual con los servicios de Azure Backup


## <a name="nfs-v41-volumes-on-azure-netapp-files"></a>Volúmenes de NFS v4.1 en Azure NetApp Files
Para información detallada sobre ANF para HANA, lea el documento [Volúmenes NFS v4.1 en Azure NetApp Files para SAP HANA](./hana-vm-operations-netapp.md)




## <a name="cost-conscious-solution-with-azure-premium-storage"></a>Solución con control de costo con Azure Premium Storage
Hasta ahora, la solución de Azure Premium Storage que se describe en este documento en la sección [Soluciones con Premium Storage y el Acelerador de escritura de Azure para máquinas virtuales de la serie M de Azure](#solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines) estaba diseñada para escenarios admitidos de producción de SAP HANA. Una de las características de las configuraciones admitidas con la producción es la separación de los volúmenes de datos de SAP HANA y el registro de la fase de puesta al día en dos volúmenes diferentes. La razón de esta separación es que las características de la carga de trabajo en los volúmenes son diferentes. Además, con las configuraciones de producción sugeridas, podría ser necesario un tipo de almacenamiento en caché diferente o incluso diferentes tipos de almacenamiento de bloques de Azure. En escenarios que no son de producción, es posible que algunas de las consideraciones tomadas para los sistemas de producción no se apliquen a sistemas que no sean de producción de menor calidad. Como resultado, se pueden combinar los datos y el volumen de registro de HANA. Aunque eventualmente con algunos culpables, como por ejemplo no cumplir con ciertos KPI de rendimiento o latencia que se requieren para los sistemas de producción. Otro aspecto para reducir costos en estos entornos puede ser el uso del [almacenamiento SSD estándar de Azure](./planning-guide-storage.md#azure-standard-ssd-storage). Tenga en cuenta que elegir Azure Storage de SSD estándar o HDD estándar afecta a los Acuerdos de Nivel de Servicio para una única máquina virtual, tal como se documenta en el artículo [Acuerdo de Nivel de Servicio para Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines).

Una alternativa menos costosa para estas configuraciones podría ser similar a la siguiente:


| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/data y /hana/log<br /> seccionado con LVM o MDAADM | /hana/shared | /root volume | /usr/sap | comments |
| --- | --- | --- | --- | --- | --- | --- | -- |
| DS14v2 | 112 GiB | 768 MB/s | 4 x P6 | 1 x E10 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| E16v3 | 128 GB | 384 MB/s | 4 x P6 | 1 x E10 | 1 x E6 | 1 x E6 | Tipo de máquina virtual no certificado por HANA <br /> No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| M32ts | 192 GiB | 500 MB/s | 3 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 5 000<sup>2</sup>. |
| E20ds_v4 | 160 GiB | 480 MB/s | 4 x P6 | 1 x E15 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| E32v3 | 256 GiB | 768 MB/s | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | Tipo de máquina virtual no certificado por HANA <br /> No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| E32ds_v4 | 256 GiB | 768 MBps | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| M32ls | 256 GiB | 500 MB/s | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 5 000<sup>2</sup>. |
| E48ds_v4 | 384 GiB | 1152 Mbps | 6 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| E64v3 | 432 GiB | 1200 MB/s | 6 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| E64ds_v4 | 504 GiB | 1200 MB/s |  7 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | No obtendrá una latencia de almacenamiento inferior a 1 ms<sup>1</sup>. |
| M64ls | 512 GB | 1000 MB/s | 7 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 10 000<sup>2</sup>. |
| M32dms_v2, M32ms_v2 | 875 GiB | 500 MB/s | 6 x P15 | 1 x E30 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 5 000<sup>2</sup>. |
| M64s, M64ds_v2, M64s_v2 | 1024 GiB | 1000 MB/s | 7 x P15 | 1 x E30 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 10 000<sup>2</sup>. |
| M64ms, M64dms_v2, M64ms_v2| 1792 GiB | 1000 MB/s | 6 x P20 | 1 x E30 | 1 x E6 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 10 000<sup>2</sup>. |
| M128s, M128ds_v2, M128s_v2 | 2048 GiB | 2000 MB/s |6 x P20 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |
| M192ds_v2, M192s_v2 | 2048 GiB | 2000 MB/s |6 x P20 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |
| M128ms, M128dms_v2, M128ms_v2  | 3800 GiB | 2000 MB/s | 5 x P30 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |
| M192dms_v2, M192ms_v2  | 4096 GiB | 2000 MB/s | 5 x P30 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |
| M208s_v2 | 2850 GiB | 1000 MB/s | 4 x P30 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 10 000<sup>2</sup>. |
| M208ms_v2 | 5700 GiB | 1000 MB/s | 4 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 10 000<sup>2</sup>. |
| M416s_v2 | 5700 GiB | 2000 MB/s | 4 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |
| M416ms_v2 | 11400 GiB | 2000 MB/s | 7 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | El uso de Acelerador de escritura para los datos combinados y el volumen de registro limitará la tasa de IOPS a 20 000<sup>2</sup>. |


<sup>1</sup> [Acelerador de escritura de Azure](../../how-to-enable-write-accelerator.md) no se puede usar con las familias de máquinas virtuales Ev4. Como resultado del uso de Azure Premium Storage, la latencia de E/S no será inferior a 1 ms.

<sup>2</sup> La familia de máquinas virtuales admite [Acelerador de escritura de Azure](../../how-to-enable-write-accelerator.md), pero existe la posibilidad de que el límite de IOPS de Acelerador de escritura limite las funcionalidades de IOPS de las configuraciones de disco.

En el caso de combinar el volumen de datos y de registro para SAP HANA, los discos que forman el volumen seccionado no deben tener habilitada la memoria caché de lectura o escritura.

Se muestran los tipos de máquina virtual que no están certificados con SAP y, como tal, no se incluyen en el denominado [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Los comentarios de los clientes fueron que los tipos de máquina virtual no enumerados se usaban correctamente para algunas tareas que no son de producción.


## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte:

- [Guía sobre la alta disponibilidad de SAP HANA para máquinas virtuales de Azure](./sap-hana-availability-overview.md).