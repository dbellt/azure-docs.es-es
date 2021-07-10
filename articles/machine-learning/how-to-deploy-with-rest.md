---
title: Implementación de modelos mediante puntos de conexión en línea administrados con API REST (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a implementar modelos mediante puntos de conexión en línea administrados con API REST.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: rsethur
ms.author: seramasu
ms.date: 05/25/2021
ms.reviewer: laobri
ms.openlocfilehash: b8162a9770aea9d8cb3d1220f0ab81169151f781
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747606"
---
# <a name="deploy-models-with-rest-preview"></a>Implementación de modelos con REST (versión preliminar)

Aprenda a usar Azure Machine Learning API REST para implementar modelos (versión preliminar).

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

## <a name="set-endpoint-name"></a>Establecimiento de un nombre de punto de conexión

> [!NOTE]
> Los nombres de los puntos de conexión deben ser únicos en el nivel de región de Azure. Por ejemplo, solo puede haber un punto de conexión con el nombre my-endpoint in westus2.

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="set_endpoint_name":::

## <a name="azure-machine-learning-managed-online-endpoints"></a>Puntos de conexión en línea administrados de Azure Machine Learning
Los puntos de conexión en línea administrados (versión preliminar) permiten implementar el modelo sin tener que crear y administrar la infraestructura subyacente. En este artículo, creará un punto de conexión en línea y una implementación, y lo validará invocándolo. Pero primero tendrá que registrar los recursos necesarios para la implementación, incluidos el modelo, el código y el entorno.

Hay muchas formas de crear puntos de conexión en línea de Azure Machine Learning, [incluida la CLI de ](how-to-deploy-managed-online-endpoints.md), y visualmente con [Studio](how-to-use-managed-online-endpoint-studio.md). En el ejemplo siguiente se muestra un punto de conexión en línea administrado con la API REST.

## <a name="create-machine-learning-assets"></a>Crear recursos de aprendizaje automático

En primer lugar, configure los recursos Azure Machine Learning para configurar el trabajo.

En las siguientes llamadas de API REST, usamos `SUBSCRIPTION_ID`, `RESOURCE_GROUP`, `LOCATION` y `WORKSPACE` como marcadores de posición. Reemplace los marcadores de posición por sus propios valores. 

Las REST administrativa solicita un [token de autenticación de entidad de servicio](how-to-manage-rest.md#retrieve-a-service-principal-authentication-token). Reemplace `TOKEN` por su propio valor. Este token se puede recuperar con el siguiente comando:

```bash
TOKEN=$(az account get-access-token --query accessToken -o tsv)
```

El proveedor de servicios utiliza el argumento `api-version` para garantizar la compatibilidad. El argumento `api-version` varía de un servicio a otro. La versión de Azure Machine Learning API actual es `2021-03-01-preview`. Establezca la versión de la API como una variable para dar cabida a versiones futuras:

```bash
API_VERSION="2021-03-01-preview"
```

### <a name="get-storage-account-details"></a>Obtención de los detalles de una cuenta de almacenamiento

Para registrar el modelo y el código, primero deben cargarse en una cuenta de almacenamiento. Los detalles de la cuenta de almacenamiento están disponibles en el almacén de datos. En este ejemplo, se obtiene el almacén de datos predeterminado y la cuenta de Azure Storage del área de trabajo. Consulte el área de trabajo con una solicitud GET para obtener un archivo JSON con la información.

Puede usar la herramienta [jq](https://stedolan.github.io/jq/) para analizar el resultado de JSON y obtener los valores necesarios. También puede usar Azure Portal para buscar la misma información:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="get_storage_details":::

Obtención de la clave de almacenamiento:

```bash
AZURE_STORAGE_KEY=$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT | jq '.[0].value')
```

### <a name="upload--register-code"></a>Carga y registro del código

Ahora que tiene el almacén de datos, puede cargar el script de puntuación. Use Azure Storage CLI para cargar un blob en el contenedor predeterminado:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="upload_code":::

> [!TIP]
> Para realizar dicha carga también se pueden usar otros métodos, como Azure Portal o el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

Una vez que cargue el código, puede especificarlo con una solicitud PUT y hacer referencia al almacén de datos con `datastoreId`:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="create_code":::

### <a name="upload-and-register-model"></a>Carga y registro del modelo

De forma similar al código, cargue los archivos del modelo:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="upload_model":::

Ahora, registre el modelo:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="create_model":::

### <a name="create-environment"></a>Creación del entorno
La implementación debe ejecutarse en un entorno que tenga las dependencias necesarias. Cree el entorno con una solicitud PUT. Use una imagen de Docker desde Microsoft Container Registry. La imagen de Docker se puede configurar con `Docker` y agregar dependencias de Conda con `condaFile`.

En el siguiente fragmento de código, el contenido de un entorno de Conda (archivo YAML) se ha leído en una variable de entorno:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="create_environment":::

### <a name="create-endpoint"></a>Creación de un punto de conexión

Cree el punto de conexión en línea:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="create_endpoint":::

### <a name="create-deployment"></a>Creación de una implementación

Cree una implementación debajo del punto de conexión:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="create_deployment":::

### <a name="invoke-the-endpoint-to-score-data-with-your-model"></a>Invocación del punto de conexión para puntuar los datos con el modelo

Se necesitan el identificador URI de la puntuación y el token de acceso para invocar el punto de conexión. En primer lugar, obtenga el identificador URI de la puntuación:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="get_endpoint":::

Obtenga el token de acceso del punto de conexión:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="get_access_token":::

Ahora, utilice curl para invocar el punto de conexión:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="score_endpoint":::

### <a name="check-the-logs"></a>Comprobación de los registros

Compruebe los registros de implementación:

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="get_deployment_logs":::

### <a name="delete-the-endpoint"></a>Eliminación del punto de conexión

Si no va a usar la implementación, debe eliminarla con el siguiente comando (se elimina el punto de conexión y todas las implementaciones subyacentes):

:::code language="rest" source="~/azureml-examples-main/cli/deploy-rest.sh" id="delete_endpoint":::

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a implementar el modelo [mediante la CLI](how-to-deploy-managed-online-endpoints.md).
* Aprenda a implementar el modelo [mediante Studio](how-to-use-managed-online-endpoint-studio.md).
* Aprenda a [supervisar los puntos de conexión en línea](how-to-monitor-online-endpoints.md).
