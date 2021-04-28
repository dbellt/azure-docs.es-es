---
title: Configuración de ANF de máquinas virtuales de Azure en SAP HANA | Microsoft Docs
description: Recomendaciones de almacenamiento de Azure NetApp Files para SAP HANA.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP, Azure, ANF, HANA, Azure NetApp Files, instantánea
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 01/23/2021
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 09fc8f9697f418533131e86c069afd3157a71c78
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108142988"
---
# <a name="nfs-v41-volumes-on-azure-netapp-files-for-sap-hana"></a>Volúmenes NFS v4.1 en Azure NetApp Files para SAP HANA

Azure NetApp Files proporciona recursos compartidos de NFS nativos que se pueden usar para los volúmenes **/hana/shared**, **/hana/data** y **/hana/log**. El uso de recursos compartidos de NFS basados en ANF para los volúmenes **/hana/data** y **/hana/log** requiere el uso del protocolo NFS v4.1. El protocolo NFS v3 no admite el uso de los volúmenes **/hana/data** y **/hana/log** al basar los recursos compartidos en ANF. 


> [!IMPORTANT]
> **No** se admite el protocolo NFS v3 implementado en Azure NetApp Files para su uso con **/hana/data** y **/hana/log**. El uso de NFS 4.1 es obligatorio para los volúmenes **/hana/data** y **/hana/log** desde un punto de vista funcional. Por su parte, para el volumen **/hana/shared**, se puede usar el protocolo NFS v3 o NFS v4.1 desde un punto de vista funcional.

## <a name="important-considerations"></a>Consideraciones importantes

A la hora de considerar Azure NetApp Files para SAP Netweaver y SAP HANA, tenga en cuenta los siguientes aspectos importantes:

- El grupo de capacidad mínimo es de 4 TiB.  
- El tamaño del volumen mínimo es de 100 GiB.
- Azure NetApp Files, y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montan, se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](../../../virtual-network/virtual-network-peering-overview.md) de la misma región.
- Es importante que las máquinas virtuales se implementen muy cerca del almacenamiento de Azure NetApp para conseguir una latencia baja.  
- La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
- Asegúrese de que la latencia del servidor de base de datos en el volumen de ANF se mide y está por debajo de 1 milisegundo.
- El rendimiento de un volumen de Azure NetApp es una función de la cuota del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Al ajustar el tamaño de los volúmenes de Azure NetApp de HANA, asegúrese de que el rendimiento resultante cumple los requisitos del sistema HANA.
- Intente "consolidar" volúmenes para obtener más rendimiento en un volumen de mayor tamaño; por ejemplo, use un volumen para /sapmnt, /usr/sap/trans, etc., si es posible.  
- Azure NetApp Files ofrece la [directiva de exportación](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md): puede controlar los clientes permitidos, el tipo de acceso (lectura y escritura, solo lectura, etc.). 
- La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica Azure NetApp Files no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure.   
- El identificador de usuario para <b>sid</b>adm y el identificador de grupo para `sapsys` en las máquinas virtuales deben coincidir con la configuración de Azure NetApp Files. 

> [!IMPORTANT]
> Para las cargas de trabajo de SAP HANA, una baja latencia resulta fundamental. Trabaje con su representante de Microsoft para asegurarse de que las máquinas virtuales y los volúmenes de Azure NetApp Files se implementan en ubicaciones muy cercanas entre sí.  

> [!IMPORTANT]
> Si hubiera una discrepancia entre el id. de usuario para <b>sid</b>adm y el id. de grupo para `sapsys` entre la máquina virtual y la configuración de Azure NetApp, los permisos de archivos en volúmenes de Azure NetApp, montados en la máquina virtual, aparecerían como `nobody`. Asegúrese de especificar el identificador de usuario correcto para <b>sid</b>adm y el identificador de grupo para `sapsys` al [incorporar un nuevo sistema](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u) a Azure NetApp Files.


## <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Ajuste del tamaño de la base de datos HANA en Azure NetApp Files

El rendimiento de un volumen de Azure NetApp es una función del tamaño del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

Es importante comprender el tamaño de la relación de rendimiento y que hay límites físicos para una LIF (interfaz lógica) de la SVM (máquina virtual de almacenamiento).

En la tabla siguiente se muestra que podría tener sentido crear un volumen "Estándar" de gran tamaño para almacenar las copias de seguridad y que no tiene sentido crear un volumen "Ultra" mayor de 12 TB, porque se superaría la capacidad de ancho de banda físico de un solo LIF. 

El rendimiento máximo de un LIF y una única sesión de Linux está comprendido entre 1,2 y 1,4 GB/s. Si necesita más rendimiento para /hana/data, puede usar la creación de particiones del volumen de datos de SAP HANA para fragmentar la actividad de E/S durante la recarga de datos o los puntos de retorno de HANA en varios archivos de datos de HANA ubicados en varios recursos compartidos de NFS. Para obtener más detalles sobre la fragmentación del volumen de datos de HANA, lea estos artículos:

- [Guía para administradores de HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.05/en-US/40b2b2a880ec4df7bac16eae3daef756.html?q=hana%20data%20volume%20partitioning)
- [Blog sobre SAP HANA: creación de particiones de volúmenes de datos](https://blogs.sap.com/2020/10/07/sap-hana-partitioning-data-volumes/)
- [Nota de SAP n.º 2400005](https://launchpad.support.sap.com/#/notes/2400005)
- [Nota de SAP #2700123](https://launchpad.support.sap.com/#/notes/2700123)


| Size  | Rendimiento Estándar | Rendimiento Premium | Rendimiento Ultra |
| --- | --- | --- | --- |
| 1 TB | 16 MB/s | 64 MB/s | 128 MB/s |
| 2 TB | 32 MB/s | 128 MB/s | 256 MB/s |
| 4 TB | 64 MB/s | 256 MB/s | 512 MB/s |
| 10 TB | 160 MB/s | 640 MB/s | 1280 MB/s |
| 15 TB | 240 MB/s | 960 MB/s | 1400 MB/s |
| 20 TB | 320 MB/s | 1280 MB/s | 1400 MB/s |
| 40 TB | 640 MB/s | 1400 MB/s | 1400 MB/s |

Es importante comprender que los datos se escriben en los mismos discos SSD en el back-end de almacenamiento. Se ha creado la cuota de rendimiento del grupo de capacidad para poder administrar el entorno.
Los KPI de almacenamiento son iguales para todos los tamaños de base de datos de HANA. En casi todos los casos, esta hipótesis no refleja la realidad y la expectativa del cliente. El tamaño de los sistemas de HANA no implica necesariamente que un sistema pequeño requiera un rendimiento de almacenamiento reducido, ni que un sistema de gran tamaño requiera un rendimiento alto de almacenamiento. No obstante, por lo general se pueden esperar mayores requisitos de rendimiento para las instancias de base de datos de HANA de mayor tamaño. Como resultado de las reglas de dimensionamiento de SAP para el hardware subyacente, estas instancias de HANA de mayor tamaño también proporcionan más recursos de CPU y un paralelismo superior en tareas como la carga de datos después del reinicio de una instancia. En consecuencia, se deben adoptar los tamaños de volumen para las expectativas y los requisitos del cliente, y no solo controlados por requisitos de capacidad puros.

Al diseñar la infraestructura de SAP en Azure, debe tener en cuenta los requisitos mínimos de rendimiento del almacenamiento (para sistemas de producción) de SAP, lo cual se traduce en estas características de rendimiento mínimas:

| Tipo de volumen y tipo de E/S | KPI mínimo solicitado por SAP | Nivel de servicio Premium | Nivel de servicio Ultra |
| --- | --- | --- | --- |
| Escritura de volumen de registro | 250 MB/s | 4 TB | 2 TB |
| Escritura de volumen de datos | 250 MB/s | 4 TB | 2 TB |
| Lectura de volumen de datos | 400 MB/s | 6,3 TB | 3,2 TB |

Dado que se exigen los tres KPI, el volumen de **/hana/data** debe dimensionarse a la capacidad de mayor tamaño para cumplir los requisitos mínimos de lectura.

En el caso de los sistemas HANA, que no requieren ancho de banda grande, los tamaños de los volúmenes de ANF pueden ser más pequeños. Y, en caso de que un sistema HANA requiera más rendimiento, el volumen podría adaptarse mediante el cambio de tamaño de la capacidad en línea. No se ha definido ningún KPI para los volúmenes de copia de seguridad. Sin embargo, el rendimiento del volumen de copia de seguridad es fundamental para un entorno que funcione correctamente. El rendimiento del volumen de registros y datos debe estar diseñado para cumplir con las expectativas de los clientes.

> [!IMPORTANT]
> Independientemente de la capacidad que implemente en un volumen NFS individual, se espera que el rendimiento se nivele en el rango de 1,2-1,4 GB/s de ancho de banda usado por un consumidor en una máquina virtual. Esto tiene que ver con la arquitectura subyacente de la oferta de ANF y los límites de sesiones de Linux relacionadas alrededor de NFS. Las cifras relativas al rendimiento y a la capacidad de proceso que se indican en el artículo [Banco de pruebas de rendimiento de resultados de pruebas para Azure NetApp Files](../../../azure-netapp-files/performance-benchmarks-linux.md) se obtuvieron en un volumen NFS compartido con varias máquinas virtuales cliente y como resultado con varias sesiones. Dicho escenario es diferente del escenario que medimos en SAP. Ahí medimos la capacidad de procesamiento de una sola máquina virtual en un volumen NFS Hospedado en ANF.

Para cumplir los requisitos de rendimiento mínimo de SAP para los datos y el registro, y de acuerdo con las directrices para **/hana/shared**, los tamaños recomendados serían los siguientes:

| Volumen | Size<br /> Capa Premium Storage | Size<br /> Capa de almacenamiento Ultra | Protocolo NFS admitido |
| --- | --- | --- |
| /hana/log/ | 4 TiB | 2 TiB | v4.1 |
| /hana/data | 6,3 TiB | 3,2 TiB | v4.1 |
| /hana/shared scale-up | Mín. (1 TB, 1 x RAM)  | Mín. (1 TB, 1 x RAM) | v3 o v4.1 |
| /hana/shared scale-out | 1 x RAM del nodo de trabajo<br /> por 4 nodos de trabajo  | 1 x RAM del nodo de trabajo<br /> por 4 nodos de trabajo  | v3 o v4.1 |
| /hana/logbackup | 3 x RAM  | 3 x RAM | v3 o v4.1 |
| /hana/backup | 2 x RAM  | 2 x RAM | v3 o v4.1 |

En todos los volúmenes, NFS v4.1 es muy recomendable.

Los tamaños de los volúmenes de copia de seguridad son estimaciones. Los requisitos exactos deben definirse en función de la carga de trabajo y los procesos de funcionamiento. En el caso de las copias de seguridad, podría consolidar muchos volúmenes para diferentes instancias de SAP HANA en uno o dos volúmenes de mayor tamaño, lo que podría tener un nivel de servicio de ANF inferior.

> [!NOTE]
> Las recomendaciones de dimensionamiento de Azure NetApp Files que se han indicado en este documento tienen como objetivo satisfacer los requisitos mínimos que SAP expresa a sus proveedores de infraestructura. En escenarios reales de implementaciones de clientes y de cargas de trabajo, es posible que no sea suficiente. Utilice estas recomendaciones como punto de partida y adáptelas en función de los requisitos de la carga de trabajo específica.  

Por tanto, podría considerar la posibilidad de implementar un rendimiento similar para los volúmenes de ANF, tal como se indicó anteriormente en el almacenamiento de disco Ultra. Tenga en cuenta también los tamaños enumerados para los volúmenes de las diferentes SKU de máquina virtual como ya se ha hecho en las tablas de disco Ultra.

> [!TIP]
> Puede cambiar el tamaño de los volúmenes de Azure NetApp Files de manera dinámica, sin necesidad de `unmount` los volúmenes, detener las máquinas virtuales o detener SAP HANA. Esto permite que la aplicación se adapte a las demandas de rendimiento esperadas e imprevistas.

La documentación sobre cómo implementar una configuración de escalabilidad horizontal de SAP HANA con un nodo de espera mediante los volúmenes de NFS v4.1 hospedados en ANF y publicados en [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md).


## <a name="availability"></a>Disponibilidad
Las actualizaciones del sistema y las actualizaciones de ANF se aplican sin afectar al entorno del cliente. Se define un [Acuerdo de nivel de servicio del 99,99 %](https://azure.microsoft.com/support/legal/sla/netapp/).


## <a name="volumes-and-ip-addresses-and-capacity-pools"></a>Volúmenes, direcciones IP y grupos de capacidades
Con ANF, es importante comprender cómo se compila la infraestructura subyacente. Un grupo de capacidad tan solo es una estructura, lo que simplifica la creación de un modelo de facturación para ANF. Un grupo de capacidad no tiene ninguna relación física con la infraestructura subyacente. Si crea un grupo de capacidad, tan solo se crea un shell que se puede cargar. Cuando se crea un volumen, la primera SVM (Máquina virtual de almacenamiento) se crea en un clúster de varios sistemas NetApp. Se crea una única dirección IP para que esta SVM acceda al volumen. Si crea varios volúmenes, todos estos se distribuyen en esta SVM a través de este clúster de NetApp de varios controladores. Incluso si solo obtiene una dirección IP, los datos se distribuyen en varios controladores. ANF tiene una lógica que distribuye de forma automática las cargas de trabajo de los clientes cuando los volúmenes o la capacidad del almacenamiento configurado alcanzan un nivel interno predefinido. Es posible que observe estos casos porque se asigna una nueva dirección IP para acceder a los volúmenes.

##<a name="log-volume-and-log-backup-volume"></a>Volumen de registro y volumen de copia de seguridad de registros
El "volumen de registro" ( **/hana/log**) se usa para escribir el registro de la fase de puesta al día en línea. Por lo tanto, hay archivos abiertos ubicados en este volumen y no tiene sentido crear una instantánea del mismo. Los archivos de registro en línea de la fase de puesta al día se archivan, o se hace una copia de seguridad de estos, en el volumen de copia de seguridad de registros cuando el archivo de registro en línea de la fase de puesta al día está completo o se ejecuta una copia de seguridad de registros de la fase de puesta al día. Para proporcionar un rendimiento de copia de seguridad razonable, el volumen de copia de seguridad de registros requiere un buen rendimiento. Para optimizar los costos de almacenamiento, puede tener sentido consolidar el volumen de copia de seguridad de registros de varias instancias de HANA. Esto es así para que varias instancias de HANA aprovechen el mismo volumen y escriban sus copias de seguridad en directorios diferentes. Con este tipo de consolidación, se puede obtener más rendimiento, ya que es necesario aumentar ligeramente el tamaño del volumen. 

Esto mismo se aplica al volumen que se usa para escribir copias de seguridad completas de bases de datos de HANA.  
 

## <a name="backup"></a>Copia de seguridad
Además de las copias de seguridad de streaming y el servicio Azure Backup, que realiza copias de seguridad de bases de datos de SAP HANA, tal como se describe en el artículo [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](../../../backup/sap-hana-db-about.md), Azure NetApp Files permite realizar copias de seguridad de instantáneas basadas en almacenamiento. 

SAP HANA admite lo siguiente:

- Copias de seguridad de instantáneas basadas en almacenamiento a partir de SAP HANA 1.0 SPS7
- Compatibilidad con copias de seguridad de instantáneas basadas en almacenamiento para entornos de HANA de Contenedor de varias bases de datos (MDC) a partir de SAP HANA 2.0 SPS4


La creación de copias de seguridad de instantáneas basadas en almacenamiento es un procedimiento sencillo que consta de cuatro pasos: 
1. Creación de una instantánea de base de datos de HANA (interna): una actividad que debe realizar por su cuenta o mediante herramientas. 
1. SAP HANA escribe datos en los archivos de datos para crear un estado coherente en el almacenamiento: HANA realiza este paso como resultado de la creación de una instantánea de HANA.
1. Creación de una instantánea en el volumen **/hana/data** volumen en el almacenamiento: un paso que debe realizar por su cuenta o mediante herramientas. No es necesario realizar una instantánea en el volumen **/hana/log**.
1. Eliminación de la instantánea de base de datos de HANA (interna) y reanudación del funcionamiento normal: un paso que debe realizar por su cuenta o mediante herramientas.

> [!WARNING]
> En caso de que falte el último paso o este no se pueda realizar, se producirá un impacto grave en la demanda de memoria de SAP HANA y podrá provocar una detención de SAP HANA.

```
BACKUP DATA FOR FULL SYSTEM CREATE SNAPSHOT COMMENT 'SNAPSHOT-2019-03-18:11:00';
```

![Copia de seguridad de instantáneas de ANF para SAP HANA](media/hana-vm-operations-netapp/storage-snapshot-scenario.png)

```
az netappfiles snapshot create -g mygroup --account-name myaccname --pool-name mypoolname --volume-name myvolname --name mysnapname 
```

![Copia de seguridad de instantáneas de ANF para SAP HANA, parte 2](media/hana-vm-operations-netapp/storage-snapshot.png)

```
BACKUP DATA FOR FULL SYSTEM CLOSE SNAPSHOT BACKUP_ID 47110815 SUCCESSFUL SNAPSHOT-2020-08-18:11:00';
```

Este procedimiento de copia de seguridad de instantáneas se puede administrar de varias maneras, mediante el uso de diversas herramientas. Un ejemplo de esto es el script de Python "ntaphana_azure.py" disponible en GitHub [https://github.com/netapp/ntaphana](https://github.com/netapp/ntaphana) Este es un código de ejemplo, proporcionado "tal cual" sin ningún mantenimiento o soporte técnico.



> [!CAUTION]
> Una instantánea en sí misma no es una copia de seguridad protegida, ya que se encuentra en el mismo almacenamiento físico que el volumen del que se ha tomado una instantánea. Es obligatorio "proteger" al menos una instantánea por día en otra ubicación. Esto puede hacerse en el mismo entorno, en una región de Azure remota o en almacenamiento de blobs de Azure.


En el caso de los usuarios de productos de copia de seguridad de CommVault, una segunda opción es CommVault IntelliSnap V.11.21 y versiones posteriores. Esta versión de CommVault y las posteriores ofrecen compatibilidad con Azure NetApp Files. En el artículo [CommVault IntelliSnap 11.21](https://documentation.commvault.com/11.21/essential/116350_getting_started_with_backup_and_restore_operations_for_azure_netapp_file_services_smb_shares_and_nfs_shares.html) se proporciona más información.


### <a name="back-up-the-snapshot-using-azure-blob-storage"></a>Copia de seguridad de la instantánea mediante almacenamiento de blobs de Azure
La realización de una copia de seguridad en el almacenamiento de blobs de Azure es un método rápido y rentable para guardar copias de seguridad de instantáneas de almacenamiento de base de datos de HANA basadas en ANF. Para guardar las instantáneas en el almacenamiento de blobs de Azure, es preferible usar la herramienta azcopy. Descargue la versión más reciente de esta herramienta e instálela, por ejemplo, en el directorio bin donde esté instalado el script de Python de GitHub.
Descargue la versión más reciente de la herramienta azcopy:

```
root # wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1
Saving to: ‘azcopy_v10.tar.gz’
```

La característica más avanzada es la opción SYNC. Si se usa la opción SYNC, azcopy mantiene sincronizado el directorio de origen y el de destino. El uso del parámetro **--delete-destination** es importante. Sin este parámetro, azcopy no elimina los archivos del sitio de destino y el uso de espacio en el lado de destino aumentaría. Cree un contenedor de Blob en bloques en una cuenta de almacenamiento de Azure. Después, cree la clave SAS para el contenedor de blobs y sincronice la carpeta de instantáneas con el contenedor de blobs de Azure.

Por ejemplo, si se debe sincronizar una instantánea diaria con el contenedor de blobs de Azure para proteger los datos y solo se debe conservar una instantánea, se puede usar el comando siguiente.

```
root # > azcopy sync '/hana/data/SID/mnt00001/.snapshot' 'https://azacsnaptmytestblob01.blob.core.windows.net/abc?sv=2021-02-02&ss=bfqt&srt=sco&sp=rwdlacup&se=2021-02-04T08:25:26Z&st=2021-02-04T00:25:26Z&spr=https&sig=abcdefghijklmnopqrstuvwxyz' --recursive=true --delete-destination=true
```

## <a name="next-steps"></a>Pasos siguientes
Lea el artículo:

- [Alta disponibilidad de SAP HANA para máquinas virtuales de Azure](./sap-hana-availability-overview.md)