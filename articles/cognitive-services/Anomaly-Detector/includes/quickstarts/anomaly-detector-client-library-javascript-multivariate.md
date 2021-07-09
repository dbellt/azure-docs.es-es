---
title: Inicio rápido de la biblioteca cliente multivariante de JavaScript de Anomaly Detector
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/29/2021
ms.author: mbullwin
ms.openlocfilehash: 087640433d592c7e2885e68240e98e7283bf3407
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165143"
---
Comience a usar la biblioteca cliente multivariante de Anomaly Detector para JavaScript. Siga estos pasos para instalar el paquete y empezar a usar los algoritmos proporcionados por el servicio. Las nuevas API de detección de anomalías multivariante permiten a los desarrolladores integrar fácilmente inteligencia artificial avanzada para detectar anomalías de grupos de métricas, sin necesidad de tener conocimientos de aprendizaje automático ni usar datos etiquetados. Las dependencias y correlaciones entre distintas señales se consideran automáticamente como factores clave. De este modo, podrá proteger de forma proactiva los sistemas complejos frente a errores.

Utilice la biblioteca cliente multivariante de Anomaly Detector para JavaScript con el fin de:

* Detectar anomalías de nivel del sistema para un grupo de series temporales.
* Cuando una serie temporal individual no ofrece demasiada información y se deben examinar todas las señales para detectar un problema.
* Mantenimiento predictivo de recursos físicos costosos con decenas o cientos de tipos de sensores diferentes que miden diversos aspectos del estado del sistema.

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/anomalydetector/ai-anomaly-detector) | [Paquete (npm)](https://www.npmjs.com/package/@azure/ai-anomaly-detector) | [Código de ejemplo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/anomalydetector/ai-anomaly-detector/samples/v3/javascript/sample_multivariate_detection.js)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org/)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector </a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

Cree un archivo llamado `index.js` e importe las bibliotecas siguientes:
```javascript
'use strict'

const fs = require('fs');
const parse = require("csv-parse/lib/sync");
const { AnomalyDetectorClient } = require('@azure/ai-anomaly-detector');
const { AzureKeyCredential } = require('@azure/core-auth');
```

Cree variables para el punto de conexión y la clave de Azure del recurso. Cree otra variable para el archivo de datos de ejemplo.

> [!NOTE]
> Siempre tendrá la opción de usar una de estas dos claves. Esto es para permitir la rotación de claves segura. Para este inicio rápido, use la primera clave. 
   

```javascript
const apiKey = "YOUR_API_KEY";
const endpoint = "YOUR_ENDPOINT";
const data_source = "YOUR_SAMPLE_ZIP_FILE_LOCATED_IN_AZURE_BLOB_STORAGE_WITH_SAS";
```

Para usar las API multivariantes de Anomaly Detector, primero debe entrenar sus propios modelos. Los datos de entrenamiento son un conjunto de varias series temporales que cumplen los siguientes requisitos:

Cada serie temporal debe ser un archivo CSV con dos columnas (solo dos), "timestamp" y "value" (todo en minúsculas) como fila de encabezado. Los valores de "timestamp" deben cumplir la norma ISO 8601; los de "value" pueden ser números enteros o decimales con cualquier número de posiciones decimales. Por ejemplo:

|timestamp | value|
|-------|-------|
|2019-04-01T00:00:00Z| 5|
|2019-04-01T00:01:00Z| 3.6|
|2019-04-01T00:02:00Z| 4|
|`...`| `...` |

Cada archivo CSV debe tener el nombre de una variable diferente que se usará para el entrenamiento del modelo. Por ejemplo, "temperature.csv" y "humidity.csv". Todos los archivos CSV deben comprimirse en un archivo ZIP sin subcarpetas. El archivo ZIP puede tener el nombre que desee. El archivo ZIP debe cargarse en Azure Blob Storage. Una vez que genere la dirección URL de SAS de blob (firmas de acceso compartido) para el archivo ZIP, se puede usar para el entrenamiento. Consulte este documento para obtener información sobre cómo generar direcciones URL de SAS a partir de Azure Blob Storage.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale los paquetes `ms-rest-azure` y `azure-ai-anomalydetector` de NPM. La biblioteca de análisis de CSV también se usa en este inicio rápido:

```console
npm install @azure/ai-anomaly-detector csv-parse
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de Anomaly Detector para Node.js:

* [Autenticar el cliente](#authenticate-the-client)
* [Entrenamiento de un modelo](#train-a-model)
* [Detectar anomalías](#detect-anomalies)
* [Exportar un modelo](#export-model)
* [Eliminar un modelo](#delete-model)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto `AnomalyDetectorClient` con el punto de conexión y las credenciales.

```javascript
const client = new AnomalyDetectorClient(endpoint, new AzureKeyCredential(apiKey));
```

## <a name="train-a-model"></a>Entrenamiento de un modelo

### <a name="construct-a-model-result"></a>Construcción de un resultado de modelo

En primer lugar, es necesario construir una solicitud de modelo. Asegúrese de que la hora de inicio y finalización se correspondan con su origen de datos.

```javascript
const Modelrequest = {
      source: data_source,
      startTime: new Date(2021,0,1,0,0,0),
      endTime: new Date(2021,0,2,12,0,0),
      slidingWindow:200
    };    
```

### <a name="train-a-new-model"></a>Entrenamiento de un modelo nuevo

Tendrá que pasar la solicitud de modelo al método `trainMultivariateModel` del cliente de Anomaly Detector.

```javascript
console.log("Training a new model...")
const train_response = await client.trainMultivariateModel(Modelrequest)
const model_id = train_response.location?.split("/").pop() ?? ""
console.log("New model ID: " + model_id)
```

Para comprobar si el entrenamiento del modelo está completo, puede realizar un seguimiento del estado del modelo:

```javascript
let model_response = await client.getMultivariateModel(model_id)
let model_status = model_response.modelInfo?.status

while (model_status != 'READY'){
    await sleep(10000).then(() => {});
    model_response = await client.getMultivariateModel(model_id)
    model_status = model_response.modelInfo?.status
}

console.log("TRAINING FINISHED.")
```

## <a name="detect-anomalies"></a>Detección de anomalías

Use las funciones `detectAnomaly` y `getDectectionResult` para determinar si existe cualquier anomalía con el origen de datos.

```javascript
console.log("Start detecting...")
const detect_request = {
    source: data_source,
    startTime: new Date(2021,0,2,12,0,0),
    endTime: new Date(2021,0,3,0,0,0)
};
const result_header = await client.detectAnomaly(model_id, detect_request)
const result_id = result_header.location?.split("/").pop() ?? ""
let result = await client.getDetectionResult(result_id)
let result_status = result.summary.status

while (result_status != 'READY'){
    await sleep(2000).then(() => {});
    result = await client.getDetectionResult(result_id)
    result_status = result.summary.status
}
```

## <a name="export-model"></a>Exportación de un modelo

> [!NOTE]
> El comando de exportación está diseñado para permitir la ejecución de modelos multivariados de Anomaly Detector en un entorno en contenedor. Actualmente, no se admite para el multivariado, pero se agregará una compatibilidad en el futuro.

Para exportar el modelo entrenado, use la función `exportModel`.

```javascript
const export_result = await client.exportModel(model_id)
const model_path = "model.zip"
const destination = fs.createWriteStream(model_path)
export_result.readableStreamBody?.pipe(destination)
console.log("New model has been exported to "+model_path+".")
```

## <a name="delete-model"></a>Eliminación del modelo

Para eliminar un modelo existente que esté disponible para el recurso actual, use la función `deleteMultivariateModel`.

```javascript
client.deleteMultivariateModel(model_id)
console.log("New model has been deleted.")
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Antes de ejecutar la aplicación, puede resultar útil comprobar el código con el [código de ejemplo completo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/anomalydetector/ai-anomaly-detector/samples/v3/javascript/sample_multivariate_detection.js).

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a dicho grupo.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Anomaly Detector API?](../../overview-multivariate.md)
* [Procedimientos recomendados cuando se usa Anomaly Detector API.](../../concepts/best-practices-multivariate.md) 