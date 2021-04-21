---
title: Tarea de varios pasos para compilar, probar y revisar imágenes
description: Introducción a tareas de varios pasos, una característica de ACR Tasks de Azure Container Registry que proporciona flujos de trabajos basados en tareas para compilar, probar y revisar imágenes de contenedor en la nube.
ms.topic: article
ms.date: 03/28/2019
ms.openlocfilehash: d57044fa8a0db7d661eb50284b34a6bbec9a1879
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781026"
---
# <a name="run-multi-step-build-test-and-patch-tasks-in-acr-tasks"></a>Ejecución de tareas de varios pasos de compilación, prueba y aplicación de revisiones en ACR Tasks

Las tareas de varios pasos amplían la funcionalidad de inserción y compilación de una única imagen de ACR Tasks con flujos de trabajo basados en varios contenedores y varios pasos. Use tareas de varios pasos para crear e insertar varias imágenes en serie o en paralelo. A continuación, ejecute esas imágenes como comandos en la ejecución de una sola tarea. Cada paso define una operación de compilación o inserción de una imagen de contenedor, y también define la ejecución de un contenedor. Cada paso de una tarea con varios pasos usa un contenedor como entorno de ejecución.

> [!IMPORTANT]
> Si ha creado anteriormente tareas durante la versión preliminar con el comando `az acr build-task`, deberá volver a crear esas tareas mediante el comando [az acr task][az-acr-task].

Por ejemplo, puede ejecutar una tarea con pasos que automatice la siguiente lógica:

1. Crear una imagen de aplicación web
1. Ejecutar el contenedor de aplicaciones web
1. Crear una imagen de prueba de aplicación web
1. Ejecutar el contenedor de prueba de aplicaciones web que realiza pruebas en el contenedor de aplicaciones en ejecución
1. Si se superan las pruebas, compilar un paquete de archivo de gráfico de Helm
1. Realizar una acción `helm upgrade` con el nuevo paquete de archivo de gráfico de Helm

Todos los pasos se realizan dentro de Azure, mediante la descarga del trabajo en los recursos de proceso de Azure; de esta forma, ya no tiene que ocuparse de la administración de la infraestructura. Aparte de su registro de contenedor de Azure, solo paga por los recursos que usa. Para información sobre los precios, consulte la sección **Container Build** en [Precios de Azure Container Registry][pricing].


## <a name="common-task-scenarios"></a>Escenarios de tareas comunes

Las tareas de varios pasos permiten escenarios como la siguiente lógica:

* Compilar, etiquetar e insertar una o varias imágenes de contenedor, en serie o en paralelo
* Ejecutar y capturar resultados de cobertura de código y pruebas unitarias
* Ejecutar y capturar pruebas funcionales ACR Tasks admite la ejecución de más de un contenedor y ejecuta una serie de solicitudes entre ellos.
* Realizar la ejecución basada en tareas, incluidos los pasos anteriores y posteriores de una compilación de imagen de contenedor
* Implementar uno o varios contenedores con el motor de implementación favorito en su entorno de destino

## <a name="multi-step-task-definition"></a>Definición de tareas de varios pasos

Una tarea de varios pasos de ACR Tasks se define como una serie de pasos dentro de un archivo YAML. Cada paso puede especificar dependencias tras la finalización correcta de uno o varios pasos anteriores. Están disponibles los siguientes tipos de pasos de tarea:

* [`build`](container-registry-tasks-reference-yaml.md#build): compila una o varias imágenes de contenedor mediante sintaxis `docker build` conocida, en serie o en paralelo.
* [`push`](container-registry-tasks-reference-yaml.md#push): inserta imágenes compiladas en un registro de contenedor. Se admiten registros privados como Azure Container Registry, dado que es el Docker Hub público.
* [`cmd`](container-registry-tasks-reference-yaml.md#cmd): ejecuta un contenedor, de forma que puede operar como una función dentro del contexto de la tarea en ejecución. Puede pasar parámetros al elemento `[ENTRYPOINT]` del contenedor y especificar propiedades como env, detach y otros parámetros conocidos de `docker run`. El tipo de paso `cmd` permite pruebas unitarias y funcionales, con la ejecución simultánea del contenedor.

Los fragmentos de código siguientes muestran cómo combinar estos tipos de pasos de la tarea. Las tareas de varios pasos pueden ser tan simples como la creación de una sola imagen a partir de un Dockerfile y la inserción en el registro, con un archivo YAML parecido a:

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID .
  - push: ["$Registry/hello-world:$ID"]
```

O pueden ser más complejas como en esta definición de tarea ficticia de varios pasos que incluye los pasos para las acciones build, test, helm package y helm deploy (no se muestra el registro de contenedor ni la configuración del repositorio de Helm):

```yml
version: v1.1.0
steps:
  - id: build-web
    build: -t $Registry/hello-world:$ID .
    when: ["-"]
  - id: build-tests
    build: -t $Registry/hello-world-tests ./funcTests
    when: ["-"]
  - id: push
    push: ["$Registry/helloworld:$ID"]
    when: ["build-web", "build-tests"]
  - id: hello-world-web
    cmd: $Registry/helloworld:$ID
  - id: funcTests
    cmd: $Registry/helloworld:$ID
    env: ["host=helloworld:80"]
  - cmd: $Registry/functions/helm package --app-version $ID -d ./helm ./helm/helloworld/
  - cmd: $Registry/functions/helm upgrade helloworld ./helm/helloworld/ --reuse-values --set helloworld.image=$Registry/helloworld:$ID
```

Consulte [ejemplos de tareas](container-registry-tasks-samples.md) para archivos YAML y Dockerfiles de tareas de varios pasos para varios escenarios.

## <a name="run-a-sample-task"></a>Ejecución de una tarea de ejemplo

Las tareas admiten ejecución manual, lo que se conoce como "ejecución rápida", y ejecución automatizada tras la confirmación de Git o la actualización de la imagen base.

Para ejecutar una tarea, primero defina los pasos de la tarea en un archivo YAML y luego ejecute el comando [az acr run][az-acr-run] de la CLI de Azure.

Este es un ejemplo de comando de la CLI de Azure que ejecuta una tarea mediante un archivo YAML de tarea de ejemplo. Los pasos son compilar y luego insertar una imagen. Antes de ejecutar este comando, actualice `\<acrName\>` con el nombre de su propio registro de contenedor de Azure.

```azurecli
az acr run --registry <acrName> -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

Al ejecutar la tarea, la salida debe mostrar el progreso de cada paso que se define en el archivo YAML. En la siguiente salida, los pasos aparecen como `acb_step_0` y `acb_step_1`.

```azurecli
az acr run --registry myregistry -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

```output
Sending context to registry: myregistry...
Queued a run with ID: yd14
Waiting for an agent...
2018/09/12 20:08:44 Using acb_vol_0467fe58-f6ab-4dbd-a022-1bb487366941 as the home volume
2018/09/12 20:08:44 Creating Docker network: acb_default_network
2018/09/12 20:08:44 Successfully set up Docker network: acb_default_network
2018/09/12 20:08:44 Setting up Docker configuration...
2018/09/12 20:08:45 Successfully set up Docker configuration
2018/09/12 20:08:45 Logging in to registry: myregistry.azurecr-test.io
2018/09/12 20:08:46 Successfully logged in
2018/09/12 20:08:46 Executing step: acb_step_0
2018/09/12 20:08:46 Obtaining source code and scanning for dependencies...
2018/09/12 20:08:47 Successfully obtained source code and scanned for dependencies
Sending build context to Docker daemon  109.6kB
Step 1/1 : FROM hello-world
 ---> 4ab4c602aa5e
Successfully built 4ab4c602aa5e
Successfully tagged myregistry.azurecr-test.io/hello-world:yd14
2018/09/12 20:08:48 Executing step: acb_step_1
2018/09/12 20:08:48 Pushing image: myregistry.azurecr-test.io/hello-world:yd14, attempt 1
The push refers to repository [myregistry.azurecr-test.io/hello-world]
428c97da766c: Preparing
428c97da766c: Layer already exists
yd14: digest: sha256:1a6fd470b9ce10849be79e99529a88371dff60c60aab424c077007f6979b4812 size: 524
2018/09/12 20:08:55 Successfully pushed image: myregistry.azurecr-test.io/hello-world:yd14
2018/09/12 20:08:55 Step id: acb_step_0 marked as successful (elapsed time in seconds: 2.035049)
2018/09/12 20:08:55 Populating digests for step id: acb_step_0...
2018/09/12 20:08:57 Successfully populated digests for step id: acb_step_0
2018/09/12 20:08:57 Step id: acb_step_1 marked as successful (elapsed time in seconds: 6.832391)
The following dependencies were found:
- image:
    registry: myregistry.azurecr-test.io
    repository: hello-world
    tag: yd14
    digest: sha256:1a6fd470b9ce10849be79e99529a88371dff60c60aab424c077007f6979b4812
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: latest
    digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
  git: {}


Run ID: yd14 was successful after 19s
```

Para más información sobre las compilaciones automatizadas tras la confirmación de Git o la actualización de la imagen base, consulte los tutoriales [Automatización de compilaciones de imágenes](container-registry-tutorial-build-task.md) y [Compilaciones de actualización de imágenes base](container-registry-tutorial-base-image-update.md).

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar referencias y ejemplos de las tareas de varios pasos aquí:

* [Referencia de tareas](container-registry-tasks-reference-yaml.md): tipos de pasos de tareas, sus propiedades y uso.
* [Ejemplos de tareas](container-registry-tasks-samples.md): archivos `task.yaml` y de Docker de ejemplo para varios escenarios, de sencillos a complejos.
* [Repositorio de cmd](https://github.com/AzureCR/cmd): una colección de contenedores que se usan como comandos para tareas de ACR.

<!-- IMAGES -->

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[task-examples]: https://github.com/Azure-Samples/acr-tasks
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-run]: /cli/azure/acr#az_acr_run
[az-acr-task]: /cli/azure/acr/task
