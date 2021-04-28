---
title: ¿Qué son las canalizaciones de Machine Learning?
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo las canalizaciones de Machine Learning ayudan a crear, optimizar y administrar los flujos de trabajo de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: laobri
author: lobrien
ms.date: 02/26/2021
ms.custom: devx-track-python
ms.openlocfilehash: 57f5da06909436e0cbce92559c29c309ca9e20e3
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107819239"
---
# <a name="what-are-azure-machine-learning-pipelines"></a>¿Qué son las canalizaciones de Azure Machine Learning?

En este artículo, descubrirá cómo las canalizaciones de Machine Learning le ayudan a crear, optimizar y administrar los flujos de trabajo de aprendizaje automático. 

<a name="compare"></a>
## <a name="which-azure-pipeline-technology-should-i-use"></a>¿Qué tecnología de canalización de Azure se debe usar?

La nube de Azure proporciona varios tipos de canalizaciones, cada una con una finalidad diferente. En la tabla siguiente se enumeran las diferentes canalizaciones y para qué se usan:

| Escenario | Rol principal | Oferta de Azure | Oferta de OSS | Canalización canónica | Puntos destacados | 
| -------- | --------------- | -------------- | ------------ | -------------- | --------- | 
| Orquestación de modelo (aprendizaje automático) | Científico de datos | Canalizaciones de Azure Machine Learning | Canalizaciones de Kubeflow | Datos -> Modelo | Distribución, almacenamiento en caché, Code First, reutilización | 
| Orquestación de datos (preparación de datos) | Ingeniero de datos | [Canalizaciones de Azure Data Factory](../data-factory/concepts-pipelines-activities.md) | Airflow de Apache | Datos -> Datos | Actividades centradas en datos y movimiento fuertemente tipado |
| Orquestación de códigos y aplicaciones (CI/CD) | Desarrollo/operaciones de aplicaciones | [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) | Jenkins | Código + modelo-> Aplicación o servicio | Compatibilidad con la mayoría de las actividades abiertas y flexibles, colas de aprobación, fases con restricción de acceso | 

## <a name="what-can-machine-learning-pipelines-do"></a>¿Qué pueden hacer las canalizaciones de Machine Learning?

Una canalización de Azure Machine Learning es un flujo de trabajo ejecutable independiente de una tarea de aprendizaje automático completa. Las subtareas se encapsulan como una serie de pasos en la canalización. Una canalización de Azure Machine Learning puede ser tan simple como una llamada a un script de Python, por lo que _puede_ hacer prácticamente de todo. Las canalizaciones _se deben_ centrar en tareas de aprendizaje automático como:

+ La preparación de datos, incluida la importación, validación y limpieza, manipulación y transformación, normalización y almacenamiento provisional.
+ La configuración del entrenamiento, incluida la parametrización de argumentos, rutas de archivos y configuraciones de registros o informes.
+ Entrenamiento y validación de forma eficaz y repetida. La eficiencia puede provenir de especificar subconjuntos de datos específicos, distintos recursos de proceso de hardware, el procesamiento distribuido y la supervisión del progreso
+ La implementación, incluido el control de versiones, escalado, aprovisionamiento y control de acceso.

Los pasos independientes permiten que varios científicos de datos trabajen en la misma canalización al mismo tiempo sin necesidad de sobrecargar los recursos de proceso. Los pasos independientes también facilitan el uso de distintos tamaños y tipos de proceso para cada paso.

Una vez diseñada la canalización, a menudo se aplican más ajustes en torno al bucle de aprendizaje de la canalización. Cuando se vuelve a ejecutar una canalización, la ejecución salta a los pasos necesarios para volver a ejecutarse, como un script de entrenamiento actualizado. Se omiten los pasos que no es necesario volver a ejecutar. 

Con las canalizaciones, puede elegir usar hardware diferente para distintas tareas. Azure coordina los distintos [destinos de proceso](concept-azure-machine-learning-architecture.md) que se usan para que los datos intermedios fluyan sin problema a los destinos del proceso de bajada.

Puede [realizar un seguimiento de las métricas de los experimentos de canalización](./how-to-log-view-metrics.md) directamente en Azure Portal o en la [página de aterrizaje del área de trabajo (versión preliminar)](https://ml.azure.com). Una vez que se publica una canalización, puede configurar un punto de conexión REST que le permite volver a ejecutar la canalización desde cualquier plataforma o pila.

En resumen, las canalizaciones pueden ayudar con todas las tareas complejas del ciclo de vida del aprendizaje automático. Otras tecnologías de canalización de Azure tienen sus propias ventajas. Las [canalizaciones de Azure Data Factory](../data-factory/concepts-pipelines-activities.md) destaca en el trabajo con datos y [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) es la herramienta correcta para la implementación y la integración continua. Pero si su enfoque es el aprendizaje automático, es probable que las canalizaciones de Azure Machine Learning sean la mejor opción para sus necesidades de flujo de trabajo. 

### <a name="analyzing-dependencies"></a>Análisis de dependencias

Muchos ecosistemas de programación tienen herramientas que organizan las dependencias de recursos, bibliotecas o compilaciones. Por lo general, estas herramientas usan marcas de tiempo de archivo para calcular las dependencias. Cuando se cambia un archivo, solo se actualiza el archivo y sus elementos dependientes (se descargan, recompilan o empaquetan). Las canalizaciones de Azure Machine Learning amplían este concepto. Al igual que las herramientas de compilación tradicionales, las canalizaciones calculan las dependencias entre los pasos y solo realizan los recálculos necesarios. 

El análisis de dependencias de las canalizaciones de Azure Machine Learning es más sofisticado que una simple marca de tiempo. Cada paso puede ejecutarse en un entorno de hardware y software diferente. La preparación de los datos puede ser un proceso lento, pero no es necesario ejecutarlo en hardware con GPU eficaces, es posible que algunos pasos requieran software específico del sistema operativo, puede que quiera usar entrenamiento distribuido, etc. 

Azure Machine Learning organiza automáticamente todas las dependencias entre los pasos de la canalización. Esta orquestación podría incluir la rotación de imágenes de Docker, la asociación y desasociación de recursos de proceso y el movimiento de datos entre los pasos de manera coherente y automática.

### <a name="coordinating-the-steps-involved"></a>Coordinación de los pasos implicados

Al crear y ejecutar un objeto `Pipeline`, se producen los siguientes pasos de alto nivel:

+ En cada paso, el servicio calcula los requisitos de:
    + Recursos de proceso de hardware
    + Recursos del sistema operativo (imágenes de Docker)
    + Recursos de software (dependencias de Conda/virtualenv)
    + Entradas de datos 
+ El servicio determina las dependencias entre los pasos, lo que da como resultado un grafo de ejecución dinámico.
+ Cuando se ejecuta cada nodo del grafo de ejecución:
    + El servicio configura el entorno de hardware y software necesario (quizás reutilizando los recursos existentes).
    + El paso se ejecuta, lo que proporciona información de registro y supervisión al objeto `Experiment` que lo contiene.
    + Cuando se completa el paso, sus salidas se preparan como entradas al paso siguiente o se escriben en el almacenamiento.
    + Los recursos que ya no son necesarios se finalizan y desasocian.

![Pasos de la canalización](./media/concept-ml-pipelines/run_an_experiment_as_a_pipeline.png)

## <a name="building-pipelines-with-the-python-sdk"></a>Creación de canalizaciones con el SDK de Python

En el [SDK de Python para Azure Machine Learning ](/python/api/overview/azure/ml/install), una canalización es un objeto de Python definido en el módulo de `azureml.pipeline.core`. Un objeto [Pipeline](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline%28class%29) contiene una secuencia ordenada de uno o varios objetos [PipelineStep](/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep). La clase `PipelineStep` es abstracta y los pasos reales serán de subclases como [EstimatorStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimatorstep), [PythonScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.pythonscriptstep) o [DataTransferStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep). La clase [ModuleStep ](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.modulestep) contiene una secuencia reutilizable de pasos que se pueden compartir entre las canalizaciones. Un `Pipeline` se ejecuta como parte de un `Experiment`.

Una canalización de Azure Machine Learning está asociada a un área de trabajo de Azure Machine Learning y un paso de la canalización está asociado a un destino de proceso disponible dentro de esa área de trabajo. Para más información, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning en Azure Portal](./how-to-manage-workspace.md) o [¿Qué son los destinos de proceso en Azure Machine Learning?](./concept-compute-target.md)

### <a name="a-simple-python-pipeline"></a>Una canalización de Python sencilla

Este fragmento de código muestra los objetos y las llamadas necesarios para crear y ejecutar un `Pipeline`:

```python
ws = Workspace.from_config() 
blob_store = Datastore(ws, "workspaceblobstore")
compute_target = ws.compute_targets["STANDARD_NC6"]
experiment = Experiment(ws, 'MyExperiment') 

input_data = Dataset.File.from_files(
    DataPath(datastore, '20newsgroups/20news.pkl'))
prepped_data_path = OutputFileDatasetConfig(name="output_path")

dataprep_step = PythonScriptStep(
    name="prep_data",
    script_name="dataprep.py",
    source_directory="prep_src",
    compute_target=compute_target,
    arguments=["--prepped_data_path", prepped_data_path],
    inputs=[input_dataset.as_named_input('raw_data').as_mount() ]
    )

prepped_data = prepped_data_path.read_delimited_files()

train_step = PythonScriptStep(
    name="train",
    script_name="train.py",
    compute_target=compute_target,
    arguments=["--prepped_data", prepped_data],
    source_directory="train_src"
)
steps = [ dataprep_step, train_step ]

pipeline = Pipeline(workspace=ws, steps=steps)

pipeline_run = experiment.submit(pipeline)
pipeline_run.wait_for_completion()
```

El fragmento de código se inicia con objetos comunes de Azure Machine Learning, un `Workspace`, a `Datastore`, un [ComputeTarget](/python/api/azureml-core/azureml.core.computetarget) y un `Experiment`. Luego, el código crea los objetos para contener `input_data` y `prepped_data_path`. El objeto `input_data` es una instancia de [FileDataset](/python/api/azureml-core/azureml.data.filedataset) y el objeto `prepped_data_path` es una instancia de [OutputFileDatasetConfig](/python/api/azureml-core/azureml.data.output_dataset_config.outputfiledatasetconfig). En el caso de `OutputFileDatasetConfig`, el comportamiento predeterminado es copiar la salida en el almacén de datos `workspaceblobstore` en la ruta de acceso `/dataset/{run-id}/{output-name}`, donde `run-id` es el identificador de la ejecución y `output-name` es un valor generado automáticamente si no lo especifica el desarrollador.

El código de preparación de datos (no mostrado) escribe archivos delimitados en `prepped_data_path`. Estas salidas del paso de preparación de datos se pasan como `prepped_data` al paso de entrenamiento. 

La matriz `steps` contiene los dos `PythonScriptStep`, `dataprep_step` y `train_step`. Azure Machine Learning analiza la dependencia de los datos de `prepped_data` y ejecuta `dataprep_step` antes que `train_step`. 

A continuación, el código crea una instancia del propio objeto `Pipeline` y pasa la matriz de pasos y el área de trabajo. La llamada a `experiment.submit(pipeline)` inicia la ejecución de la canalización de Azure ML. La llamada a `wait_for_completion()` se bloquea hasta que finaliza la canalización. 

Para más información sobre cómo conectar la canalización con los datos, consulte los artículos [Acceso a los datos en Azure Machine Learning](concept-data.md) y [Movimiento de datos a los pasos de canalización de Machine Learning (Python) y entre ellos](how-to-move-data-in-out-of-pipelines.md). 

## <a name="building-pipelines-with-the-designer"></a>Creación de canalizaciones con el diseñador

Los desarrolladores que prefieren una superficie de diseño visual pueden usar el diseñador de Azure Machine Learning para crear canalizaciones. Puede acceder a esta herramienta desde la selección de **Diseñador** en la página principal del área de trabajo.  El diseñador le permite arrastrar y colocar pasos en la superficie de diseño. 

Al diseñar las canalizaciones visualmente, las entradas y salidas de un paso se muestran de manera visible. Puede arrastrar y colocar conexiones de datos, lo que le permite comprender y modificar rápidamente el flujo de datos de la canalización.

![Ejemplo del diseñador de Azure Machine Learning](./media/concept-designer/designer-drag-and-drop.gif)

## <a name="key-advantages"></a>Ventajas clave

Las ventajas clave de usar canalizaciones para los flujos de trabajo de aprendizaje automático son:

|Ventaja clave|Descripción|
|:-------:|-----------|
|**Ejecuciones&nbsp;desatendidas**|Programe los pasos para la ejecución en paralelo o en secuencia de manera confiable y desatendida. La preparación de datos y su modelado pueden durar días o semanas y las canalizaciones le permiten centrarse en otras tareas mientras se ejecuta el proceso. |
|**Proceso heterogéneo**|Utilice varias canalizaciones coordinadas de forma confiable en ubicaciones de almacenamiento y recursos de procesos heterogéneos y escalables. Use de manera eficaz los recursos de proceso disponibles mediante la ejecución de pasos individuales de la canalización en diferentes destinos de proceso, como HDInsight, instancias de Data Science VM de la GPU y Databricks.|
|**Reusabilidad**|Cree plantillas de canalizaciones para escenarios específicos, como reentrenamiento y puntuación por lotes. Desencadene las canalizaciones publicadas desde sistemas externos a través de llamadas REST sencillas.|
|**Seguimiento y control de versiones**|En lugar de hacer un seguimiento manual de los datos y los resultados conforme itera, use el SDK de canalizaciones para asignar de forma explícita un nombre y una versión a los orígenes de datos, las entradas y las salidas. También puede administrar scripts y datos por separado para aumentar la productividad.|
| **Modularidad** | Separar las áreas de preocupación y aislar los cambios permite que el software evolucione más rápidamente con una mayor calidad. | 
|**Colaboración**|Las canalizaciones permiten que los científicos de datos colaboren en todas las áreas del proceso de diseño de aprendizaje automático, a la vez que pueden trabajar de manera simultánea en los pasos de la canalización.|

## <a name="next-steps"></a>Pasos siguientes

Las canalizaciones de Azure Machine Learning constituyen un potente recurso que ya comienza a ofrecer valor en las primeras fases del desarrollo. El valor aumenta a medida que crece el equipo y el proyecto. En este artículo se ha explicado cómo se especifican las canalizaciones con el SDK de Azure Machine Learning Python y cómo se organizan en Azure. Ha visto código fuente simple y ha conocido algunas de las clases de `PipelineStep` que están disponibles. Ya debe tener una idea de cuándo usar las canalizaciones de Azure Machine Learning y cómo las ejecuta Azure. 

+ Aprenda a [crear su primera canalización](./how-to-create-machine-learning-pipelines.md).

+ Aprenda a [ejecutar predicciones por lotes en grandes cantidades de datos](tutorial-pipeline-batch-scoring-classification.md ).

+ Consulte los documentos de referencia del SDK para conocer el [centro de las canalizaciones](/python/api/azureml-pipeline-core/) y los [pasos de canalización](/python/api/azureml-pipeline-steps/).

+ Pruebe los cuadernos de Jupyter Notebook que muestran [canalizaciones de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines). Aprenda a [ejecutar cuadernos para explorar este servicio](samples-notebooks.md).