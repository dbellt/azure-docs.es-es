---
title: 'ML Studio (clásico): Migración a Azure Machine Learning'
description: Migre desde Studio (clásico) a Azure Machine Learning para acceder a una plataforma de ciencia de datos modernizada.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: 7bb7c3a2d19b4dbb84d09829e2ed181a62ef7fbe
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111901302"
---
# <a name="migrate-to-azure-machine-learning"></a>Migración a Azure Machine Learning

En este artículo, aprenderá a migrar recursos de Studio (clásico) a Azure Machine Learning. En este momento, para migrar los recursos, debe recompilar manualmente los experimentos.

Azure Machine Learning proporciona una plataforma de ciencia de datos modernizada que combina enfoques sin código y de código primero. Para obtener más información sobre las diferencias entre Studio (clásico) y Azure Machine Learning, consulte la sección [Evaluar Azure Machine Learning](#step-1-assess-azure-machine-learning).


## <a name="recommended-approach"></a>Enfoque recomendado

Para migrar a Azure Machine Learning, se recomienda usar el siguiente enfoque:

> [!div class="checklist"]
> * Paso 1: Evaluar Azure Machine Learning
> * Paso 2: Crear un plan de migración
> * Paso 3: Recompilar los experimentos y servicios web
> * Paso 4: Integrar las aplicaciones cliente
> * Paso 5: Limpiar los recursos de Studio (clásico)


## <a name="step-1-assess-azure-machine-learning"></a>Paso 1: Evaluar Azure Machine Learning
1. Consiga más información sobre [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/): sus ventajas, costos y arquitectura.

1. Compare las funcionalidades de Azure Machine Learning y Studio (clásico).

    >[!NOTE]
    > La característica de **diseñador** de Azure Machine Learning ofrece una experiencia de arrastrar y colocar similar a Studio (clásico). Sin embargo, Azure Machine Learning también proporciona sólidos [flujos de trabajo de código primero](../concept-model-management-and-deployment.md) como alternativa. Esta serie de migración se centra en el diseñador, ya que es más similar a la experiencia de Studio (clásico).

    [!INCLUDE [aml-compare-classic](../../../includes/machine-learning-compare-classic-aml.md)]

3. Compruebe que los módulos esenciales de Studio (clásico) se admiten en el diseñador de Azure Machine Learning. Para obtener más información, consulte la tabla de [asignación de módulos entre Studio (clásico) y el diseñador](#studio-classic-and-designer-module-mapping) a continuación.

4. [Cree un área de trabajo de Azure Machine Learning](../how-to-manage-workspace.md?tabs=azure-portal).

## <a name="step-2-create-a-migration-plan"></a>Paso 2: Crear un plan de migración

1. Identifique los **conjuntos de datos**, **modelos** y **servicios web** de Studio (clásico) que desea migrar.

1. Determine el impacto que una migración tendrá en su empresa.

1. Cree un plan de migración.

## <a name="step-3-rebuild-experiments-and-web-services"></a>Paso 3: Recompilar los experimentos y servicios web

1. [Migre los conjuntos de datos a Azure Machine Learning](migrate-register-dataset.md).
1. Use el diseñador para [recompilar los experimentos](migrate-rebuild-experiment.md).
1. Use el diseñador para [volver a implementar los servicios web](migrate-rebuild-web-service.md).

    >[!NOTE]
    > Azure Machine Learning también admite flujos de trabajo de código primero para [conjuntos de datos](../how-to-create-register-datasets.md), [entrenamiento](../how-to-set-up-training-targets.md) e [implementación](../how-to-deploy-and-where.md).

## <a name="step-4-integrate-client-apps"></a>Paso 4: Integrar las aplicaciones cliente

1. Modifique las aplicaciones cliente que invocan los servicios web de Studio (clásico) para que usen los nuevos [puntos de conexión de Azure Machine Learning](migrate-rebuild-integrate-with-client-app.md).

## <a name="step-5-cleanup-studio-classic-assets"></a>Paso 5: Limpiar los recursos de Studio (clásico)

1. [Limpie los recursos de Studio (clásico)](export-delete-personal-data-dsr.md) para evitar cargos adicionales. Puede que desee conservar recursos como reserva hasta que haya comprobado las cargas de trabajo de Azure Machine Learning.

## <a name="studio-classic-and-designer-module-mapping"></a>Asignación de módulos entre Studio (clásico) y el diseñador

En la tabla siguiente puede consultar qué módulos debe usar al recompilar los experimentos de Studio (clásico) en el diseñador.


> [!IMPORTANT]
> El diseñador implementa los módulos mediante paquetes de Python de código abierto, en lugar de usar paquetes de C# como hace Studio (clásico). Debido a esta diferencia, la salida de los módulos del diseñador puede variar ligeramente de sus homólogos de Studio (clásico).


|Category|Módulo de Studio (clásico)|Módulo del diseñador de reemplazo|
|--------------|----------------|--------------------------------------|
|Entrada y salida de datos|- Enter Data Manually (Introducir datos manualmente) </br> - Export Data (Exportar datos) </br> - Import Data (Importar datos) </br> - Load Trained Model (Cargar modelo entrenado) </br> - Unpack Zipped Datasets (Desempaquetar conjuntos de datos comprimidos)|- Enter Data Manually (Introducir datos manualmente) </br> - Export Data (Exportar datos) </br> - Import Data (Importar datos)|
|Conversiones de formato de datos|- Convert to CSV (Convertir a CSV) </br> - Convert to Dataset (Convertir a conjunto de datos) </br> - Convert to ARFF (Convertir a ARFF) </br> - Convert to SVMLight (Convertir a SVMLight) </br> - Convert to TSV (Convertir a TSV)|- Convert to CSV (Convertir a CSV) </br> - Convert to Dataset (Convertir a conjunto de datos)|
|Data Transformation - Manipulation (Transformación de datos - Manipulación)|- Add columns (Agregar columnas)</br> - Add Rows (Agregar filas) </br> - Apply SQL Transformation (Aplicar transformación de SQL) </br> - Cleaning Missing Data (Limpiar datos que faltan) </br> - Convert to Indicator Values (Convertir en valores de indicador) </br> - Edit Metadata (Editar metadatos) </br> - Join Data (Combinar datos) </br> - Remove Duplicate Rows (Quitar filas duplicadas) </br> - Select Columns in Dataset (Seleccionar columnas del conjunto de datos) </br> - Select Columns Transform (Seleccionar transformación de columnas) </br> - SMOTE </br> - Group Categorical Values (Agrupar valores de categorías)|- Add columns (Agregar columnas)</br> - Add Rows (Agregar filas) </br> - Apply SQL Transformation (Aplicar transformación de SQL) </br> - Cleaning Missing Data (Limpiar datos que faltan) </br> - Convert to Indicator Values (Convertir en valores de indicador) </br> - Edit Metadata (Editar metadatos) </br> - Join Data (Combinar datos) </br> - Remove Duplicate Rows (Quitar filas duplicadas) </br> - Select Columns in Dataset (Seleccionar columnas del conjunto de datos) </br> - Select Columns Transform (Seleccionar transformación de columnas) </br> - SMOTE|
|Data Transformation – Scale and Reduce (Transformación de datos - Escalar y reducir) |- Clip Values (Recorte de valores) </br> - Group Data into Bins (Agrupar datos en rangos) </br> - Normalize Data (Normalizar datos) </br>- Principal Component Analysis (Análisis de componentes principales) |- Clip Values (Recorte de valores) </br> - Group Data into Bins (Agrupar datos en rangos) </br> - Normalize Data (Normalizar datos)|
|Data Transformation – Sample and Split (Transformación de datos - Muestrear y dividir)|- Partition and Sample (Partición y muestra) </br> - Split Data (Dividir datos)|- Partition and Sample (Partición y muestra) </br> - Split Data (Dividir datos)|
|Data Transformation – Filter (Transformación de datos - Filtro) |- Apply Filter (Aplicar filtro) </br> - FIR Filter (Filtro FIR) </br> - IIR Filter (Filtro IIR) </br> - Median Filter (Filtro de mediana) </br> - Moving Average Filter (Filtro de media móvil) </br> - Threshold Filter (Filtro de umbral) </br> - User Defined Filter (Filtro definido por el usuario)||
|Data Transformation – Learning with Counts (Transformación de datos - Aprendizaje con recuentos) |- Build Counting Transform (Crear transformación de recuento) </br> - Export Count Table (Exportar tabla de recuentos) </br> - Import Count Table (Importar tabla de recuentos) </br> - Merge Count Transform (Combinar transformación de recuento)</br>  - Modify Count Table Parameters (Modificar parámetros de tabla de recuento)||
|Selección de características |- Filter Based Feature Selection (Selección de características basada en filtro) </br> - Fisher Linear Discriminant Analysis (Análisis discriminante lineal de Fisher)  </br> - Permutation Feature Importance (Importancia de la característica de permutación) |- Filter Based Feature Selection (Selección de características basada en filtro) </br>  - Permutation Feature Importance (Importancia de la característica de permutación)|
| Model - Classification (Modelo - Clasificación)| - Multiclass Decision Forest (Bosque de decisión multiclase) </br> - Multiclass Decision Jungle (Selva de decisión multiclase)  </br> - Multiclass Logistic Regression (Regresión logística multiclase)  </br>- Multiclass Neural Network (Red neuronal multiclase)  </br>- One-vs-All Multiclass (Uno frente a todos multiclase) </br>- Two-Class Averaged Perceptron (Perceptrón promedio de dos clases) </br>- Two-Class Bayes Point Machine (Máquina del punto de Bayes de dos clases) </br>- Two-Class Boosted Decision Tree (Árbol de decisión ampliado de dos clases)  </br> - Two-Class Decision Forest (Bosque de decisión de dos clases)  </br> - Two-Class Decision Jungle (Selva de decisión de dos clases)  </br> - Two-Class Locally-Deep SVM (SVM localmente profunda de dos clases) </br> - Two-Class Logistic Regression (Regresión logística de dos clases)  </br> - Two-Class Neural Network (Red neuronal de dos clases) </br> - Two-Class Support Vector Machine (Máquina de vectores de soporte de dos clases)  | - Multiclass Decision Forest (Bosque de decisión multiclase) </br>  - Multiclass Boost Decision Tree (Árbol de decisión ampliado multiclase)  </br> - Multiclass Logistic Regression (Regresión logística multiclase) </br> - Multiclass Neural Network (Red neuronal multiclase) </br> - One-vs-All Multiclass (Uno frente a todos multiclase)  </br> - Two-Class Averaged Perceptron (Perceptrón promedio de dos clases)  </br> - Two-Class Boosted Decision Tree (Árbol de decisión ampliado de dos clases)  </br> - Two-Class Decision Forest (Bosque de decisión de dos clases) </br>-  Two-Class Logistic Regression (Regresión logística de dos clases) </br> - Two-Class Neural Network (Red neuronal de dos clases) </br>-   Two-Class Support Vector Machine (Máquina de vectores compatible con dos clases)  |
| Model - Clustering (Modelo - Agrupación en clústeres)| - K-means clustering (Agrupación en clústeres K-means)| - K-means clustering (Agrupación en clústeres K-means)|
| Model - Regression (Modelo - Regresión)| - Bayesian Linear Regression (Regresión lineal bayesiana)  </br> - Boosted Decision Tree Regression (Regresión de árbol de decisión incrementado)  </br>- Decision Forest Regression (Regresión de bosque de decisión)  </br> - Fast Forest Quantile Regression (Regresión rápida de bosque por cuantiles)  </br> - Linear Regression (Regresión lineal) </br> - Neural Network Regression (Regresión de red neuronal) </br> - Ordinal Regression  Poisson Regression (Regresión ordinal, Regresión de Poisson)| - Boosted Decision Tree Regression (Regresión de árbol de decisión incrementado)  </br>- Decision Forest Regression (Regresión de bosque de decisión)  </br> - Fast Forest Quantile Regression (Regresión rápida de bosque por cuantiles) </br> - Linear Regression (Regresión lineal)  </br> - Neural Network Regression (Regresión de red neuronal) </br> - Poisson Regression (Regresión de Poisson)|
| Model – Anomaly Detection (Modelo - Detección de anomalías)| - One-Class SVM (SVM de una clase)  </br> - PCA-Based Anomaly Detection (Detección de anomalías basada en PCA) | - PCA-Based Anomaly Detection (Detección de anomalías basada en PCA)|
| Machine Learning – Evaluate (Aprendizaje automático - Evaluar)  | - Cross Validate Model (Modelo de validación cruzada)  </br>- Evaluate Model (Evaluar modelo)  </br>- Evaluate Recommender (Evaluar recomendador) | - Cross Validate Model (Modelo de validación cruzada)  </br>- Evaluate Model (Evaluar modelo) </br> - Evaluate Recommender (Evaluar recomendador)|
| Machine Learning – Train (Aprendizaje automático - Entrenar)| - Sweep Clustering (Agrupación en clústeres de barrido)  </br> - Train Anomaly Detection Model (Entrenar modelo de detección de anomalías) </br>- Train Clustering Model (Entrenar modelo de agrupación en clústeres)  </br> - Train Matchbox Recommender (Entrenar un recomendador de Matchbox)</br> Train Model (Entrenar modelo)  </br>- Tune Model Hyperparameters (Optimizar los hiperparámetros del modelo)| - Train Anomaly Detection Model (Entrenar modelo de detección de anomalías)  </br> - Train Clustering Model (Entrenar modelo de agrupación en clústeres) </br> - Train Model (Entrenar modelo)</br> - Train PyTorch Model (Entrenar modelo PyTorch)  </br>- Train SVD Recommender (Entrenar recomendador de SVD)  </br>- Train Wide and Deep Recommender (Entrenamiento del recomendador ancho y profundo) </br>- Tune Model Hyperparameters (Optimizar los hiperparámetros del modelo)|
| Machine Learning – Score (Aprendizaje automático - Puntuar)| - Apply Transformation (Aplicar transformación)  </br>- Assign Data to Clusters (Asignar datos a clústeres)  </br>Score Matchbox Recommender (Puntuar recomendador de Matchbox) </br> - Score Model (Puntuar modelo)|-  Apply Transformation (Aplicar transformación)  </br> - Assign Data to Clusters (Asignar datos a clústeres) </br> - Score Image Model (Puntuar modelo de imagen)  </br> - Score Model (Puntuar modelo) </br>Score SVD Recommender (Puntuar recomendador de SVD) </br> -Score Wide and Deep Recommender (Puntuación del recomendador ancho y profundo)|
| Módulos de la biblioteca OpenCV| - Import Images (Importar imágenes) </br>- Pre-trained Cascade Image Classification (Clasificación de imágenes en cascada previamente entrenada) | |
| Módulos de lenguaje de Python| Execute Python Script (Ejecutar script de Python)| Execute Python Script (Ejecutar script de Python)  </br> - Create Python Model (Crear modelo de Python) |
| Módulos del lenguaje R  | - Execute R Script (Ejecutar script R)  </br> - Create R Model (Crear modelo R)| - Execute R Script (Ejecutar script R)|
| Funciones estadísticas | - Apply Math Operation (Aplicar operación matemática) </br>-  Compute Elementary Statistics (Cálculo de estadísticas elementales)  </br>- Compute Linear Correlation (Cálculo de la correlación lineal)  </br>- Evaluate Probability Function (Evaluación de la función de probabilidad)  </br>- Replace Discrete Values (Reemplazar valores discretos)  </br>- Summarize Data (Resumen de datos)  </br>- Test Hypothesis using t-Test (Probar la hipótesis mediante una prueba T)| - Apply Math Operation (Aplicar operación matemática)  </br>- Summarize Data (Resumen de datos)|
| Text Analytics| - Detect Languages (Detección de idiomas)  </br>- Extract Key Phrases from Text (Extraer frases clave del texto)  </br>- Extract N-Gram Features from Text (Extracción de características de n-gramas de texto)  </br>- Feature Hashing (Hash de características) </br>- Latent Dirichlet Allocation (Asignación de Dirichlet latente)  </br>- Named Entity Recognition (Reconocimiento de entidades con nombre) </br>- Preprocess Text (Preprocesamiento de texto)  </br>Score Vowpal Wabbit Version 7-10 Model (Puntuar un modelo de Vowpal Wabbit versión 7-10)  </br>Score Vowpal Wabbit Version 8 Model (Puntuar un modelo de Vowpal Wabbit versión 8) </br>Train Vowpal Wabbit Version 7-10 Model (Entrenar un modelo de Vowpal Wabbit versión 7-10)  </br>Train Vowpal Wabbit Version 8 Model (Entrenar un modelo de Vowpal Wabbit versión 8) |-  Convert Word to Vector (Conversión de palabra en vector) </br> - Extract N-Gram Features from Text (Extracción de características de n-gramas de texto) </br>-  Feature Hashing (Hash de características)  </br>- Latent Dirichlet Allocation (Asignación de Dirichlet latente) </br>- Preprocess Text (Preprocesamiento de texto)  </br>- Score Vowpal Wabbit Model (Puntuación del modelo de Vowpal Wabbit) </br> - Train Vowpal Wabbit Model (Entrenamiento del modelo de Vowpal Wabbit)|
| Serie temporal| - Time Series Anomaly Detection (Detección de anomalías en una serie temporal) | |
| Servicio web | - Input (Entrada) </br> -   Output (Salida) | - Input (Entrada) </br>  - Salida|
| Computer Vision| | - Apply Image Transformation (Aplicación de transformación de imagen) </br> - Convert to Image Directory (Conversión al directorio de imagen) </br> - Init Image Transformation (Transformación de imagen init) </br> - Split Image Directory (División de directorio de imagen)  </br> - DenseNet Image Classification (Clasificación de imágenes DenseNet)   </br>- ResNet Image Classification (Clasificación de imágenes ResNet) |

Para obtener más información sobre cómo usar los módulos individuales del diseñador, consulte la [referencia de módulos del diseñador](../algorithm-module-reference/module-reference.md).

### <a name="what-if-a-designer-module-is-missing"></a>¿Qué ocurre si falta un módulo de diseñador?

El diseñador de Azure Machine Learning contiene los módulos más populares de Studio (clásico). También incluye nuevos módulos que aprovechan las técnicas de aprendizaje automático más recientes. 

Si la migración está bloqueada debido a que faltan módulos en el diseñador, póngase en contacto con nosotros mediante la [creación de una incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="example-migration"></a>Ejemplo de migración

En la siguiente migración de un experimento se destacan algunas de las diferencias entre Studio (clásico) y Azure Machine Learning.

### <a name="datasets"></a>Conjuntos de datos

En Studio (clásico), los **conjuntos de datos** se guardaban en el área de trabajo y solo se podían usar en Studio (clásico).

![automobile-price-classic-dataset](./media/migrate-overview/studio-classic-dataset.png)

En Azure Machine Learning, los **conjuntos de datos** se registran en el área de trabajo y se pueden usar en todo Azure Machine Learning. Para obtener más información sobre las ventajas de los conjuntos de datos de Azure Machine Learning, consulte [Acceso seguro a datos en Azure Machine Learning](../concept-data.md#reference-data-in-storage-with-datasets).

![automobile-price-aml-dataset](./media/migrate-overview/aml-dataset.png)

### <a name="pipeline"></a>Canalización

En Studio (clásico), los **experimentos** contenían la lógica de procesamiento del trabajo. Creaba experimentos con módulos de arrastrar y colocar.

![automobile-price-classic-experiment](./media/migrate-overview/studio-classic-experiment.png)

En Azure Machine Learning, las **canalizaciones** contienen la lógica de procesamiento del trabajo. Puede crear canalizaciones con módulos de arrastrar y colocar, o bien escribiendo código.

![automobile-price-aml-pipeline](./media/migrate-overview/aml-pipeline.png)

### <a name="web-service-endpoint"></a>Punto de conexión de servicio web

Studio (clásico) usaba **REQUEST/RESPOND API** para la predicción en tiempo real y **BATCH EXECUTION API** para la predicción por lotes o el reentrenamiento.

![automobile-price-classic-webservice](./media/migrate-overview/studio-classic-web-service.png)

Azure Machine Learning usa **puntos de conexión en tiempo real** para la predicción en tiempo real y **puntos de conexión de canalización** para la predicción por lotes o el reentrenamiento.

![automobile-price-aml-endpoint](./media/migrate-overview/aml-endpoint.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha conocido los requisitos generales para migrar a Azure Machine Learning. Para obtener los pasos detallados, consulte los demás artículos de la serie de migración de Studio (clásico):

1. **Información general sobre la migración**.
1. [Migración de un conjunto de datos de Studio a Azure Machine Learning](migrate-register-dataset.md).
1. [Recompilación de una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).
1. [Recompilación de un servicio web de Studio (clásico)](migrate-rebuild-web-service.md).
1. [Integración de un servicio web de Azure Machine Learning con aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).
1. [Migración de los módulos Execute R Script (Ejecutar script R) de Studio (Clásico)](migrate-execute-r-script.md).