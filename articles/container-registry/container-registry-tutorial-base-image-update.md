---
title: 'Tutorial: Desencadenamiento de la compilación de imágenes en la actualización de imágenes base'
description: En este tutorial aprenderá a configurar una tarea de Azure Container Registry que desencadena automáticamente compilaciones de imágenes de contenedor en la nube cuando se actualiza una imagen de base del mismo registro.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 6d83fb7796cecab686d52d561266bd41daef9e01
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780810"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-an-azure-container-registry"></a>Tutorial: Automatización de compilaciones de imágenes de contenedor al actualizarse una imagen base en una instancia de Azure Container Registry 

[ACR Tasks](container-registry-tasks-overview.md) admite compilaciones automatizadas de imágenes de contenedor al [actualizar una imagen de base del contenedor](container-registry-tasks-base-images.md). Por ejemplo, cuando se hace una revisión del sistema operativo o del marco de trabajo de la aplicación para una de las imágenes de base. 

En este tutorial aprenderá a crear una tarea en ACR que desencadene una compilación en la nube cuando se inserte una imagen de base de contenedor en el mismo registro. También puede probar un tutorial para crear una tarea en ACR que desencadene una compilación de imagen cuando se inserte una imagen de base en [otro registro de contenedor de Azure](container-registry-tutorial-private-base-image-update.md). 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Compilación de la imagen base
> * Creación de una imagen de aplicación en el mismo registro para realizar el seguimiento de la imagen de base 
> * Actualización de la imagen base para que desencadene una tarea de imagen de aplicación
> * Mostrar la tarea desencadenada
> * Comprobación de la imagen de aplicación actualizada

## <a name="prerequisites"></a>Prerrequisitos

### <a name="complete-the-previous-tutorials"></a>Complete los tutoriales anteriores.

En este tutorial se da por supuesto que ya ha configurado el entorno y ha completado los pasos de los dos primeros tutoriales de la serie, en los que ha realizado las siguientes tareas:

- Creación de una instancia de Azure Container Registry
- Bifurcación del repositorio de ejemplo
- Clonación del repositorio de ejemplo
- Creación de un token de acceso personal de GitHub

Si aún no lo ha hecho, complete los primeros tutoriales antes de continuar:

[Compilación de imágenes de contenedor en la nube con Azure Container Registry Tasks](container-registry-tutorial-quick-task.md)

[Automatización de compilaciones de imágenes de contenedor con Azure Container Registry Tasks](container-registry-tutorial-build-task.md)

### <a name="configure-the-environment"></a>Configuración del entorno

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
- En este artículo se necesita la versión 2.0.46 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

Rellene estas variables de entorno de shell con valores adecuados para el entorno. Este paso no es estrictamente necesario, pero hace que la ejecución de los comandos de varias líneas de la CLI de Azure en este tutorial sea un poco más fácil. Si no rellena estas variables de entorno, debe reemplazar manualmente cada valor siempre que aparezca en los comandos de ejemplo.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```


### <a name="base-image-update-scenario"></a>Escenario de actualización de imagen base

Este tutorial le guía por un escenario de actualización de imagen de base en el que una imagen de base y una imagen de aplicación se mantienen en un solo registro. 

El [código de ejemplo][code-sample] incluye dos archivos Dockerfile: una imagen de aplicación y una imagen especificada como su base. En las secciones siguientes creará una tarea de ACR Tasks que desencadena automáticamente una compilación de la imagen de aplicación cuando se inserta una nueva versión de la imagen base en el mismo registro de contenedor.

* [Dockerfile-app][dockerfile-app]: una pequeña aplicación web de Node.js que representa una página web estática que muestra la versión de Node.js en la que se basa. Se simula la cadena de versión: se muestra el contenido de una variable de entorno, `NODE_VERSION`, que se definió en la imagen base.

* [Dockerfile-base][dockerfile-base]: la imagen que `Dockerfile-app` especifica como su base. Está basada en una imagen de [Node][base-node] e incluye la variable de entorno `NODE_VERSION`.

En las secciones siguientes, va a crear una tarea, actualizar el valor `NODE_VERSION` en el archivo de Docker de la imagen base y, finalmente, a usar ACR Tasks para compilar la imagen base. Cuando ACR Tasks inserta la nueva imagen base en el registro, este desencadena automáticamente una compilación de la imagen de aplicación. Si lo desea, puede ejecutar la imagen de contenedor de la aplicación localmente para ver las diferentes cadenas de versión de las imágenes compiladas.

En este tutorial, ACR Tasks compila e inserta una imagen de contenedor de aplicación especificada en un archivo Dockerfile. Azure Container Registry Tasks también puede ejecutar [tareas de varios pasos](container-registry-tasks-multi-step.md) mediante un archivo YAML que define los pasos para compilar, insertar y, opcionalmente, probar varios contenedores.

## <a name="build-the-base-image"></a>Compilación de la imagen base

Empiece por compilar la imagen de base con una *tarea rápida* de ACR Tasks mediante [az acr build][az-acr-build]. Como se ha descrito en el [primer tutorial](container-registry-tutorial-quick-task.md) de la serie, este proceso no solo sirve para compilar la imagen, si no también para insertarla en el registro de contenedor si la compilación es correcta.

```azurecli
az acr build --registry $ACR_NAME --image baseimages/node:15-alpine --file Dockerfile-base .
```

## <a name="create-a-task"></a>Crea una tarea.

A continuación, cree una tarea con [az acr task create][az-acr-task-create]:

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name baseexample1 \
    --image helloworld:{{.Run.ID}} \
    --arg REGISTRY_NAME=$ACR_NAME.azurecr.io \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT
```

Esta tarea es similar a la creada en el [tutorial anterior](container-registry-tutorial-build-task.md). Indica a ACR Tasks que desencadene una compilación de la imagen cuando se inserten confirmaciones en el repositorio especificado por `--context`. Mientras que el archivo de Dockerfile que se usa para compilar la imagen en el tutorial anterior especifica una imagen base pública (`FROM node:15-alpine`), el archivo Dockerfile en esta tarea ([Dockerfile-app][dockerfile-app]) especifica una imagen base en el mismo registro:

```dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
```

Esta configuración facilita simular una revisión del marco de trabajo en la imagen base más adelante en este tutorial.

## <a name="build-the-application-container"></a>Compilación del contenedor de aplicación

Use [az acr task run][az-acr-task-run] para desencadenar manualmente la tarea y compilar la imagen de aplicación. Este paso es necesario para que la tarea supervise la dependencia de la imagen de la aplicación en la imagen de base.

```azurecli
az acr task run --registry $ACR_NAME --name baseexample1
```

Una vez finalizada la tarea, tome nota del **identificador de ejecución** (por ejemplo, "da6") si desea completar el siguiente paso opcional.

### <a name="optional-run-application-container-locally"></a>Opcional: Ejecutar contenedor de aplicación localmente

Si está trabajando de forma local (no en Cloud Shell), y tiene Docker instalado, ejecute el contenedor para ver la aplicación representada en un explorador web antes de recompilar su imagen base. Si va a usar Cloud Shell, omita esta sección (Cloud Shell no admite `az acr login` ni `docker run`).

En primer lugar, autentíquese en el registro de contenedor con [az acr login][az-acr-login]:

```azurecli
az acr login --name $ACR_NAME
```

Ahora, ejecute el contenedor localmente con `docker run`. Reemplace **\<run-id\>** por el identificador de ejecución de la salida del paso anterior (por ejemplo, "da6"). En este ejemplo el contenedor se denomina `myapp` e incluye el parámetro `--rm` para quitar el contenedor cuando lo detenga.

```bash
docker run -d -p 8080:80 --name myapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Vaya a `http://localhost:8080` en el explorador, allí verá el número de versión de Node.js representado en la página web de manera parecida a la siguiente. En un paso posterior, cambiará la versión agregando una "a" a la cadena de versión.

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-01.png" alt-text="Captura de pantalla que muestra una aplicación de ejemplo en el explorador":::

Ejecute el siguiente comando para detener y eliminar el contenedor:

```bash
docker stop myapp
```

## <a name="list-the-builds"></a>Lista de las compilaciones

A continuación, muestre las tareas que ACR Tasks ha completado para el registro mediante el comando [az acr task list-runs][az-acr-task-list-runs]:

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Si completó el tutorial anterior (y no eliminó el registro), debería ver una salida parecida a la siguiente. Tome nota del número de ejecuciones de tareas y del identificador de ejecución más reciente, para que pueda comparar la salida después de actualizar la imagen base en la sección siguiente.

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
cax       baseexample1    linux       Succeeded  Manual     2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:29
cav       example2        linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:55
cau       example1        linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:40
cat       taskhelloworld  linux       Succeeded  Manual     2020-11-20T23:07:29Z  00:00:27
```

## <a name="update-the-base-image"></a>Actualización de la imagen base

Aquí puede simular una revisión de la plataforma en la imagen base. Edite **Dockerfile-base** y agregue una "a" después del número de versión definido en `NODE_VERSION`:

```dockerfile
ENV NODE_VERSION 15.2.1a
```

Ejecute una tarea rápida para compilar la imagen base modificada. Tome nota del **identificador de ejecución** de la salida.

```azurecli
az acr build --registry $ACR_NAME --image baseimages/node:15-alpine --file Dockerfile-base .
```

Una vez que haya finalizado la compilación y que ACR Tasks haya insertado la nueva imagen base en el registro, este desencadena una compilación de la imagen de aplicación. Es posible que la tarea que creó anteriormente tarde unos instantes en desencadenar la compilación de la imagen de aplicación, ya que debe detectar la imagen base recién compilada e insertada.

## <a name="list-updated-build"></a>Lista de compilaciones actualizadas

Ahora que ha actualizado la imagen base, vuelva a enumerar las ejecuciones de tareas para compararlas con la lista anterior. Si al principio la salida no es diferente, ejecute periódicamente el comando hasta ver cómo aparece la nueva ejecución de tarea en la lista.

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

La salida es similar a la siguiente. El DESENCADENADOR de la última compilación ejecutada debe ser "Image Update", lo cual indica que la tarea la inició la tarea rápida de la imagen base.

```output
Run ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
cav       example2        linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:55
cau       example1        linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:40
cat       taskhelloworld  linux       Succeeded  Manual        2020-11-20T23:07:29Z  00:00:27
```

Si quiere realizar el siguiente paso opcional y ejecutar el contenedor recién compilado para ver el número de versión actualizado, anote el valor del **identificador de ejecución** de la compilación que desencadenó "Image Update" (en la salida anterior, es "ca11").

### <a name="optional-run-newly-built-image"></a>Opcional: Ejecución de la imagen recién compilada

Si está trabajando de forma local (no en Cloud Shell), y tiene Docker instalado, ejecute la nueva imagen de aplicación después de terminar su compilación. Sustituya `<run-id>` por el identificador de ejecución que obtuvo en el paso anterior. Si va a usar Cloud Shell, omita esta sección (Cloud Shell no admite `docker run`).

```bash
docker run -d -p 8081:80 --name updatedapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Vaya a http://localhost:8081 en el explorador, allí verá el número de versión de Node.js actualizado (con la "a") en la página web:

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-02.png" alt-text="Captura de pantalla que muestra la aplicación de ejemplo actualizada en el explorador":::


Es importante observar que ha actualizado la imagen **base** con un nuevo número de versión, pero que también la imagen de **aplicación** recién compilada muestra la nueva versión. ACR Tasks recopiló el cambio en la imagen base y recompiló automáticamente la imagen de aplicación.

Ejecute el siguiente comando para detener y eliminar el contenedor:

```bash
docker stop updatedapp
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar una tarea para desencadenar automáticamente compilaciones de imágenes de contenedor cuando se actualiza la imagen base de la imagen. Ahora, pase al siguiente tutorial para aprender a desencadenar tareas según una programación definida.

> [!div class="nextstepaction"]
> [Ejecutar una tarea según una programación](container-registry-tasks-scheduled.md)

<!-- LINKS - External -->
[base-node]: https://hub.docker.com/_/node/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-update]: /cli/azure/acr/task#az_acr_task_update
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-task-list-runs]: /cli/azure/acr
[az-acr-task]: /cli/azure/acr
