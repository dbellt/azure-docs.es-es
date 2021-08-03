---
title: Uso de cloud-init en una máquina virtual Linux de Azure
description: Procedimiento para usar cloud-init con el fin de actualizar e instalar paquetes en una máquina virtual Linux con la CLI de Azure
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 06/01/2021
ms.author: cynthn
ms.subservice: cloud-init
ms.openlocfilehash: ab44e9710fc4a0a86d3225b11f7755400bca0b79
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111853305"
---
# <a name="use-cloud-init-to-update-and-install-packages-in-a-linux-vm-in-azure"></a>Usar Uso de cloud-init para instalar paquetes en una máquina virtual Linux en Azure
En este artículo se muestra el uso de [cloud-init](https://cloudinit.readthedocs.io) para actualizar paquetes en una máquina virtual Linux (VM) o en conjuntos de escalado de máquinas virtuales en el momento del aprovisionamiento en Azure. Estos scripts de cloud-init se ejecutan durante el primer arranque una vez que Azure ha aprovisionado los recursos. Para obtener más información acerca del funcionamiento nativo de cloud-init en Azure y las distribuciones de Linux compatibles, consulte la [introducción a cloud-init](using-cloud-init.md).

## <a name="update-a-vm-with-cloud-init"></a>Actualización de una máquina virtual con cloud-init
Por motivos de seguridad, es posible que desee configurar una máquina virtual para aplicar las actualizaciones más recientes en el primer arranque. Como cloud-init funciona en distintas distribuciones de Linux, no es necesario especificar `apt` ni `yum` para el administrador de paquetes. En lugar de eso, se define `package_upgrade` para permitir que el proceso cloud-init determine el mecanismo adecuado para la distribución en uso. 

En este ejemplo, usaremos Azure Cloud Shell. Para ver en acción del proceso de actualización, cree un archivo denominado *cloud_init_upgrade.txt* y pegue la configuración siguiente. 

Seleccione el botón **Probar** en el bloque de código que aparece abajo para abrir Cloud Shell. Si desea crear el archivo y ver una lista de los editores disponibles en Cloud Shell, escriba lo siguiente:

```azurecli-interactive
sensible-editor cloud_init_upgrade.txt 
```

Copie el texto siguiente y péguelo en el archivo `cloud_init_upgrade.txt`. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea.  

```yaml
#cloud-config
package_upgrade: true
packages:
- httpd
```

Antes de realizar la implementación, debe crear un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Ahora, cree una máquina virtual con [az vm create](/cli/azure/vm) y especifique el archivo cloud-init con el parámetro `--custom-data` como se indica a continuación:

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_upgrade.txt \
  --generate-ssh-keys 
```

SSH a la dirección IP pública de la máquina virtual que se muestra en la salida del comando anterior. Escriba su propia **dirección IP pública**, como se indica a continuación:

```bash
ssh <publicIpAddress>
```

Ejecute la herramienta de administración de paquetes y compruebe si hay actualizaciones.

```bash
sudo yum update
```

Como cloud-init comprobó e instaló las actualizaciones en el arranque, no debería haber ninguna actualización que aplicar.  Puede ver el proceso de actualización, el número de paquetes modificados, así como la instalación de `httpd` mediante `yum history` y revisar una salida similar a la siguiente.

```bash
Loaded plugins: fastestmirror, langpacks
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     3 | -t -y install httpd      | 2018-04-20 22:42 | Install        |    5
     2 | -t -y upgrade            | 2018-04-20 22:38 | I, U           |   65
     1 |                          | 2017-12-12 20:32 | Install        |  522
```

## <a name="next-steps"></a>Pasos siguientes
Para ejemplos de cloud-init de cambios de configuración adicionales, vea lo siguiente:
 
- [Incorporación de otro usuario de Linux a una máquina virtual](cloudinit-add-user.md)
- [Ejecución de un administrador de paquetes para actualizar los existentes durante el primer arranque](cloudinit-update-vm.md)
- [Cambio del nombre de host de la máquina virtual local](cloudinit-update-vm-hostname.md) 
- [Instalación de un paquete de aplicación, actualización de los archivos de configuración e inserción de claves](tutorial-automate-vm-deployment.md)
