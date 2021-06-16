---
title: Entrenamiento del modelo de PyTorch
titleSuffix: Azure Machine Learning
description: Use el módulo Entrenamiento del modelo de PyTorch en el diseñador de Azure Machine Learning para entrenar modelos desde cero o para ajustar los modelos existentes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 03/19/2021
ms.openlocfilehash: cfc866a7b7a63982377f8a2498b9538bd870e381
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111969124"
---
# <a name="train-pytorch-model"></a>Entrenamiento del modelo de PyTorch

En este artículo se describe cómo usar el módulo **Entrenamiento del modelo de PyTorch** en el diseñador de Azure Machine Learning para entrenar modelos de PyTorch como DenseNet. El entrenamiento tiene lugar después de definir un modelo y establecer sus parámetros y requiere datos etiquetados. 

Actualmente, el módulo **Entrenamiento del modelo de PyTorch** admite el entrenamiento distribuido y de nodo único.

## <a name="how-to-use-train-pytorch-model"></a>Cómo usar Entrenamiento del modelo de PyTorch 

1. Agregue los módulos [DenseNet](densenet.md) o [ResNet](resnet.md) al borrador de la canalización en el diseñador.

2. Agregue el módulo **Entrenamiento del modelo de PyTorch** a la canalización. Puede encontrar este módulo en la categoría **Entrenamiento del modelo**. Expanda **Entrenar** y luego arrastre el módulo **Entrenamiento del modelo de PyTorch** a la canalización.

   > [!NOTE]
   > El módulo **Entrenamiento del modelo de PyTorch** se ejecuta mejor en un proceso de tipo **GPU** para conjuntos de datos grandes; de lo contrario, se producirá un error en la canalización. Puede seleccionar el proceso para un módulo específico en el panel derecho del módulo estableciendo **Use other compute target** (Utilizar otro destino de proceso).

3.  En la entrada izquierda, adjunte un modo no entrenado. Conecte el conjunto de datos de entrenamiento y el conjunto de datos de validación a la entrada de la parte central y derecha de **Entrenamiento del modelo de PyTorch**.

    En el caso de un modelo no entrenado, debe ser un modelo de PyTorch como DenseNet; de lo contrario, se producirá un error "InvalidModelDirectoryError".

    En el caso de un conjunto de datos, el conjunto de datos de entrenamiento debe ser un directorio de imagen con etiquetas. Para más información sobre cómo obtener un directorio de imagen con etiquetas, consulte **Conversión a directorio de imagen**. Si no está etiquetado, se producirá una un error "NotLabeledDatasetError".

    El conjunto de datos de entrenamiento y el conjunto de datos de validación deben tener las mismas categorías de etiqueta; de lo contrario, se producirá un error InvalidDatasetError.

4.  En **Epochs** (Épocas), especifique el número de épocas que le gustaría entrenar. Se iterará todo el conjunto de valores cada época, de forma predeterminada 5.

5.  Para **Tamaño del lote**, especifique el número de instancias que se van a entrenar en un lote, de forma predeterminada 16.

6.  En **Warmup step number** (número de paso de preparación), especifique durante cuántas épocas quiere preparar el entrenamiento, en caso de que la velocidad de aprendizaje inicial sea un poco mayor para empezar a converger, de manera predeterminada es 0.

7.  En **Velocidad de aprendizaje**, especifique un valor para la *velocidad de aprendizaje*; el valor predeterminado es 0,001. La velocidad de aprendizaje controla el tamaño del paso que se usa en el optimizador, como sgd, cada vez que se prueba y se corrige el modelo.

    Al establecer una velocidad menor, se prueba el modelo más a menudo, con el riesgo de que pueda quedarse atascado en un nivel local. Al establecer una velocidad mayor, puede convergir con mayor rapidez, con el riesgo de superar los mínimos verdaderos.

    > [!NOTE]
    > Si la pérdida de entrenamiento se convierte en NaN durante el entrenamiento, lo que puede deberse a una velocidad de aprendizaje demasiado alta, disminuir la velocidad de aprendizaje puede ayudar.
    > En el entrenamiento distribuido, para mantener estable el descenso de gradiente, se calcula la velocidad de aprendizaje real mediante `lr * torch.distributed.get_world_size()`, ya que el tamaño de lote del grupo de procesos es igual al de la muestra completa multiplicado por ese único proceso.
    > Se aplica la decadencia polinómica de la velocidad de aprendizaje y puede ayudar a mejorar el rendimiento del modelo.

8.  En **Random seed** (Inicialización aleatoria), escriba opcionalmente un valor de entero que se usará como inicialización. Se recomienda usar una inicialización si desea garantizar la reproducibilidad del experimento a través de las ejecuciones.

9.  Por **Patience** (Paciencia), especifique para cuántas épocas quiere detener el entrenamiento con antelación si la pérdida de validación no se reduce consecutivamente. De forma predeterminada 3.

10. En **Frecuencia de impresión**, especifique la frecuencia de impresión del registro de entrenamiento en iteraciones por cada época; de manera predeterminada es 10.

11. Envíe la canalización. Si el conjunto de datos tiene un tamaño mayor, tardará un tiempo, por lo que se recomienda un proceso de GPU.

## <a name="distributed-training"></a>Entrenamiento distribuido

En el entrenamiento distribuido, la carga de trabajo para entrenar un modelo se divide y se comparte entre varios procesadores pequeños, denominados nodos de trabajo. Estos nodos de trabajo trabajan en paralelo para acelerar el entrenamiento del modelo. Actualmente, el diseñador admite el entrenamiento distribuido para el módulo **Entrenamiento del modelo de PyTorch**.

### <a name="training-time"></a>Tiempo de entrenamiento

El entrenamiento distribuido permite entrenar en un conjunto de datos de gran tamaño, como ImageNet (1000 clases, 1,2 millones de imágenes) en tan solo algunas horas mediante **Entrenamiento del modelo de PyTorch**. En la tabla siguiente se muestra el tiempo de entrenamiento y el rendimiento durante el entrenamiento de 50 épocas de Resnet50 en ImageNet desde cero según diferentes dispositivos.

| Dispositivos       | Tiempo de entrenamiento  | Rendimiento de entrenamiento  | Precisión de validación del primer resultado | Precisión de validación de los primeros cinco resultados |
| ------------- | -------------- | -------------------- | ------------------------- | ------------------------- |
| 16 V100 GPU  | 6 h 22 min        | ~3200 imágenes/s     | 68,83 %                    | 88,84 %                    | 
| 8 V100 GPU   | 12 h 21 min       | ~1670 imágenes/s     | 68,84 %                    | 88,74 %                    |

Haga clic en la pestaña "Métricas" de este módulo y consulte los grafos de métricas de entrenamiento, como "Train images per second" (Imágenes de entrenamiento por segundo) y "Top-1 accuracy" (Precisión del primer resultado).

[![Captura de pantalla que muestra las métricas de entrenamiento](./media/module/train-pytorch-model-train-metrics.png)](././media/module/train-pytorch-model-train-metrics.png#lightbox)

### <a name="how-to-enable-distributed-training"></a>Cómo habilitar el entrenamiento distribuido

Para habilitar el entrenamiento distribuido para el módulo **Entrenamiento del modelo de PyTorch**, puede establecerlo desde **Configuración de ejecución** en el panel derecho del módulo. Solo se admite el **[clúster de proceso de AML](../how-to-create-attach-compute-cluster.md?tabs=python)** para el entrenamiento distribuido.

> [!NOTE]
> Se necesitan **varias GPU** para activar el entrenamiento distribuido porque el módulo Entrenamiento del modelo de PyTorch de back-end de NCCL necesita CUDA.

1. Seleccione el módulo y abra el panel derecho. Expanda la sección **Parámetros de ejecución**.

    [![Captura de pantalla que muestra cómo establecer el entrenamiento distribuido en Parámetros de ejecución](./media/module/distributed-training-run-setting.png)](./media/module/distributed-training-run-setting.png#lightbox)

2. Asegúrese de que ha seleccionado Proceso de AML para el destino de proceso.

3. En la sección **Resource layout** (diseño de recursos), debe establecer los valores siguientes:

    - **Recuento de nodos**: número de nodos en el destino de proceso usados para el entrenamiento. Debe ser **menor o igual que** el **Número máximo de nodos** del clúster de proceso. De manera predeterminada, tiene un valor de 1, lo que significa un único trabajo de nodo.

    - **Process count per node** (recuento de procesos por nodo): número de procesos desencadenados por nodo. Debe ser **menor o igual que** la **Unidad de procesamiento** del proceso. De manera predeterminada, tiene un valor de 1, lo que significa un único trabajo de proceso.

    Haga clic en el nombre del proceso en la página de detalles del proceso correspondiente para comprobar el **Número máximo de nodos** y la **Unidad de procesamiento**.

    [![Captura de pantalla que muestra cómo comprobar un clúster de proceso](./media/module/compute-cluster-node.png)](./media/module/compute-cluster-node.png#lightbox)

Puede obtener más información acerca del entrenamiento distribuido en Azure Machine Learning [aquí](../concept-distributed-training.md).

### <a name="troubleshooting-for-distributed-training"></a>Solución de problemas de entrenamiento distribuido

Si habilita el entrenamiento distribuido para este módulo, habrá registros de controladores para cada proceso. `70_driver_log_0` es para el proceso maestro. Puede comprobar los registros de controladores para obtener los detalles de errores de cada proceso en la pestaña **Resultados y registros** del panel derecho.

[![Captura de pantalla que muestra el registro de controladores](./media/module/distributed-training-error-driver-log.png)](./media/module/distributed-training-error-driver-log.png#lightbox) 

Si se produce un error en el entrenamiento distribuido habilitado para el módulo sin ningún registro `70_driver`, puede comprobar `70_mpi_log` para ver los detalles del error.

En el ejemplo siguiente se muestra un error común, que implica que el **Recuento de procesos por nodo** es mayor que la **Unidad de procesamiento** del proceso.

[![Captura de pantalla que muestra el registro de mpi](./media/module/distributed-training-error-mpi-log.png)](./media/module/distributed-training-error-mpi-log.png#lightbox)

Puede consultar [este artículo](designer-error-codes.md) para obtener más detalles acerca de la solución de problemas de los módulos.

## <a name="results"></a>Results

Una vez completada la ejecución de la canalización, para usar el modelo para la puntuación, conecte [Entrenamiento del modelo de PyTorch](train-PyTorch-model.md) a [Puntuación del modelo de imagen](score-image-model.md), para predecir valores para los nuevos ejemplos de entrada.

## <a name="technical-notes"></a>Notas técnicas
###  <a name="expected-inputs"></a>Entradas esperadas  

| Nombre               | Tipo                    | Descripción                              |
| ------------------ | ----------------------- | ---------------------------------------- |
| Modelo no entrenado    | UntrainedModelDirectory | Modelo no entrenado, requiere PyTorch         |
| Conjunto de datos de entrenamiento   | ImageDirectory          | Conjunto de datos de entrenamiento                         |
| Conjunto de datos de validación | ImageDirectory          | Conjunto de datos de validación para la evaluación cada época |

###  <a name="module-parameters"></a>Parámetros del módulo  

| Nombre          | Intervalo            | Tipo    | Valor predeterminado | Descripción                              |
| ------------- | ---------------- | ------- | ------- | ---------------------------------------- |
| Épocas        | >0               | Entero | 5       | Seleccionar la columna que contiene la columna de etiqueta o resultado |
| Tamaño de lote    | >0               | Entero | 16      | Número de instancias que se van a entrenar en un lote   |
| Número de paso de preparación | >=0         | Entero | 0       | Número de épocas para preparar el entrenamiento |
| Velocidad de aprendizaje | >=double.Epsilon | Float   | 0,1   | Velocidad de aprendizaje inicial para el optimizador de descenso de gradiente estocástico. |
| Inicialización aleatoria   | Any              | Entero | 1       | La inicialización para el generador de números aleatorios usado por el modelo. |
| Paciencia      | >0               | Entero | 3       | Número de épocas para la detención con antelación del entrenamiento   |
| Frecuencia de impresión |             >0 | Entero | 10      | Frecuencia de impresión del registro de entrenamiento en iteraciones por cada época |

###  <a name="outputs"></a>Salidas  

| Nombre          | Tipo           | Descripción   |
| ------------- | -------------- | ------------- |
| Modelo entrenado | ModelDirectory | Modelo entrenado |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning.