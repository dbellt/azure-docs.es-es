---
title: 'Azure VMware Solution by CloudSimple: copia de seguridad de máquinas virtuales de carga de trabajo en la nube privada con Veeam'
description: Indica cómo puede realizar una copia de seguridad de las máquinas virtuales que se están ejecutando en una nube privada de CloudSimple basada en Azure con Veeam B&R 9.5
author: shortpatti
ms.author: v-patsho
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 115cb4203966bdf4cb60373d9c08994b6eb11576
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108182613"
---
# <a name="back-up-workload-vms-on-cloudsimple-private-cloud-using-veeam-br"></a>Copia de seguridad de máquinas virtuales de carga de trabajo en la nube privada de CloudSimple con Veeam B&R

Esta guía indica cómo puede realizar una copia de seguridad de las máquinas virtuales que se están ejecutando en una nube privada de CloudSimple basada en Azure con Veeam B&R 9.5.

## <a name="about-the-veeam-back-up-and-recovery-solution"></a>Acerca de la solución de copia de seguridad y recuperación de Veeam

La solución de Veeam incluye los siguientes componentes.

**Servidor de copia de seguridad**

El servidor de copia de seguridad es un servidor de Windows (VM) que actúa como centro de control para Veeam y realiza estas funciones: 

* Coordina las tareas de copia de seguridad, replicación, comprobación de recuperación y restauración
* Controla la programación de trabajos y la asignación de recursos
* Le permite configurar y administrar componentes de la infraestructura de copia de seguridad y especificar una configuración global para ella.

**Servidores proxy**

Los servidores proxy se instalan entre el servidor de copia de seguridad y otros componentes de la infraestructura de copia de seguridad. Estos servidores administran las siguientes funciones:

* Recuperación de datos de máquina virtual desde el almacenamiento de producción
* Compresión
* Desduplicación
* Cifrado
* Transmisión de datos al repositorio de copias de seguridad

**Repositorio de copias de seguridad**

El repositorio de copias de seguridad es la ubicación de almacenamiento en la que Veeam mantiene los archivos de copia de seguridad, las copias de máquinas virtuales y los metadatos de las máquinas virtuales replicadas.  El repositorio puede ser un servidor de Windows o Linux con discos locales (o con NFS/SMB montado) o un dispositivo de desduplicación de almacenamiento de hardware.

### <a name="veeam-deployment-scenarios"></a>Escenarios de implementación de Veeam
Puede aprovechar Azure para proporcionar un repositorio de copias de seguridad y un destino de almacenamiento para copias de seguridad y archivado a largo plazo. Todo el tráfico de red de copia de seguridad entre las máquinas virtuales de la nube privada y el repositorio de copias de seguridad de Azure viaja a través de un vínculo de baja latencia y ancho de banda alto. El tráfico de replicación entre regiones viaja por la red interna de backplane de Azure, lo cual reduce los costos de ancho de banda para los usuarios.

**Implementación básica**

En el caso de entornos con menos de 30 TB para realizar copias de seguridad, CloudSimple recomienda la siguiente configuración:

* El servidor de copia de seguridad de Veeam y el servidor proxy deben estar instalados en la misma máquina virtual de la nube privada.
* Un repositorio principal de copia de seguridad basado en Linux en Azure configurado como destino de los trabajos de copia de seguridad.
* `azcopy` se usa para copiar los datos del repositorio principal de copia de seguridad en un contenedor de blobs de Azure que se replica en otra región.

![Diagrama que muestra escenarios de implementación de Veeam básicos.](media/veeam-basicdeployment.png)

**Implementación avanzada**

En el caso de entornos con más de 30 TB para realizar copias de seguridad, CloudSimple recomienda la siguiente configuración:

* Un servidor proxy por nodo en el clúster de vSAN, tal y como recomienda Veeam.
* Un repositorio de copias de seguridad principal basado en Windows en la nube privada para almacenar en caché cinco días de datos para restauraciones rápidas.
* Un repositorio de copias de seguridad de Linux en Azure como destino de los trabajos de copia de seguridad para una retención de mayor duración. Este repositorio se debe configurar como un repositorio de copias de seguridad con escalabilidad horizontal.
* `azcopy` se usa para copiar los datos del repositorio principal de copia de seguridad en un contenedor de blobs de Azure que se replica en otra región.

![Escenarios de implementación básicos](media/veeam-advanceddeployment.png)

En la ilustración anterior, observe que el proxy de copia de seguridad es una máquina virtual con acceso de adición en caliente a discos de máquinas virtuales de carga de trabajo en el almacén de datos de vSAN. Veeam usa el modo de transporte de proxy de copia de seguridad de dispositivo virtual para vSAN.

## <a name="requirements-for-veeam-solution-on-cloudsimple"></a>Requisitos de la solución Veeam en CloudSimple

La solución Veeam requiere que haga lo siguiente:

* Proporcione sus propias licencias de Veeam.
* Implemente y administre Veeam para hacer una copia de seguridad de las cargas de trabajo que se ejecutan en la nube privada de CloudSimple.

Esta solución le proporciona un control total sobre la herramienta de copia de seguridad de Veeam y ofrece la opción de usar la interfaz de Veeam nativa o el complemento Veeam vCenter para administrar los trabajos de copia de seguridad de máquinas virtuales.

Si ya es un usuario de Veeam, puede ignorar la sección sobre los componentes de la solución Veeam y continuar directamente con los [escenarios de implementación de Veeam](#veeam-deployment-scenarios).

## <a name="install-and-configure-veeam-backups-in-your-cloudsimple-private-cloud"></a>Instalación y configuración de copias de seguridad de Veeam en la nube privada de CloudSimple

En las secciones siguientes se describe cómo instalar y configurar una solución de copia de seguridad de Veeam para la nube privada de CloudSimple.

El proceso de implementación consta de estos pasos:

1. [Interfaz de usuario de vCenter: Configuración de los servicios de infraestructura en la nube privada](#vcenter-ui-set-up-infrastructure-services-in-your-private-cloud)
2. [Portal de CloudSimple: Configuración de redes de la nube privada para Veeam](#cloudsimple-private-cloud-set-up-private-cloud-networking-for-veeam)
3. [Portal de CloudSimple: Escalado de privilegios](#cloudsimple-private-cloud-escalate-privileges-for-cloudowner)
4. [Azure Portal: Conexión de la red virtual a la nube privada](#azure-portal-connect-your-virtual-network-to-the-private-cloud)
5. [Azure Portal: Creación de un repositorio de copias de seguridad en Azure](#azure-portal-connect-your-virtual-network-to-the-private-cloud)
6. [Azure Portal: Configuración del almacenamiento de blobs de Azure para la retención de datos a largo plazo](#configure-azure-blob-storage-for-long-term-data-retention)
7. [Interfaz de usuario de vCenter de la nube privada: Instalación de Veeam B&R](#vcenter-console-of-private-cloud-install-veeam-br)
8. [Consola de Veeam: Configuración del software de copia de seguridad y recuperación de Veeam](#veeam-console-install-veeam-backup-and-recovery-software)
9. [Portal de CloudSimple: Configuración de privilegios de acceso y anulación del escalado de privilegios de Veeam](#cloudsimple-portal-set-up-veeam-access-and-de-escalate-privileges)

### <a name="before-you-begin"></a>Antes de empezar

Antes de empezar la implementación de Veeam se necesita lo siguiente:

* Una suscripción de Azure de su propiedad
* Un grupo de recursos de Azure creado previamente
* Una red virtual de Azure en su suscripción
* Una cuenta de almacenamiento de Azure
* Una [nube privada](create-private-cloud.md) creada mediante el portal de CloudSimple.  

Durante la fase de implementación, se necesitan los siguientes elementos:

* Plantillas de VMware para Windows para instalar Veeam (por ejemplo, Windows Server 2012 R2, imagen de 64 bits)
* Una VLAN disponible identificada para la red de copia de seguridad
* CIDR de la subred que se va a asignar a la red de copia de seguridad
* Elementos multimedia instalables (ISO) de Veeam 9.5 u3 cargados en el almacén de datos de vSAN de la nube privada

### <a name="vcenter-ui-set-up-infrastructure-services-in-your-private-cloud"></a>Interfaz de usuario de vCenter: Configuración de los servicios de infraestructura en la nube privada

Configure los servicios de infraestructura en la nube privada para facilitar la administración de las cargas de trabajo y las herramientas.

* Puede agregar un proveedor de identidades externo tal y como se describe en [Configuración de orígenes de identidades de vCenter para usar Active Directory](set-vcenter-identity.md) en cualquiera de los siguientes casos:

  * Quiere identificar a los usuarios de su instancia local de Active Directory (AD) en su nube privada.
  * Quiere configurar una instancia de AD en la nube privada para todos los usuarios.
  * Quiere utilizar Azure AD.
* Para proporcionar servicios de resolución de nombres, administración de direcciones IP y búsqueda de direcciones IP para las cargas de trabajo de la nube privada, configure un servidor DHCP y DNS tal y como se describe en el artículo [Configuración de aplicaciones y cargas de trabajo de DNS y DHCP en la nube privada de CloudSimple](dns-dhcp-setup.md).

### <a name="cloudsimple-private-cloud-set-up-private-cloud-networking-for-veeam"></a>Nube privada de CloudSimple: Configuración de redes de la nube privada para Veeam

Acceda al portal de CloudSimple para configurar las redes de la nube privada para la solución Veeam.

Cree una VLAN para la red de copia de seguridad y asígnele un valor de CIDR de subred. Para obtener instrucciones, consulte el artículo [Creación y administración de subredes y VLAN.](create-vlan-subnet.md)

Cree reglas de firewall entre la subred de administración y la red de copia de seguridad para permitir el tráfico de red en los puertos que usa Veeam. Consulte el tema de Veeam sobre [puertos usados](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95). Para obtener instrucciones sobre la creación de reglas de firewall, consulte [Configuración de tablas y reglas de firewall](firewall.md).

En la tabla siguiente se proporciona una lista de puertos.

| Icono | Descripción | Icono | Descripción |
| ------------ | ------------- | ------------ | ------------- |
| Servidor de copia de seguridad  | vCenter  | HTTPS/TCP  | 443 |
| Servidor de copia de seguridad <br> *Necesario para implementar los componentes de replicación y copia de seguridad de Veeam* | Proxy de copia de seguridad  | TCP/UDP  | 135, 137 a 139 y 445 |
    | Servidor de copia de seguridad   | DNS  | UDP  | 53  | 
    | Servidor de copia de seguridad   | Servidor de notificaciones de actualización de Veeam  | TCP  | 80  | 
    | Servidor de copia de seguridad   | Servidor de actualización de licencias de Veeam  | TCP  | 443  | 
    | Proxy de copia de seguridad   | vCenter |   |   | 
    | Proxy de copia de seguridad  | Repositorio de copias de seguridad de Linux   | TCP  | 22  | 
    | Proxy de copia de seguridad  | Repositorio de copias de seguridad de Windows  | TCP  | 49152 - 65535   | 
    | Repositorio de copias de seguridad  | Proxy de copia de seguridad  | TCP  | 2500 -5000  | 
    | Repositorio de copias de seguridad de origen<br> *Se usa para los trabajos de copia de seguridad*  | Repositorio de copias de seguridad de destino  | TCP  | 2500 - 5000  | 

Cree reglas de firewall entre la subred de carga de trabajo y la red de copia de seguridad como se describe en [Configuración de tablas y reglas de firewall](firewall.md).  En caso de copias de seguridad y restauración con reconocimiento de aplicaciones, se deben abrir [puertos adicionales](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95) en las máquinas virtuales de carga de trabajo que hospedan las aplicaciones específicas.

De forma predeterminada, CloudSimple proporciona un vínculo de ExpressRoute de 1 Gbps. Para tamaños de entorno mayores, puede que necesite un vínculo con un ancho de banda mayor. Póngase en contacto con Soporte técnico de Azure para más información sobre vínculos con un mayor ancho de banda.

Para continuar con la instalación, necesita la clave de autorización y el identificador URI del circuito del mismo nivel y acceso a su suscripción de Azure.  Esta información está disponible en la página Virtual Network Connection en el portal de CloudSimple. Para instrucciones, consulte [Obtención de información de emparejamiento para una conexión de red virtual de Azure a CloudSimple](virtual-network-connection.md). Si tiene algún problema para obtener la información, [póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="cloudsimple-private-cloud-escalate-privileges-for-cloudowner"></a>Nube privada de CloudSimple: Escalado de privilegios para el usuario "cloudowner"

El usuario "cloudowner" predeterminado no tiene privilegios suficientes en la instancia de vCenter de la nube privada para instalar VEEAM, por lo que se deben escalar los privilegios de vCenter del usuario. Para obtener más información, consulte [Escalar privilegios](escalate-private-cloud-privileges.md).

### <a name="azure-portal-connect-your-virtual-network-to-the-private-cloud"></a>Azure Portal: Conexión de la red virtual a la nube privada

Conecte la red virtual a la nube privada siguiendo las instrucciones de [Conexión de la red virtual de Azure mediante ExpressRoute](azure-expressroute-connection.md).

### <a name="azure-portal-create-a-backup-repository-vm"></a>Azure Portal: Creación de un repositorio de copias de seguridad

1. Cree una máquina virtual D2 v3 estándar con 2 vCPU y 8 GB de memoria.
2. Seleccione la imagen basada en CentOS 7.4.
3. Configure un grupo de seguridad de red para la máquina virtual. Compruebe que la máquina virtual no tiene una dirección IP pública y que no es accesible desde la red pública de Internet.
4. Cree una cuenta de usuario basada en un nombre de usuario y una contraseña para la nueva máquina virtual. Para obtener instrucciones, consulte [Creación de una máquina virtual Linux en Azure Portal](../virtual-machines/linux/quick-create-portal.md).
5. Cree una unidad de disco duro de 512 GiB estándar y conéctela a la máquina virtual del repositorio.  Para obtener más instrucciones, consulte [Conexión de un disco de datos administrado a una máquina virtual Windows en Azure Portal](../virtual-machines/windows/attach-managed-disk-portal.md).
6. [Cree un volumen de XFS en el disco administrado](https://www.digitalocean.com/docs/volumes/how-to/). Inicie sesión en la máquina virtual con las credenciales mencionadas anteriormente. Ejecute el siguiente script para crear un volumen lógico, agréguele el disco, cree una [partición](https://www.digitalocean.com/docs/volumes/how-to/partition/) del sistema de archivos XFS y [monte](https://www.digitalocean.com/docs/volumes/how-to/mount/) la partición en la ruta de acceso /backup1.

    Script de ejemplo:

    ```
    sudo pvcreate /dev/sdc
    sudo vgcreate backup1 /dev/sdc
    sudo lvcreate -n backup1 -l 100%FREE backup1
    sudo mkdir -p /backup1
    sudo chown veeamadmin /backup1
    sudo chmod 775 /backup1
    sudo mkfs.xfs -d su=64k -d sw=1 -f /dev/mapper/backup1-backup1
    sudo mount -t xfs /dev/mapper/backup1-backup1 /backup1
    ```

7. Exponga /backup1 como un punto de montaje de NFS en el servidor de copia de seguridad de Veeam que se ejecuta en la nube privada. Para obtener instrucciones, consulte el artículo de Digital Ocean sobre [configuración de un montaje de NFS en CentOS 6](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-centos-6). Use este nombre del recurso compartido de NFS cuando configure el repositorio de copias de seguridad en el servidor de copia de seguridad de Veeam.

8. Configure reglas de filtrado en el grupo de seguridad de red de la máquina virtual del repositorio de copias de seguridad para permitir explícitamente todo el tráfico de red hacia la máquina virtual y desde esta.

> [!NOTE]
> Veeam Backup & Replication usa el protocolo SSH para comunicarse con los repositorios de copias de seguridad de Linux y requiere la utilidad SCP en repositorios de Linux. Compruebe que el demonio de SSH esté configurado correctamente y que SCP esté disponible en el host de Linux.

### <a name="configure-azure-blob-storage-for-long-term-data-retention"></a>Configuración del almacenamiento de blobs de Azure para la retención de datos a largo plazo

1. Cree una cuenta de almacenamiento de uso general (GPv2) de tipo estándar y un contenedor de blobs, tal como se describe en el vídeo de Microsoft [Introducción a Azure Storage](https://azure.microsoft.com/resources/videos/get-started-with-azure-storage).
2. Cree un contenedor de almacenamiento de Azure, tal como se describe en [Creación de un contenedor](/rest/api/storageservices/create-container).
2. Descargue la utilidad de línea de comandos `azcopy` para Linux desde Microsoft. Puede usar los siguientes comandos del shell de Bash en CentOS 7.5.

    ```
    wget -O azcopy.tar.gz https://aka.ms/downloadazcopylinux64
    tar -xf azcopy.tar.gz
    sudo ./install.sh
    sudo yum -y install libunwind.x86_64
    sudo yum -y install icu
    ```

3. Use el comando `azcopy` para copiar los archivos de copia de seguridad en el contenedor de blobs.  Consulte [Transferencia de datos con AzCopy en Linux](../storage/common/storage-use-azcopy-v10.md) para obtener comandos detallados.

### <a name="vcenter-console-of-private-cloud-install-veeam-br"></a>Consola de vCenter de la nube privada: Instalación de Veeam B&R

Acceda a vCenter desde su nube privada para crear una cuenta de servicio de Veeam, instale Veeam B&R 9.5 y configure Veeam mediante la cuenta de servicio.

1. Cree un nuevo rol denominado "rol de copia de seguridad de Veeam" y asígnele los permisos necesarios según las recomendaciones de Veeam. Para más información, consulte el tema de Veeam sobre [permisos necesarios](https://helpcenter.veeam.com/docs/backup/vsphere/required_permissions.html?ver=95).
2. Cree un nuevo grupo denominado "grupo de usuarios de Veeam" en vCenter y asígnelo al "rol de copia de seguridad de Veeam".
3. Cree un nuevo usuario denominado "cuenta de servicio de Veeam" y agréguelo al "grupo de usuarios de Veeam".

    ![Creación de una cuenta de servicio de Veeam](media/veeam-vcenter01.png)

4. Cree un grupo de puertos distribuido en vCenter mediante la VLAN de la red de copia de seguridad. Para más información, consulte el vídeo de VMware sobre [creación de un grupo de puertos distribuido en el cliente web de vSphere](https://www.youtube.com/watch?v=wpCd5ZbPOpA).
5. Cree las máquinas virtuales para los servidores proxy y de copia de seguridad de Veeam en vCenter según los [requisitos del sistema de Veeam](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html?ver=95). Puede usar Windows 2012 R2 o Linux. Para más información, consulte [Requisitos para el uso de los repositorios de copias de seguridad de Linux](https://www.veeam.com/kb2216).
6. Monte los elementos multimedia instalables (ISO) de Veeam como un dispositivo CDROM en la máquina virtual del servidor de copia de seguridad de Veeam.
7. Mediante una sesión RDP en la máquina con Windows 2012 R2 (el destino de la instalación de Veeam), [instale Veeam B&R 9.5u3](https://helpcenter.veeam.com/docs/backup/vsphere/install_vbr.html?ver=95) en una máquina virtual de Windows 2012 R2.
8. Busque la dirección IP interna de la máquina virtual del servidor de copia de seguridad de Veeam y configúrela para que sea estática en el servidor DHCP. Los pasos exactos necesarios para ello dependen del servidor DHCP. Por ejemplo, el artículo de Netgate sobre <a href="https://www.netgate.com/docs/pfsense/dhcp/dhcp-server.html" target="_blank">asignaciones estáticas de DHCP</a> explica cómo configurar un servidor DHCP mediante un enrutador de pfSense.

### <a name="veeam-console-install-veeam-backup-and-recovery-software"></a>Consola de Veeam: Instalación del software de copia de seguridad y recuperación de Veeam

Mediante la consola de Veeam, configure el software de copia de seguridad y recuperación. Para más información, consulte [Veeam Backup & Replication v9: Instalación e implementación](https://www.youtube.com/watch?v=b4BqC_WXARk).

1. Agregue VMware vSphere como un entorno de servidor administrado. Cuando se le solicite, proporcione las credenciales de la cuenta de servicio de Veeam que creó al principio de [Consola de vCenter de la nube privada: Instalación de Veeam B&R](#vcenter-console-of-private-cloud-install-veeam-br).

    * Utilice la configuración predeterminada para el control de carga y la configuración avanzada predeterminada.
    * Establezca la ubicación del servidor de montaje en el servidor de copia de seguridad.
    * Cambie la ubicación de copia de seguridad configurada del servidor Veeam al repositorio remoto.

2. Agregue el servidor Linux en Azure como repositorio de copias de seguridad.

    * Utilice la configuración predeterminada para el control de carga y para la configuración avanzada. 
    * Establezca la ubicación del servidor de montaje en el servidor de copia de seguridad.
    * Cambie la ubicación de copia de seguridad configurada del servidor Veeam al repositorio remoto.

3. Habilite el cifrado de la copia de seguridad de configuración en **Inicio > Configuración de copia de seguridad**.

4. Agregue una máquina virtual de Windows Server como servidor proxy para el entorno de VMware. Mediante el uso de reglas de tráfico para un proxy, cifre los datos de copia de seguridad a través de la conexión.

5. Configure los trabajos de copia de seguridad.
    * Para configurar los trabajos de copia de seguridad, siga las instrucciones de [Creación de un trabajo de copia de seguridad](https://www.youtube.com/watch?v=YHxcUFEss4M).
    * Habilite el cifrado de los archivos de copia de seguridad en **Configuración avanzada > Almacenamiento**.

6. Configure los trabajos de copia de seguridad.

    * Para configurar los trabajos de copia de seguridad, siga las instrucciones del vídeo sobre [creación de un trabajo de copia de seguridad](https://www.youtube.com/watch?v=LvEHV0_WDWI&t=2s).
    * Habilite el cifrado de los archivos de copia de seguridad en **Configuración avanzada > Almacenamiento**.

### <a name="cloudsimple-portal-set-up-veeam-access-and-de-escalate-privileges"></a>Portal de CloudSimple: Configuración de privilegios de acceso y anulación del escalado de privilegios de Veeam
Cree una dirección IP pública para el servidor de recuperación y copia de seguridad de Veeam. Para obtener instrucciones, consulte el artículo [Asignación de direcciones IP públicas](public-ips.md).

Cree una regla de firewall para permitir que el servidor de copia de seguridad de Veeam cree una conexión de salida al sitio web de Veeam para descargar actualizaciones y revisiones en el puerto TCP 80. Para obtener instrucciones, consulte [Configuración de tablas y reglas de firewall](firewall.md).

Para anular el escalado de privilegios, consulte [Anular el escalado de privilegios](escalate-private-cloud-privileges.md#de-escalate-privileges).

## <a name="references"></a>Referencias

### <a name="cloudsimple-references"></a>Referencias de CloudSimple

* [Creación de una nube privada](create-private-cloud.md)
* [Creación y administración de VLAN y subredes](create-vlan-subnet.md)
* [Orígenes de identidades de vCenter](set-vcenter-identity.md)
* [Configuración de DNS y DHCP de carga de trabajo](dns-dhcp-setup.md)
* [Escalado de privilegios](escalate-privileges.md)
* [Configuración de tablas y reglas de firewall](firewall.md)
* [Permisos de nube privada](learn-private-cloud-permissions.md)
* [Asignación de direcciones IP públicas](public-ips.md)

### <a name="veeam-references"></a>Referencias de Veeam

* [Puertos usados](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95)
* [Permisos necesarios](https://helpcenter.veeam.com/docs/backup/vsphere/required_permissions.html?ver=95)
* [Requisitos del sistema](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html?ver=95)
* [Instalación de Veeam Backup & Replication](https://helpcenter.veeam.com/docs/backup/vsphere/install_vbr.html?ver=95)
* [Módulos y permisos necesarios para la compatibilidad con FLR para varios sistemas operativos y con el repositorio para Linux](https://www.veeam.com/kb2216)
* Para más información, consulte [el vídeo Veeam Backup & Replication v9: Instalación e implementación](https://www.youtube.com/watch?v=b4BqC_WXARk).
* Vídeo [Veeam v9: Creación de un trabajo de copia de seguridad](https://www.youtube.com/watch?v=YHxcUFEss4M)
* Vídeo [Veeam v9: Creación de un trabajo de copia de seguridad](https://www.youtube.com/watch?v=LvEHV0_WDWI&t=2s)

### <a name="azure-references"></a>Referencias de Azure

* [Configuración de una puerta de enlace de red virtual para ExpressRoute con Azure Portal](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md)
* [Conexión de una red virtual con un circuito: otra suscripción](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription)
* [Creación de una máquina virtual Linux en Azure Portal](../virtual-machines/linux/quick-create-portal.md)
* [Conexión de un disco de datos administrado a una máquina virtual Windows en Azure Portal](../virtual-machines/windows/attach-managed-disk-portal.md)
* [Introducción a Azure Storage: Vídeo](https://azure.microsoft.com/resources/videos/get-started-with-azure-storage)
* [Creación de un contenedor](/rest/api/storageservices/create-container)
* [Transferencia de datos con AzCopy en Linux](../storage/common/storage-use-azcopy-v10.md)

### <a name="vmware-references"></a>Referencias de VMware

* [Creación de un grupo de puertos distribuido en el cliente web de vSphere: Vídeo](https://www.youtube.com/watch?v=wpCd5ZbPOpA)

### <a name="other-references"></a>Otras referencias

* [Creación de un volumen de XFS en el disco administrado: RedHat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-xfs)
* [Configuración de un montaje de NFS en CentOS 7: HowToForge](https://www.howtoforge.com/nfs-server-and-client-on-centos-7)
* [Configuración del servidor DHCP: Netgate](https://www.netgate.com/docs/pfsense/dhcp/dhcp-server.html)
