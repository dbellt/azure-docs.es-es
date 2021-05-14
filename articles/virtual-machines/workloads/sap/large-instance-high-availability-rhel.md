---
title: Alta disponibilidad de Azure (instancias grandes) para SAP en RHEL
description: Aprenda a automatizar una conmutación por error de base de datos SAP HANA mediante un clúster de Pacemaker en Red Hat Enterprise Linux.
author: jaawasth
ms.author: jaawasth
ms.service: virtual-machines-sap
ms.topic: how-to
ms.date: 04/19/2021
ms.openlocfilehash: f7b6e6efbbd17655b4f68d79ac26ee34ae754a3b
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107728453"
---
# <a name="azure-large-instances-high-availability-for-sap-on-rhel"></a>Alta disponibilidad de Azure (instancias grandes) para SAP en RHEL

> [!NOTE]
> Este artículo contiene referencias al término *lista negra*, un término que Microsoft ya no usa. Cuando se elimine este término del software, se eliminará también de este artículo.

En este artículo va a aprender a configurar el clúster de Pacemaker en RHEL 7.6 para automatizar una conmutación por error de base de datos SAP HANA. Para realizar los pasos de esta guía debe entender bien Linux, SAP HANA y Pacemaker.

En la tabla siguiente se incluyen los nombres de host que se usan en este artículo. Los bloques de código del artículo muestran los comandos que deben ejecutarse, así como la salida de esos comandos. Preste mucha atención a los nodos a los que se hace referencia en cada comando.

| Tipo | Nombre de host | Nodo|
|-------|-------------|------|
|Host principal|`sollabdsm35`|Nodo 1|
|Host secundario|`sollabdsm36`|Nodo 2|

## <a name="configure-your-pacemaker-cluster"></a>Configuración del clúster de Pacemaker


Para empezar a configurar el clúster, configure el intercambio de claves SSH a fin de establecer la confianza entre los nodos.

1. Use los comandos siguientes para crear un elemento `/etc/hosts` idéntico en ambos nodos.

    ```
    root@sollabdsm35 ~]# cat /etc/hosts
    27.0.0.1 localhost localhost.azlinux.com
    10.60.0.35 sollabdsm35.azlinux.com sollabdsm35 node1
    10.60.0.36 sollabdsm36.azlinux.com sollabdsm36 node2
    10.20.251.150 sollabdsm36-st
    10.20.251.151 sollabdsm35-st
    10.20.252.151 sollabdsm36-back
    10.20.252.150 sollabdsm35-back
    10.20.253.151 sollabdsm36-node
    10.20.253.150 sollabdsm35-node
    ```

2.  Cree e intercambie las claves SSH.
    1. Genere las claves SSH.

    ```
       [root@sollabdsm35 ~]# ssh-keygen -t rsa -b 1024
       [root@sollabdsm36 ~]# ssh-keygen -t rsa -b 1024
    ```
    2. Copie las claves en los hosts para SSH sin contraseña.
    
       ```
       [root@sollabdsm35 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm35
       [root@sollabdsm35 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm36
       [root@sollabdsm36 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm35
       [root@sollabdsm36 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm36
       ```

3.  Deshabilite SELinux en ambos nodos.
    ```
    [root@sollabdsm35 ~]# vi /etc/selinux/config

    ...

    SELINUX=disabled

    [root@sollabdsm36 ~]# vi /etc/selinux/config

    ...

    SELINUX=disabled

    ```  

4. Reinicie los servidores y luego use el siguiente comando para comprobar el estado de SELinux.
    ```
    [root@sollabdsm35 ~]# sestatus

    SELinux status: disabled

    [root@sollabdsm36 ~]# sestatus

    SELinux status: disabled
    ```

5. Configure NTP (Protocolo de tiempo de red). La hora y las zonas horarias de ambos nodos del clúster deben coincidir. Use el siguiente comando para abrir `chrony.conf` y compruebe el contenido del archivo.
    1. El siguiente contenido debe agregarse al archivo de configuración. Cambie los valores reales según el entorno.
        ```
        vi /etc/chrony.conf
    
         Use public servers from the pool.ntp.org project.
    
         Please consider joining the pool (http://www.pool.ntp.org/join.html).
    
        server 0.rhel.pool.ntp.org iburst
       ```
    
    2. Habilite el servicio chrony. 
      
        ```
        systemctl enable chronyd
    
        systemctl start chronyd
    
        
    
        chronyc tracking
    
        Reference ID : CC0BC90A (voipmonitor.wci.com)
    
        Stratum : 3
    
        Ref time (UTC) : Thu Jan 28 18:46:10 2021
    
        chronyc sources
    
        210 Number of sources = 8
    
        MS Name/IP address Stratum Poll Reach LastRx Last sample
    
        ===============================================================================
    
        ^+ time.nullroutenetworks.c> 2 10 377 1007 -2241us[-2238us] +/- 33ms
    
        ^* voipmonitor.wci.com 2 10 377 47 +956us[ +958us] +/- 15ms
    
        ^- tick.srs1.ntfo.org 3 10 177 801 -3429us[-3427us] +/- 100ms
        ```

6. Actualización del sistema
    1. En primer lugar, instale las actualizaciones más recientes en el sistema para empezar a instalar el dispositivo SBD.
    1. Si no quiere una actualización completa del sistema, aunque esté recomendada, actualice los siguientes paquetes como mínimo.
        1. `resource-agents-sap-hana`
        1. `selinux-policy`
        1. `iscsi-initiator-utils`

  
        ```
        node1:~ # yum update
        ```

7. Instale los repositorios de SAP HANA y RHEL-HA.

    ```
    subscription-manager repos –list

    subscription-manager repos
    --enable=rhel-sap-hana-for-rhel-7-server-rpms

    subscription-manager repos --enable=rhel-ha-for-rhel-7-server-rpms
    ```
      

8. Instale las herramientas Pacemaker, SBD, OpenIPMI, ipmitools y fencing_sbd en todos los nodos.

    ``` 
    yum install pcs sbd fence-agent-sbd.x86_64 OpenIPMI
    ipmitool
    ```

  ## <a name="configure-watchdog"></a>Configuración del guardián

En esta sección va a aprender a configurar el guardián. En esta sección se usan los dos hosts, `sollabdsm35` y `sollabdsm36`, a los que se hace referencia al principio de este artículo.

1. Asegúrese de que el demonio del guardián no se esté ejecutando en ningún sistema.
    ```
    [root@sollabdsm35 ~]# systemctl disable watchdog
    [root@sollabdsm36 ~]# systemctl disable watchdog
    [root@sollabdsm35 ~]# systemctl stop watchdog
    [root@sollabdsm36 ~]# systemctl stop watchdog
    [root@sollabdsm35 ~]# systemctl status watchdog

    ● watchdog.service - watchdog daemon

    Loaded: loaded (/usr/lib/systemd/system/watchdog.service; disabled;
    vendor preset: disabled)

    Active: inactive (dead)

    Nov 28 23:02:40 sollabdsm35 systemd[1]: Collecting watchdog.service

    ```

2. El guardián de Linux predeterminado, que se instala durante la instalación, es el guardián de iTCO, que no es compatible con los sistemas UCS y HPE SDFlex. Por lo tanto, este guardián se debe deshabilitar.
    1. El guardián equivocado se ha instalado y cargado en el sistema:
       ```
       sollabdsm35:~ # lsmod |grep iTCO
   
       iTCO_wdt 13480 0
   
       iTCO_vendor_support 13718 1 iTCO_wdt
       ```

    2. Descargue el controlador equivocado del entorno:
       ```  
       sollabdsm35:~ # modprobe -r iTCO_wdt iTCO_vendor_support
   
       sollabdsm36:~ # modprobe -r iTCO_wdt iTCO_vendor_support
       ```  
        
    3. Para asegurarse de que el controlador no se cargue durante el siguiente arranque del sistema, debe incluirse en la lista de bloqueados. Para incluir los módulos iTCO en la lista de bloqueados, agregue lo siguiente al final del archivo `50-blacklist.conf`:
       ```
       sollabdsm35:~ # vi /etc/modprobe.d/50-blacklist.conf
   
        unload the iTCO watchdog modules
   
       blacklist iTCO_wdt
   
       blacklist iTCO_vendor_support
       ```
    4. Copie el archivo en el host secundario.
       ```
       sollabdsm35:~ # scp /etc/modprobe.d/50-blacklist.conf sollabdsm35:
       /etc/modprobe.d/50-blacklist.conf
       ```  

    5. Compruebe si se ha iniciado el servicio IPMI. Es importante que el temporizador de IPMI no se esté ejecutando. La administración del temporizador se realiza desde el servicio SBD de Pacemaker.
       ```
       sollabdsm35:~ # ipmitool mc watchdog get
   
       Watchdog Timer Use: BIOS FRB2 (0x01)
   
       Watchdog Timer Is: Stopped
   
       Watchdog Timer Actions: No action (0x00)
   
       Pre-timeout interval: 0 seconds
   
       Timer Expiration Flags: 0x00
   
       Initial Countdown: 0 sec
   
       Present Countdown: 0 sec
   
       ``` 

3. De manera predeterminada no se crea el dispositivo requerido /dev/watchdog.

    ```
    sollabdsm35:~ # ls -l /dev/watchdog

    ls: cannot access /dev/watchdog: No such file or directory
    ```

4. Configure el guardián de IPMI.

    ``` 
    sollabdsm35:~ # mv /etc/sysconfig/ipmi /etc/sysconfig/ipmi.org

    sollabdsm35:~ # vi /etc/sysconfig/ipmi

    IPMI_SI=yes
    DEV_IPMI=yes
    IPMI_WATCHDOG=yes
    IPMI_WATCHDOG_OPTIONS="timeout=20 action=reset nowayout=0
    panic_wdt_timeout=15"
    IPMI_POWEROFF=no
    IPMI_POWERCYCLE=no
    IPMI_IMB=no
    ```
5. Copie el archivo de configuración del guardián en el host secundario.
    ```
    sollabdsm35:~ # scp /etc/sysconfig/ipmi
    sollabdsm36:/etc/sysconfig/ipmi
    ```
6.  Habilite e inicie el servicio IPMI.
    ```
    [root@sollabdsm35 ~]# systemctl enable ipmi

    Created symlink from
    /etc/systemd/system/multi-user.target.wants/ipmi.service to
    /usr/lib/systemd/system/ipmi.service.

    [root@sollabdsm35 ~]# systemctl start ipmi

    [root@sollabdsm36 ~]# systemctl enable ipmi

    Created symlink from
    /etc/systemd/system/multi-user.target.wants/ipmi.service to
    /usr/lib/systemd/system/ipmi.service.

    [root@sollabdsm36 ~]# systemctl start ipmi
    ```
     Ahora se ha iniciado el servicio IPMI y se ha creado el dispositivo /dev/watchdog, pero el temporizador sigue detenido. Más adelante, SBD va a administrar el restablecimiento del guardián y a habilitar el temporizador de IPMI.
7.  Compruebe que /dev/watchdog existe pero que no está en uso.
    ```
    [root@sollabdsm35 ~]# ipmitool mc watchdog get
    Watchdog Timer Use: SMS/OS (0x04)
    Watchdog Timer Is: Stopped
    Watchdog Timer Actions: No action (0x00)
    Pre-timeout interval: 0 seconds
    Timer Expiration Flags: 0x10
    Initial Countdown: 20 sec
    Present Countdown: 20 sec

    [root@sollabdsm35 ~]# ls -l /dev/watchdog
    crw------- 1 root root 10, 130 Nov 28 23:12 /dev/watchdog
    [root@sollabdsm35 ~]# lsof /dev/watchdog
    ```

## <a name="sbd-configuration"></a>Configuración de SBD
En esta sección va a aprender a configurar SBD. En esta sección se usan los dos hosts, `sollabdsm35` y `sollabdsm36`, a los que se hace referencia al principio de este artículo.

1.  Asegúrese de que el disco iSCSI o FC esté visible en ambos nodos. En este ejemplo se usa un dispositivo SBD basado en FC. Para más información sobre las limitaciones de SBD, consulte la [guía de diseño para clústeres de alta disponibilidad de RHEL: consideraciones de SBD](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Faccess.redhat.com%2Farticles%2F2941601&data=04%7C01%7Cralf.klahr%40microsoft.com%7Cd49d7a3e3871449cdecc08d8c77341f1%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637478645171139432%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C1000&sdata=c%2BUAC5gmgpFNWZCQFfiqcik8CH%2BmhH2ly5DsOV1%2FE5M%3D&reserved=0).
2.  El id. de LUN debe ser idéntico en todos los nodos.
  
3.  Compruebe el estado de múltiples rutas en el dispositivo SBD.
    ```
    multipath -ll
    3600a098038304179392b4d6c6e2f4b62 dm-5 NETAPP ,LUN C-Mode
    size=1.0G features='4 queue_if_no_path pg_init_retries 50
    retain_attached_hw_handle' hwhandler='1 alua' wp=rw
    |-+- policy='service-time 0' prio=50 status=active
    | |- 8:0:1:2 sdi 8:128 active ready running
    | `- 10:0:1:2 sdk 8:160 active ready running
    `-+- policy='service-time 0' prio=10 status=enabled
    |- 8:0:3:2 sdj 8:144 active ready running
    `- 10:0:3:2 sdl 8:176 active ready running
    ```

4.  Cree los discos de SBD y configure el vallado primitivo del clúster. Este paso se debe ejecutar en el primer nodo.
    ```
    sbd -d /dev/mapper/3600a098038304179392b4d6c6e2f4b62 -4 20 -1 10 create 

    Initializing device /dev/mapper/3600a098038304179392b4d6c6e2f4b62
    Creating version 2.1 header on device 4 (uuid:
    ae17bd40-2bf9-495c-b59e-4cb5ecbf61ce)

    Initializing 255 slots on device 4

    Device /dev/mapper/3600a098038304179392b4d6c6e2f4b62 is initialized.
    ```

5.  Copie la configuración de SBD en el nodo 2.
    ```
    vi /etc/sysconfig/sbd

    SBD_DEVICE="/dev/mapper/3600a09803830417934d6c6e2f4b62" 
    SBD_PACEMAKER=yes
    SBD_STARTMODE=always
    SBD_DELAY_START=no
    SBD_WATCHDOG_DEV=/dev/watchdog
    SBD_WATCHDOG_TIMEOUT=15
    SBD_TIMEOUT_ACTION=flush,reboot
    SBD_MOVE_TO_ROOT_CGROUP=auto
    SBD_OPTS=

    scp /etc/sysconfig/sbd node2:/etc/sysconfig/sbd
    ```

6.  Compruebe que el disco de SBD es visible desde ambos nodos.
    ```
    sbd -d /dev/mapper/3600a098038304179392b4d6c6e2f4b62 dump

    ==Dumping header on disk /dev/mapper/3600a098038304179392b4d6c6e2f4b62

    Header version : 2.1

    UUID : ae17bd40-2bf9-495c-b59e-4cb5ecbf61ce

    Number of slots : 255
    Sector size : 512
    Timeout (watchdog) : 5
    Timeout (allocate) : 2
    Timeout (loop) : 1
    Timeout (msgwait) : 10

    ==Header on disk /dev/mapper/3600a098038304179392b4d6c6e2f4b62 is dumped
    ```

7.  Agregue el dispositivo SBD al archivo de configuración de SBD.

    ```
    # SBD_DEVICE specifies the devices to use for exchanging sbd messages
    # and to monitor. If specifying more than one path, use ";" as
    # separator.
    #

    SBD_DEVICE="/dev/mapper/3600a098038304179392b4d6c6e2f4b62"
    ## Type: yesno
     Default: yes
     # Whether to enable the pacemaker integration.
    SBD_PACEMAKER=yes
    ```

## <a name="cluster-initialization"></a>Inicialización del clúster
En esta sección se inicializa el clúster. En esta sección se usan los dos hosts, `sollabdsm35` y `sollabdsm36`, a los que se hace referencia al principio de este artículo.

1.  Configure la contraseña de usuario del clúster (todos los nodos).
    ```
    passwd hacluster
    ```
2.  Inicie PCS en todos los sistemas.
    ```
    systemctl enable pcsd
    ```
  

3.  Detenga el firewall y deshabilítelo (en todos los nodos).
    ```
    systemctl disable firewalld 

     systemctl mask firewalld

    systemctl stop firewalld
    ```

4.  Inicie el servicio PCSD.
    ```
    systemctl start pcsd
    ```

5.  Ejecute la autenticación del clúster solo desde el nodo 1.

    ```
    pcs cluster auth sollabdsm35 sollabdsm36

        Username: hacluster

            Password:
            sollabdsm35.localdomain: Authorized
            sollabdsm36.localdomain: Authorized

     ``` 

6.  Cree el clúster.
    ```
    pcs cluster setup --start --name hana sollabdsm35 sollabdsm36
    ```
  

7.  Compruebe el estado del clúster.

    ```
    pcs cluster status

    Cluster name: hana

    WARNINGS:

    No stonith devices and stonith-enabled is not false

    Stack: corosync

    Current DC: sollabdsm35 (version 1.1.20-5.el7_7.2-3c4c782f70) -
    partition with quorum

    Last updated: Sat Nov 28 20:56:57 2020

    Last change: Sat Nov 28 20:54:58 2020 by hacluster via crmd on
    sollabdsm35

    2 nodes configured

    0 resources configured

    Online: [ sollabdsm35 sollabdsm36 ]

    No resources

    Daemon Status:

    corosync: active/disabled

    pacemaker: active/disabled

    pcsd: active/disabled
    ```

8. Si un nodo no se une al clúster, compruebe si el firewall se sigue ejecutando.

9. Creación y habilitación del dispositivo SBD
    ```
    pcs stonith create SBD fence_sbd devices=/dev/mapper/3600a098038303f4c467446447a
    ```
  
10. Detenga el clúster para reiniciar sus servicios (en todos los nodos).

    ```
    pcs cluster stop --all
    ```

11. Reinicie los servicios del clúster (en todos los nodos).

    ```
    systemctl stop pcsd
    systemctl stop pacemaker
    systemctl stop corocync
    systemctl enable sbd
    systemctl start corosync
    systemctl start pacemaker
    systemctl start pcsd
    ```

12. Corosync debe iniciar el servicio SBD.

    ```
    systemctl status sbd

    ● sbd.service - Shared-storage based fencing daemon

    Loaded: loaded (/usr/lib/systemd/system/sbd.service; enabled; vendor
    preset: disabled)

    Active: active (running) since Wed 2021-01-20 01:43:41 EST; 9min ago
    ```

13. Reinicie el clúster (si no se ha iniciado automáticamente desde PCSD).

    ```
    pcs cluster start –-all

    sollabdsm35: Starting Cluster (corosync)...

    sollabdsm36: Starting Cluster (corosync)...

    sollabdsm35: Starting Cluster (pacemaker)...

    sollabdsm36: Starting Cluster (pacemaker)...
    ```
  

14. Habilite la configuración de Stonith.
    ```
    pcs stonith enable SBD --device=/dev/mapper/3600a098038304179392b4d6c6e2f4d65
    pcs property set stonith-watchdog-timeout=20
    pcs property set stonith-action=reboot
    ```
  

15. Compruebe el nuevo estado del clúster con un recurso.
    ```
    pcs status

    Cluster name: hana

    Stack: corosync

    Current DC: sollabdsm35 (version 1.1.16-12.el7-94ff4df) - partition
    with quorum

    Last updated: Tue Oct 16 01:50:45 2018

    Last change: Tue Oct 16 01:48:19 2018 by root via cibadmin on
    sollabdsm35

    2 nodes configured

    1 resource configured

    Online: [ sollabdsm35 sollabdsm36 ]

    Full list of resources:

    SBD (stonith:fence_sbd): Started sollabdsm35

    Daemon Status:

    corosync: active/disabled

    pacemaker: active/disabled

    pcsd: active/enabled

    sbd: active/enabled

    [root@node1 ~]#
    ```
  

16. Ahora debe ejecutarse el temporizador de IPMI y SBD debe abrir el dispositivo /dev/watchdog.

    ```
    ipmitool mc watchdog get

    Watchdog Timer Use: SMS/OS (0x44)

    Watchdog Timer Is: Started/Running

    Watchdog Timer Actions: Hard Reset (0x01)

    Pre-timeout interval: 0 seconds

    Timer Expiration Flags: 0x10

    Initial Countdown: 20 sec

    Present Countdown: 19 sec

    [root@sollabdsm35 ~] lsof /dev/watchdog

    COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME

    sbd 117569 root 5w CHR 10,130 0t0 323812 /dev/watchdog
    ```

17. Compruebe el estado de SBD.

    ```
    sbd -d /dev/mapper/3600a098038304445693f4c467446447a list

    0 sollabdsm35 clear

    1 sollabdsm36 clear
    ```
  

18. Pruebe el vallado de SBD; para ello, bloquee el kernel.

    * Desencadene el bloqueo del kernel.

      ```
      echo c > /proc/sysrq-trigger

      System must reboot after 5 Minutes (BMC timeout) or the value which is
      set as panic_wdt_timeout in the /etc/sysconfig/ipmi config file.
      ```
  
    * La segunda prueba que se va a realizar consiste en vallar un nodo mediante comandos de PCS.

      ```
      pcs stonith fence sollabdsm36
      ```
  

19. En el resto del clúster de SAP HANA puede deshabilitar STONITH si establece:

   * pcs property set `stonith-enabled=false`
   * A veces es más fácil mantener STONITH desactivado durante la instalación del clúster, ya que evitará reinicios inesperados del sistema.
   * Este parámetro debe establecerse en true para un uso productivo. Si este parámetro no se establece en true, no se admite el clúster.
   * pcs property set `stonith-enabled=true`

## <a name="hana-integration-into-the-cluster"></a>Integración de HANA en el clúster

En esta sección se integra HANA en el clúster. En esta sección se usan los dos hosts, `sollabdsm35` y `sollabdsm36`, a los que se hace referencia al principio de este artículo.

Hay dos opciones para integrar HANA. La primera es una solución optimizada para costo en la que se puede usar el sistema secundario para ejecutar el sistema QAS. Este método no se recomienda, ya que no deja ningún sistema para probar actualizaciones en el software del clúster, el sistema operativo o HANA, y las actualizaciones de configuración pueden provocar un tiempo de inactividad no planeado del sistema PRD. Además, si es necesario activar el sistema PRD en el sistema secundario, se debe apagar QAS en el nodo secundario. La segunda opción consiste en instalar el sistema QAS en un clúster y usar un segundo clúster para PRD. Esta opción además permite probar todos los componentes antes de que pasen a producción. En este artículo se muestra cómo configurar la segunda opción.


* Este proceso se basa en la descripción de RHEL de la página:

  * https://access.redhat.com/articles/3004101

 ### <a name="steps-to-follow-to-configure-hsr"></a>Pasos que se deben seguir para configurar HSR

1.  Estas son las acciones que se ejecutan en el nodo 1 (principal).
    1. Asegúrese de que el modo de registro de la base de datos esté establecido en normal.

       ```  
   
       * su - hr2adm
   
       * hdbsql -u system -p $YourPass -i 00 "select value from
       "SYS"."M_INIFILE_CONTENTS" where key='log_mode'"
   
       
   
       VALUE
   
       "normal"
       ```
    2. La replicación del sistema SAP HANA solo funciona una vez realizada la copia de seguridad inicial. El siguiente comando crea una copia de seguridad inicial en el directorio `/tmp/`. Seleccione un sistema de archivos de copia de seguridad adecuado para la base de datos. 
       ```
       * hdbsql -i 00 -u system -p $YourPass "BACKUP DATA USING FILE
       ('/tmp/backup')"
   
   
   
       Backup files were created
   
       ls -l /tmp
   
       total 2031784
   
       -rw-r----- 1 hr2adm sapsys 155648 Oct 26 23:31 backup_databackup_0_1
   
       -rw-r----- 1 hr2adm sapsys 83894272 Oct 26 23:31 backup_databackup_2_1
   
       -rw-r----- 1 hr2adm sapsys 1996496896 Oct 26 23:31 backup_databackup_3_1
   
       ```  

    3. Haga una copia de seguridad de todos los contenedores de base de datos de esta.
       ``` 
       * hdbsql -i 00 -u system -p $YourPass -d SYSTEMDB "BACKUP DATA USING
       FILE ('/tmp/sydb')"     
   
       * hdbsql -i 00 -u system -p $YourPass -d SYSTEMDB "BACKUP DATA FOR HR2
       USING FILE ('/tmp/rh2')"
   
       ```

    4. Habilite el proceso HSR en el sistema de origen.
       ```
       hdbnsutil -sr_enable --name=DC1

       nameserver is active, proceeding ...

       successfully enabled system as system replication source site

       done.
       ```

    5. Compruebe el estado del sistema principal.
       ```
       hdbnsutil -sr_state
    
       System Replication State
   
   
       online: true
   
       mode: primary
   
       operation mode: primary
   
       site id: 1
   
       site name: DC1
   
       
   
       is source system: true
   
       is secondary/consumer system: false
   
       has secondaries/consumers attached: false
   
       is a takeover active: false
   
       
   
       Host Mappings:
   
       ~~~~~~~~~~~~~~
   
       Site Mappings:
   
       ~~~~~~~~~~~~~~
   
       DC1 (primary/)
   
       Tier of DC1: 1
   
       Replication mode of DC1: primary
   
       Operation mode of DC1:
   
       done.
       ```

 2. Estas son las acciones que se ejecutan en el nodo 2 (secundario).
     1. Detenga la base de datos.
       ```
       su – hr2adm
    
       sapcontrol -nr 00 -function StopSystem
       ```
    

     2. Solo en SAP HANA 2.0, copie los archivos `PKI SSFS_HR2.KEY` y `SSFS_HR2.DAT` del sistema SAP HANA desde el nodo principal en el secundario.
       ```
       scp
       root@node1:/usr/sap/HR2/SYS/global/security/rsecssfs/key/SSFS_HR2.KEY
       /usr/sap/HR2/SYS/global/security/rsecssfs/key/SSFS_HR2.KEY
   
       
   
       scp
       root@node1:/usr/sap/HR2/SYS/global/security/rsecssfs/data/SSFS_HR2.DAT
       /usr/sap/HR2/SYS/global/security/rsecssfs/data/SSFS_HR2.DAT
       ```

     3. Habilite el secundario como sitio de replicación.
       ``` 
       su - hr2adm
   
       hdbnsutil -sr_register --remoteHost=node1 --remoteInstance=00
       --replicationMode=syncmem --name=DC2
   
       
   
       adding site ...
   
       --operationMode not set; using default from global.ini/[system_replication]/operation_mode: logreplay
   
       nameserver node2:30001 not responding.
   
       collecting information ...
   
       updating local ini files ...
   
       done.
   
       ```

     4. Inicie la base de datos.
       ```
       sapcontrol -nr 00 -function StartSystem 
       ```
    
     5. Compruebe el estado de la base de datos.
       ```
       hdbnsutil -sr_state
   
       ~~~~~~~~~
       System Replication State
   
       online: true
   
       mode: syncmem
   
       operation mode: logreplay
   
       site id: 2
   
       site name: DC2   
   
       is source system: false
   
       is secondary/consumer system: true
   
       has secondaries/consumers attached: false
   
       is a takeover active: false
   
       active primary site: 1
   
       
   
       primary primarys: node1
   
       Host Mappings:
   
   
   
       node2 -> [DC2] node2
   
       node2 -> [DC1] node1
   
       
   
       Site Mappings:
   
   
   
       DC1 (primary/primary)
   
       |---DC2 (syncmem/logreplay)
   
       
   
       Tier of DC1: 1
   
       Tier of DC2: 2
   
       
   
       Replication mode of DC1: primary
   
       Replication mode of DC2: syncmem
   
       Operation mode of DC1: primary
   
       Operation mode of DC2: logreplay
   
       
   
       Mapping: DC1 -> DC2
   
       done.
       ~~~~~~~~~~~~~~
       ```

3. También es posible obtener más información sobre el estado de replicación:
    ```
    ~~~~~
    hr2adm@node1:/usr/sap/HR2/HDB00> python
    /usr/sap/HR2/HDB00/exe/python_support/systemReplicationStatus.py

    | Database | Host | Port | Service Name | Volume ID | Site ID | Site
    Name | Secondary | Secondary | Secondary | Secondary | Secondary |
    Replication | Replication | Replication |

    | | | | | | | | Host | Port | Site ID | Site Name | Active Status |
    Mode | Status | Status Details |

    | SYSTEMDB | node1 | 30001 | nameserver | 1 | 1 | DC1 | node2 | 30001
    | 2 | DC2 | YES | SYNCMEM | ACTIVE | |

    | HR2 | node1 | 30007 | xsengine | 2 | 1 | DC1 | node2 | 30007 | 2 |
    DC2 | YES | SYNCMEM | ACTIVE | |

    | HR2 | node1 | 30003 | indexserver | 3 | 1 | DC1 | node2 | 30003 | 2
    | DC2 | YES | SYNCMEM | ACTIVE | |


    status system replication site "2": ACTIVE

    overall system replication status: ACTIVE

    Local System Replication State
    

    mode: PRIMARY

    site id: 1

    site name: DC1
    ```
  

#### <a name="log-replication-mode-description"></a>Descripción del modo de replicación de registros

Para obtener más información sobre el modo de replicación de registros, vea la [documentación oficial de SAP](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/627bd11e86c84ec2b9fcdf585d24011c.html).
  

#### <a name="network-setup-for-hana-system-replication"></a>Configuración de red para la replicación del sistema HANA


Para asegurarse de que el tráfico de replicación use la VLAN adecuada para la replicación, debe configurarse correctamente en `global.ini`. Si se omite este paso, HANA usa la VLAN de acceso para la replicación, lo que podría no ser deseable.


En los siguientes ejemplos se muestra la configuración de resolución de nombres de host para la replicación del sistema en un sitio secundario. Se pueden identificar tres redes distintas:

* Red pública con direcciones en el rango de 10.0.1.*

* Red para la comunicación SAP HANA interna entre los hosts de cada sitio: 192.168.1.*

* Red dedicada para la replicación del sistema: 10.5.1.*

En el primer ejemplo, el parámetro `[system_replication_communication]listeninterface` se ha establecido en `.global` y solo se han especificado los hosts del sitio de replicación vecino.

En el ejemplo siguiente, el parámetro `[system_replication_communication]listeninterface` se ha establecido en `.internal` y se han especificado todos los hosts de ambos sitios.

  

Para más información, consulte [Configuración de red para la replicación del sistema de SAP HANA](https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html).

  

Para la replicación del sistema no es necesario editar el archivo `/etc/hosts`, los nombres de host internos ("virtuales") deben asignarse a direcciones IP en el archivo `global.ini` para crear una red dedicada para la replicación del sistema. La sintaxis para esto es la siguiente:

global.ini

[system_replication_hostname_resolution]

<ip-address_site>=<internal-host-name_site>


## <a name="configure-sap-hana-in-a-pacemaker-cluster"></a>Configuración de SAP HANA en un clúster de Pacemaker
En esta sección va a aprender a configurar SAP HANA en un clúster de Pacemaker. En esta sección se usan los dos hosts, `sollabdsm35` y `sollabdsm36`, a los que se hace referencia al principio de este artículo.

Asegúrese de cumplir los siguientes requisitos previos:  

* El clúster de Pacemaker está configurado de acuerdo con la documentación y tiene un vallado correcto y en funcionamiento

* El inicio de SAP HANA al arrancar está deshabilitado en todos los nodos del clúster, ya que este va a administrar el inicio y la detención

* La replicación y la adquisición del sistema de SAP HANA mediante herramientas de SAP funcionan correctamente entre los nodos del clúster

* SAP HANA contiene una cuenta de supervisión que el clúster puede usar desde ambos nodos

* Ambos nodos están suscritos a los canales "Alta disponibilidad" y "RHEL for SAP HANA" (RHEL 6, RHEL 7)

  

* En general, ejecute todos los comandos de PCS solo desde un nodo, ya que CIB se actualiza automáticamente desde el shell de PCS.

* [Más información sobre la directiva de cuórum](https://access.redhat.com/solutions/645843)

### <a name="steps-to-configure"></a>Pasos de configuración 
1. Configure PCS.
    ```
    [root@node1 ~]# pcs property unset no-quorum-policy (optional – only if it was set before)
    [root@node1 ~]# pcs resource defaults resource-stickiness=1000
    [root@node1 ~]# pcs resource defaults migration-threshold=5000
    ```
2.  Configure Corosync.
    Para más información, consulte [cómo puedo configurar mi clúster de alta disponibilidad de RHEL 7 con Pacemaker y corosync](https://access.redhat.com/solutions/1293523).
    ```
    cat /etc/corosync/corosync.conf

    totem {

    version: 2

    secauth: off

    cluster_name: hana

    transport: udpu

    }

    

    nodelist {

    node {

    ring0_addr: node1.localdomain

    nodeid: 1

    }

    

    node {

    ring0_addr: node2.localdomain

    nodeid: 2

    }

    }

    

    quorum {

    provider: corosync_votequorum

    two_node: 1

    }

    

    logging {

    to_logfile: yes

    logfile: /var/log/cluster/corosync.log

    to_syslog: yes

    }

    ```
  

3.  Cree un recurso SAPHanaTopology clonado.
    El recurso SAPHanaTopology recopila el estado y la configuración de la replicación del sistema de SAP HANA en cada nodo. SAPHanaTopology requiere que se configuren los siguientes atributos.
       ```
       pcs resource create SAPHanaTopology_HR2_00 SAPHanaTopology SID=HR2 InstanceNumber=00 --clone clone-max=2 clone-node-max=1    interleave=true
       ```

    | Nombre del atributo | Descripción  |
    |---|---|
    | SID | Identificador del sistema (SID) de SAP de la instalación de SAP HANA. Debe ser el mismo para todos los nodos. |
    | NúmeroInstancia | Identificador de instancia de SAP de dos dígitos.|

    * Estado del recurso
       ```
       pcs resource show SAPHanaTopology_HR2_00
   
       InstanceNumber 2-digit SAP Instance identifier.
       pcs resource show SAPHanaTopology_HR2_00-clone
   
       Clone: SAPHanaTopology_HR2_00-clone
   
        Meta Attrs: clone-max=2 clone-node-max=1 interleave=true
   
        Resource: SAPHanaTopology_HR2_00 (class=ocf provider=heartbeat
       type=SAPHanaTopology)
   
        Attributes: InstanceNumber=00 SID=HR2
   
        Operations: monitor interval=60 timeout=60
       (SAPHanaTopology_HR2_00-monitor-interval-60)
   
        start interval=0s timeout=180
       (SAPHanaTopology_HR2_00-start-interval-0s)
   
        stop interval=0s timeout=60 (SAPHanaTopology_HR2_00-stop-interval-0s)
   
       ```

4.  Cree el recurso SAPHana principal/secundario.
    * El recurso SAPHana es responsable de iniciar, detener y reubicar la base de datos SAP HANA. Este recurso debe ejecutarse como un recurso de clúster principal o secundario. El recurso tiene los siguientes atributos.

| Nombre del atributo            | ¿Necesario? | Valor predeterminado | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|---------------------------|-----------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SID                       | Sí       | None          | Identificador del sistema (SID) de SAP de la instalación de SAP HANA. Debe ser igual para todos los nodos.                                                                                                                                                                                                                                                                                                                                                                                       |
| NúmeroInstancia            | Sí       | ninguno          | Identificador de instancia de SAP de dos dígitos.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| PREFER_SITE_TAKEOVER      | no        | sí           | ¿Debería el clúster preferir cambiar a la instancia secundaria en lugar de reiniciar la primaria localmente? ("no": prefiere reiniciar localmente; "yes": prefiere adquirirlo en el sitio remoto)                                                                                                                                                                                                                                                                                            |
|                           |           |               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| AUTOMATED_REGISTER        | no        | FALSE         | ¿Debe registrarse la anterior instancia SAP HANA principal como secundaria después de la adquisición de DUPLICATE_PRIMARY_TIMEOUT? ("false": no, se necesita intervención manual; "true": sí, el agente de recursos registrará la principal anterior como secundaria).                                                                                                                                                                                                                        |
| DUPLICATE_PRIMARY_TIMEOUT | no        | 7200          | Diferencia de tiempo (en segundos) necesaria entre las marcas de tiempo principales, si se produce una situación de doble principal. Si la diferencia de tiempo es menor que el intervalo de tiempo, el clúster contiene una o ambas instancias en estado "WAITING" (En espera). Esto es para dar al administrador la oportunidad de reaccionar en una conmutación por error. Una réplica principal anterior con error se registrará después de que se supere la diferencia horaria. Después de este registro en la nueva base de datos principal, la replicación del sistema sobrescribirá todos los datos. |

5.  Cree el recurso HANA.
    ```
    pcs resource create SAPHana_HR2_00 SAPHana SID=HR2 InstanceNumber=00 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200   AUTOMATED_REGISTER=true primary notify=true clone-max=2 clone-node-max=1 interleave=true

    pcs resource show SAPHana_HR2_00-primary


    Primary: SAPHana_HR2_00-primary

        Meta Attrs: clone-max=2 clone-node-max=1 interleave=true notify=true

        Resource: SAPHana_HR2_00 (class=ocf provider=heartbeat type=SAPHana)

        Attributes: AUTOMATED_REGISTER=false DUPLICATE_PRIMARY_TIMEOUT=7200
    InstanceNumber=00 PREFER_SITE_TAKEOVER=true SID=HR2

        Operations: demote interval=0s timeout=320
    (SAPHana_HR2_00-demote-interval-0s)

        monitor interval=120 timeout=60 (SAPHana_HR2_00-monitor-interval-120)

        monitor interval=121 role=Secondary timeout=60
    (SAPHana_HR2_00-monitor-

        interval-121)

        monitor interval=119 role=Primary timeout=60 (SAPHana_HR2_00-monitor-

        interval-119)

        promote interval=0s timeout=320 (SAPHana_HR2_00-promote-interval-0s)

        start interval=0s timeout=180 (SAPHana_HR2_00-start-interval-0s)

        stop interval=0s timeout=240 (SAPHana_HR2_00-stop-interval-0s)

    
    
    crm_mon -A1

    ....

    2 nodes configured

    5 resources configured

    Online: [ node1.localdomain node2.localdomain ]

    Active resources:

    .....

    Node Attributes:

    * Node node1.localdomain:

    + hana_hr2_clone_state : PROMOTED

    + hana_hr2_remoteHost : node2

    + hana_hr2_roles : 4:P:primary1:primary:worker:primary

    + hana_hr2_site : DC1

    + hana_hr2_srmode : syncmem

    + hana_hr2_sync_state : PRIM

    + hana_hr2_version : 2.00.033.00.1535711040

    + hana_hr2_vhost : node1

    + lpa_hr2_lpt : 1540866498

    + primary-SAPHana_HR2_00 : 150

    * Node node2.localdomain:

    + hana_hr2_clone_state : DEMOTED

    + hana_hr2_op_mode : logreplay

    + hana_hr2_remoteHost : node1

    + hana_hr2_roles : 4:S:primary1:primary:worker:primary

    + hana_hr2_site : DC2

    + hana_hr2_srmode : syncmem

    + hana_hr2_sync_state : SOK

    + hana_hr2_version : 2.00.033.00.1535711040

    + hana_hr2_vhost : node2

    + lpa_hr2_lpt : 30

    + primary-SAPHana_HR2_00 : 100
    ```

6.  Cree un recurso de dirección IP virtual.
    El clúster contendrá la dirección IP virtual para llegar a la instancia principal de SAP HANA. A continuación se muestra un comando de ejemplo para crear un recurso de IPaddr2 con IP 10.7.0.84/24.
    ```
    pcs resource create vip_HR2_00 IPaddr2 ip="10.7.0.84"
    pcs resource show vip_HR2_00

    Resource: vip_HR2_00 (class=ocf provider=heartbeat type=IPaddr2)

        Attributes: ip=10.7.0.84

        Operations: monitor interval=10s timeout=20s
    (vip_HR2_00-monitor-interval-10s)

        start interval=0s timeout=20s (vip_HR2_00-start-interval-0s)

        stop interval=0s timeout=20s (vip_HR2_00-stop-interval-0s)
    ```

7.  Cree restricciones.
    * Para una operación correcta, es necesario asegurarse de que los recursos de SAPHanaTopology se inician antes de iniciar los recursos de SAPHana y también de que la dirección IP virtual esté presente en el nodo donde se ejecuta el recurso principal de SAPHana. Para ello, es necesario crear las dos restricciones siguientes.
       ```
       pcs constraint order SAPHanaTopology_HR2_00-clone then SAPHana_HR2_00-primary symmetrical=false
       pcs constraint colocation add vip_HR2_00 with primary SAPHana_HR2_00-primary 2000
       ```

###  <a name="testing-the-manual-move-of-saphana-resource-to-another-node"></a>Prueba de la migración manual del recurso SAPHana a otro nodo

#### <a name="sap-hana-takeover-by-cluster"></a>(Adquisición de SAP Hana por parte del clúster)


Para comprobar la migración del recurso SAPHana de un nodo a otro, use el siguiente comando. Tenga en cuenta que la opción `--primary` no debe usarse al ejecutar el siguiente comando debido al funcionamiento interno del recurso SAPHana.
```pcs resource move SAPHana_HR2_00-primary```

Después de cada invocación del comando de migración del recurso PCS, el clúster crea restricciones de ubicación para lograr el traslado del recurso. Estas restricciones deben quitarse para permitir la conmutación automática por error en el futuro.
Para quitarlas, puede usar el comando siguiente.
```
pcs resource clear SAPHana_HR2_00-primary
crm_mon -A1
Node Attributes:
    * Node node1.localdomain:
    + hana_hr2_clone_state : DEMOTED
    + hana_hr2_remoteHost : node2
    + hana_hr2_roles : 2:P:primary1::worker:
    + hana_hr2_site : DC1
    + hana_hr2_srmode : syncmem
    + hana_hr2_sync_state : PRIM
    + hana_hr2_version : 2.00.033.00.1535711040
    + hana_hr2_vhost : node1
    + lpa_hr2_lpt : 1540867236
    + primary-SAPHana_HR2_00 : 150
    * Node node2.localdomain:
    + hana_hr2_clone_state : PROMOTED
    + hana_hr2_op_mode : logreplay
    + hana_hr2_remoteHost : node1
    + hana_hr2_roles : 4:S:primary1:primary:worker:primary
    + hana_hr2_site : DC2
    + hana_hr2_srmode : syncmem
    + hana_hr2_sync_state : SOK
    + hana_hr2_version : 2.00.033.00.1535711040
    + hana_hr2_vhost : node2
    + lpa_hr2_lpt : 1540867311
    + primary-SAPHana_HR2_00 : 100
```
  

* Inicie sesión en HANA como comprobación.

  * Host degradado:

    ```
    hdbsql -i 00 -u system -p $YourPass -n 10.7.0.82

    result:

            * -10709: Connection failed (RTE:[89006] System call 'connect'
    failed, rc=111:Connection refused (10.7.0.82:30015))
    ```
  
  * Host promocionado:

    ```
    hdbsql -i 00 -u system -p $YourPass -n 10.7.0.84
    
    Welcome to the SAP HANA Database interactive terminal.
    
    
    
    Type: \h for help with commands
    
    \q to quit
    
    
    
    hdbsql HR2=>
    
    
    
    
    DB is online
    ```
  

Con la opción `AUTOMATED_REGISTER=false` no se puede cambiar de uno a otro.

Si esta opción se establece en false, debe volver a registrar el nodo:
```
hdbnsutil -sr_register --remoteHost=node2 --remoteInstance=00 --replicationMode=syncmem --name=DC1
```

Ahora el nodo 2, que era el principal, actúa como host secundario.

Considere la posibilidad de establecer esta opción en true para automatizar el registro del host degradado.
  
```
pcs resource update SAPHana_HR2_00-primary AUTOMATED_REGISTER=true
pcs cluster node clear node1
```

Si prefiere el registro automático, depende del escenario del cliente. Volver a registrar automáticamente el nodo después de una adquisición será más fácil para el equipo de operaciones. Sin embargo, puede que desee registrar el nodo manualmente para ejecutar primero pruebas adicionales y asegurarse de que todo funciona según lo previsto.
