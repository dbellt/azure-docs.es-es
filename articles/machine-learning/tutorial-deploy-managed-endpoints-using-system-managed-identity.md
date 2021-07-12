---
title: Acceso a recursos de Azure con un punto de conexión en línea administrado
titleSuffix: Azure Machine Learning
description: Acceda de forma segura a los recursos de Azure para la implementación del modelo de Machine Learning con un punto de conexión en línea administrado y una identidad administrada asignada por el sistema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: seramasu
ms.reviewer: laobri
author: rsethur
ms.date: 05/25/2021
ms.topic: tutorial
ms.custom: tutorial
ms.openlocfilehash: 731d9a64c9ef144e8e51e9bce319a031056958ae
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071558"
---
# <a name="tutorial-access-azure-resources-with-a-managed-online-endpoint-and-system-managed-identity-preview"></a>Tutorial: Acceso a recursos de Azure con un punto de conexión en línea administrado e identidad administrada por el sistema (versión preliminar)

En este tutorial, aprenderá a acceder de forma segura a los recursos de Azure desde el script de puntuación con un punto de conexión en línea administrado y una identidad administrada asignada por el sistema.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]
En este tutorial se muestra cómo realizar las siguientes acciones con la CLI de Azure y su extensión de ML:

> [!div class="checklist"]
> * Establecimiento de los valores predeterminados que usará la CLI de Azure
> * Configuración de las variables que se usarán con el punto de conexión en línea administrado
> * Creación de una cuenta de almacenamiento de blobs y un contenedor de blobs 
> * Creación de un punto de conexión en línea administrado
> * Concesión del permiso de identidad administrada asignada por el sistema para acceder al almacenamiento
> * Creación de una implementación asociada al punto de conexión administrado


## <a name="prerequisites"></a>Requisitos previos

* Para usar Azure Machine Learning, debe tener una suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Debe instalar y configurar la CLI de Azure y la extensión de Machine Learning. Para más información, consulte [Instalación, configuración y uso de la CLI 2.0 (versión preliminar)](how-to-configure-cli.md). 

* Debe tener un grupo de recursos de Azure en el que el usuario (o la entidad de servicio que use) tenga acceso de `Contributor`. Tendrá un grupo de recursos de este tipo si configuró la extensión de Machine Learning según el artículo anterior. 

* Debe tener un área de trabajo de Azure Machine Learning. Tendrá este tipo de área de trabajo si configuró la extensión de ML según el artículo anterior.

* Un modelo de Machine Learning listo para la puntuación y la implementación.


## <a name="set-the-defaults-for-azure-cli"></a>Establecimiento de los valores predeterminados para la CLI de Azure

Para asegurarse de que se usan los recursos correctos a lo largo de este tutorial, establezca los valores predeterminados del identificador de la suscripción de Azure, el área de trabajo de Azure Machine Learning y el grupo de recursos que desea usar. Esto le permite evitar tener que pasar repetidamente los valores cada vez que llame a un comando de la CLI de Azure. 

> [!IMPORTANT]
> Asegúrese de que su cuenta de usuario tiene asignado el rol "Administrador de acceso de usuarios" en el grupo de recursos. 

```azurecli
az account set --subscription <subscription id>
az configure --defaults workspace=<azureml workspace name> group=<resource group>
```

## <a name="define-the-configuration-yaml-file-for-your-deployment"></a>Definición del archivo de configuración YAML para su implementación

Para implementar un punto de conexión administrado, primero debe definir la configuración del punto de conexión en un archivo YAML.

El siguiente ejemplo de código crea un punto de conexión administrado que:  
* Muestra los archivos YAML del directorio `endpoints/online/managed/managed-identities/`.
* Define el nombre por el que se desea hacer referencia al punto de conexión, `my-sai-endpoint`.
* Especifica el tipo de autorización que se usará para acceder al punto de conexión, `auth-mode: key`.
* Especifica que el tipo de punto de conexión que desea crear es un punto de conexión `online`. 
* Indica que el punto de conexión tiene una implementación asociada llamada `blue`.
* Configura los detalles de la implementación como, por ejemplo, qué modelo se va a implementar y qué entorno y script de puntuación se usarán.

:::code language="yaml" source="~/azureml-examples-main/cli/endpoints/online/managed/managed-identities/2-sai-deployment.yml":::


## <a name="configure-variables-for-your-deployment"></a>Configuración de variables para la implementación

Configure los nombres de las variables para el área de trabajo, la ubicación del área de trabajo y el punto de conexión que desea crear. El código siguiente exporta estos valores como variables de entorno en el punto de conexión:

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="set_variables" :::

A continuación, especifique el nombre que desea para la cuenta de almacenamiento de blobs, el contenedor de blobs y el archivo. Estos nombres de variable se definen aquí y se hace referencia a ellos en los comandos `az storage account create` y `az storage container create` de la sección siguiente.

El código siguiente exporta esos valores como variables de entorno:

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="configure_storage_names" :::


Una vez exportadas estas variables, cree un archivo de texto localmente. Cuando se implementa el punto de conexión, el script de puntuación accederá a este archivo de texto mediante la identidad administrada asignada por el sistema que se genera tras la creación del punto de conexión.

## <a name="create-blob-storage-and-container"></a>Creación de un contenedor y un almacenamiento de blobs

En este ejemplo, creará una cuenta de almacenamiento de blobs y un contenedor de blobs y, a continuación, cargará el archivo de texto creado anteriormente en el contenedor de blobs. 

Primero, cree una cuenta de almacenamiento. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="create_storage_account" :::

A continuación, cree el contenedor de blobs en la cuenta de almacenamiento. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="create_storage_container" :::

Después, cargue el archivo de texto en el contenedor de blobs. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="upload_file_to_storage" :::

## <a name="create-a-managed-online-endpoint"></a>Creación de un punto de conexión en línea administrado

El código siguiente crea un punto de conexión en línea administrado sin especificar una implementación. La creación de la implementación se realiza más adelante en el tutorial.

Al crear un punto de conexión administrado, se crea de forma predeterminada una identidad administrada asignada por el sistema para el punto de conexión.

>[!IMPORTANT]
> Las identidades administradas asignadas por el sistema son inmutables y no se pueden cambiar una vez creadas.

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="create_endpoint" :::

Compruebe el estado del punto de conexión con lo siguiente. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="check_endpoint_Status" :::


## <a name="give-storage-permission-to-system-assigned-managed-identity"></a>Concesión de permiso de almacenamiento a la identidad administrada asignada por el sistema

Puede conceder al punto de conexión administrado permiso para acceder al almacenamiento a través de su identidad administrada asignada por el sistema. 

Recupere la identidad administrada asignada por el sistema que se creó para el punto de conexión. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="get_system_identity" :::

Desde aquí, puede conceder a la identidad administrada asignada por el sistema permiso para acceder al almacenamiento.

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="give_permission_to_user_storage_account" :::

## <a name="scoring-script-to-access-azure-resource"></a>Script de puntuación para acceder al recurso de Azure

Consulte el siguiente script de puntuación para saber cómo usar el token de identidad administrada asignado por el sistema para acceder a los recursos de Azure. En este escenario, el recurso de Azure es la cuenta de almacenamiento creada en secciones anteriores. 

:::code language="python" source="~/azureml-examples-main/cli/endpoints/online/model-1/onlinescoring/score_managedidentity.py":::

## <a name="create-a-deployment-using-your-configuration"></a>Creación de una implementación mediante su configuración

Cree una implementación asociada al punto de conexión administrado.

Esta implementación puede tardar aproximadamente entre 8 y 14 minutos, dependiendo de si el entorno o la imagen subyacentes se están creando por primera vez. Las implementaciones subsiguientes que usen el mismo entorno serán más rápidas.

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="deploy" :::

Compruebe el estado de la implementación. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="check_deploy_Status" :::

> [!NOTE]
> El método init del script de puntuación lee el archivo de la cuenta de almacenamiento mediante el token de identidad administrada asignado por el sistema. 

Para comprobar la salida del método init, consulte el registro de implementación con el código siguiente. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="check_deployment_log" :::

Una vez completado este comando, habrá registrado el modelo, el entorno y el punto de conexión en el área de trabajo de Azure Machine Learning.

### <a name="confirm-your-endpoint-deployed-successfully"></a>Confirmación de que el punto de conexión se implementó correctamente

Una vez implementado el punto de conexión, confirme que funciona correctamente. Los detalles de la inferencia varían de un modelo a otro. En este tutorial, los parámetros de la consulta JSON son parecidos a los siguientes: 

:::code language="json" source="~/azureml-examples-main/cli/endpoints/online/model-1/sample-request.json" :::

Para llamar al punto de conexión, ejecute: 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="test_endpoint" :::


## <a name="delete-the-endpoint-and-storage-account"></a>Eliminación del punto de conexión y la cuenta de almacenamiento

Si no tiene previsto seguir usando el punto de conexión y el almacenamiento implementados, elimínelos para reducir los costos. Al eliminar el punto de conexión, también se eliminan todas sus implementaciones asociadas. 

::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="delete_endpoint" :::
::: code language="azurecli" source="~/azureml-examples-main/cli/deploy-managed-online-endpoint-access-resource-sai.sh" id="delete_storage_account" :::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de Azure Machine Learning, usó la CLI de Machine Learning para las tareas siguientes:

> [!div class="checklist"]
> * Establecimiento de los valores predeterminados que usará la CLI de Azure
> * Configuración de las variables que se usarán con el punto de conexión
> * Creación de una cuenta de almacenamiento de blobs y un contenedor de blobs 
> * Creación de un punto de conexión administrado
> * Concesión del permiso de identidad administrada asignada por el sistema para acceder al almacenamiento
> * Creación de una implementación asociada al punto de conexión administrado

* Para más información sobre cómo usar la CLI, consulte [Uso de la extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).
* Para refinar las consultas JSON para que solo devuelvan datos específicos, consulte [Resultados de los comandos de consulta de la CLI de Azure](/cli/azure/query-azure-cli).
* Para más información sobre el esquema de YAML, consulte el documento de [referencia de YAML del punto de conexión en línea](reference-online-endpoint-yaml.md).
