---
title: Purgar etiquetas y manifiestos
description: Use un comando de purga para eliminar varias etiquetas y manifiestos de una instancia de Azure Container Registry en función de la edad y un filtro de etiquetas y, opcionalmente, programar operaciones de purga.
ms.topic: article
ms.date: 02/19/2021
ms.openlocfilehash: 562d1940459cb1594b7cd9aca2af280b05a4e419
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784198"
---
# <a name="automatically-purge-images-from-an-azure-container-registry"></a>Purga automática de imágenes a partir de una instancia de Azure Container Registry

Cuando se usa una instancia de Azure Container Registry como parte de un flujo de trabajo de desarrollo, el registro puede rellenarse rápidamente con imágenes u otros artefactos que no son necesarios después de un breve período. Es posible que quiera eliminar todas las etiquetas que tengan más de una duración determinada o que coincidan con un filtro de nombre especificado. Para eliminar varios artefactos rápidamente, en este artículo se presenta el comando `acr purge`, que se puede ejecutar como una instancia de ACR Tasks [programada](container-registry-tasks-scheduled.md) o a petición. 

El comando `acr purge` está distribuido actualmente en una imagen de contenedor pública (`mcr.microsoft.com/acr/acr-cli:0.4`), creada a partir de código fuente del repositorio [acr-cli](https://github.com/Azure/acr-cli) en GitHub.

Puede usar Azure Cloud Shell o una instalación local de la CLI de Azure para ejecutar los ejemplos de instancia de ACR Tasks de este artículo. Si quiere usarla de forma local, necesitará la versión 2.0.76 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install]. 

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar. Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

> [!WARNING]
> Use el comando `acr purge` con precaución: los datos de las imágenes eliminadas son IRRECUPERABLES. Si tiene sistemas que extraen imágenes por resumen de manifiesto (en lugar de por el nombre de la imagen), no debe purgar imágenes sin etiqueta. La eliminación de imágenes sin etiqueta impedirá que esos sistemas extraigan las imágenes del registro. En lugar de extraer por manifiesto, considere la posibilidad de adoptar un esquema de *etiquetado único*, un [procedimiento recomendado](container-registry-image-tag-version.md).

Si quiere eliminar los manifiestos o las etiquetas de una sola imagen mediante comandos de la CLI de Azure, consulte [Eliminación de imágenes de contenedor en Azure Container Registry](container-registry-delete.md).

## <a name="use-the-purge-command"></a>Uso del comando de purga

El comando de contenedor `acr purge` elimina imágenes por etiqueta en un repositorio que coinciden con un filtro de nombre y que son anteriores a una duración especificada. De manera predeterminada, solo se eliminan las referencias de etiqueta, no los [manifiestos](container-registry-concepts.md#manifest) subyacentes ni los datos de la capa. El comando también tiene una opción para eliminar manifiestos. 

> [!NOTE]
> `acr purge` no elimina una etiqueta de imagen ni un repositorio cuando el atributo `write-enabled` está establecido en `false`. Para más información, consulte [Bloqueo de una imagen de contenedor en una instancia de Azure Container Registry](container-registry-image-lock.md).

`acr purge` está diseñado para ejecutarse como un comando de contenedor en una [instancia de ACR Tasks](container-registry-tasks-overview.md), de modo que se autentique automáticamente con el registro en el que se ejecuta la tarea y realice acciones allí. En los ejemplos de tareas de este artículo se usa el [alias](container-registry-tasks-reference-yaml.md#aliases) de comando `acr purge` en lugar de un comando de imagen de contenedor completo.

Como mínimo, especifique lo siguiente cuando ejecute `acr purge`:

* `--filter`: repositorio y *expresión regular* para filtrar etiquetas en el repositorio. Ejemplos: `--filter "hello-world:.*"` coincide con todas las etiquetas del repositorio `hello-world`, y `--filter "hello-world:^1.*"` coincide con las etiquetas que comienzan por `1`. Pase varios parámetros `--filter` para purgar varios repositorios.
* `--ago`: [cadena de duración](https://golang.org/pkg/time/) de estilo Go para indicar una duración más allá de la cual se eliminan las imágenes. La duración consta de una secuencia de uno o más números decimales, cada uno con un sufijo de unidad. Las unidades de tiempo válidas incluyen "d" para los días, "h" para las horas y "m" para los minutos. Por ejemplo, `--ago 2d3h6m` selecciona todas las imágenes filtradas que se han modificado por última vez hace más de 2 días, 3 horas y 6 minutos, y `--ago 1.5h` selecciona las imágenes que se han modificado por última vez hace más de una hora y media.

`acr purge` admite varios parámetros opcionales. En los ejemplos de este artículo se emplean los dos siguientes:

* `--untagged`: especifica que se eliminan los manifiestos que no tienen etiquetas asociadas (*manifiestos sin etiqueta*).
* `--dry-run`: especifica que no se elimina ningún dato, pero el resultado es el mismo que si el comando se ejecutase sin este marcador. Este parámetro es útil para probar un comando de purga con el fin de asegurarse de que no elimina accidentalmente los datos que se quieren conservar.
* `--keep`: especifica que se conservará el número x más reciente de etiquetas que se van a eliminar.
* `--concurrency`: especifica un número de tareas de purga que se van a procesar de forma simultánea. Si no se proporciona este parámetro, se usa un valor predeterminado.

Para parámetros adicionales, ejecute `acr purge --help`. 

El comando `acr purge` admite otras características de los comandos de ACR Tasks, incluidas las [variables de ejecución](container-registry-tasks-reference-yaml.md#run-variables) y los [registros de ejecución de tareas](container-registry-tasks-logs.md) que se transmiten y también se guardan para poderse recuperar posteriormente.

### <a name="run-in-an-on-demand-task"></a>Ejecución en una tarea a petición

En el ejemplo siguiente se usa el comando [az acr run][az-acr-run] para ejecutar el comando `acr purge` a petición. En este ejemplo se eliminan todas las etiquetas de imagen y los manifiestos del repositorio `hello-world` en *myregistry* que se modificaron hace más de un día. El comando de contenedor se pasa mediante una variable de entorno. La tarea se ejecuta sin un contexto de origen.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --untagged --ago 1d"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  /dev/null
```

### <a name="run-in-a-scheduled-task"></a>Ejecución de una tarea programada

En el ejemplo siguiente se usa el comando [az acr task create][az-acr-task-create] para crear una [instancia de ACR Tasks programada](container-registry-tasks-scheduled.md) diariamente. La tarea purga las etiquetas modificadas hace más de siete días en el repositorio `hello-world`. El comando de contenedor se pasa mediante una variable de entorno. La tarea se ejecuta sin un contexto de origen.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --ago 7d"

az acr task create --name purgeTask \
  --cmd "$PURGE_CMD" \
  --schedule "0 0 * * *" \
  --registry myregistry \
  --context /dev/null
```

Ejecute el comando [az acr task show][az-acr-task-show] para ver que el desencadenador de temporizador está configurado.

### <a name="purge-large-numbers-of-tags-and-manifests"></a>Purga de un gran número de etiquetas y manifiestos

La purga de un gran número de etiquetas y manifiestos puede tardar varios minutos o más. Para purgar miles de etiquetas y manifiestos, es posible que el comando tenga que ejecutarse más tiempo que el tiempo de espera predeterminado de 600 segundos para una tarea a petición o de 3600 segundos para una tarea programada. Si se supera el tiempo de espera, solo se elimina un subconjunto de etiquetas y manifiestos. Para asegurarse de que se ha completado una purga a gran escala, pase el parámetro `--timeout` para aumentar el valor. 

Por ejemplo, la siguiente tarea a petición establece un tiempo de espera de 3600 segundos (una hora):

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --ago 1d --untagged"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  --timeout 3600 \
  /dev/null
```

## <a name="example-scheduled-purge-of-multiple-repositories-in-a-registry"></a>Ejemplo: purga programada de varios repositorios en un registro

Este ejemplo se describe mediante el uso de `acr purge` para limpiar periódicamente varios repositorios en un registro. Por ejemplo, podría tener una canalización de desarrollo que inserte imágenes en los repositorios `samples/devimage1` y `samples/devimage2`. Las imágenes de desarrollo se importan periódicamente en un repositorio de producción para las implementaciones, por lo que dichas imágenes ya no se necesitan. Con una periodicidad semanal, se purgan los repositorios `samples/devimage1` y `samples/devimage2` para preparar el trabajo de la semana próxima.

### <a name="preview-the-purge"></a>Versión preliminar de la purga

Antes de eliminar los datos, se recomienda ejecutar una tarea de purga a petición mediante el parámetro `--dry-run`. Esta opción permite ver las etiquetas y los manifiestos que purgará el comando sin quitar los datos. 

En el ejemplo siguiente, el filtro de cada repositorio selecciona todas las etiquetas. El parámetro `--ago 0d` hace coincidir las imágenes de cualquier antigüedad de los repositorios que coinciden con los filtros. Modifique los criterios de selección según sea necesario para su escenario. El parámetro `--untagged` indica que se eliminen los manifiestos además de las etiquetas. El comando de contenedor se pasa al comando [az acr run][az-acr-run] mediante una variable de entorno.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge \
  --filter 'samples/devimage1:.*' --filter 'samples/devimage2:.*' \
  --ago 0d --untagged --dry-run"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  /dev/null
```

Revise la salida del comando para ver las etiquetas y los manifiestos que coinciden con los parámetros de selección. Dado que el comando se ejecuta con `--dry-run`, no se elimina ningún dato.

Salida del ejemplo:

```console
[...]
Deleting tags for repository: samples/devimage1
myregistry.azurecr.io/samples/devimage1:232889b
myregistry.azurecr.io/samples/devimage1:a21776a
Deleting manifests for repository: samples/devimage1
myregistry.azurecr.io/samples/devimage1@sha256:81b6f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e788b
myregistry.azurecr.io/samples/devimage1@sha256:3ded859790e68bd02791a972ab0bae727231dc8746f233a7949e40f8ea90c8b3
Deleting tags for repository: samples/devimage2
myregistry.azurecr.io/samples/devimage2:5e788ba
myregistry.azurecr.io/samples/devimage2:f336b7c
Deleting manifests for repository: samples/devimage2
myregistry.azurecr.io/samples/devimage2@sha256:8d2527cde610e1715ad095cb12bc7ed169b60c495e5428eefdf336b7cb7c0371
myregistry.azurecr.io/samples/devimage2@sha256:ca86b078f89607bc03ded859790e68bd02791a972ab0bae727231dc8746f233a

Number of deleted tags: 4
Number of deleted manifests: 4
[...]
```

### <a name="schedule-the-purge"></a>Programación de la purga

Después de comprobar el simulacro, cree una tarea programada para automatizar la purga. En el siguiente ejemplo se programa una tarea semanal el domingo a la 1:00 UTC para ejecutar el comando de purga anterior:

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge \
  --filter 'samples/devimage1:.*' --filter 'samples/devimage2:.*' \
  --ago 0d --untagged"

az acr task create --name weeklyPurgeTask \
  --cmd "$PURGE_CMD" \
  --schedule "0 1 * * Sun" \
  --registry myregistry \
  --context /dev/null
```

Ejecute el comando [az acr task show][az-acr-task-show] para ver que el desencadenador de temporizador está configurado.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre otras opciones para [eliminar datos de imagen](container-registry-delete.md) en Azure Container Registry.

Para más información sobre el almacenamiento de imágenes, consulte [Almacenamiento de imágenes de contenedor en Azure Container Registry](container-registry-storage.md).

<!-- LINKS - External -->

[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-acr-run]: /cli/azure/acr#az_acr_run
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
