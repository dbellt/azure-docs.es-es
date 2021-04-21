---
title: Inserción y extracción de imagen de contenedor
description: Inserte y extraiga imágenes de Docker en un registro de contenedor privado de Azure mediante la CLI de Docker.
ms.topic: article
ms.date: 01/23/2019
ms.custom: seodec18, H1Hack27Feb2017
ms.openlocfilehash: 48f5f1707881ac8461e12212be631d3b80c16ca7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783834"
---
# <a name="push-your-first-image-to-your-azure-container-registry-using-the-docker-cli"></a>Inserción de la primera imagen en el registro de contenedor de Azure mediante la CLI de Docker

Un registro de contenedor de Azure almacena y administra imágenes privadas de contenedor y otros artefactos, de una forma similar a la que [Docker Hub](https://hub.docker.com/) almacena imágenes de contenedor. Puede usar la [interfaz de la línea de comandos de Docker](https://docs.docker.com/engine/reference/commandline/cli/) (CLI de Docker) para, entre otras, realizar las siguientes operaciones de imágenes de contenedor en el registro de contenedor: [iniciar sesión](https://docs.docker.com/engine/reference/commandline/login/), [insertar](https://docs.docker.com/engine/reference/commandline/push/), [extraer](https://docs.docker.com/engine/reference/commandline/pull/), etc.

En los pasos siguientes, se descarga una [imagen de Nginx](https://store.docker.com/images/nginx) pública, se le asigna una etiqueta para el registro de contenedor de Azure privado, se inserta en el registro y luego se extrae de este.

## <a name="prerequisites"></a>Prerrequisitos

* **Registro de contenedor de Azure**: cree un registro de contenedor en la suscripción de Azure. Por ejemplo, use [Azure Portal](container-registry-get-started-portal.md) o la [CLI de Azure](container-registry-get-started-azure-cli.md).
* **CLI de Docker**: también debe tener instalado Docker localmente. Docker proporciona paquetes que permiten configurar Docker fácilmente en cualquier sistema [macOS][docker-mac], [Windows][docker-windows] o [Linux][docker-linux].

## <a name="log-in-to-a-registry"></a>Inicio de sesión en un registro

Hay [varias maneras de autenticar](container-registry-authentication.md) en el registro de contenedor privado. Es el método recomendado cuando se trabaja en una línea de comandos con el comando de la CLI de Azure [az acr login](/cli/azure/acr#az_acr_login). Por ejemplo, para iniciar sesión en un registro denominado *MiRegistro*, inicie sesión en la CLI de Azure y, a continuación, autentíquese en el registro:

```azurecli
az login
az acr login --name myregistry
```

También puede iniciar sesión con [docker login](https://docs.docker.com/engine/reference/commandline/login/). Por ejemplo, puede que haya [asignado una entidad de servicio](container-registry-authentication.md#service-principal) al registro para ver un escenario de automatización. Cuando ejecute el siguiente comando, proporcione de forma interactiva el identificador de aplicación (nombre de usuario) y la contraseña de la entidad de servicio cuando se le solicite. Para consultar procedimientos recomendados para administrar credenciales de inicio de sesión, vea la referencia del comando [docker login](https://docs.docker.com/engine/reference/commandline/login/):

```
docker login myregistry.azurecr.io
```

Ambos comandos devuelven `Login Succeeded` una vez completados.
> [!NOTE]
>* Es posible que quiera usar Visual Studio Code con la extensión de Docker para un inicio de sesión más rápido y cómodo.

> [!TIP]
> Especifique siempre el nombre completo del registro (en minúsculas) cuando se usa `docker login` y al etiquetar imágenes para insertar en el registro. En los ejemplos de este artículo, el nombre completo es *myregistry.azurecr.io*.

## <a name="pull-a-public-nginx-image"></a>Extracción de una imagen de Nginx pública

En primer lugar, extraiga una imagen pública de Nginx en el equipo local. En este ejemplo se extrae una imagen desde Microsoft Container Registry.

```
docker pull mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

## <a name="run-the-container-locally"></a>Ejecute el contenedor localmente

Ejecute el siguiente comando [docker run](https://docs.docker.com/engine/reference/run/) para iniciar una instancia local del contenedor de Nginx de forma interactiva (`-it`) en el puerto 8080. El argumento `--rm` especifica que el contenedor debe quitarse cuando lo detenga.

```
docker run -it --rm -p 8080:80 mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

Vaya a `http://localhost:8080` para ver la página web predeterminada que suministra Nginx en el contenedor en ejecución. Debería ver una página similar a la siguiente:

![Nginx en un equipo local](./media/container-registry-get-started-docker-cli/nginx.png)

Dado que inició el contenedor de forma interactiva con `-it`, puede ver la salida del servidor Nginx en la línea de comandos después de navegar a él en el explorador.

Para detener y quitar el contenedor, presione `Control`+`C`.

## <a name="create-an-alias-of-the-image"></a>Creación de un alias de la imagen

Utilice [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) para crear un alias de la imagen, con la ruta de acceso completa al registro. Este ejemplo especifica el espacio de nombres `samples` para evitar el desorden en la raíz del registro.

```
docker tag mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine myregistry.azurecr.io/samples/nginx
```

Para más información sobre cómo etiquetar con espacios de nombres, vea la sección [Espacios de nombres del repositorio](container-registry-best-practices.md#repository-namespaces) de [Procedimientos recomendados para Azure Container Registry](container-registry-best-practices.md).

## <a name="push-the-image-to-your-registry"></a>Inserción de la imagen en el registro

Ahora que ha etiquetado la imagen con la ruta de acceso completa para el registro privado, puede insertarla en el registro con [docker push](https://docs.docker.com/engine/reference/commandline/push/):

```
docker push myregistry.azurecr.io/samples/nginx
```

## <a name="pull-the-image-from-your-registry"></a>Extracción de la imagen del registro

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para extraer la imagen del registro:

```
docker pull myregistry.azurecr.io/samples/nginx
```

## <a name="start-the-nginx-container"></a>Inicie el contenedor de Nginx

Use el comando [docker run](https://docs.docker.com/engine/reference/run/) para ejecutar la imagen que ha extraído del registro:

```
docker run -it --rm -p 8080:80 myregistry.azurecr.io/samples/nginx
```

Vaya a `http://localhost:8080` para ver el contenedor en ejecución.

Para detener y quitar el contenedor, presione `Control`+`C`.

## <a name="remove-the-image-optional"></a>Retirada de la imagen (opcional)

Si ya no necesita la imagen Nginx, puede eliminarla localmente con el comando [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/).

```
docker rmi myregistry.azurecr.io/samples/nginx
```

Para quitar imágenes del registro de contenedor de Azure, puede usar el comando de la CLI de Azure [az acr repository delete](/cli/azure/acr/repository#az_acr_repository_delete). Por ejemplo, el siguiente comando elimina el manifiesto al que se hace referencia mediante la etiqueta `samples/nginx:latest`, todos los datos de la capa únicos y todas las demás etiquetas que hacen referencia al manifiesto.

```azurecli
az acr repository delete --name myregistry --image samples/nginx:latest
```

## <a name="next-steps"></a>Pasos siguientes

¡Ahora que conoce los fundamentos, ya está listo para empezar a usar el registro! Por ejemplo, implemente imágenes de contenedor del Registro en:

* [Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-prepare-app.md)
* [Azure Container Instances](../container-instances/container-instances-tutorial-prepare-app.md)
* [Service Fabric](../service-fabric/service-fabric-tutorial-create-container-images.md)

Opcionalmente, instale la [extensión de Docker para Visual Studio Code](https://code.visualstudio.com/docs/azure/docker) y la extensión de la [cuenta de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) para trabajar con los registros de contenedor de Azure. Extraiga e inserte imágenes en un registro de contenedor de Azure o ejecute ACR Tasks y, todo ello, en Visual Studio Code.


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/
