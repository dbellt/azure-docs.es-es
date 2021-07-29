---
title: Instalación de la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
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
ms.date: 05/19/2021
ms.author: phjensen
ms.openlocfilehash: 0eae929d60ce852e7a98f12430de796aa664d86f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110481456"
---
# <a name="install-azure-application-consistent-snapshot-tool"></a>Instalación de la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía de instalación de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files o Azure (instancias grandes).

> [!IMPORTANT]
> Las instalaciones distribuidas son la única opción para los sistemas de **Azure (instancias grandes)** , ya que se implementan en una red privada.  Por lo tanto, las instalaciones de AzAcSnap deben realizarse en cada sistema para garantizar la conectividad.

## <a name="introduction"></a>Introducción

El instalador automático descargable está diseñado para facilitar la configuración y la ejecución de las herramientas de instantáneas con privilegios de usuario no raíz (por ejemplo, azacsnap). El instalador configurará el usuario y colocará las herramientas de instantáneas en el subdirectorio `$HOME/bin` de este (valor predeterminado = `/home/azacsnap/bin`).
El instalador automático intenta determinar la configuración correcta y las rutas de acceso de todos los archivos en función de la configuración del usuario que realiza la instalación (por ejemplo, raíz). Si los pasos de los requisitos previos (habilitar la comunicación con el almacenamiento y SAP HANA) se ejecutan como raíz, durante la instalación se copian la clave privada y `hdbuserstore` en la ubicación del usuario de copia de seguridad. Sin embargo, un administrador capacitado puede realizar los pasos para permitir la comunicación con el back-end de almacenamiento y SAP HANA manualmente tras la instalación.

## <a name="prerequisites-for-installation"></a>Requisitos previos a la instalación

Siga las instrucciones para configurar y ejecutar los comandos de instantáneas y de recuperación ante desastres. Se recomienda completar los pasos siguientes como raíz antes de instalar y usar las herramientas de instantáneas.

1. **Sistema operativo revisado**. Consulte la configuración de SMT y las revisiones en [Instalación y configuración de SAP HANA (instancias grandes) en Azure](../virtual-machines/workloads/sap/hana-installation.md#operating-system).
1. **Sincronización temporal configurada**. El cliente deberá proporcionar un servidor horario compatible con NTP y configurar el sistema operativo conforme a este.
1. **HANA instalado**. Consulte las instrucciones de instalación de HANA en [Instalación de SAP NetWeaver en la base de datos de HANA](/archive/blogs/saponsqlserver/sap-netweaver-installation-on-hana-database).
1. **[Comunicación habilitada con el almacenamiento](#enable-communication-with-storage)** (consulte la sección independiente para más detalles). El cliente debe configurar SSH con un par de claves pública y privada, y proporcionar la clave pública para cada nodo donde está previsto que se ejecuten las herramientas de instantáneas durante la configuración del back-end de almacenamiento de Microsoft Operations.
   1. **Para Azure NetApp Files (consulte la sección independiente para más detalles)** . El cliente debe generar el archivo de autenticación de la entidad de servicio.
      
      > [!IMPORTANT]
      > Cuando se valida la comunicación con Azure NetApp Files, se puede producir un error o agotarse el tiempo de espera. Asegúrese de que las reglas de firewall no bloqueen el tráfico de salida del sistema que ejecuta AzAcSnap en las siguientes direcciones y puertos TCP/IP:
      > - (https://)management.azure.com:443
      > - (https://)login.microsoftonline.com:443
      
   1. **Para Azure Large Instance (consulte la sección independiente para más detalles)** . El cliente debe configurar SSH con un par de claves pública y privada, y proporcionar la clave pública para cada nodo donde está previsto que se ejecuten las herramientas de instantáneas durante la configuración del back-end de almacenamiento de Microsoft Operations.

      Para probarlo, use SSH para conectarse a uno de los nodos (por ejemplo, `ssh -l <Storage UserName> <Storage IP Address>`).
      Escriba `exit` para cerrar la sesión del símbolo del sistema de almacenamiento.

      Microsoft Operations proporcionará el usuario de almacenamiento y la dirección IP de almacenamiento en el momento del aprovisionamiento.
  
1. **[Comunicación habilitada con SAP HANA](#enable-communication-with-sap-hana)** (consulte la sección independiente para más detalles). El cliente debe configurar un usuario de SAP HANA adecuado con los privilegios necesarios para realizar la instantánea.
   1. Esta configuración se puede probar desde la línea de comandos como se indica a continuación con el texto en `grey`.
      1. HANAv1

            `hdbsql -n <HANA IP address> -i <HANA instance> -U <HANA user> "\s"`

      1. HANAv2

            `hdbsql -n <HANA IP address> -i <HANA instance> -d SYSTEMDB -U <HANA user> "\s"`

      - Los ejemplos anteriores son para la comunicación con SAP HANA por medios distintos de SSL.

## <a name="enable-communication-with-storage"></a>Habilitación de la comunicación con el almacenamiento

En esta sección se explica cómo habilitar la comunicación con el almacenamiento.

### <a name="azure-netapp-files"></a>Azure NetApp Files

Creación de una entidad de servicio de RBAC

1. En una sesión de Azure Cloud Shell, asegúrese de que ha iniciado sesión en la suscripción donde desee que se le asocie la entidad de servicio de manera predeterminada:

    ```azurecli-interactive
    az account show
    ```

1. Si la suscripción no es correcta, use:

    ```azurecli-interactive
    az account set -s <subscription name or id>
    ```

1. Cree una entidad de servicio con la CLI de Azure como en el ejemplo siguiente:

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth
    ```

    1. Esto debería generar una salida similar a la del ejemplo siguiente:

        ```output
        {
          "clientId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "clientSecret": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "subscriptionId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "tenantId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
          "resourceManagerEndpointUrl": "https://management.azure.com/",
          "activeDirectoryGraphResourceId": "https://graph.windows.net/",
          "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
          "galleryEndpointUrl": "https://gallery.azure.com/",
          "managementEndpointUrl": "https://management.core.windows.net/"
        }
        ```

    > Este comando asignará automáticamente el rol colaborador de RBAC a la entidad de servicio en el nivel de suscripción; puede limitar el ámbito al grupo de recursos específico donde se crearán los recursos en las pruebas.

1. Corte y pegue el contenido de la salida en un archivo denominado `azureauth.json` almacenado en el mismo sistema que el comando `azacsnap`, y proteja el archivo con los permisos de sistema adecuados.

### <a name="azure-large-instance"></a>Azure (instancias grandes)

La comunicación con el back-end de almacenamiento se ejecuta a través de un canal SSH cifrado. Los siguientes pasos de ejemplo sirven de guía de configuración de SSH para esta comunicación.

1. Modifique el archivo `/etc/ssh/ssh_config`.

    Consulte la siguiente salida, donde se ha agregado la línea `MACs hmac-sha1`:

    ```output
    # RhostsRSAAuthentication no
    # RSAAuthentication yes
    # PasswordAuthentication yes
    # HostbasedAuthentication no
    # GSSAPIAuthentication no
    # GSSAPIDelegateCredentials no
    # GSSAPIKeyExchange no
    # GSSAPITrustDNS no
    # BatchMode no
    # CheckHostIP yes
    # AddressFamily any
    # ConnectTimeout 0
    # StrictHostKeyChecking ask
    # IdentityFile ~/.ssh/identity
    # IdentityFile ~/.ssh/id_rsa
    # IdentityFile ~/.ssh/id_dsa
    # Port 22
    Protocol 2
    # Cipher 3des
    # Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-
    cbc
    # MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd
    MACs hmac-sha
    # EscapeChar ~
    # Tunnel no
    # TunnelDevice any:any
    # PermitLocalCommand no
    # VisualHostKey no
    # ProxyCommand ssh -q -W %h:%p gateway.example.com
    ```

1. Cree un par de claves, privada y pública.

    Al usar el comando de ejemplo siguiente para generar el par de claves, no escriba una contraseña durante la creación de claves.

    ```bash
    ssh-keygen -t rsa –b 5120 -C ""
    ```

1. Envíe la clave pública a Microsoft Operations.

    Envíe la salida del comando `cat /root/.ssh/id_rsa.pub` (ejemplo siguiente) a Microsoft Operations para permitir que las herramientas de instantáneas se comuniquen con el subsistema de almacenamiento.

    ```bash
    cat /root/.ssh/id_rsa.pub
    ```

    ```output
    ssh-rsa
    AAAAB3NzaC1yc2EAAAADAQABAAABAQDoaRCgwn1Ll31NyDZy0UsOCKcc9nu2qdAPHdCzleiTWISvPW
    FzIFxz8iOaxpeTshH7GRonGs9HNtRkkz6mpK7pCGNJdxS4wJC9MZdXNt+JhuT23NajrTEnt1jXiVFH
    bh3jD7LjJGMb4GNvqeiBExyBDA2pXdlednOaE4dtiZ1N03Bc/J4TNuNhhQbdsIWZsqKt9OPUuTfD
    j0XvwUTLQbR4peGNfN1/cefcLxDlAgI+TmKdfgnLXIsSfbacXoTbqyBRwCi7p+bJnJD07zSc9YCZJa
    wKGAIilSg7s6Bq/2lAPDN1TqwIF8wQhAg2C7yeZHyE/ckaw/eQYuJtN+RNBD
    ```

## <a name="enable-communication-with-sap-hana"></a>Habilitación de la comunicación con SAP HANA

Las herramientas de instantáneas se comunican con SAP HANA y necesitan que un usuario con los permisos adecuados inicie y lance el punto de almacenamiento de la base de datos. En el ejemplo siguiente se muestra la configuración del usuario de SAP HANA v2 y `hdbuserstore` para la comunicación con la base de datos de SAP HANA.

Los siguientes comandos de ejemplo configuran un usuario (AZACSNAP) en SYSTEMDB de la base de datos
SAP HANA 2; cambie la dirección IP, los nombres de usuario y las contraseñas según corresponda:

1. Conéctese a SYSTEMDB para crear el usuario:

    ```bash
    hdbsql -n <IP_address_of_host>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD>
    ```

    ```output
    Welcome to the SAP HANA Database interactive terminal.

    Type: \h for help with commands
    \q to quit

    hdbsql SYSTEMDB=>
    ```

1. Cree el usuario:

    En este ejemplo se crea el usuario AZACSNAP en SYSTEMDB.

    ```sql
    hdbsql SYSTEMDB=> CREATE USER AZACSNAP PASSWORD <AZACSNAP_PASSWORD_CHANGE_ME> NO FORCE_FIRST_PASSWORD_CHANGE;
    ```

1. Conceda los permisos de usuario:

    En este ejemplo se establece el permiso para que el usuario AZACSNAP permita realizar una instantánea de almacenamiento coherente con la base de datos.

    ```sql
    hdbsql SYSTEMDB=> GRANT BACKUP ADMIN, CATALOG READ, MONITORING TO AZACSNAP;
    ```

1. *OPCIONAL*: evite que expire la contraseña del usuario.

    > [!NOTE]
    > Compruebe las directivas corporativas antes de efectuar este cambio.

   En este ejemplo se deshabilita la caducidad de la contraseña para el usuario AZACSNAP; sin este cambio, la contraseña del usuario expirará y se impedirá que las instantáneas se tomen correctamente.  

   ```sql
   hdbsql SYSTEMDB=> ALTER USER AZACSNAP DISABLE PASSWORD LIFETIME;
   ```

1. Configure Secure User Store (Almacén seguro para los usuarios) de SAP HANA (cambio de contraseña). En este ejemplo se usa el comando `hdbuserstore` desde el shell de Linux para configurar Secure User Store (Almacén seguro para los usuarios) de SAP HANA.

    ```bash
    hdbuserstore Set AZACSNAP <IP_address_of_host>:30013 AZACSNAP <AZACSNAP_PASSWORD_CHANGE_ME>
    ```

1. Compruebe que Secure User Store (Almacén de usuario seguro) esté configurado correctamente y use el comando `hdbuserstore` para mostrar una salida similar a la del ejemplo siguiente. Puede encontrar más información sobre el uso de `hdbuserstore` en el sitio web de SAP.

    ```bash
    hdbuserstore List
    ```

    ```output
    DATA FILE : /home/azacsnap/.hdb/sapprdhdb80/SSFS_HDB.DAT
    KEY FILE : /home/azacsnap/.hdb/sapprdhdb80/SSFS_HDB.KEY

    KEY AZACSNAP
    ENV : <IP_address_of_host>:
    USER: AZACSNAP
    ```

### <a name="using-ssl-for-communication-with-sap-hana"></a>Uso de SSL para la comunicación con SAP HANA

La herramienta `azacsnap` emplea el comando `hdbsql` de SAP HANA para comunicarse con SAP HANA. Esto incluye el uso de las opciones de SSL al cifrar la comunicación con SAP HANA. `azacsnap` usa la opción de SSL del comando `hdbsql` como se indica a continuación.

Con la opción `azacsnap --ssl` siempre se usa lo siguiente:

- `-e`: habilita el cifrado TLS/SSL. El servidor elige el mayor valor disponible.
- `-ssltrustcert`: especifica si se debe validar el certificado del servidor.
- `-sslhostnameincert "*"`: especifica el nombre de host usado para comprobar la identidad del servidor. Al especificar `"*"` como nombre de host, no se valida el nombre de host del servidor.

La comunicación SSL también requiere archivos de almacén de claves y de almacenamiento de confianza.  Aunque estos archivos se pueden almacenar en ubicaciones predeterminadas en una instalación de Linux, para garantizar que se utiliza el material de clave correcto para los distintos sistemas SAP HANA (es decir, en los casos en los que se usan diferentes archivos de almacén de claves y de almacenamiento de confianza para cada sistema SAP HANA), `azacsnap` espera que los archivos de almacén de claves y de almacenamiento de confianza se almacenen en la ubicación `securityPath`, tal y como se especifica en el archivo de configuración de `azacsnap`.

#### <a name="key-store-files"></a>Archivos del almacén de claves

- Si usa varios identificadores de seguridad con el mismo material de clave es más fácil crear vínculos a la ubicación securityPath, como se define en el archivo de configuración de `azacsnap`.  Asegúrese de que estos valores existen para todos los identificadores de seguridad mediante SSL.
  - Para openssl:
    - `ln $HOME/.ssl/key.pem <securityPath>/<SID>_keystore`
  - Para commoncrypto:
    - `ln $SECUDIR/sapcli.pse <securityPath>/<SID>_keystore`
- Si usa varios identificadores de seguridad con material de clave distinto para cada uno, copie los archivos (o mueva y cambie el nombre de estos) en la ubicación securityPath, según se define en el archivo de configuración de `azacsnap` de los identificadores de seguridad.
  - Para openssl:
    - `mv key.pem <securityPath>/<SID>_keystore`
  - Para commoncrypto:
    - `mv sapcli.pse <securityPath>/<SID>_keystore`

Cuando `azacsnap` llama a `hdbsql`, agrega `-sslkeystore=<securityPath>/<SID>_keystore` a la línea de comandos.

#### <a name="trust-store-files"></a>Archivos del almacén de confianza

- Si usa varios identificadores de seguridad con el mismo material de clave, cree vínculos rígidos a la ubicación securityPath como se define en el archivo de configuración de `azacsnap`.  Asegúrese de que estos valores existen para todos los identificadores de seguridad mediante SSL.
  - Para openssl:
    - `ln $HOME/.ssl/trust.pem <securityPath>/<SID>_truststore`
  - Para commoncrypto:
    - `ln $SECUDIR/sapcli.pse <securityPath>/<SID>_truststore`
- Si usa varios identificadores de seguridad con distinto material de clave para cada uno de ellos, copie los archivos (o muévalos y cámbieles el nombre) en la ubicación securityPath como se define en el archivo de configuración de `azacsnap` de los identificadores de seguridad.
  - Para openssl:
    - `mv trust.pem <securityPath>/<SID>_truststore`
  - Para commoncrypto:
    - `mv sapcli.pse <securityPath>/<SID>_truststore`

> [!NOTE]
> El componente `<SID>` de los nombres de archivo debe ser el identificador del sistema SAP HANA en mayúsculas (por ejemplo, `H80`, `PR1`, etc.).

Cuando `azacsnap` llama a `hdbsql`, agrega `-ssltruststore=<securityPath>/<SID>_truststore` a la línea de comandos.

Por lo tanto, la ejecución de `azacsnap -c test --test hana --ssl openssl` donde `SID` es `H80` en el archivo de configuración ejecutaría las conexiones de `hdbsql` como se indica a continuación:

```bash
hdbsql \
    -e \
    -ssltrustcert \
    -sslhostnameincert "*" \
    -sslprovider openssl \
    -sslkeystore ./security/H80_keystore \
    -ssltruststore ./security/H80_truststore
    "sql statement"
```

> [!NOTE]
> El carácter `\` es un ajuste de línea de la línea de comandos que mejora la claridad de los distintos parámetros que se pasan en la línea de comandos.

## <a name="installing-the-snapshot-tools"></a>Instalación de las herramientas de instantáneas

El instalador automático descargable está diseñado para facilitar la configuración y la ejecución de las herramientas de instantáneas con privilegios de usuario no raíz (por ejemplo, azacsnap). El instalador configurará el usuario y colocará las herramientas de instantáneas en el subdirectorio `$HOME/bin` de este (valor predeterminado = `/home/azacsnap/bin`).

El instalador automático intenta determinar la configuración correcta y las rutas de acceso de todos los archivos en función de la configuración del usuario que realiza la instalación (por ejemplo, raíz). Si los distintos pasos de configuración anteriores (habilitar la comunicación con el almacenamiento y SAP HANA) se ejecutan como raíz, durante la instalación se copian la clave privada y `hdbuserstore` en la ubicación del usuario de copia de seguridad. Sin embargo, un administrador capacitado puede realizar los pasos para permitir la comunicación con el back-end de almacenamiento y SAP HANA manualmente tras la instalación.

> [!NOTE]
> Para las versiones anteriores de SAP HANA en instalaciones de Azure Large Instance, el directorio de las herramientas de instantáneas instaladas previamente era `/hana/shared/<SID>/exe/linuxx86_64/hdb`.

Con los [pasos de los requisitos previos](#prerequisites-for-installation) completados se pueden instalar las herramientas de instantáneas con el instalador automático, como se indica a continuación:

1. Copie el instalador automático descargado en el sistema de destino.
1. Ejecútelo como usuario `root` (consulte el ejemplo siguiente). Si es necesario, haga que el archivo sea ejecutable con el comando `chmod +x *.run`.

Al ejecutar el comando del instalador automático sin ningún argumento, se mostrará ayuda sobre su uso para instalar las herramientas de instantáneas de la siguiente manera:

```bash
chmod +x azacsnap_installer_v5.0.run
./azacsnap_installer_v5.0.run
```

```output
Usage: ./azacsnap_installer_v5.0.run [-v] -I [-u <HLI Snapshot Command user>]
./azacsnap_installer_v5.0.run [-v] -X [-d <directory>]
./azacsnap_installer_v5.0.run [-h]

Switches enclosed in [] are optional for each command line.
- h prints out this usage.
- v turns on verbose output.
- I starts the installation.
- u is the Linux user to install the scripts into, by default this is
'azacsnap'.
- X will only extract the commands.
- d is the target directory to extract into, by default this is
'./snapshot_cmds'.
Examples of a target directory are ./tmp or /usr/local/bin
```

> [!NOTE]
> El instalador automático tiene una opción para extraer (-X) las herramientas de instantáneas del paquete sin creación ni configuración de usuarios. Esto permite a un administrador con experiencia completar los pasos de configuración manualmente o copiar los comandos para actualizar una instalación existente.

### <a name="easy-installation-of-snapshot-tools-default"></a>Instalación sencilla de herramientas de instantáneas (valor predeterminado)

El instalador se ha diseñado para instalar rápidamente las herramientas de instantáneas para SAP HANA en Azure. De forma predeterminada, si el instalador se ejecuta solo con la opción -I, realizará los pasos siguientes:

1. Creación del usuario de instantánea "azacsnap" y el directorio principal, y establecimiento de la pertenencia a grupos.
1. Configuración del inicio de sesión del usuario de azacsnap `~/.profile`.
1. Búsqueda en el sistema de archivos de los directorios que se van a agregar a `$PATH` de azacsnap, normalmente son las rutas de acceso a las herramientas de SAP HANA, como `hdbsql` y `hdbuserstore`.
1. Búsqueda en el sistema de archivos de los directorios que se van a agregar a `$LD_LIBRARY_PATH`de azacsnap. Muchos comandos requieren que se establezca una ruta de acceso de biblioteca para la correcta ejecución, así se configura para el usuario instalado.
1. Copia de las claves SSH para el almacenamiento de back-end de azacsnap desde el usuario "raíz" (el usuario que ejecuta la instalación). Se supone que el usuario "raíz" ya ha configurado la conectividad con el almacenamiento.
    - Consulte la sección "[Habilitación de la comunicación con el almacenamiento](#enable-communication-with-storage)".
1. Copie el almacén de conexión seguro para los usuarios de SAP HANA para el usuario de destino, azacsnap. Se supone que el usuario "raíz" ya ha configurado el almacén seguro para los usuarios; consulte la sección "Habilitación de la comunicación con SAP HANA".
1. Las herramientas de instantáneas se extraen en `/home/azacsnap/bin/`.
1. Los comandos de `/home/azacsnap/bin/` tienen establecidos los permisos (control y bit ejecutable, etc.).

En el ejemplo siguiente se muestra la salida correcta del instalador cuando se ejecuta con la opción de instalación predeterminada.

```bash
./azacsnap_installer_v5.0.run -I
```

```output
+-----------------------------------------------------------+
| Azure Application Consistent Snapshot tool Installer      |
+-----------------------------------------------------------+
|-> Installer version '5.0'
|-> Create Snapshot user 'azacsnap', home directory, and set group membership.
|-> Configure azacsnap .profile
|-> Search filesystem for directories to add to azacsnap's $PATH
|-> Search filesystem for directories to add to azacsnap's $LD_LIBRARY_PATH
|-> Copying SSH keys for back-end storage for azacsnap.
|-> Copying HANA connection keystore for azacsnap.
|-> Extracting commands into /home/azacsnap/bin/.
|-> Making commands in /home/azacsnap/bin/ executable.
|-> Creating symlink for hdbsql command in /home/azacsnap/bin/.
+-----------------------------------------------------------+
| Install complete! Follow the steps below to configure.    |
+-----------------------------------------------------------+
+-----------------------------------------------------------+
|  Install complete!  Follow the steps below to configure.  |
+-----------------------------------------------------------+

1. Change into the snapshot user account.....
     su - azacsnap
2. Set up the HANA Secure User Store..... (command format below)
     hdbuserstore Set <ADMIN_USER> <HOSTNAME>:<PORT> <admin_user> <password>
3. Change to location of commands.....
     cd /home/azacsnap/bin/
4. Configure the customer details file.....
     azacsnap -c configure --configuration new
5. Test the connection to storage.....
     azacsnap -c test --test storage
6. Test the connection to HANA.....
   a. without SSL
     azacsnap -c test --test hana
   b. with SSL,  you will need to choose the correct SSL option
     azacsnap -c test --test hana --ssl=<commoncrypto|openssl>
7. Run your first snapshot backup..... (example below)
     azacsnap -c backup --volume=data --prefix=hana_test --frequency=15min --retention=1
```

### <a name="uninstall-the-snapshot-tools"></a>Desinstalación de las herramientas de instantáneas

Si las herramientas de instantáneas se han instalado con la configuración predeterminada, para desinstalarlas solo se necesita eliminar el usuario para el que se han instalado los comandos (valor predeterminado = azacsnap).

```bash
userdel -f -r azacsnap
```

### <a name="manual-installation-of-the-snapshot-tools"></a>Instalación manual de las herramientas de instantáneas

En algunos casos es necesario instalar las herramientas manualmente, pero se recomienda usar la opción predeterminada del instalador para facilitar este proceso.

Cada línea que empieza por un carácter `#` demuestra que el usuario raíz ejecuta los comandos de ejemplo que siguen al carácter. `\` al final de una línea es el carácter de continuación de línea estándar de los comandos de shell.

Como superusuario raíz puede realizar una instalación manual con los siguientes pasos:

1. Obtención del identificador de grupo "sapsys", en este caso, el identificador de grupo = 1010.

    ```bash
    grep sapsys /etc/group
    ```

    ```output
    sapsys:x:1010:
    ```

1. Creación del usuario de instantánea "azacsnap" y el directorio principal, y establecimiento de la pertenencia a grupos con identificador de grupo del paso 1.

    ```bash
    useradd -m -g 1010 -c "Azure SAP HANA Snapshots User" azacsnap
    ```

1. Es necesario asegurarse de que `.profile` exista para el inicio de sesión de azacsnap.

    ```bash
    echo "" >> /home/azacsnap/.profile
    ```

1. Búsqueda en el sistema de archivos de los directorios que se van a agregar a $PATH de azacsnap, normalmente son las rutas de acceso a las herramientas de SAP HANA, como `hdbsql` y `hdbuserstore`.

    ```bash
    HDBSQL_PATH=`find -L /hana/shared/[A-z0-9][A-z0-9][A-z0-9]/HDB*/exe /usr/sap/hdbclient -name hdbsql -exec dirname {} + 2> /dev/null | sort | uniq | tr '\n' ':'`
    ```

1. Incorporación de la RUTA DE ACCESO actualizada al perfil del usuario.

    ```bash
    echo "export PATH=\"\$PATH:$HDBSQL_PATH\"" >> /home/azacsnap/.profile
    ```

1. Búsqueda en el sistema de archivos de los directorios que se van a agregar a $LD_LIBRARY_PATH de azacsnap.

    ```bash
    NEW_LIB_PATH=`find -L /hana/shared/[A-z0-9][A-z0-9][A-z0-9]/HDB*/exe /usr/sap/hdbclient -name "*.so" -exec dirname {} + 2> /dev/null | sort | uniq | tr '\n' ':'`
    ```

1. Incorporación de la ruta de biblioteca actualizada al perfil del usuario.

    ```bash
    echo "export LD_LIBRARY_PATH=\"\$LD_LIBRARY_PATH:$NEW_LIB_PATH\"" >> /home/azacsnap/.profile
    ```

1. En las instancias grandes de Azure:
    1. Copia de las claves SSH para el almacenamiento de back-end de azacsnap desde el usuario "raíz" (el usuario que ejecuta la instalación). Se supone que el usuario "raíz" ya ha configurado la conectividad con el almacenamiento.
       > Consulte la sección "[Habilitación de la comunicación con el almacenamiento](#enable-communication-with-storage)".

        ```bash
        cp -pr ~/.ssh /home/azacsnap/.
        ```

    1. Establecimiento de los permisos de usuario correctamente para los archivos SSH.

        ```bash
        chown -R azacsnap.sapsys /home/azacsnap/.ssh
        ```

1. En Azure NetApp Files:
    1. Configuración de la ruta de acceso de `DOTNET_BUNDLE_EXTRACT_BASE_DIR` del usuario según la guía de extracción de un solo archivo de .NET Core.
        1. SUSE Linux

            ```bash
            echo "export DOTNET_BUNDLE_EXTRACT_BASE_DIR=\$HOME/.net" >> /home/azacsnap/.profile
            echo "[ -d $DOTNET_BUNDLE_EXTRACT_BASE_DIR] && chmod 700 $DOTNET_BUNDLE_EXTRACT_BASE_DIR" >> /home/azacsnap/.profile
            ```

        1. RHEL

            ```bash
            echo "export DOTNET_BUNDLE_EXTRACT_BASE_DIR=\$HOME/.net" >> /home/azacsnap/.bash_profile
            echo "[ -d $DOTNET_BUNDLE_EXTRACT_BASE_DIR] && chmod 700 $DOTNET_BUNDLE_EXTRACT_BASE_DIR" >> /home/azacsnap/.bash_profile
            ```

1. Copie el almacén de conexión seguro para los usuarios de SAP HANA para el usuario de destino, azacsnap. Se supone que el usuario "raíz" ya ha configurado el almacén seguro para los usuarios.
    > Consulte la sección "[Habilitación de la comunicación con SAP HANA](#enable-communication-with-sap-hana)".

    ```bash
    cp -pr ~/.hdb /home/azacsnap/.
    ```

1. Establecimiento de los permisos de usuario correctamente para los archivos `hdbuserstore`.

    ```bash
    chown -R azacsnap.sapsys /home/azacsnap/.hdb
    ```

1. Extracción de las herramientas de instantáneas en/home/azacsnap/bin/.

    ```bash
    ./azacsnap_installer_v5.0.run -X -d /home/azacsnap/bin
    ```

1. Conversión de los comandos en ejecutables.

    ```bash
    chmod 700 /home/azacsnap/bin/*
    ```

1. Es necesario asegurarse de que los permisos de control correctos están establecidos en el directorio principal del usuario.

    ```bash
    chown -R azacsnap.sapsys /home/azacsnap/*
    ```

### <a name="complete-the-setup-of-snapshot-tools"></a>Finalización de la configuración de las herramientas de instantáneas

El instalador proporciona los pasos necesarios para completar la instalación de las herramientas de instantáneas.
Siga estos pasos para configurar y probar las herramientas de instantáneas.  Una vez correctas las pruebas, realice la primera instantánea de almacenamiento coherente con la base de datos.

En la siguiente salida se muestran los pasos que se deben realizar después de ejecutar el instalador con las opciones de instalación predeterminadas:

1. Cambio a la cuenta de usuario de instantánea.
    1. `su - azacsnap`
1. Configuración del almacén seguro para los usuarios de HANA.
   1. `hdbuserstore Set <ADMIN_USER> <HOSTNAME>:<PORT> <admin_user> <password>`
1. Cambio a la ubicación de los comandos.
   1. `cd /home/azacsnap/bin/`
1. Configuración del archivo de detalles del cliente.
   1. `azacsnap -c configure –-configuration new`
1. Prueba de la conexión al almacenamiento...
   1. `azacsnap -c test –-test storage`
1. Prueba de la conexión a HANA...
    1. Sin SSL:
       1. `azacsnap -c test –-test hana`
    1. Con SSL: elija la opción de SSL correcta.
       1. `azacsnap -c test –-test hana --ssl=<commoncrypto|openssl>`
1. Ejecución de la primera copia de seguridad de instantánea.
    1. `azacsnap -c backup –-volume data--prefix=hana_test --retention=1`

El paso 2 será necesario si no se realizó el paso "[Habilitación de la comunicación con SAP HANA](#enable-communication-with-sap-hana)" antes de la instalación.

> [!NOTE]
> Los comandos de prueba deben ejecutarse correctamente. De lo contrario, producirán un error.

## <a name="configuring-the-database"></a>Configuración de la base de datos

En esta sección se explica cómo configurar la base de datos.

### <a name="sap-hana-configuration"></a>Configuración de SAP HANA

Hay algunos cambios recomendados que se deben aplicar a SAP HANA para asegurar la protección de las copias de seguridad de los registros y el catálogo. De forma predeterminada, `basepath_logbackup` y `basepath_catalogbackup` generarán sus archivos en el directorio `$(DIR_INSTANCE)/backup/log` y es poco probable que esta ruta de acceso se encuentre en un volumen con `azacsnap` configurado para crear una instantánea de estos archivos que no se protejan con instantáneas de almacenamiento.

Los siguientes ejemplos de comandos de `hdbsql` están diseñados para mostrar la configuración de las rutas de acceso del catálogo y del registro en ubicaciones que se encuentran en volúmenes de almacenamiento donde `azacsnap` puede realizar instantáneas. Asegúrese de comprobar que los valores de la línea de comandos coinciden con la configuración de SAP HANA local.

### <a name="configure-log-backup-location"></a>Configuración de la ubicación de copia de seguridad de los registros

En este ejemplo, el cambio se realiza en el parámetro `basepath_logbackup`.

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'basepath_logbackup') = '/hana/logbackups/H80' WITH RECONFIGURE"
```

### <a name="configure-catalog-backup-location"></a>Configuración de la ubicación de copia de seguridad del catálogo

En este ejemplo, el cambio se realiza en el parámetro `basepath_catalogbackup`. En primer lugar, compruebe que la ruta de acceso de `basepath_catalogbackup` existe en el sistema de archivos, si no es el caso, creéla con el mismo control que el directorio.

```bash
ls -ld /hana/logbackups/H80/catalog
```

```output
drwxr-x--- 4 h80adm sapsys 4096 Jan 17 06:55 /hana/logbackups/H80/catalog
```

Si es necesario crear la ruta de acceso, en el ejemplo siguiente se crea la ruta de acceso y se establecen los permisos y el control correctos. Estos comandos tendrán que ejecutarse como raíz.

```bash
mkdir /hana/logbackups/H80/catalog
chown --reference=/hana/shared/H80/HDB00 /hana/logbackups/H80/catalog
chmod --reference=/hana/shared/H80/HDB00 /hana/logbackups/H80/catalog
ls -ld /hana/logbackups/H80/catalog
```

```output
drwxr-x--- 4 h80adm sapsys 4096 Jan 17 06:55 /hana/logbackups/H80/catalog
```

En el siguiente ejemplo se cambiará la configuración de SAP HANA.

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'basepath_catalogbackup') = '/hana/logbackups/H80/catalog' WITH RECONFIGURE"
```

### <a name="check-log-and-catalog-backup-locations"></a>Comprobación de la ubicación de las copias de seguridad del catálogo y del registro

Después de realizar los cambios anteriores, confirme que la configuración sea correcta con el siguiente comando.
En este ejemplo, la configuración que se ha establecido siguiendo las instrucciones anteriores se mostrará como configuración del SISTEMA.

> Esta consulta también devuelve la configuración PREDETERMINADA para la comparación.

```bash
hdbsql -jaxC -n <HANA_ip_address> - i 00 -U AZACSNAP "select * from sys.m_inifile_contents where (key = 'basepath_databackup' or key ='basepath_datavolumes' or key = 'basepath_logbackup' or key = 'basepath_logvolumes' or key = 'basepath_catalogbackup')"
```

```output
global.ini,DEFAULT,,,persistence,basepath_catalogbackup,$(DIR_INSTANCE)/backup/log
global.ini,DEFAULT,,,persistence,basepath_databackup,$(DIR_INSTANCE)/backup/data
global.ini,DEFAULT,,,persistence,basepath_datavolumes,$(DIR_GLOBAL)/hdb/data
global.ini,DEFAULT,,,persistence,basepath_logbackup,$(DIR_INSTANCE)/backup/log
global.ini,DEFAULT,,,persistence,basepath_logvolumes,$(DIR_GLOBAL)/hdb/log
global.ini,SYSTEM,,,persistence,basepath_catalogbackup,/hana/logbackups/H80/catalog
global.ini,SYSTEM,,,persistence,basepath_datavolumes,/hana/data/H80
global.ini,SYSTEM,,,persistence,basepath_logbackup,/hana/logbackups/H80
global.ini,SYSTEM,,,persistence,basepath_logvolumes,/hana/log/H80
```

### <a name="configure-log-backup-timeout"></a>Configuración del tiempo de espera de la copia de seguridad de registros

La configuración predeterminada de SAP HANA para realizar una copia de seguridad de registros es 900 segundos (15 minutos). Se recomienda reducir este valor a 300 segundos (es decir, 5 minutos).  Después se pueden ejecutar copias de seguridad periódicas (por ejemplo, cada 10 minutos) si se agrega el volumen log_backups a la sección OTHER del volumen del archivo de configuración.

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'log_backup_timeout_s') = '300' WITH RECONFIGURE"
```

#### <a name="check-log-backup-timeout"></a>Comprobación del tiempo de espera de la copia de seguridad de registros

Después de realizar el cambio en el tiempo de espera de la copia de seguridad de registros, asegúrese de que se ha establecido de la manera siguiente.
En este ejemplo, la configuración que se ha establecido se mostrará como configuración del SISTEMA, pero esta consulta también devolverá la configuración PREDETERMINADA para la comparación.

```bash
hdbsql -jaxC -n <HANA_ip_address> - i 00 -U AZACSNAP "select * from sys.m_inifile_contents where key like '%log_backup_timeout%' "
```

```output
global.ini,DEFAULT,,,persistence,log_backup_timeout_s,900
global.ini,SYSTEM,,,persistence,log_backup_timeout_s,300
```

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la herramienta Azure Application Consistent Snapshot](azacsnap-cmd-ref-configure.md)
