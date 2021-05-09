---
title: Recuperación ante desastres con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: En este artículo se explica cómo realizar la recuperación ante desastres con la herramienta Azure Application Consistent Snapshot, la cual se puede usar con Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: ad91260eac0f638d339baf64071cad6475ac19a7
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929820"
---
# <a name="disaster-recovery-using-azure-application-consistent-snapshot-tool"></a>Recuperación ante desastres con la herramienta Azure Application Consistent Snapshot

En este artículo se explica cómo realizar la recuperación ante desastres con la herramienta Azure Application Consistent Snapshot, la cual se puede usar con Azure NetApp Files.

> [!IMPORTANT]
> Esta operación solo se aplica a **Azure Large Instance**.

## <a name="introduction"></a>Introducción

La plataforma Azure Large Instance también puede tener un sitio de recuperación ante desastres configurado donde se puedan replicar las instantáneas de volumen de almacenamiento.  Si las instantáneas se han configurado correctamente con este tipo de valores, se puede realizar la recuperación ante desastres en este sitio.  Este documento pretende ser una guía para la recuperación ante desastres con esta configuración.

## <a name="prerequisites-for-disaster-recovery-setup"></a>Requisitos previos para la configuración de la recuperación ante desastres

Antes de planear la conmutación por error de recuperación ante desastres, se deben cumplir los siguientes requisitos previos.

- Tener un nodo de recuperación ante desastres aprovisionado en el sitio de recuperación ante desastres. Hay dos opciones: uno es la recuperación ante desastres normal y otro es la recuperación ante desastres multipropósito.
- Tener la replicación del almacenamiento en funcionamiento. El equipo de Microsoft Operations realiza la configuración de la replicación de almacenamiento en el momento del aprovisionamiento automático para la recuperación ante desastres. Puede supervisar la replicación del almacenamiento mediante el comando `azacsnap -c details --details replication` en el sitio de recuperación ante desastres.
- Haber configurado las instantáneas de almacenamiento en la ubicación principal.
- Tener una instancia de HANA instalada en el sitio de recuperación ante desastres para la instancia principal con el mismo identificador de seguridad que la instancia principal.
- Leer y comprender el procedimiento de conmutación por error para la recuperación ante desastres descrito en [Alta disponibilidad y recuperación ante desastres de SAP HANA para instancias grandes en Azure](../virtual-machines/workloads/sap/hana-failover-procedure.md).
- Haber configurado las instantáneas de almacenamiento en la ubicación de recuperación ante desastres.
- Haber creado un archivo de configuración (por ejemplo, `DR.json`) con los volúmenes de almacenamiento para la recuperación ante desastres y la información asociada en el servidor de recuperación ante desastres.
- Haber completado los pasos que se indican en el sitio de recuperación ante desastres para:
  - Habilitar la comunicación con el almacenamiento.
  - Habilitar la comunicación con SAP HANA.

## <a name="set-up-disaster-recovery"></a>Configuración de la recuperación ante desastres

Microsoft admite la replicación a nivel de almacenamiento para la recuperación ante desastres. Existen dos formas de configurar la recuperación ante desastres:

Una es la recuperación ante desastres **normal** y otra es la recuperación ante desastres **multipropósito**. En la **normal** hay una instancia dedicada en la ubicación de recuperación ante desastres para la conmutación por error. En el escenario **multipropósito** hay otra instancia de HANA de control de calidad o de desarrollo que se ejecuta en la unidad de instancia grande de HANA en el sitio de recuperación ante desastres. Pero también se ha instalado una instancia de HANA preinstalada que está inactiva y tiene el mismo identificador de seguridad que la instancia de HANA que quiere conmutar por error a esa unidad de instancia grande de HANA. Microsoft Operations configura el entorno para que incluya la replicación de almacenamiento en función de la entrada proporcionada en el formulario de solicitud de servicio (SRF) en el momento de la incorporación.

> [!IMPORTANT]
> Asegúrese de que se cumplen todos los requisitos previos para la configuración de la recuperación ante desastres.

## <a name="monitor-data-replication-from-primary-to-dr-site"></a>Supervisión de la replicación de datos del sitio principal al sitio de recuperación ante desastres

El equipo de Microsoft Operations ya administra y supervisa el vínculo de recuperación ante desastres del sitio principal al de recuperación ante desastres.
Puede supervisar la replicación de los datos del servidor principal al servidor de recuperación ante desastres mediante el comando de instantánea `azacsnap -c details --details replication`.

## <a name="perform-a-failover-to-dr-site"></a>Conmutación por error al sitio de recuperación ante desastres

Ejecute el comando de conmutación por error en el sitio de recuperación ante desastres (`azacsnap -c restore --restore revertvolume`).

> [!IMPORTANT]
> El comando `azacsnap -c restore --restore revertvolume` interrumpe la replicación de almacenamiento del sitio de producción en el sitio de recuperación ante desastres. Debe ponerse en contacto con el equipo de Microsoft Operations para configurar la replicación de nuevo. Una vez rehabilitar la replicación, se inicializarán todos los datos del almacenamiento de recuperación ante desastres de ese identificador de seguridad. El comando que realiza la conmutación por error pone a disposición la instantánea de almacenamiento replicada más reciente. Si necesita restaurar a una instantánea anterior, abra una solicitud de soporte técnico para que el equipo de Operations le ayude a proporcionar una instantánea anterior restaurada en el sitio de recuperación ante desastres.

Estos son los pasos generales que deben seguirse para la conmutación por error de recuperación ante desastres:

- Cierre la instancia de HANA en el sitio **principal**. Esta acción solo es necesaria si realmente está realizando la conmutación por error al sitio de recuperación ante desastres, para evitar incoherencias en los datos.
- Cierre la instancia de HANA en el nodo de recuperación ante desastres para el identificador de seguridad de producción.
- Ejecute el comando `azacsnap -c restore --restore revertvolume` en el nodo de recuperación ante desastres con el identificador de seguridad que se va a recuperar.
  - El comando rompe el vínculo de replicación de almacenamiento del sitio principal al sitio de recuperación ante desastres.
  - El comando solo restaura los volúmenes /data y /logbackups, el volumen /shared NO se recupera, sino que se usa el /shared existente para identificador de seguridad de la ubicación de recuperación ante desastres.
  - Monte los volúmenes/data y /logbackups: asegúrese de agregarlos al archivo fstab.
- Restaure la instantánea de HANA SYSTEMDB. HANA Studio solo muestra la instantánea de HANA más reciente disponible en la instantánea de almacenamiento restaurada como parte de la ejecución del comando `azacsnap -c restore --restore revertvolume`.
- Recupere la base de datos del inquilino.
- Inicie la instancia de HANA en el sitio de recuperación ante desastres para el identificador de seguridad de producción (por ejemplo, H80 en este caso).
- Realice las pruebas correspondientes.

### <a name="example-performing-disaster-recovery"></a>Ejemplo de recuperación ante desastres

En esta subsección se describen los pasos en detalle para la conmutación por error al sitio de recuperación ante desastres.

#### <a name="step-1-get-the-volume-details-of-the-dr-node"></a>Paso 1: Obtención de los detalles del volumen del nodo de recuperación ante desastres

Ejecute el comando `df –h` para enumerar los sistemas de archivos y los volúmenes asociados a los que hacer referencia después de la conmutación por error.

```bash
df -h
```

```output
Filesystem Size Used Avail Use% Mounted on
devtmpfs 378G 8.0K 378G 1% /dev
tmpfs 569G 0 569G 0%
/dev/shm
tmpfs 378G 18M 378G 1% /run
tmpfs 378G 0 378G 0%
/sys/fs/cgroup
/dev/mapper/3600a098038304445622b4b584c575a66-part2 47G 20G 28G 42% /
/dev/mapper/3600a098038304445622b4b584c575a66-part1 979M 57M 856M 7% /boot
172.18.20.241:/hana_log_h80_mnt00003_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00001_t020_vol 512G 5.5G 507G 2% /hana/log/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00003_t020_vol 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00002_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00002_t020_vol 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00001_t020_vol 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_shared_h80_t020_vol/usr_sap_node1 2.7T 11G 2.7T 1% /usr/sap/H80
tmpfs 76G 0 76G 0% /run/user/0
172.18.20.241:/hana_shared_h80_t020_vol 2.7T 11G 2.7T 1% /hana/shared
172.18.20.241:/hana_data_h80_mnt00001_t020_xdp 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00002_t020_xdp 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00003_t020_xdp 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_backups_h80_t020_xdp 512G 15G 498G 3% /hana/logbackups/H80_T250
```

#### <a name="step-2-shut-down-hana-on-the-primary-site"></a>Paso 2: Cierre de HANA en el sitio principal

Si realiza una conmutación por error completa de cargas de trabajo de producción y es posible la conexión al sitio de producción principal, cierre las instancias de SAP HANA que se vayan a conmutar por error para la recuperación ante desastres.

Por ejemplo, si ha iniciado sesión como raíz, en el ejemplo siguiente se muestra el apagado de SAP HANA.  Reemplace <sid> por el identificador de seguridad de SAP HANA.

```bash
su - <sid>adm
HDB stop
```

#### <a name="step-3-shut-down-hana-on-dr-site"></a>Paso 3: Cierre de HANA en el sitio de recuperación ante desastres

Es importante apagar SAP HANA en el sitio de recuperación ante desastres antes de restaurar los volúmenes.

Por ejemplo, si ha iniciado sesión como raíz, en el ejemplo siguiente se muestra el apagado de SAP HANA.  Reemplace <sid> por el identificador de seguridad de SAP HANA.

```bash
su - <sid>adm
HDB stop
```

> [!IMPORTANT]
> Asegúrese de que las instancias de HANA en el sitio de recuperación ante desastres están sin conexión antes de restaurar los volúmenes.

#### <a name="step-4-restore-the-volumes"></a>Paso 4: Restauración de los volúmenes

```bash
azacsnap -c restore --restore revertvolume --dbsid H80
```

**_Salida del comando de conmutación por error para la recuperación ante desastres_**.

```bash
azacsnap --configfile DR.json -c restore --restore revertvolume --dbsid H80
```

```output
* This program is designed for those customers who have previously installed the
  Production HANA instance in the Disaster Recovery Location either as a
  stand-alone instance or as part of a multi-purpose environment.
* This program should be executed from the Disaster Recovery location otherwise
  unintended consequences may result.
* This program is intended to allow the customer to complete a Disaster Recovery
  failover.
* Any other restore points must be handled by Microsoft Operations.
* All volumes ('data' and 'other') are reverted to their most recent snapshot.
* The SnapMirror replication relationship between Prod and DR will be broken.

  CAUTION: a failback will be required after running this command and failback
   might not be a quick process and will require multiple steps in coordination
   with Microsoft Operations.

Do you wish to continue? (y/n) [n]: y
Checking state of HLI volumes for SID 'H80'
Configured volumes (Data and Other) are not quiesced for revert, will retry in 00:00:10 seconds
Volumes All Ok to Revert = True
Reverting volume 'hana_data_h80_mnt00001_t020_xdp' to snapshot 'H80_HANA_DATA_30MIN.2020-09-16_0330.0'
DR.json Data Volume #1 'hana_data_h80_mnt00001_t020_xdp' assigning to mountpoint 'mnt00001'
Reverting volume 'hana_log_backups_h80_t020_xdp01' to snapshot 'H80_HANA_LOGS_3MIN_X9.2020-09-16_0339.recent'
DR.json Other Volume #1 'hana_log_backups_h80_t020_xdp01' assigning to mountpoint '01'
HLI Volume revert completed for SID 'H80'
Displaying Mount Points by Volume as follows:
10.50.251.34:/hana_data_h80_mnt00001_t020_xdp  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
10.50.251.36:/hana_log_backups_h80_t020_xdp01  /hana/log_backups/H80/01 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
*********************  HANA DR Restore Steps  **********************************
* Please complete the following steps to recover your HANA database:           *
* 1. Ensure ALL the target mount points exist to mount the snapshot clones.    *
*    e.g. mkdir /hana/logbackups/H99_SOURCE                                    *
* 2. Add Mount Point Details from 'Displaying Mount Points by Volume' as       *
*    output above into /etc/fstab of DR Server.                                *
* 3. Mount newly added filesystems.                                            *
* 4. Perform HANA Snapshot Recovery using HANA Studio.                         *
********************************************************************************
```

> [!NOTE]
> Es necesario realizar los pasos que se indican al final en la consola para completar la preparación del almacenamiento para la conmutación por error para la recuperación ante desastres.

#### <a name="step-5-unmount-unnecessary-filesystems"></a>Paso 5: Desmontaje de los sistemas de archivos innecesarios

Ejecute el comando `umount` para desmontar los archivos de sistema o los volúmenes que no sean necesarios.

```bash
umount <Mount point>
```

Desmonte los puntos de montaje de copia de seguridad de datos y de registros. Puede tener varios puntos de montaje de datos en el escenario de escalado horizontal.

#### <a name="step-6-configure-the-mount-points"></a>Paso 6: Configuración de los puntos de montaje

Modifique el archivo `/etc/fstab` para marcar con comentario las entradas de las copias de seguridad de datos y de registros del identificador de seguridad principal (en este ejemplo, H80) y agregue las nuevas entradas de punto de montaje creadas a partir de los volúmenes de recuperación ante desastres del sitio principal. Las nuevas entradas de punto de montaje se proporcionan en la salida del comando.

- Comente los puntos de montaje existentes que se ejecutan en el sitio de recuperación ante desastres con el carácter `#`:

  ```output
  #172.18.20.241:/hana_data_h80_mnt00001_t020_vol /hana/data/H80/mnt00001 nfs     rw,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  #172.18.20.241:/hana_log_backups_h80_t020 /hana/logbackups/H80 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  ```

- Agregue las siguientes líneas a `/etc/fstab`.
  > Debe ser la misma salida del comando.

  ```output
  10.50.251.34:/hana_data_h80_mnt00001_t020_xdp  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  10.50.251.36:/hana_log_backups_h80_t020_xdp01  /hana/log_backups/H80/01 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  ```

#### <a name="step-7-mount-the-recovery-volumes"></a>Paso 7: Montaje de los volúmenes de recuperación

Ejecute el comando `mount –a` para montar todos los puntos de montaje.

```bash
mount -a
```

Ahora, si ejecuta `df –h` debería ver los volúmenes `*_dp` montados.

```bash
df -h
```

```output
Filesystem Size Used Avail Use% Mounted on
devtmpfs 378G 8.0K 378G 1% /dev
tmpfs 569G 0 569G 0% /dev/shm
tmpfs 378G 18M 378G 1% /run
tmpfs 378G 0 378G 0% /sys/fs/cgroup
/dev/mapper/3600a098038304445622b4b584c575a66-part2 47G 20G 28G 42% /
/dev/mapper/3600a098038304445622b4b584c575a66-part1 979M 57M 856M 7% /boot
172.18.20.241:/hana_log_h80_mnt00003_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00001_t020_vol 512G 5.5G 507G 2% /hana/log/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00003_t020_vol 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00002_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00002_t020_vol 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00001_t020_vol 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_shared_h80_t020_vol/usr_sap_node1 2.7T 11G 2.7T 1% /usr/sap/H80
tmpfs 76G 0 76G 0% /run/user/0
172.18.20.241:/hana_shared_h80_t020_vol 2.7T 11G 2.7T 1% /hana/shared
172.18.20.241:/hana_data_h80_mnt00001_t020_xdp 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00002_t020_xdp 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00003_t020_xdp 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_backups_h80_t020_xdp 512G 15G 498G 3% /hana/logbackups/H80_T250
```

#### <a name="step-8-recover-the-systemdb"></a>Paso 8: Recuperación de SYSTEMDB

En HANA Studio, haga clic con el botón derecho en la instancia de SYSTEMDB y elija "Backup and Recovery" (Copia de seguridad y recuperación) y "Recover System Database" (Recuperar base de datos del sistema).

Consulte la guía para recuperar una base de datos a partir de una instantánea, SYSTEMDB en particular.

#### <a name="step-9-recover-the-tenant-database"></a>Paso 9: Recuperación de la base de datos del inquilino

En HANA Studio, haga clic con el botón derecho en la instancia de SYSTEMDB y elija "Backup and Recovery" (Copia de seguridad y recuperación) y "Recover Tenant Database" (Recuperar base de datos del inquilino).

Consulte la guía para recuperar una base de datos a partir de una instantánea, las del INQUILINO en particular.

### <a name="run-azacsnap--c-backup-at-the-dr-site"></a>Ejecución de `azacsnap -c backup` en el sitio de recuperación ante desastres

Si está ejecutando copias de seguridad basadas en instantáneas en el sitio de recuperación ante desastres, el nombre del servidor de HANA configurado en el archivo de configuración de `azacsnap` en el sitio de recuperación ante desastres debe ser el mismo que el nombre del servidor de producción.

> [!IMPORTANT]
> La ejecución de `azacsnap -c backup` puede crear instantáneas de almacenamiento en el sitio de recuperación ante desastres que no se replican automáticamente en otro sitio.  Trabaje con Microsoft Operations para entender mejor la devolución de archivos o datos al sitio de producción original.

## <a name="next-steps"></a>Pasos siguientes

- [Obtención de los detalles de la instantánea](azacsnap-cmd-ref-details.md)
- [Realizar una copia de seguridad](azacsnap-cmd-ref-backup.md)
