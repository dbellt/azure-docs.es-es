---
title: 'Inicio rápido: creación de una imagen de contenedor a petición en Azure'
description: Use los comandos de Azure Container Registry para compilar, insertar y ejecutar rápidamente una imagen de contenedor Docker a petición en la nube de Azure.
ms.topic: quickstart
ms.date: 09/25/2020
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: d3ef126812cb36874b74c8569d141787f71a29da
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781458"
---
# <a name="quickstart-build-and-run-a-container-image-using-azure-container-registry-tasks"></a>Inicio rápido: Compilación y ejecución de una imagen de contenedor en Azure Container Registry Tasks

En este tutorial, se usan comandos de [Azure Container Registry Tasks][container-registry-tasks-overview] para compilar, insertar y ejecutar rápidamente una imagen de un contenedor Docker de forma nativa en Azure, sin que sea necesario instalar Docker a nivel local. ACR Tasks es un conjunto de características de Azure Container Registry que le ayudan a administrar y modificar imágenes de contenedor durante el ciclo de vida de este. En este ejemplo se muestra cómo descargar el ciclo de desarrollo de imágenes de contenedor de "bucle interno" a la nube con compilaciones a petición mediante una instancia local de Dockerfile. 

Después de este inicio rápido, explore las características más avanzadas de ACR Tasks, para lo que puede usar los [tutoriales](container-registry-tutorial-quick-task.md). ACR Tasks puede automatizar las compilaciones de imágenes basadas en actualizaciones de la imagen base o de confirmaciones de código, o probar varios contenedores, en paralelo, entre otros escenarios. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]
    
- Para realizar este inicio rápido es necesaria la versión 2.0.58 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Si aún no tiene un registro de contenedor, cree un grupo de recursos con el comando [az group create][az-group-create]. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

Cree un registro de contenedor mediante el comando [az acr create][az-acr-create]. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. En el ejemplo siguiente, se usa *myContainerRegistry008*. Actualice este valor a uno único.

```azurecli-interactive
az acr create --resource-group myResourceGroup \
  --name myContainerRegistry008 --sku Basic
```

En este ejemplo se crea un registro *Básico*, una opción de menor costo para los desarrolladores que están aprendiendo Azure Container Registry. Para más información sobre los niveles de servicio disponibles, consulte [SKU de Azure Container Registry][container-registry-skus].

## <a name="build-and-push-image-from-a-dockerfile"></a>Creación e inserción de una imagen desde Dockerfile

Use ahora Azure Container Registry para compilar e insertar una imagen. En primer lugar, cree un directorio de trabajo local y, después, cree un archivo Dockerfile denominado *Dockerfile* con una única línea: `FROM mcr.microsoft.com/hello-world`. Este es un ejemplo sencillo para crear una imagen de contenedor de Linux desde la imagen `hello-world` hospedada en Microsoft Container Registry. Puede crear su propio archivo Dockerfile estándar y compilar imágenes para otras plataformas. Si está trabajando en un shell de Bash, cree el archivo Dockerfile con el siguiente comando:

```bash
echo FROM mcr.microsoft.com/hello-world > Dockerfile
```

Ejecute el comando [az acr build][az-acr-build], que compila la imagen y, una vez que la imagen se ha creado correctamente, la envía al registro. En el ejemplo siguiente se compila e inserta la imagen `sample/hello-world:v1`. El signo `.` al final del comando establece la ubicación del Dockerfile, en este caso el directorio actual.

```azurecli-interactive
az acr build --image sample/hello-world:v1 \
  --registry myContainerRegistry008 \
  --file Dockerfile . 
```

La salida de una compilación e inserciones correctas is similar a esta:

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/build_archive_b0bc1e5d361b44f0833xxxx41b78c24e.tar.gz'...
Sending context (1.856 KiB) to registry: mycontainerregistry008...
Queued a build with ID: ca8
Waiting for agent...
2019/03/18 21:56:57 Using acb_vol_4c7ffa31-c862-4be3-xxxx-ab8e615c55c4 as the home volume
2019/03/18 21:56:57 Setting up Docker configuration...
2019/03/18 21:56:58 Successfully set up Docker configuration
2019/03/18 21:56:58 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Executing step ID: build. Working directory: '', Network: ''
2019/03/18 21:56:59 Obtaining source code and scanning for dependencies...
2019/03/18 21:57:00 Successfully obtained source code and scanned for dependencies
2019/03/18 21:57:00 Launching container with name: build
Sending build context to Docker daemon  13.82kB
Step 1/1 : FROM mcr.microsoft.com/hello-world
latest: Pulling from hello-world
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586fxxxx21577a99efb77324b0fe535
Successfully built fce289e99eb9
Successfully tagged mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:01 Successfully executed container: build
2019/03/18 21:57:01 Executing step ID: push. Working directory: '', Network: ''
2019/03/18 21:57:01 Pushing image: mycontainerregistry008.azurecr.io/sample/hello-world:v1, attempt 1
The push refers to repository [mycontainerregistry008.azurecr.io/sample/hello-world]
af0b15c8625b: Preparing
af0b15c8625b: Layer already exists
v1: digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a size: 524
2019/03/18 21:57:03 Successfully pushed image: mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:03 Step ID: build marked as successful (elapsed time in seconds: 2.543040)
2019/03/18 21:57:03 Populating digests for step ID: build...
2019/03/18 21:57:05 Successfully populated digests for step ID: build
2019/03/18 21:57:05 Step ID: push marked as successful (elapsed time in seconds: 1.473581)
2019/03/18 21:57:05 The following dependencies were found:
2019/03/18 21:57:05
- image:
    registry: mycontainerregistry008.azurecr.io
    repository: sample/hello-world
    tag: v1
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: v1
    digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
  git: {}

Run ID: ca8 was successful after 10s
```

## <a name="run-the-image"></a>Ejecución de la imagen

Ahora, ejecute rápidamente la imagen que ha compilado e insertado en el registro. Aquí se usa [az acr run][az-acr-run] para ejecutar el comando del contenedor. En el flujo de trabajo de desarrollo del contenedor, este podría ser un paso de validación previo a la implementación la imagen, o bien puede incluir el comando en un [archivo YAML que contiene varios pasos][container-registry-tasks-multi-step]. 

En el ejemplo siguiente se utiliza `$Registry` para especificar el registro en que se ejecuta el comando:

```azurecli-interactive
az acr run --registry myContainerRegistry008 \
  --cmd '$Registry/sample/hello-world:v1' /dev/null
```

El parámetro `cmd` del ejemplo ejecuta el contenedor en su configuración predeterminada, pero `cmd` admite más parámetros `docker run`, o incluso otros comandos `docker`.

La salida es similar a la siguiente:

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
Sending context (1.855 KiB) to registry: mycontainerre...
Queued a run with ID: cab
Waiting for an agent...
2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
2019/03/19 19:01:53 Setting up Docker configuration...
2019/03/19 19:01:54 Successfully set up Docker configuration
2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2019/03/19 19:01:55 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

2019/03/19 19:01:56 Successfully executed container: acb_step_0
2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)

Run ID: cab was successful after 6s
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete][az-group-delete] para quitar el grupo de recursos, el registro de contenedor y las imágenes de contenedor almacenadas allí.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha utilizado características de ACR Tasks para compilar, insertar y ejecutar rápidamente una imagen de un contenedor Docker de forma nativa en Azure, sin que sea necesario instalar Docker a nivel local. Continúe con los tutoriales de Azure Container Registry Tasks, donde aprenderá a usar ACR Tasks para automatizar las compilaciones y actualizaciones de imágenes.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Container Registry Tasks][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[azure-account]: https://azure.microsoft.com/free/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-run]: /cli/azure/acr#az_acr_run
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tasks-overview]: container-registry-tasks-overview.md
[container-registry-tasks-multi-step]: container-registry-tasks-multi-step.md
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
