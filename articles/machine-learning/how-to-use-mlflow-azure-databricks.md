---
title: MLflow Tracking para experimentos de aprendizaje automático de Azure Databricks
titleSuffix: Azure Machine Learning
description: Configure MLflow con Azure Machine Learning para registrar métricas y artefactos de experimentos de aprendizaje automático de Azure Databricks.
services: machine-learning
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.topic: how-to
ms.custom: devx-track-python
ms.openlocfilehash: dcaa0f964b2a3c047826bb82ae8d1121e38f1796
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378862"
---
# <a name="track-azure-databricks-ml-experiments-with-mlflow-and-azure-machine-learning-preview"></a>Seguimiento de experimentos de aprendizaje automático de Azure Databricks con MLflow y Azure Machine Learning (versión preliminar)

En este artículo, aprenderá a habilitar el identificador URI de seguimiento y la API de registro de MLflow, conocidos en conjunto como [MLflow Tracking](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), para conectar los experimentos de Azure Databricks (ADB), MLflow y Azure Machine Learning.

[MLflow](https://www.mlflow.org) es una biblioteca de código abierto para administrar el ciclo de vida de los experimentos de aprendizaje automático. MLFlow Tracking es un componente de MLflow que registra las métricas de ejecución de entrenamiento y los artefactos de modelo y realiza un seguimiento de ellos. Más información sobre [Azure Databricks y MLflow](/azure/databricks/applications/mlflow/). 

Consulte [Seguimiento de ejecuciones de experimentos con MLflow y Azure Machine Learning (versión preliminar)](how-to-use-mlflow.md) para conocer otras integraciones de la funcionalidad de MLflow y Azure Machine Learning.

Si tiene un proyecto de MLflow para entrenar con Azure Machine Learning, consulte [Entrenamiento de modelos de Machine Learning con MLflow y Azure Machine Learning (versión preliminar)](how-to-train-mlflow-projects.md).

>[!NOTE]
> Como biblioteca de código abierto, MLflow cambia con frecuencia. Como tal, la funcionalidad que se pone a disposición a través de la integración de Azure Machine Learning y MLflow debe considerarse como una vista previa y no es totalmente compatible con Microsoft.

> [!TIP]
> La información de este documento va destinada principalmente a aquellos científicos de datos y desarrolladores que deseen supervisar el proceso de entrenamiento del modelo. Los administradores que estén interesados en la supervisión del uso de recursos y eventos desde Azure Machine Learning, como cuotas, ejecuciones de entrenamiento completadas o implementaciones de modelos completadas pueden consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Requisitos previos

* Instale el paquete `azureml-mlflow`. 
    * Este paquete incorpora automáticamente `azureml-core` del [SDK de Azure Machine Learning de Python](/python/api/overview/azure/ml/install), que proporciona la conectividad de MLflow para acceder al área de trabajo.
* Un [área de trabajo y un clúster de Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).
* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
    * Consulte qué [permisos de acceso necesita para realizar las operaciones de MLflow con el área de trabajo](how-to-assign-roles.md#mlflow-operations).

## <a name="track-azure-databricks-runs"></a>Seguimiento de las ejecuciones de Azure Databricks

MLflow Tracking con Azure Machine Learning le permite almacenar las métricas y los artefactos registrados de las ejecuciones de Azure Databricks en su: 

* Área de trabajo de Azure Databricks
* Área de trabajo de Azure Machine Learning

Después de crear el área de trabajo y el clúster de Azure Databricks: 

1. Instale la biblioteca *azureml-mlflow* desde PyPi para asegurarse de que el clúster tenga acceso a las funciones y las clases necesarias.

1. Configure el cuaderno del experimento.

1. Conecte las áreas de trabajo de Azure Databricks y Azure Machine Learning.

En las secciones siguientes encontrará información adicional sobre estos pasos para poder ejecutar correctamente los experimentos de MLflow con Azure Databricks. 

## <a name="install-libraries"></a>Instalar bibliotecas

Para instalar las bibliotecas en el clúster, vaya a la pestaña **Libraries** (Bibliotecas) y seleccione **Install New** (Instalar nuevo).

 ![mlflow con azure databricks](./media/how-to-use-mlflow-azure-databricks/azure-databricks-cluster-libraries.png)

En el campo **Package** (Paquete), escriba azureml-mlflow y, luego, seleccione Install (Instalar). Repita este paso según sea necesario para instalar otros paquetes adicionales en el clúster para el experimento.

 ![Instalación de la biblioteca mlflow de Azure DB](./media/how-to-use-mlflow-azure-databricks/install-libraries.png)

## <a name="set-up-your-notebook"></a>Configuración del cuaderno 

Una vez configurado el clúster de ADB: 
1. Seleccione **Workspaces** (Áreas de trabajo) en el panel de navegación izquierdo. 
1. Expanda el menú desplegable de áreas de trabajo y seleccione **Import** (Importar).
1. Arrastre y coloque, o examine para encontrar, el cuaderno del experimento para importar el área de trabajo de ADB.
1. Seleccione **Import** (Importar). El cuaderno del experimento se abre automáticamente.
1. En el título del cuaderno de la parte superior izquierda, seleccione el clúster que quiere asociar al cuaderno del experimento. 

## <a name="connect-your-azure-databricks-and-azure-machine-learning-workspaces"></a>Conectar las áreas de trabajo de Azure Databricks y Azure Machine Learning

La vinculación del área de trabajo de ADB con el área de trabajo de Azure Machine Learning le permite realizar un seguimiento de los datos del experimento en el área de trabajo de Azure Machine Learning.

Para vincular el área de trabajo de ADB con un área de trabajo de Azure Machine Learning nueva o existente: 
1. Inicie sesión en el [portal de Azure](https://ms.portal.azure.com).
1. Vaya a la página **Información general** del área de trabajo de ADB.
1. Seleccione el botón **Link Azure Machine Learning workspace** (Vincular el área de trabajo de Azure Machine Learning) en la parte inferior derecha. 

 ![Vínculo a las áreas de trabajo de Azure DB y Azure Machine Learning](./media/how-to-use-mlflow-azure-databricks/link-workspaces.png)

## <a name="mlflow-tracking-in-your-workspaces"></a>MLflow Tracking en las áreas de trabajo

Después de crear una instancia del área de trabajo, MLflow Tracking se establece automáticamente para que se realice el seguimiento en todos los lugares siguientes:

* El área de trabajo vinculada de Azure Machine Learning.
* El área de trabajo ADB original. 

Todos los experimentos se redirigen al servicio de seguimiento de Azure Machine Learning administrado.

El código siguiente debe estar en el cuaderno del experimento para obtener el área de trabajo de Azure Machine Learning vinculada. 

Este código: 

*  Obtiene los detalles de la suscripción a Azure para crear una instancia del área de trabajo de Azure Machine Learning. 

* Da por hecho que tiene un grupo de recursos y un área de trabajo de Azure Machine Learning. Si no es así, puede [crearlos](how-to-manage-workspace.md). 

* Establece el nombre del experimento. Aquí, el valor de `user_name` es coherente con el valor de `user_name` asociado con el área de trabajo de Azure Databricks.

```python
import mlflow
import mlflow.azureml
import azureml.mlflow
import azureml.core

from azureml.core import Workspace

subscription_id = 'subscription_id'

# Azure Machine Learning resource group NOT the managed resource group
resource_group = 'resource_group_name' 

#Azure Machine Learning workspace name, NOT Azure Databricks workspace
workspace_name = 'workspace_name'  

# Instantiate Azure Machine Learning workspace
ws = Workspace.get(name=workspace_name,
                   subscription_id=subscription_id,
                   resource_group=resource_group)

#Set MLflow experiment. 
experimentName = "/Users/{user_name}/{experiment_folder}/{experiment_name}" 
mlflow.set_experiment(experimentName) 

```

### <a name="set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace"></a>Establecimiento del seguimiento de MLflow para que solo realice el seguimiento en el área de trabajo de Azure Machine Learning

Si prefiere administrar los experimentos con seguimiento en una ubicación centralizada, puede establecer el seguimiento de MLflow para que **solo** realice el seguimiento en el área de trabajo de Azure Machine Learning. 

Incluya el código siguiente en el script:

```python
uri = ws.get_mlflow_tracking_uri()
mlflow.set_tracking_uri(uri)
```

En el script de entrenamiento, importe `mlflow` para usar las API de registro de MLflow y empiece a registrar las métricas de ejecución. En el ejemplo siguiente, se registra la métrica de pérdida de tiempo. 

```python
import mlflow 
mlflow.log_metric('epoch_loss', loss.item()) 
```

## <a name="register-models-with-mlflow"></a>Registro de modelos con MLflow

Una vez entrenado el modelo, puede registrarlo en el servidor de seguimiento de back-end con el método `mlflow.<model_flavor>.log_model()`. `<model_flavor>` hace referencia al marco asociado al modelo. [Conozca qué tipos de modelos se admiten](https://mlflow.org/docs/latest/models.html#model-api). 

De forma predeterminada, el servidor de seguimiento de back-end es el área de trabajo de Azure Databricks, a menos que decida [establecer MLflow Tracking para que solo realice el seguimiento en el área de trabajo de Azure Machine Learning](#set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace), en cuyo caso, el servidor de seguimiento será dicha área de trabajo.   

* **Si no existe un modelo registrado con el nombre**, el método registra un modelo nuevo, crea la versión 1 y devuelve un objeto MLflow ModelVersion. 

* **Si ya existe un modelo registrado con el nombre**, el método crea una versión del modelo y devuelve el objeto de versión. 

```python
mlflow.spark.log_model(model, artifact_path = "model", 
                       registered_model_name = 'model_name')  

mlflow.sklearn.log_model(model, artifact_path = "model", 
                         registered_model_name = 'model_name') 
```

## <a name="create-endpoints-for-mlflow-models"></a>Creación de puntos de conexión para modelos de MLflow

Cuando esté preparado para crear un punto de conexión para los modelos de aprendizaje automático, puede implementarlo como: 

* Un servicio web de solicitud-respuesta de Azure Machine Learning para la puntuación interactiva. Esta implementación le permite aprovechar y aplicar la administración de modelos de Azure Machine Learning y las funcionalidades de detección de desfase de datos a los modelos de producción. 

* Objetos de modelo de MLFlow, que se pueden usar en canalizaciones de streaming o por lotes como funciones de Python o funciones definidas por el usuario (UDF) de Pandas en áreas de trabajo de Azure Databricks.

### <a name="deploy-models-to-azure-machine-learning-endpoints"></a>Implementación de modelos en puntos de conexión de Azure Machine Learning 
Puede aprovechar la API [mlflow.azureml.deploy](https://www.mlflow.org/docs/latest/python_api/mlflow.azureml.html#mlflow.azureml.deploy) para implementar un modelo en el área de trabajo de Azure Machine Learning. Si solo registró el modelo en el área de trabajo de Azure Databricks, como se describe en la sección [Registro de modelos con MLflow](#register-models-with-mlflow), especifique el parámetro `model_name` para registrar el modelo en el área de trabajo de Azure Machine Learning. 

Las ejecuciones de Azure Databricks se pueden implementar en los puntos de conexión siguientes: 
* [Azure Container Instances](how-to-deploy-mlflow-models.md#deploy-to-azure-container-instance-aci)
* [Azure Kubernetes Service](how-to-deploy-mlflow-models.md#deploy-to-azure-kubernetes-service-aks)

### <a name="deploy-models-to-adb-endpoints-for-batch-scoring"></a>Implementación de modelos en puntos de conexión de ADB para la puntuación por lotes 

Puede elegir clústeres de Azure Databricks para la puntuación por lotes. El modelo de MLFlow se carga y se usa como función definida por el usuario de Pandas de Spark para puntuar nuevos datos. 

```python
from pyspark.sql.types import ArrayType, FloatType 

model_uri = "runs:/"+last_run_id+ {model_path} 

#Create a Spark UDF for the MLFlow model 

pyfunc_udf = mlflow.pyfunc.spark_udf(spark, model_uri) 

#Load Scoring Data into Spark Dataframe 

scoreDf = spark.table({table_name}).where({required_conditions}) 


#Make Prediction 

preds = (scoreDf 

           .withColumn('target_column_name', pyfunc_udf('Input_column1', 'Input_column2', ' Input_column3', …)) 

        ) 

display(preds) 
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene pensado usar los artefactos o las métricas registradas en el área de trabajo, la funcionalidad para eliminarlos de forma individual no está disponible actualmente. Por ello, deberá eliminar el grupo de recursos que contiene la cuenta de almacenamiento y el área de trabajo para no incurrir en cargos:

1. En Azure Portal, seleccione **Grupos de recursos** a la izquierda del todo.

   ![Eliminación en Azure Portal](./media/how-to-use-mlflow-azure-databricks/delete-resources.png)

1. En la lista, seleccione el grupo de recursos que creó.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba el nombre del grupo de recursos. A continuación, seleccione **Eliminar**.

## <a name="example-notebooks"></a>Cuadernos de ejemplo

En [MLflow con cuadernos de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) se demuestran y se analizan con mayor profundidad los conceptos presentados en este artículo.

## <a name="next-steps"></a>Pasos siguientes
* [Implementación de modelos de MLflow como un servicio web de Azure](how-to-deploy-mlflow-models.md). 
* [Administra sus modelos](concept-model-management-and-deployment.md).
* [Seguimiento de ejecuciones de experimentos con MLflow y Azure Machine Learning](how-to-use-mlflow.md). 
* Más información sobre [Azure Databricks y MLflow](/azure/databricks/applications/mlflow/).