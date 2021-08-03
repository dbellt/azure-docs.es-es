---
title: 'Entrenar modelo: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo **Entrenar modelo** en Azure Machine Learning para entrenar un modelo de clasificación o regresión.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/10/2021
ms.openlocfilehash: 4bbb85593d41b56ba465f577c3be35de43ba610a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468620"
---
# <a name="train-model-module"></a>Módulo Entrenar modelo

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para entrenar un modelo de clasificación o regresión. El entrenamiento tiene lugar después de definir un modelo y establecer sus parámetros y requiere datos etiquetados. También puede usar **Entrenar modelo** para volver a entrenar un modelo existente con datos nuevos. 

## <a name="how-the-training-process-works"></a>Funcionamiento del proceso de entrenamiento

En Azure Machine Learning, crear y usar un modelo suele ser un proceso de tres pasos. 

1. Un modelo se configura eligiendo un determinado tipo de algoritmo y definiendo sus parámetros o hiperparámetros. Elija cualquiera de los siguientes tipos de modelo: 

    + Los modelos de **clasificación**, que se basan en redes neuronales, árboles de decisión y bosques de decisión, así como otros algoritmos.
    + Los modelos de **regresión**, que pueden incluir la regresión lineal estándar o que usan otros algoritmos, incluidas las redes neuronales y la regresión bayesiana.  

2. Proporcione un conjunto de datos que esté etiquetado y tenga datos compatibles con el algoritmo. Conecte los datos y el modelo a **Entrenar modelo**.

    El entrenamiento produce un formato binario concreto, iLearner, que encapsula los patrones estadísticos aprendidos a partir de los datos. No se puede modificar o leer directamente este formato; sin embargo, otros módulos pueden usar el modelo entrenado. 
    
    También puede ver las propiedades del modelo. Para más información, consulte la sección Resultados.

3. Una vez completado el entrenamiento, utilice el modelo entrenado con uno de los [módulos de puntuación](./score-model.md), para realizar predicciones sobre nuevos datos.

## <a name="how-to-use-train-model"></a>Procedimiento para usar Entrenar modelo 
    
1. Agregue el módulo **Entrenamiento de modelo** a la canalización.  Puede encontrar este módulo en la categoría **Machine Learning**. Expanda **Entrenar** y luego arrastre el módulo **Entrenamiento de modelo** a la canalización.
  
1.  En la entrada izquierda, adjunte el modo no entrenado. Adjunte el conjunto de datos de entrenamiento a la entrada de la derecha de **Entrenar modelo**.

    El conjunto de datos de entrenamiento debe contener una columna de etiqueta. Se omiten todas las filas sin etiquetas.
  
1.  En **Label column** (Columna de etiqueta), haga clic en **Edit column** (Editar columna) en el panel derecho del módulo y elija una columna que contenga resultados que el modelo pueda usar para el entrenamiento.
  
    - Para problemas de clasificación, la columna de etiqueta debe contener valores **categoriales** o valores **discretos**. Algunos ejemplos podrían ser una calificación de sí o no, un código o nombre de clasificación de enfermedades o un grupo de ingresos.  Si elige una columna no categorial, el módulo devolverá un error durante el entrenamiento.
  
    -   Para problemas de regresión, la columna de etiqueta debe contener datos **numéricos** que representen la variable de respuesta. Lo ideal es que los datos numéricos representen una escala continua. 
    
    Podrían ser ejemplos una puntuación de riesgo de crédito, el tiempo proyectado para el error de una unidad de disco duro o el número previsto de llamadas a un centro de llamadas en un día u hora determinadas.  Si no elige una columna numérica, puede obtener un error.
  
    -   Si no especifica qué columna de etiqueta debe utilizar, Azure Machine Learning intentará inferir cuál es la columna de etiqueta adecuada usando los metadatos del conjunto de datos. Si elige la columna errónea, utilice el selector de columnas para corregirlo.
  
    > [!TIP] 
    > Si tiene problemas para usar el selector de columnas, consulte el artículo [Select Columns in Dataset](./select-columns-in-dataset.md) (Seleccionar columnas en conjunto de datos) para obtener sugerencias. En él, se describen algunos escenarios y sugerencias comunes para usar las opciones **WITH RULES** (CON REGLAS) y **BY NAME** (POR NOMBRE).
  
1.  Envíe la canalización. Si tiene muchos datos, la operación puede tardar.

    > [!IMPORTANT] 
    > Si tiene una columna de identificador que sea el identificador de cada fila o una columna de texto que contenga demasiados valores únicos, **Train Model** (Entrenar modelo) puede producir un error como "Number of unique values in colum: '{column_name}' is greater than allowed" (El número de valores únicos en la columna '{nombre_de_la_columna}' es mayor que lo permitido).
    >
    > Esto se debe a que la columna de identificador alcanza el umbral de valores únicos y puede agotar la memoria. Puede utilizar [Editar metadatos](edit-metadata.md) para marcar esa columna como **Clear feature** (Borrar característica) y no se utilizará en el entrenamiento, o puede emplear el módulo [Extract N-Gram Features from Text module](extract-n-gram-features-from-text.md) (Extracción de características de n-gramas del texto) para preprocesar la columna de texto. Consulte [Códigos de error para el diseñador](././designer-error-codes.md) para más detalles sobre este error.

## <a name="model-interpretability"></a>Interpretabilidad del modelo

La interpretabilidad del modelo ofrece la posibilidad de comprender el modelo de aprendizaje automático y de presentar la base subyacente para la toma de decisiones de una manera comprensible para los seres humanos.

Actualmente, el módulo **Train Model** (Entrenar modelo) admite el [uso del paquete de interpretabilidad para explicar los modelos de aprendizaje automático](../how-to-machine-learning-interpretability-aml.md#generate-feature-importance-values-via-remote-runs). Se admiten los siguientes algoritmos integrados:

- Regresión lineal
- Regresión de red neuronal
- Regresión ampliada del árbol de decisión
- Regresión de bosque de decisión
- Regresión de Poisson
- Regresión logística de dos clases
- Máquina de vectores de soporte de dos clases
- Árbol de decisión ampliado de dos clases
- Bosque de decisión de dos clases
- Bosque de decisión de varias clases
- Regresión logística de varias clases
- Red neuronal de varias clases

Para generar explicaciones del modelo, puede seleccionar **Verdadero** en la lista desplegable **Model Explanation** (Explicación del modelo) del módulo Train Model (Entrenar modelo). De forma predeterminada, se establece en Falso en el módulo **Train Model** (Entrenar modelo). Tenga en cuenta que la generación de una explicación requiere costos de proceso adicionales.

![Captura de pantalla que muestra la casilla de explicación del modelo](./media/module/train-model-explanation-checkbox.png)

Una vez completada la ejecución de la canalización, puede visitar la pestaña **Explanations** (Explicaciones) en el panel derecho del módulo **Train Model** (Entrenar modelo) y explorar el rendimiento del modelo, el conjunto de datos y la importancia de las características.

![Captura de pantalla que muestra los gráficos de explicación del modelo](./media/module/train-model-explanations-tab.gif)

Para más información sobre el uso de las explicaciones del modelo en Azure Machine Learning, consulte el artículo de procedimientos sobre cómo [interpretar los modelos de aprendizaje automático](../how-to-machine-learning-interpretability-aml.md#generate-feature-importance-values-via-remote-runs).

## <a name="results"></a>Results

Cuando el modelo está entrenado:


+ Para usar el modelo en otras canalizaciones, seleccione el módulo y, después, seleccione el icono **Register dataset** (Registrar conjunto de datos) en la pestaña **Outputs** (Salidas) del panel derecho. Puede acceder a los modelos guardados en la paleta de módulos en **Datasets** (Conjuntos de datos).

+ Para usar el modelo para predecir nuevos valores, conéctelo al módulo [Puntuar modelo](./score-model.md), junto con nuevos datos de entrada.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning.