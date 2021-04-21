---
title: Implementación de la plataforma de inteligencia empresarial SAP BusinessObjects en Azure para Linux | Microsoft Docs
description: Implementación y configuración de la plataforma de inteligencia empresarial SAP BusinessObjects en Azure para Linux
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files,mysql
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 10/05/2020
ms.author: depadia
ms.openlocfilehash: b16a2d9f779232e59eb883f6a254be22990f5c78
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107520027"
---
# <a name="sap-businessobjects-bi-platform-deployment-guide-for-linux-on-azure"></a>Guía de implementación de la plataforma de inteligencia empresarial SAP BusinessObjects para Linux en Azure

En este artículo se describe la estrategia para implementar la plataforma SAP BOBI en Azure para Linux. En este ejemplo, se configuran dos máquinas virtuales con Managed Disks SSD Premium como directorio de instalación. Azure Database for MySQL se usa para la base de datos CMS y Azure NetApp Files para el servidor de repositorio de archivos se comparte entre ambos servidores. La aplicación web de Java de Tomcat predeterminada y la aplicación de plataforma de inteligencia empresarial se instalan juntas en ambas máquinas virtuales. Para equilibrar la carga de la solicitud de usuario, se usa Application Gateway, que tiene capacidades de descarga nativas de TLS/SSL.

Este tipo de arquitectura es eficaz para entornos pequeños de implementación o que no sean de producción. Para una implementación de producción o a gran escala, puede tener hosts independientes para la aplicación web, así como varios hosts de aplicaciones de BOBI que permitan al servidor procesar más información.

![Implementación de SAP BOBI en Azure para Linux](media/businessobjects-deployment-guide/businessobjects-deployment-linux.png)

En este ejemplo, se usa la versión del producto y el diseño del sistema de archivos siguientes:

- Plataforma de SAP BusinessObjects 4.3
- SUSE Linux Enterprise Server 12 SP5
- Azure Database for MySQL (Versión: 8.0.15)
- Conector de MySQL C API: libmysqlclient (Versión: 6.1.11)

| Sistema de archivos        | Descripción                                                                                                               | Tamaño (GB)             | Propietario  | Grupo  | Storage                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|-----------------------|--------|--------|----------------------------|
| /usr/sap           | Sistema de archivos para la instalación de la instancia de SAP BOBI, aplicación web de Tomcat predeterminada y controladores de base de datos (si es necesario). | Directrices de ajuste de tamaño de SAP | bl1adm | sapsys | Managed Disks Premium: SSD |
| /usr/sap/frsinput  | El directorio de montaje es para los archivos compartidos en todos los hosts de BOBI que se usarán como directorio de repositorios de archivos de entrada.  | Necesidad empresarial         | bl1adm | sapsys | Azure NetApp Files         |
| /usr/sap/frsoutput | El directorio de montaje es para los archivos compartidos en todos los hosts de BOBI que se usarán como directorio de repositorios de archivos de salida. | Necesidad empresarial         | bl1adm | sapsys | Azure NetApp Files         |

## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Implementación de máquinas virtuales Linux en Azure Portal

En esta sección, crearemos dos máquinas virtuales (VM) con la imagen del sistema operativo (SO) Linux para la plataforma SAP BOBI. A continuación, se indican los pasos de alto nivel para crear máquinas virtuales:

1. Cree un [grupo de recursos](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups).

2. Cree una [red virtual](../../../virtual-network/quick-create-portal.md#create-a-virtual-network).

   - No use una única subred para todos los servicios de Azure en la implementación de la plataforma de inteligencia empresarial SAP. En función de la arquitectura de la plataforma de inteligencia empresarial SAP, debe crear varias subredes. En esta implementación, crearemos tres subredes: subred de aplicación, subred de almacén de repositorio de archivos y subred de Application Gateway.
   - En Azure, Application Gateway y Azure NetApp Files deben estar siempre en una subred independiente. Consulte el artículo de [Azure Application Gateway](../../../application-gateway/configuration-overview.md) y [Directrices para el planeamiento de red de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-network-topologies.md) para obtener más detalles.

3. Cree un conjunto de disponibilidad.

   - Para lograr redundancia para cada nivel en la implementación de varias instancias, coloque las máquinas virtuales de cada nivel en un conjunto de disponibilidad. Asegúrese de separar los conjuntos de disponibilidad para cada nivel en función de su arquitectura.

4. Cree la máquina virtual 1 **(azusbosl1).**

   - Puede usar una imagen personalizada o elegir una imagen de Azure Marketplace. Consulte [Implementación de una máquina virtual desde Azure Marketplace para SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-1-deploying-a-vm-from-the-azure-marketplace-for-sap) o [Implementación de una máquina virtual con una imagen personalizada para SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-2-deploying-a-vm-with-a-custom-image-for-sap) en función de sus necesidades.

5. Cree la máquina virtual 2 **(azusbosl2).**
6. Agregue un disco SSD Premium. Se usará como directorio de instalación de SAP BOBI.

## <a name="provision-azure-netapp-files"></a>Aprovisionamiento de Azure NetApp Files

Antes de continuar con la configuración de Azure NetApp Files, familiarícese con la [documentación correspondiente](../../../azure-netapp-files/azure-netapp-files-introduction.md).

Azure NetApp Files está disponible en varias [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Compruebe si la región de Azure seleccionada ofrece Azure NetApp Files.

Para comprobar la disponibilidad de Azure NetApp Files por región, consulte [Disponibilidad de Azure NetApp Files por región de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).

Antes de implementar Azure NetApp Files, solicite la incorporación a este servicio de acuerdo con las [instrucciones para registrarse en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

### <a name="deploy-azure-netapp-files-resources"></a>Implementación de recursos de Azure NetApp Files

En las siguientes instrucciones se supone que ya ha implementado la [red virtual de Azure](../../../virtual-network/virtual-networks-overview.md). Los recursos de Azure NetApp Files y las VM en las que esos recursos se montarán se deben implementar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas.

1. Si aún no implementó los recursos, solicite la [incorporación a Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

2. Cree una cuenta de NetApp en la región de Azure seleccionada. Para ello, siga las instrucciones en [Creación de una cuenta de NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

3. Configure el grupo de capacidad de Azure NetApp Files según disponibles en [Configuración en un grupo de capacidad de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

   - La arquitectura de la plataforma de inteligencia empresarial SAP que se presenta en este artículo utiliza un único grupo de capacidad de Azure NetApp Files, el nivel de servicio *Premium*. En el caso del servidor de repositorio de archivos de inteligencia empresarial SAP en Azure, se recomienda usar un [nivel de servicio](../../../azure-netapp-files/azure-netapp-files-service-levels.md) *Premium* o *Ultra* de Azure NetApp Files.

4. Delegue una subred en Azure NetApp Files tal como se describe en las instrucciones de [Delegación de una subred en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).

5. Implemente los volúmenes de Azure NetApp Files según las instrucciones de [Creación de un volumen de NFS para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).

   El volumen de ANF se puede implementar como NFSv3 y NFSv4.1, ya que ambos protocolos son compatibles con la plataforma SAP BOBI. Implemente los volúmenes en la subred de Azure NetApp Files correspondiente. Las direcciones IP de los volúmenes de Azure NetApp se asignan automáticamente.

Tenga en cuenta que los recursos de Azure NetApp Files y las VM de Azure deben estar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas. Por ejemplo, azusbobi-frsinput, azusbobi-frsoutput son los nombres de los volúmenes y nfs://10.31.2.4/azusbobi-frsinput, nfs://10.31.2.4/azusbobi-frsoutput son las rutas de acceso de archivo para los volúmenes de Azure NetApp Files.

- Volumen azusbobi-frsinput (nfs://10.31.2.4/azusbobi-frsinput) azusbobi-frsinput (nfs://10.31.2.4/azusbobi-frsinput)
- Volumen azusbobi-frsoutput (nfs://10.31.2.4/azusbobi-frsoutput)

### <a name="important-considerations"></a>Consideraciones importantes

A medida que se crea la instancia de Azure NetApp Files para el servidor de repositorio de archivos de la plataforma SAP BOBI, tenga en cuenta lo siguiente:

1. El grupo de capacidad mínima es 4 tebibytes (TiB).
2. El tamaño del volumen mínimo es de 100 gigabytes (GiB).
3. Azure NetApp Files y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montarán se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](../../../virtual-network/virtual-network-peering-overview.md) de la misma región. Ahora también se admite el acceso de Azure NetApp Files mediante emparejamiento de red virtual en la misma región. Aún no se admite el acceso de Azure NetApp a través del emparejamiento global.
4. La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
5. Con la [directiva de exportación](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) de Azure NetApp Files, puede controlar los clientes permitidos y el tipo de acceso (lectura y escritura, solo lectura, etc.).
6. La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure.
7. Los volúmenes de Azure NetApp Files se pueden implementar como volúmenes NFSv3 o NFSv4.1. Ambos protocolos son compatibles con las aplicaciones de la plataforma de inteligencia empresarial SAP.

## <a name="configure-file-systems-on-linux-servers"></a>Configuración de sistemas de archivos en servidores Linux

En los pasos de esta sección se usan los siguientes prefijos:

**[A]** : El paso se aplica a todos los hosts.

### <a name="format-and-mount-sap-file-system"></a>Formato y montaje del sistema de archivos de SAP

1. **[A]** Enumerar todos los discos asociados.

    ```bash
    sudo lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda      8:0    0   30G  0 disk
    ├─sda1   8:1    0    2M  0 part
    ├─sda2   8:2    0  512M  0 part /boot/efi
    ├─sda3   8:3    0    1G  0 part /boot
    └─sda4   8:4    0 28.5G  0 part /
    sdb      8:16   0   32G  0 disk
    └─sdb1   8:17   0   32G  0 part /mnt
    sdc      8:32   0  128G  0 disk
    sr0     11:0    1  628K  0 rom  
    # Premium SSD of 128 GB is attached to Virtual Machine, whose device name is sdc
    ```

2. **[A]** Formatear el dispositivo de bloque para /usr/sap.

    ```bash
    sudo mkfs.xfs /dev/sdc
    ```

3. **[A]** Crear el directorio de montaje.

    ```bash
    sudo mkdir -p /usr/sap
    ```

4. **[A]** Obtener UUID del dispositivo de bloque.

    ```bash
    sudo blkid

    #It will display information about block device. Copy UUID of the formatted block device

    /dev/sdc: UUID="0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b" TYPE="xfs"
    ```

5. **[A]** Mantener la entrada de montaje del sistema de archivos en /etc/fstab.

    ```bash
    sudo echo "UUID=0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b /usr/sap xfs defaults,nofail 0 2" >> /etc/fstab
    ```

6. **[A]** Montar el sistema de archivos.

    ```bash
    sudo mount -a

    sudo df -h

    Filesystem                     Size  Used Avail Use% Mounted on
    devtmpfs                       7.9G  8.0K  7.9G   1% /dev
    tmpfs                          7.9G   82M  7.8G   2% /run
    tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
    /dev/sda4                       29G  1.8G   27G   6% /
    tmpfs                          1.6G     0  1.6G   0% /run/user/1000
    /dev/sda3                     1014M   87M  928M   9% /boot
    /dev/sda2                      512M  1.1M  511M   1% /boot/efi
    /dev/sdb1                       32G   49M   30G   1% /mnt
    /dev/sdc                       128G   29G  100G  23% /usr/sap
    ```

### <a name="mount-azure-netapp-files-volume"></a>Montaje del volumen de Azure NetApp Files

1. **[A]** Crear los directorios de montaje.

   ```bash
   sudo mkdir -p /usr/sap/frsinput
   sudo mkdir -p /usr/sap/frsoutput
   ```

2. **[A]** Configurar el sistema operativo cliente para que admita el montaje de NFSv4.1 **(solo es aplicable si se usa NFSv4.1)** .

   Si usa volúmenes de Azure NetApp Files con el protocolo NFSv4.1, ejecute la siguiente configuración en todas las máquinas virtuales, donde se deben montar volúmenes de Azure NetApp Files NFSv4.1.

   **Comprobación de la configuración del dominio NFS**

   Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **defaultv4iddomain.com**, y de que la asignación se haya establecido en **nobody**.

   ```bash
   sudo cat /etc/idmapd.conf
   # Example
   [General]
   Domain = defaultv4iddomain.com
   [Mapping]
   Nobody-User = nobody
   Nobody-Group = nobody
   ```

   > [!Important]
   >
   > Asegúrese de establecer el dominio NFS en /etc/idmapd.conf en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **defaultv4iddomain.com**. Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como "nobody".

   Comprobar `nfs4_disable_idmapping` Debe establecerse en **S**. Para crear la estructura de directorio en la que se encuentra `nfs4_disable_idmapping`, ejecute el comando mount. No podrá crear manualmente el directorio en /sys/modules, ya que el acceso está reservado para el kernel o los controladores.

   ```bash
   # Check nfs4_disable_idmapping
   cat /sys/module/nfs/parameters/nfs4_disable_idmapping

   # If you need to set nfs4_disable_idmapping to Y
   mkdir /mnt/tmp
   mount -t nfs -o sec=sys,vers=4.1 10.31.2.4:/azusbobi-frsinput /mnt/tmp
   umount /mnt/tmp

   echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping

   # Make the configuration permanent
   echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
   ```

3. **[A]** Adición de entradas de montaje

   Si usa NFSv3:

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   ```

   Si usa NFSv4.1:

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   ```

4. **[A]** Montar volúmenes de NFS.

   ```bash
   sudo mount -a

   sudo df -h

   Filesystem                     Size  Used Avail Use% Mounted on
   devtmpfs                       7.9G  8.0K  7.9G   1% /dev
   tmpfs                          7.9G   82M  7.8G   2% /run
   tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
   /dev/sda4                       29G  1.8G   27G   6% /
   tmpfs                          1.6G     0  1.6G   0% /run/user/1000
   /dev/sda3                     1014M   87M  928M   9% /boot
   /dev/sda2                      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1                       32G   49M   30G   1% /mnt
   /dev/sdc                       128G   29G  100G  23% /usr/sap
   10.31.2.4:/azusbobi-frsinput   101T   18G  100T   1% /usr/sap/frsinput
   10.31.2.4:/azusbobi-frsoutput  100T  512K  100T   1% /usr/sap/frsoutput
   ```

## <a name="configure-cms-database---azure-database-for-mysql"></a>Configuración de la base de datos CMS: Azure Database for MySQL

En esta sección se proporcionan detalles sobre cómo aprovisionar Azure Database for MySQL mediante Azure Portal. También se proporcionan instrucciones sobre cómo crear bases de datos CMS y de auditoría para la plataforma SAP BOBI y una cuenta de usuario para tener acceso a la base de datos.

Las instrucciones solo se aplican si usa Azure DB for MySQL. En el caso de otras bases de datos, consulte la documentación específica de SAP o de la base de datos para obtener instrucciones.

### <a name="create-an-azure-database-for-mysql"></a>Creación de una instancia de Azure Database for MySQL

Inicie sesión en Azure Portal y siga los pasos que se mencionan en esta [Guía de inicio rápido de Azure Database for MySQL](../../../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Durante el aprovisionamiento Azure Database for MySQL hay unos puntos que se deben tener en cuenta:

1. Seleccione la misma región para Azure Database for MySQL donde se ejecutan los servidores de aplicaciones de la plataforma de inteligencia empresarial SAP.

2. Elija una versión de base de datos compatible en función de la [matriz de disponibilidad de productos (PAM) para SAP BI](https://support.sap.com/pam) específica de la versión de SAP BOBI. Siga las mismas instrucciones de compatibilidad que se abordan para MySQL AB en SAP PAM.

3. En "Proceso y almacenamiento", seleccione **Configurar servidor** y seleccione el plan de tarifa adecuado en función del tamaño de la salida.

4. El **crecimiento automático de almacenamiento** está habilitado de manera predeterminada. Tenga en cuenta que el [almacenamiento](../../../mysql/concepts-pricing-tiers.md#storage) solo se puede escalar verticalmente, no reducir.

5. De manera predeterminada, el **período de retención de la copia de seguridad** es de siete días, pero se puede [configurar opcionalmente](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) hasta 35 días.

6. Las copias de seguridad de Azure Database for MySQL son redundantes localmente de manera predeterminada, por lo que si quiere realizar copias de seguridad del servidor en el almacenamiento con redundancia geográfica, seleccione **Redundancia geográfica** en las **opciones de redundancia de copia de seguridad**.

> [!NOTE]
> No se admite el cambio de las [opciones de redundancia de copia de seguridad](../../../mysql/concepts-backup.md#backup-redundancy-options) después de la creación del servidor.

### <a name="configure-connection-security"></a>Configuración de la seguridad de conexión

De manera predeterminada, el servidor creado está protegido con un firewall y no se puede acceder a él públicamente. Para proporcionar acceso a la red virtual en la que se ejecutan los servidores de aplicaciones de la plataforma de inteligencia empresarial SAP, siga estos pasos:  

1. Vaya a los recursos de servidor en Azure Portal y seleccione **Seguridad de la conexión** en el menú izquierdo del recurso de servidor.
2. Seleccione **Sí** en **Permitir el acceso a servicios de Azure**.
3. En las reglas de red virtual, seleccione **Agregar red virtual existente**. Seleccione la red virtual y la subred del servidor de aplicaciones de la plataforma de inteligencia empresarial SAP. También debe proporcionar acceso al cuadro de salto u otros servidores desde donde puede conectar [MySQL Workbench](../../../mysql/connect-workbench.md) a Azure Database for MySQL. MySQL Workbench se usará para crear la base de datos CMS y de auditoría.
4. Una vez agregadas las redes virtuales, seleccione **Guardar**.

### <a name="create-cms-and-audit-database"></a>Creación de la base de datos CMS y de auditoría

1. Descargue e instale MySQL Workbench desde el [sitio web de MySQL](https://dev.mysql.com/downloads/workbench/). Asegúrese de instalar MySQL Workbench en el servidor que puede tener acceso a Azure Database for MySQL.

2. Conéctese al servidor con MySQL Workbench. Para ello, siga las instrucciones que se mencionan en este [artículo](../../../mysql/connect-workbench.md#get-connection-information). Si la prueba de la conexión es correcta, aparece el mensaje siguiente:

   ![Conexión de SQL Workbench](media/businessobjects-deployment-guide/businessobjects-sql-workbench.png)

3. En la pestaña Consulta SQL, ejecute la consulta siguiente para crear el esquema para la base de datos CMS y de auditoría.

   ```sql
   # Here cmsbl1 is the database name of CMS database. You can provide the name you want for CMS database.
   CREATE SCHEMA `cmsbl1` DEFAULT CHARACTER SET utf8;

   # auditbl1 is the database name of Audit database. You can provide the name you want for CMS database.
   CREATE SCHEMA `auditbl1` DEFAULT CHARACTER SET utf8;
   ```
   
4. Cree una cuenta de usuario para conectarse al esquema.

   ```sql
   # Create a user that can connect from any host, use the '%' wildcard as a host part
   CREATE USER 'cmsadmin'@'%' IDENTIFIED BY 'password';
   CREATE USER 'auditadmin'@'%' IDENTIFIED BY 'password';

   # Grant all privileges to a user account over a specific database:
   GRANT ALL PRIVILEGES ON cmsbl1.* TO 'cmsadmin'@'%' WITH GRANT OPTION;
   GRANT ALL PRIVILEGES ON auditbl1.* TO 'auditadmin'@'%' WITH GRANT OPTION;

   # Following any updates to the user privileges, be sure to save the changes by issuing the FLUSH PRIVILEGES
   FLUSH PRIVILEGES;
   ```

5. Para comprobar los privilegios y roles de la cuenta de usuario de MySQL:

   ```sql
   USE sys;
   SHOW GRANTS for 'cmsadmin'@'%';
   +------------------------------------------------------------------------+
   | Grants for cmsadmin@%                                                  |
   +------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `cmsadmin`@`%`                                   |
   | GRANT ALL PRIVILEGES ON `cmsbl1`.* TO `cmsadmin`@`%` WITH GRANT OPTION |
   +------------------------------------------------------------------------+

   USE sys;
   SHOW GRANTS FOR 'auditadmin'@'%';
   +----------------------------------------------------------------------------+
   | Grants for auditadmin@%                                                    |
   +----------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `auditadmin`@`%`                                     |
   | GRANT ALL PRIVILEGES ON `auditbl1`.* TO `auditadmin`@`%` WITH GRANT OPTION |
   +----------------------------------------------------------------------------+
   ```

### <a name="install-mysql-c-api-connector-libmysqlclient-on-linux-server"></a>Instalación del conector de MySQL C API (libmysqlclient) en el servidor Linux

Para que el servidor de aplicaciones de SAP BOBI tenga acceso a la base de datos, requiere cliente o controladores de base de datos. El conector de MySQL C API para Linux debe usarse para tener acceso a las bases de datos CMS y de auditoría. No se admite la conexión ODBC con la base de datos CMS. En esta sección se proporcionan instrucciones sobre cómo configurar el conector de MySQL C API en Linux.

1. Consulte el artículo [Herramientas de administración y controladores de MySQL compatibles con Azure Database for MySQL](../../../mysql/concepts-compatibility.md), en el que se describen los controladores compatibles con Azure Database for MySQL. Busque el controlador **Conector/C MySQL (libmysqlclient)** en el artículo.

2. Consulte este [vínculo](https://downloads.mysql.com/archives/c-c/) para descargar los controladores.

3. Seleccione el sistema operativo y descargue el paquete RPM del componente compartido del conector MySQL. En este ejemplo, se usa la versión del conector mysql-connector-c-shared-6.1.11.

4. Instale el conector en todas las instancias de la aplicación de SAP BOBI.

   ```bash
   # Install rpm package
   SLES: sudo zypper install <package>.rpm
   RHEL: sudo yum install <package>.rpm
   ```

5. Compruebe la ruta de acceso de libmysqlclient.so.

   ```bash
   # Find the location of libmysqlclient.so file
   whereis libmysqlclient

   # sample output
   libmysqlclient: /usr/lib64/libmysqlclient.so
   ```

6. Establezca LD_LIBRARY_PATH para que apunte al directorio `/usr/lib64` de la cuenta de usuario que se utilizará para la instalación.

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LD_LIBRARY_PATH=/usr/lib64
   ```

## <a name="server-preparation"></a>Preparación del servidor

En los pasos de esta sección se usan los siguientes prefijos:

**[A]** : El paso se aplica a todos los hosts.

1. **[A]** En función del tipo de Linux (SLES o RHEL), debe establecer los parámetros de kernel e instalar las bibliotecas necesarias. Consulte la sección **Requisitos del sistema** del [Manual de instalación de la plataforma de Business Intelligence para UNIX](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US).

2. **[A]** Asegúrese de que la zona horaria de la máquina está configurada correctamente. Consulte la [sección Requisitos adicionales de UNIX y Linux](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US/46b143336e041014910aba7db0e91070.html) del manual de instalación.

3. **[A]** Cree una cuenta de usuario (**bl1** adm) y un grupo (sapsys) en el que se puedan ejecutar los procesos en segundo plano del software. Use esta cuenta para ejecutar la instalación y el software. La cuenta no requiere privilegios de raíz.

4. **[A]** Establezca el entorno de cuenta de usuario (**bl1** adm) para usar una configuración regional UTF-8 compatible y asegúrese de que el software de la consola admite juegos de caracteres UTF-8. Para asegurarse de que el sistema operativo usa la configuración regional correcta, establezca las variables de entorno LC_ALL y LANG en su configuración regional preferida en el entorno de usuario (**bl1** adm).

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LANG=en_US.utf8
   export LC_ALL=en_US.utf8
   ```

5. **[A]** Configure la cuenta de usuario (**bl1** adm).

   ```bash
   # Set ulimit for bl1adm to unlimited
   root@azusbosl1:~> ulimit -f unlimited bl1adm
   root@azusbosl1:~> ulimit -u unlimited bl1adm

   root@azusbosl1:~> su - bl1adm
   bl1adm@azusbosl1:~> ulimit -a

   core file size          (blocks, -c) unlimited
   data seg size           (kbytes, -d) unlimited
   scheduling priority             (-e) 0
   file size               (blocks, -f) unlimited
   pending signals                 (-i) 63936
   max locked memory       (kbytes, -l) 64
   max memory size         (kbytes, -m) unlimited
   open files                      (-n) 1024
   pipe size            (512 bytes, -p) 8
   POSIX message queues     (bytes, -q) 819200
   real-time priority              (-r) 0
   stack size              (kbytes, -s) 8192
   cpu time               (seconds, -t) unlimited
   max user processes              (-u) unlimited
   virtual memory          (kbytes, -v) unlimited
   file locks                      (-x) unlimited
   ```

6. Descargue y extraiga medios para la plataforma de inteligencia empresarial SAP BusinessObjects desde el marketplace de servicios de SAP.

## <a name="installation"></a>Instalación

Compruebe la configuración regional de la cuenta de usuario **bl1** adm en el servidor.

```bash
bl1adm@azusbosl1:~> locale
LANG=en_US.utf8
LC_ALL=en_US.utf8
```

Vaya a medios de la plataforma de inteligencia empresarial SAP BusinessObjects y ejecute el comando siguiente con el usuario **bl1** adm:

```bash
./setup.sh -InstallDir /usr/sap/BL1
```

Siga el manual de instalación de la [plataforma SAP BOBI](https://help.sap.com/viewer/product/SAP_BUSINESSOBJECTS_BUSINESS_INTELLIGENCE_PLATFORM) de UNIX específico de su versión. Hay algunos puntos a tener en cuenta durante la instalación de la plataforma SAP BOBI.

- En la pantalla **Configurar el registro del producto**, puede usar una clave de licencia temporal para las soluciones de SAP BusinessObjects de la nota de SAP [1288121](https://launchpad.support.sap.com/#/notes/1288121) o puede generar la clave de licencia en el marketplace de servicios de SAP.

- En la pantalla **Seleccionar tipo de instalación**, seleccione instalación **completa** en el primer servidor (azusbosl1). Para el resto del servidor (azusbosl2), seleccione la opción **Personalizar/Expandir**, que expandirá el programa de configuración de BOBI existente.

- En **Seleccionar base de datos predeterminada o existente**, seleccione **Configurar una base de datos existente**, que le solicitará que seleccione base de datos CMS y de auditoría. Seleccione **MySQL** para el tipo de base de datos CMS y el tipo de base de datos de auditoría.

  También puede seleccionar Ninguna base de datos de auditoría si no quiere configurar la auditoría durante la instalación.

- Seleccione las opciones adecuadas en la **pantalla Seleccionar servidor de aplicaciones web de Java** según la arquitectura de SAP BOBI. En este ejemplo, se ha seleccionado la opción 1, que instala el servidor de Tomcat en la misma plataforma SAP BOBI.

- Escriba la información de la base de datos CMS en **Configurar la base de datos del repositorio de CMS: MySQL**. Entrada de ejemplo para la información de la base de datos CMS para la instalación de Linux. Azure Database for MySQL se usa en el puerto 3306 predeterminado.
  
  ![Implementación de SAP BOBI en Linux: base de datos CMS](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cms.png)

- (Opcional) Escriba la información de la base de datos de auditoría en **Configurar la base de datos del repositorio de auditoría: MySQL**. Entrada de ejemplo para la información de la base de datos de auditoría para la instalación de Linux.

  ![Implementación de SAP BOBI en Linux: base de datos de auditoría](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-audit.png)

- Siga las instrucciones e introduzca las entradas necesarias para completar la instalación.

Para la implementación de varias instancias, ejecute la configuración de instalación en el segundo host (azusbosl2). En la pantalla **Seleccionar tipo de instalación**, seleccione **Personalizar/Expandir**, opción que expandirá la configuración de BOBI existente.

En la oferta de Azure Database for MySQL, se usa una puerta de enlace para redirigir las conexiones a las instancias de servidor. Una vez establecida la conexión, el cliente de MySQL muestra la versión de MySQL establecida en la puerta de enlace, no la versión real que se ejecuta en la instancia del servidor MySQL. Para determinar la versión de la instancia del servidor MySQL, use el comando `SELECT VERSION();` en el símbolo del sistema de MySQL. Por lo tanto, en la consola de administración central (CMC), encontrará una versión de base de datos diferente, que es básicamente la versión establecida en la puerta de enlace. Consulte [Versiones admitidas de servidores de Azure Database for MySQL](../../../mysql/concepts-supported-versions.md) para obtener más detalles.

![Implementación de SAP BOBI en Linux: configuración de CMC](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cmc.png)

```sql
# Run direct query to the database using MySQL Workbench

select version();

+-----------+
| version() |
+-----------+
| 8.0.15    |
+-----------+
```

## <a name="post-installation"></a>Después de la instalación

### <a name="tomcat-clustering---session-replication"></a>Agrupación en clústeres de Tomcat: replicación de sesión

Tomcat admite la agrupación en clústeres de dos o más servidores de aplicaciones para la replicación de sesión y la conmutación por error. Las sesiones de la plataforma SAP BOBI están serializadas, una sesión de usuario puede conmutar por error a otra instancia de Tomcat, incluso cuando se produce un error en un servidor de aplicaciones.

Por ejemplo, si un usuario está conectado a un servidor web que produce un error mientras el usuario navega por una jerarquía de carpetas en la aplicación de inteligencia empresarial SAP. Con un clúster configurado correctamente, el usuario puede seguir navegando por la jerarquía de carpetas sin que se redirija a la página de inicio de sesión.

En la nota de SAP [2808640](https://launchpad.support.sap.com/#/notes/2808640), se proporcionan los pasos para configurar la agrupación en clústeres de Tomcat mediante multidifusión. Sin embargo, en Azure no se admite la multidifusión. Para que el clúster de Tomcat funcione en Azure, debe usar [StaticMembershipInterceptor](https://tomcat.apache.org/tomcat-8.0-doc/config/cluster-interceptor.html#Static_Membership) (nota de SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)). Consulte [Agrupación en clústeres de Tomcat mediante pertenencia estática para la plataforma de inteligencia empresarial SAP BusinessObjects](https://blogs.sap.com/2020/09/04/sap-on-azure-tomcat-clustering-using-static-membership-for-sap-businessobjects-bi-platform/) en el blog de SAP para configurar el clúster de Tomcat en Azure.

### <a name="load-balancing-web-tier-of-sap-bi-platform"></a>Nivel web de equilibrio de carga de la plataforma de inteligencia empresarial SAP

En la implementación de varias instancias de SAP BOBI, los servidores de aplicaciones web de Java (nivel web) se ejecutan en dos o más hosts. Para distribuir la carga de usuarios uniformemente entre los servidores web, puede usar un equilibrador de carga entre los usuarios finales y los servidores web. En Azure, puede usar Azure Load Balancer o Azure Application Gateway para administrar el tráfico a los servidores de aplicaciones web. En la sección siguiente se explican los detalles de cada oferta.

#### <a name="azure-load-balancer-network-based-load-balancer"></a>Azure Load Balancer (equilibrador de carga basado en red)

[Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) es un equilibrador de carga de nivel 4 (TCP, UDP) de alto rendimiento y baja latencia que distribuye el tráfico entre máquinas virtuales correctas. Un sondeo de estado de equilibrador de carga supervisa un puerto determinado en cada máquina virtual y solo distribuye tráfico a máquinas virtuales operativas. Puede elegir un equilibrador de carga público o un equilibrador de carga interno, en función de si quiere que la plataforma de inteligencia empresarial SAP sea accesible desde Internet o no. Tiene redundancia de zona, lo que garantiza una alta disponibilidad en las instancias de Availability Zones.

Consulte la sección Equilibrador de carga interno en la ilustración siguiente, donde el servidor de aplicaciones web se ejecuta en el puerto 8080, el puerto HTTP predeterminado de Tomcat, que lo supervisará el sondeo de estado. Por lo tanto, cualquier solicitud entrante que proceda de usuarios finales se redirigirá a los servidores de aplicaciones web (azusbosl1 o azusbosl2) en el grupo de back-end. El equilibrador de carga no admite la terminación TLS/SSL (también conocida como descarga TLS/SSL). Si usa Azure Load Balancer para distribuir el tráfico entre servidores web, se recomienda usar Standard Load Balancer.

> [!NOTE]
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](high-availability-guide-standard-load-balancer-outbound-connections.md).

![Azure Load Balancer para equilibrar el tráfico entre servidores web](media/businessobjects-deployment-guide/businessobjects-deployment-load-balancer.png)

#### <a name="azure-application-gateway-web-application-load-balancer"></a>Azure Application Gateway (equilibrador de carga de aplicaciones web)

[Azure Application Gateway (AGW)](../../../application-gateway/overview.md) proporciona un controlador de entrega de aplicaciones (ADC) como servicio, que se usa para ayudar a la aplicación a dirigir el tráfico de usuario a uno o varios servidores de aplicaciones web. Ofrece diversas funcionalidades de equilibrio de carga de capa 7, como la descarga SSL/TLS, firewall de aplicaciones web (WAF), afinidad de sesión basada en cookies y otras, para las aplicaciones.

En la plataforma de inteligencia empresarial SAP, Application Gateway dirige el tráfico web de la aplicación a los recursos especificados en un grupo de back-end: azusbosl1 o azusbos2. Se asignan escuchas a los puertos, se crean reglas y se agregan recursos a un grupo de back-end. En la ilustración siguiente, la puerta de enlace de aplicación con una dirección IP (10.31.3.20) de front-end privada actúa como punto de entrada para los usuarios, controla las conexiones de TLS/SSL (HTTPS: TCP/443) entrantes, descifra TLS/SSL y pasa la solicitud no cifrada (HTTP: TCP/8080) a los servidores del grupo de back-end. Con la característica de terminación TLS/SSL integrada, solo es necesario mantener un certificado TLS/SSL en la puerta de enlace de aplicación, lo que simplifica las operaciones.

![Application Gateway para equilibrar el tráfico entre servidores web](media/businessobjects-deployment-guide/businessobjects-deployment-application-gateway.png)

Para configurar Application Gateway para el servidor web de SAP BOBI, puede consultar [Servidores web de SAP BOBI de equilibrio de carga mediante Azure Application Gateway](https://blogs.sap.com/2020/09/17/sap-on-azure-load-balancing-web-application-servers-for-sap-bobi-using-azure-application-gateway/) en el blog de SAP.

> [!NOTE]
> Se recomienda usar Azure Application Gateway para equilibrar la carga del tráfico en el servidor web, ya que proporciona características parecidas, como la descarga SSL, la centralización de la administración de SSL para reducir la sobrecarga de cifrado y descifrado en el servidor, el algoritmo Round-Robin para distribuir el tráfico, las capacidades de Web Application Firewall (WAF), la alta disponibilidad, etc.

### <a name="sap-businessobjects-bi-platform---back-up-and-restore"></a>Plataforma de inteligencia empresarial SAP BusinessObjects: copia de seguridad y restauración

La copia de seguridad y restauración es un proceso de creación de copias periódicas de datos y aplicaciones en una ubicación independiente. Por lo tanto, se puede restaurar o recuperar al estado anterior si se pierden o dañan los datos o las aplicaciones originales. También es un componente esencial de cualquier estrategia de recuperación ante desastres empresarial.

Para desarrollar una estrategia completa de copia de seguridad y restauración para la plataforma SAP BOBI, identifique los componentes que provocan el tiempo de inactividad del sistema o la interrupción de la aplicación. En la plataforma SAP BOBI, la copia de seguridad de los componentes siguientes es fundamental para proteger la aplicación.

- Directorio de instalación de SAP BOBI (Managed Disks Premium)
- Servidor de repositorio de archivos (Azure NetApp Files o archivos Premium de Azure)
- Base de datos CMS (Azure Database for MySQL o base de datos en Azure VM)

En la siguiente sección se describe cómo implementar una estrategia de copia de seguridad y restauración para cada componente en la plataforma SAP BOBI.

#### <a name="backup--restore-for-sap-bobi-installation-directory"></a>Copia de seguridad y restauración para el directorio de instalación de SAP BOBI

En Azure, la manera más sencilla de realizar copias de seguridad de los servidores de aplicaciones y de todos los discos conectados es mediante el servicio de [Azure Backup](../../../backup/backup-overview.md). Proporciona copias de seguridad independientes y aisladas para impedir la destrucción accidental de los datos en las máquinas virtuales. Las copias de seguridad se almacenan en un almacén de Recovery Services con administración integrada de puntos de recuperación. La configuración y la escalabilidad son sencillas, las copias de seguridad están optimizadas y puede restaurarlas fácilmente cuando sea necesario.

Como parte del proceso de copia de seguridad, se realiza una instantánea y los datos se transfieren al almacén de Recovery Services sin que ello afecte a las cargas de trabajo de producción. La instantánea proporciona diferentes niveles de coherencia, como se describe en el artículo [Coherencia de instantáneas](../../../backup/backup-azure-vms-introduction.md#snapshot-consistency). También puede optar por realizar una copia de seguridad de un subconjunto de los discos de datos de una máquina virtual mediante el uso de la funcionalidad de copia de seguridad y restauración de discos selectivos. Para obtener más información, consulte el documento [Copia de seguridad de máquinas virtuales de Azure](../../../backup/backup-azure-vms-introduction.md) y [Preguntas más frecuentes sobre la copia de seguridad de máquinas virtuales de Azure](../../../backup/backup-azure-vm-backup-faq.yml).

#### <a name="backup--restore-for-file-repository-server"></a>Copia de seguridad y restauración del servidor de repositorio de archivos

Para **Azure NetApp Files**, puede crear instantáneas a petición y programar la creación automática de instantáneas mediante el uso de directivas de instantáneas. Las copias de instantáneas proporcionan una copia de un momento dado del volumen de ANF. Para obtener más información, consulte [Administración de instantáneas mediante Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-manage-snapshots.md).

La copia de seguridad de **Azure Files** se integra con el servicio nativo de [Azure Backup](../../../backup/backup-overview.md), que centraliza la función de copia de seguridad y restauración junto con la copia de seguridad de las máquinas virtuales y simplifica el trabajo de las operaciones. Para obtener más información, vea [Copia de seguridad de recursos compartidos de archivos de Azure](../../../backup/azure-file-share-backup-overview.md) y [Preguntas acerca de la copia de seguridad de archivos de Azure](../../../backup/backup-azure-files-faq.yml).

#### <a name="backup--restore-for-cms-database"></a>Copia de seguridad y restauración de la base de datos CMS

Azure Database for MySQL es una oferta de DBaaS de Azure que crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Azure Database for MySQL realiza copias de seguridad de los archivos de datos y del registro de transacciones. En función del tamaño de almacenamiento máximo admitido, se realizan copias de seguridad completas y diferenciales (servidores de almacenamiento de hasta 4 TB) o copias de seguridad de instantáneas (servidores de almacenamiento de hasta 16 TB). Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. De manera predeterminada, el período de retención de la copia de seguridad es de siete días, pero se puede [configurar opcionalmente](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) hasta tres días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

Estos archivos de copia de seguridad no están expuestos al usuario y no se pueden exportar. Estas copias de seguridad solo se pueden usar en operaciones de restauración de Azure Database for MySQL. Puede usar [mysqldump](../../../mysql/concepts-migrate-dump-restore.md) para copiar una base de datos. Para más información, consulte [Copia de seguridad y restauración en Azure Database for MySQL](../../../mysql/concepts-backup.md).

Para la base de datos instalada en Virtual Machines, puede usar las herramientas de copia de seguridad estándar o [Azure Backup](../../../backup/sap-hana-db-about.md) para la base de datos de HANA. Además, si los servicios y las herramientas de Azure no satisfacen sus necesidades, puede usar otras herramientas de copia de seguridad o scripts para crear copias de seguridad de discos.

## <a name="sap-businessobjects-bi-platform-reliability"></a>Confiabilidad de la plataforma de inteligencia empresarial SAP BusinessObjects

La plataforma de inteligencia empresarial SAP BusinessObjects incluye diferentes niveles, que están optimizados para operaciones y tareas específicas. Cuando un componente de cualquier nivel no esté disponible, la aplicación SAP BOBI dejará de estar accesible o no funcionará ninguna de sus funcionalidades. Por lo tanto, debe asegurarse de que cada nivel esté diseñado para ser confiable y mantener la aplicación operativa sin interrupción empresarial.

Esta sección se centra en las siguientes opciones de la plataforma SAP BOBI:

- **Alta disponibilidad:** Una plataforma de alta disponibilidad tiene al menos dos de todo en la región de Azure para mantener la aplicación operativa si uno de los servidores deja de estar disponible.
- **Recuperación ante desastres:** Se trata de un proceso de restauración de la funcionalidad de la aplicación si hay alguna pérdida catastrófica, como cuando toda la región de Azure deja de estar disponible debido a algún desastre natural.

La implementación de esta solución varía en función de la naturaleza de la configuración del sistema en Azure. Por lo tanto, el cliente debe adaptar la solución de alta disponibilidad y recuperación ante desastres en función de sus necesidades empresariales.

### <a name="high-availability"></a>Alta disponibilidad

Alta disponibilidad se refiere a un conjunto de tecnologías que minimiza las interrupciones de TI al proporcionar una continuidad empresarial de los servicios o la aplicación mediante componentes redundantes, con tolerancia de errores o protegidos mediante conmutación por error dentro del mismo centro de datos. En nuestro caso, el centro de datos se encuentra en una región de Azure. En el artículo [Escenarios y arquitectura de alta disponibilidad para SAP](sap-high-availability-architecture-scenarios.md) se proporciona información inicial sobre las diferentes técnicas de alta disponibilidad y la recomendación ofrecida en Azure para aplicaciones SAP, que complementarán las instrucciones de esta sección.

En función del resultado de tamaño de la plataforma SAP BOBI, debe diseñar el panorama y determinar la distribución de los componentes de inteligencia empresarial en Azure Virtual Machines y las subredes. El nivel de redundancia en la arquitectura distribuida depende del objetivo de tiempo de recuperación (RTO) y del objetivo de punto de recuperación (RPO) necesarios para la empresa. La plataforma SAP BOBI incluye diferentes niveles y componentes en cada nivel que se deben diseñar para lograr redundancia. Por lo tanto, si se produce un error en un componente, no habrá ninguna interrupción en la aplicación SAP BOBI. Por ejemplo,

- Servidores de aplicaciones redundantes, como servidores de aplicaciones de inteligencia empresarial y servidores web.
- Componentes únicos, como la base de datos CMS, el servidor de repositorio de archivos y Load Balancer.

En la siguiente sección se describe cómo lograr una alta disponibilidad en cada componente de la plataforma SAP BOBI.

#### <a name="high-availability-for-application-servers"></a>Alta disponibilidad en los servidores de aplicaciones

En el caso de los servidores de aplicaciones web y de inteligencia empresarial, independientemente de si se instalan de forma independiente o conjunta, no se necesita una solución específica de alta disponibilidad. Puede lograr una alta disponibilidad mediante redundancia, es decir, configurando varias instancias de inteligencia empresarial y servidores web en varias instancias de Azure Virtual Machines.

Para reducir el impacto del tiempo de inactividad debido a uno o varios eventos, se recomienda seguir la práctica de alta disponibilidad para los servidores de aplicaciones que se ejecutan en varias máquinas virtuales.

- Use Availability Zones para protegerse frente a errores en el centro de datos.
- Configure varias instancias de Virtual Machines en un conjunto de disponibilidad para la redundancia.
- Use Managed Disks para las máquinas virtuales de un conjunto de disponibilidad.
- Configure cada capa de aplicación en conjuntos de disponibilidad independientes.

Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales con Linux](../../availability.md).

#### <a name="high-availability-for-cms-database"></a>Alta disponibilidad de la base de datos CMS

Si usa el servicio de base de datos como servicio (DBaaS) de Azure para la base de datos CMS, el marco de alta disponibilidad se proporciona de manera predeterminada. Solo tiene que seleccionar las capacidades de alta disponibilidad, redundancia y resistencia inherentes a la región y el servicio, sin necesidad de configurar ningún componente adicional. Para obtener más información sobre el Acuerdo de Nivel de Servicio de la oferta de DBaaS admitida en Azure, consulte [Alta disponibilidad en Azure Database for MySQL](../../../mysql/concepts-high-availability.md) y [Alta disponibilidad para Azure SQL Database](../../../azure-sql/database/high-availability-sla.md).

Para otras implementaciones de DBMS para la base de datos CMS, consulte [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), que proporciona información sobre la implementación de DBMS y su enfoque para lograr alta disponibilidad.

#### <a name="high-availability-for-file-repository-server"></a>Alta disponibilidad para el servidor de repositorio de archivos

El servidor de repositorio de archivos (FRS) hace referencia a los directorios del disco donde se almacena contenido como informes, universos y conexiones. Se comparte en todos los servidores de aplicaciones de ese sistema. Por lo tanto, debe asegurarse de que sea de alta disponibilidad.

En Azure, puede elegir [archivos de Azure Premium](../../../storage/files/storage-files-introduction.md) o [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) para el recurso compartido de archivos que están diseñados para ser de alta disponibilidad y muy duraderos por naturaleza. Para más información, consulte la sección [Redundancia](../../../storage/files/storage-files-planning.md#redundancy) para Azure Files.

> [!NOTE]
> El protocolo SMB para Azure Files está disponible con carácter general, pero la compatibilidad con el protocolo NFS para Azure Files se encuentra actualmente en versión preliminar. Para obtener más información, consulte [La compatibilidad de NFS 4.1 con Azure Files ya se encuentra en versión preliminar](https://azure.microsoft.com/en-us/blog/nfs-41-support-for-azure-files-is-now-in-preview/).

Como este servicio de recurso compartido de archivos no está disponible en todas las regiones, asegúrese de consultar el sitio de [Productos disponibles por región](https://azure.microsoft.com/en-us/global-infrastructure/services/) para buscar información actualizada. Si el servicio no está disponible en su región, puede crear un servidor NFS desde el que pueda compartir el sistema de archivos con la aplicación SAP BOBI. Sin embargo, también debe tener en cuenta su alta disponibilidad.

#### <a name="high-availability-for-load-balancer"></a>Alta disponibilidad para Load Balancer

Para distribuir el tráfico a través de un servidor web, puede usar Azure Load Balancer o Azure Application Gateway. La redundancia para cualquiera de los equilibradores de carga se puede lograr en función de la SKU que elija para la implementación.

- Para Azure Load Balancer, la redundancia se puede lograr mediante la configuración del front-end de Standard Load Balancer como con redundancia de zona. Para más información, consulte [Standard Load Balancer y Availability Zones](../../../load-balancer/load-balancer-standard-availability-zones.md).
- Para Application Gateway, se puede lograr alta disponibilidad en función del tipo de nivel seleccionado durante la implementación.
  - La SKU v1 admite escenarios de alta disponibilidad cuando se han implementado dos o más instancias. Azure distribuye estas instancias entre dominios de actualización y de errores para asegurarse de que las instancias no produzcan un error todas al mismo tiempo. Por lo tanto, con esta SKU, se puede lograr redundancia dentro de la zona.
  - La SKU v2 garantiza automáticamente que las nuevas instancias se distribuyan entre dominios de error y dominios de actualización. Si elige la redundancia de zona, las instancias más recientes también se distribuyen entre las zonas de disponibilidad para ofrecer resistencia ante errores de zona. Para más información, consulte [Escalabilidad automática y Application Gateway con redundancia de zona v2](../../../application-gateway/application-gateway-autoscaling-zone-redundant.md).

#### <a name="reference-high-availability-architecture-for-sap-businessobjects-bi-platform"></a>Arquitectura de alta disponibilidad de referencia para la plataforma de inteligencia empresarial SAP BusinessObjects

En la arquitectura de referencia se describe la configuración de la plataforma SAP BOBI con el conjunto de disponibilidad, que proporciona redundancia y disponibilidad de las máquinas virtuales dentro de la zona. La arquitectura muestra el uso de diferentes servicios de Azure, como Azure Application Gateway, Azure NetApp Files y Azure Database for MySQL para la plataforma SAP BOBI, que ofrece redundancia integrada, lo que reduce la complejidad de la administración de diferentes soluciones de alta disponibilidad.

En la ilustración siguiente, el tráfico entrante (HTTPS: TCP/443) tiene una carga equilibrada mediante la SKU v1 de Azure Application Gateway, que es de alta disponibilidad cuando se implementa en dos o más instancias. Se implementan varias instancias del servidor web, los servidores de administración y los servidores de procesamiento en instancias de Virtual Machines independientes para lograr redundancia y cada nivel se implementa en conjuntos de disponibilidad independientes. Azure NetApp Files tiene redundancia integrada dentro del centro de datos, por lo que los volúmenes de ANF para el servidor de repositorio de archivos tendrán una alta disponibilidad. La base de datos CMS se aprovisiona en Azure Database for MySQL (DBaaS), que tiene una alta disponibilidad inherente. Para más información, consulte la guía de [Alta disponibilidad en Azure Database for MySQL](../../../mysql/concepts-high-availability.md).

![Redundancia de la plataforma de inteligencia empresarial SAP BusinessObjects mediante conjuntos de disponibilidad](media/businessobjects-deployment-guide/businessobjects-deployment-high-availability.png)

En la arquitectura anterior se proporciona información sobre cómo se puede realizar la implementación de SAP BOBI en Azure. Sin embargo, no cubre todas las posibles opciones de configuración de la plataforma SAP BOBI en Azure. El cliente puede personalizar su implementación en función de sus necesidades empresariales, eligiendo diferentes productos o servicios para distintos componentes, como Load Balancer, el servidor de repositorio de archivos y DBMS.

En varias regiones de Azure, se ofrece Availability Zones, lo que significa que tiene un suministro independiente de origen de energía, refrigeración y red. Permite al cliente implementar la aplicación en dos o tres zonas de disponibilidad. Para el cliente que quiere lograr una alta disponibilidad en AZ, puede implementar la plataforma SAP BOBI en todas las zonas de disponibilidad, asegurándose de que todos los componentes de la aplicación tengan redundancia de zona.

### <a name="disaster-recovery"></a>Recuperación ante desastres

En la instrucción de esta sección se explica la estrategia para proporcionar protección de recuperación ante desastres para la plataforma SAP BOBI. Complementa el documento [Recuperación ante desastres para SAP](../../../site-recovery/site-recovery-sap.md), que representa los recursos principales para el enfoque general de recuperación ante desastres de SAP.

#### <a name="reference-disaster-recovery-architecture-for-sap-businessobjects-bi-platform"></a>Arquitectura de recuperación ante desastres de referencia para la plataforma de inteligencia empresarial SAP BusinessObjects

Esta arquitectura de referencia está ejecutando una implementación de varias instancias de la plataforma SAP BOBI con servidores de aplicaciones redundantes. Para la recuperación ante desastres, debe ser capaz de realizar la conmutación por error de todos los niveles en una región secundaria. Cada nivel usa una estrategia diferente para proporcionar protección mediante la recuperación ante desastres.

![Recuperación ante desastres de la plataforma de inteligencia empresarial SAP BusinessObjects](media/businessobjects-deployment-guide/businessobjects-deployment-disaster-recovery.png)

#### <a name="load-balancer"></a>Equilibrador de carga

Load Balancer se usa para distribuir el tráfico entre servidores de aplicaciones web de la plataforma SAP BOBI. Para lograr la recuperación ante desastres de Azure Application Gateway, implemente la configuración paralela de Application Gateway en la región secundaria.

#### <a name="virtual-machines-running-web-and-bi-application-servers"></a>Máquinas virtuales que ejecutan servidores de aplicaciones web y de inteligencia empresarial

El servicio de Azure Site Recovery se puede usar para replicar máquinas virtuales que ejecutan servidores de aplicaciones web y de inteligencia empresarial en la región secundaria. Replica los servidores en la región secundaria para que, cuando se produzcan desastres o interrupciones, pueda conmutar por error fácilmente a su entorno replicado y seguir trabajando.

#### <a name="file-repository-servers"></a>Servidores de repositorio de archivos

- **Azure NetApp Files** proporciona volúmenes NFS y SMB, por lo que se puede usar cualquier herramienta de copia basada en archivos para replicar datos entre regiones de Azure. Para obtener más información sobre cómo copiar el volumen de ANF en otra región, consulte [Preguntas más frecuentes acerca de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region).

  Puede usar la replicación entre regiones de Azure NetApp Files, que se encuentra actualmente en [versión preliminar](https://azure.microsoft.com/en-us/blog/azure-netapp-files-cross-region-replication-and-new-enhancements-in-preview/), que usa la tecnología de NetApp SnapMirror®. Por tanto, solo los bloques modificados se envían a través de la red en un formato comprimido y eficaz. Esta tecnología propietaria reduce la cantidad de datos necesarios para replicar entre regiones, lo que ahorra costos de transferencia de datos. También acorta el tiempo de replicación, por lo que puede lograr un objetivo de punto de restauración (RPO) más pequeño. Para obtener más información, consulte [Requisitos y consideraciones del uso de la replicación entre regiones](../../../azure-netapp-files/cross-region-replication-requirements-considerations.md).

- **Los archivos de Azure Premium** solo admiten el almacenamiento con redundancia local (LRS) y con redundancia de zona (ZRS). Para la estrategia de recuperación ante desastres de los archivos de Azure Premium, use [AzCopy](../../../storage/common/storage-use-azcopy-v10.md) o [Azure PowerShell](/powershell/module/az.storage/) para copiar los archivos en otra cuenta de almacenamiento en una región distinta. Para obtener más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../../../storage/common/storage-disaster-recovery-guidance.md).

#### <a name="cms-database"></a>Base de datos CMS

Azure Database for MySQL proporciona varias opciones para recuperar la base de datos si se produce algún desastre. Elija la opción adecuada que se adapte a su empresa.

- Habilite las réplicas de lectura entre regiones para mejorar el planeamiento de la continuidad empresarial y recuperación ante desastres. Puede crear hasta cinco réplicas desde el servidor de origen. Las réplicas de lectura se actualizan de manera asincrónica mediante la tecnología de replicación de registros binarios de MySQL. Las réplicas son nuevos servidores que se administran de forma similar a los servidores de Azure Database for MySQL normales. Obtenga más información sobre las réplicas de lectura, las regiones disponibles, las restricciones y la conmutación por error en el [artículo sobre los conceptos de las réplicas de lectura](../../../mysql/concepts-read-replicas.md).

- Use la característica de replicación geográfica de Azure Database for MySQL que permite restaurar el servidor mediante copias de seguridad con redundancia geográfica. Se puede acceder a estas copias de seguridad incluso cuando la región en la que se hospeda el servidor está sin conexión. Puede realizar la restauración a partir de estas copias de seguridad en cualquier otra región y volver a poner en línea el servidor.

  > [!NOTE]
  > La restauración geográfica solo es posible si se ha aprovisionado el servidor con almacenamiento de copia de seguridad con redundancia geográfica. No se admite el cambio de las **opciones de redundancia de copia de seguridad** después de la creación del servidor. Para obtener más información, consulte el artículo [Redundancia de copia de seguridad](../../../mysql/concepts-backup.md#backup-redundancy-options).

A continuación, se muestra la recomendación para la recuperación ante desastres de cada nivel usado en este ejemplo.

| Niveles de la plataforma SAP BOBI   | Recomendación                                                                                           |
|---------------------------|----------------------------------------------------------------------------------------------------------|
| Azure Application Gateway | Configuración paralela de Application Gateway en la región secundaria                                                |
| Servidores de aplicaciones web   | Replicación mediante Site Recovery                                                                         |
| Servidores de aplicaciones de inteligencia empresarial    | Replicación mediante Site Recovery                                                                         |
| Azure NetApp Files        | Herramienta de copia basada en archivos para replicar datos en la región secundaria **o** replicación entre regiones de ANF (versión preliminar) |
| Azure Database for MySQL  | Réplicas de lectura entre regiones **o** restauración de la copia de seguridad de copias de seguridad con redundancia geográfica.                             |

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la recuperación ante desastres para la implementación de una aplicación de SAP de niveles múltiples](../../../site-recovery/site-recovery-sap.md)
- [Planeamiento e implementación de Azure Virtual Machines para SAP](planning-guide.md)
- [Implementación de Azure Virtual Machines para SAP](deployment-guide.md)
- [Implementación de DBMS de Azure Virtual Machines para SAP](./dbms_guide_general.md)
