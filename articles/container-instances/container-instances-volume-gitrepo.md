---
title: Montar el volumen de gitRepo en un grupo de contenedores
description: Más información acerca de cómo montar un volumen de gitRepo para clonar un repositorio de GIT en las instancias de Container Instances
ms.topic: article
ms.date: 06/15/2018
ms.openlocfilehash: 7c1249e3120dd680c52bf74fb045bedf5202b9f2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763728"
---
# <a name="mount-a-gitrepo-volume-in-azure-container-instances"></a>Montaje de un volumen de gitRepo en Azure Container Instances

Obtenga información acerca de cómo montar un volumen de *gitRepo* para clonar un repositorio de GIT en las instancias de Container Instances.

> [!NOTE]
> El montaje de un volumen de *gitRepo* está actualmente restringido a los contenedores Linux. Aunque se está trabajando para incorporar todas las características a los contenedores Windows, puede consultar las diferencias actuales entre plataformas en la [introducción](container-instances-overview.md#linux-and-windows-containers).

## <a name="gitrepo-volume"></a>Volumen de gitRepo

El volumen de *gitRepo* monta un directorio y clona el repositorio de Git especificado en él en el inicio del contenedor. Al usar un volumen de *gitRepo* en las instancias de Container Instances, puede evitar agregar el código para hacerlo en las aplicaciones.

Al montar un volumen de *gitRepo*, puede establecer tres propiedades para configurar el volumen:

| Propiedad | Obligatorio | Descripción |
| -------- | -------- | ----------- |
| `repository` | Sí | Dirección URL completa, incluidos `http://` o `https://`, del repositorio de GIT que se va a clonar.|
| `directory` | No | Directorio en el que se debe clonar el repositorio. La ruta de acceso no debe contener "`..`" ni empezar por ello.  Si especifica "`.`", el repositorio se clona en el directorio del volumen. De lo contrario, el repositorio de GIT se clona en un subdirectorio del nombre indicado dentro del directorio del volumen. |
| `revision` | No | Hash de confirmación de la revisión que se va a clonar. Si no se especifica, se clona la revisión de `HEAD`. |

## <a name="mount-gitrepo-volume-azure-cli"></a>Montaje de un volumen de gitRepo: Azure CLI

Para montar un volumen de gitRepo al implementar instancias de contenedor con la [CLI de Azure](/cli/azure), utilice los parámetros `--gitrepo-url` y `--gitrepo-mount-path` en el comando [az container create][az-container-create]. Opcionalmente, puede especificar el directorio del volumen en que va a realizar la clonación (`--gitrepo-dir`) y el hash de confirmación de la revisión que se va a clonar (`--gitrepo-revision`).

Este comando de ejemplo clona la aplicación de ejemplo [aci-helloworld][aci-helloworld] de Microsoft en `/mnt/aci-helloworld` en la instancia de contenedor:

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name hellogitrepo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --dns-name-label aci-demo \
    --ports 80 \
    --gitrepo-url https://github.com/Azure-Samples/aci-helloworld \
    --gitrepo-mount-path /mnt/aci-helloworld
```

Para comprobar que se ha montado el volumen de gitRepo, inicie un shell en el contenedor con [az container exec][az-container-exec] y enumere el directorio:

```azurecli
az container exec --resource-group myResourceGroup --name hellogitrepo --exec-command /bin/sh
```

```output
/usr/src/app # ls -l /mnt/aci-helloworld/
total 16
-rw-r--r--    1 root     root           144 Apr 16 16:35 Dockerfile
-rw-r--r--    1 root     root          1162 Apr 16 16:35 LICENSE
-rw-r--r--    1 root     root          1237 Apr 16 16:35 README.md
drwxr-xr-x    2 root     root          4096 Apr 16 16:35 app
```

## <a name="mount-gitrepo-volume-resource-manager"></a>Montaje de un volumen de gitRepo: Resource Manager

Para montar un volumen de GitRepo al implementar instancias de contenedor con un [plantilla de Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), rellene la matriz `volumes` en la sección `properties` del grupo contenedor de la plantilla. Luego, en todos los contenedores del grupo de contenedores en los que desee montar el volumen de *GitRepo*, rellene la matriz `volumeMounts` en la sección `properties` de la definición del contenedor.

Por ejemplo, la siguiente plantilla de Resource Manager crea un grupo de contenedores que consta de un único contenedor. El contenedor clona dos repositorios de GitHub especificados por los bloques del volumen de *gitRepo*. El segundo volumen incluye propiedades adicionales que especifican un directorio en el que clonar y el hash de confirmación de una revisión específica que se va a clonar.

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-gitrepo.json -->
[!code-json[volume-gitrepo](~/azure-docs-json-samples/container-instances/aci-deploy-volume-gitrepo.json)]

A continuación, se muestra la estructura del directorio resultante de los dos repositorios clonados definidos en la plantilla anterior:

```
/mnt/repo1/aci-helloworld
/mnt/repo2/my-custom-clone-directory
```

Para ver un ejemplo de implementación de instancias de contenedor con una plantilla de Azure Resource Manager, consulte [Implementación de grupos de varios contenedores en Azure Container Instances](container-instances-multi-container-group.md).

## <a name="private-git-repo-authentication"></a>Autenticación del repositorio de Git privado

Para montar un volumen de gitRepo para un repositorio Git privado, especifique las credenciales en la dirección URL del repositorio. Normalmente, las credenciales tienen forma de un nombre de usuario y un token de acceso personal (PAT) que concede acceso de ámbito al repositorio.

Por ejemplo, el parámetro `--gitrepo-url` de la CLI de Azure para un repositorio GitHub privado tendría un aspecto similar al siguiente (donde "gituser" es el nombre de usuario de GitHub y "abcdef1234fdsa4321abcdef" es el token de acceso personal del usuario):

```console
--gitrepo-url https://gituser:abcdef1234fdsa4321abcdef@github.com/GitUser/some-private-repository
```

Para ver un repositorio GIT de Azure Repos, especifique cualquier nombre de usuario (puede usar "azurereposuser" como en el ejemplo siguiente) en combinación con un token de acceso personal válido:

```console
--gitrepo-url https://azurereposuser:abcdef1234fdsa4321abcdef@dev.azure.com/your-org/_git/some-private-repository
```

Para más información acerca de los tokens de acceso personal para GitHub y Azure Repos, vea lo siguiente:

GitHub: [Creating a personal access token for the command line][pat-github] (Creación de un token de acceso personal para la línea de comandos)

Azure Repos: [Create personal access tokens to authenticate access][pat-repos] (Creación de tokens de acceso personales para autenticar el acceso)

## <a name="next-steps"></a>Pasos siguientes

Aprenda a montar otros tipos de volúmenes en Azure Container Instances:

* [Montaje de un recurso compartido de archivos de Azure en Azure Container Instances](container-instances-volume-azure-files.md)
* [Montaje de un volumen de emptyDir en Azure Container Instances](container-instances-volume-emptydir.md)
* [Montaje de un volumen secreto en Azure Container Instances](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-helloworld]: https://github.com/Azure-Samples/aci-helloworld
[pat-github]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
[pat-repos]: /azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-container-exec]: /cli/azure/container#az_container_exec
