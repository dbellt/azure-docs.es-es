---
title: Configuración de la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una guía para ejecutar el comando de configuración de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.openlocfilehash: f12e9ab2ab0faf77520805e3be645a14dbd38885
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930000"
---
# <a name="configure-azure-application-consistent-snapshot-tool"></a>Configuración de la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía para ejecutar el comando de configuración de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="introduction"></a>Introducción

El archivo de configuración se puede crear o editar mediante el comando `azacsnap -c configure`.

## <a name="command-options"></a>Opciones de comando

El comando `-c configure` tiene las siguientes opciones:

- `--configuration new` para crear un archivo de configuración nuevo.

- `--configuration edit` para editar un archivo de configuración existente.

- `[--configfile <config filename>]` es un parámetro opcional que permite nombres de archivo de configuración personalizados.

## <a name="configuration-file-for-snapshot-tools"></a>Archivo de configuración para herramientas de instantáneas

Se puede crear un archivo de configuración mediante la ejecución de `azacsnap -c configure --configuration new`.  De manera predeterminada, el nombre de archivo de configuración es `azacsnap.json`.  Un nombre de archivo personalizado se puede usar con el parámetro `--configfile=` (por ejemplo, `--configfile=<customname>.json`). El ejemplo siguiente es para la configuración de instancias grandes de Azure:

```bash
azacsnap -c configure --configuration new
```

```output
Building new config file
Add comment to config file (blank entry to exit adding comments):This is a new config file for `azacsnap`
Add comment to config file (blank entry to exit adding comments):
Add database to config? (y/n) [n]: y
HANA SID (for example, H80): H80
HANA Instance Number (for example, 00): 00
HANA HDB User Store Key (for example, `hdbuserstore List`): AZACSNAP
HANA Server's Address (hostname or IP address): testing01
Add ANF Storage to database section? (y/n) [n]:
Add HLI Storage to database section? (y/n) [n]: y
Add DATA Volume to HLI Storage section of Database section? (y/n) [n]: y
Storage User Name (for example, clbackup25): clt1h80backup
Storage IP Address (for example, 192.168.1.30): 172.18.18.11
Storage Volume Name (for example, hana_data_h80_testing01_mnt00001_t250_vol): hana_data_h80_testing01_mnt00001_t020_vol
Add DATA Volume to HLI Storage section of Database section? (y/n) [n]:
Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]:
Add HLI Storage to database section? (y/n) [n]:
Add database to config? (y/n) [n]:
Editing configuration complete, writing output to 'azacsnap.json'
```

## <a name="details-of-required-values"></a>Detalles de los valores necesarios

En las secciones siguientes se proporciona una guía detallada sobre los distintos valores necesarios para el archivo de configuración.

### <a name="sap-hana-values"></a>Valores de SAP HANA

Al agregar una *base de datos* a la configuración, se requieren los siguientes valores:

- **Dirección del servidor de HANA** = nombre de host o dirección IP del servidor de SAP HANA.
- **SID de HANA** = identificador del sistema de SAP HANA.
- **Número de instancia de HANA** = número de instancia de SAP HANA.
- **Clave de almacenamiento de usuario de HDB de HANA** = usuario de SAP HANA configurado con permisos para ejecutar copias de seguridad de bases de datos.

- Nodo único: dirección IP y nombre de host del nodo.
- HSR con STONITH: dirección IP y nombre de host del nodo.
- Escalabilidad horizontal (N + N, N + M): dirección IP y nombre de host del nodo maestro actual.
- HSR sin STONITH: dirección IP y nombre de host del nodo.
- SID múltiple en un nodo único: nombre de host y dirección IP del nodo que hospeda esos SID.

### <a name="azure-large-instance-hli-storage-values"></a>Valores de almacenamiento de instancias grandes de Azure (HLI)

Al agregar un *almacenamiento de HLI* a la sección de la base de datos, se requieren los siguientes valores:

- **Nombre de usuario de almacenamiento** = este valor es el nombre de usuario que se usa para establecer la conexión SSH con el almacenamiento.
- **Dirección IP de almacenamiento** = dirección del sistema de almacenamiento.
- **Nombre del volumen de almacenamiento** = nombre del volumen del que se va a realizar la instantánea.  Este valor se puede determinar de varias maneras, quizás la más sencilla consiste en probar el siguiente comando de shell:

    ```bash
    grep nfs /etc/fstab | cut -f2 -d"/" | sort | uniq
    ```

    ```output
    hana_data_p40_soldub41_mnt00001_t020_vol
    hana_log_backups_p40_soldub41_t020_vol
    hana_log_p40_soldub41_mnt00001_t020_vol
    hana_shared_p40_soldub41_t020_vol
    ```

### <a name="azure-netapp-files-anf-storage-values"></a>Valores de almacenamiento de Azure NetApp Files (ANF)

Al agregar un *almacenamiento de ANF* a la sección de la base de datos, se requieren los siguientes valores:

- **Nombre de archivo de autenticación de la entidad de servicio** = es el archivo `authfile.json` generado en Cloud Shell al configurar la comunicación con el almacenamiento de Azure NetApp Files.
- **Id. de recurso del volumen de almacenamiento de ANF completo** = id. de recurso completo del volumen del que se está realizando la instantánea.  Se puede recuperar desde: Azure Portal –> ANF –> Volumen –> Configuración/Propiedades –> Id. de recurso

## <a name="configuration-file-overview-azacsnapjson"></a>Información general sobre el archivo de configuración (`azacsnap.json`)

En el ejemplo siguiente, `azacsnap.json` se configura con el SID.

Los valores de parámetro se deben establecer en el entorno de SAP HANA específico del cliente.
En el caso del sistema de **instancias grandes de Azure**, esta información la proporciona la administración de servicios de Microsoft durante la llamada de incorporación o entrega y está disponible en un archivo de Excel que se proporciona durante la entrega. Abra una solicitud de servicio si necesita volver a proporcionar esta información.

El siguiente es solo un ejemplo. Actualice todos los valores según corresponda.

```bash
cat azacsnap.json
```

```output
{
  "version": "5.0",
  "logPath": "./logs",
  "securityPath": "./security",
  "comments": [],
  "database": [
    {
      "hana": {
        "serverAddress": "sapprdhdb80",
        "sid": "H80",
        "instanceNumber": "00",
        "hdbUserStoreName": "SCADMIN",
        "savePointAbortWaitSeconds": 600,
        "hliStorage": [
          {
            "dataVolume": [
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_data_h80_azsollabbl20a31_mnt00001_t210_vol"
              },
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_shared_h80_azsollabbl20a31_t210_vol"
              }
            ],
            "otherVolume": [
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_log_backups_h80_azsollabbl20a31_t210_vol"
              }
            ]
          }
        ],
        "anfStorage": []
      }
    }
  ]
}
```

> [!NOTE]
> En el caso de un escenario de recuperación ante desastres en el que se van a ejecutar copias de seguridad en el sitio de recuperación ante desastres, el nombre del servidor de HANA configurado en el archivo de configuración de recuperación ante desastres (por ejemplo, `DR.json`) en el sitio de recuperación ante desastres debe ser el mismo que el nombre del servidor de producción.

> [!NOTE]
> En el caso de instancias grandes de Azure, la dirección IP de almacenamiento debe estar en la misma subred que el grupo de servidores. Por ejemplo, en este caso, nuestra subred del grupo de servidores es 172. 18. 18.0/24 y nuestra dirección IP de almacenamiento asignada es 172.18.18.11.

## <a name="next-steps"></a>Pasos siguientes

- [Prueba de la herramienta Azure Application Consistent Snapshot](azacsnap-cmd-ref-test.md)
