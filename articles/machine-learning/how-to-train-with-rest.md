---
title: Entrenamiento de modelos con REST (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar modelos y a crear trabajos con API REST.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: wenxwei
ms.author: wenxwei
ms.date: 05/25/2021
ms.reviewer: peterlu
ms.openlocfilehash: fadcb4c8b24e9c0dcc1cfe39529af4a1c75cc36c
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110383027"
---
# <a name="train-models-with-rest-preview"></a>Entrenamiento de modelos con REST (versión preliminar)

Aprenda a usar la Azure Machine Learning API REST para crear y administrar trabajos de entrenamiento (versión preliminar).

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

La API REST utiliza verbos HTTP estándar para crear, recuperar, actualizar y eliminar recursos. La API REST funciona con cualquier lenguaje o herramienta que pueda hacer solicitudes HTTP. La estructura sencilla de REST hace que sea una buena opción en entornos de scripting y en la automatización de MLOps.

En este artículo, aprenderá a usar las nuevas API REST para:

> [!div class="checklist"]
> * Crear recursos de aprendizaje automático
> * Crear un trabajo de entrenamiento básico 
> * Crear un trabajo de barrido de ajuste de hiperparámetros

## <a name="prerequisites"></a>Prerequisites

- Una **suscripción de Azure** en la que tenga derechos administrativos Si no tiene este tipo de suscripción, pruebe la [suscripción personal gratis o de pago](https://aka.ms/AMLFree).
- Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Una entidad de servicio en el área de trabajo. Las solicitudes administrativas de REST usan la [autenticación de entidad de servicio](how-to-setup-authentication.md#use-service-principal-authentication).
- Un token de autenticación de entidad de servicio. Para recuperar este token, siga los pasos que se describen en la sección [Recuperación de un token de autenticación de entidad de servicio](./how-to-manage-rest.md#retrieve-a-service-principal-authentication-token). 
- La utilidad **curl**. El programa **curl** está disponible en el [subsistema de Windows para Linux](/windows/wsl/install-win10) o en cualquier distribución de UNIX. En PowerShell, **curl** es un alias de **Invoke-WebRequest** y `curl -d "key=val" -X POST uri` se convierte en `Invoke-WebRequest -Body "key=val" -Method POST -Uri uri`. 

## <a name="azure-machine-learning-jobs"></a>Trabajos de Azure Machine Learning
Un trabajo es un recurso que especifica todos los aspectos de un trabajo de cálculo. Agrega tres cosas:

- ¿Qué se ejecuta?
- ¿Cómo se ejecuta?
- ¿Dónde se ejecuta?

Hay muchas formas de enviar un trabajo de Azure Machine Learning, entre las que se encuentran el SDK, la CLI y visualmente con Studio. En el ejemplo siguiente se envía un trabajo de entrenamiento de LightGBM con la API REST.

## <a name="create-machine-learning-assets"></a>Crear recursos de aprendizaje automático

En primer lugar, configure los recursos Azure Machine Learning para configurar el trabajo.

En las siguientes llamadas de API REST, usamos `$SUBSCRIPTION_ID`, `$RESOURCE_GROUP`, `$LOCATION` y `$WORKSPACE` como marcadores de posición. Reemplace los marcadores de posición por sus propios valores. 

Las REST administrativa solicita un [token de autenticación de entidad de servicio](how-to-manage-rest.md#retrieve-a-service-principal-authentication-token). Reemplace `$TOKEN` por su propio valor. Este token se puede recuperar con el siguiente comando:

```bash
TOKEN=$(az account get-access-token --query accessToken -o tsv)
```

El proveedor de servicios utiliza el argumento `api-version` para garantizar la compatibilidad. El argumento `api-version` varía de un servicio a otro. La versión de Azure Machine Learning API actual es `2021-03-01-preview`. Establezca la versión de la API como una variable para dar cabida a versiones futuras:

```bash
API_VERSION="2021-03-01-preview"
```

### <a name="compute"></a>Compute

La ejecución de trabajos de aprendizaje automático requiere recursos de proceso. Puede enumerar los recursos de proceso del área de trabajo:

```bash
curl "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.MachineLearningServices/workspaces/$WORKSPACE/computes?api-version=$API_VERSION \
--header "Authorization: Bearer $TOKEN"
```

En este ejemplo, se usa un clúster de proceso existente denominado `cpu-cluster`. Establecemos el nombre de proceso como variable para la encapsulación:

```bash
COMPUTE_NAME="cpu-cluster"
```

> [!TIP]
> Puede [crear o sobrescribir un recurso de proceso con nombre con una solicitud PUT](./how-to-manage-rest.md#create-and-modify-resources-using-put-and-post-requests). 

### <a name="environment"></a>Entorno 

El ejemplo de LightGBM debe ejecutarse en un entorno de LightGBM. Cree el entorno con una solicitud PUT. Use una imagen de Docker desde Microsoft Container Registry.

La imagen de Docker se puede configurar con `Docker` y agregar dependencias de Conda con `condaFile`: 

:::code language="rest" source="~/azureml-examples-cli-preview/cli/how-to-train-rest.sh" id="create_environment":::

### <a name="datastore"></a>Almacén de datos

El trabajo de entrenamiento debe ejecutarse en datos, por lo que debe especificar un almacén de datos. En este ejemplo, se obtiene el almacén de datos predeterminado y la cuenta de Azure Storage del área de trabajo. Consulte el área de trabajo con una solicitud GET para devolver un archivo JSON con la información.

Puede usar la herramienta [jq](https://stedolan.github.io/jq/) para analizar el resultado de JSON y obtener los valores necesarios. También puede usar Azure Portal para buscar la misma información.

```bash
response=$(curl --location --request GET "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.MachineLearningServices/workspaces/$WORKSPACE/datastores?api-version=$API_VERSION&isDefault=true" \
--header "Authorization: Bearer $TOKEN")

AZURE_STORAGE_ACCOUNT=$(echo $response | jq '.value[0].properties.contents.accountName')
AZUREML_DEFAULT_DATASTORE=$(echo $response | jq '.value[0].name')
AZUREML_DEFAULT_CONTAINER=$(echo $response | jq '.value[0].properties.contents.containerName')
AZURE_STORAGE_KEY=$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT | jq '.[0].value')
```

### <a name="data"></a>data

Ahora que tiene el almacén de datos, puede crear un conjunto de datos. En este ejemplo, use el conjunto de datos `iris.csv` común y apunte a él en `path`. 

:::code language="rest" source="~/azureml-examples-cli-preview/cli/how-to-train-rest.sh" id="create_data":::

### <a name="code"></a>Código

Ahora que tiene el conjunto de datos y el almacén de datos, puede cargar el script de entrenamiento que se ejecutará en el trabajo. Use Azure Storage CLI para cargar un blob en su contenedor predeterminado. Para realizar dicha carga también se pueden usar otros métodos, como Azure Portal o el Explorador de Azure Storage.


```bash
az storage blob upload-batch -d $AZUREML_DEFAULT_CONTAINER/src \
 -s jobs/train/lightgbm/iris/src --account-name $AZURE_STORAGE_ACCOUNT --account-key $AZURE_STORAGE_KEY
```

Una vez que cargue el código, puede especificarlo con una solicitud PUT y hacer referencia al almacén de datos con `datastoreId`. 

:::code language="rest" source="~/azureml-examples-cli-preview/cli/how-to-train-rest.sh" id="create_code":::

## <a name="submit-a-training-job"></a>Envío de un trabajo de entrenamiento

Ahora que los recursos están colocados, puede ejecutar el trabajo LightGBM, que genera un modelo entrenado y metadatos. Necesita la siguiente información para configurar el trabajo de entrenamiento: 

- **run_id**: [Opcional] el nombre del trabajo, que debe ser único en todos los trabajos. A menos que se especifique un nombre en el archivo YAML a través del campo `name` o de la línea de comandos a través de `--name/-n`, se genera y usa automáticamente un GUID o UUID para el nombre.
- **jobType**: el tipo de trabajo. En el caso de un trabajo de entrenamiento básico, use `Command`.
- **codeId**: ruta de acceso al script de entrenamiento.
- **command**: el comando que se debe ejecutar. Los datos de entrada se pueden escribir en el comando y se puede hacer referencia a ellos con el enlace de datos. 
- **environmentId**: la ruta de acceso al entorno.
- **inputDataBindings**: el enlace de datos puede ayudarle a hacer referencia a los datos de entrada. Cree una variable de entorno y el nombre del enlace se agregará a AZURE_ML_INPUT_, al que puede hacer referencia en `command`. Para crear un enlace de datos, debe agregar la ruta de acceso a los datos que creó como `dataId`. 
- **experimentName**: [Opcional] etiqueta el trabajo para ayudarle a organizar los trabajos en Estudio de Azure Machine Learning. El registro de ejecución de cada trabajo se organiza en el experimento correspondiente de la pestaña "Experiment" (Experimento) de Studio. Si se omite, el valor predeterminado de las etiquetas será el nombre del directorio de trabajo cuando se cree el trabajo.
- **compute**: `target` especifica el destino de proceso, que puede ser la ruta de acceso a su proceso. `instanceCount` especifica el número de instancias que necesita para el trabajo.

Use los siguientes comandos para enviar el trabajo de entrenamiento:

:::code language="rest" source="~/azureml-examples-cli-preview/cli/how-to-train-rest.sh" id="create_job":::

## <a name="submit-a-hyperparameter-sweep-job"></a>Envío de un trabajo de barrido de hiperparámetros

Azure Machine Learning también permite ajustar eficazmente hiperparámetros de entrenamiento. Puede crear un barrido de ajuste de hiperparámetros con las API REST. Para más información sobre las opciones de ajuste de hiperparámetros de Azure Machine Learning, consulte [Hiperparámetros que ajustan un modelo](how-to-tune-hyperparameters.md). Especifique los parámetros de ajuste de hiperparámetros para configurar el barrido:

- **jobType**: el tipo de trabajo. En el caso de un trabajo de barrido, será `Sweep`. 
- **algorithm:** el algoritmo de muestreo "aleatorio" suele ser un buen punto de partida. Vea el [esquema](https://azuremlschemas.azureedge.net/latest/sweepJob.schema.json) del trabajo de barrido para conocer la enumeración de las opciones. 
- **trial**: la configuración del trabajo de comando para cada evaluación que se va a ejecutar. 
- **objective**: `primaryMetric` es la métrica de optimización, que debe coincidir con el nombre de una métrica registrada desde el código de entrenamiento. `goal` especifica la dirección (minimizar o maximizar). Vea el [esquema](https://azuremlschemas.azureedge.net/latest/sweepJob.schema.json) para conocer la enumeración completa de las opciones. 
- **searchSpace**: diccionario de los hiperparámetros de los que se realiza un barrido. La clave es un nombre para el hiperparámetro, por ejemplo, `learning_rate`. El valor es la distribución de hiperparámetros. Vea el [esquema](https://azuremlschemas.azureedge.net/latest/sweepJob.schema.json) para conocer la enumeración de las opciones.
- **maxTotalTrials**: el número máximo de evaluaciones individuales que se ejecutan.
- **maxConcurrentTrials**: [Opcional] número máximo de evaluaciones que se ejecutan simultáneamente en el clúster de proceso.
- **timeout**: [Opcional] número máximo de minutos durante los que se ejecuta el trabajo de barrido.

Para crear un trabajo de barrido con el mismo ejemplo de LightGBM, use los siguientes comandos: 

:::code language="rest" source="~/azureml-examples-cli-preview/cli/how-to-train-rest.sh" id="create_a_sweep_job":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un modelo entrenado, aprenda [a implementarlo](how-to-deploy-and-where.md).