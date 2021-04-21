---
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 05/07/2020
ms.author: danlep
ms.openlocfilehash: 429377cd50e83195cb1c3a422416fdb35644a28e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773483"
---
## <a name="create-a-docker-enabled-virtual-machine"></a>Creación de una máquina virtual con funcionalidad Docker

Para fines de prueba, use una máquina virtual Ubuntu con funcionalidad Docker para acceder a un registro de contenedor de Azure. Para usar la autenticación de Azure Active Directory en el registro, instale también la [CLI de Azure][azure-cli] en la máquina virtual. Si ya tiene una máquina virtual de Azure, omita este paso.

Puede usar el mismo grupo de recursos para la máquina virtual y el registro de contenedor. Esta configuración simplifica la limpieza al final, pero no es necesaria. Si elige crear un grupo de recursos independiente para la máquina virtual y la red virtual, ejecute [az group create][az-group-create]. En el ejemplo siguiente se da por sentado que ha establecido variables de entorno para el nombre del grupo de recursos y la ubicación del registro:

```azurecli
az group create --name $RESOURCE_GROUP --location $REGISTRY_LOCATION
```

Ahora, implemente una máquina virtual de Azure con Ubuntu mediante [az vm create][az-vm-create]. En el ejemplo siguiente se crea una máquina virtual denominada *myDockerVM*.

```azurecli
VM_NAME=myDockerVM

az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

La máquina virtual tarda unos minutos en crearse. Cuando se complete el comando, anote el valor `publicIpAddress` mostrado por la CLI de Azure. Use esta dirección para establecer conexiones SSH con la máquina virtual.

### <a name="install-docker-on-the-vm"></a>Instalación de Docker en la máquina virtual

Después de ejecutar la máquina virtual, establezca una conexión SSH con la máquina virtual. Reemplace *publicIpAddress* por la dirección IP pública de la máquina virtual.

```bash
ssh azureuser@publicIpAddress
```

Ejecute los siguientes comandos para instalar Docker en la máquina virtual Ubuntu:

```bash
sudo apt-get update
sudo apt install docker.io -y
```

Después de la instalación, ejecute el siguiente comando para comprobar que Docker se ejecute correctamente en la máquina virtual:

```bash
sudo docker run -it hello-world
```

Salida:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### <a name="install-the-azure-cli"></a>Instalación de la CLI de Azure

Siga los pasos del artículo [Instalación de la CLI de Azure con apt](/cli/azure/install-azure-cli-apt) para instalar la CLI de Azure en la máquina virtual de Ubuntu. Por ejemplo:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Salga de la conexión SSH.

[azure-cli]: /cli/azure/install-azure-cli
[az-vm-create]: /cli/azure/vm#az_vm_create
[az-group-create]: /cli/azure/group