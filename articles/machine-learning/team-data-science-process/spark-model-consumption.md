---
title: 'Puesta en funcionamiento de modelos de Machine Learning compilados con Spark: proceso de ciencia de datos en equipos'
description: Cómo cargar y puntuar modelos de aprendizaje almacenados en Azure Blob Storage (WASB) con Python.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 4a8d46b5ff10fdaa62f7747aac654e0b1ca78c42
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259156"
---
# <a name="operationalize-spark-built-machine-learning-models"></a>Operacionalización de modelos de aprendizaje automático creados con Spark

En este tema se muestra cómo operacionalizar un modelo de aprendizaje automático con Python en clústeres de HDInsight Spark. Se describe cómo cargar modelos de aprendizaje automático creados con Spark MLlib y almacenados en Azure Blob Storage (WASB), además de cómo puntuarlos con conjuntos de datos que también se han guardado en WASB. Muestra cómo procesar previamente los datos de entrada, transformar las características mediante las funciones de indexación y codificación en el kit de herramientas MLlib, y cómo crear un objeto de datos de punto con etiqueta que se puede usar como entrada para la puntuación con los modelos de aprendizaje automático. Los modelos usados para la puntuación son regresión lineal, regresión logística, modelos de bosque aleatorio y modelos de árboles impulsados por gradiente.

## <a name="spark-clusters-and-jupyter-notebooks"></a>Clústeres de Spark y cuadernos de Jupyter
En este tutorial se proporcionan los pasos de configuración y el código para operacionalizar un modelo de ML para usar un clúster de HDInsight Spark 1.6 y un clúster de Spark 2.0. También se proporciona el código para estos procedimientos en los cuadernos de Jupyter.

### <a name="notebook-for-spark-16"></a>Notebook para Spark 1.6
En el cuaderno de Jupyter [pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) se muestra cómo operacionalizar un modelo guardado con Python en clústeres de HDInsight. 

### <a name="notebook-for-spark-20"></a>Notebook para Spark 2.0
Para modificar este cuaderno de Jupyter para Spark 1.6 para usarlo con un clúster de HDInsight Spark 2.0, reemplace el archivo de código Python por [este archivo](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py). Este código muestra cómo utilizar los modelos creados en Spark 2.0.


## <a name="prerequisites"></a>Requisitos previos

1. Necesita una cuenta de Azure y un clúster de Spark 1.6 o Spark 2.0 HDInsight para completar este tutorial. Consulte el artículo [Información general sobre la ciencia de los datos con Spark en HDInsight de Azure](spark-overview.md) para obtener instrucciones sobre cómo satisfacer estos requisitos. Ese tema también contiene una descripción de los datos de taxis de Nueva York de 2013 que se usan aquí, además de instrucciones sobre cómo ejecutar el código de un cuaderno de Jupyter Notebook en el clúster Spark. 
2. Cree los modelos de aprendizaje automático que se van a puntuar aquí. para hacerlo, consulte el tema [Exploración y modelado de datos con Spark](spark-data-exploration-modeling.md) relativo al clúster de Spark 1.6 o los cuadernos de Spark 2.0. 
3. Los cuadernos de Spark 2.0 usan un conjunto de datos adicional para la tarea de clasificación, el conocido conjunto de datos Airline On-time departure de 2011 y 2012. Se proporciona una descripción de los cuadernos y de los vínculos a estos en el archivo [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) del repositorio de GitHub que los contiene. No obstante, este código y los cuadernos vinculados son genéricos y deberían funcionar en cualquier clúster de Spark. Los pasos de configuración y administración del clúster pueden ser ligeramente diferentes de los que se muestran aquí si no está usando Spark en HDInsight. 

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>Configuración: Ubicaciones de almacenamiento, bibliotecas y el contexto de Spark preestablecido
Spark puede leer y escribir en un blob de Azure Storage (WASB), por lo que cualquiera de los datos existentes almacenados allí pueden procesarse mediante Spark y volver a almacenarse en WASB.

Para guardar modelos o archivos en WASB, la ruta de acceso debe especificarse correctamente. Se puede hacer referencia al contenedor predeterminado asociado al clúster de Spark con una ruta de acceso que comience por *"wasb///"* . El ejemplo de código siguiente especifica la ubicación de los datos que se van a leer y la ruta de acceso del directorio de almacenamiento del modelo donde se guardará la salida del modelo. 

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>Establecimiento de rutas de directorio para las ubicaciones de almacenamiento de WASB
Los modelos se guardan en: "wasb:///user/remoteuser/NYCTaxi/Models". Si esta ruta de acceso no está configurada correctamente, no se cargará los modelos para su puntuación.

Los resultados puntuados se guardan en: "wasb:///user/remoteuser/NYCTaxi/ScoredResults". Si la ruta de acceso a la carpeta es incorrecta, los resultados no se guardarán en esa carpeta.   

> [!NOTE]
> La ruta de acceso a las ubicaciones de los archivos se puede copiar y pegar en los marcadores de posición en este código, en la salida de la última celda del Notebook **machine-learning-data-science-spark-data-exploration-modeling.ipynb** .   
> 
> 

Este es el código para establecer rutas de acceso de directorio: 

```python
# LOCATION OF DATA TO BE SCORED (TEST DATA)
taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";

# SET THE MODEL STORAGE DIRECTORY PATH 
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 

# SET SCORDED RESULT DIRECTORY PATH
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 

# FILE LOCATIONS FOR THE MODELS TO BE SCORED
logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

# RECORD START TIME
import datetime
datetime.datetime.now()
```

**SALIDA:**

datetime.datetime(2016, 4, 25, 23, 56, 19, 229403)

### <a name="import-libraries"></a>Importación de bibliotecas
Establezca el contexto de Spark e importe las bibliotecas necesarias con el siguiente código:

```python
#IMPORT LIBRARIES
import pyspark
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql import SQLContext
import matplotlib
import matplotlib.pyplot as plt
from pyspark.sql import Row
from pyspark.sql.functions import UserDefinedFunction
from pyspark.sql.types import *
import atexit
from numpy import array
import numpy as np
import datetime
```

### <a name="preset-spark-context-and-pyspark-magics"></a>Contexto de Spark preestablecido e instrucciones mágicas de PySpark
Los kernels de PySpark que se proporcionan con cuadernos de Jupyter Notebook tienen contextos preestablecidos. Por consiguiente, no es necesario establecer explícitamente los contextos de Spark o Hive antes de empezar a trabajar con la aplicación que se esté desarrollando. Estos contextos están disponibles de forma predeterminada:

* sc: Para Spark 
* sqlContext: Para Hive

El kernel PySpark proporciona algunas “instrucciones mágicas” predefinidas, que son comandos especiales que se pueden llamar con %%. Hay dos comandos de este tipo que se utilizan en estos ejemplos de código.

* **%%local** Especifica que el código de las líneas siguientes se ejecutará localmente. El código debe ser un código de Python válido.
* **%%sql -o \<variable name>** 
* Ejecuta una consulta de Hive en el sqlContext. Si se pasa el parámetro -o, el resultado de la consulta se conserva en el contexto %%local de Python como trama de datos de Pandas.

Para obtener más información sobre los kernels de cuadernos de Jupyter Notebook y las instrucciones mágicas predefinidas llamadas con %% (por ejemplo, %%local) que proporcionan, consulte [Kernels disponibles para cuadernos de Jupyter con clústeres de Spark en HDInsight basados en Linux en HDInsight (versión preliminar)](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="ingest-data-and-create-a-cleaned-data-frame"></a>Inserción de datos y creación de una trama de datos limpia
Esta sección contiene el código de una serie de tareas necesarias para incorporar los datos que se van a puntuar. Lee una muestra del 0,1 % combinado del archivo de tarifas y carreras de taxi (almacenado como un archivo .tsv), da formato a los datos y, después, crea un marco de datos limpio.

Los archivos de tarifas y carreras de taxi se combinaron según el procedimiento indicado en el: tema [Proceso de ciencia de datos en equipos en acción: uso de clústeres de Hadoop de HDInsight](hive-walkthrough.md).

```python
# INGEST DATA AND CREATE A CLEANED DATA FRAME

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT FILE FROM PUBLIC BLOB
taxi_test_file = sc.textFile(taxi_test_file_loc)

# GET SCHEMA OF THE FILE FROM HEADER
taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)

# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))

# GET SCHEMA OF THE FILE FROM HEADER
schema_string = taxi_test_file.first()
fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
fields[7].dataType = IntegerType() #Pickup hour
fields[8].dataType = IntegerType() # Pickup week
fields[9].dataType = IntegerType() # Weekday
fields[10].dataType = IntegerType() # Passenger count
fields[11].dataType = FloatType() # Trip time in secs
fields[12].dataType = FloatType() # Trip distance
fields[19].dataType = FloatType() # Fare amount
fields[20].dataType = FloatType() # Surcharge
fields[21].dataType = FloatType() # Mta_tax
fields[22].dataType = FloatType() # Tip amount
fields[23].dataType = FloatType() # Tolls amount
fields[24].dataType = FloatType() # Total amount
fields[25].dataType = IntegerType() # Tipped or not
fields[26].dataType = IntegerType() # Tip class
taxi_schema = StructType(fields)

# CREATE DATA FRAME
taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)

# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
    .drop('direct_distance').drop('surcharge')\
    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_cleaned.cache()
taxi_df_test_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_test_cleaned.registerTempTable("taxi_test")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 46,37 segundos

## <a name="prepare-data-for-scoring-in-spark"></a>Preparación de los datos para la puntuación en Spark
En esta sección se explica cómo indexar, codificar y ajustar la escala de características categóricas para prepararlas para su uso en algoritmos de aprendizaje supervisado de MLlib para clasificación y regresión.

### <a name="feature-transformation-index-and-encode-categorical-features-for-input-into-models-for-scoring"></a>Transformación de características: indexación y codificación de características categóricas para su incorporación en modelos para puntuación
En esta sección se muestra cómo indexar datos de categorías mediante `StringIndexer` y cómo codificar características con la entrada de `OneHotEncoder` en los modelos.

[StringIndexer](https://spark.apache.org/docs/latest/ml-features.html#stringindexer) codifica una columna de cadena de etiquetas en una columna de índices de etiqueta. Los índices se ordenan por frecuencias de etiqueta. 

[OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) asigna una columna de índices de etiqueta a una columna de vectores binarios, con un solo valor como máximo. Esta codificación permite aplicar algoritmos que esperan características con valores continuos, como la regresión logística, a características categóricas.

```python
#INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_test 
"""
taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_with_newFeatures.cache()
taxi_df_test_with_newFeatures.count()

# INDEX AND ONE-HOT ENCODING
stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
indexed = model.transform(taxi_df_test_with_newFeatures)
encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
model = stringIndexer.fit(encoded1)
indexed = model.transform(encoded1)
encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
model = stringIndexer.fit(encoded2)
indexed = model.transform(encoded2)
encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
encoded3 = encoder.transform(indexed)

# INDEX AND ENCODE TRAFFIC TIME BINS
stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
model = stringIndexer.fit(encoded3)
indexed = model.transform(encoded3)
encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
encodedFinal = encoder.transform(indexed)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 5,37 segundos

### <a name="create-rdd-objects-with-feature-arrays-for-input-into-models"></a>Creación de objetos RDD con matrices de características para su entrada en los modelos
Esta sección contiene código que muestra cómo indexar datos de texto categóricos como un objeto RDD y cómo codificarlos como “uno de n” para poder usarlos para entrenar y probar la regresión logística de MLlib y otros modelos basados en árboles. Los datos indexados se almacenan en objetos de [conjunto de datos distribuido resistente (RDD)](https://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html) . Los RDD son la abstracción básica en Spark. Un objeto RDD representa una colección inmutable con particiones de los elementos con los que se puede trabajar en paralelo con Spark.

También contiene código que muestra cómo escalar datos con el elemento `StandardScalar` que proporciona MLlib para usarse en la regresión lineal con descenso de gradiente estocástico (SGD), un popular algoritmo para entrenar una amplia variedad de modelos de aprendizaje automático. [StandardScaler](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.tree.RandomForest.html#pyspark.mllib.tree.RandomForest
) se usa para escalar las características a la varianza de la unidad. El ajuste de la escala de las características, también conocido como normalización de los datos, garantiza que características con valores situados muy en los extremos no tengan un peso excesivo en la función objetivo. 

```python
# CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import StandardScaler, StandardScalerModel
from pyspark.mllib.util import MLUtils
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingRegression(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
def parseRowOneHotRegression(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)

# FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)

# SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
oneHotTESTregScaled = scaler.transform(oneHotTESTreg)

# CACHE RDDS IN MEMORY
indexedTESTbinary.cache();
oneHotTESTbinary.cache();
indexedTESTreg.cache();
oneHotTESTreg.cache();
oneHotTESTregScaled.cache();

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 11,72 segundos

## <a name="score-with-the-logistic-regression-model-and-save-output-to-blob"></a>Puntuación con el modelo de regresión logística y guardado de la salida en un blob
El código de esta sección muestra cómo cargar un modelo de regresión logística guardado en Azure Blob Storage, y cómo usarlo para predecir si se dio o no una propina en una carrera de taxi, puntuarlo con métricas de clasificación estándar y, después, guardar y trazar los resultados en un almacenamiento de blobs. Los resultados puntuados se almacenan en objetos RDD. 

```python
# SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.classification import LogisticRegressionModel

## LOAD SAVED MODEL
savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))

## SAVE SCORED RESULTS (RDD) TO BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
dirfilename = scoredResultDir + logisticregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 19,22 segundos

## <a name="score-a-linear-regression-model"></a>Puntuación de un modelo de regresión lineal
Usamos [LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.regression.LinearRegressionWithSGD.html
) para entrenar un modelo de regresión lineal que usa descenso de gradiente estocástico (SGD) para predecir los importes de las propinas de forma óptima. 

El código de esta sección muestra cómo cargar un modelo de regresión lineal desde Almacenamiento de blobs de Azure, puntuarlo mediante variables con ajuste de la escala y, después, guardar los resultados en el blob.

```python
#SCORE LINEAR REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

#LOAD LIBRARIES
from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel

# LOAD MODEL AND SCORE USING ** SCALED VARIABLES **
savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = scoredResultDir + linearregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 16,63 segundos

## <a name="score-classification-and-regression-random-forest-models"></a>Puntuación de los modelos de bosque aleatorio de clasificación y regresión
El código de esta sección muestra cómo cargar los modelos de bosque aleatorio de clasificación y regresión guardados en Almacenamiento de blobs de Azure, puntuar su rendimiento con medidas estándar de clasificación y regresión y, después, guardar los resultados en Almacenamiento de blobs.

[bosques aleatorios](https://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) son conjuntos de árboles de decisión.  Combinan varios árboles de decisión para reducir el riesgo de sobreajuste. Los bosques aleatorios pueden controlar características categóricas, amplían la configuración de clasificación multiclase, no requieren ajustar la escala de las características y pueden capturar errores de alineación e interacciones de las características. Los bosques aleatorios son uno de los modelos de aprendizaje automático de más éxito para clasificación y regresión.

[spark.mllib](https://spark.apache.org/mllib/) admite bosques aleatorios para realizar operaciones de clasificación binaria y multiclase, y de regresión; las dos emplean características de categorías y continuas. 

```python
# SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES    
from pyspark.mllib.tree import RandomForest, RandomForestModel


# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 31,07 segundos

## <a name="score-classification-and-regression-gradient-boosting-tree-models"></a>Puntuación de modelos de árboles impulsados por gradiente de clasificación y regresión
El código de esta sección muestra cómo cargar los modelos de árboles impulsados por gradiente de clasificación y regresión guardados en Almacenamiento de blobs de Azure, puntuar su rendimiento con medidas estándar de clasificación y regresión y, después, guardar los resultados en Almacenamiento de blobs. 

**spark.mllib** admite GBT para realizar operaciones de clasificación binaria y de regresión; las dos emplean características de categorías y continuas. 

Los [árboles de potenciación del gradiente](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) son conjuntos de árboles de decisión. Los GBT entrenan árboles de decisión de forma iterativa para minimizar una función de pérdida. Los GBT pueden controlar características categóricas, no requieren el escalado de características y pueden capturar la falta de linearidad e interacciones de las características. Este algoritmo también se pueden usar en una configuración de clasificación multiclase.

```python
# SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

#LOAD AND SCORE THE MODEL
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + btclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

# LOAD AND SCORE MODEL 
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + btregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 14,6 segundos

## <a name="clean-up-objects-from-memory-and-print-scored-file-locations"></a>Limpieza de objetos de la memoria e impresión de las ubicaciones de los archivos puntuados

```python
# UNPERSIST OBJECTS CACHED IN MEMORY
taxi_df_test_cleaned.unpersist()
indexedTESTbinary.unpersist();
oneHotTESTbinary.unpersist();
indexedTESTreg.unpersist();
oneHotTESTreg.unpersist();
oneHotTESTregScaled.unpersist();


# PRINT OUT PATH TO SCORED OUTPUT FILES
print "logisticRegFileLoc: " + logisticregressionfilename;
print "linearRegFileLoc: " + linearregressionfilename;
print "randomForestClassificationFileLoc: " + rfclassificationfilename;
print "randomForestRegFileLoc: " + rfregressionfilename;
print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
print "BoostedTreeRegressionFileLoc: " + btregressionfilename;
```

**SALIDA:**

logisticRegFileLoc: LogisticRegressionWithLBFGS_2016-05-0317_22_38.953814.txt

linearRegFileLoc: LinearRegressionWithSGD_2016-05-0317_22_58.878949

randomForestClassificationFileLoc: RandomForestClassification_2016-05-0317_23_15.939247.txt

randomForestRegFileLoc: RandomForestRegression_2016-05-0317_23_31.459140.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification_2016-05-0317_23_49.648334.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression_2016-05-0317_23_56.860740.txt

## <a name="consume-spark-models-through-a-web-interface"></a>Uso de modelos de Spark mediante una interfaz web
Spark proporciona un mecanismo para el envío remoto de trabajos por lotes o consultas interactivas mediante una interfaz REST con un componente llamado Livy. Livy está habilitado de forma predeterminada en el clúster de Spark en HDInsight. Para más información sobre Livy, consulte: [Envío remoto de trabajos de Spark mediante Livy](../../hdinsight/spark/apache-spark-livy-rest-interface.md). 

Puede usar Livy para enviar de forma remota un trabajo que puntúa por lotes un archivo almacenado en un blob de Azure y, después, escribe los resultados en otro blob. Para ello, cargue el script de Python desde  
[GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) en el blob del clúster de Spark. Puede usar una herramienta como el **Explorador de Microsoft Azure Storage** o **AzCopy** para copiar el script en el blob del clúster. En este caso, se carga el script en **_wasb:///example/python/ConsumeGBNYCReg.py_**.   

> [!NOTE]
> Las claves de acceso que necesita se encuentran en el portal de la cuenta de almacenamiento asociada con el clúster de Spark. 
> 
> 

Una vez cargado en esta ubicación, el script se ejecutará dentro del clúster de Spark en un contexto distribuido. Carga el modelo y ejecuta predicciones en los archivos de entrada basándose en el modelo.  

Para invocar este script de forma remota con una sencilla solicitud HTTPS/REST en Livy.  Este es un comando de CURL para construir la solicitud HTTP para invocar el script de Python de forma remota. Reemplace CLUSTERLOGIN, CLUSTERPASSWORD, CLUSTERNAME por los valores adecuados para su clúster de Spark.

```console
# CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{\"file\": \"wasb:///example/python/ConsumeGBNYCReg.py\"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches
```

Puede usar cualquier lenguaje en el sistema remoto para invocar el trabajo de Spark a través de Livy con una sencilla llamada HTTPS con autenticación básica.   

> [!NOTE]
> Sería conveniente usar la biblioteca de solicitudes de Python para realizar esta llamada HTTP, pero actualmente no está instalada de forma predeterminada en Azure Functions. Por ese motivo se usan bibliotecas HTTP más antiguas.   
> 
> 

Este es el código de Python para la llamada HTTP:

```python
#MAKE AN HTTPS CALL ON LIVY. 

import os

# OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILABLE BY DEFAULT
import httplib, urllib, base64

# REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
host = '<spark cluster name>.azurehdinsight.net:443'
username='<username>'
password='<password>'

#AUTHORIZATION
conn = httplib.HTTPSConnection(host)
auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}

# SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
# IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
response = conn.getresponse().read()
print(response)
conn.close()
```

También puede agregar este código de Python a [Azure Functions](https://azure.microsoft.com/documentation/services/functions/) para desencadenar el envío de un trabajo de Spark que puntúa un blob en función de diversos eventos, como un temporizador o la creación o actualización de un blob. 

Si prefiere usar un cliente sin código, use [Azure Logic Apps](https://azure.microsoft.com/documentation/services/app-service/logic/) para invocar la puntuación por lotes de Spark; para ello, defina una acción HTTP en el **Diseñador de Logic Apps** y establezca sus parámetros. 

* En Azure Portal, cree una nueva aplicación lógica seleccionando **+Nuevo** -> **Web y móvil** -> **Aplicación lógica**. 
* Escriba el nombre de la aplicación lógica y del plan de App Service para que aparezca el **Diseñador de aplicaciones lógicas**.
* Seleccione una acción HTTP y especifique los parámetros mostrados en la ilustración siguiente:

![Diseñador de Logic Apps](./media/spark-model-consumption/spark-logica-app-client.png)

## <a name="whats-next"></a>¿Qué sigue?
**Validación cruzada y barrido de hiperparámetros**: Consulte [Exploración y modelado avanzados de datos con Spark](spark-advanced-data-exploration-modeling.md) para saber cómo se pueden entrenar modelos con el barrido de hiperparámetros y la validación cruzada.

