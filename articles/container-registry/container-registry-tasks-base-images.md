---
title: 'Actualizaciones de imagen base: Tasks'
description: Obtenga información sobre las imágenes base para las imágenes de contenedor de aplicación y sobre cómo una actualización de imagen base puede desencadenar una instancia de Azure Container Registry Tasks.
ms.topic: article
ms.date: 01/22/2019
ms.openlocfilehash: fc501cc1db654c11675e5a4f0d19a5a56b63a165
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781188"
---
# <a name="about-base-image-updates-for-acr-tasks"></a>Acerca de las actualizaciones de imagen base para ACR Tasks

En este artículo se proporciona información general sobre las actualizaciones de la imagen base de una aplicación y cómo estas actualizaciones pueden desencadenar una instancia de Azure Container Registry Tasks.

## <a name="what-are-base-images"></a>¿Qué son las imágenes base?

Los archivos de Docker que definen la mayoría de las imágenes de contenedor especifican una imagen primaria en la que se basa la imagen y que, a menudo, se denomina su *imagen base*. Las imágenes base normalmente contienen el sistema operativo, por ejemplo [Alpine Linux][base-alpine] o [Windows Nano Server][base-windows], sobre el que se aplica el resto de capas del contenedor. También pueden incluir marcos de trabajo de la aplicación, como [Node.js][base-node] o [.NET Core][base-dotnet]. Estas imágenes base se basan normalmente en imágenes ascendentes públicas. Algunas de las imágenes de aplicación podrían compartir una imagen base común.

El mantenedor de la imagen actualiza frecuentemente la imagen base para que incluya nuevas características o mejoras en el sistema operativo o plataforma de la imagen. Las revisiones de seguridad son otra causa habitual de una actualización de la imagen base. Cuando se producen estas actualizaciones ascendentes, también debe actualizar las imágenes base para incluir la corrección crítica. Cada imagen de aplicación se debe entonces volver a compilar para incluir estas correcciones ascendentes ahora incluidas en la imagen base.

En algunos casos, como en un equipo de desarrollo privado, una imagen base podría especificar más que el sistema operativo o el marco. Por ejemplo, una imagen base podría ser una imagen de componentes de servicio compartido a la que debe realizarse un seguimiento. Es posible que los miembros de un equipo tengan que realizar un seguimiento de esta imagen base para realizar pruebas, o tengan que actualizar periódicamente la imagen al desarrollar imágenes de aplicaciones.

## <a name="maintain-copies-of-base-images"></a>Mantenimiento de copias de imágenes base

En el caso de cualquier contenido de los registros que dependa del contenido base mantenido en un registro público como Docker Hub, se recomienda copiar el contenido en un registro de contenedor de Azure o en otro registro privado. A continuación, asegúrese de compilar las imágenes de la aplicación mediante una referencia a las imágenes base privadas. Azure Container Registry proporciona una capacidad de [importación de imágenes](container-registry-import-images.md) para copiar fácilmente contenido de registros públicos u otros registros de contenedor de Azure. En la sección siguiente se describe el uso de ACR Tasks para realizar el seguimiento de las actualizaciones de imagen base al compilar actualizaciones de la aplicación. Puede realizar un seguimiento de las actualizaciones de imagen base en sus propios registros de contenedor de Azure y, opcionalmente, en los registros públicos ascendentes.

## <a name="track-base-image-updates"></a>Seguimiento de actualizaciones de imagen base

ACR Tasks incluye la posibilidad de compilar automáticamente las imágenes cuando se actualiza una imagen base del contenedor. Puede usar esta capacidad para mantener y actualizar las copias de las imágenes base públicas en los registros de contenedor de Azure y, después, volver a generar las imágenes de la aplicación que dependan de las imágenes base.

ACR Tasks detecta dinámicamente las dependencias de la imagen base al compilar una imagen de contenedor. Como resultado, puede detectar cuando se actualiza la imagen base de la imagen de una aplicación. Con una tarea de compilación preconfigurada, ACR Tasks puede recompilar automáticamente cada imagen de aplicación que haga referencia a la imagen base. Gracias a esta detección y recompilación automáticas, ACR Tasks le ahorra tiempo y esfuerzo que normalmente son necesarios para actualizar y realizar el seguimiento de forma manual de cada una de las imágenes de aplicación que hacen referencia a la imagen base actualizada.

## <a name="base-image-locations"></a>Ubicaciones de las imágenes base

En el caso de las compilaciones de imágenes desde un Dockerfile, una tarea ACR detecta las dependencias de las imágenes base en las siguientes ubicaciones:

* La misma instancia de Azure Container Registry en la que se ejecuta la tarea
* Otro registro de contenedor de Azure privado, en la misma región o en otra 
* Un repositorio público de Docker Hub 
* Un repositorio público de Microsoft Container Registry

Si la imagen base especificada en la instrucción `FROM` reside en alguna de esas ubicaciones, la instancia de ACR Tasks agrega un enlace para asegurarse de que esta imagen se recompile siempre que se actualiza su base.

## <a name="base-image-notifications"></a>Notificaciones de imagen base

El tiempo entre el momento en que se actualiza una imagen base y el momento en que se desencadena la tarea dependiente depende de la ubicación de la imagen base:

* **Imágenes base de un repositorio público en Docker Hub o MCR**: para las imágenes base en repositorios públicos, una tarea de ACR Tasks comprueba las actualizaciones de la imagen en un intervalo aleatorio de entre 10 y 60 minutos. Las tareas dependientes se ejecutan en consecuencia.
* **Imágenes base de un registro de contenedor de Azure**: para las imágenes base de los registros de contenedor de Azure, una tarea de ACR Tasks desencadena inmediatamente una ejecución cuando se actualiza su imagen base. La imagen base puede estar en el mismo ACR en el que se ejecuta la tarea o en un ACR diferente en cualquier región.

## <a name="additional-considerations"></a>Consideraciones adicionales

* **Imágenes base para imágenes de aplicación**: Actualmente, una instancia de ACR Tasks solo realiza el seguimiento de las actualizaciones de las imágenes base de las imágenes de las aplicaciones (*tiempo de ejecución*). No realiza un seguimiento de las actualizaciones de las imágenes base de las imágenes intermedias (*tiempo de compilación*) que se usan en Dockerfiles de varias fases.  

* **Habilitado de forma predeterminada**: Cuando se crea una instancia de ACR Tasks con el comando [az acr task create][az-acr-task-create], de manera predeterminada la tarea se *habilita* para desencadenarse con una actualización de imagen base. Es decir, la propiedad `base-image-trigger-enabled` está establecida en True. Si quiere deshabilitar este comportamiento en una tarea, actualice la propiedad en False. Por ejemplo, ejecute el siguiente comando [az acr task update][az-acr-task-update]:

  ```azurecli
  az acr task update --registry myregistry --name mytask --base-image-trigger-enabled False
  ```

* **Desencadenador para seguimiento de dependencias**: Para que una instancia de ACR Tasks pueda determinar y supervisar las dependencias de una imagen de contenedor (lo cual incluye su imagen base), primero debe desencadenar la tarea de compilación de la imagen **al menos una vez**. Por ejemplo, puede desencadenar la tarea de forma manual mediante el comando [az acr task run][az-acr-task-run].

* **Etiqueta stable para imagen base**: Para desencadenar una tarea cuando se actualice la imagen base, la imagen base debe tener una etiqueta *stable*, como `node:9-alpine`. Este etiquetado es común para las imágenes base que se actualizan a una versión estable más reciente mediante revisiones del sistema operativo y el marco. Si se actualiza la imagen base con una nueva etiqueta de versión, no se desencadena una tarea. Para obtener más información sobre el etiquetado de imágenes, consulte la [guía sobre procedimientos recomendados](container-registry-image-tag-version.md). 

* **Otros desencadenadores de tarea**: En una tarea desencadenada por actualizaciones de la imagen base, también puede habilitar desencadenadores en función de la [confirmación del código fuente](container-registry-tutorial-build-task.md) o una [programación](container-registry-tasks-scheduled.md). Una actualización de la imagen base también puede desencadenar una [tareas de varios pasos](container-registry-tasks-multi-step.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes tutoriales para ver escenarios de automatización de las compilaciones de imágenes de aplicación después de actualizar una imagen base:

* [Automatización de compilaciones de imágenes de contenedor al actualizarse una imagen base en el mismo registro](container-registry-tutorial-base-image-update.md)

* [Automatización de compilaciones de imágenes de contenedor al actualizarse una imagen base en otro registro](container-registry-tutorial-base-image-update.md)


<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-pack-build]: /cli/azure/acr/pack#az_acr_pack_build
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-update]: /cli/azure/acr/task#az_acr_task_update
[az-login]: /cli/azure/reference-index#az_login
[az-login-service-principal]: /cli/azure/authenticate-azure-cli

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
