---
title: Publicación de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Ejecute flujos de trabajo de aprendizaje automático con canalizaciones de aprendizaje automático y el SDK de Azure Machine Learning para Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: laobri
author: lobrien
ms.date: 8/25/2020
ms.topic: how-to
ms.custom: contperf-fy21q1
ms.openlocfilehash: f8ed4a498029b00922219ffdc22ec2db3a1490e8
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889401"
---
# <a name="publish-and-track-machine-learning-pipelines"></a>Publicación y seguimiento de canalizaciones de aprendizaje automático



En este artículo se muestra cómo compartir una canalización de aprendizaje automático con sus compañeros o clientes.

Las canalizaciones de aprendizaje automático son flujos de trabajo reutilizables para las tareas de aprendizaje automático. Una ventaja de las canalizaciones es aumentar la colaboración. También puede usar canalizaciones de versión, lo que permite a los clientes utilizar el modelo actual mientras trabaja en una nueva versión. 

## <a name="prerequisites"></a>Requisitos previos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) que contenga todos los recursos de canalización.

* [Configure su entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) con el SDK ya instalado.

* Cree y ejecute una canalización de aprendizaje automático, como en el siguiente [Tutorial: Compilación de una canalización de Azure Machine Learning para la puntuación por lotes](tutorial-pipeline-batch-scoring-classification.md). Para otras opciones, consulte [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](./how-to-create-machine-learning-pipelines.md).

## <a name="publish-a-pipeline"></a>Publicar una canalización

Una vez que tenga una canalización en funcionamiento, puede publicarla para que se ejecute con diferentes entradas. Para que el punto de conexión REST de una canalización ya publicada acepte parámetros, tiene que configurar la canalización para que use objetos `PipelineParameter` para los argumentos que varían.

1. Para crear un parámetro de canalización, use un objeto [PipelineParameter](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter) con un valor predeterminado.

   ```python
   from azureml.pipeline.core.graph import PipelineParameter
   
   pipeline_param = PipelineParameter(
     name="pipeline_arg",
     default_value=10)
   ```

2. Agregue este objeto `PipelineParameter` como parámetro a cualquiera de los pasos de la canalización tal como se muestra a continuación:

   ```python
   compareStep = PythonScriptStep(
     script_name="compare.py",
     arguments=["--comp_data1", comp_data1, "--comp_data2", comp_data2, "--output_data", out_data3, "--param1", pipeline_param],
     inputs=[ comp_data1, comp_data2],
     outputs=[out_data3],
     compute_target=compute_target,
     source_directory=project_folder)
   ```

3. Publique esta canalización para que acepte un parámetro cuando se invoque.

   ```python
   published_pipeline1 = pipeline_run1.publish_pipeline(
        name="My_Published_Pipeline",
        description="My Published Pipeline Description",
        version="1.0")
   ```

## <a name="run-a-published-pipeline"></a>Ejecutar una canalización publicada

Todas las canalizaciones publicadas tienen un punto de conexión REST. Con el punto de conexión de canalización, puede desencadenar una ejecución de la canalización desde cualquier sistema externo, incluidos los clientes que no sean de Python. Este punto de conexión habilita la "repetibilidad administrada" en los escenarios de puntuación y nuevo entrenamiento.

> [!IMPORTANT]
> Si utiliza el control basado en roles de Azure (Azure RBAC) para administrar el acceso a la canalización, [establezca los permisos del escenario de canalización (entrenamiento o puntuación)](how-to-assign-roles.md#common-scenarios).

Para invocar el proceso de ejecución de la canalización anterior, necesitará un token de encabezado de autenticación de Azure Active Directory. La obtención de este tipo de token se describe en la referencia de [clase AzureCliAuthentication](/python/api/azureml-core/azureml.core.authentication.azurecliauthentication) y en el cuaderno [Autenticación en Azure Machine Learning](https://aka.ms/pl-restep-auth).

```python
from azureml.pipeline.core import PublishedPipeline
import requests

response = requests.post(published_pipeline1.endpoint,
                         headers=aad_token,
                         json={"ExperimentName": "My_Pipeline",
                               "ParameterAssignments": {"pipeline_arg": 20}})
```

El argumento `json` de la solicitud POST debe contener, para la clave `ParameterAssignments`, un diccionario que contenga los parámetros de canalización y sus valores. Además, el argumento `json` puede contener las claves siguientes:

| Clave | Descripción |
| --- | --- | 
| `ExperimentName` | El nombre del experimento asociado a este punto de conexión |
| `Description` | Texto de forma libre que describe el punto de conexión | 
| `Tags` | Pares clave-valor de forma libre que se pueden usar para etiquetar y anotar solicitudes  |
| `DataSetDefinitionValueAssignments` | Diccionario usado para cambiar conjuntos de datos sin volver a entrenar (vea la siguiente explicación) | 
| `DataPathAssignments` | Diccionario usado para cambiar rutas de acceso de datos sin volver a entrenar (vea la siguiente explicación) | 

### <a name="run-a-published-pipeline-using-c"></a>Ejecución de una canalización publicada con C# 

En el código siguiente, se muestra cómo se llama de forma asincrónica a una canalización desde C#. Este fragmento de código parcial no forma parte de ningún ejemplo de Microsoft y solo ilustra la estructura de la llamada. No se muestra el proceso completo de control de errores o clases. 

```csharp
[DataContract]
public class SubmitPipelineRunRequest
{
    [DataMember]
    public string ExperimentName { get; set; }

    [DataMember]
    public string Description { get; set; }

    [DataMember(IsRequired = false)]
    public IDictionary<string, string> ParameterAssignments { get; set; }
}

// ... in its own class and method ... 
const string RestEndpoint = "your-pipeline-endpoint";

using (HttpClient client = new HttpClient())
{
    var submitPipelineRunRequest = new SubmitPipelineRunRequest()
    {
        ExperimentName = "YourExperimentName", 
        Description = "Asynchronous C# REST api call", 
        ParameterAssignments = new Dictionary<string, string>
        {
            {
                // Replace with your pipeline parameter keys and values
                "your-pipeline-parameter", "default-value"
            }
        }
    };

    string auth_key = "your-auth-key"; 
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", auth_key);

    // submit the job
    var requestPayload = JsonConvert.SerializeObject(submitPipelineRunRequest);
    var httpContent = new StringContent(requestPayload, Encoding.UTF8, "application/json");
    var submitResponse = await client.PostAsync(RestEndpoint, httpContent).ConfigureAwait(false);
    if (!submitResponse.IsSuccessStatusCode)
    {
        await WriteFailedResponse(submitResponse); // ... method not shown ...
        return;
    }

    var result = await submitResponse.Content.ReadAsStringAsync().ConfigureAwait(false);
    var obj = JObject.Parse(result);
    // ... use `obj` dictionary to access results
}
```

### <a name="run-a-published-pipeline-using-java"></a>Ejecución de una canalización publicada con Java

En el código siguiente, se muestra una llamada a una canalización que requiere autenticación (consulte [Configuración de la autenticación para recursos y flujos de trabajo de Azure Machine Learning](how-to-setup-authentication.md)). Si la canalización está implementada públicamente, no se necesitan las llamadas que generan `authKey`. En este fragmento de código parcial, no se muestra la clase de Java ni el código reutilizable de control de excepciones. El código utiliza `Optional.flatMap` para encadenar funciones que pueden devolver un objeto `Optional` vacío. Al utilizar `flatMap`, el código es más corto y más claro, pero tenga en cuenta que `getRequestBody()` pasa por alto las excepciones.

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Optional;
// JSON library
import com.google.gson.Gson;

String scoringUri = "scoring-endpoint";
String tenantId = "your-tenant-id";
String clientId = "your-client-id";
String clientSecret = "your-client-secret";
String resourceManagerUrl = "https://management.azure.com";
String dataToBeScored = "{ \"ExperimentName\" : \"My_Pipeline\", \"ParameterAssignments\" : { \"pipeline_arg\" : \"20\" }}";

HttpClient client = HttpClient.newBuilder().build();
Gson gson = new Gson();

HttpRequest tokenAuthenticationRequest = tokenAuthenticationRequest(tenantId, clientId, clientSecret, resourceManagerUrl);
Optional<String> authBody = getRequestBody(client, tokenAuthenticationRequest);
Optional<String> authKey = authBody.flatMap(body -> Optional.of(gson.fromJson(body, AuthenticationBody.class).access_token);;
Optional<HttpRequest> scoringRequest = authKey.flatMap(key -> Optional.of(scoringRequest(key, scoringUri, dataToBeScored)));
Optional<String> scoringResult = scoringRequest.flatMap(req -> getRequestBody(client, req));
// ... etc (`scoringResult.orElse()`) ... 

static HttpRequest tokenAuthenticationRequest(String tenantId, String clientId, String clientSecret, String resourceManagerUrl)
{
    String authUrl = String.format("https://login.microsoftonline.com/%s/oauth2/token", tenantId);
    String clientIdParam = String.format("client_id=%s", clientId);
    String resourceParam = String.format("resource=%s", resourceManagerUrl);
    String clientSecretParam = String.format("client_secret=%s", clientSecret);

    String bodyString = String.format("grant_type=client_credentials&%s&%s&%s", clientIdParam, resourceParam, clientSecretParam);

    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create(authUrl))
        .POST(HttpRequest.BodyPublishers.ofString(bodyString))
        .build();
    return request;
}

static HttpRequest scoringRequest(String authKey, String scoringUri, String dataToBeScored)
{
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create(scoringUri))
        .header("Authorization", String.format("Token %s", authKey))
        .POST(HttpRequest.BodyPublishers.ofString(dataToBeScored))
        .build();
    return request;

}

static Optional<String> getRequestBody(HttpClient client, HttpRequest request) {
    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        if (response.statusCode() != 200) {
            System.out.println(String.format("Unexpected server response %d", response.statusCode()));
            return Optional.empty();
        }
        return Optional.of(response.body());
    }catch(Exception x)
    {
        System.out.println(x.toString());
        return Optional.empty();
    }
}

class AuthenticationBody {
    String access_token;
    String token_type;
    int expires_in;
    String scope;
    String refresh_token;
    String id_token;
    
    AuthenticationBody() {}
}
```

### <a name="changing-datasets-and-datapaths-without-retraining"></a>Cambio de conjuntos de datos y rutas de acceso de datos sin volver a entrenar

Tal vez quiera entrenar e inferir en diferentes conjuntos de datos y rutas de acceso de datos. Por ejemplo, puede que desee entrenar un conjunto de datos menor, pero sacar inferencias del conjunto de datos completo. Puede cambiar los conjuntos de datos con la clave `DataSetDefinitionValueAssignments` en el argumento `json` de la solicitud. Puede cambiar las rutas de acceso de datos con `DataPathAssignments`. La técnica para ambos es similar:

1. En el script de definición de la canalización, cree un parámetro `PipelineParameter` para el conjunto de datos. Cree un `DatasetConsumptionConfig` o `DataPath` a partir de `PipelineParameter`:

    ```python
    tabular_dataset = Dataset.Tabular.from_delimited_files('https://dprepdata.blob.core.windows.net/demo/Titanic.csv')
    tabular_pipeline_param = PipelineParameter(name="tabular_ds_param", default_value=tabular_dataset)
    tabular_ds_consumption = DatasetConsumptionConfig("tabular_dataset", tabular_pipeline_param)
    ```

1. En el script de ML, acceda al conjunto de datos especificado de forma dinámica mediante `Run.get_context().input_datasets`:

    ```python
    from azureml.core import Run
    
    input_tabular_ds = Run.get_context().input_datasets['tabular_dataset']
    dataframe = input_tabular_ds.to_pandas_dataframe()
    # ... etc ...
    ```

    Observe que el script de ML tiene acceso al valor especificado para `DatasetConsumptionConfig` (`tabular_dataset`) y no al valor de `PipelineParameter` (`tabular_ds_param`).

1. En el script de definición de la canalización, establezca `DatasetConsumptionConfig` como un parámetro en `PipelineScriptStep`:

    ```python
    train_step = PythonScriptStep(
        name="train_step",
        script_name="train_with_dataset.py",
        arguments=["--param1", tabular_ds_consumption],
        inputs=[tabular_ds_consumption],
        compute_target=compute_target,
        source_directory=source_directory)
    
    pipeline = Pipeline(workspace=ws, steps=[train_step])
    ```

1. Para cambiar los conjuntos de datos de forma dinámica en la llamada de REST de inferencia, use `DataSetDefinitionValueAssignments`:
    
    ```python
    tabular_ds1 = Dataset.Tabular.from_delimited_files('path_to_training_dataset')
    tabular_ds2 = Dataset.Tabular.from_delimited_files('path_to_inference_dataset')
    ds1_id = tabular_ds1.id
    d22_id = tabular_ds2.id
    
    response = requests.post(rest_endpoint, 
                             headers=aad_token, 
                             json={
                                "ExperimentName": "MyRestPipeline",
                               "DataSetDefinitionValueAssignments": {
                                    "tabular_ds_param": {
                                        "SavedDataSetReference": {"Id": ds1_id #or ds2_id
                                    }}}})
    ```

Los cuadernos [Presentación del conjunto de datos y PipelineParameter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb) y [Presentación de la ruta de acceso de datos y PipelineParameter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-datapath-and-pipelineparameter.ipynb) contienen ejemplos completos de esta técnica.

## <a name="create-a-versioned-pipeline-endpoint"></a>Creación de un punto de conexión de canalización con versiones

Puede crear un punto de conexión de canalización con varias canalizaciones publicadas detrás. Esta técnica le proporciona un punto de conexión de REST fijo a medida que itera por las canalizaciones de ML y las actualiza.

```python
from azureml.pipeline.core import PipelineEndpoint

published_pipeline = PipelineEndpoint.get(workspace=ws, name="My_Published_Pipeline")
pipeline_endpoint = PipelineEndpoint.publish(workspace=ws, name="PipelineEndpointTest",
                                            pipeline=published_pipeline, description="Test description Notebook")
```

## <a name="submit-a-job-to-a-pipeline-endpoint"></a>Envío de un trabajo a un punto de conexión de canalización

Puede enviar un trabajo a la versión predeterminada de un punto de conexión de canalización:

```python
pipeline_endpoint_by_name = PipelineEndpoint.get(workspace=ws, name="PipelineEndpointTest")
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment")
print(run_id)
```

También puede enviar un trabajo a una versión específica:

```python
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment", pipeline_version="0")
print(run_id)
```

Lo mismo puede hacerse mediante la API REST:

```python
rest_endpoint = pipeline_endpoint_by_name.endpoint
response = requests.post(rest_endpoint, 
                         headers=aad_token, 
                         json={"ExperimentName": "PipelineEndpointExperiment",
                               "RunSource": "API",
                               "ParameterAssignments": {"1": "united", "2":"city"}})
```

## <a name="use-published-pipelines-in-the-studio"></a>Uso de canalizaciones publicadas en Studio

También puede ejecutar una canalización publicada desde Studio:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. [Vea su área de trabajo](how-to-manage-workspace.md#view).

1. En el panel izquierdo, seleccione **Puntos de conexión**.

1. En la parte superior, seleccione **Pipeline endpoints** (Puntos de conexión de la canalización).
 ![lista de las canalizaciones publicadas de Machine Learning](./media/how-to-create-your-first-pipeline/pipeline-endpoints.png)

1. Seleccione una canalización específica para ejecutar, utilizar o revisar los resultados de las ejecuciones anteriores del punto de conexión de la canalización.

## <a name="disable-a-published-pipeline&quot;></a>Deshabilitar una canalización publicada

Para ocultar una canalización de la lista de canalizaciones publicadas hay que deshabilitarla, ya sea en Studio o en el SDK:

```python
# Get the pipeline by using its ID from Azure Machine Learning studio
p = PublishedPipeline.get(ws, id=&quot;068f4885-7088-424b-8ce2-eeb9ba5381a6")
p.disable()
```

Puede habilitarla de nuevo con `p.enable()`. Para más información, consulte la referencia de la [clase PublishedPipeline](/python/api/azureml-pipeline-core/azureml.pipeline.core.publishedpipeline).

## <a name="next-steps"></a>Pasos siguientes

- Use [estos cuadernos de Jupyter en GitHub](https://aka.ms/aml-pipeline-readme) para explorar aún más canalizaciones de Machine Learning.
- Consulte la ayuda de referencia del SDK para el paquete [azureml-pipelines-core](/python/api/azureml-pipeline-core/) y el paquete [azureml-pipelines-steps](/python/api/azureml-pipeline-steps/).
- Consulte los [procedimientos](how-to-debug-pipelines.md) para obtener sugerencias sobre la depuración y la solución de problemas de canalizaciones.