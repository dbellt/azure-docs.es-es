---
title: Creación de una base de datos Oracle en una VM de Azure | Microsoft Docs
description: Ponga en funcionamiento rápidamente una base de datos de Oracle Database 12c en el entorno de Azure.
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: quickstart
ms.date: 10/05/2020
ms.author: kegorman
ms.openlocfilehash: 4b15d683a54f665e948dc31b51df039d20c58f7a
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110081378"
---
# <a name="create-an-oracle-database-in-an-azure-vm"></a>Creación de una base de datos Oracle en una VM de Azure

En esta guía se describe el uso de la CLI de Azure para implementar una máquina virtual de Azure a partir de la [imagen de la galería del marketplace de Oracle](https://azuremarketplace.microsoft.com/marketplace/apps/Oracle.OracleDatabase12102EnterpriseEdition?tab=Overview) con el fin de crear una base de datos de Oracle 19c. Una vez implementado el servidor, el usuario conectará a través de SSH con el fin de configurar la base de datos Oracle. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido es preciso que ejecute la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente se crea un grupo de recursos llamado *rg-oracle* en la ubicación *eastus*.

```azurecli-interactive
az group create --name rg-oracle --location eastus
```

## <a name="create-virtual-machine"></a>Crear máquina virtual

Para crear una máquina virtual, use el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente se crea una máquina virtual denominada `vmoracle19c`. También se crean claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.  

```azurecli-interactive 
az vm create ^
    --resource-group rg-oracle ^
    --name vmoracle19c ^
    --image Oracle:oracle-database-19-3:oracle-database-19-0904:latest ^
    --size Standard_DS2_v2 ^
    --admin-username azureuser ^
    --generate-ssh-keys ^
    --public-ip-address-allocation static ^
    --public-ip-address-dns-name vmoracle19c

```

Después de crear la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. Tome nota del valor de `publicIpAddress`, Use esta dirección para acceder a la máquina virtual.

```output
{
  "fqdns": "",
  "id": "/subscriptions/{snip}/resourceGroups/rg-oracle/providers/Microsoft.Compute/virtualMachines/vmoracle19c",
  "location": "eastus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "rg-oracle"
}
```
## <a name="create-and-attach-a-new-disk-for-oracle-datafiles-and-fra"></a>Creación y asociación de un disco nuevo para los archivos de datos de Oracle y FRA

```bash
az vm disk attach --name oradata01 --new --resource-group rg-oracle --size-gb 64 --sku StandardSSD_LRS --vm-name vmoracle19c
```

## <a name="open-ports-for-connectivity"></a>Abrir puertos para la conectividad
En esta tarea, debe configurar algunos puntos de conexión externos para el cliente de escucha de la base de datos y EM Express mediante la configuración del grupo de seguridad de red de Azure que protege la máquina virtual. 

1. Para abrir el punto de conexión que se usa para acceder a la base de datos Oracle de forma remota, cree una regla de grupo de seguridad de red de la manera siguiente:
   ```bash
   az network nsg rule create ^
       --resource-group rg-oracle ^
       --nsg-name vmoracle19cNSG ^
       --name allow-oracle ^
       --protocol tcp ^
       --priority 1001 ^
       --destination-port-range 1521
   ```
2. Para abrir el punto de conexión que usa para acceder a Oracle EM Express de forma remota, cree una regla de grupo de seguridad de red con az network nsg rule create de la manera siguiente:
   ```bash
   az network nsg rule create ^
       --resource-group rg-oracle ^
       --nsg-name vmoracle19cNSG ^
       --name allow-oracle-EM ^
       --protocol tcp ^
       --priority 1002 ^
       --destination-port-range 5502
   ```
3. Si es necesario, obtenga de nuevo la dirección IP pública de la máquina virtual con az network public-ip show de la manera siguiente:

   ```bash
   az network public-ip show ^
       --resource-group rg-oracle ^
       --name vmoracle19cPublicIP ^
       --query [ipAddress] ^
       --output tsv
   ```

## <a name="prepare-the-vm-environment"></a>Preparación del entorno de la máquina virtual

1. Conexión a la máquina virtual

   Para crear una sesión de SSH con la máquina virtual, use el siguiente comando. Reemplace la dirección IP por el valor `publicIpAddress` para la máquina virtual.

   ```bash
   ssh azureuser@<publicIpAddress>
   ```

2. Cambie al usuario raíz.

   ```bash
   sudo su -
   ```

3. Compruebe el último dispositivo de disco creado al que se va a dar formato para usar archivos de datos de Oracle contenedores.

   ```bash
   ls -alt /dev/sd*|head -1
   ```

   La salida debe ser similar a esta:
   ```output
   brw-rw----. 1 root disk 8, 16 Dec  8 22:57 /dev/sdc
   ```

4. Aplique formato al dispositivo. 
   Como usuario raíz, ejecute parted en el dispositivo. 
   
   En primer lugar, cree una etiqueta de disco:
   ```bash
   parted /dev/sdc mklabel gpt
   ```
   A continuación, cree una partición primaria que abarque todo el disco:
   ```bash
   parted -a optimal /dev/sdc mkpart primary 0GB 64GB   
   ```
   Por último, consulte los detalles del dispositivo mediante la impresión de sus metadatos:
   ```bash
   parted /dev/sdc print
   ```
   La salida debería tener un aspecto similar al siguiente:
   ```bash
   # parted /dev/sdc print
   Model: Msft Virtual Disk (scsi)
   Disk /dev/sdc: 68.7GB
   Sector size (logical/physical): 512B/4096B
   Partition Table: gpt
   Disk Flags:
   Number  Start   End     Size    File system  Name     Flags
    1      1049kB  64.0GB  64.0GB  ext4         primary
   ```

5. Cree un sistema de archivos en la partición del dispositivo.

   ```bash
   mkfs -t ext4 /dev/sdc1
   ```

6. Creación de un punto de montaje
   ```bash
   mkdir /u02
   ```

7. Montaje del disco

   ```bash
   mount /dev/sdc1 /u02
   ```

8. Cambie los permisos sobre el punto de montaje.

   ```bash
   chmod 777 /u02
   ```

9. Agregue el montaje al archivo/etc/fstab. 

   ```bash
   echo "/dev/sdc1               /u02                    ext4    defaults        0 0" >> /etc/fstab
   ```
   
10. Actualice el archivo ***/etc/hosts*** con la dirección IP pública y el nombre de host.

    Cambie la ***dirección IP pública y el nombre de la máquina virtual*** para que reflejen los valores reales:
  
    ```bash
    echo "<Public IP> <VMname>.eastus.cloudapp.azure.com <VMname>" >> /etc/hosts
    ```
11. Actualice el archivo de nombre de host.
    
    Use el comando siguiente para agregar el nombre de dominio de la máquina virtual al archivo **/etc/hostname**. Se supone que ha creado el grupo de recursos y la máquina virtual en la región **eastus**:
    
    ```bash
    sed -i 's/$/\.eastus\.cloudapp\.azure\.com &/' /etc/hostname
    ```

12. Abra los puertos de firewall.
    
    Como SELinux está habilitado de forma predeterminada en la imagen del marketplace, es necesario abrir el firewall al tráfico del puerto del cliente de escucha de la base de datos 1521 y al puerto de Enterprise Manager Express 5502. Ejecute los siguientes comandos como usuario raíz:

    ```bash
    firewall-cmd --zone=public --add-port=1521/tcp --permanent
    firewall-cmd --zone=public --add-port=5502/tcp --permanent
    firewall-cmd --reload
    ```
   

## <a name="create-the-database"></a>Creación de la base de datos

El software de Oracle ya está instalado en la imagen de Marketplace. Cree una base de datos de ejemplo de la manera siguiente: 

1.  Cambie al usuario **oracle**:

    ```bash
    sudo su - oracle
    ```
2. Iniciar el agente de escucha de la base de datos

   ```bash
   lsnrctl start
   ```
   La salida será similar a la siguiente:
  
   ```output
   LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 20-OCT-2020 01:58:18

   Copyright (c) 1991, 2019, Oracle.  All rights reserved.

   Starting /u01/app/oracle/product/19.0.0/dbhome_1/bin/tnslsnr: please wait...

   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
   Log messages written to /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
   Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))

   Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
   STATUS of the LISTENER
   ------------------------
   Alias                     LISTENER
   Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
   Start Date                20-OCT-2020 01:58:18
   Uptime                    0 days 0 hr. 0 min. 0 sec
   Trace Level               off
   Security                  ON: Local OS Authentication
   SNMP                      OFF
   Listener Log File         /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
   Listening Endpoints Summary...
     (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))
   The listener supports no services
   The command completed successfully
   ```
3. Cree un directorio de datos para los archivos de datos de Oracle.

   ```bash
   mkdir /u02/oradata
   ```
    

3.  Ejecute el Asistente para la creación de bases de datos:

    ```bash
    dbca -silent \
       -createDatabase \
       -templateName General_Purpose.dbc \
       -gdbname test \
       -sid test \
       -responseFile NO_VALUE \
       -characterSet AL32UTF8 \
       -sysPassword OraPasswd1 \
       -systemPassword OraPasswd1 \
       -createAsContainerDatabase false \
       -databaseType MULTIPURPOSE \
       -automaticMemoryManagement false \
       -storageType FS \
       -datafileDestination "/u02/oradata/" \
       -ignorePreReqs
    ```

    La operación de creación de la base de datos tarda unos minutos.

    Verá una salida similar a la siguiente:

    ```output
        Prepare for db operation
       10% complete
       Copying database files
       40% complete
       Creating and starting Oracle instance
       42% complete
       46% complete
       50% complete
       54% complete
       60% complete
       Completing Database Creation
       66% complete
       69% complete
       70% complete
       Executing Post Configuration Actions
       100% complete
       Database creation complete. For details check the logfiles at: /u01/app/oracle/cfgtoollogs/dbca/test.
       Database Information:
       Global Database Name:test
       System Identifier(SID):test
       Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/test/test.log" for further details.
    ```

4. Establecimiento de las variables de Oracle

    Antes de conectarse, debe establecer la variable de entorno *ORACLE_SID*:

    ```bash
        export ORACLE_SID=test
    ```

    También debe agregar la variable ORACLE_SID al archivo `.bashrc` de usuarios de `oracle` para los inicios de sesión futuros con el siguiente comando:

    ```bash
    echo "export ORACLE_SID=test" >> ~oracle/.bashrc
    ```

## <a name="oracle-em-express-connectivity"></a>Conectividad con Oracle EM Express

Para disponer de una herramienta de administración de GUI con la que pueda explorar la base de datos, configure Oracle EM Express. Para conectarse a Oracle EM Express, primero debe configurar el puerto en Oracle. 

1. Conéctese a la base de datos mediante sqlplus:

    ```bash
    sqlplus sys as sysdba
    ```

2. Una vez conectado, establezca el puerto 5502 para EM Express.

    ```bash
    exec DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
    ```

3.  Conecte EM Express desde el explorador. Asegúrese de que el explorador sea compatible con Express EM (es necesario que Flash esté instalado): 

    ```https
    https://<VM ip address or hostname>:5502/em
    ```

    Puede iniciar sesión mediante la cuenta **SYS** y activar la casilla **as sysdba**. Use la contraseña **OraPasswd1** que estableció durante la instalación. 

    ![Captura de pantalla de la página de inicio de sesión de Oracle OEM Express](./media/oracle-quick-start/oracle_oem_express_login.png)

## <a name="automate-database-startup-and-shutdown"></a>Automatizar el arranque y el apagado de la base de datos

De forma predeterminada, la base de datos Oracle no se inicia automáticamente al reiniciar la máquina virtual. Para configurar la base de datos Oracle para iniciarse automáticamente, inicie primero una sesión como raíz. Después, cree y actualice algunos archivos del sistema.

1. Inicio de sesión como raíz

    ```bash
    sudo su -
    ```

2.  Ejecute el siguiente comando para cambiar la marca de inicio automatizada de `N` a `Y` en el archivo `/etc/oratab`:

    ```bash
    sed -i 's/:N/:Y/' /etc/oratab
    ```

3.  Cree un archivo denominado `/etc/init.d/dbora` y pegue el siguiente contenido:

    ```bash
    #!/bin/sh
    # chkconfig: 345 99 10
    # Description: Oracle auto start-stop script.
    #
    # Set ORA_HOME to be equivalent to $ORACLE_HOME.
    ORA_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
    ORA_OWNER=oracle

    case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        # Remove "&" if you don't want startup as a background process.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart $ORA_HOME" &
        touch /var/lock/subsys/dbora
        ;;

    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut $ORA_HOME" &
        rm -f /var/lock/subsys/dbora
        ;;
    esac
    ```

4.  Cambie los permisos de los archivos con *chmod* como se indica a continuación:

    ```bash
    chgrp dba /etc/init.d/dbora
    chmod 750 /etc/init.d/dbora
    ```

5.  Cree vínculos simbólicos para el inicio y el apagado de la manera siguiente:

    ```bash
    ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
    ```

6.  Para probar los cambios, reinicie la máquina virtual:

    ```bash
    reboot
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de explorar la primera base de datos Oracle en Azure y la VM ya no se necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, la VM y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a proteger la base de datos en Azure con [Estrategias de copia de seguridad de Oracle](./oracle-database-backup-strategies.md)

Conozca otras [soluciones de Oracle en Azure](./oracle-overview.md). 

Pruebe el tutorial sobre la [instalación y configuración de Oracle Automated Storage Management](configure-oracle-asm.md).
