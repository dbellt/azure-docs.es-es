---
title: Autenticación con una identidad administrada
description: Proporcione acceso a las imágenes en el registro de contenedor privado mediante una identidad de Azure administrada que haya asignado el usuario o el sistema.
ms.topic: article
ms.date: 01/16/2019
ms.openlocfilehash: 213f49356fdc2444f8bc2cb4635e96015aff0a61
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781548"
---
# <a name="use-an-azure-managed-identity-to-authenticate-to-an-azure-container-registry"></a>Use la identidad administrada de Azure para autenticarse en Azure Container Registry 

Use una [instancia de Managed Identities for Azure Resources](../active-directory/managed-identities-azure-resources/overview.md) para autenticarse en Azure Container Registry desde otro recurso de Azure, sin necesidad de proporcionar o administrar las credenciales del registro. Por ejemplo, configure una identidad administrada asignada por un usuario o por el sistema en una VM de Linux para acceder a las imágenes de contenedor desde el registro de contenedor tan fácilmente como cuando usa un registro público. O bien, configure un clúster de Azure Kubernetes Service para que use su [identidad administrada](../aks/use-managed-identity.md) para extraer imágenes de contenedor de Azure Container Registry para implementaciones de pod.

En este artículo, conocerá mejor las identidades administradas y aprenderá a realizar las siguientes tareas:

> [!div class="checklist"]
> * Habilitación de una identidad asignada por el usuario o asignada por el sistema en una VM de Azure.
> * Concesión del acceso de identidad a una instancia de Azure Container Registry.
> * Uso de la identidad administrada para acceder al registro y extraer una imagen de contenedor. 

Para crear los recursos de Azure, en este artículo se requiere que ejecute la versión de la CLI de Azure 2.0.55 o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

Para configurar un registro de contenedor e insertar una imagen de contenedor en él, también debe tener Docker instalado localmente. Docker proporciona paquetes que permiten configurar Docker fácilmente en cualquier sistema [macOS][docker-mac], [Windows][docker-windows] o [Linux][docker-linux].

## <a name="why-use-a-managed-identity"></a>¿Por qué usar una identidad administrada?

Si no está familiarizado con la característica Managed Identities for Azure Resources, consulte esta [introducción](../active-directory/managed-identities-azure-resources/overview.md).

Una vez que haya configurado los recursos de Azure seleccionados con una identidad administrada, conceda a la identidad el acceso a otro recurso, tal como haría con cualquier entidad de seguridad. Por ejemplo, asigne a una identidad administrada un rol con permisos para extraer, insertar y extraer o de otro tipo en un registro privado de Azure. (Para obtener una lista completa de los roles de registros, consulte [Roles y permisos de Azure Container Registry](container-registry-roles.md)). Puede permitir que una identidad acceda a uno o más recursos.

Después, use esta identidad para autenticarse en cualquier [servicio que admita la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication), sin necesidad de tener credenciales en el código. Según el escenario, elija cómo autenticarse mediante la identidad administrada. Para usar la identidad con el fin de acceder a Azure Container Registry desde una máquina virtual, autentíquese con Azure Resource Manager. 

> [!NOTE]
> Actualmente, los servicios como Azure Web App for Containers o Azure Container Instances no pueden usar su identidad administrada para autenticarse con Azure Container Registry al extraer una imagen de contenedor para implementar el propio recurso de contenedor. La identidad solo está disponible después de que el contenedor esté en ejecución. Para implementar estos recursos mediante imágenes de Azure Container Registry, se recomienda un método de autenticación diferente como [entidad de servicio](container-registry-auth-service-principal.md).

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

Si aún no tiene una instancia de Azure Container Registry, cree un registro e inserte una imagen de contenedor de ejemplo en él. Para obtener instrucciones, consulte [Guía de inicio rápido: Creación de un registro de contenedor privado con la CLI de Azure](container-registry-get-started-azure-cli.md).

En este artículo, se supone que tiene la imagen de contenedor `aci-helloworld:v1` almacenada en el registro. Los ejemplos utilizan el nombre de registro *myContainerRegistry*. Reemplácelo con los nombres de su registro e imagen en pasos posteriores.

## <a name="create-a-docker-enabled-vm"></a>Creación de una máquina virtual con funcionalidad Docker

Cree una máquina virtual de Ubuntu con funcionalidad Docker. También deberá instalar la [CLI de Azure](/cli/azure/install-azure-cli) en la máquina virtual. Si ya tiene una máquina virtual de Azure, omita el paso para crear la máquina virtual.

Implemente una máquina virtual de Ubuntu predeterminada con [az vm create][az-vm-create]. En el ejemplo siguiente, se crea una máquina virtual llamada *myDockerVM* en el grupo de recursos existente llamado *myResourceGroup*:

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myDockerVM \
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

Ejecute el comando siguiente para instalar Docker en la máquina virtual:

```bash
sudo apt update
sudo apt install docker.io -y
```

Después de la instalación, ejecute el siguiente comando para comprobar que Docker se ejecute correctamente en la máquina virtual:

```bash
sudo docker run -it mcr.microsoft.com/hello-world
```

Salida:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### <a name="install-the-azure-cli"></a>Instalación de la CLI de Azure

Siga los pasos del artículo [Instalación de la CLI de Azure con apt](/cli/azure/install-azure-cli-apt) para instalar la CLI de Azure en la máquina virtual de Ubuntu. Para este artículo, asegúrese de instalar la versión 2.0.55 o una versión posterior.

Salga de la sesión SSH.

## <a name="example-1-access-with-a-user-assigned-identity"></a>Ejemplo 1: acceso con una identidad asignada por el usuario

### <a name="create-an-identity"></a>Creación de una identidad

Cree una identidad en la suscripción con el comando [az identity create](/cli/azure/identity#az_identity_create). Puede usar el mismo grupo de recursos que usó anteriormente para crear el registro de contenedor o la máquina virtual, o usar uno diferente.

```azurecli-interactive
az identity create --resource-group myResourceGroup --name myACRId
```

Para configurar la identidad en los pasos siguientes, use el comando [az identity show][az_identity_show] para almacenar en variables los id. de recurso y de entidad de servicio de la entidad.

```azurecli
# Get resource ID of the user-assigned identity
userID=$(az identity show --resource-group myResourceGroup --name myACRId --query id --output tsv)

# Get service principal ID of the user-assigned identity
spID=$(az identity show --resource-group myResourceGroup --name myACRId --query principalId --output tsv)
```

Dado que necesita el identificador de la identidad en un paso posterior para iniciar sesión en la CLI desde una máquina virtual, use el siguiente comando para mostrar el valor en pantalla:

```bash
echo $userID
```

El identificador tiene el formato:

```
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACRId
```

### <a name="configure-the-vm-with-the-identity"></a>Configuración de la máquina virtual con la identidad

El siguiente comando [az vm identity assign][az-vm-identity-assign] configura la máquina virtual de Docker con la identidad asignada por el usuario:

```azurecli
az vm identity assign --resource-group myResourceGroup --name myDockerVM --identities $userID
```

### <a name="grant-identity-access-to-the-container-registry"></a>Concesión de acceso a la identidad para el registro de contenedor

Ahora, configure la identidad para que acceda al registro de contenedor. Utilice primero el comando [az acr show][az-acr-show] para obtener el identificador de recurso del registro:

```azurecli
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Use el comando [az role assignment create][az-role-assignment-create] para asignar el rol AcrPull al registro. Este rol proporciona [permisos de extracción](container-registry-roles.md) en el registro. Para proporcionar los permisos de extracción e inserción, asigne el rol ACRPush.

```azurecli
az role assignment create --assignee $spID --scope $resourceID --role acrpull
```

### <a name="use-the-identity-to-access-the-registry"></a>Uso de la identidad para tener acceso al registro

Establezca una conexión SSH con la máquina virtual de Docker que está configurada con la identidad. Ejecute los siguientes comandos de la CLI de Azure mediante la instancia de la CLI de Azure que instaló en la máquina virtual.

En primer lugar, autentíquese en la CLI de Azure con [az login][az-login] y la identidad que configuró en la máquina virtual. Como valor de `<userID>`, use el identificador de identidad que recuperó en el paso anterior. 

```azurecli
az login --identity --username <userID>
```

Después, autentíquese en el registro con [az acr login][az-acr-login]. Cuando se usa este comando, la CLI utiliza el token de Active Directory que creó al ejecutar `az login` para autenticar sin problemas la sesión con el registro de contenedor. (Según la configuración de la máquina virtual, es posible que deba ejecutar este comando y los comandos de docker con `sudo`).

```azurecli
az acr login --name myContainerRegistry
```

Verá el mensaje `Login succeeded`. A continuación, puede ejecutar los comandos `docker` sin proporcionar las credenciales. Por ejemplo, ejecute [docker pull][docker-pull] para extraer la imagen `aci-helloworld:v1` al especificar el nombre del servidor de inicio de sesión del registro. El nombre del servidor de inicio de sesión consta del nombre del registro de contenedor (todo en minúsculas) seguido por `.azurecr.io`; por ejemplo, `mycontainerregistry.azurecr.io`.

```
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

## <a name="example-2-access-with-a-system-assigned-identity"></a>Ejemplo 2: Acceso con una identidad asignada por el sistema

### <a name="configure-the-vm-with-a-system-managed-identity"></a>Configuración de la máquina virtual con una identidad administrada por el sistema

El siguiente comando [az vm identity assign][az-vm-identity-assign] configura la máquina virtual de Docker con la identidad asignada por el sistema:

```azurecli
az vm identity assign --resource-group myResourceGroup --name myDockerVM 
```

Use el comando [az vm show][az-vm-show] para establecer una variable con el valor de `principalId` (el identificador de la entidad de servicio) de la identidad de la máquina virtual con el fin de usarla en pasos posteriores.

```azurecli-interactive
spID=$(az vm show --resource-group myResourceGroup --name myDockerVM --query identity.principalId --out tsv)
```

### <a name="grant-identity-access-to-the-container-registry"></a>Concesión de acceso a la identidad para el registro de contenedor

Ahora, configure la identidad para que acceda al registro de contenedor. Utilice primero el comando [az acr show][az-acr-show] para obtener el identificador de recurso del registro:

```azurecli
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Use el comando [az role assignment create][az-role-assignment-create] para asignar el rol AcrPull a la identidad. Este rol proporciona [permisos de extracción](container-registry-roles.md) en el registro. Para proporcionar los permisos de extracción e inserción, asigne el rol ACRPush.

```azurecli
az role assignment create --assignee $spID --scope $resourceID --role acrpull
```

### <a name="use-the-identity-to-access-the-registry"></a>Uso de la identidad para tener acceso al registro

Establezca una conexión SSH con la máquina virtual de Docker que está configurada con la identidad. Ejecute los siguientes comandos de la CLI de Azure mediante la instancia de la CLI de Azure que instaló en la máquina virtual.

En primer lugar, autentíquese en la CLI de Azure con [az login][az-login] y la identidad asignada por el sistema en la máquina virtual.

```azurecli
az login --identity
```

Después, autentíquese en el registro con [az acr login][az-acr-login]. Cuando se usa este comando, la CLI utiliza el token de Active Directory que creó al ejecutar `az login` para autenticar sin problemas la sesión con el registro de contenedor. (Según la configuración de la máquina virtual, es posible que deba ejecutar este comando y los comandos de docker con `sudo`).

```azurecli
az acr login --name myContainerRegistry
```

Verá el mensaje `Login succeeded`. A continuación, puede ejecutar los comandos `docker` sin proporcionar las credenciales. Por ejemplo, ejecute [docker pull][docker-pull] para extraer la imagen `aci-helloworld:v1` al especificar el nombre del servidor de inicio de sesión del registro. El nombre del servidor de inicio de sesión consta del nombre del registro de contenedor (todo en minúsculas) seguido por `.azurecr.io`; por ejemplo, `mycontainerregistry.azurecr.io`.

```
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, obtuvo información sobre cómo usar las identidades administradas con Azure Container Registry y ha aprendido a realizar las siguientes tareas:

> [!div class="checklist"]
> * Habilitación de una identidad asignada por el usuario o asignada por el sistema en una VM de Azure.
> * Concesión del acceso de identidad a una instancia de Azure Container Registry.
> * Uso de la identidad administrada para acceder al registro y extraer una imagen de contenedor.

* Obtenga más información sobre las [identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/index.yml).


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az_login
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-vm-create]: /cli/azure/vm#az_vm_create
[az-vm-show]: /cli/azure/vm#az_vm_show
[az-vm-identity-assign]: /cli/azure/vm/identity#az_vm_identity_assign
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-identity-show]: /cli/azure/identity#az_identity_show
[azure-cli]: /cli/azure/install-azure-cli
