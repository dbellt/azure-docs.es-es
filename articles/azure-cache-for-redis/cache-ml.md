---
title: Implementación de un modelo de Machine Learning en Azure Functions con Azure Cache for Redis
description: En este artículo, implementará un modelo desde Azure Machine Learning como aplicación de funciones en Azure Functions mediante una instancia de Azure Cache for Redis. Azure Cache for Redis es eficaz y escalable (cuando se empareja con un modelo de Azure Machine Learning, obtiene baja latencia y un rendimiento elevado en la aplicación).
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 09/30/2020
ms.openlocfilehash: 0541b626168fb680daa2fc5c0c14df5bc8a4ea7c
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111904182"
---
# <a name="deploy-a-machine-learning-model-to-azure-functions-with-azure-cache-for-redis"></a>Implementación de un modelo de Machine Learning en Azure Functions con Azure Cache for Redis

En este artículo, implementará un modelo desde Azure Machine Learning como aplicación de funciones en Azure Functions mediante una instancia de Azure Cache for Redis.  

Azure Cache for Redis es eficaz y escalable. Cuando se empareja con un modelo de Azure Machine Learning, obtiene baja latencia y un rendimiento elevado en la aplicación. Un par de escenarios en los que una memoria caché es beneficiosa: cuando se infieren los datos y para los resultados de la inferencia de modelos reales. En cualquiera de los dos, los metadatos o resultados se almacenan en memoria, lo que genera un aumento del rendimiento.

> [!NOTE]
> Aunque tanto Azure Machine Learning como Azure Functions están disponibles con carácter general, la posibilidad de empaquetar un modelo desde Machine Learning Service en Functions está en versión preliminar.  
>

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/).
* Un área de trabajo de Azure Machine Learning. Para más información, consulte el artículo [Crear un área de trabajo](../machine-learning/how-to-manage-workspace.md).
* [Azure CLI](/cli/azure/install-azure-cli).
* Un modelo de aprendizaje automático entrenado registrado en el área de trabajo. Si no tiene un modelo, use el [Tutorial: Entrenamiento de modelos de clasificación de imágenes](../machine-learning/tutorial-train-models-with-aml.md) para entrenar y registrar uno.

> [!IMPORTANT]
> En los fragmentos de código de este artículo se supone que se han establecido las siguientes variables:
>
> * `ws`: su área de trabajo de Azure Machine Learning.
> * `model`: el modelo registrado que se implementará.
> * `inference_config`: la configuración de inferencia del modelo.
>
> Para más información sobre la definición de estas variables, consulte [Implementación de modelos con Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md).

## <a name="create-an-azure-cache-for-redis-instance"></a>Creación de una instancia de Azure Redis Cache

Podrá implementar un modelo de Machine Learning en Azure Functions con cualquier instancia de caché Básica, Estándar o Premium. Para crear una instancia de caché, siga estos pasos.  

1. Vaya a la página principal de Azure Portal o abra el menú de barra lateral y seleccione **Crear un recurso**.

1. En la página **Nuevo**, seleccione **Base de datos** y, a continuación, seleccione **Azure Cache for Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Selección de Azure Cache for Redis.":::

1. En la página **Nueva instancia de Redis Cache**, configure las opciones de la nueva caché.

   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nombre DNS** | Escriba un nombre único global. | El nombre de caché debe ser una cadena comprendida entre 1 y 63 caracteres. La cadena puede contener números, letras o guiones. El nombre debe comenzar y terminar por un número o una letra y no puede contener guiones consecutivos. El *nombre de host* de la instancia de caché será *\<DNS name>.redis.cache.windows.net*. |
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que se creará esta nueva instancia de Azure Cache for Redis. |
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. |
   | **Ubicación** | Desplácese hacia abajo y seleccione una ubicación. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la memoria caché. |
   | **Plan de tarifa** | Desplácese hacia abajo y seleccione un [plan de tarifa](https://azure.microsoft.com/pricing/details/cache/). |  El plan de tarifa determina el tamaño, el rendimiento y las características disponibles para la memoria caché. Para más información, consulte la [introducción a Azure Redis Cache](cache-overview.md). |

1. Seleccione la pestaña **Redes** o elija el botón **Redes** situado en la parte inferior de la página.

1. En la pestaña **Redes**, seleccione el método de conectividad.

1. Seleccione la pestaña **Siguiente: Opciones avanzadas** o seleccione el botón **Siguiente: Opciones avanzadas** en la parte inferior de la página.

1. En la pestaña **Opciones avanzadas** de una instancia de caché básica o estándar, seleccione el botón de alternancia de habilitación si desea habilitar un puerto que no sea TLS.

1. En la pestaña **Opciones avanzadas** de la instancia de caché Premium, configure el puerto no TLS, la agrupación en clústeres y la persistencia de datos.

1. Seleccione el botón **Siguiente: Opciones avanzadas** o elija el botón **Siguiente: Etiquetas** situado en la parte inferior de la página.

1. Opcionalmente, en la pestaña **Etiquetas**, escriba el nombre y el valor si desea clasificar el recurso.

1. Seleccione **Revisar + crear**. Pasará a la pestaña Revisar y crear, donde Azure validará la configuración.

1. Tras aparecer el mensaje verde Validación superada, seleccione **Crear**.

La caché tarda un tiempo en crearse. Puede supervisar el progreso en la página **Información general** de Azure Cache for Redis. Cuando **Estado** se muestra como **En ejecución**, la memoria caché está lista para su uso.

## <a name="prepare-for-deployment"></a>Preparar la implementación

Antes de realizar la implementación, debe definir qué necesita para ejecutar el modelo como un servicio web. En la lista siguiente se describen los principales elementos necesarios para una implementación:

* Un __script de entrada__. Este script acepta solicitudes, puntúa la solicitud mediante el modelo y devuelve los resultados.

    > [!IMPORTANT]
    > El script de entrada es específico del modelo; debe comprender el formato de los datos de la solicitud entrante, el formato de los datos que espera el modelo y el formato de los datos que se devuelven a los clientes.
    >
    > Si los datos de la solicitud están en un formato que el modelo no puede usar, el script puede transformarlos a un formato aceptable. También puede transformar la respuesta antes de devolverla al cliente.
    >
    > Cuando se empaqueta para Functions, la entrada se trata de forma predeterminada como texto. Si está interesado en consumir los bytes sin formato de la entrada (por ejemplo, con desencadenadores de blobs), conviene usar [AMLRequest para aceptar datos sin procesar](../machine-learning/how-to-deploy-advanced-entry-script.md#binary-data).

En el caso de la función run, asegúrese de que se conecta a un punto de conexión de Redis.

```python
import json
import numpy as np
import os
import redis
from sklearn.externals import joblib

def init():
    global model
    global azrediscache
    azrediscache = redis.StrictRedis(host='<host_url>', port=6380, password="<access_key>", ssl=True)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        input = azrediscache.get(data)
        result = model.predict(input)
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Para más información sobre el script de entrada, consulte [Definición del código de puntuación.](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-entry-script)

* **Dependencias**, como scripts de asistente o paquetes de Python/Conda, necesarias para ejecutar el modelo o el script de entrada.

Estas entidades se encapsulan en una __configuración de inferencia__. La configuración de inferencia hace referencia al script de entrada y a otras dependencias.

> [!IMPORTANT]
> Al crear una configuración de inferencia para su uso con Azure Functions, debe usar un objeto [Environment](/python/api/azureml-core/azureml.core.environment%28class%29). Tenga en cuenta que si va a definir su propio entorno personalizado, debe enumerar azureml-defaults con la versión >= 1.0.45 como dependencia de PIP. Este paquete contiene la funcionalidad necesaria para hospedar el modelo como un servicio web. En el ejemplo siguiente se muestra cómo crear un objeto de entorno y cómo usarlo con una configuración de inferencia:
>
> ```python
> from azureml.core.environment import Environment
> from azureml.core.conda_dependencies import CondaDependencies
>
> # Create an environment and add conda dependencies to it
> myenv = Environment(name="myenv")
> # Enable Docker based environment
> myenv.docker.enabled = True
> # Build conda dependencies
> myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
>                                                            pip_packages=['azureml-defaults', 'redis'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](../machine-learning/how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte [Implementación de modelos con Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-inference-configuration).

> [!IMPORTANT]
> Al realizar la implementación en Functions, no es necesario crear una __configuración de implementación__.

## <a name="install-the-sdk-preview-package-for-functions-support"></a>Instalación del paquete de versión preliminar de SDK para la compatibilidad con Functions

Para compilar paquetes para Azure Functions, debe instalar el paquete de versión preliminar de SDK.

```bash
pip install azureml-contrib-functions
```

## <a name="create-the-image"></a>Crear la imagen

Para crear la imagen de Docker que se implementa en Azure Functions, use [azureml.comtrib.functions.package](/python/api/azureml-contrib-functions/azureml.contrib.functions) o la función de paquete específica del desencadenador que desea usar. El siguiente fragmento de código muestra cómo crear un paquete con un desencadenador HTTP a partir del modelo y la configuración de inferencia:

> [!NOTE]
> En el fragmento de código se da por supuesto que `model` contiene un modelo registrado y que `inference_config` contiene la configuración del entorno de inferencia. Para más información, consulte [Implementación de modelos con Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md).

```python
from azureml.contrib.functions import package
from azureml.contrib.functions import HTTP_TRIGGER
model_package = package(ws, [model], inference_config, functions_enabled=True, trigger=HTTP_TRIGGER)
model_package.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(model_package.location)
```

Si `show_output=True`, se muestra la salida del proceso de compilación de Docker. Una vez finalizado el proceso, la imagen se ha creado en Azure Container Registry para su área de trabajo. Una vez creada la imagen, se muestra la ubicación en Azure Container Registry. La ubicación devuelta tiene el formato `<acrinstance>.azurecr.io/package@sha256:<imagename>`.

> [!NOTE]
> Actualmente, el empaquetado para Functions admite desencadenadores HTTP, desencadenadores de blobs y desencadenadores de Service Bus. Para más información sobre los desencadenadores, vea [Enlaces de Azure Functions](../azure-functions/functions-bindings-storage-blob-trigger.md#blob-name-patterns).

> [!IMPORTANT]
> Guarde la información de la ubicación, ya que se usa al implementar la imagen.

## <a name="deploy-image-as-a-web-app"></a>Implementación de la imagen como una aplicación web

1. Use el siguiente comando para obtener las credenciales de inicio de sesión del registro de contenedor de Azure que contiene la imagen. Reemplace `<myacr>` por el valor devuelto anteriormente de `package.location`:

    ```azurecli-interactive
    az acr credential show --name <myacr>
    ```

    La salida de este comando es similar al siguiente documento JSON:

    ```json
    {
    "passwords": [
        {
        "name": "password",
        "value": "Iv0lRZQ9762LUJrFiffo3P4sWgk4q+nW"
        },
        {
        "name": "password2",
        "value": "=pKCxHatX96jeoYBWZLsPR6opszr==mg"
        }
    ],
    "username": "myml08024f78fd10"
    }
    ```

    Guarde el valor de __username__ y uno de los dos de __passwords__.

1. Si aún no tiene un grupo de recursos o un plan de App Service para implementar el servicio, los siguientes comandos muestran cómo crear ambos:

    ```azurecli-interactive
    az group create --name myresourcegroup --location "West Europe"
    az appservice plan create --name myplanname --resource-group myresourcegroup --sku B1 --is-linux
    ```

    En este ejemplo, se usa un plan de tarifa _Básico de Linux_ (`--sku B1`).

    > [!IMPORTANT]
    > Las imágenes que crea Azure Machine Learning usan Linux, por lo que debe usar el parámetro `--is-linux`.

1. Cree la cuenta de almacenamiento que va a usar como almacenamiento de trabajos web y obtenga la cadena de conexión correspondiente. Reemplace `<webjobStorage>` por el nombre que desea usar.

    ```azurecli-interactive
    az storage account create --name <webjobStorage> --location westeurope --resource-group myresourcegroup --sku Standard_LRS
    ```

    ```azurecli-interactive
    az storage account show-connection-string --resource-group myresourcegroup --name <webJobStorage> --query connectionString --output tsv
    ```

1. Para crear la aplicación de funciones, use el siguiente comando. Reemplace `<app-name>` por el nombre que desea usar. Reemplace `<acrinstance>` y `<imagename>` por los valores de `package.location`devueltos anteriormente. Reemplace `<webjobStorage>` por el nombre de la cuenta de almacenamiento del paso anterior:

    ```azurecli-interactive
    az functionapp create --resource-group myresourcegroup --plan myplanname --name <app-name> --deployment-container-image-name <acrinstance>.azurecr.io/package:<imagename> --storage-account <webjobStorage>
    ```

    > [!IMPORTANT]
    > Llegado este punto, la aplicación de funciones ya se ha creado, pero como no hemos proporcionado la cadena de conexión del desencadenador HTTP ni las credenciales a la instancia de Azure Container Registry que contiene la imagen, la aplicación de funciones no está activa. En los siguientes pasos, facilitaremos la información de autenticación y de cadena de conexión del registro de contenedor.

1. Use el siguiente comando para proporcionar a la aplicación de funciones las credenciales necesarias para acceder al registro de contenedor. Reemplace `<app-name>` por el nombre de la aplicación de funciones. Reemplace `<acrinstance>` y `<imagetag>` por los valores de la llamada de CLI de Azure del paso anterior. Reemplace `<username>` y `<password>` por la información de inicio de sesión de ACR recuperada anteriormente:

    ```azurecli-interactive
    az functionapp config container set --name <app-name> --resource-group myresourcegroup --docker-custom-image-name <acrinstance>.azurecr.io/package:<imagetag> --docker-registry-server-url https://<acrinstance>.azurecr.io --docker-registry-server-user <username> --docker-registry-server-password <password>
    ```

    Este comando devuelve información similar al siguiente documento JSON:

    ```json
    [
    {
        "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
        "slotSetting": false,
        "value": "false"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_URL",
        "slotSetting": false,
        "value": "https://myml08024f78fd10.azurecr.io"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_USERNAME",
        "slotSetting": false,
        "value": "myml08024f78fd10"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
        "slotSetting": false,
        "value": null
    },
    {
        "name": "DOCKER_CUSTOM_IMAGE_NAME",
        "value": "DOCKER|myml08024f78fd10.azurecr.io/package:20190827195524"
    }
    ]
    ```

En este momento, la aplicación de funciones comienza a cargar la imagen.

> [!IMPORTANT]
> Pueden transcurrir varios minutos hasta que se cargue la imagen. Puede supervisar el progreso en Azure Portal.

## <a name="test-azure-functions-http-trigger"></a>Prueba del desencadenador HTTP de Azure Functions

Ahora ejecutaremos y probaremos nuestro desencadenador HTTP de Azure Functions.

1. Vaya a la aplicación de función en Azure Portal.
1. En el desarrollador, seleccione **Código + prueba**.
1. En el lado derecho, seleccione la pestaña **Entrada**.
1. Seleccione el botón **Ejecutar** para probar el desencadenador HTTP de Azure Functions.

Ahora ha implementado correctamente un modelo desde Azure Machine Learning como aplicación de funciones mediante una instancia de Azure Cache for Redis. Obtenga más información sobre Azure Cache for Redis yendo a los vínculos de la sección siguiente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a seguir con el tutorial siguiente, puede mantener los recursos creados en esta guía de inicio rápido y volverlos a utilizar.

En caso contrario, si ya ha terminado con la guía de inicio rápido, puede eliminar los recursos de Azure creados en esta guía para evitar cargos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. Cuando elimine un grupo de recursos, todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado los recursos para hospedar este ejemplo dentro de un grupo de recursos existente que contiene recursos que desea mantener, puede eliminar cada recurso individualmente de sus hojas respectivas, en lugar de eliminar el grupo de recursos.

### <a name="to-delete-a-resource-group"></a>Para eliminar un grupo de recursos

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Grupos de recursos**.

2. En el cuadro de texto **Filtrar por nombre...**, escriba el nombre del grupo de recursos. En el grupo de recursos, en la lista de resultados, seleccione **...** y, después, **Eliminar grupo de recursos**.

Se le pedirá que confirme la eliminación del grupo de recursos. Escriba el nombre del grupo de recursos para confirmar y, después, seleccione **Eliminar**.

Transcurridos unos instantes, el grupo de recursos y todos sus recursos se eliminan.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Azure Cache for Redis](./cache-overview.md)
* Obtenga información sobre cómo configurar la aplicación de Functions en la documentación de [Functions](../azure-functions/functions-create-function-linux-custom-image.md).
* [Referencia de API](/python/api/azureml-contrib-functions/azureml.contrib.functions)
* Creación de una [aplicación de Python que use Azure Cache for Redis](./cache-python-get-started.md)
