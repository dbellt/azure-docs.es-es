---
title: Acciones de GitHub para CI/CD
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo crear un flujo de trabajo de Acciones de GitHub para entrenar un modelo en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: juliakm
ms.author: jukullam
ms.date: 10/19/2020
ms.topic: how-to
ms.custom: github-actions-azure
ms.openlocfilehash: 9bc12c74043deea4e9c9ec9bcc457b0b82f09cbc
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889311"
---
# <a name="use-github-actions-with-azure-machine-learning"></a>Uso de Acciones de GitHub con Azure Machine Learning

Introducción a [Acciones de GitHub](https://docs.github.com/en/actions) para entrenar un modelo en Azure Machine Learning. 

> [!NOTE]
> Las Acciones de GitHub para Azure Machine Learning se proporcionan tal cual y no son totalmente compatibles con Microsoft. Si encuentra problemas con una acción específica, abra un problema en el repositorio para la acción. Por ejemplo, si se produce un problema con la acción aml-deploy, informe del problema en el repositorio [https://github.com/Azure/aml-deploy]( https://github.com/Azure/aml-deploy).

## <a name="prerequisites"></a>Prerrequisitos 

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join).  

## <a name="workflow-file-overview"></a>Información general sobre el archivo de flujo de trabajo

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

El archivo tiene cuatro secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Defina una entidad de servicio. <br /> 2. Cree un secreto de GitHub. |
|**Conexión** | 1. Conéctese al área de trabajo de Machine Learning. <br /> 2. Conéctese a un destino de proceso. |
|**Ejecutar** | 1. Envíe una ejecución de aprendizaje. |
|**Implementar** | 1. Registre el modelo en el registro de Azure Machine Learning. 1. Se implementa el modelo. |

## <a name="create-repository"></a>Crear repositorio

Cree un nuevo repositorio de las [operaciones de ML con Acciones de GitHub y la plantilla de Azure Machine Learning](https://github.com/machine-learning-apps/ml-template-azure). 

1. Abra la [plantilla](https://github.com/machine-learning-apps/ml-template-azure) en GitHub. 
2. Seleccione **Usar esta plantilla**. 

    :::image type="content" source="media/how-to-github-actions-machine-learning/gh-actions-use-template.png" alt-text="Selección de Usar esta plantilla":::
3. Cree un nuevo repositorio a partir de la plantilla. Establezca el nombre del repositorio en `ml-learning` o en el nombre de su elección. 


## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

```azurecli-interactive
az ad sp create-for-rbac --name "myML" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name> \
                            --sdk-auth
```

En este ejemplo, reemplace los marcadores de posición por su identificador de suscripción, el nombre del grupo de recursos y el nombre de la aplicación. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación App Service similar al siguiente. Copie este objeto JSON para más adelante.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

## <a name="configure-the-github-secret"></a>Configuración del secreto de GitHub

1. En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

2. Pegue la salida JSON completa del comando de la CLI de Azure en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_CREDENTIALS`.

## <a name="connect-to-the-workspace"></a>Conexión a un área de trabajo

Use la [acción de área de trabajo de Azure Machine Learning](https://github.com/marketplace/actions/azure-machine-learning-workspace) para conectarse al área de trabajo de Azure Machine Learning. 

```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

De manera predeterminada, la acción espera un archivo `workspace.json`. Si el archivo JSON tiene un nombre diferente, puede especificarlo con el parámetro de entrada `parameters_file`. Si no hay ningún archivo, se creará uno nuevo con el nombre del repositorio.


```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          parameters_file: "alternate_workspace.json"
```
La acción escribe las propiedades del área de trabajo Azure Resource Manager (ARM) en un archivo de configuración, que se seleccionará en todas las futuras Acciones de GitHub de Azure Machine Learning. El archivo se guarda en `GITHUB_WORKSPACE/aml_arm_config.json`. 

## <a name="connect-to-a-compute-target-in-azure-machine-learning"></a>Conexión a un destino de proceso en Azure Machine Learning

Use la [acción Proceso de Azure Machine Learning](https://github.com/Azure/aml-compute) para conectarse a un destino de proceso en Azure Machine Learning.  Si el destino de proceso existe, la acción se conectará a él. De lo contrario, la acción creará un nuevo destino de proceso. La [acción de proceso de AML](https://github.com/Azure/aml-compute) solo es compatible con el clúster de proceso de Azure ML y Azure Kubernetes Service (AKS). 

```yaml
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```
## <a name="submit-training-run"></a>Envío de una ejecución de entrenamiento

Use la [acción de entrenamiento de Azure Machine Learning](https://github.com/Azure/aml-run) para enviar un ScriptRun, un estimador o una canalización a Azure Machine Learning. 

```yaml
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

## <a name="register-model-in-registry"></a>Registro del modelo en el registro

Use la [acción de registro de modelo de Azure Machine Learning](https://github.com/Azure/aml-registermodel) para registrar un modelo en Azure Machine Learning.

```yaml
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}
```

## <a name="deploy-model-to-azure-machine-learning-to-aci"></a>Implementación de un modelo en Azure Machine Learning en ACI

Use la [acción de implementación de Azure Machine Learning](https://github.com/Azure/aml-deploy) para implementar un modelo y crear un punto de conexión para el modelo. También puede usar la implementación de Azure Machine Learning para implementar Azure Kubernetes Service. Vea [este flujo de trabajo de ejemplo](https://github.com/Azure-Samples/mlops-enterprise-template) para un modelo que se implementa en Azure Kubernetes Service.

```yaml
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="complete-example"></a>Ejemplo completo

Entrene el modelo e impleméntelo en Azure Machine Learning. 

```yaml
# Actions train a model on Azure Machine Learning
name: Azure Machine Learning training and deployment
on:
  push:
    branches:
      - master
    # paths:
    #   - 'code/*'
jobs:
  train:
    runs-on: ubuntu-latest
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Check Out Repository
      id: checkout_repository
      uses: actions/checkout@v2
        
    # Connect or Create the Azure Machine Learning Workspace
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Connect or Create a Compute Target in Azure Machine Learning
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Submit a training run to the Azure Machine Learning
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}

    # Register model in Azure Machine Learning model registry
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}

    # Deploy model in Azure Machine Learning to ACI
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el repositorio y el grupo de recursos ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos y el repositorio de GitHub. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](./how-to-create-machine-learning-pipelines.md)