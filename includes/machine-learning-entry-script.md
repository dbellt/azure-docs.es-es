---
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: include
ms.date: 07/31/2020
ms.author: gopalv
ms.openlocfilehash: 95acb028867caf6f497dd99ad3082efcaab09c7b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "93325294"
---
El script de entrada recibe los datos enviados a un servicio web implementado y los pasa al modelo. A continuación, toma la respuesta devuelta por el modelo y la envía al cliente. *El script es específico para su modelo*. Debe entender los datos que el modelo espera y devuelve.

Estas son las dos tareas que debe realizar en el script de entrada:

1. Cargar el modelo (mediante una función llamada `init()`)
1. Ejecutar el modelo sobre los datos de entrada (mediante una función llamada `run()`)

Vamos a examinar estos pasos de forma detallada.

### <a name="writing-init"></a>Escritura de init() 

#### <a name="loading-a-registered-model"></a>Carga de un modelo registrado

Los modelos registrados se almacenan en una ruta de acceso a la que apunta una variable de entorno llamada `AZUREML_MODEL_DIR`. Para más información sobre la estructura de directorios exacta, consulte [Búsqueda de archivos de modelo en el script de entrada](../articles/machine-learning/how-to-deploy-advanced-entry-script.md#load-registered-models).

#### <a name="loading-a-local-model"></a>Carga de un modelo local

Si optó por registrar el modelo y pasarlo como parte del directorio de origen, puede leerlo como lo haría localmente, pasando la ruta de acceso al modelo con relación al script de puntuación. Por ejemplo, si tenía un directorio estructurado como:

```bash

- source_dir
    - score.py
    - models
        - model1.onnx

```

podría cargar los modelos con el siguiente código de Python:

```python
import os

model = open(os.path.join('.', 'models', 'model1.onnx'))
```

### <a name="writing-run"></a>Escritura de run()

`run()` se ejecuta cada vez que el modelo recibe una solicitud de puntuación, y espera que el cuerpo de la solicitud sea un documento JSON con la siguiente estructura:

```json
{
    "data": <model-specific-data-structure>
}

```

La entrada a `run()` es una cadena de Python que contiene lo que sigue a la clave "data".

En el ejemplo siguiente se muestra cómo cargar un modelo de scikit-learn registrado y puntuarlo con datos de numpy:

```python
import json
import numpy as np
import os
from sklearn.externals import joblib


def init():
    global model
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(data):
    try:
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Para ver ejemplos más avanzados, como la generación automática de esquemas de Swagger y datos binarios (es decir, imágenes), lea [el artículo sobre la creación avanzada de scripts de entrada](../articles/machine-learning/how-to-deploy-advanced-entry-script.md).