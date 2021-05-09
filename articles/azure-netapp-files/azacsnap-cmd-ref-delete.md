---
title: Eliminación con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una guía para ejecutar el comando de eliminación de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.openlocfilehash: d02f0e8ff3d8d6da37e0e457cea2e4878ff1a2b0
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930018"
---
# <a name="delete-using-azure-application-consistent-snapshot-tool"></a>Eliminación con la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía para ejecutar el comando de eliminación de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="introduction"></a>Introducción

Es posible eliminar instantáneas de volumen y entradas del catálogo de bases de datos con el comando `azacsnap -c delete`.

> [!IMPORTANT]
> Las instantáneas creadas menos de 10 minutos antes de ejecutar este comando no deben eliminarse debido a la posibilidad de que se produzcan interferencias con la replicación de instantáneas.

## <a name="command-options"></a>Opciones de comando

El comando `-c delete` tiene las siguientes opciones:

- `--delete hana` al usarse con las opciones `--dbsid <SID>` y `--hanabackupid <HANA backup id>` eliminará las entradas del catálogo de copias de seguridad de SAP HANA que coincidan con los criterios.

- `--delete storage` al usarse con la opción `--snapshot <snapshot name>` eliminará la instantánea del sistema de almacenamiento de back-end.

- `--delete sync` al usarse con las opciones `--dbsid <SID>` y `--hanabackupid <HANA backup id>` obtiene el nombre de la instantánea de almacenamiento del catálogo de copias de seguridad para `<HANA backup id>` y, a continuación, elimina la entrada del catálogo de copias de seguridad _y_ la instantánea de cualquiera de los volúmenes que contienen la instantánea con nombre.

- `--delete sync` al usarse con `--snapshot <snapshot name>` comprobará si hay entradas en el catálogo de copias de seguridad para `<snapshot name>`, obtiene el identificador de copia de seguridad de SAP HANA y elimina la entrada del catálogo de copias de seguridad _y_ la instantánea de cualquiera de los volúmenes que contienen la instantánea con nombre.

- `[--force]` (opcional) *Úselo con precaución*.  Esta operación forzará la eliminación sin pedir confirmación.

- `[--configfile <config filename>]` es un parámetro opcional que permite nombres de archivo de configuración personalizados.

### <a name="delete-a-snapshot-using-sync-option"></a>Eliminación de una instantánea mediante la opción `sync`

```bash
azacsnap -c delete --delete sync --dbsid H80 --hanabackupid 157979797979
```

> [!NOTE]
> Comprueba si hay entradas en el catálogo de copias de seguridad para el identificador de copia de seguridad de SAP HANA 157979797979, obtiene el nombre de la instantánea de almacenamiento y elimina la entrada del catálogo de copias de seguridad y la instantánea de cualquiera de los volúmenes que contienen la instantánea con nombre.

```bash
azacsnap -c delete --delete sync --snapshot hana_hourly.2020-01-22_2358
```

> [!NOTE]
> Comprueba si hay entradas en el catálogo de copias de seguridad para la instantánea denominada hana_hourly.2020-01-22_2358, obtiene el identificador de copia de seguridad de SAP HANA y elimina la entrada del catálogo de copias de seguridad y la instantánea de cualquiera de los volúmenes que contienen la instantánea con nombre.

### <a name="delete-a-snapshot-using-hana-option"></a>Eliminación de una instantánea mediante la opción `hana`

```bash
azacsnap -c delete --delete hana --dbsid H80 --hanabackupid 157979797979
```

> [!NOTE]
> Elimina el identificador de copia de seguridad de SAP HANA 157979797979 del catálogo de copias de seguridad para el identificador de seguridad H80.

### <a name="delete-a-snapshot-using-storage-option"></a>Eliminación de una instantánea mediante la opción `storage`

```bash
azacsnap -c delete --delete storage --snapshot hana_hourly.2020-01-22_2358
```

> [!NOTE]
> Elimina la instantánea de todos los volúmenes que contengan una instantánea con el nombre hana_hourly.2020-01-22_2358.

**Salida mediante la opción `--delete storage`**

Se le pide al usuario que confirme la eliminación.

```bash
azacsnap -c delete --delete storage --snapshot azacsnap-hsr-ha.2020-07-02T221702.2535255Z
```

```output
Processing delete request for snapshot 'azacsnap-hsr-ha.2020-07-02T221702.2535255Z'.
Are you sure you want to permanently delete the snapshot 'azacsnap-hsr-ha.2020-07-02T221702.2535255Z' from all storage volumes.? (y/n) [n]: y
Snapshot deletion completed
```

Es posible evitar la confirmación del usuario mediante el parámetro opcional `--force`, como se indica a continuación:

```bash
azacsnap -c delete --delete storage --snapshot azacsnap-hsr-ha.2020-07-02T222201.4988618Z --force
```

```output
Processing delete request for snapshot 'azacsnap-hsr-ha.2020-07-02T222201.4988618Z'.
Snapshot deletion completed
```

## <a name="next-steps"></a>Pasos siguientes

- [Obtención de los detalles de la instantánea](azacsnap-cmd-ref-details.md)
- [Realizar una copia de seguridad](azacsnap-cmd-ref-backup.md)
