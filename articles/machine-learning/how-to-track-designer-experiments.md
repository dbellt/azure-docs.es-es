---
title: Registro de las métricas en el diseñador
titleSuffix: Azure Machine Learning
description: Supervise los experimentos del diseñador de aprendizaje automático de Azure. Habilite el registro mediante el módulo Execute Python Script (Ejecutar script de Python) y vea los resultados registrados en Studio.
services: machine-learning
author: likebupt
ms.author: keli19
ms.reviewer: peterlu
ms.service: machine-learning
ms.subservice: core
ms.date: 01/11/2021
ms.topic: how-to
ms.custom: designer
ms.openlocfilehash: bbf4168e59f04829603008bedea380f76a67264a
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884577"
---
# <a name="enable-logging-in-azure-machine-learning-designer-pipelines"></a>Habilitación del registro en canalizaciones del diseñador de Azure Machine Learning


En este artículo, aprenderá a agregar código de registro a las canalizaciones del diseñador. También aprenderá a ver esos registros mediante el portal web de Azure Machine Learning Studio.

Para más información sobre el registro de métricas con la experiencia de creación del SDK, consulte [Supervisión de métricas y ejecuciones de experimentos de Azure ML](how-to-log-view-metrics.md).

## <a name="enable-logging-with-execute-python-script"></a>Habilitación del registro con Execute Python Script (Ejecutar script de Python)

Use el módulo [Execute Python Script](./algorithm-module-reference/execute-python-script.md) (Ejecutar script de Python) para habilitar el registro en las canalizaciones del diseñador. Puede registrar cualquier valor mediante este flujo de trabajo, pero es especialmente útil registrar las métricas del módulo __Evaluate Model__ (Evaluar modelo) para realizar un seguimiento del rendimiento del modelo en diferentes ejecuciones.

En el ejemplo siguiente se muestra cómo registrar el error cuadrático medio de dos modelos entrenados mediante los módulos Evaluate Model (Evaluar modelo) y Execute Python Script (Ejecutar script de Python).

1. Conecte un módulo __Execute Python Script__ (Ejecutar script de Python) a la salida del módulo __Evaluate Model__ (Evaluar modelo).

    ![Conecte el módulo de ejecución de script de Python a el módulo evaluar el modelo](./media/how-to-log-view-metrics/designer-logging-pipeline.png)

1. Pegue el código siguiente en el editor de código de __Execute Python Script__ (Ejecutar script de Python) para registrar el error absoluto medio del modelo entrenado. Puede usar un patrón similar para registrar cualquier otro valor en el diseñador:

    ```python
    # dataframe1 contains the values from Evaluate Model
    def azureml_main(dataframe1=None, dataframe2=None):
        print(f'Input pandas.DataFrame #1: {dataframe1}')
    
        from azureml.core import Run
    
        run = Run.get_context()
    
        # Log the mean absolute error to the parent run to see the metric in the run details page.
        # Note: 'run.parent.log()' should not be called multiple times because of performance issues.
        # If repeated calls are necessary, cache 'run.parent' as a local variable and call 'log()' on that variable.
        parent_run = Run.get_context().parent
        
        # Log left output port result of Evaluate Model. This also works when evaluate only 1 model.
        parent_run.log(name='Mean_Absolute_Error (left port)', value=dataframe1['Mean_Absolute_Error'][0])
        # Log right output port result of Evaluate Model. The following line should be deleted if you only connect one Score Module to the` left port of Evaluate Model module.
        parent_run.log(name='Mean_Absolute_Error (right port)', value=dataframe1['Mean_Absolute_Error'][1])

        return dataframe1,
    ```
    
Este código usa el SDK de Azure Machine Learning para Python para registrar los valores. Se emplea Run.get_context() para obtener el contexto de la ejecución actual. Después, se registran los valores en ese contexto con el método run.parent.log(). Se usa `parent` para registrar los valores en la ejecución de la canalización primaria en lugar de en la ejecución del módulo.

Para más información sobre cómo usar el SDK de Python para registrar valores, consulte [Habilitación del registro en ejecuciones de entrenamiento de Azure ML](how-to-log-view-metrics.md).

## <a name="view-logs"></a>Ver registros

Una vez completada la ejecución de la canalización, puede ver *Mean_Absolute_Error* en la página de experimentos.

1. Vaya a la sección **Experiments** (Experimentos).
1. Seleccione el tipo de experimento.
1. Seleccione la ejecución en el experimento que quiere ver.
1. Seleccione **Métricas**.

    ![Visualización de las métricas de ejecución en Studio](./media/how-to-log-view-metrics/experiment-page-metrics-across-runs.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a usar registros en el diseñador. Para conocer los pasos siguientes, consulte estos artículos relacionados:


* Consulte [Depuración y solución de problemas de las canalizaciones de aprendizaje automático](how-to-debug-pipelines.md#azure-machine-learning-designer) para aprender a solucionar problemas con las canalizaciones del diseñador.
* Consulte [Habilitación del registro en ejecuciones de entrenamiento de Azure ML](how-to-log-view-metrics.md) para aprender a usar el SDK de Python para registrar las métricas en la experiencia de creación del SDK.
* Información sobre cómo usar [Ejecutar script de Python](./algorithm-module-reference/execute-python-script.md) en el diseñador.
