---
title: Creación y carga de un VHD de SUSE Linux en Azure
description: Aprenda a crear y cargar un disco duro virtual de Azure (VHD) que contiene un sistema operativo SUSE Linux.
author: danielsollondon
ms.service: virtual-machines
ms.subservice: imaging
ms.collection: linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: 7788d7e7a8aa92d4c06b0895c5bea000e13e2d85
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713133"
---
# <a name="prepare-a-sles-or-opensuse-leap-virtual-machine-for-azure"></a>Preparación de una máquina virtual SLES u openSUSE Leap para Azure


En este artículo se supone que ya ha instalado un sistema operativo Linux SUSE u openSUSE Leap en un disco duro virtual. Existen varias herramientas para crear archivos .vhd; por ejemplo, una solución de virtualización como Hyper-V. Para obtener instrucciones, consulte [Instalación del rol de Hyper-V y configuración de una máquina Virtual](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="sles--opensuse-leap-installation-notes"></a>Notas sobre la instalación de SLES/openSUSE Leap
* Consulte también [Notas generales sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más consejos sobre la preparación de Linux para Azure.
* El formato VHDX no se admite en Azure, solo el **VHD fijo**.  Puede convertir el disco al formato VHD con el Administrador de Hyper-V o el cmdlet Convert-VHD.
* Al instalar el sistema Linux se recomienda utilizar las particiones estándar en lugar de un LVM (que a menudo viene de forma predeterminada en muchas instalaciones). De este modo se impedirá que el nombre del LVM entre en conflicto con las máquinas virtuales clonadas, especialmente si en algún momento hace falta adjuntar un disco de SO a otra máquina virtual para solucionar problemas. [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) o [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid) se pueden utilizar en discos de datos si así se prefiere.
* No cree una partición de intercambio en el disco del SO. El agente de Linux se puede configurar para crear un archivo de intercambio en el disco de recursos temporal.  Puede encontrar más información al respecto en los pasos que vienen a continuación.
* En Azure, todos los discos duros virtuales deben tener un tamaño virtual alineado con 1 MB. Al convertir un disco sin procesar en un disco duro virtual, tiene que asegurarse de que su tamaño es un múltiplo de 1 MB antes de la conversión. Para más información, consulte [Notas sobre la instalación de Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="use-suse-studio"></a>Uso de SUSE Studio
[SUSE Studio](https://studioexpress.opensuse.org/) puede crear y administrar fácilmente sus imágenes de SLES y openSUSE Leap para Hyper-V y Azure. Este es el enfoque recomendado para personalizar sus propias imágenes SLES y openSUSE Leap.

Como alternativa a la creación de su propio VHD, SUSE también publica imágenes de BYOS ("traiga su propia suscripción") para SLES en [VM Depot](https://www.microsoft.com/research/wp-content/uploads/2016/04/using-and-contributing-vms-to-vm-depot.pdf).

## <a name="prepare-suse-linux-enterprise-server-for-azure"></a>Preparación de SUSE Linux Enterprise Server para Azure
1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. Registre el sistema de SUSE Linux Enterprise para permitir que descargue actualizaciones e instale paquetes.
4. Actualice el sistema con las revisiones más recientes:

    ```console
    # sudo zypper update
    ```
    
5. Instale el agente Linux de Azure y cloud-init

    ```console
    # SUSEConnect -p sle-module-public-cloud/15.2/x86_64  (SLES 15 SP2)
    # sudo zypper refresh
    # sudo zypper install python-azure-agent
    # sudo zypper install cloud-init
    ```

6. Habilite waagent y cloud-init para que se inicien en el arranque

    ```console
    # sudo chkconfig waagent on
    # systemctl enable cloud-init-local.service
    # systemctl enable cloud-init.service
    # systemctl enable cloud-config.service
    # systemctl enable cloud-final.service
    # systemctl daemon-reload
    # cloud-init clean
    ```

7. Actualice la configuración de waagent y cloud-init

    ```console
    # sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    # sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf

    # sudo sh -c 'printf "datasource:\n  Azure:" > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg'
    # sudo sh -c 'printf "reporting:\n  logging:\n    type: log\n  telemetry:\n    type: hyperv" > /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg'
    ```

8. Edite el archivo /etc/default/grub para asegurarse de que los registros de la consola se envíen al puerto serie y, luego, actualice el archivo de configuración principal con grub2-mkconfig -o /boot/grub2/grub.cfg

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```
    Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores.
    
9. Asegúrese de que el archivo /etc/fstab haga referencia al disco mediante su UUID (by-uuid)
         
10. Modifique las reglas udev para impedir que se generen reglas estáticas para las interfaces Ethernet. Estas reglas pueden causar problemas al clonar una máquina virtual en Microsoft Azure o Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```
   
11. Se recomienda editar el archivo "/etc/sysconfig/network/dhcp" y cambiar el parámetro `DHCLIENT_SET_HOSTNAME` por lo siguiente:

    ```config
    DHCLIENT_SET_HOSTNAME="no"
    ```

12. En "/etc/sudoers", convierta en comentario o quite las líneas siguientes, si existen:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

13. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque. Este es normalmente el valor predeterminado.

14. Configuración de intercambio
 
    No cree espacio de intercambio en el disco del sistema operativo.

    Anteriormente, el agente de Linux de Azure se usaba para configurar automáticamente un espacio de intercambio mediante el disco de recursos local que se adjunta a la máquina virtual, después de que la máquina virtual se aprovisione en Azure. Ahora, en cambio, esto se administra mediante cloud-init; **no tiene** que usar el agente de Linux para formatear el disco de recursos, crear el archivo de intercambio y modificar los parámetros siguientes en `/etc/waagent.conf` adecuadamente:

    ```console
    # sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    # sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```

    Si quiere montar, formatear y crear un intercambio, puede:
    * Pasarlo como una configuración de cloud-init cada vez que cree una máquina virtual.
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
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log

    # waagent -force -deprovision+user
    # rm -f ~/.bash_history
    

    # export HISTSIZE=0

    # logout
    ```
16. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

---
## <a name="prepare-opensuse-152"></a>Preparación de openSUSE 15.2 y versiones posteriores
1. Seleccione la máquina virtual en el panel central del Administrador de Hyper-V.
2. Haga clic en **Conectar** para abrir la ventana de la máquina virtual.
3. En el shell, ejecute el comando '`zypper lr`'. Si este comando devuelve una salida similar a la siguiente, los repositorios están configurados según lo esperado y no es necesario ningún ajuste (tenga en cuenta que los números de versión pueden variar):

   | # | Alias                 | Nombre                  | habilitado | Actualizar
   | - | :-------------------- | :-------------------- | :------ | :------
   | 1 | Cloud:Tools_15.2      | Cloud:Tools_15.2      | Sí     | Sí
   | 2 | openSUSE_15.2_OSS     | openSUSE_15.2_OSS     | Sí     | Sí
   | 3 | openSUSE_15.2_Updates | openSUSE_15.2_Updates | Sí     | Sí

    Si el comando devuelve el mensaje "No repositories defined...", utilice los comandos siguientes para agregar estos repositorios:

    ```console
    # sudo zypper ar -f http://download.opensuse.org/repositories/Cloud:Tools/openSUSE_15.2 Cloud:Tools_15.2
    # sudo zypper ar -f https://download.opensuse.org/distribution/15.2/repo/oss openSUSE_15.2_OSS
    # sudo zypper ar -f http://download.opensuse.org/update/15.2 openSUSE_15.2_Updates
    ```

    Puede verificar entonces que se han agregado los repositorios al volver a ejecutar el comando '`zypper lr`'. En caso de que no haya un repositorio de actualizaciones relevante habilitado, habilítelo con el comando siguiente:

    ```console
    # sudo zypper mr -e [NUMBER OF REPOSITORY]
    ```

4. Actualice el kernel a la versión más reciente disponible:

    ```console
    # sudo zypper up kernel-default
    ```

    O para actualizar el sistema con todos los parches más recientes:

    ```console
    # sudo zypper update
    ```

5. Instale el Agente de Linux de Azure.

    ```console
    # sudo zypper install WALinuxAgent
    ```

6. Modifique la línea de arranque de kernel de su configuración grub para que incluya parámetros de kernel adicionales para Azure. Para ello, abra "/boot/grub/menu.lst" en un editor de texto y asegúrese de que el kernel predeterminado incluye los parámetros siguientes:

    ```config-grub
     console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Así se asegurará de que todos los mensajes de la consola se envían al primer puerto serie, lo que puede ayudar al soporte técnico de Azure con los problemas de depuración de errores. Además, elimine los parámetros siguientes de la línea de arranque de kernel, si es que están:

    ```config-grub
     libata.atapi_enabled=0 reserve=0x1f0,0x8
    ```

7. Se recomienda editar el archivo "/etc/sysconfig/network/dhcp" y cambiar el parámetro `DHCLIENT_SET_HOSTNAME` por lo siguiente:

    ```config
     DHCLIENT_SET_HOSTNAME="no"
    ```

8. **Importante:** En "/etc/sudoers", convierta en comentario o quite las líneas siguientes, si existen:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

9. Asegúrese de que el servidor SSH se haya instalado y configurado para iniciarse en el tiempo de arranque.  Este es normalmente el valor predeterminado.
10. No cree espacio de intercambio en el disco del SO.

    El Agente de Linux de Azure puede configurar automáticamente un espacio de intercambio utilizando el disco de recursos local que se adjunta a la máquina virtual después de aprovisionarse en Azure. Tenga en cuenta que el disco de recursos local es un disco *temporal* que debe vaciarse cuando la máquina virtual se desaprovisiona. Después de instalar el Agente de Linux de Azure (consulte el paso anterior), modifique apropiadamente los parámetros siguientes en /etc/waagent.conf:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

11. Ejecute los comandos siguientes para desaprovisionar la máquina virtual y prepararla para aprovisionarse en Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

12. Asegúrese de que el Agente de Linux de Azure se ejecute al inicio:

    ```console
    # sudo systemctl enable waagent.service
    ```

13. Haga clic en **Acción -> Apagar** en el Administrador de Hyper-V. El VHD de Linux ya está listo para cargarse en Azure.

## <a name="next-steps"></a>Pasos siguientes
Ya está listo para usar el disco duro virtual de SUSE para crear nuevas máquinas virtuales de Azure. Si es la primera vez que carga el archivo .vhd en Azure, vea [Crear una VM Linux a partir de un disco personalizado](upload-vhd.md#option-1-upload-a-vhd).
