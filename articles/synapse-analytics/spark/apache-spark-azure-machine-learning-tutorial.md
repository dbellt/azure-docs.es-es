---
title: 'Tutorial: Entrenamiento de un modelo en Python mediante el aprendizaje automático automatizado'
description: Tutorial para entrenar un modelo de aprendizaje automático en Python mediante Apache Spark y el aprendizaje automático automatizado.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: machine-learning
ms.date: 06/30/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.openlocfilehash: f85c693f038143543a3a90cc2774f3e6efea3c9b
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952267"
---
# <a name="tutorial-train-a-model-in-python-with-automated-machine-learning"></a>Tutorial: Entrenamiento de un modelo en Python mediante el aprendizaje automático automatizado

Azure Machine Learning es un entorno en la nube que permite entrenar, implementar, automatizar, administrar y realizar un seguimiento de los modelos de aprendizaje automático. 

En este tutorial, usará el [aprendizaje automático automatizado](../../machine-learning/concept-automated-ml.md) de Azure Machine Learning para crear un modelo de regresión y predecir las tarifas de taxi. Este proceso llega hasta el mejor modelo al aceptar los valores de configuración y los datos de entrenamiento, y recorrer automáticamente las combinaciones de distintos métodos, modelos y configuraciones de hiperparámetros.

En este tutorial, aprenderá a:
- Descargar los datos mediante Apache Spark y Azure Open Datasets.
- Transformar y limpiar los datos mediante DataFrames de Apache Spark.
- Entrenar un modelo de regresión de aprendizaje automático automatizado.
- Calcular la precisión del modelo.

## <a name="before-you-begin"></a>Antes de empezar

- Cree un grupo de Apache Spark sin servidor siguiendo las indicaciones de [Inicio rápido: Creación de un grupo de Apache Spark sin servidor mediante Synapse Studio](../quickstart-create-apache-spark-pool-studio.md).
- Complete el tutorial sobre el [configuración del área de trabajo de Azure Machine Learning](../../machine-learning/quickstart-create-resources.md) si no dispone de un área de trabajo de Azure Machine Learning. 

## <a name="understand-regression-models"></a>Descripción de los modelos de regresión

Los *modelos de regresión* predicen los valores de salida numéricos basados en indicadores independientes. En la regresión, el objetivo es ayudar a establecer la relación entre esas variables de predicción independientes al determinar el modo en que una variable afecta a las demás.  

### <a name="example-based-on-new-york-city-taxi-data"></a>Ejemplo basado en los datos de taxis de Nueva York

En este ejemplo, usará Spark para hacer un análisis de los datos relativos a las propinas de trayectos en taxi en Nueva York. Los datos están disponibles a través de [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Este subconjunto de datos contiene información sobre los trayectos de los taxis amarillos, como el recorrido, la hora y la ubicación de inicio y fin del trayecto, y el costo.

> [!IMPORTANT]
> Se pueden aplicar cargos adicionales por la extracción de estos datos de la ubicación de almacenamiento. En los pasos siguientes, desarrollará un modelo para predecir las tarifas de los taxis de Nueva York. 

##  <a name="download-and-prepare-the-data"></a>Descarga y preparación de los datos

A continuación, se indica cómo puede hacerlo.

1. Cree un cuaderno mediante el kernel de PySpark. Para obtener instrucciones, consulte [Creación de un cuaderno](../quickstart-apache-spark-notebook.md#create-a-notebook).
   
    > [!Note]
    > Debido a la existencia del kernel PySpark, no necesitará crear ningún contexto explícitamente. El contexto de Spark se crea automáticamente al ejecutar la primera celda de código.
  
2. Dado que los datos sin procesar están en formato Parquet, puede usar el contexto de Spark para arrastrar directamente el archivo a la memoria como un DataFrame. Cree un DataFrame de Spark. Para ello, recupere los datos mediante Open Datasets API. Aquí se usan las propiedades `schema on read` de DataFrame de Spark para deducir los tipos de datos y el esquema. 
   
    ```python
    blob_account_name = "azureopendatastorage"
    blob_container_name = "nyctlc"
    blob_relative_path = "yellow"
    blob_sas_token = r""

    # Allow Spark to read from the blob remotely
    wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
    spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),blob_sas_token)

    # Spark read parquet; note that it won't load any data yet
    df = spark.read.parquet(wasbs_path)

    ```

3. En función del tamaño del grupo de Spark, el tamaño los datos sin procesar podría ser muy grande o se podría tardar demasiado tiempo en acceder a los datos. Puede filtrar estos datos para obtener detalles más específicos, como un mes de datos, mediante el uso de los filtros ```start_date``` y ```end_date```. Una vez que haya filtrado un DataFrame, también ejecutará la función ```describe()``` en el nuevo DataFrame para ver las estadísticas de resumen de cada campo. 

   En las estadísticas de resumen se puede ver que los datos contienen algunas irregularidades. Por ejemplo, las estadísticas muestran que la distancia del recorrido mínimo es menor que 0. Es necesario filtrar estos puntos de datos irregulares.
   
   ```python
   # Create an ingestion filter
   start_date = '2015-01-01 00:00:00'
   end_date = '2015-12-31 00:00:00'

   filtered_df = df.filter('tpepPickupDateTime > "' + start_date + '" and tpepPickupDateTime< "' + end_date + '"')

   filtered_df.describe().show()
   ```

4. Genere las características del conjunto de datos. Para ello, seleccione un conjunto de columnas y cree diversas características basadas en tiempo en el campo `datetime` de recogida. Filtre los valores atípicos que se identificaron en el paso anterior y quite las últimas columnas porque no son necesarias para el entrenamiento.
   
   ```python
   from datetime import datetime
   from pyspark.sql.functions import *

   # To make development easier, faster, and less expensive, downsample for now
   sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)

   taxi_df = sampled_taxi_df.select('vendorID', 'passengerCount', 'tripDistance',  'startLon', 'startLat', 'endLon' \
                                   , 'endLat', 'paymentType', 'fareAmount', 'tipAmount'\
                                   , column('puMonth').alias('month_num') \
                                   , date_format('tpepPickupDateTime', 'hh').alias('hour_of_day')\
                                   , date_format('tpepPickupDateTime', 'EEEE').alias('day_of_week')\
                                   , dayofmonth(col('tpepPickupDateTime')).alias('day_of_month')
                                   ,(unix_timestamp(col('tpepDropoffDateTime')) - unix_timestamp(col('tpepPickupDateTime'))).alias('trip_time'))\
                           .filter((sampled_taxi_df.passengerCount > 0) & (sampled_taxi_df.passengerCount < 8)\
                                   & (sampled_taxi_df.tipAmount >= 0)\
                                   & (sampled_taxi_df.fareAmount >= 1) & (sampled_taxi_df.fareAmount <= 250)\
                                   & (sampled_taxi_df.tipAmount < sampled_taxi_df.fareAmount)\
                                   & (sampled_taxi_df.tripDistance > 0) & (sampled_taxi_df.tripDistance <= 200)\
                                   & (sampled_taxi_df.rateCodeId <= 5)\
                                   & (sampled_taxi_df.paymentType.isin({"1", "2"})))
   taxi_df.show(10)
   ```
   
   Como puede ver, se creará un DataFrame con columnas adicionales para el día del mes, la hora de recogida, el día de la semana y el tiempo total del recorrido. 

   ![Imagen del DataFrame de taxis.](./media/azure-machine-learning-spark-notebook/dataset.png#lightbox)

## <a name="generate-test-and-validation-datasets"></a>Generación de conjuntos de datos de prueba y validación

Una vez que disponga del conjunto de datos final, puede dividir los datos en conjuntos de entrenamiento y de prueba mediante la función ```random_ split ``` de Spark. Con ayuda de las ponderaciones proporcionadas, esta función separa aleatoriamente los datos en el conjunto de datos de entrenamiento para el entrenamiento del modelo y el conjunto de datos de validación para las pruebas.

```python
# Random split dataset using Spark; convert Spark to pandas
training_data, validation_data = taxi_df.randomSplit([0.8,0.2], 223)

```
Este paso garantiza que los puntos de datos para probar el modelo terminado no se hayan usado para entrenar el modelo. 

## <a name="connect-to-an-azure-machine-learning-workspace"></a>Conexión a un área de trabajo de Azure Machine Learning
En Azure Machine Learning, un área de trabajo es una clase que acepta una suscripción de Azure e información sobre los recursos. También crea un recurso en la nube para supervisar y realizar un seguimiento de las ejecuciones del modelo. En este paso, creará un objeto de área de trabajo a partir del área de trabajo existente de Azure Machine Learning.
   
```python
from azureml.core import Workspace

# Enter your workspace subscription, resource group, name, and region.
subscription_id = "<enter your subscription ID>" #you should be owner or contributor
resource_group = "<enter your resource group>" #you should be owner or contributor
workspace_name = "<enter your workspace name>" #your workspace name
workspace_region = "<enter workspace region>" #your region

ws = Workspace(workspace_name = workspace_name,
               subscription_id = subscription_id,
               resource_group = resource_group)

```

## <a name="convert-a-dataframe-to-an-azure-machine-learning-dataset"></a>Conversión de un DataFrame en un conjunto de datos de Azure Machine Learning
Para enviar un experimento remoto, convierta el conjunto de datos en una instancia ```TabularDatset``` de Azure Machine Learning. [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset) representa los datos en formato tabular al analizar los archivos proporcionados.

El código siguiente obtiene el área de trabajo existente y el almacén de datos predeterminado de Azure Machine Learning. A continuación, pasa las ubicaciones del almacén de datos y de los archivos al parámetro de la ruta de acceso para crear una nueva instancia de ```TabularDataset```. 

```python
import pandas 
from azureml.core import Dataset

# Get the Azure Machine Learning default datastore
datastore = ws.get_default_datastore()
training_pd = training_data.toPandas().to_csv('training_pd.csv', index=False)

# Convert into an Azure Machine Learning tabular dataset
datastore.upload_files(files = ['training_pd.csv'],
                       target_path = 'train-dataset/tabular/',
                       overwrite = True,
                       show_progress = True)
dataset_training = Dataset.Tabular.from_delimited_files(path = [(datastore, 'train-dataset/tabular/training_pd.csv')])
```
![Imagen del conjunto de datos cargado.](./media/azure-machine-learning-spark-notebook/upload-dataset.png)

## <a name="submit-an-automated-experiment"></a>Envío de un experimento automatizado

Las secciones siguientes le guiarán por el proceso para enviar un experimento de aprendizaje automático automatizado.

### <a name="define-training-settings"></a>Definición de la configuración del entrenamiento
1. Para enviar un experimento, es preciso definir sus parámetros y la configuración del modelo con fines de entrenamiento. Para obtener la lista completa de opciones de configuración, consulte [Configuración de experimentos de ML automatizado en Python](../../machine-learning/how-to-configure-auto-train.md).

   ```python
   import logging

   automl_settings = {
       "iteration_timeout_minutes": 10,
       "experiment_timeout_minutes": 30,
       "enable_early_stopping": True,
       "primary_metric": 'r2_score',
       "featurization": 'auto',
       "verbosity": logging.INFO,
       "n_cross_validations": 2}
   ```

1. Pase la configuración de entrenamiento definida como un parámetro `kwargs` a un objeto `AutoMLConfig`. Como está usando Spark, también debe pasar el contexto de Spark, al que se puede tener acceso automáticamente mediante la variable ```sc```. Además, debe especificar los datos de entrenamiento y el tipo de modelo, que en este caso es de regresión.

   ```python
   from azureml.train.automl import AutoMLConfig

   automl_config = AutoMLConfig(task='regression',
                                debug_log='automated_ml_errors.log',
                                training_data = dataset_training,
                                spark_context = sc,
                                model_explainability = False, 
                                label_column_name ="fareAmount",**automl_settings)
   ```

> [!NOTE]
>Los pasos previos al procesamiento del aprendizaje automático automatizado forman parte del modelo subyacente. Estos pasos incluyen la normalización de características, el control de los datos que faltan y la conversión de texto a valores numéricos. Cuando utilice el modelo con fines de predicción, se aplicarán automáticamente los mismos pasos previos al procesamiento que se aplican a los datos de entrada durante el entrenamiento.

### <a name="train-the-automatic-regression-model"></a>Entrenamiento del modelo de regresión automática 
A continuación, creará un objeto de experimento en el área de trabajo de Azure Machine Learning. Un experimento actúa como un contenedor para las ejecuciones individuales. 

```python
from azureml.core.experiment import Experiment

# Start an experiment in Azure Machine Learning
experiment = Experiment(ws, "aml-synapse-regression")
tags = {"Synapse": "regression"}
local_run = experiment.submit(automl_config, show_output=True, tags = tags)

# Use the get_details function to retrieve the detailed output for the run.
run_details = local_run.get_details()
```
Cuando haya finalizado el experimento, la salida devolverá detalles sobre las iteraciones completadas. Para cada iteración, verá el tipo de modelo, la duración de la ejecución y la precisión del entrenamiento. El campo `BEST` hace un seguimiento de la puntuación de entrenamiento que se ejecuta mejor en función del tipo de métrica.

![Captura de pantalla de la salida del modelo.](./media/azure-machine-learning-spark-notebook/model-output.png)

> [!NOTE]
> Después de enviar el experimento de aprendizaje automático automatizado, se ejecutan varias iteraciones y tipos de modelo. Esta ejecución normalmente tarda entre 60 y 90 minutos. 

### <a name="retrieve-the-best-model"></a>Recuperación del mejor modelo
Para seleccionar el mejor modelo de las iteraciones, use la función ```get_output```, ya que devuelve la mejor ejecución y el modelo idóneo. El código siguiente recuperará la mejor ejecución y el modelo idóneo para cualquier métrica registrada o para una iteración concreta.

```python
# Get best model
best_run, fitted_model = local_run.get_output()
```

### <a name="test-model-accuracy"></a>Probar la precisión del modelo
1. Para probar la precisión del modelo, use el mejor modelo para ejecutar predicciones de tarifas de los taxis en el conjunto de datos de prueba. La función ```predict``` usa el mejor modelo y predice los valores de `y` (importe de la tarifa) a partir del conjunto de datos de validación. 

   ```python
   # Test best model accuracy
   validation_data_pd = validation_data.toPandas()
   y_test = validation_data_pd.pop("fareAmount").to_frame()
   y_predict = fitted_model.predict(validation_data_pd)
   ```

1. El error cuadrático medio (RMSE) es una medida que se usa con frecuencia y señala las diferencias entre los valores de ejemplo previstos por un modelo y los valores observados. Para calcular el error cuadrático medio de los resultados, se compara el DataFrame `y_test` con los valores previstos por el modelo. 

   La función ```mean_squared_error``` toma dos matrices y calcula el promedio del error cuadrático entre ellos. Posteriormente, se obtiene la raíz cuadrada del resultado. Esta métrica indica aproximadamente cuánto se alejan las predicciones de las tarifas de taxi de los valores reales.

   ```python
   from sklearn.metrics import mean_squared_error
   from math import sqrt

   # Calculate root-mean-square error
   y_actual = y_test.values.flatten().tolist()
   rmse = sqrt(mean_squared_error(y_actual, y_predict))

   print("Root Mean Square Error:")
   print(rmse)
   ```

   ```Output
   Root Mean Square Error:
   2.309997102577151
   ```
   El error de la media cuadrática es una buena medida para conocer la precisión con la que el modelo predice la respuesta. A partir de los resultados, verá que el modelo es bastante bueno para predecir las tarifas de los taxis a partir de las características del conjunto de datos, normalmente en el margen de 2,00 USD.

1. Ejecute el siguiente código para calcular el error absoluto porcentual medio. Esta métrica expresa la precisión como un porcentaje del error. Para ello, calcula una diferencia absoluta entre cada valor predicho y real, y después suma todas las diferencias. A continuación, expresa esa suma en forma de porcentaje del total de los valores reales.

   ```python
   # Calculate mean-absolute-percent error and model accuracy 
   sum_actuals = sum_errors = 0

   for actual_val, predict_val in zip(y_actual, y_predict):
       abs_error = actual_val - predict_val
       if abs_error < 0:
           abs_error = abs_error * -1

       sum_errors = sum_errors + abs_error
       sum_actuals = sum_actuals + actual_val

   mean_abs_percent_error = sum_errors / sum_actuals

   print("Model MAPE:")
   print(mean_abs_percent_error)
   print()
   print("Model Accuracy:")
   print(1 - mean_abs_percent_error)
   ```

   ```Output
   Model MAPE:
   0.03655071038487368

   Model Accuracy:
   0.9634492896151263
   ```
   A partir de las dos métricas de precisión de la predicción, verá que el modelo es bastante bueno para predecir las tarifas de los taxis a partir de las características del conjunto de datos. 

1. Después de afinar el modelo de regresión lineal, es preciso determinar si el modelo se ajusta bien a los datos. Para ello, trace los valores de las tarifas reales frente la salida prevista. Además, también calculará la medida de R cuadrado para determinar lo cerca que los datos están de la línea de regresión ajustada.

   ```python
   import matplotlib.pyplot as plt
   import numpy as np
   from sklearn.metrics import mean_squared_error, r2_score

   # Calculate the R2 score by using the predicted and actual fare prices
   y_test_actual = y_test["fareAmount"]
   r2 = r2_score(y_test_actual, y_predict)

   # Plot the actual versus predicted fare amount values
   plt.style.use('ggplot')
   plt.figure(figsize=(10, 7))
   plt.scatter(y_test_actual,y_predict)
   plt.plot([np.min(y_test_actual), np.max(y_test_actual)], [np.min(y_test_actual), np.max(y_test_actual)], color='lightblue')
   plt.xlabel("Actual Fare Amount")
   plt.ylabel("Predicted Fare Amount")
   plt.title("Actual vs Predicted Fare Amount R^2={}".format(r2))
   plt.show()

   ```
   ![Captura de pantalla que muestra el trazado de la regresión.](./media/azure-machine-learning-spark-notebook/fare-amount.png)

   En los resultados, se puede ver que la medida de R cuadrado se atribuye el 95 % de la varianza. Esto también se constata por medio del trazado real frente al trazado observado. Cuanta mayor sea la varianza correspondiente al modelo de regresión, más cerca estarán los puntos de datos de la línea de regresión ajustada.  

## <a name="register-the-model-to-azure-machine-learning"></a>Registro del modelo en Azure Machine Learning
Una vez que haya constatado el mejor modelo, puede registrarlo en Azure Machine Learning. Posteriormente, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

```python
description = 'My automated ML model'
model_path='outputs/model.pkl'
model = best_run.register_model(model_name = 'NYCGreenTaxiModel', model_path = model_path, description = description)
print(model.name, model.version)
```
```Output
NYCGreenTaxiModel 1
```
## <a name="view-results-in-azure-machine-learning"></a>Visualización de los resultados en Azure Machine Learning
Para acceder a los resultados de las iteraciones, desplácese al experimento en el área de trabajo de Azure Machine Learning. Aquí puede obtener detalles adicionales sobre el estado de la ejecución, los modelos que se han probado y otras métricas del modelo. 

![Captura de pantalla que muestra un área de trabajo de Azure Machine Learning.](./media/azure-machine-learning-spark-notebook/azure-machine-learning-workspace.png)

## <a name="next-steps"></a>Pasos siguientes
- [Azure Synapse Analytics](../index.yml)
- [Tutorial: Compilación de una aplicación de aprendizaje automático con MLlib de Apache Spark y Azure Synapse Analytics](./apache-spark-machine-learning-mllib-notebook.md)