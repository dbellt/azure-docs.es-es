---
title: Solución de problemas con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece contenido de solución de problemas para el uso de la herramienta Azure Application Consistent Snapshot con Azure NetApp Files.
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
ms.topic: troubleshooting
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: 66272e59dd67375b1da119648cf461bd0c2baa9b
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929604"
---
# <a name="troubleshoot-azure-application-consistent-snapshot-tool"></a>Solución de problemas con la herramienta Azure Application Consistent Snapshot

Este artículo ofrece contenido de solución de problemas para su uso con la herramienta Azure Application Consistent Snapshot que se puede usar con Azure NetApp Files.

A continuación se indican algunos problemas comunes que pueden surgir al ejecutar los comandos. Siga las instrucciones de resolución mencionadas para solucionar el problema. Si el problema persiste, abra una solicitud de servicio técnico desde Azure Portal y asigne la solicitud a la cola de instancias grandes de SAP HANA para que responda el equipo de Soporte técnico de Microsoft.

## <a name="log-files"></a>Archivos de registro

Uno de los mejores orígenes de información para depurar los errores con AzAcSnap son los archivos de registro.  

### <a name="log-file-location"></a>Ubicación del archivo de registro

Los archivos de registro se almacenan en el directorio configurado según el parámetro `logPath` en el archivo de configuración de AzAcSnap.  El nombre de archivo de configuración predeterminado es `azacsnap.json` y el valor predeterminado de `logPath` es `"./logs"`, lo que significa que los archivos de registro se escriben en el directorio `./logs` de la ubicación en la que se ejecuta el comando `azacsnap`.  Al convertir `logPath` en una ubicación absoluta (por ejemplo, `/home/azacsnap/logs`), se asegurará de que `azacsnap` genera los registros en `/home/azacsnap/logs`, independientemente de dónde se haya ejecutado el comando `azacsnap`.

### <a name="log-file-naming"></a>Nomenclatura de los archivos de registro

El nombre del archivo de registro se basa en el nombre de la aplicación (por ejemplo, `azacsnap`), la opción de comando (`-c`) utilizada (por ejemplo, `backup`, `test`, `details`, etc.) y el nombre del archivo de configuración (por ejemplo, el valor predeterminado = `azacsnap.json`).  Por lo tanto, si usa el comando `-c backup`, el nombre del archivo de registro predeterminado es `azacsnap-backup-azacsnap.log` y se escribe en el directorio configurado por `logPath`.  

Esta convención de nomenclatura se estableció para permitir varios archivos de configuración, uno por cada base de datos, y facilitar la ubicación de los archivos de registro asociados.  Por lo tanto, si el nombre del archivo de configuración es `SID.json`, el nombre de archivo de resultados al usar las opciones de `azacsnap -c backup --configfile SID.json` será `azacsnap-backup-SID.log`.

### <a name="result-file-and-syslog"></a>Archivo de resultados y syslog

Para la opción de comando `-c backup`, AzAcSnap escribe en un archivo `*.result` y en el registro del sistema (`/var/log/messages`) con el comando `logger`.  El nombre del archivo `*.result` tiene el mismo nombre de base que el [archivo de registro](#log-file-naming) y entra en la misma [ubicación que el archivo de registro](#log-file-location).  Se trata de un archivo de salida simple de una línea como en los ejemplos siguientes.

Salida de ejemplo del archivo `*.result`.
```output
Database # 1 (PR1) : completed ok
```

Salida de ejemplo del archivo `/var/log/messages`.
```output
Dec 17 09:01:13 azacsnap-rhel azacsnap: Database # 1 (PR1) : completed ok
```

## <a name="failed-communication-with-azure-netapp-files"></a>Error de comunicación con Azure NetApp Files

Cuando se valida la comunicación con Azure NetApp Files, se puede producir un error o agotarse el tiempo de espera.  Asegúrese de que las reglas de firewall no bloqueen el tráfico de salida del sistema que ejecuta AzAcSnap en las siguientes direcciones y puertos TCP/IP:

- (https://)management.azure.com:443
- (https://)login.microsoftonline.com:443 

## <a name="failed-communication-with-sap-hana"></a>Error de comunicación con SAP HANA

Al validar la comunicación con SAP HANA mediante la ejecución de una prueba con `azacsnap -c test --test hana`, se muestra el siguiente error:

```output
> azacsnap -c test --test hana
BEGIN : Test process started for 'hana'
BEGIN : SAP HANA tests
CRITICAL: Command 'test' failed with error:
Cannot get SAP HANA version, exiting with error: 127
```

**Solución:**

1. Compruebe el archivo de configuración (por ejemplo, `azacsnap.json`) de cada instancia de HANA para asegurarse de que los valores de la base de datos de SAP HANA sean correctos.
1. Intente ejecutar el siguiente comando para comprobar si el comando `hdbsql` está en la ruta de acceso y puede conectarse al servidor de SAP HANA. En el ejemplo siguiente se muestra la ejecución correcta del comando y su salida.

    ```bash
    hdbsql -n 172.18.18.50 - i 00 -d SYSTEMDB -U AZACSNAP "\s"
    ```

    ```output
    host          : 172.18.18.50
    sid           : H80
    dbname        : SYSTEMDB
    user          : AZACSNAP
    kernel version: 2.00.040.00.1553674765
    SQLDBC version:        libSQLDBCHDB 2.04.126.1551801496
    autocommit    : ON
    locale        : en_US.UTF-8
    input encoding: UTF8
    sql port      : saphana1:30013
    ```

    En este ejemplo, el comando `hdbsql` no está en el `$PATH` del usuario.

    ```bash
    hdbsql -n 172.18.18.50 - i 00 -U SCADMIN "select version from sys.m_database"
    ```

    ```output
    If 'hdbsql' is not a typo you can use command-not-found to lookup the package that contains it, like this:
    cnf hdbsql
    ```

    En este ejemplo, el comando `hdbsql` se agrega temporalmente al `$PATH` del usuario, pero cuando se ejecuta muestra que la clave de conexión no se ha configurado correctamente con el comando `hdbuserstore Set` (consulte la guía de introducción para más información):

    ```bash
    export PATH=$PATH:/hana/shared/H80/exe/linuxx86_64/hdb/
    ```

    ```bash
    hdbsql -n 172.18.18.50 -i 00 -U SCADMIN "select version from sys.m_database"
    ```

    ```output
    * -10104: Invalid value for KEY (SCADMIN)
    ```

    > [!NOTE]
    > Para la incorporación permanente al `$PATH` del usuario, actualice el archivo `$HOME/.profile` del usuario.

## <a name="the-hdbuserstore-location"></a>Ubicación de `hdbuserstore`

Al configurar la comunicación con SAP HANA se usa el programa `hdbuserstore` para crear la configuración de comunicación segura.  El programa `hdbuserstore` suele encontrarse en `/usr/sap/<SID>/SYS/exe/hdb/` o en `/usr/sap/hdbclient`.  Normalmente, el instalador agrega la ubicación correcta al `$PATH` del usuario de `azacsnap`.

## <a name="failed-test-with-storage"></a>Prueba no superada con el almacenamiento

El comando `azacsnap -c test --test storage` no se completa correctamente.

### <a name="azure-large-instance"></a>Azure (instancias grandes)

El ejemplo siguiente procede de la ejecución de `azacsnap` en SAP HANA en Azure Large Instance:

```bash
azacsnap -c test --test storage
```

```output
The authenticity of host '172.18.18.11 (172.18.18.11)' can't be established.
ECDSA key fingerprint is SHA256:QxamHRn3ZKbJAKnEimQpVVCknDSO9uB4c9Qd8komDec.
Are you sure you want to continue connecting (yes/no)?
```

**Solución:** El error anterior suele mostrarse cuando el usuario de almacenamiento de Azure Large Instance no tiene acceso al almacenamiento subyacente. Para permitir el acceso al almacenamiento con el usuario de almacenamiento proporcionado, ejecute el comando `ssh` y valide la comunicación con la plataforma de almacenamiento.

```bash
ssh <StorageBackupname>@<Storage IP address> "volume show -fields volume"
```

Ejemplo con la salida esperada:

```bash
ssh clt1h80backup@10.8.0.16 "volume show -fields volume"
```

```output
vserver volume
--------------------------------- ------------------------------
osa33-hana-c01v250-client25-nprod hana_data_h80_mnt00001_t020_vol
osa33-hana-c01v250-client25-nprod hana_data_h80_mnt00002_t020_vol
```

#### <a name="the-authenticity-of-host-172181811-172181811-cant-be-established"></a>No se puede establecer la autenticidad del host "172.18.18.11 (172.18.18.11)".

```bash
azacsnap -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 1 task(s) to Test Snapshots for Storage Volume Type 'data'
The authenticity of host '10.3.0.18 (10.3.0.18)' can't be established.
ECDSA key fingerprint is SHA256:cONAr0lpafb7gY4l31AdWTzM3s9LnKDtpMdPA+cxT7Y.
Are you sure you want to continue connecting (yes/no)?
```

**Solución:** No seleccione Yes (Sí). Asegúrese de que la dirección IP de almacenamiento sea correcta. Si el problema persiste, confirme la dirección IP de almacenamiento con el equipo de Microsoft Operations.

### <a name="azure-netapp-files"></a>Azure NetApp Files

El ejemplo siguiente procede de la ejecución de `azacsnap` en una máquina virtual con Azure NetApp Files:

```bash
azacsnap --configfile azacsnap.json.NOT-WORKING -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 1 task(s) to Test Snapshots for Storage Volume Type 'data'
ERROR: Could not create StorageANF object [authFile = 'azureauth.json']
```

**Solución:**

1. Compruebe la existencia del archivo de la entidad de servicio, `azureauth.json`, como se estableció en el archivo de configuración `azacsnap.json`.
1. Compruebe el archivo de registro (por ejemplo, `logs/azacsnap-test-azacsnap.log`) para ver si la entidad de servicio (`azureauth.json`) tiene el contenido correcto.  Ejemplo de registro a continuación:

      ```output
      [19/Nov/2020:18:39:49 +13:00] DEBUG: [PID:0020080:StorageANF:659] [1] Innerexception: Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException AADSTS7000215: Invalid client secret is provided.
      ```

1. Compruebe el archivo de registro (por ejemplo, `logs/azacsnap-test-azacsnap.log`) para ver si la entidad de servicio (`azureauth.json`) ha expirado. Ejemplo de registro a continuación:

      ```output
      [19/Nov/2020:18:41:10 +13:00] DEBUG: [PID:0020257:StorageANF:659] [1] Innerexception: Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException AADSTS7000222: The provided client secret keys are expired. Visit the Azure Portal to create new keys for your app, or consider using certificate credentials for added security: https://docs.microsoft.com/azure/active-directory/develop/active-directory-certificate-credentials
      ```

## <a name="next-steps"></a>Pasos siguientes

- [Sugerencias](azacsnap-tips.md)
