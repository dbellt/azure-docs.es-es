---
title: Pruebas de la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: En este artículo se explica cómo ejecutar el comando de pruebas de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.openlocfilehash: 86a670602f448f59decbc64d8a12278697c43596
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929856"
---
# <a name="test-azure-application-consistent-snapshot-tool"></a>Prueba de la herramienta Azure Application Consistent Snapshot

En este artículo se explica cómo ejecutar el comando de pruebas de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="introduction"></a>Introducción

Una prueba de la configuración se realiza mediante el comando `azacsnap -c test`.

## <a name="command-options"></a>Opciones de comando

El comando `-c test` tiene las siguientes opciones:

- `--test hana` prueba la conexión a la instancia de SAP HANA.

- `--test storage` prueba la comunicación con la interfaz de almacenamiento subyacente mediante la creación de una instantánea de almacenamiento temporal en todos los volúmenes `data` configurados y, a continuación, su eliminación. 

- `--test all` realizará las pruebas de `hana` y `storage` en secuencia.

- `[--configfile <config filename>]` es un parámetro opcional que permite nombres de archivo de configuración personalizados.

## <a name="check-connectivity-with-sap-hana-azacsnap--c-test---test-hana"></a>Prueba de conectividad con `azacsnap -c test --test hana` de SAP HANA

Este comando comprueba la conectividad de HANA para todas las instancias de HANA en el archivo de configuración. Usa HDBuserstore para conectarse a SYSTEMDB y captura la información del SID.

Para SSL, este comando puede tomar el siguiente argumento opcional:

- `--ssl=` fuerza una conexión cifrada con la base de datos y define el método de cifrado que se usa para comunicarse con SAP HANA, ya sea `openssl` o `commoncrypto`. Si se define, este comando espera encontrar dos archivos en el mismo directorio, estos archivos se deben denominar después del SID correspondiente. Consulte [Uso de SSL para la comunicación con SAP HANA](azacsnap-installation.md#using-ssl-for-communication-with-sap-hana).

### <a name="output-of-the-azacsnap--c-test---test-hana-command"></a>Salida del comando `azacsnap -c test --test hana`

```bash
azacsnap -c test --test hana
```

```output
BEGIN : Test process started for 'hana'
BEGIN : SAP HANA tests
PASSED: Successful connectivity to HANA version 2.00.032.00.1533114046
END   : Test process complete for 'hana'
```

## <a name="check-connectivity-with-storage-azacsnap--c-test---test-storage"></a>Comprobación de la conectividad con el almacenamiento `azacsnap -c test --test storage`

El comando `azacsnap` tomará una instantánea de todos los volúmenes de datos configurados para comprobar que tiene el acceso correcto a los volúmenes de cada instancia de SAP HANA. Se crea una instantánea temporal y después se quita para cada volumen de datos con el fin de comprobar el acceso a la instantánea para cada sistema de archivos.

### <a name="output-of-the-azacsnap--c-test---test-storage-command"></a>Salida del comando `azacsnap -c test --test storage`

```bash
azacsnap -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 2 task(s) to Test Snapshots for Storage Volume Type 'data'
PASSED: Task#2/2 Storage test successful for Volume
PASSED: Task#1/2 Storage test successful for Volume
END   : Storage tests complete
END   : Test process complete for 'storage'
```

> [!NOTE]
> En el caso de instancias grandes de Azure, el comando `azacsnap -c test --test storage` extrapola la generación de almacenamiento y la SKU de HLI.  En función de esta información, se proporcionan instrucciones sobre la configuración de las instantáneas de "arranque" (vea la línea que comienza con la salida `Action:`).

```output
SID1   : Generation 4
Storage: ams07-a700s-saphan-1-01v250-client25-nprod
HLI SKU: S96
Action : Configure the 'boot' snapshots on ALL the servers.
```

## <a name="next-steps"></a>Pasos siguientes

- [Copia de seguridad de instantáneas con la herramienta Azure Application Consistent Snapshot](azacsnap-cmd-ref-backup.md)
