---
title: Uso de cloud-init para configurar una partición de intercambio en una máquina virtual Linux
description: Procedimiento para usar cloud-init con el fin de configurar una partición de intercambio en una máquina virtual Linux con la CLI de Azure
author: mimckitt
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: mimckitt
ms.subservice: cloud-init
ms.openlocfilehash: bc55bf12b766002fff7fda45af0d802164a2b503
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784132"
---
# <a name="use-cloud-init-to-configure-a-swap-partition-on-a-linux-vm"></a>Uso de cloud-init para configurar una partición de intercambio en una máquina virtual Linux
En este artículo se explica cómo usar [cloud-init](https://cloudinit.readthedocs.io) para configurar la partición de intercambio en diversas distribuciones de Linux. La partición de intercambio tradicionalmente se ha configurado mediante el agente Linux (WALA) en función de las distribuciones que necesitan una.  En este documento se detalla el proceso de creación de la partición de intercambio a petición durante el tiempo de aprovisionamiento mediante cloud-init.  Para obtener más información acerca del funcionamiento nativo de cloud-init en Azure y las distribuciones de Linux compatibles, consulte la [introducción a cloud-init](using-cloud-init.md).

## <a name="create-swap-partition-for-ubuntu-based-images"></a>Creación de la partición de intercambio para imágenes basadas en Ubuntu
De forma predeterminada en Azure, las imágenes de la galería de Ubuntu no crean particiones de intercambio. Para habilitar la configuración de la partición de intercambio durante el tiempo de aprovisionamiento de la máquina virtual mediante cloud-ini, vea el [documento AzureSwapPartitions](https://wiki.ubuntu.com/AzureSwapPartitions) en la wiki de Ubuntu.

## <a name="create-swap-partition-for-red-hat-and-centos-based-images"></a>Creación de la partición de intercambio para imágenes basadas en CentOS y Red Hat

En el shell actual, cree un archivo denominado *cloud_init_swappart.txt* y pegue la configuración siguiente. Para este ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor cloud_init_swappart.txt` para crear el archivo y ver una lista de editores disponibles. Elija el número 1 para utilizar el editor **nano**. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea.  

```yaml
#cloud-config
disk_setup:
  ephemeral0:
    table_type: gpt
    layout: [66, [33,82]]
    overwrite: true
fs_setup:
  - device: ephemeral0.1
    filesystem: ext4
  - device: ephemeral0.2
    filesystem: swap
mounts:
  - ["ephemeral0.1", "/mnt"]
  - ["ephemeral0.2", "none", "swap", "sw,nofail,x-systemd.requires=cloud-init.service", "0", "0"]
```

El montaje se crea con la opción `nofail` para asegurarse de que el arranque continuará incluso si el montaje no se completa correctamente.

Antes de implementar esta imagen, debe crear un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Ahora, cree una máquina virtual con [az vm create](/cli/azure/vm) y especifique el archivo cloud-init con `--custom-data cloud_init_swappart.txt` como se indica a continuación:

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_swappart.txt \
  --generate-ssh-keys 
```

## <a name="verify-swap-partition-was-created"></a>Comprobación de que se ha creado la partición de intercambio
SSH a la dirección IP pública de la máquina virtual que se muestra en la salida del comando anterior. Escriba su propia **dirección IP pública**, como se indica a continuación:

```bash
ssh <publicIpAddress>
```

Una vez haya accedido mediante SSH a la máquina virtual, compruebe si se ha creado la partición de intercambio.

```bash
swapon -s
```

La salida de este comando debe tener este aspecto:

```text
Filename                Type        Size    Used    Priority
/dev/sdb2  partition   2494440 0   -1
```

> [!NOTE] 
> Si tiene una imagen de Azure existente con una partición de intercambio configurada y quiere cambiar la configuración de la partición de intercambio para nuevas imágenes, debe quitar la partición de intercambio existente. Consulte el documento sobre cómo personalizar imágenes para aprovisionar mediante cloud-init para obtener más información.

## <a name="next-steps"></a>Pasos siguientes
Para ejemplos de cloud-init de cambios de configuración adicionales, vea lo siguiente:
 
- [Incorporación de otro usuario de Linux a una máquina virtual](cloudinit-add-user.md)
- [Ejecución de un administrador de paquetes para actualizar los existentes durante el primer arranque](cloudinit-update-vm.md)
- [Cambio del nombre de host de la máquina virtual local](cloudinit-update-vm-hostname.md) 
- [Instalación de un paquete de aplicación, actualización de los archivos de configuración e inserción de claves](tutorial-automate-vm-deployment.md)
