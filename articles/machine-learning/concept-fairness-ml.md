---
title: Equidad en el aprendizaje automático (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre la equidad en el aprendizaje automático y cómo el paquete Fairlearn de Python puede ayudarle a evaluar y mitigar la parcialidad.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: luquinta
author: luisquintanilla
ms.date: 04/02/2021
ms.custom: responsible-ml
ms.openlocfilehash: 45c63780eac021301e5b01c0ff8abea7d704a4cf
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220528"
---
# <a name="machine-learning-fairness-preview"></a>Equidad en el aprendizaje automático (versión preliminar)

Obtenga información sobre la equidad en el aprendizaje automático y cómo el paquete de Python de código abierto [Fairlearn](https://fairlearn.github.io/) puede ayudarle a mitigar los problemas de parcialidad en los modelos de Machine Learning. 

## <a name="what-is-machine-learning-fairness"></a>¿Qué es la equidad en el aprendizaje automático?

La inteligencia artificial y los sistemas de aprendizaje automático pueden mostrar un comportamiento parcial. Una manera de definir el comportamiento parcial es fijarse en el daño que hace o impacto que tiene en las personas. Existen muchos tipos de daños que los sistemas de IA pueden ocasionar. Para más información, consulte la [nota clave de NeurIPS 2017 de Kate Crawford](https://www.youtube.com/watch?v=fMym_BKWQzk).

Estos son dos tipos comunes de daños que puede causar la IA:

- Daño en la asignación: Un sistema de inteligencia artificial extiende o retiene oportunidades, recursos o información para determinados grupos. Entre los ejemplos se incluyen la contratación, las admisiones escolares y los préstamos, donde un modelo podría ser mucho mejor a la hora de seleccionar buenos candidatos entre un grupo específico de personas que entre otros grupos.

- Daño en la calidad del servicio: un sistema de AI puede no funcionar tan bien para un grupo de personas como lo haría en otro. Por ejemplo, un sistema de reconocimiento de voz puede no funcionar correctamente para las mujeres, pero sí para los hombres.

Para reducir el comportamiento parcial de los sistemas de AI, tiene que evaluar y mitigar estos daños.

## <a name="fairness-assessment-and-mitigation-with-fairlearn"></a>Evaluación y mitigación de imparcialidad con Fairlearn

Fairlearn es un paquete de Python de código abierto que permite a los desarrolladores de sistemas de aprendizaje automático evaluar la equidad de sus sistemas y mitigar la parcialidad.

>[!NOTE]
> La imparcialidad es un desafío técnico y social. Muchos aspectos de la imparcialidad, tales como la justicia y el proceso de vencimiento, no se capturan en métricas cuantitativas de imparcialidad. Asimismo, muchas métricas cuantitativas de imparcialidad no se pueden satisfacer simultáneamente. El objetivo del paquete de código abierto de Fairlearn es permitir a los usuarios evaluar las distintas estrategias de impacto y mitigación. En última instancia, es responsabilidad de los usuarios humanos crear modelos de inteligencia artificial y aprendizaje automático con el fin de crear ventajas para su escenario.

El paquete de código abierto de Fairlearn tiene dos componentes:

- Panel de evaluación: es un widget de Jupyter Notebook para evaluar cómo afectan las predicciones de un modelo a grupos diferentes. También permite comparar varios modelos mediante métricas de imparcialidad y rendimiento.
- Algoritmos de mitigación: es un conjunto de algoritmos para mitigar la imparcialidad en la clasificación y la regresión binarias.

Juntos, estos componentes permiten a los científicos de datos y a los líderes empresariales administrar cualquier intercambio entre imparcialidad y rendimiento, y seleccionar la estrategia de mitigación que mejor se adapte a sus necesidades.

## <a name="assess-fairness-in-machine-learning-models"></a>Acceso a la equidad en los modelos de aprendizaje automático

En el paquete de código abierto de Fairlearn, la equidad se conceptualiza a través de un enfoque conocido como **equidad de grupo**, que pregunta: ¿Qué grupos de usuarios están en riesgo de experimentar daños? Los grupos pertinentes, también conocidos como subpoblaciónes, se definen a través de **características confidenciales** o atributos confidenciales. Las características confidenciales se pasan a un estimador en el paquete de código abierto de Fairlearn como un vector o una matriz denominada `sensitive_features`. El término sugiere que el diseñador del sistema debe mantener la confidencialidad de estas características al evaluar la imparcialidad del grupo. 

Algo que se debe tener en cuenta es si estas características contienen implicaciones de privacidad debido a datos privados. De todos modos, la palabra "confidencial" no implica que estas características no se utilicen para realizar predicciones.

>[!NOTE]
> Una evaluación de equidad no es un ejercicio puramente técnico.  El paquete de código abierto de Fairlearn puede ayudarle a evaluar la equidad de un modelo, pero no llevará a cabo la evaluación.  El paquete de código abierto Fairlearn ayuda a identificar métricas cuantitativas para evaluar la equidad, pero los desarrolladores también deben realizar un análisis cualitativo para evaluar la equidad de sus propios modelos.  Las características confidenciales indicadas anteriormente son un ejemplo de este tipo de análisis cualitativo.     

Durante la fase de valoración, la imparcialidad se cuantifica a través de las métricas de disparidad. **Las métricas de disparidad** pueden evaluar y comparar el comportamiento del modelo en grupos diferentes, ya sea como proporciones o diferencias. El paquete de código abierto Fairlearn admite dos clases de métricas de disparidad:


- Disparidad en el rendimiento del modelo: estos conjuntos de métricas calculan la disparidad (diferencia) en los valores de la métrica de rendimiento seleccionada en los distintos subgrupos. Estos son algunos ejemplos:

  - disparidad en la tasa de precisión
  - disparidad en la tasa de error
  - disparidad en la precisión
  - disparidad en la coincidencia
  - disparidad en la métrica MAE
  - muchos otros

- Disparidad en la tasa de selección: esta métrica contiene la diferencia de la tasa de selección entre distintos subgrupos. Un ejemplo de esto es la disparidad en la tasa de aprobación de préstamos. La tasa de selección indica la fracción de los puntos de referencia de cada clase clasificada como 1 (en la clasificación binaria) o la distribución de los valores de predicción (en la regresión).

## <a name="mitigate-unfairness-in-machine-learning-models"></a>Mitigación de la parcialidad en los modelos de aprendizaje automático

### <a name="parity-constraints"></a>Restricciones de paridad

El paquete de código abierto Fairlearn incluye una variedad de algoritmos de mitigación de la imparcialidad. Estos algoritmos admiten un conjunto de restricciones en el comportamiento del predictor denominadas **restricciones de paridad** o criterios. Las restricciones de paridad requieren que algunos aspectos del comportamiento de predicción sean comparables en los grupos que definen las características confidenciales (por ejemplo, razas diferentes). Los algoritmos de mitigación en el paquete de código abierto de Fairlearn usan estas restricciones de paridad para mitigar los problemas de equidad observados.

>[!NOTE]
> La mitigación de la imparcialidad en un modelo significa reducir la imparcialidad, pero esta mitigación técnica no puede eliminar por completo esta imparcialidad.  Los algoritmos de mitigación de la imparcialidad en el paquete de código abierto de Fairlearn pueden proporcionar estrategias de mitigación sugeridas para reducir la imparcialidad en un modelo de aprendizaje automático, pero no son soluciones para eliminar la imparcialidad por completo.  Puede que haya otras restricciones o criterios de paridad que se deben tener en cuenta para cada modelo de aprendizaje automático de un desarrollador determinado. Los desarrolladores que usan Azure Machine Learning deben determinar por sí mismos si la mitigación elimina suficientemente cualquier injusticia en el uso previsto y la implementación de los modelos de aprendizaje automático.  

El paquete de código abierto Fairlearn admite los siguientes tipos de restricciones de paridad: 

|Restricción de paridad  | Propósito  |Tarea de Machine Learning  |
|---------|---------|---------|
|Paridad demográfica     |  Mitigar los daños en la asignación | Clasificación binaria, regresión |
|Probabilidades igualadas  | Diagnosticar los daños en la asignación y la calidad del servicio | Clasificación binaria        |
|Igualdad de oportunidades | Diagnosticar los daños en la asignación y la calidad del servicio | Clasificación binaria        |
|Pérdida de grupos limitada     |  Mitigar los daños en la calidad del servicio | Regresión |

### <a name="mitigation-algorithms"></a>Algoritmos de mitigación

El paquete de código abierto de Fairlearn proporciona algoritmos de mitigación de parcialidad de postprocesamiento y reducción:

- Reducción: Estos algoritmos toman un estimador de aprendizaje automático estándar de caja negra (por ejemplo, un modelo de LightGBM) y generan un conjunto de modelos que se vuelven a entrenar con una secuencia de conjuntos de datos de entrenamiento que se han vuelto a ponderar. Por ejemplo, los solicitantes de un sexo determinado podrían estar ponderados en mayor o menor medida para volver a entrenar los modelos y reducir así las disparidades entre los diferentes sexos. A continuación, los usuarios pueden elegir un modelo que les ofrezca el mejor equilibrio entre la precisión (u otra métrica de rendimiento) y la disparidad, lo que generalmente debería basarse en reglas de negocios y cálculos de costos.  
- Posprocesamiento: estos algoritmos usan un clasificador existente y la característica confidencial como entrada. A continuación, derivan una transformación de la predicción del clasificador para aplicar las restricciones de imparcialidad especificadas. La mayor ventaja de la optimización del umbral es su simplicidad y flexibilidad, ya que no es necesario volver a entrenar el modelo. 

| Algoritmo | Descripción | Tarea de Machine Learning | Características confidenciales | Restricciones de paridad admitidas | Tipo de algoritmo |
| --- | --- | --- | --- | --- | --- |
| `ExponentiatedGradient` | Enfoque de la caja negra para la clasificación imparcial descrita en [Enfoque de reducción para la clasificación imparcial](https://arxiv.org/abs/1803.02453) | Clasificación binaria | Categorías | [Paridad demográfica](#parity-constraints), [Probabilidades igualadas](#parity-constraints) | Reducción |
| `GridSearch` | Enfoque de la caja descrita en [Enfoque de reducción para la clasificación imparcial](https://arxiv.org/abs/1803.02453)| Clasificación binaria | Binary | [Paridad demográfica](#parity-constraints), [Probabilidades igualadas](#parity-constraints) | Reducción |
| `GridSearch` | Enfoque de la caja negra que implementa una variante de la búsqueda de cuadrícula de la regresión imparcial con el algoritmo, para la pérdida de grupos limitada descrita en [Regresión imparcial: Definiciones cuantitativas y algoritmos basados en la reducción](https://arxiv.org/abs/1905.12843) | Regresión | Binary | [Pérdida de grupos limitada](#parity-constraints) | Reducción |
| `ThresholdOptimizer` | Algoritmo de posprocesamiento basado en el artículo [La igualdad de oportunidades en el aprendizaje supervisado](https://arxiv.org/abs/1610.02413). Esta técnica toma como entrada un clasificador existente y la característica confidencial, y deriva una transformación monótona de la predicción del clasificador para aplicar las restricciones de paridad especificadas. | Clasificación binaria | Categorías | [Paridad demográfica](#parity-constraints), [Probabilidades igualadas](#parity-constraints) | Posprocesamiento |

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a usar los diferentes componentes al consultar [GitHub](https://github.com/fairlearn/fairlearn/), la [guía del usuario](https://fairlearn.github.io/main/user_guide/index.html), [ejemplos](https://fairlearn.github.io/main/auto_examples/index.html) y [cuadernos de muestra](https://github.com/fairlearn/fairlearn/tree/master/notebooks) de Fairlearn.
- Aprenda [cómo](how-to-machine-learning-fairness-aml.md) habilitar la evaluación de equidad de los modelos de aprendizaje automático en Azure Machine Learning.
- Consulte los [cuadernos de muestra](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness) para ver escenarios adicionales de evaluación de equidad en Azure Machine Learning. 
