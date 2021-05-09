---
title: Restauración con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una guía para ejecutar el comando de restauración de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.topic: reference
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: df292fc59d46d59c94f43dae0b1827c068b3e057
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929874"
---
# <a name="restore-using-azure-application-consistent-snapshot-tool"></a>Restauración con la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía para ejecutar el comando de restauración de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="introduction"></a>Introducción

La restauración de un volumen a partir de una instantánea se realiza mediante el comando `azacsnap -c restore`.

> [!IMPORTANT]
> Esto no realiza una recuperación de base de datos, solo una restauración de los volúmenes tal y como se describe en cada una de las opciones siguientes.

## <a name="command-options"></a>Opciones de comando

El comando `-c restore` tiene las siguientes opciones:

- `--restore snaptovol` Crea un volumen basado en la última instantánea del volumen de destino. Este comando crea un nuevo volumen "clonado" basado en el volumen de destino configurado mediante la última instantánea del volumen como base para crear el nuevo volumen.  Este comando no interrumpe la replicación de almacenamiento del sitio principal al secundario. En su lugar, se crean clones de la instantánea más reciente disponible en el sitio de recuperación ante desastres y se muestran puntos de montaje del sistema de archivos recomendados de los volúmenes clonados. Este comando debe ejecutarse en el sistema de Azure (instancias grandes) **de la región de recuperación ante desastres** (es decir, el sistema de conmutación por error de destino).

- `--restore revertvolume` Revierte el volumen de destino a un estado anterior en función de la instantánea más reciente  mediante este comando como parte de la conmutación por error de recuperación ante desastres en la región de recuperación ante desastres emparejada. Este comando **detiene** la replicación de almacenamiento desde el sitio principal al sitio secundario y revierte los volúmenes de recuperación ante desastres de destino a su instantánea más reciente disponible en los volúmenes de recuperación ante desastres junto con los puntos de montaje del sistema de archivos recomendados para los volúmenes de recuperación ante desastres revertidos. Este comando debe ejecutarse en el sistema de Azure (instancias grandes) **de la región de recuperación ante desastres** (es decir, el sistema de conmutación por error de destino).
    > [!NOTE]
    > El subcomando (`--restore revertvolume`) solo está disponible para Azure (instancias grandes) y no está disponible para Azure NetApp Files.
- `--dbsid <SAP HANA SID>` es el SID de SAP HANA que se selecciona del archivo de configuración al que se van a aplicar los comandos de restauración de volumen.

- `[--configfile <config filename>]` es un parámetro opcional que permite nombres de archivo de configuración personalizados.

## <a name="perform-a-test-dr-failover-azacsnap--c-restore---restore-snaptovol"></a>Realización de una conmutación por error de recuperación ante desastres de prueba `azacsnap -c restore --restore snaptovol`

Este comando es como el comando de conmutación por error de DR "completo" (`--restore restorevolume`), pero en lugar de interrumpir la replicación entre el sitio principal y el sitio de recuperación ante desastres, se crea un volumen clonado fuera de los volúmenes de recuperación ante desastres, lo que permite la restauración de la instantánea más reciente en el sitio de recuperación ante desastres. El cliente puede usar esos volúmenes clonados para probar la recuperación ante desastres sin tener que ejecutar una conmutación por error completa de su entorno de HANA que interrumpa el acuerdo de replicación entre el sitio principal y el sitio de recuperación ante desastres.

- Se pueden probar varios puntos de restauración diferentes de esta manera, cada uno con su propio punto de restauración.
- El clon se designa con la marca de tiempo del momento en que se ejecutó el comando y representa los datos más recientes y otras instantáneas disponibles al ejecutarse.

> [!IMPORTANT]
> Esta operación solo se aplica a Azure (instancias grandes).
>
> - Cuando se ejecuta este comando, se necesita un correo electrónico con el que pueda contactar Microsoft Operations antes de la eliminación de los clones después de 4 semanas.
> - Cada ejecución de este comando creará un nuevo clon que Microsoft Operations debe eliminar cuando se complete la prueba.
> - Los volúmenes clonados creados se eliminarán automáticamente después de 4 semanas.

El archivo de configuración (por ejemplo, `DR.json`) debe contener solo los volúmenes de recuperación ante desastres y no los volúmenes de producción; de lo contrario, podrían crearse clones de los volúmenes de producción.

### <a name="output-of-the-azacsnap--c-restore---restore-snaptovol-command-for-single-node-scenario"></a>Salida del comando `azacsnap -c restore --restore snaptovol` (para un escenario de un solo nodo)

```output
> azacsnap --configfile DR.json -c restore --restore snaptovol --dbsid H80
* This program is designed for those customers who have previously installed the
  Production HANA instance in the Disaster Recovery Location either as a
  stand-alone instance or as part of a multi-purpose environment.
* This program should be executed from the Disaster Recovery location otherwise
  unintended consequences may result.
* This program is intended to allow the customer to simulate a Disaster Recovery
  failover without actually requiring a failover and subsequent failback.
* Any other restore points must be handled by Microsoft Operations.
* As part of the process, a clone is created of the each of the 'data' and 'other'
  volumes per the configuration file.

Do you wish to continue? (y/n) [n]: y

About to create clones of volumes based on the latest snapshot, these will be
kept for 4 weeks before being automatically deleted by Microsoft Operations.
Enter an email address to contact when deleting clones: <b>person@nowhere.com</b>
Checking state of HLI volumes for SID 'PEW'
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Displaying Mount Points by Volume as follows:
10.50.251.34:/hana_data_h80_sapprdhdb80_mnt00001_t020_xdp_rwclone_20200916_0256  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
10.50.251.36:/hana_log_backups_h80_sapprdhdb80_t020_xdp_rwclone_20200916_0256  /hana/log_backups/H80/01 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
*******************  HANA Test DR Restore Steps  ******************************
* Complete the following steps to recover your HANA database:           *
* 1. Ensure ALL the target mount points exist to mount the snapshot clones.    *
*    e.g. mkdir /hana/logbackups/H99_SOURCE                                    *
* 2. Add Mount Point Details from 'Displaying Mount Points by Volume' as       *
*    output above into /etc/fstab of DR Server.                                *
* 3. Mount newly added filesystems.                                            *
* 4. Perform HANA Snapshot Recovery using HANA Studio.                         *
********************************************************************************
*  These snapshot copies (clones) are kept for 4 weeks before                  *
*  being automatically removed.                                                *
*  Please contact Microsoft Operations to delete them earlier.                 *
********************************************************************************
```

> [!IMPORTANT]
> La salida "Displaying Mount Points by Volume" (Mostrando puntos de montaje por volumen) es diferente para los distintos escenarios.

## <a name="perform-full-dr-failover-azacsnap--c-restore---restore-revertvolume"></a>Realización de una conmutación por error de recuperación ante desastres completa `azacsnap -c restore --restore revertvolume`

Este comando **detiene** la replicación del almacenamiento del sitio principal al secundario, restaura la instantánea más reciente en los volúmenes de recuperación ante desastres y proporciona los puntos de montaje de los volúmenes de recuperación ante desastres.

Este comando DEBE ejecutarse en el servidor de recuperación ante desastres mediante un archivo de configuración (por ejemplo, `DR.json`) SOLO con volúmenes de recuperación ante desastres.

Realice una conmutación por error al sitio de recuperación ante desastres mediante la ejecución del comando `azacsnap -c restore --restore revertvolume`.  Este comando requiere que se agregue un SID como parámetro. Este es el identificador de seguridad de la instancia de HANA, que debe recuperarse en el sitio de recuperación ante desastres.

> [!IMPORTANT]
> Ejecute este comando solo si planea realizar el ejercicio de recuperación ante desastres o una prueba. Este comando interrumpe la replicación. Debe ponerse en contacto con Microsoft Operations para volver a habilitar la replicación.

En el nivel superior, estos son los pasos para ejecutar una conmutación por error de recuperación ante desastres:

- Cierre la instancia de HANA en el sitio **principal**. Esta acción solo es necesaria si realmente está realizando la conmutación por error al sitio de recuperación ante desastres para evitar incoherencias en los datos.
- Cierre la instancia de HANA en el nodo de recuperación ante desastres para el identificador de seguridad de producción.
- Ejecute el comando `azacsnap -c restore --restore revertvolume` en el nodo de recuperación ante desastres con el identificador de seguridad que se va a recuperar.
  - El comando rompe el vínculo de replicación de almacenamiento del sitio principal al sitio de recuperación ante desastres.
  - El comando restaura los volúmenes "data" y "other" tal y como están configurados.  Normalmente, esta operación sería para los volúmenes de los sistemas de archivos `/hana/data` y `/hana/logbackups`.  NO se recupera el sistema de archivos `/hana/shared`, sino que usa el sistema de archivos `/hana/shared` existente para el identificador de seguridad en la ubicación de recuperación ante desastres.
  - Monte los volúmenes `/hana/data` y `/hana/logbackups`: asegúrese de que se agregan al archivo `/etc/fstab`.
- Restaure la instantánea de HANA SYSTEMDB. HANA Studio solo muestra la instantánea de HANA más reciente disponible en la instantánea de almacenamiento restaurada como parte de la ejecución del comando de la instantánea `azacsnap -c restore --restore revertvolume`.
- Recupere la base de datos del inquilino.
- Inicie la instancia de HANA en el sitio de recuperación ante desastres para el identificador de seguridad de producción (por ejemplo, H80 en este caso).
- Realice cualquier prueba de base de datos.

## <a name="next-steps"></a>Pasos siguientes

- [Realizar una copia de seguridad](azacsnap-cmd-ref-backup.md)
- [Obtención de los detalles de la instantánea](azacsnap-cmd-ref-details.md)
