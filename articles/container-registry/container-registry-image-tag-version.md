---
title: Prácticas recomendadas de etiquetas de imagen
description: Procedimientos recomendados para el etiquetado y el control de versiones de imágenes del contenedor de Docker al insertar imágenes en Azure Container Registry y extraer imágenes de este
author: stevelasker
ms.topic: article
ms.date: 07/10/2019
ms.author: stevelas
ms.openlocfilehash: 9dfd29f2265e599f3cc7c412b81c3b7b93af40db
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106109720"
---
# <a name="recommendations-for-tagging-and-versioning-container-images"></a>Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor

Al insertar las imágenes de contenedor en un registro de contenedor y, después, implementarlas, necesita una estrategia de etiquetado y control de versiones de las imágenes. En este artículo se describen dos enfoques y dónde se ajusta cada uno durante el ciclo de vida del contenedor:

* **Etiquetas estables**: etiquetas que se vuelven a utilizar, por ejemplo, para indicar una versión importante o secundaria, como *mycontainerimage:1.0*.
* **Etiquetas únicas**: una etiqueta diferente para cada imagen que se inserta en un registro, como *mycontainerimage:abc123*.

## <a name="stable-tags"></a>Etiquetas estables

**Recomendación**: utilice etiquetas estables para mantener **imágenes base** para las compilaciones del contenedor. Evite las implementaciones con etiquetas estables, ya que estas etiquetas siguen recibiendo actualizaciones y pueden introducir inconsistencias en los entornos de producción.

*Las etiquetas estables* significan que un desarrollador, o un sistema de compilación, puede continuar con la extracción de una etiqueta específica, que continúa recibiendo actualizaciones. Estable no significa que el contenido esté inmovilizado. Por el contrario, estable implica que la imagen debe ser estable para la intención de esa versión. Para mantenerse "estable", se puede utilizar para aplicar las revisiones de seguridad o actualizaciones de la plataforma.

### <a name="example"></a>Ejemplo

Un equipo de la plataforma envía la versión 1.0. Saben que enviarán actualizaciones, incluso actualizaciones secundarias. Para admitir las etiquetas estables para una versión principal o secundaria determinada, tienen dos conjuntos de etiquetas estables.

* `:1`: una etiqueta estable para la versión principal. `1` representa la versión primera* "más reciente" o "más nueva".
* `:1.0`: una etiqueta estable para la versión 1.0, que permite a un desarrollador enlazarse a las actualizaciones de 1.0 y no actualizarse a la versión 1.1 cuando se publica.

El equipo también utiliza la etiqueta `:latest`, que señala a la etiqueta estable más reciente, independientemente de la versión principal actual.

Cuando las actualizaciones de la imagen base están disponibles, o cualquier tipo de versión de servicio de la plataforma, las imágenes con las etiquetas estables se actualizan al último hash que representa la versión estable más reciente de esa versión.

En este caso, se proporciona servicio continuamente a las etiquetas principales y secundarias. En un escenario de imagen base, esto permite al propietario de la imagen proporcionar imágenes con servicio.

### <a name="delete-untagged-manifests"></a>Eliminar manifiestos sin etiquetas

Si se actualiza una imagen con una etiqueta estable, se elimina la etiqueta de la imagen etiquetada previamente, con lo que se obtiene una imagen huérfana. El manifiesto de la imagen anterior y los datos únicos de la capa permanecen en el Registro. Para mantener el tamaño del Registro, puede eliminar periódicamente los manifiestos no etiquetados resultantes de actualizaciones de imagen estables. Por ejemplo, [purgue automáticamente](container-registry-auto-purge.md) los manifiestos sin etiquetas con una antigüedad superior a una duración especificada o defina una [directiva de retención](container-registry-retention-policy.md) para manifiestos no etiquetados.

## <a name="unique-tags"></a>Etiquetas únicas

**Recomendación**: utilice etiquetas únicas para **implementaciones**, especialmente en un entorno que pueda escalar en varios nodos. Es probable que desee realizar implementaciones deliberadas de una versión coherente de los componentes. Si el contenedor se reinicia o un orquestador se escala horizontalmente con más instancias, los hosts no extraerán accidentalmente una versión más nueva, incoherente con los otros nodos.

El etiquetado único simplemente significa que cada imagen que se inserta en un registro tiene una etiqueta única. No se reutilizan las etiquetas. Hay varios patrones que puede seguir para generar etiquetas únicas, como los siguientes:

* **Marca de fecha y hora**: este enfoque es bastante común, ya que puede indicar claramente cuando se creó la imagen. ¿Pero, cómo correlacionar el sistema de compilación? ¿Tiene que buscar la compilación que se completó al mismo tiempo? ¿Qué zona horaria se encuentra? ¿Todos los sistemas de compilación están calibrados a UTC?
* **Confirmación de Git**: este enfoque funciona hasta que empieza la compatibilidad con actualizaciones de la imagen base. Si se produce una actualización de la imagen base, el sistema de compilación se inicia con la misma confirmación de Git que la compilación anterior. Sin embargo, la imagen base tiene contenido nuevo. En general, una confirmación de Git proporciona una etiqueta *semiestable*.
* **Hash de manifiesto**: cada imagen de contenedor insertada en un registro de contenedores se asocia con un manifiesto, que se identifica por un hash SHA-256 único o código hash. Aunque único, el hash es largo, difícil de leer y no correlacionado con el entorno de compilación.
* **Id. de compilación**: esta opción puede ser la mejor ya que es probablemente incremental, y le permite correlacionarse con la compilación específica para buscar todos los artefactos y registros. Sin embargo, como un hash de manifiesto, puede ser difícil para un humano leerlo.

  Si la organización tiene varios sistemas de compilación, el prefijo de la etiqueta con el nombre del sistema de compilación es una variación de esta opción: `<build-system>-<build-id>`. Por ejemplo, puede diferenciar las compilaciones del sistema de compilación Jenkins del equipo de API y del sistema de compilación de Azure Pipelines del equipo web.

### <a name="lock-deployed-image-tags"></a>Bloquear etiquetas de imágenes implementadas

Como práctica recomendada, es recomendable [bloquear](container-registry-image-lock.md) cualquier etiqueta de imagen implementada definiendo su atributo `write-enabled` como `false`. Esta práctica evita la eliminación accidental de una imagen del Registro y, posiblemente, la interrupción de las implementaciones. Puede incluir el paso de bloqueo en la canalización de versión.

El bloqueo de una imagen implementada le permite igualmente quitar otras imágenes no implementadas del Registro mediante características de Azure Container Registry para el mantenimiento del registro. Por ejemplo, [purgue automáticamente](container-registry-auto-purge.md) los manifiestos sin etiquetas o las imágenes sin bloquear con una antigüedad superior a una duración especificada o defina una [directiva de retención](container-registry-retention-policy.md) para manifiestos no etiquetados.

## <a name="next-steps"></a>Pasos siguientes

Para obtener una explicación más detallada de los conceptos de este artículo, consulte la entrada de blog sobre el [Etiquetado de Docker: procedimientos recomendados para el etiquetado y el control de versiones de las imágenes de Docker](https://stevelasker.blog/2018/03/01/docker-tagging-best-practices-for-tagging-and-versioning-docker-images/).

Para maximizar el rendimiento y rentabilizar el uso del registro de contenedor de Azure, consulte [Procedimientos recomendados para Azure Container Registry](container-registry-best-practices.md).

<!-- IMAGES -->


<!-- LINKS - Internal -->

