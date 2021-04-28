---
title: Inicio rápido de la biblioteca cliente multivariante de Anomaly Detector .NET
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/06/2021
ms.author: mbullwin
ms.openlocfilehash: a657b8f5bf967131a0168dbea5bb1db86b3b559e
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800373"
---
Comience a usar la biblioteca cliente multivariante de Anomaly Detector para .NET. Siga estos pasos para instalar el paquete y empezar a usar los algoritmos proporcionados por el servicio. Las nuevas API de detección de anomalías multivariante permiten a los desarrolladores integrar fácilmente inteligencia artificial avanzada para detectar anomalías de grupos de métricas, sin necesidad de tener conocimientos de aprendizaje automático ni usar datos etiquetados. Las dependencias y correlaciones entre distintas señales se consideran automáticamente como factores clave. De este modo, podrá proteger de forma proactiva los sistemas complejos frente a errores.

Utilice la biblioteca cliente multivariante de Anomaly Detector para .NET con el fin de:

* Detectar anomalías de nivel del sistema para un grupo de series temporales.
* Cuando una serie temporal individual no ofrece demasiada información y se deben examinar todas las señales para detectar un problema.
* Mantenimiento predictivo de recursos físicos costosos con decenas o cientos de tipos de sensores diferentes que miden diversos aspectos del estado del sistema.

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/anomalydetector/Azure.AI.AnomalyDetector) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.AI.AnomalyDetector/3.0.0-preview.3)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector </a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y seleccione el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. Más adelante en este inicio rápido, debe pegar la clave y el punto de conexión en el código que se incluye a continuación.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-net-core-application"></a>Creación de una aplicación de .NET Core

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `anomaly-detector-quickstart-multivariate`. Este comando crea un sencillo proyecto "Hola mundo" con un solo archivo de origen de C#: *Program.cs*.

```dotnetcli
dotnet new console -n anomaly-detector-quickstart-multivariate
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```dotnetcli
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores.

```output
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Anomaly Detector para .NET con el siguiente comando:

```dotnetcli
dotnet add package Azure.AI.AnomalyDetector --version 3.0.0-preview.3
```

En el directorio del proyecto, abra el archivo *program.cs* y agregue lo siguiente mediante `directives`:

```csharp
using System;
using System.Collections.Generic;
using System.Drawing.Text;
using System.IO;
using System.Linq;
using System.Linq.Expressions;
using System.Net.NetworkInformation;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;
using Azure.AI.AnomalyDetector.Models;
using Azure.Core.TestFramework;
using Microsoft.Identity.Client;
using NUnit.Framework;
```

En el método `main()` de la aplicación, cree variables para el punto de conexión del recurso de Azure, la clave de API y un origen de datos personalizado.

```csharp
string endpoint = "YOUR_API_KEY";
string apiKey =  "YOUR_ENDPOINT";
string datasource = "YOUR_SAMPLE_ZIP_FILE_LOCATED_IN_AZURE_BLOB_STORAGE_WITH_SAS";
```

 Para usar las API multivariante de Anomaly Detector, es necesario entrenar nuestro modelo antes de usar la detección. Los datos usados con fines de entrenamiento constituyen un lote de series temporales, y cada serie temporal debe tener formato CSV con dos columnas: marca de tiempo y valor. Todas las series temporales deben comprimirse en un archivo ZIP y cargarse a [Azure Blob Storage](../../../../storage/blobs/storage-blobs-introduction.md#blobs). De forma predeterminada, el nombre de archivo se usará para representar la variable de la serie temporal. Como alternativa, se puede incluir un archivo meta.json en el archivo ZIP si desea que el nombre de la variable sea diferente al nombre del archivo .zip. Una vez que se genera una [URL de SAS (firmas de acceso compartido) de blob](../../../../storage/common/storage-sas-overview.md), podemos usar la dirección URL para el archivo ZIP para el entrenamiento.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo llevar a cabo las siguientes acciones mediante la biblioteca cliente multivariante de Anomaly Detector para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Entrenamiento del modelo](#train-the-model)
* [Detectar anomalías](#detect-anomalies)
* [Exportar un modelo](#export-model)
* [Eliminar un modelo](#delete-model)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un cliente de Anomaly Detector con el punto de conexión y la clave.

```csharp
var endpointUri = new Uri(endpoint);
var credential = new AzureKeyCredential(apiKey)

AnomalyDetectorClient client = new AnomalyDetectorClient(endpointUri, credential);
```

## <a name="train-the-model"></a>Entrenar el modelo

Cree una nueva tarea asincrónica privada como se indica a continuación para entrenar el modelo. Usará `TrainMultivariateModel` para entrenar el modelo y `GetMultivariateModelAysnc` para comprobar cuándo se ha completado el entrenamiento.

```csharp
private async Task trainAsync(AnomalyDetectorClient client, string datasource, DateTimeOffset start_time, DateTimeOffset end_time, int max_tryout = 500)
{
    try
    {
        Console.WriteLine("Training new model...");

        int model_number = await getModelNumberAsync(client, false).ConfigureAwait(false);
        Console.WriteLine(String.Format("{0} available models before training.", model_number));

        ModelInfo data_feed = new ModelInfo(datasource, start_time, end_time);
        Response response_header = client.TrainMultivariateModel(data_feed);
        response_header.Headers.TryGetValue("Location", out string trained_model_id_path);
        Guid trained_model_id = Guid.Parse(trained_model_id_path.Split('/').LastOrDefault());
        Console.WriteLine(trained_model_id);

        // Wait until the model is ready. It usually takes several minutes
        Response<Model> get_response = await client.GetMultivariateModelAsync(trained_model_id).ConfigureAwait(false);
        ModelStatus? model_status = null;
        int tryout_count = 0;
        TimeSpan create_limit = new TimeSpan(0, 3, 0);
        while (tryout_count < max_tryout & model_status != ModelStatus.Ready)
        {
            System.Threading.Thread.Sleep(10000);
            get_response = await client.GetMultivariateModelAsync(trained_model_id).ConfigureAwait(false);
            ModelInfo model_info = get_response.Value.ModelInfo;
            Console.WriteLine(String.Format("model_id: {0}, createdTime: {1}, lastUpdateTime: {2}, status: {3}.", get_response.Value.ModelId, get_response.Value.CreatedTime, get_response.Value.LastUpdatedTime, model_info.Status));

            if (model_info != null)
            {
                model_status = model_info.Status;
            }
            tryout_count += 1;
        };
        get_response = await client.GetMultivariateModelAsync(trained_model_id).ConfigureAwait(false);

        if (model_status != ModelStatus.Ready)
        {
            Console.WriteLine(String.Format("Request timeout after {0} tryouts", max_tryout));
        }

        model_number = await getModelNumberAsync(client).ConfigureAwait(false);
        Console.WriteLine(String.Format("{0} available models after training.", model_number));
        return trained_model_id;
    }
    catch (Exception e)
    {
        Console.WriteLine(String.Format("Train error. {0}", e.Message));
        throw new Exception(e.Message);
    }
}
```

## <a name="detect-anomalies"></a>Detección de anomalías

Para detectar anomalías con el modelo recién entrenado, cree un `private async Task` denominado `detectAsync`. Creará un nuevo `DetectionRequest` y lo pasará como parámetro a `DetectAnomalyAsync`.

```csharp
private async Task<DetectionResult> detectAsync(AnomalyDetectorClient client, string datasource, Guid model_id, DateTimeOffset start_time, DateTimeOffset end_time, int max_tryout = 500)
{
    try
    {
        Console.WriteLine("Start detect...");
        Response<Model> get_response = await client.GetMultivariateModelAsync(model_id).ConfigureAwait(false);

        DetectionRequest detectionRequest = new DetectionRequest(datasource, start_time, end_time);
        Response result_response = await client.DetectAnomalyAsync(model_id, detectionRequest).ConfigureAwait(false);
        var ok = result_response.Headers.TryGetValue("Location", out string result_id_path);
        Guid result_id = Guid.Parse(result_id_path.Split('/').LastOrDefault());
        // get detection result
        Response<DetectionResult> result = await client.GetDetectionResultAsync(result_id).ConfigureAwait(false);
        int tryout_count = 0;
        while (result.Value.Summary.Status != DetectionStatus.Ready & tryout_count < max_tryout)
        {
            System.Threading.Thread.Sleep(2000);
            result = await client.GetDetectionResultAsync(result_id).ConfigureAwait(false);
            tryout_count += 1;
        }

        if (result.Value.Summary.Status != DetectionStatus.Ready)
        {
            Console.WriteLine(String.Format("Request timeout after {0} tryouts", max_tryout));
            return null;
        }

        return result.Value;
    }
    catch (Exception e)
    {
        Console.WriteLine(String.Format("Detection error. {0}", e.Message));
        throw new Exception(e.Message);
    }
}
```

## <a name="export-model"></a>Exportación de un modelo

Para exportar el modelo que ha entrenado anteriormente, cree un `private async Task` denominado `exportAysnc`. Usará `ExportModelAsync` y pasará el identificador del modelo al modelo que desea exportar.

```csharp
private async Task exportAsync(AnomalyDetectorClient client, Guid model_id, string model_path = "model.zip")
{
    try
    {
        Stream model = await client.ExportModelAsync(model_id).ConfigureAwait(false);
        if (model != null)
        {
            var fileStream = File.Create(model_path);
            model.Seek(0, SeekOrigin.Begin);
            model.CopyTo(fileStream);
            fileStream.Close();
        }
    }
    catch (Exception e)
    {
        Console.WriteLine(String.Format("Export error. {0}", e.Message));
        throw new Exception(e.Message);
    }
}
```

## <a name="delete-model"></a>Eliminación de un modelo

Para eliminar un modelo que ha creado anteriormente, use `DeleteMultivariateModelAsync` y pase el identificador del modelo que desea eliminar. Para recuperar un identificador de modelo, puede usar `getModelNumberAsync`:

```csharp
private async Task deleteAsync(AnomalyDetectorClient client, Guid model_id)
{
    await client.DeleteMultivariateModelAsync(model_id).ConfigureAwait(false);
    int model_number = await getModelNumberAsync(client).ConfigureAwait(false);
    Console.WriteLine(String.Format("{0} available models after deletion.", model_number));
}
private async Task<int> getModelNumberAsync(AnomalyDetectorClient client, bool delete = false)
{
    int count = 0;
    AsyncPageable<ModelSnapshot> model_list = client.ListMultivariateModelAsync(0, 10000);
    await foreach (ModelSnapshot x in model_list)
    {
        count += 1;
        Console.WriteLine(String.Format("model_id: {0}, createdTime: {1}, lastUpdateTime: {2}.", x.ModelId, x.CreatedTime, x.LastUpdatedTime));
        if (delete & count < 4)
        {
            await client.DeleteMultivariateModelAsync(x.ModelId).ConfigureAwait(false);
        }
    }
    return count;
}
```

## <a name="main-method"></a>Main (método)

Ahora que tiene todos los elementos, debe agregar código al método principal para llamar a las tareas recién creadas.

```csharp

{
    //read endpoint and apiKey
     string endpoint = "YOUR_API_KEY";
    string apiKey =  "YOUR_ENDPOINT";
    string datasource = "YOUR_SAMPLE_ZIP_FILE_LOCATED_IN_AZURE_BLOB_STORAGE_WITH_SAS";
    Console.WriteLine(endpoint);
    var endpointUri = new Uri(endpoint);
    var credential = new AzureKeyCredential(apiKey);

    //create client
    AnomalyDetectorClient client = new AnomalyDetectorClient(endpointUri, credential);

    // train
    TimeSpan offset = new TimeSpan(0);
    DateTimeOffset start_time = new DateTimeOffset(2021, 1, 1, 0, 0, 0, offset);
    DateTimeOffset end_time = new DateTimeOffset(2021, 1, 2, 12, 0, 0, offset);
    Guid? model_id_raw = null;
    try
    {
        model_id_raw = await trainAsync(client, datasource, start_time, end_time).ConfigureAwait(false);
        Console.WriteLine(model_id_raw);
        Guid model_id = model_id_raw.GetValueOrDefault();

        // detect
        start_time = end_time;
        end_time = new DateTimeOffset(2021, 1, 3, 0, 0, 0, offset);
        DetectionResult result = await detectAsync(client, datasource, model_id, start_time, end_time).ConfigureAwait(false);
        if (result != null)
        {
            Console.WriteLine(String.Format("Result ID: {0}", result.ResultId));
            Console.WriteLine(String.Format("Result summary: {0}", result.Summary));
            Console.WriteLine(String.Format("Result length: {0}", result.Results.Count));
        }

        // export model
        await exportAsync(client, model_id).ConfigureAwait(false);

        // delete
        await deleteAsync(client, model_id).ConfigureAwait(false);
    }
    catch (Exception e)
    {
        String msg = String.Format("Multivariate error. {0}", e.Message);
        if (model_id_raw != null)
        {
            await deleteAsync(client, model_id_raw.GetValueOrDefault()).ConfigureAwait(false);
        }
        Console.WriteLine(msg);
        throw new Exception(msg);
    }
}

```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `dotnet run` desde el directorio de la aplicación.

```dotnetcli
dotnet run
```

## <a name="next-steps"></a>Pasos siguientes

* [Procedimientos recomendados multivariante de Anomaly Detector](../../concepts/best-practices-multivariate.md)
