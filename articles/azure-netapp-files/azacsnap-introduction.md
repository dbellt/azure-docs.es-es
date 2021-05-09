---
title: Qué es la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una introducción a la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.topic: conceptual
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: 1cd22ad59da4961644c4e4d05150ba8565bf47a0
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929766"
---
# <a name="what-is-azure-application-consistent-snapshot-tool"></a>¿Qué es la herramienta Azure Application Consistent Snapshot?

Azure Application Consistent Snapshot (AzAcSnap) es una herramienta de línea de comandos que permite proteger los datos de bases de datos de terceros mediante el control de la orquestación necesaria para ponerlos en un estado coherente con la aplicación antes de tomar una instantánea del almacenamiento. Después de esto se devuelven a un estado operativo.

## <a name="supported-platforms-and-os"></a>Sistemas operativos y plataformas compatibles

- **Bases de datos**
  - SAP HANA (consulte la [matriz de compatibilidad](azacsnap-get-started.md#snapshot-support-matrix-from-sap) para obtener más información)

- **Sistemas operativos**
  - SUSE Linux Enterprise Server 12+
  - Red Hat Enterprise Linux 7+

## <a name="benefits-of-using-azacsnap"></a>Ventajas del uso de AzAcSnap

AzAcSnap aprovecha las funcionalidades de replicación e instantánea de volumen de Azure NetApp Files y Azure Large Instance.  Proporciona las siguientes ventajas:

- **Protección de datos coherente con la aplicación**: AzAcSnap es una solución centralizada para la copia de seguridad de archivos de base de datos críticos. Garantiza la coherencia de la base de datos antes de realizar una instantánea de volumen de almacenamiento. Como resultado, garantiza que la instantánea del volumen de almacenamiento se pueda usar para la recuperación de la base de datos.
- **Administración del catálogo de bases de datos**: al usar AzAcSnap con una base de datos que tiene un catálogo de copia de seguridad integrado, los registros del catálogo se mantienen actualizados con las instantáneas de almacenamiento.  Esta funcionalidad permite a los administradores de las bases de datos ver la actividad de copia de seguridad.
- **Protección de volumen ad hoc**: esta funcionalidad es útil para los volúmenes que no son de bases de datos y que no necesitan inactividad de la aplicación para realizar instantáneas de almacenamiento.  Entre los ejemplos se incluyen los volúmenes de copia de seguridad de registros de SAP HANA o los volúmenes SAPTRANS.
- **Clonación de volúmenes de almacenamiento**: esta funcionalidad proporciona clones de los volúmenes de almacenamiento con un espacio eficaz para fines de desarrollo y pruebas.
- **Compatibilidad con la recuperación ante desastres**: AzAcSnap aprovecha la replicación del volumen de almacenamiento para proporcionar opciones para recuperar instantáneas coherentes con la aplicación replicada en un sitio remoto.

AzAcSnap es un binario sencillo.  No necesita agentes o complementos adicionales para interactuar con la base de datos o el almacenamiento (Azure NetApp Files mediante Azure Resource Manager y Azure Large Instance mediante SSH).  AzAcSnap debe estar instalado en un sistema con conectividad con la base de datos y el almacenamiento.  Sin embargo, la flexibilidad de instalación y configuración permite una sola instalación centralizada o una instalación totalmente distribuida con copias instaladas en cada instalación de base de datos.

## <a name="architecture-overview"></a>Información general sobre la arquitectura

AzAcSnap puede instalarse en el mismo host que la base de datos (SAP HANA) o en un sistema centralizado.  Sin embargo, debe haber conectividad de red con los servidores de bases de datos y el back-end de almacenamiento (Azure Resource Manager para Azure NetApp Files o SSH para Azure Large Instance).

AzAcSnap es una aplicación ligera que se ejecuta normalmente desde un programador externo.  En la mayoría de los sistemas Linux, esta operación es `cron`, que es en lo que se centrará la documentación.  Pero el programador podría ser una herramienta alternativa, siempre que pueda importar el perfil de shell del usuario `azacsnap`.  La importación de la configuración del entorno del usuario garantiza que las rutas de acceso y los permisos de archivo se inicializan correctamente.

## <a name="command-synopsis"></a>Sinopsis de los comandos

El formato general de los comandos es el siguiente: `azacsnap -c [command] --[command] [sub-command] --[flag-name] [flag-value]`.

## <a name="command-options"></a>Opciones de comando

Las opciones de comando son las siguientes, con los comandos en las viñetas principales y los subcomandos asociados en las viñetas con sangría:

- **`-h`** Proporciona ayuda de línea de comandos ampliada con ejemplos sobre el uso de AzAcSnap.
- **`-c configure`** Este comando proporciona una interfaz de estilo preguntas y respuestas interactiva para crear o modificar el archivo de configuración de `azacsnap` (valor predeterminado = `azacsnap.json`).
  - **`--configuration new`** Creará un archivo de configuración.
  - **`--configuration edit`** Modificará un archivo de configuración existente.
  - Consulte la [referencia del comando de configuración](azacsnap-cmd-ref-configure.md).
- **`-c test`** Valida el archivo de configuración y la conectividad de prueba.
  - **`--test hana`** Prueba la conexión a la instancia de SAP HANA.
  - **`--test storage`** Prueba la comunicación con la interfaz de almacenamiento subyacente mediante la creación de una instantánea de almacenamiento temporal en todos los volúmenes `data` configurados y, a continuación, su eliminación.
  - **`--test all`** Realizará las pruebas de `hana` y `storage` una detrás de otra.
  - Consulte la [referencia del comando de prueba](azacsnap-cmd-ref-test.md).
- **`-c backup`** Es el comando principal para ejecutar instantáneas de almacenamiento coherentes con la base de datos para los datos (volúmenes de datos SAP HANA) y otros (por ejemplo, volúmenes compartidos, copias de seguridad de registros o arranque).
  - **`--volume data`** Para obtener una instantánea de todos los volúmenes en la estrofa `dataVolume` del archivo de configuración.
  - **`--volume other`** Para obtener una instantánea de todos los volúmenes en la estrofa `otherVolume` del archivo de configuración.
  - Consulte la [referencia del comando de copia de seguridad](azacsnap-cmd-ref-backup.md).
- **`-c details`** Proporciona información sobre las instantáneas o la replicación.
  - **`--details snapshots`** Proporciona una lista de detalles básicos acerca de las instantáneas de cada volumen que se ha configurado.
  - **`--details replication`** Proporciona los detalles básicos del estado de replicación desde el sitio de producción al sitio de recuperación ante desastres.
  - Consulte la [referencia del comando de detalles](azacsnap-cmd-ref-details.md).
- **`-c delete`** Este comando elimina una instantánea de almacenamiento o un conjunto de instantáneas. Puede usar el identificador de copia de seguridad de SAP HANA que se encuentra en HANA Studio o el nombre de la instantánea de almacenamiento. El identificador de copia de seguridad solo está asociado a las instantáneas de `hana`, creadas para los volúmenes de datos y compartidos. En otros casos, si se escribe el nombre de la instantánea, se buscan todas las instantáneas que coincidan con él.
  - Consulte el comando de [eliminación](azacsnap-cmd-ref-delete.md).
- **`-c restore`** proporciona dos métodos para restaurar una instantánea en un volumen, ya sea mediante la creación de un volumen basado en la instantánea o la reversión de un volumen a un estado anterior.
  - **`--restore snaptovol`** Crea un volumen basado en la última instantánea del volumen de destino.
  - **`-c restore --restore revertvolume`** Revierte el volumen de destino a un estado anterior en función de la instantánea más reciente.
  - Consulte la [referencia del comando de restauración](azacsnap-cmd-ref-restore.md).
- **`[--configfile <configfilename>]`** Parámetro de línea de comandos opcional para proporcionar un nombre de archivo de configuración JSON diferente.  Esto es especialmente útil para crear un archivo de configuración independiente por identificador de seguridad (por ejemplo, `--configfile H80.json`).

## <a name="next-steps"></a>Pasos siguientes

- [Primeros pasos con la herramienta Azure Application Consistent Snapshot](azacsnap-get-started.md)
