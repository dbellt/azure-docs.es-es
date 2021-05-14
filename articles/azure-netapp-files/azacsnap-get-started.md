---
title: Primeros pasos con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una guía de instalación de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.openlocfilehash: eee7204413face46870de049b38a79061de28347
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929802"
---
# <a name="get-started-with-azure-application-consistent-snapshot-tool"></a>Primeros pasos con la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía de instalación de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="getting-the-snapshot-tools"></a>Obtención de las herramientas de instantáneas

Se recomienda que los clientes obtengan la versión más reciente del [instalador de AzAcSnap](https://aka.ms/azacsnapdownload) de Microsoft.

El archivo de instalación automática tiene asociado un [archivo de firma del instalador de AzAcSnap](https://aka.ms/azacsnapdownloadsignature) que está firmado con la clave pública de Microsoft para permitir la comprobación de GPG del instalador descargado.

Una vez completadas estas descargas, siga los pasos de esta guía para la instalación.

### <a name="verifying-the-download"></a>Comprobación de la descarga

El instalador, que puede descargarse desde la sección anterior, tiene un archivo de firma PGP asociado con una extensión de nombre de archivo `.asc`. Este archivo permite asegurarse de que el instalador descargado es un archivo proporcionado por Microsoft que se ha comprobado. La clave pública PGP de Microsoft que se usa para firmar paquetes de Linux está disponible aquí (<https://packages.microsoft.com/keys/microsoft.asc>) y se ha usado para firmar el archivo de firma.

La clave pública PGP de Microsoft se puede importar en el entorno local de un usuario de la siguiente manera:

```bash
wget https://packages.microsoft.com/keys/microsoft.asc
gpg --import microsoft.asc
```

Los comandos siguientes confían en la clave pública PGP de Microsoft:

1. Enumere las claves del almacén.
2. Edite la clave de Microsoft.
3. Compruebe la huella digital con `fpr`.
4. Firme la clave para confiar en ella.

```bash
gpg --list-keys
```

Claves enumeradas:
```output
----<snip>----
pub rsa2048 2015- 10 - 28 [SC]
BC528686B50D79E339D3721CEB3E94ADBE1229CF
uid [ unknown] Microsoft (Release signing) gpgsecurity@microsoft.com
```

```bash
gpg --edit-key gpgsecurity@microsoft.com
```

Salida de la sesión interactiva `gpg` que firma la clave pública de Microsoft:
```output
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
pub rsa2048/EB3E94ADBE1229CF
created: 2015- 10 - 28 expires: never usage: SC
trust: unknown validity: unknown
[ unknown] (1). Microsoft (Release signing) <gpgsecurity@microsoft.com>

gpg> fpr
pub rsa2048/EB3E94ADBE1229CF 2015- 10 - 28 Microsoft (Release signing)
<gpgsecurity@microsoft.com>
Primary key fingerprint: BC52 8686 B50D 79E3 39D3 721C EB3E 94AD BE12 29CF

gpg> sign
pub rsa2048/EB3E94ADBE1229CF
created: 2015- 10 - 28 expires: never usage: SC
trust: unknown validity: unknown
Primary key fingerprint: BC52 8686 B50D 79E3 39D3 721C EB3E 94AD BE12 29CF
Microsoft (Release signing) <gpgsecurity@microsoft.com>
Are you sure that you want to sign this key with your
key "XXX XXXX <xxxxxxx@xxxxxxxx.xxx>" (A1A1A1A1A1A1)
Really sign? (y/N) y

gpg> quit
Save changes? (y/N) y
```

El archivo de firma PGP para el instalador se puede comprobar de la siguiente manera:

```bash
gpg --verify azacsnap_installer_v5.0.run.asc azazsnap_installer_v5.0.run
```

```output
gpg: Signature made Sat 13 Apr 2019 07:51:46 AM STD
gpg: using RSA key EB3E94ADBE1229CF
gpg: Good signature from "Microsoft (Release signing)
<gpgsecurity@microsoft.com>" [full]
```

Para más información sobre el uso de GPG, consulte el [manual de privacidad de GNU](https://www.gnupg.org/gph/en/manual/book1.html).

## <a name="supported-scenarios"></a>Escenarios admitidos

Las herramientas de instantánea se pueden usar en los siguientes escenarios.

- Identificador de seguridad único
- Varios identificadores de seguridad
- HSR
- Escalado horizontal
- MDC (solo se admite un único inquilino)
- Contenedor único
- Sistema operativo SUSE
- Sistema operativo RHEL
- SKU de tipo I
- SKU de tipo II

Consulte [Escenarios admitidos para instancias grandes de HANA](../virtual-machines/workloads/sap/hana-supported-scenario.md).

## <a name="snapshot-support-matrix-from-sap"></a>Matriz de compatibilidad con instantáneas de SAP

La siguiente matriz se proporciona como guía sobre qué versiones de SAP HANA son compatibles con SAP para copias de seguridad de instantáneas de almacenamiento.

| Versiones de base de datos       |1.0 SPS12|2.0 SPS0|2.0 SPS1|2.0 SPS2|2.0 SPS3|2.0 SPS4|
|-------------------------|---------|--------|--------|--------|--------|--------|
|Base de datos de contenedor único| √       | √      | -      | -      | -      | -      |
|Un solo inquilino de MDC        | -       | -      | √      | √      | √      | √      |
|Varios inquilinos de MDC     | -       | -      | -      | -      | -      | √      |
> √ = <small>compatible con SAP para instantáneas de almacenamiento</small>

## <a name="important-things-to-remember"></a>Cosas que recordar

- Después de la instalación de las herramientas de instantáneas, supervise continuamente el espacio de almacenamiento disponible y, si es necesario, elimine las instantáneas antiguas periódicamente para evitar que se agote el almacenamiento.
- Utilice siempre las herramientas de instantáneas más recientes.
- Utilice la misma versión de las herramientas de instantáneas en todo el entorno.
- Pruebe las herramientas de instantáneas y familiarícese con los parámetros necesarios y con la salida del comando antes de usarlo en el sistema de producción.
- Al configurar el usuario de HANA para copia de seguridad (más adelante se ofrece información detallada), debe configurar el usuario para cada instancia de HANA. Cree una cuenta de usuario de SAP HANA para acceder a la instancia de HANA en SYSTEMDB (y no en la base de datos del identificador de seguridad) para MDC. En el entorno de contenedor único, se puede configurar en la base de datos de inquilinos.
- Los clientes deben proporcionar la clave pública SSH para el acceso al almacenamiento. Esta acción debe realizarse una vez por nodo y para cada usuario en el que se ejecute el comando.
- El número de instantáneas por volumen está limitado a 250.
- Si edita manualmente el archivo de configuración, use siempre un editor de texto de Linux, como "vi", y no los editores de Windows, como el Bloc de notas. Si usa un editor de Windows, podría dañarse el formato del archivo.
  - Configure `hdbuserstore` para que el usuario de SAP HANA se comunique con SAP HANA.
- Para la recuperación ante desastres: Las herramientas de instantáneas deben probarse en el nodo de recuperación ante desastres antes de configurar la recuperación ante desastres.
- Supervise periódicamente el espacio en disco. La eliminación automatizada del registro se administra con la opción `--trim` de `azacsnap -c backup` de SAP HANA 2 y versiones posteriores.
- **Riesgo de que no se realicen las instantáneas**: las herramientas de instantáneas solo interactúan con el nodo del sistema SAP HANA especificado en el archivo de configuración.  Si este nodo deja de estar disponible, no existe ningún mecanismo para iniciar automáticamente la comunicación con otro nodo.  
  - En un escenario de **escalabilidad horizontal de SAP HANA con espera**, es habitual instalar y configurar las herramientas de instantáneas en el nodo maestro. Sin embargo, si el nodo maestro deja de estar disponible, el nodo en espera tomará el rol de nodo maestro. En este caso, el equipo de implementación debe configurar las herramientas de instantáneas en ambos nodos (maestro y en espera) para evitar que se omitan instantáneas. En el estado normal, el nodo maestro tomará las instantáneas de HANA iniciadas por crontab, pero después de la conmutación por error del nodo maestro, esas instantáneas tendrán que ejecutarse desde otro nodo, como el nuevo nodo maestro (anteriormente en espera). Para lograr este resultado, el nodo en espera necesitaría tener la herramienta de instantáneas instalada, la comunicación de almacenamiento habilitada, hdbuserstore configurado, `azacsnap.json` configurado y los comandos de crontab preconfigurados antes de la conmutación por error.
  - Para un escenario de **alta disponibilidad de HSR de SAP HANA**, se recomienda instalar, configurar y programar las herramientas de instantáneas en ambos nodos (principal y secundario). A continuación, si el nodo principal deja de estar disponible, el nodo secundario tomará el control y las instantáneas se realizarán en él. En el estado normal, el nodo principal tomará las instantáneas de HANA iniciadas por crontab y el nodo secundario intentará realizar instantáneas, pero no lo conseguirá porque el principal funciona correctamente.  Sin embargo, después de la conmutación por error del nodo principal, esas instantáneas se ejecutarán desde el nodo secundario. Para lograr este resultado, el nodo secundario necesita tener la herramienta de instantáneas instalada, la comunicación de almacenamiento habilitada, `hdbuserstore` configurado, azacsnap.json configurado y los comandos de crontab habilitados antes de la conmutación por error.

## <a name="guidance-provided-in-this-document"></a>Guía proporcionada en este documento

Se proporciona la siguiente guía para ilustrar el uso de las herramientas de instantáneas.

### <a name="taking-snapshot-backups"></a>Realización de copias de seguridad de instantáneas

- [Cuáles son los requisitos previos para las instantáneas de almacenamiento](azacsnap-installation.md#prerequisites-for-installation)
  - [Habilitación de la comunicación con el almacenamiento](azacsnap-installation.md#enable-communication-with-storage)
  - [Habilitación de la comunicación con SAP HANA](azacsnap-installation.md#enable-communication-with-sap-hana)
- [Realización manual de instantáneas](azacsnap-tips.md#take-snapshots-manually)
- [Configuración de copias de seguridad de instantáneas automáticas](azacsnap-tips.md#setup-automatic-snapshot-backup)
- [Supervisión de las instantáneas](azacsnap-tips.md#monitor-the-snapshots)
- [Eliminación de una instantánea](azacsnap-tips.md#delete-a-snapshot)
- [Restauración de una instantánea](azacsnap-tips.md#restore-a-snapshot)
- [Restauración de una instantánea `boot`](azacsnap-tips.md#restore-a-boot-snapshot)
- [Hechos clave que se deben conocer sobre las instantáneas](azacsnap-tips.md#key-facts-to-know-about-snapshots)

> Las instantáneas se prueban para un solo SID y varios SID.

### <a name="performing-disaster-recovery"></a>Realización de la recuperación ante desastres

- [Cuáles son los requisitos previos para la configuración de la recuperación ante desastres](azacsnap-disaster-recovery.md#prerequisites-for-disaster-recovery-setup)
- [Configuración de la recuperación ante desastres](azacsnap-disaster-recovery.md#set-up-disaster-recovery)
- [Supervisión de la replicación de datos del sitio principal al sitio de recuperación ante desastres](azacsnap-disaster-recovery.md#monitor-data-replication-from-primary-to-dr-site)
- [Conmutación por error al sitio de recuperación ante desastres](azacsnap-disaster-recovery.md#perform-a-failover-to-dr-site)

## <a name="next-steps"></a>Pasos siguientes

- [Instalación de la herramienta Azure Application Consistent Snapshot](azacsnap-installation.md)
