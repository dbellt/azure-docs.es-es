---
title: Conmutación por error y recuperación ante desastres
titleSuffix: Azure Machine Learning
description: Aprenda a planear la recuperación ante desastres y garantizar la continuidad empresarial para Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 05/05/2021
ms.openlocfilehash: 7114d01ec165b3c31632b0465f3a98dd834ce1e8
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804396"
---
# <a name="failover-for-business-continuity-and-disaster-recovery"></a>Conmutación por error para la continuidad empresarial y la recuperación ante desastres

Para maximizar el tiempo de actividad, planee con antelación la continuidad empresarial y prepare todo lo necesario para la recuperación ante desastres con Azure Machine Learning. 

Microsoft se esfuerza por garantizar que los servicios de Azure siempre estén disponibles. Sin embargo, es posible que se produzcan interrupciones en el servicio. Se recomienda disponer de un plan de recuperación ante desastres para abordar las interrupciones de servicio en regiones. En este artículo, aprenderá a:

* Planear una implementación multirregional de Azure Machine Learning y los recursos asociados.
* Diseñar la alta disponibilidad para su solución.
* Iniciar una conmutación por error a otra región.

> [!NOTE]
> El servicio Azure Machine Learning no proporciona por sí mismo conmutación automática por error ni recuperación ante desastres.

## <a name="understand-azure-services-for-azure-machine-learning"></a>Descripción de los servicios de Azure para Azure Machine Learning

Azure Machine Learning depende de varios servicios de Azure y tiene varias capas. Algunos de estos servicios se aprovisionan en su suscripción (cliente). El usuario es responsable de la configuración de alta disponibilidad de estos servicios. Otros se crean en una suscripción de Microsoft y son administrados por Microsoft. 

Los servicios de Azure incluyen:

* **Infraestructura de Azure Machine Learning**: Un entorno administrado por Microsoft para el área de trabajo de Azure Machine Learning.

* **Recursos asociados**: recursos aprovisionados en la suscripción durante la creación del área de trabajo de Azure Machine Learning. Estos recursos incluyen Azure Storage, Azure Key Vault, Azure Container Registry y Application Insights. El usuario es responsable de la configuración de alta disponibilidad de estos recursos.
  * El almacenamiento predeterminado contiene datos, como el modelo, los datos de registro de entrenamiento y el conjunto de datos.
  * Key Vault tiene credenciales para Azure Storage, Container Registry y almacenes de datos.
  * Container Registry tiene una imagen de Docker para entornos de aprendizaje e inferencia.
  * Application Insights se usa para la supervisión de Azure Machine Learning.

* **Recursos de proceso**: recursos creados después de la implementación del área de trabajo. Por ejemplo, puede crear una instancia de proceso o un clúster de proceso para entrenar un modelo de Machine Learning.
  * Instancia de proceso y clúster de proceso: Entornos de desarrollo de modelos administrado por Microsoft.
  * Otros recursos: Los recursos de computación que se pueden asociar a Azure Machine Learning, como Azure Kubernetes Service (AKS), Azure Databricks, Azure Container Instances y Azure HDInsight. El usuario es responsable de la configuración de alta disponibilidad de estos recursos.

* **Otros almacenes de datos**: Azure Machine Learning puede montar otros almacenes de datos, como Azure Storage, Azure Data Lake Storage y Azure SQL Database para datos de aprendizaje.  Estos almacenes de datos se aprovisionan en la suscripción. El usuario es responsable de la configuración de las opciones de alta disponibilidad.

En la tabla siguiente se muestran los servicios de Azure administrados por Microsoft y los administrados por el usuario. También se indican los servicios de alta disponibilidad, de forma predeterminada.

| Servicio | Administrado por | Alta disponibilidad de forma predeterminada |
| ----- | ----- | ----- |
| **Infraestructura de Azure Machine Learning** | Microsoft | |
| **Recursos asociados** |
| Azure Storage | Los | |
| Key Vault | Los | ✓ |
| Container Registry | Los | |
| Application Insights | Los | N/D |
| **Recursos de proceso** |
| Instancia de proceso | Microsoft |  |
| Clúster de proceso | Microsoft |  |
| Otros recursos de proceso, como AKS, <br>Azure Databricks, Container Instances, HDInsight | Usted |  |
| **Otros almacenes de datos**, como Azure Storage, SQL Database,<br> Azure Database for PostgreSQL, Azure Database for MySQL, <br>Sistema de archivos de Azure Databricks | Usted | |

En el resto de este artículo se describen las acciones que el usuario tiene que realizar para que cada uno de estos servicios sea de alta disponibilidad.

## <a name="plan-for-multi-regional-deployment"></a>Planeamiento de la implementación multirregional

Una implementación multirregional se basa en la creación de Azure Machine Learning y otros recursos (infraestructura) en dos regiones de Azure. Si se produce una interrupción en una región, se puede cambiar a la otra. Cuando se plantee dónde implementar sus recursos, tenga en cuenta lo siguiente:

* __Disponibilidad por región:__ utilice regiones próximas a sus usuarios. Para comprobar la disponibilidad por región para Azure Machine Learning, consulte los [productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).
* __Regiones emparejadas de Azure__: coordinan las actualizaciones de la plataforma y priorizan las iniciativas de recuperación según las necesidades. Para más información, consulte [Regiones emparejadas de Azure](../best-practices-availability-paired-regions.md).
* __Disponibilidad del servicio__: determine si la disponibilidad de los recursos utilizados por la solución debe ser activa/activa, activa/en espera o activa/pasiva.
    
    * __activa/activa__: ambas regiones están activas al mismo tiempo y una de ellas se puede empezar a usar inmediatamente.
    * __activa/en espera__: la región primaria está activa y la secundaria tiene recursos críticos (por ejemplo, modelos implementados) preparados para iniciarse. Los recursos no críticos tendrían que implementarse manualmente en la región secundaria.
    * __activa/pasiva__: la región primaria está activa y la región secundaria tiene el servicio Azure Machine Learning y otros recursos implementados, junto con los datos necesarios. Recursos como los modelos, las implementaciones de modelos o las canalizaciones tendrían que implementarse manualmente.

> [!TIP]
> En función de los requisitos de su empresa, puede optar por tratar de forma diferente los recursos de Azure Machine Learning. Por ejemplo, es posible que desee usar la configuración activa/activa para los modelos implementados (inferencia) y la configuración activa/pasiva para los experimentos (entrenamiento).

Azure Machine Learning se agrega a otros servicios. Algunos servicios se pueden configurar para la replicación en otras regiones. Otros servicios se deben crear manualmente en varias regiones. En la tabla siguiente se proporciona una lista de servicios responsables de la replicación y una descripción de la configuración:

| Servicio de Azure | Replicación geográfica | Configuración |
| ----- | ----- | ----- |
| Área de trabajo de Machine Learning | Usted | Cree un área de trabajo en las regiones seleccionadas. |
| Proceso de Machine Learning | Usted | Cree los recursos de proceso en las regiones seleccionadas. En el caso de los recursos de proceso escalables de forma dinámica, asegúrese de que ambas regiones proporcionen una cuota de proceso suficiente en relación con sus necesidades. |
| Key Vault | Microsoft | Use la misma instancia de Key Vault con el área de trabajo y los recursos de Azure Machine Learning en ambas regiones. Key Vault conmuta por error de forma automática en una región secundaria. Para más información, consulte [Redundancia y disponibilidad de Azure Key Vault](../key-vault/general/disaster-recovery-guidance.md).|
| Container Registry | Microsoft | Configure la instancia de Container Registry para que replique geográficamente los registros en la región emparejada para Azure Machine Learning. Use la misma versión para ambas instancias de área de trabajo. Para más información, consulte [Replicación geográfica en Azure Container Registry](../container-registry/container-registry-geo-replication.md). |
| Cuenta de almacenamiento | Usted | Azure Machine Learning no admite la conmutación por error en la __cuenta de almacenamiento predeterminada__ por medio del almacenamiento con redundancia geográfica (GRS), el almacenamiento con redundancia de zona geográfica (GZRS), el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS). Cree una cuenta de almacenamiento separada para el almacenamiento predeterminado de cada área de trabajo. </br>Cree servicios o cuentas de almacenamiento separadas para otro almacenamiento de datos. Para más información, vea [Redundancia de Azure Storage](../storage/common/storage-redundancy.md). |
| Application Insights | Los | Cree una instancia de Application Insights para el área de trabajo en ambas regiones. Para ajustar el período de retención de datos y los detalles consulte [Recopilación, retención y almacenamiento de datos en Application Insights](../azure-monitor/app/data-retention-privacy.md#how-long-is-the-data-kept). |

Para acelerar la recuperación y el reinicio en la región secundaria, se recomiendan los siguientes procedimientos de desarrollo:

* Utilice plantillas de Azure Resource Manager. Las plantillas generan "infraestructura como código" y permiten implementar rápidamente servicios en ambas regiones.
* Para evitar el desvío entre las dos regiones, actualice las canalizaciones de integración e implementación continuas para que se implementen en ambas regiones.
* Cuando automatice las implementaciones, incluya la configuración de los recursos de proceso asociados al área de trabajo, como Azure Kubernetes Service.
* Cree asignaciones de rol para los usuarios de ambas regiones.
* Cree recursos de red, como las redes virtuales y los puntos de conexión privados de Azure, para ambas regiones. Asegúrese de que los usuarios tengan acceso a ambos entornos de red. Por ejemplo, las configuraciones de VPN y DNS para ambas redes virtuales.

### <a name="compute-and-data-services"></a>Servicios de proceso y de datos

En función de cuáles sean sus necesidades, podrá tener más servicios de proceso o de datos para que Azure Machine Learning los utilice. Por ejemplo, puede usar Azure Kubernetes Services o Azure SQL Database. Use la siguiente información para aprender a configurar estos servicios con fines de alta disponibilidad.

__Recursos de proceso__

* **Azure Kubernetes Service**: consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en Azure Kubernetes Service (AKS)](../aks/operator-best-practices-multi-region.md) y [Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad](../aks/availability-zones.md). Si el clúster de AKS se creó en Azure Machine Learning Studio, el SDK o la CLI, no se admite alta disponibilidad entre regiones.
* **Azure Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](/azure/databricks/scenarios/howto-regional-disaster-recovery).
* **Container Instances**. Un orquestador es responsable de la conmutación por error. Consulte [Azure Container Instances y orquestadores de contenedores](../container-instances/container-instances-orchestrator-relationship.md).
* **HDInsight**: consulte [Servicios de alta disponibilidad admitidos en Azure HDInsight](../hdinsight/hdinsight-high-availability-components.md).

__Servicios de datos__

* **Contenedor de Azure Blob Storage, Azure Files y Data Lake Storage Gen2**: consulte [Redundancia de Azure Storage](../storage/common/storage-redundancy.md).
* **Data Lake Storage Gen1**: Consulte [Guía de alta disponibilidad y recuperación ante desastres para Data Lake Storage Gen1](../data-lake-store/data-lake-store-disaster-recovery-guidance.md).
* **SQL Database**: Consulte [Alta disponibilidad para Azure SQL Database e Instancia administrada de SQL](../azure-sql/database/high-availability-sla.md).
* **Azure Database for PostgreSQL**: consulte [Conceptos de alta disponibilidad en Azure Database for PostgreSQL: servidor único](../postgresql/concepts-high-availability.md).
* **Azure Database for MySQL**: consulte [Información acerca de la continuidad empresarial en Azure Database for MySQL](../mysql/concepts-business-continuity.md).
* **Sistema de archivos de Azure Databricks**: consulte [Recuperación ante desastres regional para clústeres de Azure Databricks](/azure/databricks/scenarios/howto-regional-disaster-recovery).

> [!TIP]
> Si proporciona su propia clave administrada por el cliente para implementar el área de trabajo de Azure Machine Learning, Azure Cosmos DB también se aprovisiona dentro de la suscripción. En ese caso, es responsable de configurar la configuración de alta disponibilidad. Consulte [Alta disponibilidad con Azure Cosmos DB](../cosmos-db/high-availability.md).

## <a name="design-for-high-availability"></a>Diseño para lograr alta disponibilidad

### <a name="deploy-critical-components-to-multiple-regions"></a>Implementación de componentes críticos en varias regiones

Determine el nivel de continuidad empresarial que desea obtener. El nivel puede diferir variar de un componente a otro de la solución. Por ejemplo, tal vez desee usar una configuración activa/activa para las canalizaciones de producción o las implementaciones de modelos, y una configuración activa/inactiva con fines de experimentación.

### <a name="manage-training-data-on-isolated-storage"></a>Administración de datos de entrenamiento en un almacenamiento aislado

Al mantener el almacenamiento de datos aislado del almacenamiento predeterminado que el área de trabajo usa para los registros, podrá hacer lo siguiente:

* Adjuntar las mismas instancias de almacenamiento que los almacenes de datos a las áreas de trabajo principal y secundaria.
* Aplicar la replicación geográfica a las cuentas de almacenamiento de datos y maximizar el tiempo de actividad.

### <a name="manage-machine-learning-artifacts-as-code"></a>Administración de artefactos de aprendizaje automático como código

Las ejecuciones Azure Machine Learning se definen mediante una especificación de ejecución. Esta especificación incluye dependencias de artefactos de entrada que se administran en un nivel de instancia de área de trabajo, como entornos, conjuntos de datos y procesos. Para el envío de ejecuciones e implementaciones multirregionales, se recomiendan los procedimientos siguientes:

* Administre localmente el código base, con ayuda de un repositorio de Git.
    * Exporte cuadernos relevantes desde Estudio de Azure Machine Learning.
    * Exporte canalizaciones que se crearon como código en Estudio.

      > [!NOTE]
      > Las canalizaciones creadas en el diseñador de Studio no se pueden exportar actualmente como código.

* Administre las configuraciones como código.
    * Evite las referencias codificadas de forma rígida al área de trabajo. En su lugar, configure una referencia a la instancia del área de trabajo mediante un [archivo de configuración](how-to-configure-environment.md#workspace) y utilice [Workspace.from_config()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace#remarks) para inicializar el área de trabajo. Para automatizar el proceso, utilice el comando [az ml folder attach](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/folder#ext_azure_cli_ml_az_ml_folder_attach) en la [extensión de la CLI de Azure para el aprendizaje automático](reference-azure-machine-learning-cli.md).
    * Use asistentes de envío de ejecuciones, [como ScriptRunConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig) y [Pipeline](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline(class)).
    * Use [Environments.save_to_directory() para](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment(class)#save-to-directory-path--overwrite-false-) guardar las definiciones de entorno.
    * Use Dockerfile si tiene imágenes personalizadas de Docker.
    * Use la clase [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)) para definir la colección de rutas de [datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data.datapath) utilizadas por la solución.
    * Use la clase [Inferenceconfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig) para implementar modelos como puntos de conexión de inferencia.

## <a name="initiate-a-failover"></a>Inicio de una conmutación por error

### <a name="continue-work-in-the-failover-workspace"></a>Continuación del trabajo en el área de trabajo de conmutación por error

Cuando el área de trabajo principal deja de estar disponible, puede cambiar al área de trabajo secundaria con fines de experimentación y desarrollo. Azure Machine Learning no envía automáticamente ejecuciones al área de trabajo secundaria si se produce una interrupción. Actualice la configuración del código para que apunte al nuevo recurso de área de trabajo. Se recomienda evitar la codificación rígida de las referencias al área de trabajo. En su lugar, use un [archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace) para minimizar los pasos manuales del usuario para cambiar las áreas de trabajo. Asegúrese de actualizar también cualquier automatización, como las canalizaciones de integración e implementación continuas, en la nueva área de trabajo.

Azure Machine Learning no puede sincronizar ni recuperar artefactos o metadatos entre instancias de área de trabajo. En función de su enfoque de implementación de aplicaciones, es posible que tenga que cambiar los artefactos de sitio o volver a crear entradas de experimentación, como los objetos de conjunto de datos, en el área de trabajo de conmutación por error para continuar con el envío de ejecuciones. Si ha configurado los recursos de las áreas de trabajo principal y secundaria para que compartan recursos asociados con la replicación geográfica habilitada, puede que algunos objetos estén disponibles directamente en el área de trabajo de conmutación por error. Este es el caso, por ejemplo, si ambas áreas de trabajo comparten las mismas imágenes de Docker, almacenes de datos configurados y recursos de Azure Key Vault. En el diagrama siguiente se muestra una configuración en la que dos áreas de trabajo comparten las mismas imágenes (1), almacenes de datos (2) y recursos de Key Vault (3).

![Referenciación a la configuración de recursos](./media/how-to-high-availability-machine-learning/bcdr-resource-configuration.png)

> [!NOTE]
> Cualquier trabajo que se esté ejecutando cuando se produzca una interrupción del servicio no pasarán automáticamente al área de trabajo secundaria. Además es poco probable que los trabajos se reanuden y finalicen correctamente en el área de trabajo principal una vez resuelta la interrupción. En su lugar, los trabajos se deben volver a enviar, ya sea en el área de trabajo secundaria o en la principal (una vez resuelta la interrupción).

### <a name="moving-artifacts-between-workspaces"></a>Movilidad de artefactos entre áreas de trabajo

En función del enfoque de recuperación, puede que tenga que copiar artefactos, como objetos de conjunto de datos y de modelo, entre las áreas de trabajo para seguir trabajando. Actualmente, la movilidad de artefactos entre áreas de trabajo es limitada. Se recomienda administrar los artefactos como código siempre que sea posible para que se puedan volver a crear en la instancia de conmutación por error.

Los artefactos siguientes se pueden exportar e importar entre áreas de trabajo mediante la [extensión de la CLI de Azure para el aprendizaje automático](reference-azure-machine-learning-cli.md):

| Artefacto | Exportación | Importar |
| ----- | ----- | ----- |
| Modelos | [az ml model download --model-id {ID} --target-dir {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model#ext_azure_cli_ml_az_ml_model_download) | [az ml model register –name {NAME} --path {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model) |
| Entornos | [az ml environment download -n {NAME} -d {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment#ext_azure_cli_ml_az_ml_environment_download) | [az ml environment register -d {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment#ext_azure_cli_ml_az_ml_environment_register) |
| Canalizaciones de Azure ML (generadas por código) | [az ml pipeline get --path {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/pipeline#ext_azure_cli_ml_az_ml_pipeline_get) | [az ml pipeline create --name {NAME} -y {PATH}](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/pipeline#ext_azure_cli_ml_az_ml_pipeline_create)

> [!TIP]
> * Los __conjunto de datos registrados__ no se pueden descargar ni cambiar de sitio. Se incluyen los conjuntos de datos generados por Azure ML, como los conjuntos de datos de canalización intermedia. Sin embargo, los conjuntos de datos que hagan referencia a una ubicación de archivos compartidos a la que pueden acceder ambas áreas de trabajo, o en donde se replica el almacenamiento de datos subyacente, se pueden registrar en ambas áreas de trabajo. Use [az ml dataset register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/dataset#ext_azure_cli_ml_az_ml_dataset_register) para registrar un conjunto de datos.
>
> * Las __salidas de ejecución__ se almacenan en la cuenta de almacenamiento predeterminada asociada a un área de trabajo. Aunque las salidas de ejecución pueden quedar inaccesibles desde la interfaz de usuario de Studio en caso de una interrupción del servicio, se puede acceder directamente a los datos a través de la cuenta de almacenamiento. Para obtener más información sobre cómo trabajar con datos almacenados en blobs, consulte [Creación, descarga y enumeración de blobs mediante la CLI de Azure](../storage/blobs/storage-quickstart-blobs-cli.md).
## <a name="next-steps"></a>Pasos siguientes

Para implementar Azure Machine Learning con los recursos asociados con la configuración de alta disponibilidad, use una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/machine-learning-advanced).
