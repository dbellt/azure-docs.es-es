---
title: 'Definición y uso de flujos de trabajo de contenido mediante la herramienta de revisión: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Puede usar el diseñador de flujos de trabajo de Content Moderator para definir flujos de trabajo y umbrales personalizados en función de las directivas de contenido.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: pafarley
ms.openlocfilehash: 87d643e7658477fb13f857b8b123889d82c5dfff
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112033840"
---
# <a name="define-and-use-moderation-workflows"></a>Definición y uso de los flujos de trabajo de moderación

[!INCLUDE [deprecation notice](../includes/tool-deprecation.md)]

En esta guía, aprenderá a configurar y usar [flujos de trabajo](../review-api.md#workflows) en el sitio web de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com). Los flujos de trabajo son filtros personalizados basados en la nube que puede usar para controlar el contenido de forma más eficaz. Los flujos de trabajo pueden conectarse a diversos servicios para filtrar el contenido de maneras diferentes y luego realizar la acción apropiada. En esta guía se muestra cómo usar el conector de Content Moderator (que se incluye de forma predeterminada) para filtrar el contenido y configurar revisiones humanas en un escenario típico de moderación.

## <a name="create-a-new-workflow"></a>Creación de un área de trabajo

Vaya al sitio de la [herramienta de revisión de Content Moderator](https://contentmoderator.cognitive.microsoft.com/) y regístrese. En la pestaña **Settings** (Configuración), seleccione **Workflows** (Flujos de trabajo).

![Configuración de flujos de trabajo](images/2-workflows-0.png)

En la siguiente pantalla, seleccione **Add Workflow** (Agregar flujo de trabajo).

![Adición de un flujo de trabajo](images/2-workflows-1.png)

### <a name="assign-a-name-and-description"></a>Asignación de un nombre y una descripción

Asigne un nombre al flujo de trabajo, escriba una descripción y elija si el flujo de trabajo administra imágenes o texto.

![Nombre y descripción del flujo de trabajo](images/image-workflow-create.PNG)

### <a name="define-evaluation-criteria"></a>Definición de los criterios de evaluación

En la siguiente pantalla, vaya a la sección **If** (Si). En el menú desplegable superior, elija **Condition** (Condición). Esto le permitirá configurar la condición sobre la que el flujo de trabajo realizará la acción. Si quiere usar varias condiciones, elija **Combination** (Combinación). 

A continuación, seleccione un conector. En este ejemplo se usa **Content Moderator**. Según el conector que elija, obtendrá distintas opciones para la salida de datos. Consulte la sección [Connectors](./configure.md#connectors) (Conectores) de la guía de configuración de la herramienta de revisión para saber cómo configurar otros conectores.

![Selección del conector de flujo de trabajo](images/image-workflow-connect-to.PNG)

Elija la salida deseada que usará y establezca las condiciones con la que comparar.

![Definición de la condición de flujo de trabajo](images/image-workflow-condition.PNG)

### <a name="define-the-action"></a>Definición de la acción

Vaya a la **Then** (Entonces) y seleccione una acción. En el ejemplo siguiente se crea una revisión de imágenes y se asigna una etiqueta. Si lo desea, puede agregar una ruta de acceso alternativa (Else) (Si no) y establecer también una acción para ella.

![Definición de la acción del flujo de trabajo](images/image-workflow-action.PNG)

### <a name="save-the-workflow"></a>Almacenamiento del flujo de trabajo.

Anote el nombre del flujo de trabajo; lo necesitará para iniciar un trabajo de moderación con la API de flujo de trabajo (vea a continuación). Por último, guarde el flujo de trabajo mediante el botón **Save** (Guardar) situado en la parte superior de la página.

## <a name="test-the-workflow"></a>Probar el flujo de trabajo

Ahora que ha definido un flujo de trabajo personalizado, pruébelo con contenido de ejemplo. Vaya a **Workflows** (Flujos de trabajo) y seleccione el botón **Execute Workflow** (Ejecutar flujo de trabajo) correspondiente.

![Prueba del flujo de trabajo](images/image-workflow-execute.PNG)

Guarde esta [imagen de ejemplo](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg) en la unidad local. Luego, seleccione **Choose File(s)** (Elegir archivos) y cargue la imagen en el flujo de trabajo.

![Un corredor con una cita superpuesta en la imagen](images/sample-text.jpg)

### <a name="track-progress"></a>Seguimiento del progreso

Puede ver el progreso del flujo de trabajo en la siguiente ventana emergente.

![Seguimiento de la ejecución del flujo de trabajo](images/image-workflow-job.PNG)

### <a name="verify-workflow-action"></a>Comprobación de la acción del flujo de trabajo

Vaya a la pestaña **Image** (Image) debajo de **Review** (Revisar) y compruebe que hay una revisión de imagen recién creada.

![Revisar las imágenes](images/image-workflow-review.PNG)

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a configurar y usar flujos de trabajo de moderación desde la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) de Content Moderator. A continuación, consulte la [guía de la consola de API](../try-review-api-workflow.md) para aprender a crear flujos de trabajo mediante programación.
