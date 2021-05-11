---
title: Copia de seguridad y restauración del sistema operativo de SAP HANA en Azure (instancias grandes)| Microsoft Docs
description: Realización de la copia de seguridad y restauración del sistema operativo de SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c044d09626fe4c1d4409f98fc615b5a7d70c7cc8
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230107"
---
# <a name="os-backup-and-restore"></a>Copia de seguridad y restauración del sistema operativo

En este documento se describen los pasos necesarios para realizar la copia de seguridad y restauración de archivos del sistema operativo. El procedimiento varía según determinados parámetros, como tipo I o tipo II, revisión 3 o superior, ubicación, etc. Póngase en contacto con los responsables de operaciones de Microsoft para obtener valores precisos para estos parámetros de los recursos.

## <a name="os-backup-and-restore-for-type-ii-skus-of-revision-3-stamps"></a>Copia de seguridad y restauración del sistema operativo para SKU de tipo II de los sellos de revisión 3

En este documento se describen los pasos necesarios para realizar la copia de seguridad y restauración de archivos del sistema operativo de las **SKU de tipo II** de HANA (instancias grandes) de la revisión 3.

>[!Important]
> **Este artículo no se aplica a las implementaciones de SKU de tipo II en sellos de HANA (instancias grandes) de la revisión 4.** Se puede realizar una copia de seguridad de los LUN de arranque de las unidades de HANA (instancias grandes) de tipo II que se implementan en los sellos de HANA (instancias grandes) de la revisión 4 con instantáneas de almacenamiento, ya que este es el caso con las SKU de tipo I en sellos de la revisión 3.


>[!NOTE]
>En los scripts de copia de seguridad del SO se usa el software ReaR, que viene preinstalado en el servidor.  

Después de que el equipo de Microsoft `Service Management` haya realizado el aprovisionamiento, el servidor se configura de forma predeterminada con dos programaciones de copia de seguridad para realizar una copia de seguridad de nivel de archivo del sistema operativo. Puede comprobar las programaciones de los trabajos de copia de seguridad mediante el comando siguiente:
```
#crontab –l
```
En cualquier momento se puede cambiar la programación de la copia de seguridad mediante el siguiente comando:
```
#crontab -e
```
### <a name="how-to-take-a-manual-backup"></a>Creación de una copia de seguridad manual

La copia de seguridad del sistema de archivos del sistema operativo ya está programada mediante un **trabajo de cron**. Sin embargo, también puede realizar la copia de seguridad a nivel de archivo del sistema operativo de forma manual. Para realizar una copia de seguridad manual, ejecute el siguiente comando:

```
#rear -v mkbackup
```
La siguiente pantalla muestra la copia de seguridad manual de ejemplo:

![¿Cómo?](media/HowToHLI/OSBackupTypeIISKUs/HowtoTakeManualBackup.PNG)


### <a name="how-to-restore-a-backup"></a>Restauración de una copia de seguridad

Puede restaurar una copia de seguridad completa o archivos individuales de una copia de seguridad. Para ello, use el siguiente comando:

```
#tar  -xvf  <backup file>  [Optional <file to restore>]
```
Después de la restauración, el archivo se recupera en el directorio de trabajo actual.

El comando siguiente muestra la restauración de un archivo */etc/fstabfrom* del archivo de copia de seguridad *backup.tar.gz*.
```
#tar  -xvf  /osbackups/hostname/backup.tar.gz  etc/fstab 
```
>[!NOTE] 
>Deberá copiar el archivo en la ubicación deseada después de restaurarlo de la copia de seguridad.

En la captura de pantalla siguiente se muestra la restauración de una copia de seguridad completa:

![Captura de pantalla que muestra una ventana del símbolo del sistema con la restauración.](media/HowToHLI/OSBackupTypeIISKUs/HowtoRestoreaBackup.PNG)

### <a name="how-to-install-the-rear-tool-and-change-the-configuration"></a>Instalación de la herramienta ReaR y cambio de la configuración 

Los paquetes Relax-y Recover (ReaR) se encuentra **preinstalados** en las **SKU de tipo II** de las instancias grandes de HANA, y no es necesaria ninguna acción por parte del usuario. Puede comenzar a usar directamente ReaR para la copia de seguridad del sistema operativo.
Sin embargo, en aquellos casos en que necesite instalar los paquetes por su cuenta, puede seguir los pasos que se indican para instalar y configurar esta herramienta.

Para instalar los paquetes de copia de seguridad de **ReaR**, use los siguientes comandos:

Para el sistema operativo **SLES**, use el siguiente comando:
```
#zypper install <rear rpm package>
```
Para el sistema operativo **RHEL**, use el siguiente comando: 
```
#yum install rear -y
```
Para configurar la herramienta ReaR, debe actualizar los parámetros **OUTPUT_URL** y **BACKUP_URL** del *archivo /etc/rear/local.conf*.
```
OUTPUT=ISO
ISO_MKISOFS_BIN=/usr/bin/ebiso
BACKUP=NETFS
OUTPUT_URL="nfs://nfsip/nfspath/"
BACKUP_URL="nfs://nfsip/nfspath/"
BACKUP_OPTIONS="nfsvers=4,nolock"
NETFS_KEEP_OLD_BACKUP_COPY=
EXCLUDE_VG=( vgHANA-data-HC2 vgHANA-data-HC3 vgHANA-log-HC2 vgHANA-log-HC3 vgHANA-shared-HC2 vgHANA-shared-HC3 )
BACKUP_PROG_EXCLUDE=("${BACKUP_PROG_EXCLUDE[@]}" '/media' '/var/tmp/*' '/var/crash' '/hana' '/usr/sap'  ‘/proc’)
```

En la captura de pantalla siguiente se muestra la restauración de una copia de seguridad completa: ![Captura de pantalla que muestra una ventana del símbolo del sistema con la restauración mediante la herramienta ReaR.](media/HowToHLI/OSBackupTypeIISKUs/RearToolConfiguration.PNG)


## <a name="os-backup-and-restore-for-all-other-skus"></a>Copia de seguridad y restauración del sistema operativo de todas las demás SKU

La siguiente información describe los pasos necesarios para realizar la copia de seguridad y restauración de archivos del sistema operativo de todas las **SKU de tipo II** de HANA (instancias grandes) de la revisión 3.

### <a name="how-to-take-a-manual-backup"></a>Creación de una copia de seguridad manual

Obtenga las herramientas de instantáneas de Microsoft más recientes para SAP HANA en [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md) y configúrelas para que se ejecuten con regularidad mediante `crontab` con la marca `--type=boot`. De esta forma se garantizan copias de seguridad normales del sistema operativo. En el ejemplo siguiente se muestra una programación cron en `/etc/crontab` para la copia de seguridad del sistema operativo de una SKU de tipo I:

```
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeI --prefix=dailyboot --frequncy=15min --retention=28
```

En el ejemplo siguiente se muestra una programación cron en `/etc/crontab` para una copia de seguridad del sistema operativo de SKU de tipo II:

```
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeII --prefix=dailyboot --frequency=15min --retention=28
```

Referencias adicionales
- [Configuración de instantáneas de almacenamiento](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-backup-restore#set-up-storage-snapshots)
- Guía de herramientas de instantáneas de Microsoft para SAP HANA en [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md).

### <a name="how-to-restore-a-backup"></a>Restauración de una copia de seguridad

La operación de restauración no se puede realizar desde el propio sistema operativo. Para ello, envíe una incidencia de soporte técnico a los responsables de operaciones de Microsoft. La operación de restauración requiere que la instancia de HLI esté en estado apagado, por lo que debe programarla en consecuencia.

### <a name="managed-os-snapshots"></a>Instantáneas del sistema operativo administradas

Azure puede realizar automáticamente copias de seguridad del sistema operativo para los recursos de HLI. Estas copias de seguridad se realizan una vez al día y Azure conserva hasta las tres copias de seguridad más recientes. Esta opción está habilitada de forma predeterminada para todos los clientes de las siguientes regiones:
- Oeste de EE. UU.
- Este de Australia
- Sudeste de Australia
- Centro-sur de EE. UU.
- Este de EE. UU. 2

Esta instalación está parcialmente disponible en las siguientes regiones:
- Este de EE. UU.
- Norte de Europa
- Oeste de Europa

No se puede modificar la frecuencia o el período de retención de las copias de seguridad realizadas por esta instalación. En caso de que se necesite una estrategia de copia de seguridad del sistema operativo diferente para los recursos de HLI, puede optar por no participar en esta instalación; para ello, presente una incidencia de soporte técnico a los responsables de operaciones de Microsoft y, luego, configure las herramientas de instantáneas de Microsoft para SAP HANA para realizar copias de seguridad del sistema operativo con las instrucciones proporcionadas en la sección anterior de este documento.