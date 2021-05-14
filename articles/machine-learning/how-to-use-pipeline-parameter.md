---
title: Uso de parámetros de canalización en el diseñador para compilar canalizaciones versátiles
titleSuffix: Azure Machine Learning
description: Cómo usar los parámetros de canalización en el diseñador de Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: keli19
author: likebupt
ms.date: 04/09/2020
ms.topic: how-to
ms.custom: designer
ms.openlocfilehash: ba5af77022c3f230fdaf77d115a1c1a4b2151c3e
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888159"
---
# <a name="use-pipeline-parameters-in-the-designer-to-build-versatile-pipelines"></a>Uso de parámetros de canalización en el diseñador para compilar canalizaciones versátiles

Use los parámetros de canalización para crear canalizaciones flexibles en el diseñador. Los parámetros de canalización permiten establecer valores de forma dinámica en el entorno de ejecución para encapsular la lógica de canalización y reutilizar los recursos.

Los parámetros de canalización son especialmente útiles al volver a enviar una ejecución de canalización, los [modelos de reentrenamiento](how-to-retrain-designer.md) o la [realización de predicciones por lotes](how-to-run-batch-predictions-designer.md).

En este artículo, aprenderá a realizar las siguientes tareas:

> [!div class="checklist"]
> * Crear parámetros de canalización
> * Eliminar y administrar parámetros de canalización
> * Desencadenar ejecuciones de canalización al ajustar los parámetros de canalización

## <a name="prerequisites"></a>Prerrequisitos

* Un área de trabajo de Azure Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* Para una introducción guiada al diseñador, complete el [tutorial correspondiente](tutorial-designer-automobile-price-train-score.md). 

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="create-pipeline-parameter"></a>Creación de un parámetro de canalización

Hay tres formas de crear un parámetro de canalización en el diseñador:
- Cree un parámetro de canalización en el panel de configuración y enlácelo a un módulo.
- Promueva un parámetro de módulo a un parámetro de canalización.
- Promueva un conjunto de datos a un parámetro de canalización.

> [!NOTE]
> Los parámetros de canalización solo admiten tipos de datos básicos como `int`, `float` y `string`.

### <a name="option-1-create-a-pipeline-parameter-in-the-settings-panel"></a>Opción 1: Creación de un parámetro de canalización en el panel de configuración

En esta sección, creará un parámetro de canalización en el panel de configuración.

En este ejemplo, se crea un parámetro de canalización que define cómo una canalización rellena los datos que faltan mediante el módulo **Clean Missing Data** (Limpiar datos que faltan).

1. Junto al nombre del borrador de la canalización, seleccione el **icono de engranaje** para abrir el panel **Configuración**.

1. En la sección **Parámetros de canalización**, seleccione el icono **+** .

1.  Escriba un nombre para el parámetro y un valor predeterminado. 

    Por ejemplo, escriba `replace-missing-value` como nombre de parámetro y `0` como valor predeterminado.

   ![Captura de pantalla que muestra cómo crear un parámetro de canalización](media/how-to-use-pipeline-parameter/create-pipeline-parameter.png)


Después de crear un parámetro de canalización, debe [adjuntarlo al parámetro de módulo](#attach-module-parameter-to-pipeline-parameter) que desea establecer dinámicamente.

### <a name="option-2-promote-a-module-parameter"></a>Opción 2: Promoción de un parámetro de módulo

La manera más sencilla de crear un parámetro de canalización para un valor de módulo es promover un parámetro de módulo. Utilice los pasos siguientes para promover un parámetro de módulo a un parámetro de canalización:

1. Seleccione el módulo al que desea adjuntar un parámetro de canalización.
1. En el panel de detalles del módulo, pase el mouse sobre el parámetro que quiere especificar.
1. Seleccione los puntos suspensivos ( **...** ) que aparecen.
1. Seleccione **Add to pipeline parameter** (Agregar al parámetro de canalización).

    ![Captura de pantalla que muestra cómo promover el parámetro de módulo a la canalización parameter1](media/how-to-use-pipeline-parameter/promote-module-para-to-pipeline-para.png)

1. Indique un nombre de parámetro y un valor predeterminado.
1. Seleccione **Guardar**.

Ahora puede especificar nuevos valores para este parámetro siempre que envíe esta canalización.

### <a name="option-3-promote-a-dataset-to-a-pipeline-parameter"></a>Opción 3: Promoción de un conjunto datos a un parámetro de canalización

Si quiere enviar la canalización con conjuntos de datos variables, debe promover el conjunto de valores a un parámetro de canalización:

1. Seleccione el conjunto de datos que desea convertir en un parámetro de canalización.

1. En el panel de detalles del conjunto de datos, active **Set as pipeline parameter** (Establecer como parámetro de canalización).

   ![Captura de pantalla que muestra cómo establecer el conjunto de datos como parámetro de canalización](media/how-to-use-pipeline-parameter/set-dataset-as-pipeline-parameter.png)

Ahora puede especificar otro conjunto de datos mediante el parámetro de canalización la próxima vez que ejecute la canalización.

## <a name="attach-and-detach-module-parameter-to-pipeline-parameter"></a>Asociación y desasociación del parámetro de módulo en el parámetro de canalización 

En esta sección, aprenderá a asociar y desasociar el parámetro de módulo en el parámetro de canalización.

### <a name="attach-module-parameter-to-pipeline-parameter"></a>Asociación del parámetro de módulo al parámetro de canalización

Puede adjuntar los mismos parámetros de módulos duplicados al mismo parámetro de canalización si desea modificar el valor de una sola vez al desencadenar la ejecución de la canalización.

En el siguiente ejemplo se ha duplicado el módulo **Clean Missing Data** (Limpiar datos que faltan). En cada módulo **Clean Missing Data** (Limpiar datos que faltan), adjunte el **valor de reemplazo** al parámetro de canalización **replace-missing-value**:

1. Seleccione el módulo **Clean Missing Data** (Limpiar datos que faltan).

1. En el panel de detalles del módulo, a la derecha del lienzo, establezca el **modo de limpieza** en "Custom substitution value" (Valor de sustitución personalizado).

1. Pase el mouse por el campo **Replacement value** (Valor de reemplazo).

1. Seleccione los puntos suspensivos ( **...** ) que aparecen.

1. Seleccione el parámetro de canalización `replace-missing-value`.

   ![Captura de pantalla que muestra cómo adjuntar un parámetro de canalización](media/how-to-use-pipeline-parameter/attach-replace-value-to-pipeline-parameter.png)

Ha adjuntado correctamente el campo **Replacement value** (Valor de reemplazo) al parámetro de canalización. 


### <a name="detach-module-parameter-to-pipeline-parameter"></a>Desasociación del parámetro de módulo en el parámetro de canalización

Después de asociar el **valor de reemplazo** al parámetro de canalización, no se puede realizar ninguna acción.

Para desasociar el parámetro del módulo en el parámetro de canalización, haga clic en los puntos suspensivos ( **...** ) que hay junto al parámetro de módulo y seleccione **Detach from pipeline parameter** (Desasociar de parámetro de canalización).

 ![Captura de pantalla que muestra que no es accionable después de asociar al parámetro de canalización](media/how-to-use-pipeline-parameter/non-actionable-module-parameter.png)

## <a name="update-and-delete-pipeline-parameters"></a>Actualización y eliminación de parámetros de canalización

En esta sección, aprenderá a actualizar y eliminar parámetros de canalización.

### <a name="update-pipeline-parameters"></a>Actualización de los parámetros de canalización

Siga estos pasos para actualizar un parámetro de canalización de módulo:

1. En la parte superior del lienzo, seleccione el icono de engranaje.
1. En la sección **Parámetros de canalización**, puede ver y actualizar el nombre y el valor predeterminado de todos los parámetros de canalización.

### <a name="delete-a-dataset-pipeline-parameter"></a>Eliminación de un parámetro de canalización de conjunto de datos

Use los siguientes pasos para eliminar un parámetro de canalización del conjunto de datos:

1. Seleccione el módulo del conjunto de datos.
1. Desactive la opción **Set as pipeline parameter** (Establecer como parámetro de canalización).


### <a name="delete-module-pipeline-parameters"></a>Eliminación de parámetros de canalización de módulo

Siga estos pasos para eliminar un parámetro de canalización de módulo:

1. En la parte superior del lienzo, seleccione el icono de engranaje.

1. Seleccione los puntos suspensivos ( **...** ) junto al parámetro de canalización.

    Esta vista muestra los módulos a los que está asociado el parámetro de canalización.

    ![Captura de pantalla que muestra el parámetro de canalización actual aplicado a un módulo](media/how-to-use-pipeline-parameter/delete-pipeline-parameter2.png)

1. Seleccione **Delete parameter** (Eliminar parámetro) para eliminar el parámetro de canalización.

    > [!NOTE]
    > Al eliminar un parámetro de canalización, se desasociarán todos los parámetros del módulo adjuntos y el valor de los parámetros del módulo desasociados mantendrá el valor del parámetro de canalización actual.     

## <a name="trigger-a-pipeline-run-with-pipeline-parameters"></a>Desencadenamiento de una ejecución de canalización con parámetros de canalización 

En esta sección, aprenderá a enviar una ejecución de canalización mientras establece parámetros de canalización.

### <a name="resubmit-a-pipeline-run"></a>Reenvío de una ejecución de canalización

Después de enviar una canalización con parámetros de canalización, puede volver a enviar una ejecución de canalización con distintos parámetros:

1. Vaya a la página de detalles de canalización. En la ventana **Pipeline run overview** (Información general de ejecución de canalización), puede comprobar los valores y los parámetros de la canalización actual.

1. Seleccione **Resubmit** (Volver a enviar).
1. En **Setup pipeline run** (Configurar ejecución de canalización), especifique los nuevos parámetros de canalización. 

![Captura de pantalla que muestra el reenvío de la canalización con parámetros de canalización](media/how-to-use-pipeline-parameter/resubmit-pipeline-run.png)

### <a name="use-published-pipelines"></a>Uso de canalizaciones publicadas

También puede publicar una canalización para usar sus parámetros de canalización. Una **canalización publicada** es una canalización que se ha implementado en un recurso de proceso, que las aplicaciones cliente pueden invocar a través de un punto de conexión REST.

Los puntos de conexión publicados son especialmente útiles para escenarios de reentrenamiento y predicción de lotes. Para más información, consulte [Uso de parámetros de canalización para volver a entrenar modelos en el diseñador](how-to-retrain-designer.md) o [Ejecución de predicciones por lotes mediante el diseñador de Azure Machine Learning](how-to-run-batch-predictions-designer.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear parámetros de canalización en el diseñador. A continuación, vea cómo puede usar los parámetros de canalización para [volver a entrenar modelos](how-to-retrain-designer.md) o realizar [predicciones por lotes](how-to-run-batch-predictions-designer.md).

También puede aprender a [usar las canalizaciones mediante programación con el SDK](how-to-deploy-pipelines.md).
