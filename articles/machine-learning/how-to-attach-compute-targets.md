---
title: Configuración de los destinos de proceso de entrenamiento e inferencia
titleSuffix: Azure Machine Learning
description: Adición de recursos de proceso (destinos de proceso) al área de trabajo para usarlos en el entrenamiento y la inferencia de aprendizaje automático
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 10/02/2020
ms.topic: how-to
ms.custom: devx-track-python, contperf-fy21q1
ms.openlocfilehash: 8c60afc16a75b69c85efc29375a5f10ca147a129
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143600"
---
# <a name="set-up-compute-targets-for-model-training-and-deployment"></a>Configuración de destinos de proceso para el entrenamiento y la implementación de modelos

Aprenda a asociar recursos de proceso de Azure al área de trabajo de Azure Machine Learning.  A continuación, puede usar estos recursos como [destinos de proceso](concept-compute-target.md) de entrenamiento e inferencia en las tareas de aprendizaje automático.

En este artículo, aprenderá a configurar el área de trabajo para usar estos recursos de proceso:

* Equipo local
* Máquinas virtuales remotas
* HDInsight de Azure
* Azure Batch
* Azure Databricks
* Análisis con Azure Data Lake
* Azure Container Instances

Para usar destinos de proceso administrados por Azure Machine Learning, consulte:


* [Instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md)
* [Clúster de proceso de Azure Machine Learning](how-to-create-attach-compute-cluster.md)
* [Clúster de Azure Kubernetes Service](how-to-create-attach-kubernetes.md)

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](/python/api/overview/azure/ml/intro) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

## <a name="limitations"></a>Limitaciones

* **No cree varios datos adjuntos simultáneos en el mismo proceso** desde su área de trabajo. Por ejemplo, adjuntar un clúster de Azure Kubernetes Service a un área de trabajo con dos nombres diferentes. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

    Si quiere volver a asociar un destino de proceso, por ejemplo, para cambiar la configuración de TLS u otra configuración del clúster, primero debe eliminar los datos adjuntos existentes.

## <a name="whats-a-compute-target"></a>¿Qué es un destino de proceso?

Con Azure Machine Learning, puede entrenar el modelo en una variedad de recursos o entornos, denominados colectivamente [__destinos de proceso__](concept-azure-machine-learning-architecture.md#compute-targets). Un destino de proceso puede ser una máquina local o un recurso de nube, como una instancia de proceso de Azure Machine Learning, Azure HDInsight o una máquina virtual remota.  También puede crear destinos de proceso para la implementación de modelos, como se describe en ["Cómo y dónde implementar los modelos"](how-to-deploy-and-where.md).


## <a name="local-computer"></a><a id="local"></a>Equipo local

Cuando se usa el equipo local para el **entrenamiento**, no es necesario crear un destino de proceso.  Simplemente [envíe la ejecución de entrenamiento](how-to-set-up-training-targets.md) desde la máquina local.

Cuando se usa el equipo local para la **inferencia**, debe tener instalado Docker. Para realizar la implementación, utilice [LocalWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.local.localwebservice#deploy-configuration-port-none-) para definir el puerto que utilizará el servicio web. A continuación, use el proceso de implementación normal como se describe en [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="remote-virtual-machines"></a><a id="vm"></a>Máquinas virtuales remotas

Azure Machine Learning también admite la asociación de una máquina virtual de Azure. La máquina virtual debe ser una instancia de Data Science Virtual Machine (DSVM) de Azure. La máquina virtual ofrece una selección de herramientas y plataformas mantenidas para el desarrollo del aprendizaje automático a lo largo de todo el ciclo de vida. Para obtener más información sobre cómo usar la DSVM con Azure Machine Learning, consulte [Configuración del entorno de desarrollo](./how-to-configure-environment.md#dsvm).

> [!TIP]
> En lugar de una máquina virtual remota, se recomienda usar la [instancia de proceso de Azure Machine Learning](concept-compute-instance.md). Es una solución de proceso basada en la nube totalmente administrada que es específica de Azure Machine Learning. Para más información, consulte [creación y administración de una instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md).

1. **Crear**: Azure Machine Learning no puede crear una máquina virtual remota. En su lugar, debe crear la máquina virtual y, luego, asociarlo al área de trabajo de Azure Machine Learning. Para saber cómo crear un DSVM, consulte [Aprovisionamiento de Data Science Virtual Machine para Linux (Ubuntu)](./data-science-virtual-machine/dsvm-ubuntu-intro.md).

    > [!WARNING]
    > Azure Machine Learning solo admite máquinas virtuales que ejecuten **Ubuntu**. Al crear una máquina virtual o elegir una máquina virtual existente, debe seleccionar una máquina virtual que use Ubuntu.
    > 
    > Azure Machine Learning también requiere que la máquina virtual tenga una __dirección IP pública__.

1. **Adjuntar**: para asociar una máquina virtual existente como destino de proceso, debe proporcionar el identificador de recurso, el nombre de usuario y la contraseña de la máquina virtual. El identificador de recurso de la máquina virtual se puede construir con el identificador de la suscripción, el nombre del grupo de recursos y el nombre de la máquina virtual con el siguiente formato de cadena: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Compute/virtualMachines/<vm_name>`.

 
   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   
   attach_config = RemoteCompute.attach_configuration(resource_id='<resource_id>',
                                                   ssh_port=22,
                                                   username='<username>',
                                                   password="<password>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   O bien puede asociar la instancia de DSVM al área de trabajo [mediante Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > No cree varios datos adjuntos simultáneos en el mismo DSVM desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de Data Science Virtual Machine. Docker y Conda se utilizan para crear y configurar el entorno de entrenamiento en la DSVM.

   ```python
   from azureml.core import ScriptRunConfig
   from azureml.core.environment import Environment
   from azureml.core.conda_dependencies import CondaDependencies
   
   # Create environment
   myenv = Environment(name="myenv")
   
   # Specify the conda dependencies
   myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
   
   # If no base image is explicitly specified the default CPU image "azureml.core.runconfig.DEFAULT_CPU_IMAGE" will be used
   # To use GPU in DSVM, you should specify the default GPU base Docker image or another GPU-enabled image:
   # myenv.docker.enabled = True
   # myenv.docker.base_image = azureml.core.runconfig.DEFAULT_GPU_IMAGE
   
   # Configure the run configuration with the Linux DSVM as the compute target and the environment defined above
   src = ScriptRunConfig(source_directory=".", script="train.py", compute_target=compute, environment=myenv) 
   ```

> [!TIP]
> Si desea __quitar__ (desasociar) una máquina virtual del área de trabajo, use el método [RemoteCompute.detach ()](/python/api/azureml-core/azureml.core.compute.remotecompute#detach--).
>
> Azure Machine Learning no eliminará la máquina virtual. Debe eliminar manualmente la máquina virtual mediante el Azure Portal, la CLI o el SDK para la máquina virtual de Azure.

## <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight es una plataforma popular para el análisis de macrodatos. La plataforma proporciona Apache Spark, que se puede usar para entrenar el modelo.

1. **Crear**: Azure Machine Learning no puede crear un clúster de HDInsight. En su lugar, debe crear el clúster y, luego, asociarlo al área de trabajo de Azure Machine Learning. Para más información, consulte [Crear un clúster de Spark en HDInsight](../hdinsight/spark/apache-spark-jupyter-spark-sql.md). 

    > [!WARNING]
    > Azure Machine Learning requiere que el clúster de HDInsight tenga una __dirección IP pública__.

    Al crear el clúster, debe especificar un nombre de usuario SSH y una contraseña. Anote estos valores, ya que los necesitará al usar HDInsight como destino de proceso.
    
    Cuando se haya creado el clúster, conéctelo al nombre de host \<clustername>-ssh.azurehdinsight.net, donde \<clustername> es el nombre designado al clúster. 

1. **Adjuntar**: para asociar un clúster de HDInsight como destino de proceso, debe proporcionar el nombre de host, el nombre de usuario y la contraseña del clúster de HDInsight. El identificador de recurso del clúster de HDInsight se puede construir con el identificador de la suscripción, el nombre del grupo de recursos y el nombre del clúster de HDInsight con el siguiente formato de cadena: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.HDInsight/clusters/<cluster_name>`.

    ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase

    attach_config = HDInsightCompute.attach_configuration(resource_id='<resource_id>',
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   O bien puede asociar el clúster de HDInsight al área de trabajo [mediante Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > No cree varios datos adjuntos simultáneos en el mismo HDInsight desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

1. **Configurar**: Cree una configuración de ejecución para el destino de proceso de HDI. 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]

> [!TIP]
> Si desea __quitar__ (desasociar) un clúster de HDInsight del área de trabajo, use el método [HDInsightCompute.detach()](/python/api/azureml-core/azureml.core.compute.hdinsight.hdinsightcompute#detach--).
>
> Azure Machine Learning no elimina el clúster de HDInsight. Debe eliminarlo manualmente mediante el Azure Portal, la CLI o el SDK para Azure HDInsight.

## <a name="azure-batch"></a><a id="azbatch"></a>Azure Batch 

Azure Batch se usa para ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala de manera eficaz en la nube. AzureBatchStep se puede usar en una canalización de Azure Machine Learning para enviar trabajos al grupo de máquinas de Azure Batch.

Para adjuntar Azure Batch como destino de proceso, debe usar el SDK de Azure Machine Learning y proporcionar la siguiente información:

-    **Nombre de proceso de Azure Batch**: nombre descriptivo que se usará para el proceso en el área de trabajo.
-    **Nombre de cuenta de Azure Batch**: nombre de la cuenta de Azure Batch.
-    **Grupo de recursos**: grupo de recursos que contiene la cuenta de Azure Batch.

El código siguiente muestra cómo asociar Azure Batch como destino de proceso:

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

# Name to associate with new compute in workspace
batch_compute_name = 'mybatchcompute'

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>"  # Name of the Batch account
# Name of the resource group which contains this account
batch_resource_group = "<batch_resource_group>"

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(
        resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(
        ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

> [!WARNING]
> No cree varios datos adjuntos simultáneos en la misma instancia de Azure Batch desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

### <a name="azure-databricks"></a><a id="databricks"></a>Azure Databricks

Azure Databricks es un entorno basado en Apache Spark de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

> [!IMPORTANT} Azure Machine Learning no puede crear un destino de proceso de Azure Databricks. En su lugar, debe crear un área de trabajo de Azure Databricks y, a continuación, adjuntarla al área de trabajo de Azure Machine Learning. Para crear un recurso de área de trabajo, consulte el documento [Ejecución de un trabajo de Spark en Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).

Para adjuntar Azure Databricks como destino de proceso, proporcione la información siguiente:

* __Nombre del proceso de Databricks__: el nombre que desea asignar a este recurso de proceso.
* __Nombre de área de trabajo de Databricks__: el nombre del área de trabajo de Azure Databricks.
* __Token de acceso de Databricks__: el token de acceso usado para autenticarse en Azure Databricks. Para generar un token de acceso, consulte el documento [Autenticación](/azure/databricks/dev-tools/api/latest/authentication).

En el código siguiente se muestra cómo adjuntar Azure Databricks como un destino de proceso con el SDK de Azure Machine Learning (__el área de trabajo de Databricks debe estar presente en la misma suscripción que el área de trabajo de AML__):

```python
import os
from azureml.core.compute import ComputeTarget, DatabricksCompute
from azureml.exceptions import ComputeTargetException

databricks_compute_name = os.environ.get(
    "AML_DATABRICKS_COMPUTE_NAME", "<databricks_compute_name>")
databricks_workspace_name = os.environ.get(
    "AML_DATABRICKS_WORKSPACE", "<databricks_workspace_name>")
databricks_resource_group = os.environ.get(
    "AML_DATABRICKS_RESOURCE_GROUP", "<databricks_resource_group>")
databricks_access_token = os.environ.get(
    "AML_DATABRICKS_ACCESS_TOKEN", "<databricks_access_token>")

try:
    databricks_compute = ComputeTarget(
        workspace=ws, name=databricks_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('databricks_compute_name {}'.format(databricks_compute_name))
    print('databricks_workspace_name {}'.format(databricks_workspace_name))
    print('databricks_access_token {}'.format(databricks_access_token))

    # Create attach config
    attach_config = DatabricksCompute.attach_configuration(resource_group=databricks_resource_group,
                                                           workspace_name=databricks_workspace_name,
                                                           access_token=databricks_access_token)
    databricks_compute = ComputeTarget.attach(
        ws,
        databricks_compute_name,
        attach_config
    )

    databricks_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-databricks) en GitHub.

> [!WARNING]
> No cree varios datos adjuntos simultáneos en la misma instancia de Azure Databricks desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

### <a name="azure-data-lake-analytics"></a><a id="adla"></a>Azure Data Lake Analytics

Azure Data Lake Analytics es una plataforma de análisis de macrodatos de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

Cree una cuenta de Azure Data Lake Analytics antes de usarlo. Para crear este recurso, consulte la [Introducción a Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md) documento.

Para asociar Data Lake Analytics como destino de proceso, debe usar el SDK de Azure Machine Learning y proporcionar la siguiente información:

* __Nombre de proceso__: el nombre que desea asignar a este recurso de proceso.
* __Grupo de recursos__: el grupo de recursos que contiene la cuenta de Data Lake Analytics.
* __Nombre de cuenta__: El nombre de la cuenta de Data Lake Analytics.

El código siguiente muestra cómo asociar Data Lake Analytics como destino de proceso:

```python
import os
from azureml.core.compute import ComputeTarget, AdlaCompute
from azureml.exceptions import ComputeTargetException


adla_compute_name = os.environ.get(
    "AML_ADLA_COMPUTE_NAME", "<adla_compute_name>")
adla_resource_group = os.environ.get(
    "AML_ADLA_RESOURCE_GROUP", "<adla_resource_group>")
adla_account_name = os.environ.get(
    "AML_ADLA_ACCOUNT_NAME", "<adla_account_name>")

try:
    adla_compute = ComputeTarget(workspace=ws, name=adla_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('adla_compute_name {}'.format(adla_compute_name))
    print('adla_resource_id {}'.format(adla_resource_group))
    print('adla_account_name {}'.format(adla_account_name))
    # create attach config
    attach_config = AdlaCompute.attach_configuration(resource_group=adla_resource_group,
                                                     account_name=adla_account_name)
    # Attach ADLA
    adla_compute = ComputeTarget.attach(
        ws,
        adla_compute_name,
        attach_config
    )

    adla_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-adla) en GitHub.

> [!WARNING]
> No cree varios datos adjuntos simultáneos en el mismo ADLA desde su área de trabajo. Cada adjunto nuevo interrumpirá los adjuntos anteriores existentes.

> [!TIP]
> Las canalizaciones de Azure Machine Learning solo pueden trabajar con datos almacenados en el almacén de datos predeterminado de la cuenta de Data Lake Analytics. Si los datos con los que necesita trabajar están en un almacén no predeterminado, puede usar [`DataTransferStep`](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep) para copiar los datos antes del entrenamiento.

## <a name="azure-container-instance"></a><a id="aci"></a>Azure Container Instances

Las instancias de Azure Container Instances (ACI) se crean dinámicamente al implementar un modelo. No se puede crear o adjuntar ACI al área de trabajo de ninguna otra manera. Para más información, consulte [Implementación de un modelo en Azure Container Instances](how-to-deploy-azure-container-instance.md).

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) permite diferentes opciones de configuración cuando se usa con Azure Machine Learning. Para obtener más información, consulte [Creación y conexión de Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

## <a name="notebook-examples"></a>Ejemplos de cuadernos

Consulte estos cuadernos para ver ejemplos de entrenamiento con varios destinos de proceso:
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* Use el recurso de proceso para [configurar y enviar una ejecución de entrenamiento](how-to-set-up-training-targets.md).
* [Tutorial: Entrenamiento de un modelo](tutorial-train-models-with-aml.md) utiliza un destino de proceso administrado para entrenar un modelo.
* Obtenga información sobre cómo [ajustar los hiperparámetros eficazmente](how-to-tune-hyperparameters.md) para crear modelos mejores.
* Cuando tenga un modelo entrenado, aprenda [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).
* [Uso de Azure Machine Learning con Azure Virtual Networks](./how-to-network-security-overview.md)