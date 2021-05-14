---
title: Creación y carga de un disco duro virtual de Oracle Linux
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene el sistema operativo Oracle Linux.
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.subservice: oracle
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: 0ec82b3ad031cff98768b0cf9f1ee3bc5e650b69
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108737988"
---
# <a name="prepare-an-oracle-linux-virtual-machine-for-azure"></a>Preparación de una máquina virtual Oracle Linux para Azure

En este artículo se supone que ya ha instalado un sistema operativo Oracle Linux en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="oracle-linux-installation-notes"></a>Notas sobre la instalación de Oracle Linux
* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* Hyper-V y Azure admiten Oracle Linux tanto con el kernel de empresa ininterrumpible (UEK) como con el kernel compatible con Red Hat.
* El UEK2 de Oracle no se admite en Hyper-V y Azure porque no incluye los controladores requeridos.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet Convert-VHD.
* **Se requiere la compatibilidad de kernel para el montaje de sistemas de archivos UDF.** Al arrancar Azure la primera vez, la configuración de aprovisionamiento se pasa a la máquina virtual Linux a través de medios con formato UDF conectados al invitado. El agente Linux de Azure debe poder montar el sistema de archivos UDF para leer su configuración y aprovisionar la máquina virtual.
* Al instalar el sistema Linux se recomienda utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual para solucionar problemas. [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) o [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid) se pueden utilizar en discos de datos si así se prefiere.
* Las versiones de kernel de Linux inferiores a la versión 2.6.37 no admiten NUMA en Hyper-V con tamaños de VM más grandes. Este problema afecta principalmente a las distribuciones anteriores que usan el kernel Red Hat 2.6.32 de canal de subida y se ha corregido en Oracle Linux 6.6 y las versiones posteriores.
* No cree una partición de intercambio en el disco del SO. Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).
* Asegúrese de que el repositorio `Addons` está habilitado. Edite el archivo `/etc/yum.repos.d/public-yum-ol6.repo`(Oracle Linux 6) o `/etc/yum.repos.d/public-yum-ol7.repo`(Oracle Linux 7) y cambie la línea `enabled=0` a `enabled=1` en **[ol6_addons]** o **[ol7_addons]** en este archivo.

## <a name="oracle-linux-64-and-later"></a>Oracle Linux 6.4 y posterior
Debe completar los pasos de configuración específicos del sistema operativo para que la máquina virtual se ejecute en Azure.

1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. Desinstale NetworkManager ejecutando el comando siguiente:

    ```console
    # sudo rpm -e --nodeps NetworkManager
    ```

    **Nota:** si el paquete todavía no está instalado, se producirá un mensaje de error en este comando. Se espera que esto sea así.
4. Cree un archivo llamado **network** in the `/etc/sysconfig/` que contenga el texto siguiente:

    ```config   
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Cree un archivo llamado **ifcfg-eth0** in the `/etc/sysconfig/network-scripts/` que contenga el texto siguiente:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Asegúrese de que el servicio de red se inicie en el arranque ejecutando el comando siguiente:

    ```console
    # chkconfig network on
    ```

8. Instale python-pyasn1 ejecutando el comando siguiente:

    ```console
    # sudo yum install python-pyasn1
    ```

9. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/boot/grub/menu.lst" en un editor de texto y asegúrese de que el kernel incluye los parámetros siguientes:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.
   
   Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

   Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie.
   
   Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.
10. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
11. Instale el Agente de Linux de Azure ejecutando el comando siguiente: La versión más reciente es la 2.0.15.

    ```console
    # sudo yum install WALinuxAgent
    ```

    Tenga en cuanta que al instalar el paquete WALinuxAgent se eliminarán los paquetes NetworkManager y NetworkManager-gnome, si es que aún no se han eliminado como se indica en el paso 2.
12. No cree espacio de intercambio en el disco del SO.
    
    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

13. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

14. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

---
## <a name="oracle-linux-70-and-later"></a>Oracle Linux 7.0 y posterior
**Cambios en Oracle Linux 7**

La preparación de una máquina virtual Oracle Linux 7 para Azure es muy similar a Oracle Linux 6, aunque hay varias diferencias importantes que es necesario tener en cuenta:

* Azure admite Oracle Linux tanto con el kernel de empresa ininterrumpible (UEK) como con el kernel compatible con Red Hat. Se recomienda Oracle Linux con UEK.
* El paquete NetworkManager ya no entra en conflicto con el agente de Linux de Azure. Este paquete está instalado de manera predeterminada y recomendamos que no se quite.
* GRUB2 se utiliza ahora como cargador de arranque predeterminado; por ello, el proceso de edición de los parámetros de kernel ha cambiado (ver más adelante).
* XFS es ahora el sistema de archivos predeterminado. El sistema de archivos ext4 se puede seguir utilizando si así se desea.

**Pasos de configuración**

1. En el Administrador de Hyper-V, seleccione la máquina virtual.
2. Haga clic en **Conectar** para abrir una ventana de consola de la máquina virtual.
3. Cree un archivo llamado **network** in the `/etc/sysconfig/` que contenga el texto siguiente:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Cree un archivo llamado **ifcfg-eth0** in the `/etc/sysconfig/network-scripts/` que contenga el texto siguiente:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

5. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Asegúrese de que el servicio de red se inicie en el arranque ejecutando el comando siguiente:

    ```console
    # sudo chkconfig network on
    ```

7. Instale el paquete python-pyasn1 ejecutando el comando siguiente:

    ```console
    # sudo yum install python-pyasn1
    ```

8. Ejecute el comando siguiente para borrar los metadatos de yum actuales e instalar las actualizaciones:

    ```console 
    # sudo yum clean all
    # sudo yum -y update
    ```

9. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/etc/default/grub" en un editor de texto y edite el parámetro `GRUB_CMDLINE_LINUX` ; por ejemplo:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Así también se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores. También desactiva las convenciones de nomenclatura para los adaptadores de red en Oracle Linux 7 con el kernel de empresa ininterrumpible. Además de lo anterior, se recomienda *quitar* los parámetros siguientes:

    ```config-grub
       rhgb quiet crashkernel=auto
    ```
 
   Los arranques gráfico y silencioso no resultan útiles en un entorno de nube, donde queremos que todos los registros se envíen al puerto serie.
   
   Es posible dejar la opción `crashkernel` configurada si así se desea, pero tenga en cuenta que este parámetro reducirá la cantidad de memoria disponible en la máquina virtual en 128 MB o más, lo cual puede resultar problemático en tamaños de VM más reducidos.
10. Una vez que haya finalizado de editar "/etc/default/grub" como se indica anteriormente, ejecute el comando siguiente para volver a compilar la configuración de grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

11. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
12. Instale el Agente de Linux de Azure y sus dependencias:

    ```bash
    sudo yum install WALinuxAgent
    sudo systemctl enable waagent
    ```

13. Instalación de cloud-init para controlar el aprovisionamiento

    ```console
    yum install -y cloud-init cloud-utils-growpart gdisk hyperv-daemons

    # Configure waagent for cloud-init
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf


    echo "Adding mounts and disk_setup to init stage"
    sed -i '/ - mounts/d' /etc/cloud/cloud.cfg
    sed -i '/ - disk_setup/d' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - mounts' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - disk_setup' /etc/cloud/cloud.cfg

    echo "Allow only Azure datasource, disable fetching network setting via IMDS"
    cat > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg <<EOF
    datasource_list: [ Azure ]
    datasource:
    Azure:
        apply_network_config: False
    EOF

    if [[ -f /mnt/resource/swapfile ]]; then
    echo Removing swapfile - RHEL uses a swapfile by default
    swapoff /mnt/resource/swapfile
    rm /mnt/resource/swapfile -f
    fi

    echo "Add console log file"
    cat >> /etc/cloud/cloud.cfg.d/05_logging.cfg <<EOF

    # This tells cloud-init to redirect its stdout and stderr to
    # 'tee -a /var/log/cloud-init-output.log' so the user can see output
    # there without needing to look on the console.
    output: {all: '| tee -a /var/log/cloud-init-output.log'}
    EOF

    ```

14. Configuración de intercambio "Do not create swap space on the operating system disk" (No crear espacio de intercambio en el disco del sistema operativo).

    Anteriormente, el agente de Linux de Azure se usaba para configurar automáticamente el espacio de intercambio mediante el disco de recursos local que se asociaba a la máquina virtual después de que esta se aprovisionara en Azure. Ahora, en cambio, esto se administra mediante cloud-init; **no tiene** que usar el agente de Linux para formatear el disco de recursos, crear el archivo de intercambio y modificar los parámetros siguientes en `/etc/waagent.conf` adecuadamente:

    ```console
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```

    Si quiere montar, formatear y crear un intercambio, puede:
    * Pasarlo como una configuración de cloud-init cada vez que cree una máquina virtual
    * Usar una directiva de cloud-init preparada en la imagen que hará esto cada vez que se cree la máquina virtual:

        ```console
        cat > /etc/cloud/cloud.cfg.d/00-azure-swap.cfg << EOF
        #cloud-config
        # Generated by Azure cloud image build
        disk_setup:
          ephemeral0:
            table_type: mbr
            layout: [66, [33, 82]]
            overwrite: True
        fs_setup:
          - device: ephemeral0.1
            filesystem: ext4
          - device: ephemeral0.2
            filesystem: swap
        mounts:
          - ["ephemeral0.1", "/mnt"]
          - ["ephemeral0.2", "none", "swap", "sw", "0", "0"]
        EOF
        ```


15. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # Note: if you are migrating a specific virtual machine and do not wish to create a generalized image,
    # skip the deprovision step
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log

    # waagent -force -deprovision+user
    # rm -f ~/.bash_history
    

    # export HISTSIZE=0

    # logout
    ```

16. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

## <a name="next-steps"></a>Pasos siguientes
Ya está listo para usar el archivo .vhd de Oracle Linux para crear nuevas máquinas virtuales en Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).