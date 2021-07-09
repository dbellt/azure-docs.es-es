---
title: Inicio rápido de la biblioteca cliente multivariante de Python de Anomaly Detector
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/29/2021
ms.author: mbullwin
ms.openlocfilehash: 0cbd7415f3b6f79a6c7231c7caa7ed947b9bdc24
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110164318"
---
Comience a usar la biblioteca cliente multivariante de Anomaly Detector para Python. Siga estos pasos para instalar el inicio del paquete con los algoritmos que proporciona el servicio. Las nuevas API de detección de anomalías multivariante permiten a los desarrolladores integrar fácilmente inteligencia artificial avanzada para detectar anomalías de grupos de métricas, sin necesidad de tener conocimientos de aprendizaje automático ni usar datos etiquetados. Las dependencias y correlaciones entre distintas señales se consideran automáticamente como factores clave. De este modo, podrá proteger de forma proactiva los sistemas complejos frente a errores.

Utilice la biblioteca cliente multivariante de Anomaly Detector para Python con el fin de:

* Detectar anomalías de nivel del sistema para un grupo de series temporales.
* Cuando una serie temporal individual no ofrece demasiada información y se deben examinar todas las señales para detectar un problema.
* Mantenimiento predictivo de recursos físicos costosos con decenas o cientos de tipos de sensores diferentes que miden diversos aspectos del estado del sistema.

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/anomalydetector/azure-ai-anomalydetector) | [Paquete (PyPi)](https://pypi.org/project/azure-ai-anomalydetector/3.0.0b3/) | [Código de ejemplo](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/anomalydetector/azure-ai-anomalydetector/samples/sample_multivariate_detect.py)

## <a name="prerequisites"></a>Requisitos previos

* [Python 3.x](https://www.python.org/)
* La [biblioteca de análisis de datos de Pandas](https://pandas.pydata.org/)
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector </a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.


## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Después de instalar Python, puede instalar las bibliotecas cliente con:

```console
pip install pandas
pip install --upgrade azure-ai-anomalydetector
```

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

 Cree un archivo de Python e importe las bibliotecas siguientes.

```python
import os
import time
from datetime import datetime

from azure.ai.anomalydetector import AnomalyDetectorClient
from azure.ai.anomalydetector.models import DetectionRequest, ModelInfo
from azure.core.credentials import AzureKeyCredential
from azure.core.exceptions import HttpResponseError
```

Cree variables para la clave como una variable de entorno, la ruta de acceso a un archivo de datos de serie temporal y la ubicación de Azure de su suscripción. 

> [!NOTE]
> Siempre tendrá la opción de usar una de estas dos claves. Esto es para permitir la rotación de claves segura. Para este inicio rápido, use la primera clave. 

```python
subscription_key = "ANOMALY_DETECTOR_KEY"
anomaly_detector_endpoint = "ANOMALY_DETECTOR_ENDPOINT"
```



## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de Anomaly Detector para Python:

* [Autenticar el cliente](#authenticate-the-client)
* [Entrenamiento del modelo](#train-the-model)
* [Detectar anomalías](#detect-anomalies)
* [Exportar un modelo](#export-model)
* [Eliminar un modelo](#delete-model)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Para crear una instancia de un nuevo cliente de Anomaly Detector, debe pasar la clave de suscripción de Anomaly Detector y el punto de conexión asociado. También estableceremos un origen de datos.  

Para usar las API multivariantes de Anomaly Detector, primero debe entrenar sus propios modelos. Los datos de entrenamiento son un conjunto de varias series temporales que cumplen los siguientes requisitos:

Cada serie temporal debe ser un archivo CSV con dos columnas (solo dos), "timestamp" y "value" (todo en minúsculas) como fila de encabezado. Los valores de "timestamp" deben cumplir la norma ISO 8601; los de "value" pueden ser números enteros o decimales con cualquier número de posiciones decimales. Por ejemplo:

|timestamp | value|
|-------|-------|
|2019-04-01T00:00:00Z| 5|
|2019-04-01T00:01:00Z| 3.6|
|2019-04-01T00:02:00Z| 4|
|`...`| `...` |

Cada archivo CSV debe tener el nombre de una variable diferente que se usará para el entrenamiento del modelo. Por ejemplo, "temperature.csv" y "humidity.csv". Todos los archivos CSV deben comprimirse en un archivo ZIP sin subcarpetas. El archivo ZIP puede tener el nombre que desee. El archivo ZIP debe cargarse en Azure Blob Storage. Una vez que genere la dirección URL de SAS de blob (firmas de acceso compartido) para el archivo ZIP, se puede usar para el entrenamiento. Consulte este documento para obtener información sobre cómo generar direcciones URL de SAS a partir de Azure Blob Storage.

```python
class MultivariateSample():

def __init__(self, subscription_key, anomaly_detector_endpoint, data_source=None):
    self.sub_key = subscription_key
    self.end_point = anomaly_detector_endpoint

    # Create an Anomaly Detector client

    # <client>
    self.ad_client = AnomalyDetectorClient(AzureKeyCredential(self.sub_key), self.end_point)
    # </client>

    self.data_source = "YOUR_SAMPLE_ZIP_FILE_LOCATED_IN_AZURE_BLOB_STORAGE_WITH_SAS"
```

## <a name="train-the-model"></a>Entrenar el modelo

En primer lugar, entrenaremos el modelo, comprobaremos el estado del modelo durante el entrenamiento para determinar cuándo se ha completado el entrenamiento y, después, recuperaremos el identificador de modelo más reciente que necesitaremos cuando pasemos a la fase de detección.

```python
def train(self, start_time, end_time, max_tryout=500):

    # Number of models available now
    model_list = list(self.ad_client.list_multivariate_model(skip=0, top=10000))
    print("{:d} available models before training.".format(len(model_list)))
    
    # Use sample data to train the model
    print("Training new model...")
    data_feed = ModelInfo(start_time=start_time, end_time=end_time, source=self.data_source)
    response_header = \
    self.ad_client.train_multivariate_model(data_feed, cls=lambda *args: [args[i] for i in range(len(args))])[-1]
    trained_model_id = response_header['Location'].split("/")[-1]
    
    # Model list after training
    new_model_list = list(self.ad_client.list_multivariate_model(skip=0, top=10000))
    
    # Wait until the model is ready. It usually takes several minutes
    model_status = None
    tryout_count = 0
    while (tryout_count < max_tryout and model_status != "READY"):
        model_status = self.ad_client.get_multivariate_model(trained_model_id).model_info.status
        tryout_count += 1
        time.sleep(2)
    
    assert model_status == "READY"
    
    print("Done.", "\n--------------------")
    print("{:d} available models after training.".format(len(new_model_list)))
    
    # Return the latest model id
    return trained_model_id

```

## <a name="detect-anomalies"></a>Detección de anomalías

Use `detect_anomaly` y `get_dectection_result` para determinar si existe cualquier anomalía con el origen de datos. Deberá pasar el identificador de modelo para el modelo que acaba de entrenar.

```python
def detect(self, model_id, start_time, end_time, max_tryout=500):
    
    # Detect anomaly in the same data source (but a different interval)
    try:
        detection_req = DetectionRequest(source=self.data_source, start_time=start_time, end_time=end_time)
        response_header = self.ad_client.detect_anomaly(model_id, detection_req,
                                                        cls=lambda *args: [args[i] for i in range(len(args))])[-1]
        result_id = response_header['Location'].split("/")[-1]
    
        # Get results (may need a few seconds)
        r = self.ad_client.get_detection_result(result_id)
        tryout_count = 0
        while r.summary.status != "READY" and tryout_count < max_tryout:
            time.sleep(1)
            r = self.ad_client.get_detection_result(result_id)
            tryout_count += 1
    
        if r.summary.status != "READY":
            print("Request timeout after %d tryouts.".format(max_tryout))
            return None
    
    except HttpResponseError as e:
        print('Error code: {}'.format(e.error.code), 'Error message: {}'.format(e.error.message))
    except Exception as e:
        raise e
    
    return r
```

## <a name="export-model"></a>Exportación de un modelo

> [!NOTE]
> El comando de exportación está diseñado para permitir la ejecución de modelos multivariados de Anomaly Detector en un entorno en contenedor. Actualmente, no se admite para el multivariado, pero se agregará una compatibilidad en el futuro.

Si desea exportar un modelo, use `export_model` y pase el identificador del modelo que desea exportar:

```python
def export_model(self, model_id, model_path="model.zip"):

    # Export the model
    model_stream_generator = self.ad_client.export_model(model_id)
    with open(model_path, "wb") as f_obj:
        while True:
            try:
                f_obj.write(next(model_stream_generator))
            except StopIteration:
                break
            except Exception as e:
                raise e
```

## <a name="delete-model"></a>Eliminación del modelo

Para eliminar un modelo, use `delete_multivariate_model` y pase el identificador del modelo que desea eliminar:

```python
def delete_model(self, model_id):

    # Delete the mdoel
    self.ad_client.delete_multivariate_model(model_id)
    model_list_after_delete = list(self.ad_client.list_multivariate_model(skip=0, top=10000))
    print("{:d} available models after deletion.".format(len(model_list_after_delete)))
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Antes de ejecutar la aplicación, es necesario agregar código para llamar a las nuevas funciones creadas.

```python
if __name__ == '__main__':
    subscription_key = "ANOMALY_DETECTOR_KEY"
    anomaly_detector_endpoint = "ANOMALY_DETECTOR_ENDPOINT"

    # Create a new sample and client
    sample = MultivariateSample(subscription_key, anomaly_detector_endpoint, data_source=None)

    # Train a new model
    model_id = sample.train(datetime(2021, 1, 1, 0, 0, 0), datetime(2021, 1, 2, 12, 0, 0))

    # Reference
    result = sample.detect(model_id, datetime(2021, 1, 2, 12, 0, 0), datetime(2021, 1, 3, 0, 0, 0))
    print("Result ID:\t", result.result_id)
    print("Result summary:\t", result.summary)
    print("Result length:\t", len(result.results))

    # Export model
    sample.export_model(model_id, "model.zip")

    # Delete model
    sample.delete_model(model_id)

```

Antes de ejecutarla, puede resultar útil comprobar el proyecto con el [código de ejemplo completo](https://github.com/Azure-Samples/AnomalyDetector/blob/master/ipython-notebook/Multivariate%20API%20Demo%20Notebook.ipynb) del que se deriva este inicio rápido.

También tenemos una instancia de [Jupyter Notebook detallada](https://github.com/Azure-Samples/AnomalyDetector/blob/master/ipython-notebook/Multivariate%20API%20Demo%20Notebook.ipynb) para ayudarle a empezar a trabajar.

Ejecute la aplicación con el comando `python` en el nombre de archivo.


## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a dicho grupo.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Anomaly Detector API?](../../overview-multivariate.md)
* [Procedimientos recomendados cuando se usa Anomaly Detector API.](../../concepts/best-practices-multivariate.md) 