---
title: Implementación de un modelo de Machine Learning con un punto de conexión en línea administrado
titleSuffix: Azure Machine Learning
description: Aprenda a implementar el modelo de aprendizaje automático como un servicio web administrado automáticamente por Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: seramasu
ms.reviewer: laobri
author: rsethur
ms.date: 05/13/2021
ms.topic: how-to
ms.custom: how-to
ms.openlocfilehash: 8c14523d1d566086eff73693d6500947ccda7ba4
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382989"
---
# <a name="deploy-and-score-a-machine-learning-model-with-a-managed-online-endpoint-preview"></a>Implementación y puntuación de un modelo de aprendizaje automático con un punto de conexión en línea administrado (versión preliminar)

Los puntos de conexión en línea administrados (versión preliminar) ofrecen la posibilidad de implementar el modelo sin tener que crear y administrar la infraestructura subyacente. En este artículo, comenzará por implementar un modelo en la máquina local para depurar los errores y, después, lo implementará y probará en Azure. También aprenderá a ver los registros y a supervisar el Acuerdo de Nivel de Servicio (SLA). Empezará con un modelo y terminará con un punto de conexión HTTPS/REST escalable que se puede usar para la puntuación en línea o en tiempo real.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Requisitos previos

* Para usar Azure Machine Learning, debe tener una suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Debe instalar y configurar la CLI de Azure y la extensión de Machine Learning. Para más información, consulte [Instalación, configuración y uso de la CLI 2.0 (versión preliminar)](how-to-configure-cli.md). 

* Debe tener un grupo de recursos de Azure en el que el usuario (o la entidad de servicio que use) tenga acceso de `Contributor`. Tendrá un grupo de recursos de este tipo si configuró la extensión de Machine Learning según el artículo anterior. 

* Debe tener un área de trabajo de Azure Machine Learning. Tendrá este tipo de área de trabajo si configuró la extensión de ML según el artículo anterior.

* Si aún no ha establecido los valores predeterminados de la CLI de Azure, debe guardar la configuración predeterminada. Para evitar tener que pasar repetidamente los valores, ejecute:

   ```azurecli
   az account set --subscription <subscription id>
   az configure --defaults workspace=<azureml workspace name> group=<resource group>

* [Optional] To deploy locally, you must have [Docker engine](https://docs.docker.com/engine/install/) running locally. This step is **highly recommended**. It will help you debug issues.

## Prepare your system

To follow along with the article, clone the samples repository, and navigate to the right directory by running the following commands:

```azurecli
git clone https://github.com/Azure/azureml-examples
cd azureml-examples
cd cli
```

Establezca el nombre del punto de conexión (cambie el nombre siguiente `YOUR_ENDPOINT_NAME` por un nombre único). El siguiente comando es para entornos Unix:

```azurecli
export ENDPOINT_NAME=YOUR_ENDPOINT_NAME
```

Si usa un sistema operativo Windows, utilice este comando `set ENDPOINT_NAME=YOUR_ENDPOINT_NAME` en su lugar.

> [!NOTE]
> Los nombres de los puntos de conexión deben ser únicos en el nivel de región de Azure. Por ejemplo, solo puede haber un punto de conexión con el nombre `my-endpoint` en `westus2`. 

## <a name="define-the-endpoint-configuration"></a>Definición de la configuración del punto de conexión

Las entradas necesarias para implementar un modelo en un punto de conexión en línea son las siguientes:

- Archivos de modelo (o el nombre y la versión de un modelo ya registrado en el área de trabajo). En el ejemplo, tenemos un modelo `scikit-learn` que realiza la regresión.
- El código necesario para puntuar el modelo. En este caso, tenemos un archivo `score.py`.
- Entorno en el que se ejecuta el modelo (como verá, el entorno puede ser una imagen de Docker con dependencias de Conda o puede ser un Dockerfile).
- Configuración para especificar el tipo de instancia y la capacidad de escalado.

El fragmento de código siguiente muestra el archivo `endpoints/online/managed/simple-flow/1-create-endpoint-with-blue.yml` que captura toda la información anterior: 

:::code language="yaml" source="~/azureml-examples-main/cli/endpoints/online/managed/simple-flow/1-create-endpoint-with-blue.yml":::

> [!Note]
> Como [referencia](https://azuremlschemas.azureedge.net/latest/managedOnlineEndpoint.template.yaml), hay disponible un archivo YAML de ejemplo completamente especificado para los puntos de conexión en línea administrados.

A continuación, se muestra la referencia para el formato YAML del punto de conexión. Para saber cómo especificar estos atributos, consulte el ejemplo de YAML de este artículo o el ejemplo de YAML totalmente especificado mencionado en la nota anterior. Para más información sobre los límites relacionados con los puntos de conexión administrados, consulte [Administración y aumento de las cuotas de recursos con Azure Machine Learning](how-to-manage-quotas.md).

| Clave | Descripción |
| --- | --- |
| $schema    | [Opcional] El esquema de YAML. Puede ver el esquema del ejemplo anterior en un explorador para examinar todas las opciones disponibles en el archivo YAML.|
| name       | Nombre del punto de conexión. Debe ser único en el nivel de región de Azure.|
| traffic | Porcentaje de tráfico desde el punto de conexión que se desviará a cada implementación. Los valores de tráfico deben sumar 100. |
| auth_mode | Use `key` para la autenticación basada en claves y `aml_token` para la autenticación basada en tokens de Azure Machine Learning. `key` no expira, pero `aml_token` sí lo hace. Obtenga el token más reciente con el comando `az ml endpoint list-keys`. |
| deployments | Contiene una lista de implementaciones que se crearán en el punto de conexión. En este caso, solo tenemos una implementación, denominada `blue`. Para más información sobre varias implementaciones, consulte [Implementación segura para puntos de conexión en línea (versión preliminar)](how-to-safely-rollout-managed-endpoints.md).|

Atributos de `deployment`:

| Clave | Descripción |
| --- | --- |
| name  | Nombre de la implementación. |
| model | En este ejemplo, se especifican las propiedades del modelo en línea: `name`, `version` y `local_path`. Los archivos de modelo se cargarán y registrarán automáticamente. Una desventaja de la especificación en línea es que debe incrementar la versión manualmente si quiere actualizar los archivos del modelo. Lea el apartado **Sugerencia** de la sección siguiente para ver los procedimientos recomendados relacionados. |
| code_configuration.code.local_path | Directorio que contiene todo el código fuente de Python para puntuar el modelo. Se admiten paquetes y directorios anidados. |
| code_configuration.scoring_script | El archivo de Python del directorio de puntuación anterior. Este código de Python debe tener una función `init()` y una función `run()`. Se llamará a la función `init()` después de crear o actualizar el modelo (puede usarla para almacenar en caché el modelo en memoria, etc.). Se llama a la función `run()` en cada invocación del punto de conexión para realizar la puntuación o predicción reales. |
| Environment | Contiene los detalles del entorno para hospedar el modelo y el código. En este ejemplo, tenemos definiciones en línea que incluyen `name` `version` y `path`. En este ejemplo, se usará como imagen `environment.docker.image` y se instalarán las dependencias `conda_file` encima de ella. Para más información, consulte el apartado **Sugerencia** en la sección siguiente. |
| instance_type | La SKU de máquina virtual que hospedará las instancias de implementación. Para más información, consulte [Las SKU de máquina virtual compatibles con puntos de conexión en línea administrados](reference-managed-online-endpoints-vm-sku-list.md). |
| scale_settings.scale_type | De momento, este valor debe ser `manual`. Para realizar el escalado o la reducción vertical después de crear el punto de conexión y la implementación, actualice `instance_count` en el archivo YAML y ejecute el comando `az ml endpoint update -n $ENDPOINT_NAME --file <yaml filepath>`.|
| scale_settings.instance_count | Número de instancias de la implementación. Base el valor en la carga de trabajo esperada. Para conseguir alta disponibilidad, Microsoft recomienda establecerlo en al menos `3`. |

> [!Note]
> Para usar Azure Kubernetes Service (AKS) como destino de proceso en lugar de puntos de conexión administrados:
> 1. Cree el clúster de AKS y asócielo como destino de proceso al área de Azure Machine Learning [mediante Azure ML Studio](how-to-create-attach-compute-studio.md#whats-a-compute-target).
> 2. Use este [archivo YAML de punto de conexión](https://github.com/Azure/azureml-examples/blob/main/cli/endpoints/online/aks/simple-flow/1-create-aks-endpoint-with-blue.yml) para utilizar AKS como destino en lugar del archivo YAML del punto de conexión administrado anterior. Tendrá que editar el archivo YAML para cambiar el valor de `target` por el nombre del destino de proceso registrado.
> Los comandos de este artículo, excepto la supervisión opcional del Acuerdo de Nivel de Servicio y la integración de Log Analytics, son intercambiables entre los puntos de conexión administrados y de AKS.

### <a name="registering-your-model-and-environment-separately"></a>Registro del modelo y el entorno por separado

 En este ejemplo, se especifican las propiedades del modelo y del entorno en línea `name`, `version` y `local_path` desde las que se cargan los archivos. En segundo plano, la CLI cargará los archivos y registrará automáticamente el modelo y el entorno. Como procedimiento recomendado en producción, debe registrar por separado el modelo y el entorno y especificar el nombre y la versión registrados en el archivo YAML. El formato es `model:azureml:my-model:1` o `environment:azureml:my-env:1`.

 Para realizar el registro, puede extraer las definiciones de YAML de `model` y `environment` en archivos YAML diferentes y usar los comandos `az ml model create` y `az ml environment create`. Para más información sobre estos comandos, ejecute `az ml model create -h` y `az ml environment create -h`.

### <a name="using-different-cpu--gpu-instance-types"></a>Uso de diferentes tipos de instancia de CPU y GPU

El archivo YAML anterior usa un tipo de uso general (`Standard_F2s_v2`) y una imagen de Docker que no es GPU (en el archivo YAML, consulte el atributo `image`). Para el proceso de GPU, debe elegir una SKU de tipo de proceso de GPU y una imagen de Docker de GPU.

Puede ver los tipos de instancia de GPU y de uso general admitidos en [Las SKU de máquina virtual compatibles con los puntos de conexión en línea administrados](reference-managed-online-endpoints-vm-sku-list.md). Se puede encontrar una lista de imágenes base de CPU y GPU de Azure ML en [Imágenes base de Azure Machine Learning](https://github.com/Azure/AzureML-Containers).

### <a name="using-more-than-one-model"></a>Uso de más de un modelo

Actualmente, solo puede especificar un modelo por implementación en el archivo YAML. Si tiene más de un modelo, puede solucionar esta limitación: al registrar el modelo, copie todos los modelos (como archivos o subdirectorios) en una carpeta que use para el registro. En el script de puntuación, puede usar la variable de entorno `AZUREML_MODEL_DIR` para obtener la ruta de acceso a la carpeta raíz del modelo; se conserva la estructura de directorios subyacente.

## <a name="understand-the-scoring-script"></a>Descripción del script de puntuación

> [!Tip]
> El formato del script de puntuación para los puntos de conexión en línea administrados es el mismo que se usaba en la versión anterior de la CLI y en el SDK de Python.

Como se hace referencia en el archivo YAML anterior, `code_configuration.scoring_script` debe tener una función `init()` y una función `run()`. En este ejemplo se usa este [archivo score.py](https://github.com/Azure/azureml-examples/blob/main/cli/endpoints/online/model-1/onlinescoring/score.py). Se llama a la función `init()` cuando se inicializa o se inicia el contenedor. Esta inicialización suele producirse poco después de crear o actualizar la implementación. Escriba aquí la lógica para realizar operaciones de inicialización globales, como almacenar en caché el modelo en memoria (como se hace en este ejemplo). Se llama a la función `run()` en cada invocación del punto de conexión y debe realizar la puntuación o predicción reales. En el ejemplo, extraemos los datos de la entrada JSON, llamamos al método `predict()` del modelo `scikit-learn` y devolvemos el resultado.

## <a name="deploy-and-debug-locally-using-local-endpoints"></a>Implementación y depuración locales mediante puntos de conexión locales

Para ahorrar tiempo en la depuración, se **recomienda encarecidamente** que lleve a cabo una serie de pruebas del punto de conexión localmente.

> [!Note]
> * Para realizar la implementación localmente, debe tener instalado el [motor de Docker](https://docs.docker.com/engine/install/).
> * El motor de Docker debe estar en ejecución. Normalmente, el motor se inicia al arrancar; si no es así, puede solucionar los [problemas aquí](https://docs.docker.com/config/daemon/#start-the-daemon-manually).

> [!Important]
> El objetivo de la implementación de un punto de conexión local es validar y depurar el código y la configuración antes de implementarlo en Azure. La implementación local tiene las siguientes limitaciones:
> - Los puntos de conexión locales **no** admiten reglas de tráfico, autenticación, configuración de escalado o configuración de sondeo. 
> - Los puntos de conexión locales solo admiten una implementación por punto de conexión.
> - Actualmente, la implementación local requiere la especificación en línea del modelo y el entorno (como en el archivo YAML de ejemplo). Es decir, en una implementación local no puede usar una referencia a un modelo o a un entorno registrados en el área de trabajo de Azure Machine Learning. 

### <a name="deploy-the-model-locally"></a>Implementación del modelo localmente

Para implementar el modelo localmente, ejecute el siguiente comando:

```azurecli
az ml endpoint create --local -n $ENDPOINT_NAME -f endpoints/online/managed/simple-flow/1-create-endpoint-with-blue.yml
```

La marca `--local` indica a la CLI que implemente el punto de conexión en el entorno de Docker.

>[!NOTE]
>Si usa un sistema operativo Windows, utilice `%ENDPOINT_NAME%` en lugar de `$ENDPOINT_NAME` aquí y en los sucesivos comandos.

### <a name="check-if-the-local-deployment-succeeded"></a>Comprobación de que la implementación local se ha realizado correctamente

Consulte los registros para comprobar si el modelo se implementó sin errores:

```azurecli
az ml endpoint get-logs --local -n $ENDPOINT_NAME --deployment blue
```

### <a name="invoke-the-local-endpoint-to-score-data-with-your-model"></a>Invocación del punto de conexión local para puntuar los datos con el modelo

Invoque el punto de conexión para puntuar el modelo usando el práctico comando `invoke` y pasando los parámetros de consulta almacenados en un archivo JSON:

```azurecli
az ml endpoint invoke --local -n $ENDPOINT_NAME --request-file endpoints/online/model-1/sample-request.json
```

Si quiere usar un cliente REST (como Curl), necesita el URI de puntuación. Se puede obtener mediante el comando `az ml endpoint show --local -n $ENDPOINT_NAME`. En los datos devueltos, encontrará un atributo denominado `scoring_uri`. 

### <a name="review-the-logs-for-output-from-the-invoke-operation"></a>Revisión de los registros para obtener la salida de la operación de invocación

En el archivo `score.py` de ejemplo, el método `run()` registra una salida en la consola. Puede ver esta salida usando de nuevo el comando `get-logs`:

```azurecli
az ml endpoint get-logs --local -n $ENDPOINT_NAME --deployment blue
```

##  <a name="deploy-your-managed-online-endpoint-to-azure"></a>Implementación del punto de conexión en línea administrado en Azure 

### <a name="deploy-to-azure"></a>Implementar en Azure

Para implementar la configuración de YAML en la nube, ejecute el siguiente comando:

::: code language="azurecli" source="~/azureml-examples-main/cli/how-to-deploy-managed-online-endpoint.sh" ID="deploy" :::

Esta implementación puede tardar aproximadamente entre 8 y 14 minutos, dependiendo de si el entorno o la imagen subyacentes se están creando por primera vez. Las implementaciones subsiguientes que usen el mismo entorno serán más rápidas.

> [!Tip]
> Si prefiere no bloquear la consola de la CLI, puede agregar la marca `--no-wait` al comando. Sin embargo, se detendrá la presentación interactiva del estado de implementación.

> [!Tip]
> Use [Solución de problemas de implementación de puntos de conexión en línea administrados](how-to-troubleshoot-managed-online-endpoints.md) para depurar los errores.

### <a name="check-the-status-of-the-deployment"></a>Comprobación del estado de la implementación

El comando `show` contiene `provisioning_status` tanto para el punto de conexión como para la implementación:

::: code language="azurecli" source="~/azureml-examples-main/cli/how-to-deploy-managed-online-endpoint.sh" ID="get_status" :::

Puede enumerar todos los puntos de conexión del área de trabajo en un formato de tabla con el comando `list`:

```azurecli
az ml endpoint list --output table
```

### <a name="check-if-the-cloud-deployment-succeeded"></a>Comprobación de que la implementación en la nube se ha realizado correctamente

Consulte los registros para comprobar si el modelo se implementó sin errores:

```azurecli
az ml endpoint get-logs --local -n $ENDPOINT_NAME --deployment blue
```

De forma predeterminada, los registros se extraen del servidor de inferencia. Si quiere ver los registros del inicializador de almacenamiento (que monta los recursos, como el modelo y el código, en el contenedor), agregue la marca `--container storage-initializer`.

### <a name="invoke-the-endpoint-to-score-data-with-your-model"></a>Invocación del punto de conexión para puntuar los datos con el modelo

Puede usar el comando `invoke` o un cliente REST de su elección para invocar el punto de conexión y puntuar algunos datos: 

::: code language="azurecli" source="~/azureml-examples-main/cli/how-to-deploy-managed-online-endpoint.sh" ID="test_endpoint" :::

Puede usar de nuevo el comando `get-logs` mostrado anteriormente para ver los registros de invocación.

Para usar un cliente REST, necesitará el valor de `scoring_uri` y el token o la clave de autenticación. El valor de `scoring_uri` está disponible en la salida del comando `show`:
 
::: code language="azurecli" source="~/azureml-examples-main/cli/how-to-deploy-managed-online-endpoint.sh" ID="get_scoring_uri" :::

Observe cómo se usa `--query` para filtrar los atributos solo por lo que se necesitan. Puede encontrar más información sobre `--query` en [Consulta de la salida del comando de la CLI de Azure](/cli/azure/query-azure-cli).

Recupere las credenciales necesarias mediante el comando `get-credentials`:

```azurecli
az ml endpoint get-credentials -n $ENDPOINT_NAME
```

### <a name="optional-update-the-deployment"></a>[Opcional] Actualización de la implementación

Si quiere actualizar el código, el modelo, el entorno o la configuración de escalado, actualice el archivo YAML y ejecute el comando `az ml endpoint update`. 

>[!IMPORTANT]
> Solo puede modificar **un** aspecto (el tráfico, la configuración de escalado, el código, el modelo o el entorno) en un solo comando `update`. 

Para comprender cómo funciona `update`:

1. Abra el archivo `online/model-1/onlinescoring/score.py`.
1. Cambie la última línea de la función `init()`: después de `logging.info("Init complete")`, agregue `logging.info("Updated successfully")`. 
1. Guarde el archivo.
1. Ejecute el comando:
```azurecli
az ml endpoint update -n $ENDPOINT_NAME -f endpoints/online/managed/simple-flow/1-create-endpoint-with-blue.yml
```

> [!IMPORTANT]
> La actualización mediante YAML es declarativa. Es decir, los cambios en el archivo YAML se reflejarán en los recursos de Azure Resource Manager subyacentes (puntos de conexión e implementaciones). Este enfoque facilita [GitOps](https://www.atlassian.com/git/tutorials/gitops): *TODOS* los cambios en los puntos de conexión o implementaciones pasan por YAML (incluso `instance_count`). Como efecto secundario, si quita una implementación de YAML y ejecuta `az ml endpoint update` con el archivo, esa implementación se eliminará. Puede realizar actualizaciones sin usar el archivo YAML mediante la marca `--set `, como se describe en la siguiente sugerencia.

5. Dado que ha modificado la función `init()`, que se ejecuta cuando se crea o actualiza el punto de conexión, el mensaje `Updated successfully` estará en los registros. Recupere los 0 registros mediante la ejecución de:
```azurecli
az ml endpoint get-logs -n $ENDPOINT_NAME --deployment blue
```

En el caso poco frecuente de que quiera eliminar y volver a crear la implementación debido a un problema irresoluble, use:

```azurecli
az ml endpoint delete -n $ENDPOINT_NAME --deployment blue
```

El comando `update` también funciona con puntos de conexión locales. Use el mismo comando `az ml endpoint update` con la marca `--local`.

> [!Tip]
> Con el comando `az ml endpoint update`, puede usar el parámetro [`--set` disponible en la CLI de Azure](/cli/azure/use-cli-effectively#generic-update-arguments) para invalidar los atributos del archivo YAML **o** para establecer atributos específicos sin pasar este archivo. El uso de `--set` para atributos únicos es especialmente valioso en escenarios de desarrollo y pruebas. Por ejemplo, para escalar verticalmente el valor de `instance_count` de la primera implementación, podría usar la marca `--set deployments[0].scale_settings.instance_count=2`. Sin embargo, dado que YAML no está actualizado, esta técnica no facilita [GitOps](https://www.atlassian.com/git/tutorials/gitops).

### <a name="optional-monitor-sla-using-azure-monitor"></a>[Opcional] Supervisión del Acuerdo de Nivel de Servicio mediante Azure Monitor

Puede ver las métricas y establecer alertas en función del Acuerdo de Nivel de Servicio siguiendo las instrucciones que se indican en [Supervisión de puntos de conexión en línea administrados](how-to-monitor-online-endpoints.md).

### <a name="optional-integrate-with-log-analytics"></a>[Opcional] Integración con Log Analytics

El comando `get-logs` solo proporcionará los últimos cientos de líneas de registros de una instancia seleccionada automáticamente. Sin embargo, Log Analytics proporciona una manera de almacenar y analizar los registros de forma duradera. En primer lugar, siga los pasos que se indican en [Creación de un área de trabajo de Log Analytics en Azure Portal](/azure/azure-monitor/logs/quick-create-workspace#create-a-workspace) para crear un área de trabajo de Log Analytics.

Luego, en Azure Portal:

1. Vaya al grupo de recursos.
1. Elija el punto de conexión.
1. Seleccione la **página de recursos de ARM**.
1. Seleccione **Configuración de diagnóstico**.
1. Seleccione **Add settings** (Agregar configuración): habilite el envío de registros de la consola al área de trabajo de Log Analytics.

Tenga en cuenta que los registros pueden tardar hasta una hora en conectarse. Envíe algunas solicitudes de puntuación después de este período y, luego, compruebe los registros mediante los pasos siguientes:

1. Abra el área de trabajo de Log Analytics. 
1. En el panel de navegación izquierdo, seleccione **Registros**.
1. Cierre el menú emergente **Consultas** que se abre automáticamente.
1. Haga doble clic en **AmlOnlineEndpointConsoleLog**.
1. Seleccione *Ejecutar*.

## <a name="delete-the-endpoint-and-deployment"></a>Eliminación del punto de conexión y la implementación

Si no va a usar la implementación, debe eliminarla con el siguiente comando (se elimina el punto de conexión y todas las implementaciones subyacentes):

::: code language="azurecli" source="~/azureml-examples-main/cli/how-to-deploy-managed-online-endpoint.sh" ID="delete_endpoint" :::

## <a name="next-steps"></a>Pasos siguientes

- [Implementación segura para puntos de conexión en línea (versión preliminar)](how-to-safely-rollout-managed-endpoints.md)|
- [Solución de problemas de implementación de puntos de conexión en línea administrados](how-to-troubleshoot-managed-online-endpoints.md)