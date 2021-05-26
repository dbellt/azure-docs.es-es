---
title: Instalación, configuración y uso de la CLI 2.0
titleSuffix: Azure Machine Learning
description: Aprenda a instalar, configurar y usar la extensión de la CLI para Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: lostmygithubaccount
ms.author: copeters
ms.date: 05/25/2021
ms.reviewer: laobri
ms.openlocfilehash: e9a5562f665ee2d52a39f296fb39d9a7139be6cb
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110383015"
---
# <a name="install-set-up-and-use-the-20-cli-preview"></a>Instalación, configuración y uso de la CLI 2.0 (versión preliminar)

La extensión `ml` de la [CLI de Azure](/cli/azure/) es la interfaz de última generación de Azure Machine Learning. Permite entrenar e implementar modelos desde la línea de comandos, e incluye características que aceleran el escalado vertical y horizontal de la ciencia de datos al tiempo que se hace un seguimiento del ciclo de vida del modelo.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Requisitos previos

- Para usar la CLI, debe tener una suscripción de Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
- Para usar los comandos de la CLI de este documento desde su **entorno local**, necesita la [CLI de Azure](/cli/azure/install-azure-cli).

    > [!TIP]
    > Si usa el [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/), la CLI es accesible a través del explorador y reside en la nube.

## <a name="installation"></a>Instalación

La nueva extensión para Machine Learning **requiere la versión `>=2.15.0` de la CLI de Azure**. Asegúrese de que se cumple este requisito:

```azurecli
az version
```

Si no es así, [actualice la CLI de Azure](/cli/azure/update-azure-cli).

Compruebe las extensiones de la CLI de Azure que ha instalado:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_extension_list":::

Asegúrese de que no haya instalada ninguna extensión en conflicto con el espacio de nombres `ml`, incluida la extensión `azure-cli-ml`:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_extension_remove":::

Ahora, instale la extensión `ml`:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_ml_install":::

Ejecute el comando de ayuda para comprobar la instalación y comprobar los subcomandos disponibles:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_ml_verify":::

Puede actualizar la extensión a la versión más reciente:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_ml_update":::

## <a name="set-up"></a>Configuración

Inicio de sesión:

```azurecli
az login
```

Si tiene acceso a varias suscripciones de Azure, puede establecer la suscripción de Azure.

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_account_set":::

Si aún no existe, cree el grupo de recursos de Azure:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_group_create":::

De igual forma para el área de trabajo de aprendizaje automático:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_ml_workspace_create":::

Los subcomandos de Machine Learning requieren los parámetros `--workspace/-w` y `--resource-group/-g`. Para evitar escribirlos repetidamente, configure los valores predeterminados:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_configure_defaults":::

> [!TIP]
> La mayoría de los ejemplos de código suponen que ha establecido un área de trabajo y un grupo de recursos predeterminados. Puede invalidar estos valores en la línea de comandos.

## <a name="hello-world"></a>Hola a todos

Para seguir, clone los ejemplos del repositorio y cambie al subdirectorio `cli`:

```azurecli-interactive
git clone https://github.com/Azure/azureml-examples --depth 1
cd azureml-examples/cli
```

Para ejecutar "Hola mundo" localmente mediante Python, consulte el ejemplo del subdirectorio `jobs`:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/hello-world.yml":::

> [!IMPORTANT]
> [Docker](https://docker.io) debe instalarse y ejecutarse localmente.

Envíe el trabajo, transmita los registros a la salida de la consola y abra la ejecución en Estudio de Azure Machine Learning:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="hello_world":::

> [!IMPORTANT]
> Esto puede tardar unos minutos en ejecutarse la primera vez, ya que la imagen de Docker se extrae localmente y se ejecuta el trabajo de Azure ML. Las ejecuciones posteriores tendrán la imagen copiada en caché localmente y se completarán más rápido.

## <a name="next-steps"></a>Pasos siguientes

- [Entrenamiento de modelos mediante la extensión de CLI para Machine Learning (versión preliminar)](how-to-train-cli.md)
- [Entrenamiento e implementación de un modelo de TensorFlow de clasificación de imágenes con la extensión Azure Machine Learning para Visual Studio Code (versión preliminar)](tutorial-train-deploy-image-classification-model-vscode.md)
