---
title: Cambio del tamaño de un disco de SO con una partición GPT
description: En este artículo se proporcionan instrucciones sobre cómo cambiar el tamaño de un disco del sistema operativo que tiene una partición de tabla de particiones GUID (GPT) en Linux.
services: virtual-machines
ms.topic: article
author: kailashmsft
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.workload: infrastructure-services
ms.devlang: azurecli
ms.date: 05/03/2020
ms.author: kaib
ms.custom: seodec18
ms.openlocfilehash: 0db79728bbb963aa360743afc70aecc213bfb7bc
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107011688"
---
# <a name="resize-an-os-disk-that-has-a-gpt-partition"></a>Cambio del tamaño de un disco de SO con una partición GPT

> [!NOTE]
> Este artículo se aplica únicamente a los discos de SO que tienen una partición de tabla de particiones GUID (GPT).

En este artículo se describe cómo aumentar el tamaño de un disco del SO que tiene una partición GPT en Linux. 

## <a name="identify-whether-the-os-disk-has-an-mbr-or-gpt-partition"></a>Identificar si el disco del SO tiene una partición MBR o GPT

Use el comando `parted` para identificar si la partición de disco se ha creado con una partición de registro de arranque maestro (MBR) o una partición GPT.

### <a name="mbr-partition"></a>Partición MBR

En la salida siguiente, la **tabla de particiones** muestra el valor **msdos**. Este valor identifica una partición MBR.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Number  Start   End     Size    Type     File system  Flags
1       1049kB  525MB   524MB   primary  ext4         boot
2       525MB   34.4GB  33.8GB  primary  ext4
[user@myvm ~]#
```

### <a name="gpt-partition"></a>Partición GPT

En la salida siguiente, la **tabla de particiones** muestra el valor **gpt**. Este valor identifica una partición GPT.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name                  Flags
1       1049kB  525MB   524MB   fat16        EFI System Partition  boot
2       525MB   1050MB  524MB   xfs
3       1050MB  1052MB  2097kB                                     bios_grub
4       1052MB  68.7GB  67.7GB                                     lvm
```

Si la máquina virtual (VM) tiene una partición GPT en el disco del SO, aumente el tamaño del disco de SO.

## <a name="increase-the-size-of-the-os-disk"></a>Aumento de tamaño del disco de SO

Las instrucciones siguientes se aplican a las distribuciones aprobadas por Linux.

> [!NOTE]
> Antes de continuar, haga una copia de seguridad de la VM o realice una instantánea del disco de SO.

### <a name="ubuntu"></a>Ubuntu

Para aumentar el tamaño del disco de SO en Ubuntu 16.*x* y 18.*x*:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Reinicie la VM y, a continuación, inicie sesión en la VM como usuario **raíz**.
1. Compruebe que el disco de SO ahora presenta un tamaño mayor del sistema de archivos.

En el ejemplo siguiente, se ha cambiado el tamaño del disco de SO del portal a 100 GB. El sistema de archivos **/dev/sda1** montado en **/** muestra ahora 97 GB.

```
user@myvm:~# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  314M     0  314M   0% /dev
tmpfs          tmpfs      65M  2.3M   63M   4% /run
/dev/sda1      ext4       97G  1.8G   95G   2% /
tmpfs          tmpfs     324M     0  324M   0% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     324M     0  324M   0% /sys/fs/cgroup
/dev/sda15     vfat      105M  3.6M  101M   4% /boot/efi
/dev/sdb1      ext4       20G   44M   19G   1% /mnt
tmpfs          tmpfs      65M     0   65M   0% /run/user/1000
user@myvm:~#
```

### <a name="suse"></a>SUSE

Para aumentar el tamaño del disco de SO en SUSE 12 SP4, SUSE SLES 12 para SAP, SUSE SLES 15 y SUSE SLES 15 para SAP:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Reinicie la VM.

Cuando se haya reiniciado la máquina virtual, realice estos pasos:

1. Acceda a la VM como usuario **raíz** con este comando:

   ```
   # sudo -i
   ```

1. Use el siguiente comando para instalar el paquete **growpart**, que usará para cambiar el tamaño de la partición:

   ```
   # zypper install growpart
   ```

1. Use el comando `lsblk` para encontrar la partición montada en la raíz del sistema de archivos ( **/** ). En este caso, vemos que la partición 4 del disco **sda** del dispositivo está montada en **/** :

   ```
   # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 28.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Cambie el tamaño de la partición necesaria con el comando `growpart` y el número de partición que se especificó en el paso anterior:

   ```
   # growpart /dev/sda 4
   CHANGED: partition=4 start=3151872 old: size=59762655 end=62914527 new: size=97511391 end=100663263
   ```

1. Vuelva a ejecutar el comando `lsblk` para comprobar si ha aumentado la partición.

   El resultado siguiente muestra que se ha cambiado el tamaño de la partición **/dev/sda4** a 46,5 GB.
   
   ```
   linux:~ # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 46.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Identifique el tipo de sistema de archivos del disco del sistema operativo mediante el comando `lsblk` con la marca `-f`:

   ```
   linux:~ # lsblk -f
   NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1
   ├─sda2 vfat   EFI   AC67-D22D                            /boot/efi
   ├─sda3 xfs    BOOT  5731a128-db36-4899-b3d2-eb5ae8126188 /boot
   └─sda4 xfs    ROOT  70f83359-c7f2-4409-bba5-37b07534af96 /
   sdb
   └─sdb1 ext4         8c4ca904-cd93-4939-b240-fb45401e2ec6 /mnt/resource
   ```

1. Según el tipo del sistema de archivos, use los comandos adecuados para cambiar el tamaño del sistema de archivos.
   
   Use este comando para **xfs**:
   
   ```
   #xfs_growfs /
   ```
   
   Salida de ejemplo:
   
   ```
   linux:~ # xfs_growfs /
   meta-data=/dev/sda4              isize=512    agcount=4, agsize=1867583 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0 rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=7470331, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3647, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7470331 to 12188923
   ```
   
   Use este comando para **ext4**:
   
   ```
   #resize2fs /dev/sda4
   ```
   
1. Compruebe el aumento de tamaño del sistema de archivos para **df -Th** con este comando:
   
   ```
   #df -Thl
   ```
   
   Salida de ejemplo:
   
   ```
   linux:~ # df -Thl
   Filesystem     Type      Size  Used Avail Use% Mounted on
   devtmpfs       devtmpfs  445M  4.0K  445M   1% /dev
   tmpfs          tmpfs     458M     0  458M   0% /dev/shm
   tmpfs          tmpfs     458M   14M  445M   3% /run
   tmpfs          tmpfs     458M     0  458M   0% /sys/fs/cgroup
   /dev/sda4      xfs        47G  2.2G   45G   5% /
   /dev/sda3      xfs      1014M   86M  929M   9% /boot
   /dev/sda2      vfat      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1      ext4      3.9G   16M  3.7G   1% /mnt/resource
   tmpfs          tmpfs      92M     0   92M   0% /run/user/1000
   tmpfs          tmpfs      92M     0   92M   0% /run/user/490
   ```
   
   En el ejemplo anterior, podemos ver que se ha aumentado el tamaño del sistema de archivos para el disco de SO.

### <a name="rhel-with-lvm"></a>RHEL con LVM

1. Acceda a la VM como usuario **raíz** con este comando:

   ```bash
   [root@dd-rhel7vm ~]# sudo -i
   ```

1. Use el comando `lsblk` para determinar cuál volumen lógico (LV) está montado en la raíz del sistema de archivos ( **/** ). En este caso, se ve que **rootvg-rootlv** está montado en **/** . Si quiere usar otro sistema de archivos, sustituya el volumen lógico y el punto de montaje de este artículo.

   ```shell
   [root@dd-rhel7vm ~]# lsblk -f
   NAME                  FSTYPE      LABEL   UUID                                   MOUNTPOINT
   fd0
   sda
   ├─sda1                vfat                C13D-C339                              /boot/efi
   ├─sda2                xfs                 8cc4c23c-fa7b-4a4d-bba8-4108b7ac0135   /boot
   ├─sda3
   └─sda4                LVM2_member         zx0Lio-2YsN-ukmz-BvAY-LCKb-kRU0-ReRBzh
      ├─rootvg-tmplv      xfs                 174c3c3a-9e65-409a-af59-5204a5c00550   /tmp
      ├─rootvg-usrlv      xfs                 a48dbaac-75d4-4cf6-a5e6-dcd3ffed9af1   /usr
      ├─rootvg-optlv      xfs                 85fe8660-9acb-48b8-98aa-bf16f14b9587   /opt
      ├─rootvg-homelv     xfs                 b22432b1-c905-492b-a27f-199c1a6497e7   /home
      ├─rootvg-varlv      xfs                 24ad0b4e-1b6b-45e7-9605-8aca02d20d22   /var
      └─rootvg-rootlv     xfs                 4f3e6f40-61bf-4866-a7ae-5c6a94675193   /
   ```

1. Compruebe si hay espacio disponible en el grupo de volúmenes (VG) de LVM que contiene la partición raíz. Si hay espacio disponible, vaya al paso 12.

   ```bash
   [root@dd-rhel7vm ~]# vgdisplay rootvg
   --- Volume group ---
   VG Name               rootvg
   System ID
   Format                lvm2
   Metadata Areas        1
   Metadata Sequence No  7
   VG Access             read/write
   VG Status             resizable
   MAX LV                0
   Cur LV                6
   Open LV               6
   Max PV                0
   Cur PV                1
   Act PV                1
   VG Size               <63.02 GiB
   PE Size               4.00 MiB
   Total PE              16132
   Alloc PE / Size       6400 / 25.00 GiB
   Free  PE / Size       9732 / <38.02 GiB
   VG UUID               lPUfnV-3aYT-zDJJ-JaPX-L2d7-n8sL-A9AgJb
   ```

   En este ejemplo, la línea **Free PE / Size** muestra que hay 38,02 GB disponibles en el grupo de volúmenes. No es necesario cambiar el tamaño del disco antes de agregar espacio al grupo de volúmenes.

1. Para aumentar el tamaño del disco de SO en RHEL 7.*x* con LVM:

   1. Pare la VM.
   1. Aumente el tamaño del disco de SO desde el portal.
   1. Inicie la máquina virtual.

1. Una vez reiniciada la VM, complete los pasos siguientes:

   - Instale el paquete **cloud-utils-growpart** para proporcionar el comando **growpart**, que es necesario para aumentar el tamaño del disco del sistema operativo, y el controlador gdisk para los diseños de disco GPT. Estos paquetes están preinstalados en la mayoría de las imágenes del marketplace.

   ```bash
   [root@dd-rhel7vm ~]# yum install cloud-utils-growpart gdisk
   ```

1. Determine el disco y la partición que contienen los volúmenes físicos (PV) de LVM del grupo de volúmenes (VG) denominado **rootvg** con el comando `pvscan`. Tome nota del tamaño y el espacio disponible indicados entre corchetes ( **[** y **]** ).

   ```bash
   [root@dd-rhel7vm ~]# pvscan
     PV /dev/sda4   VG rootvg          lvm2 [<63.02 GiB / <38.02 GiB free]
   ```

1. Compruebe el tamaño de la partición mediante `lsblk`. 

   ```bash
   [root@dd-rhel7vm ~]# lsblk /dev/sda4
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  63G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Expanda la partición que contiene este volumen físico mediante `growpart`, el nombre de dispositivo y el número de partición. De esta manera, se expande la partición especificada para usar todo el espacio disponible contiguo en el dispositivo.

   ```bash
   [root@dd-rhel7vm ~]# growpart /dev/sda 4
   CHANGED: partition=4 start=2054144 old: size=132161536 end=134215680 new: size=199272414 end=201326558
   ```

1. Compruebe de nuevo que se ha cambiado el tamaño de la partición al esperado con el comando `lsblk`. Observe que, en el ejemplo, **sda4** ha pasado de 63 GB a 95 GB.

   ```bash
   [root@dd-rhel7vm ~]# lsblk /dev/sda4
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  95G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Expanda el volumen físico para usar el resto de la partición recién expandida:

   ```bash
   [root@dd-rhel7vm ~]# pvresize /dev/sda4
   Physical volume "/dev/sda4" changed
   1 physical volume(s) resized or updated / 0 physical volume(s) not resized
   ```

1. Compruebe que el nuevo tamaño del volumen físico es el esperado; para ello, compárelo con los valores originales **[size / free]** :

   ```bash
   [root@dd-rhel7vm ~]# pvscan
   PV /dev/sda4   VG rootvg          lvm2 [<95.02 GiB / <70.02 GiB free]
   ```

1. Expanda el volumen lógico deseado (LV) en la cantidad que quiera. No es necesario que el valor sea igual que todo el espacio disponible en el grupo de volúmenes. En el ejemplo siguiente, se cambia el tamaño de **/dev/mapper/rootvg-rootlv** de 2 GB a 12 GB (un aumento de 10 GB). Este comando también cambiará el tamaño del sistema de archivos.

   ```bash
   [root@dd-rhel7vm ~]# lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   ```

   Salida de ejemplo:

   ```bash
   [root@dd-rhel7vm ~]# lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   Size of logical volume rootvg/rootlv changed from 2.00 GiB (512 extents) to 12.00 GiB (3072 extents).
   Logical volume rootvg/rootlv successfully resized.
   meta-data=/dev/mapper/rootvg-rootlv isize=512    agcount=4, agsize=131072 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=524288, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 524288 to 3145728
   ```

1. El comando `lvresize` llama automáticamente al comando de cambio tamaño adecuado para el sistema de archivos del volumen lógico. Compruebe si **/dev/mapper/rootvg-rootlv**, que está montado en **/** , tiene un mayor tamaño del sistema de archivos mediante este comando:

   ```shell
   [root@dd-rhel7vm ~]# df -Th /
   ```

   Salida de ejemplo:

   ```shell
   [root@dd-rhel7vm ~]# df -Th /
   Filesystem                Type  Size  Used Avail Use% Mounted on
   /dev/mapper/rootvg-rootlv xfs    12G   71M   12G   1% /
   [root@dd-rhel7vm ~]#
   ```

> [!NOTE]
> Para usar el mismo procedimiento para cambiar el tamaño de cualquier otro volumen lógico, cambie el nombre del mismo en el paso 12.


### <a name="rhel-raw"></a>RAW con Red Hat Enterprise Linux

Para aumentar el tamaño del disco del sistema operativo en una partición RAW de RHEL:

1. Pare la VM.
1. Aumente el tamaño del disco de SO desde el portal.
1. Inicie la máquina virtual.

Cuando se haya reiniciado la máquina virtual, realice estos pasos:

1. Acceda a la VM como usuario **raíz** con el comando siguiente:

   ```bash
   [root@dd-rhel7vm ~]# sudo -i
   ```

1. Una vez reiniciada la VM, complete los pasos siguientes:

   - Instale el paquete **cloud-utils-growpart** para proporcionar el comando **growpart**, que es necesario para aumentar el tamaño del disco del sistema operativo, y el controlador gdisk para los diseños de disco GPT. Este paquete está preinstalado en la mayoría de las imágenes del marketplace.

   ```bash
   [root@dd-rhel7vm ~]# yum install cloud-utils-growpart gdisk
   ```

1. Use el comando **lsblk -f** para comprobar la partición y el tipo de sistema de archivos que contiene la partición raíz ( **/** ):

   ```bash
   [root@vm-dd-cent7 ~]# lsblk -f
   NAME    FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1  xfs          2a7bb59d-6a71-4841-a3c6-cba23413a5d2 /boot
   ├─sda2  xfs          148be922-e3ec-43b5-8705-69786b522b05 /
   ├─sda14
   └─sda15 vfat         788D-DC65                            /boot/efi
   sdb
   └─sdb1  ext4         923f51ff-acbd-4b91-b01b-c56140920098 /mnt/resource
   ```

1. Para la comprobación, empiece por mostrar la tabla de particiones del disco sda con **gdisk**. En este ejemplo, vemos un disco de 48 GB con la partición 2 con 29,0 GiB. El disco se amplió de 30 GB a 48 GB en el Azure Portal.

   ```bash
   [root@vm-dd-cent7 ~]# gdisk -l /dev/sda
   GPT fdisk (gdisk) version 0.8.10

   Partition table scan:
   MBR: protective
   BSD: not present
   APM: not present
   GPT: present

   Found valid GPT with protective MBR; using GPT.
   Disk /dev/sda: 100663296 sectors, 48.0 GiB
   Logical sector size: 512 bytes
   Disk identifier (GUID): 78CDF84D-9C8E-4B9F-8978-8C496A1BEC83
   Partition table holds up to 128 entries
   First usable sector is 34, last usable sector is 62914526
   Partitions will be aligned on 2048-sector boundaries
   Total free space is 6076 sectors (3.0 MiB)

   Number  Start (sector)    End (sector)  Size       Code  Name
      1         1026048         2050047   500.0 MiB   0700
      2         2050048        62912511   29.0 GiB    0700
   14            2048           10239   4.0 MiB     EF02
   15           10240         1024000   495.0 MiB   EF00  EFI System Partition
   ```

1. Expanda la partición para la raíz, en este caso sda2, con el comando **growpart**. El uso de este comando expande la partición para usar todo el espacio contiguo del disco.

   ```bash
   [root@vm-dd-cent7 ~]# growpart /dev/sda 2
   CHANGED: partition=2 start=2050048 old: size=60862464 end=62912512 new: size=98613214 end=100663262
   ```

1. Ahora vuelva a imprimir la nueva tabla de particiones con **gdisk**.  Observe que la partición 2 se ha expandido a 47,0 GiB:

   ```bash
   [root@vm-dd-cent7 ~]# gdisk -l /dev/sda
   GPT fdisk (gdisk) version 0.8.10

   Partition table scan:
   MBR: protective
   BSD: not present
   APM: not present
   GPT: present

   Found valid GPT with protective MBR; using GPT.
   Disk /dev/sda: 100663296 sectors, 48.0 GiB
   Logical sector size: 512 bytes
   Disk identifier (GUID): 78CDF84D-9C8E-4B9F-8978-8C496A1BEC83
   Partition table holds up to 128 entries
   First usable sector is 34, last usable sector is 100663262
   Partitions will be aligned on 2048-sector boundaries
   Total free space is 4062 sectors (2.0 MiB)

   Number  Start (sector)    End (sector)  Size       Code  Name
      1         1026048         2050047   500.0 MiB   0700
      2         2050048       100663261   47.0 GiB    0700
   14            2048           10239   4.0 MiB     EF02
   15           10240         1024000   495.0 MiB   EF00  EFI System Partition
   ```

1. Expanda el sistema de archivos en la partición con **xfs_growfs**, que es adecuado para un sistema RedHat estándar generado por el marketplace:

   ```bash
   [root@vm-dd-cent7 ~]# xfs_growfs /
   meta-data=/dev/sda2              isize=512    agcount=4, agsize=1901952 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=7607808, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3714, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7607808 to 12326651
   ```

1. Use el comando **df** para comprobar que el nuevo tamaño se vea reflejado:

   ```bash
   [root@vm-dd-cent7 ~]# df -hl
   Filesystem      Size  Used Avail Use% Mounted on
   devtmpfs        452M     0  452M   0% /dev
   tmpfs           464M     0  464M   0% /dev/shm
   tmpfs           464M  6.8M  457M   2% /run
   tmpfs           464M     0  464M   0% /sys/fs/cgroup
   /dev/sda2        48G  2.1G   46G   5% /
   /dev/sda1       494M   65M  430M  13% /boot
   /dev/sda15      495M   12M  484M   3% /boot/efi
   /dev/sdb1       3.9G   16M  3.7G   1% /mnt/resource
   tmpfs            93M     0   93M   0% /run/user/1000
   ```
