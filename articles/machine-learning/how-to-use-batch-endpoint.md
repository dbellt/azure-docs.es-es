---
title: Uso de puntos de conexión por lotes para la puntuación por lotes
titleSuffix: Azure Machine Learning
description: En este artículo, aprenderá a crear un punto de conexión por lotes para puntuar continuamente datos de gran tamaño por lotes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
author: tracych
ms.author: tracych
ms.reviewer: laobri
ms.date: 5/20/2021
ms.custom: how-to
ms.openlocfilehash: b4484a22d8839e758f7ccbb0a43904b81b028909
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382982"
---
# <a name="use-batch-endpoints-preview-for-batch-scoring"></a>Uso de puntos de conexión por lotes (versión preliminar) para la puntuación por lotes

En este artículo, aprenderá a usar puntos de conexión por lotes (versión preliminar) para realizar la puntuación por lotes. Los puntos de conexión por lotes simplifican el proceso de hospedar los modelos para la puntuación por lotes, por lo que puede centrarse en el aprendizaje automático y dejar a un lado la infraestructura. Después de crear un punto de conexión por lotes, puede desencadenar trabajos de puntuación por lotes con la CLI de Azure o desde cualquier plataforma mediante una biblioteca HTTP y la API REST. Para más información, consulte [¿Qué son los puntos de conexión de Azure Machine Learning (versión preliminar)?](concept-endpoints.md).

En este artículo, aprenderá a realizar las siguientes tareas:

> [!div class="checklist"]
> * Creación de un punto de conexión por lotes con una experiencia sin código para el modelo MLflow
> * Comprobación de los detalles de un punto de conexión por lotes
> * Inicio de un trabajo de puntuación por lotes mediante la CLI
> * Supervisión del progreso de la ejecución de trabajos de puntuación por lotes y comprobar los resultados de puntuación
> * Adición de una nueva implementación a un punto de conexión por lotes
> * Inicio de un trabajo de puntuación por lotes mediante REST

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: si aún no tiene una, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* La interfaz de la línea de comandos de Azure (CLI) y la extensión ML.

La extensión Machine Learning requiere la versión `>=2.15.0` de la CLI de Azure. Asegúrese de que se cumple este requisito:

```azurecli
az version
```

Si es necesario, actualice la CLI de Azure:

```azurecli
az upgrade
```

> [!NOTE]
>
> El comando `az upgrade` se agregó en la versión 2.11.0 y no funcionará con versiones anteriores a la 2.11.0. Las versiones anteriores se pueden actualizar reinstalando como se indica en [Instalación de la CLI de Azure](/cli/azure/update-azure-cli).
>
> Este comando también actualizará todas las extensiones instaladas de forma predeterminada. Para más opciones de `az upgrade`, consulte la [página de referencia del comando](/cli/azure/reference-index#az_upgrade).

Agregue y configure la extensión Azure ML:

```azurecli
az extension add  ml
```

Para más información sobre cómo configurar la extensión ML, consulte [Instalación, configuración y uso de la CLI 2.0 (versión preliminar)](how-to-configure-cli.md).

* Repositorio de ejemplo

Clone el [repositorio de ejemplo de AzureML](https://github.com/Azure/azureml-examples). En este artículo se usan los recursos de `/cli-preview/experiment/using-cli/assets/endpoints/batch`.

## <a name="create-a-compute-target"></a>Creación de un destino de proceso

La puntuación por lotes solo se ejecuta en recursos de informática en la nube, no de forma local. El recurso de informática en la nube se denomina "destino de proceso". Un destino de proceso es un equipo virtual reutilizable donde se pueden ejecutar flujos de trabajo de puntuación por lotes.

Ejecute el código siguiente para crear un destino de [`AmlCompute`](/python/api/azureml-core/azureml.core.compute.amlcompute(class)?view=azure-ml-py&preserve-view=true) de uso general. Para más información sobre los destinos de proceso, consulte [¿Qué son los destinos de proceso en Azure Machine Learning?](./concept-compute-target.md)

```azurecli
az ml compute create --name cpu-cluster --type AmlCompute --min-instances 0 --max-instances 5
```

## <a name="create-a-batch-endpoint"></a>Creación de un punto de conexión por lotes

Si usa un modelo MLflow, puede utilizar la creación de puntos de conexión por lotes sin código. Es decir, no es necesario preparar un script y un entorno de puntuación, ya que ambos se pueden generar automáticamente. Para más información, consulte [Entrenamiento y seguimiento de modelos de Machine Learning con MLflow y Azure Machine Learning (versión preliminar)](how-to-use-mlflow.md).

```azurecli
az ml endpoint create --type batch --file cli/endpoints/batch/create-batch-endpoint.yml
```

A continuación, se muestra el archivo YAML que define el punto de conexión por lotes de MLFlow:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/endpoints/batch/create-batch-endpoint.yml":::

| Clave | Descripción |
| --- | ----------- |
| $schema | [Opcional] El esquema de YAML. Puede ver el esquema del ejemplo anterior en un explorador para examinar todas las opciones disponibles para un archivo YAML de punto de conexión por lotes. |
| name | Nombre del punto de conexión por lotes, que debe ser único en toda la región. El valor `name` se usará como parte del URI de puntuación. El valor debe comenzar con un carácter en inglés, continuar con una combinación de números, caracteres y el símbolo `-` y finalizar con un número o un carácter. Debe tener tres caracteres de longitud, como mínimo. La expresión regular de validación es: `^[a-zA-Z][-a-zA-Z0-9]+[a-zA-Z0-9]$`|
| tipo | Tipo del punto de conexión. Use `batch` para el punto de conexión por lotes. |
| auth_mode | Use `aad_token` para la autenticación basada en tokens de Azure. |
| traffic | Porcentaje de tráfico enrutado a esta implementación. En los puntos de conexión por lotes, los únicos valores válidos para `traffic` son `0` o `100`. La implementación con un valor de tráfico de `100` está activa. Cuando se invoca, todos los datos se envían a la implementación activa. |
| deployments | Lista de implementaciones que se crearán en el punto de conexión por lotes. El ejemplo solo tiene una implementación denominada `autolog_deployment`. |

Atributos de implementación:

| Clave | Descripción |
| --- | ----------- |
| name | Nombre de la implementación. |
| model | Modelo que se va a usar para la puntuación por lotes. Use `name`, `version` y `local_path` para cargar un modelo desde la máquina local. Use el prefijo `azureml:` para hacer referencia a un recurso de modelo existente en el área de trabajo. Por ejemplo, `azureml: autolog:1` apuntaría a la versión 1 de un modelo denominado `autolog`. |
| compute.target | El destino de proceso. Use el prefijo `azureml:` para hacer referencia a un recurso de proceso existente en el área de trabajo. Por ejemplo, `azureml:cpu-cluster` apuntaría a un destino de proceso denominado `cpu-cluster`. |
| compute.instance_count | Número de nodos de proceso que se usarán para la puntuación por lotes. El valor predeterminado es `1`.|
| mini_batch_size | [Opcional] Número de archivos que `scoring_script` puede procesar en una llamada a `run()`. El valor predeterminado es `10`. |
| output_file_name | [Opcional] Nombre del archivo de salida de puntuación por lotes. El valor predeterminado es `predictions.csv`. |
| retry_settings.max_retries | [Opcional] Número máximo de intentos erróneos de una llamada a `run()` de `scoring_script`. El valor predeterminado es `3`. |
| retry_settings.timeout | [Opcional] Tiempo de espera en segundos de una llamada a `run()` de `scoring_script`. El valor predeterminado es `30`. |
| error_threshold | [Opcional] El número de errores de archivo que se deben omitir. Si el recuento de errores de toda la entrada supera este valor, el trabajo se terminará. El umbral de error es para toda la entrada y no para los minilotes individuales que se envían al método `run()`. El valor predeterminado es `-1`, que especifica que se permite cualquier número de errores sin terminar la ejecución. | 
| logging_level | [Opcional] Nivel de detalle del registro. Los valores en el aumento del nivel de detalle son: WARNING, INFO y DEBUG. El valor predeterminado es INFO. |

## <a name="check-batch-endpoint-details"></a>Comprobación de los detalles del punto de conexión por lotes

Después de crear un punto de conexión por lotes, puede usar `show` para comprobar los detalles. Use [`--query parameter`](/cli/azure/query-azure-cli) para obtener solo atributos específicos de los datos devueltos.

```azurecli
az ml endpoint show --name mybatchedp --type batch
```

## <a name="start-a-batch-scoring-job-using-cli"></a>Inicio de un trabajo de puntuación por lotes mediante la CLI

Una carga de trabajo de puntuación por lotes se ejecuta como un trabajo sin conexión. La puntuación por lotes está concebida para procesar datos de gran tamaño. Las entradas se procesan en paralelo en el clúster de proceso. Una partición de datos se asigna a un proceso en un nodo. Un único nodo con varios procesos tendrá varias particiones ejecutadas en paralelo. De forma predeterminada, la puntuación por lotes almacena las salidas de puntuación en almacenamiento de blobs. Puede iniciar un trabajo de puntuación por lotes mediante la CLI pasando las entradas de datos. También puede configurar la ubicación de las salidas y sobrescribir algunas de las opciones para obtener el mejor rendimiento.

### <a name="start-a-bath-scoring-job-with-different-inputs-options"></a>Inicio de un trabajo de puntuación por lotes con distintas opciones de entrada

Tiene tres opciones para especificar las entradas de datos.

Opción 1: Datos registrados

Use `--input-data` para pasar datos registrados de AML.

> [!NOTE]
> Durante la versión preliminar, solo se admite FileDataset. 

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-data azureml:<dataName>:<dataVersion>
```

Opción 2: Datos en la nube

Use `--input-datastore` para especificar un almacén de datos registrado de AML y `--input-path` para especificar la ruta de acceso relativa en el almacén de datos.

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-datastore azureml:<datastoreName> --input-path <relativePath>
```

Si los datos están disponibles públicamente, use `--input-path` para especificar la ruta de acceso pública.

Si va a usar el ejemplo proporcionado, puede ejecutar el siguiente comando para iniciar un trabajo de puntuación por lotes.

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-path https://pipelinedata.blob.core.windows.net/sampledata/nytaxi/taxi-tip-data.csv
```

Opción 3: Datos almacenados localmente

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-local-path <localPath>
```

### <a name="configure-the-output-location"></a>Configuración de la ubicación de salida

Los resultados de la puntuación por lotes se almacenan de forma predeterminada en el almacén de blobs predeterminado del área de trabajo dentro de una carpeta designada por el nombre del trabajo (un GUID generado por el sistema). Puede configurar dónde almacenar las salidas de puntuación al iniciar un trabajo de puntuación por lotes. Use `--output-datastore` para configurar cualquier almacén de datos registrado y `--output-path` para configurar la ruta de acceso relativa. Use `--set output_file_name` para configurar un nuevo nombre de archivo de salida.

> [!IMPORTANT]
> Debe usar una ubicación de salida única. Si el archivo de salida existe, el trabajo de puntuación por lotes generará errores. 

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-path https://pipelinedata.blob.core.windows.net/sampledata/nytaxi/taxi-tip-data.csv --output-datastore azureml:workspaceblobstore --output-path mypath --set output_file_name=mypredictions.csv
```

### <a name="overwrite-settings"></a>Sobrescritura de la configuración

Algunas opciones de configuración se pueden sobrescribir al iniciar un trabajo de puntuación por lotes para hacer un uso mejor del recurso de proceso y mejorar el rendimiento: 

* Use `--mini-batch-size` para sobrescribir `mini_batch_size` si se usa un tamaño diferente de datos de entrada. 
* Use `--instance-count` para sobrescribir `instance_count` si se necesita un recurso de proceso diferente para este trabajo. 
* Use `--set` para sobrescribir otros valores de configuración, como `max_retries`, `timeout`, `error_threshold` y `logging_level`.

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-path https://pipelinedata.blob.core.windows.net/sampledata/nytaxi/taxi-tip-data.csv --set retry_settings.max_retries=1
```

## <a name="check-batch-scoring-job-execution-progress"></a>Comprobación del progreso de la ejecución del trabajo de puntuación por lotes

Los trabajos de puntuación por lotes suelen tardar algún tiempo en procesar todo el conjunto de entradas. Puede supervisar el progreso del trabajo desde Azure Machine Learning Studio. El vínculo de Studio se proporciona en la respuesta de `invoke`, como el valor de `interactionEndpoints.Studio.endpoint`.

También puede comprobar los detalles del trabajo junto con el estado mediante la CLI.

Obtenga el nombre del trabajo de la respuesta de invocación.

```azurecli
job_name=$(az ml endpoint invoke --name mybatchedp --type batch --input-path https://pipelinedata.blob.core.windows.net/sampledata/nytaxi/taxi-tip-data.csv --query name -o tsv)
```

Use `job show` para comprobar los detalles y el estado de un trabajo de puntuación por lotes.

```azurecli
az ml job show --name $job_name
```

Transmita los registros de trabajo mediante `job stream`.

```azurecli
az ml job stream --name $job_name
```

## <a name="check-batch-scoring-results"></a>Comprobación de los resultados de la puntuación por lotes

Para ver los resultados de la puntuación:

1. Vaya a Studio.
1. Vaya a **Experimentos**.
1. Vaya a la ejecución del punto de conexión (el valor de `interactionEndpoints.Studio.endpoint` en la respuesta a `endpoint invoke`).
1. En el grafo de la ejecución, haga clic dentro del paso `batchscoring`.
1. Elija la pestaña Outputs + logs (Salidas y registros) y seleccione **Show data outputs** (Mostrar salidas de datos). 
1. Elija el icono **Ver salida**:::image type="content" source="media/how-to-use-batch-endpoint/view-data-outputs.png" alt-text="Captura de pantalla de Studio que muestra la ubicación de las salidas de los datos" lightbox="media/how-to-use-batch-endpoint/view-data-outputs.png" :::
1. En el panel emergente, copie la ruta de acceso y elija el vínculo "Open Datastore" (Abrir almacén de datos).
1. En la página de almacén de blobs resultante, pegue la ruta de acceso anterior en el cuadro de búsqueda. Encontrará las salidas de puntuación en la carpeta.
:::image type="content" source="media/how-to-use-batch-endpoint/scoring-view.gif" alt-text="Captura de pantalla que muestra cómo abrir la carpeta de puntuación y la salida de la puntuación" lightbox="media/how-to-use-batch-endpoint/scoring-view.gif":::

## <a name="add-a-deployment-to-the-batch-endpoint"></a>Adición de una implementación al punto de conexión por lotes

Un punto de conexión por lotes puede tener varias implementaciones. Cada implementación hospeda un modelo para la puntuación por lotes. 

### <a name="add-a-new-deployment"></a>Adición de una nueva implementación

Use el siguiente comando para agregar una nueva implementación a un punto de conexión por lotes existente.

```azurecli
az ml endpoint update --name mybatchedp --type batch --deployment-file cli/endpoints/batch/add-deployment.yml
```

En este ejemplo se usa un modelo que no es MLflow. Al usar un modelo de este tipo, deberá especificar el entorno y un script de puntuación en el archivo YAML:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/endpoints/batch/add-deployment.yml" :::

Más atributos de implementación para el modelo que no es MLflow:

| Clave | Descripción |
| --- | ----------- |
| code_configuration.code.local_path | El directorio que contiene todo el código fuente para puntuar el modelo. |
| code_configuration.scoring_script | El archivo de Python en el directorio anterior. Este archivo debe tener una función `init()` y una función `run()`. Use la función `init()` en el caso de cualquier preparación costosa o común. Por ejemplo, para cargar el modelo en un objeto global. Solo se llamará a esta función una vez al principio del proceso. Use `run(mini_batch)` para puntuar cada entrada; el valor de `mini_batch` es una lista de rutas de acceso de archivo. El método `run()` debe devolver un `DataFrame` de Pandas o una matriz. Estos elementos devueltos se anexan al archivo de salida común. Cada elemento de salida devuelto indica una ejecución correcta del elemento de entrada en el minilote de entrada. Asegúrese de que se incluyen datos suficientes en el resultado de la ejecución para asignar la entrada a un resultado específico de la salida de ejecución. La salida de ejecución se escribirá en el archivo de salida y no se garantiza que esté en orden, por lo que debe usar alguna clave en la salida para asignarla a la entrada correcta. |
| Environment | El entorno para puntuar el modelo en el destino de proceso. Puede definir el entorno en línea especificando el nombre, la versión y la ruta de acceso. Use `conda_file` para incluir las dependencias que se instalarán encima de `docker.image`. Use el prefijo `azureml:` para hacer referencia a un entorno existente. Por ejemplo, `azureml: mnist-env:1` apuntaría a la versión 1 de un entorno denominado `mnist-env`. |

Para revisar los detalles de la implementación, ejecute:

```azurecli
az ml endpoint show --name mybatchedp --type batch
```

### <a name="activate-the-new-deployment"></a>Activación de la nueva implementación

Para la inferencia por lotes, debe enviar el 100 % de las consultas a la implementación deseada. Para establecer la implementación recién creada como destino, use:

```azurecli
az ml endpoint update --name mybatchedp --type batch --traffic mnist_deployment:100
```

Si vuelve a examinar los detalles de la implementación, verá los cambios:

```azurecli
az ml endpoint show --name mybatchedp --type batch
```

Ahora puede invocar un trabajo de puntuación por lotes con esta nueva implementación:

```azurecli
az ml endpoint invoke --name mybatchedp --type batch --input-path https://pipelinedata.blob.core.windows.net/sampledata/mnist --mini-batch-size 10 --instance-count 2
```

## <a name="start-a-batch-scoring-job-using-rest"></a>Inicio de un trabajo de puntuación por lotes mediante REST

Los puntos de conexión por lotes tienen identificadores URI de puntuación para el acceso REST. REST permite usar cualquier biblioteca HTTP en cualquier plataforma para iniciar un trabajo de puntuación por lotes.

1. Obtenga el valor de `scoring_uri`:  

```azurecli
scoring_uri=$(az ml endpoint show --name mybatchedp --type batch --query scoring_uri -o tsv)
```

2. Obtenga el token de acceso:

```azurecli
auth_token=$(az account get-access-token --resource https://ml.azure.com --query accessToken -o tsv)
```

3. Use `scoring_uri`, el token de acceso y los datos JSON para publicar una solicitud e iniciar un trabajo de puntuación por lotes:

```bash
curl --location --request POST '$scoring_uri' --header "Authorization: Bearer $auth_token" --header 'Content-Type: application/json' --data-raw '{
"properties": {
  "dataset": {
    "dataInputType": "DataUrl",
    "Path": "https://pipelinedata.blob.core.windows.net/sampledata/mnist"
    }
  }
}'
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a usar los recursos creados, elimínelos para no incurrir en cargos:

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
1. En la lista de grupos de recursos, seleccione el que ha creado.
1. Seleccione **Eliminar grupo de recursos**.
1. Escriba el nombre del grupo de recursos. A continuación, seleccione **Eliminar**.

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear y llamar a puntos de conexión por lotes, lo que le permite puntuar grandes cantidades de datos. Consulte estos otros artículos para obtener más información sobre Azure Machine Learning:

* [Solución de problemas de puntos de conexión por lotes](how-to-troubleshoot-batch-endpoints.md)
