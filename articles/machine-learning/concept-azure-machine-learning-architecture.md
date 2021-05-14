---
title: Arquitectura y conceptos clave
titleSuffix: Azure Machine Learning
description: En este artículo se proporciona una descripción de alto nivel de la arquitectura, los términos y los conceptos que componen Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 08/20/2020
ms.custom: seoapril2019, seodec18
ms.openlocfilehash: f1eb7a5b4697801775d23091c610ab594b0b27ec
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813387"
---
# <a name="how-azure-machine-learning-works-architecture-and-concepts"></a>Funcionamiento de Azure Machine Learning: Arquitectura y conceptos

Obtenga información sobre la arquitectura y los conceptos de [Azure Machine Learning](overview-what-is-azure-ml.md).  En este artículo se ofrece una descripción general de los componentes y cómo funcionan juntos para ayudar en el proceso de creación, implementación y mantenimiento de modelos de Machine Learning.

## <a name="workspace"></a><a name="workspace"></a> Área de trabajo

Un [área de trabajo de aprendizaje automático](concept-workspace.md) es el recurso de nivel superior de Azure Machine Learning.

:::image type="content" source="media/concept-azure-machine-learning-architecture/architecture.svg" alt-text="Diagrama: arquitectura de Azure Machine Learning de un área de trabajo y sus componentes":::

El área de trabajo es el lugar centralizado para:

* Administrar los recursos que se usan para entrenar e implementar modelos, como [procesos](#compute-instance)
* Almacenar los recursos creados al usar Azure Machine Learning, incluidos:
  * [Entornos](#environments)
  * [Experimentos](#experiments)
  * [Canalizaciones](#ml-pipelines)
  * [Conjuntos de datos](#datasets-and-datastores)
  * [Models](#models)
  * [Extremos](#endpoints)

Un área de trabajo incluye otros recursos de Azure que se usan en el área de trabajo:

+ [Azure Container Registry (ACR)](https://azure.microsoft.com/services/container-registry/): registra los contenedores de docker que usa durante el entrenamiento y al implementar un modelo. Para minimizar los costos, la instancia de ACR solo se crea cuando se crean las imágenes de la implementación.
+ [Cuenta de Azure Storage](https://azure.microsoft.com/services/storage/): se usa como almacén de datos predeterminado para el área de trabajo.  Las instancias de Jupyter Notebook que se usan con las instancias de proceso de Azure Machine Learning también se almacenan aquí.
+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/): almacena información sobre la supervisión de los modelos.
+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/): almacena secretos que usan los destinos de proceso y otra información confidencial que el área de trabajo necesita.

Puede compartir un área de trabajo con otros usuarios.

## <a name="computes"></a>Procesos

<a name="compute-targets"></a> Un [destino de proceso](concept-compute-target.md) es cualquier máquina o conjunto de máquinas que use para ejecutar el script de entrenamiento u hospedar la implementación del servicio. Puede usar su equipo local o un recurso de proceso remoto como destino de proceso.  Con los destinos de proceso, puede iniciar el entrenamiento en el equipo local y, a continuación, escalar horizontalmente a la nube sin cambiar el script de entrenamiento.

Azure Machine Learning presenta dos máquinas virtuales (VM) basadas en la nube totalmente administrada que se configuran para tareas de aprendizaje automático:

* <a name="compute-instance"></a> **Instancia de proceso**: una instancia de proceso es una máquina virtual que incluye varias herramientas y entornos instalados para el aprendizaje automático. El uso principal de una instancia de proceso es para la estación de trabajo de desarrollo.  Puede empezar a ejecutar cuadernos de ejemplo sin necesidad de una configuración. Una instancia de proceso también se puede usar como destino de proceso para los trabajos de entrenamiento e inferencia.

* **Clústeres de proceso**: los clústeres de proceso son un clúster de máquinas virtuales con funcionalidades de escalado de varios nodos. Los clústeres de proceso son más prácticos para los destinos de proceso de producción y trabajos grandes.  El clúster se escala verticalmente de manera automática cuando se envía un trabajo.  Úselo como destino de proceso de entrenamiento o para la implementación de desarrollo/pruebas.

Para más información sobre los destinos de proceso de entrenamiento, vea [Entrenamiento de destinos de proceso](concept-compute-target.md#train).  Para más información sobre los destinos de proceso de implementación, vea [Destinos de implementación](concept-compute-target.md#deploy).

## <a name="datasets-and-datastores"></a>Conjuntos de datos y almacenes de datos

Los [**Conjuntos de datos de Azure Machine Learning**](concept-data.md#datasets) hacen que sea más sencillo acceder a los datos y trabajar con ellos. Mediante la creación de un conjunto de datos, puede crear una referencia a la ubicación del origen de datos, junto con una copia de sus metadatos. Dado que los datos permanecen en su ubicación existente, no incurre en ningún costo de almacenamiento adicional ni arriesga la integridad de los orígenes de datos.

Para obtener más información, consulte [Creación y registro de los conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md).  Para ver más ejemplos del uso de los conjuntos de datos, consulte los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/datasets-tutorial).

Los conjuntos de datos usan [almacenes de datos](concept-data.md#datastores) para conectarse de forma segura a los servicios de Azure Storage. Los almacenes de datos almacenan información de conexión sin poner en riesgo sus credenciales de autenticación ni la integridad de su origen de datos original. Almacenan información de conexión, como el identificador de suscripción y la autorización de token de su instancia de Key Vault asociada con el área de trabajo, para que pueda acceder de forma segura al almacenamiento sin tener que codificarlos de forma rígida en el script.

## <a name="environments"></a>Entornos

[Área de trabajo](#workspace) > **Entornos**

Un [entorno](concept-environments.md) es la encapsulación del lugar en el que se produce el entrenamiento o la puntuación del modelo de Machine Learning. El entorno especifica los paquetes, las variables de entorno y la configuración de software de Python en torno a los scripts de entrenamiento y puntuación.  

Para ver ejemplos de código, consulte la sección "Administración de entornos" de [Uso de entornos](how-to-use-environments.md#manage-environments).

## <a name="experiments"></a>Experimentos

[Área de trabajo](#workspace) > **Experimentos**

Un experimento es una agrupación de varias ejecuciones de un script determinado. Siempre pertenece a un área de trabajo. Cuando envíe una ejecución, proporcione un nombre de experimento. La información de la ejecución se almacena en ese experimento. Si el nombre no existe al enviar un experimento, este último se crea automáticamente.
  
Para obtener un ejemplo del uso de un experimento, consulte [Tutorial: Entrenamiento del primer modelo](tutorial-1st-experiment-sdk-train.md).

### <a name="runs"></a>Ejecuciones

[Área de trabajo](#workspace) > [Experimentos](#experiments) > **Ejecutar**

Una ejecución hace referencia a una única ejecución de un script de entrenamiento. Un experimento normalmente contendrá varias ejecuciones.

Azure Machine Learning registra todas las ejecuciones y almacena la siguiente información en el experimento:

* Metadatos sobre la ejecución (marca de tiempo, duración, etc.).
* Métricas que el script registra.
* Archivos de salida que recopila automáticamente el experimento o que carga explícitamente usted mismo.
* Instantánea del directorio que contiene los scripts antes de la ejecución.

Una ejecución se produce cuando se envía un script para entrenar un modelo. Una ejecución puede tener cualquier número de ejecuciones secundarias. Por ejemplo, la ejecución de nivel superior puede tener dos ejecuciones secundarias, cada una de las cuales puede tener sus propias ejecuciones secundarias.

### <a name="run-configurations"></a>Configuraciones de ejecución

[Área de trabajo](#workspace) > [Experimentos](#experiments) > [Ejecutar](#runs) > **Configuración de ejecución**

Una configuración de ejecución define cómo se debe ejecutar un script en un destino de proceso especificado. La configuración se usa para especificar el script, el destino de proceso y el entorno de aprendizaje automático de Azure en el que se ejecutará, cualquier configuración específica del trabajo distribuido y algunas propiedades adicionales. Para más información sobre el conjunto completo de opciones configurables para las ejecuciones, consulte [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig).

Una configuración de ejecución puede conservarse en un archivo dentro del directorio que contiene el script de entrenamiento.   O bien, se puede crear como un objeto en memoria y usarse para enviar una ejecución.

Para encontrar ejemplos de configuraciones de ejecución, consulte [Configuración de una ejecución de entrenamiento](how-to-set-up-training-targets.md).

### <a name="snapshots"></a>Instantáneas

[Área de trabajo](#workspace) > [Experimentos](#experiments) > [Ejecutar](#runs) > **Instantánea**

Al enviar una ejecución, Azure Machine Learning comprime el directorio que contiene el script como un archivo zip y lo envía al destino de proceso. A continuación, el archivo .zip se extrae y el script se ejecuta. Azure Machine Learning también almacena el archivo .zip como una instantánea como parte del registro de ejecución. Cualquier persona con acceso al área de trabajo puede buscar un registro de ejecución y descargar la instantánea.

### <a name="logging"></a>Registro

Azure Machine Learning registra automáticamente las métricas de ejecución estándar. Sin embargo, también puede [usar el SDK de Python para registrar métricas arbitrarias](how-to-log-view-metrics.md).

Hay varias maneras de ver los registros: supervisar el estado de ejecución en tiempo real o ver los resultados después de la finalización. Para obtener más información, consulte [Supervisión y visualización de registros de ejecución de ML](how-to-log-view-metrics.md).


> [!NOTE]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]

### <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Esto funciona con las ejecuciones enviadas mediante una configuración de ejecución de script o una canalización de aprendizaje automático. También funciona para las ejecuciones enviadas desde el SDK o la CLI de Machine Learning.

Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

### <a name="training-workflow"></a>Flujo de trabajo de entrenamiento

Al ejecutar un experimento para entrenar un modelo, se realizan los pasos siguientes. Estos están ilustrados en el diagrama del flujo de trabajo de entrenamiento siguiente:

* Para llamar a Azure Machine Learning se usa el identificador de la instantánea de código guardada en la sección anterior.
* Azure Machine Learning crea un identificador de ejecución (opcional) y un token de Machine Learning Service, que más adelante se usan en los destinos de proceso, como Proceso de Machine Learning o las máquinas virtuales, para comunicarse con este servicio.
* Puede elegir un destino de proceso administrado (como Proceso de Machine Learning) o un destino de proceso no administrado (como máquinas virtuales) para ejecutar los trabajos de entrenamiento. Estos son los flujos de datos de ambos escenarios:
   * Máquinas virtuales/HDInsight: se accede a ellos mediante las credenciales de SSH de un almacén de claves de la suscripción de Microsoft. Azure Machine Learning ejecuta código de administración en el destino de proceso que:

   1. Prepara el entorno. (Docker es una opción para máquinas virtuales y equipos locales. Consulte los siguientes pasos de Proceso de Machine Learning para entender cómo funciona la ejecución de experimentos en el contenedor de Docker).
   1. Descarga el código.
   1. Configura las variables de entorno y realiza las configuraciones.
   1. Ejecuta scripts de usuario (la instantánea de código mencionada en la sección anterior).

   * Proceso de Machine Learning, al que se accede mediante una identidad administrada por el área de trabajo.
Dado que Proceso de Machine Learning es un destino de proceso administrado (es decir, está administrado por Microsoft), se ejecuta en la suscripción de Microsoft.

   1. Si es necesario, pone en marcha la construcción de un Docker remoto.
   1. El código de administración se escribe en el recurso compartido de Azure Files del usuario.
   1. El contenedor se inicia con un comando inicial. Es decir, el código de administración que se describe en el paso anterior.

* Una vez finalizada la ejecución, puede consultar las ejecuciones y métricas. En el diagrama de flujo siguiente, este paso se produce cuando el destino de proceso de entrenamiento escribe las métricas de ejecución de nuevo en Azure Machine Learning desde el almacenamiento de la base de datos de Cosmos DB. Los clientes pueden llamar a Azure Machine Learning. Machine Learning, a su vez, extrae las métricas de la base de datos de Cosmos DB y las devuelve al cliente.

[![Flujo de trabajo de entrenamiento](media/concept-azure-machine-learning-architecture/training-and-metrics.png)](media/concept-azure-machine-learning-architecture/training-and-metrics.png#lightbox)

## <a name="models"></a>Modelos

En su forma más simple, un modelo es un fragmento de código que toma una entrada y genera una salida. El hecho de crear un modelo de Machine Learning implica seleccionar un algoritmo, proporcionarle datos y [ajustar los hiperparámetros](how-to-tune-hyperparameters.md). El entrenamiento es un proceso iterativo que genera un modelo entrenado, que encapsula lo que el modelo ha aprendido durante el proceso de aprendizaje.

Puede importar un modelo que se haya entrenado fuera de Azure Machine Learning. O bien, puede entrenar un modelo mediante el envío de una [ejecución](#runs) de [experimento](#experiments) a un [destino de proceso](#compute-targets) en Azure Machine Learning. Una vez que tenga un modelo, [regístrelo](#register-model) en el área de trabajo.

Azure Machine Learning es independiente del marco de trabajo. Al crear un modelo, puede usar cualquier marco de aprendizaje automático popular, como Scikit-learn, XGBoost, PyTorch, TensorFlow y Chainer.

Para ver un ejemplo de cómo entrenar un modelo con Scikit-learn, consulte [Tutorial: Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).


### <a name="model-registry"></a><a name="register-model"></a> Registro de modelos

[Área de trabajo](#workspace) > **Modelos**

El **registro de modelos** permite realizar un seguimiento de todos los modelos del área de trabajo de Azure Machine Learning.

Los modelos se identifican por el nombre y la versión. Cada vez que registra un modelo con el mismo nombre que uno ya existente, el registro interpreta que se trata de una nueva versión. La versión se incrementa y el nuevo modelo se registra con el nombre en cuestión.

Una vez registrado el modelo, puede proporcionar etiquetas de metadatos adicionales y, después, usar esas etiquetas al buscar modelos.

> [!TIP]
> Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo de Azure Machine Learning. Después del registro, puede descargar o implementar el modelo registrado y recibir todos los archivos que se registraron.

No se puede eliminar un modelo registrado que esté usando una implementación activa.

Para obtener un ejemplo de registro de un modelo, consulte [Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

## <a name="deployment"></a>Implementación

Implemente un [modelo registrado](#register-model) como un punto de conexión de servicio. Necesita los siguientes componentes:

* **Entorno**. Este entorno encapsula las dependencias necesarias para ejecutar el modelo para la inferencia.
* **Código de puntuación**. Este script acepta solicitudes, puntúa las solicitudes mediante el modelo y devuelve los resultados.
* **Configuración de inferencia**. La configuración de inferencia especifica el entorno, el script de entrada y otros componentes necesarios para ejecutar el modelo como un servicio.

Para más información sobre estos componentes, vea [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

### <a name="endpoints"></a>Puntos de conexión

[Área de trabajo](#workspace) > **Puntos de conexión**

Un punto de conexión es la creación de una instancia del modelo en un servicio web que puede hospedarse en la nube o un módulo de IoT para las implementaciones de dispositivos integrados.

#### <a name="web-service-endpoint"></a>Punto de conexión de servicio web

Al implementar un modelo como un servicio web, el punto de conexión se puede implementar en Azure Container Instances, Azure Kubernetes Service o FPGA. El servicio se crea a partir del modelo, el script y los archivos asociados. Estos se colocan en una imagen de contenedor base que contiene el entorno de ejecución para el modelo. La imagen tiene un punto de conexión HTTP de carga equilibrada que recibe solicitudes de puntuación que se envían al servicio web.

Puede habilitar la telemetría de Application Insights o la telemetría del modelo para supervisar el servicio web. Solo usted puede acceder a los datos de telemetría.  Se almacenan en las instancias de Application Insights y de la cuenta de almacenamiento. Si ha habilitado el ajuste automático de escala, Azure ajustará automáticamente la escala de su implementación.

En el diagrama siguiente se muestra el flujo de trabajo de inferencia para un modelo implementado como un punto de conexión de servicio web:

Estos son los detalles:

* El usuario registra un modelo mediante un cliente, como el SDK de Azure Machine Learning.
* El usuario crea una imagen mediante un modelo, un archivo de puntuación y otras dependencias del modelo.
* La imagen de Docker se crea y se almacena en Azure Container Registry.
* El servicio web se implementa en el destino de proceso (Container Instances/AKS) mediante la imagen creada en el paso anterior.
* Los detalles de la solicitud de puntuación se almacenan en Application Insights, que se incluye en la suscripción del usuario.
* También se insertan datos de telemetría en la suscripción de Microsoft o Azure.

[![Flujo de trabajo de inferencia](media/concept-azure-machine-learning-architecture/inferencing.png)](media/concept-azure-machine-learning-architecture/inferencing.png#lightbox)


Para obtener un ejemplo de implementación de un modelo como servicio web, consulte [Implementación de un modelo de clasificación de imágenes en Azure Container Instances](tutorial-deploy-models-with-aml.md).

#### <a name="real-time-endpoints"></a>Puntos de conexión en tiempo real

Al implementar un modelo entrenado en el diseñador, puede [implementar el modelo como punto de conexión en tiempo real](tutorial-designer-automobile-price-deploy.md). Normalmente, un punto de conexión en tiempo real recibe una única solicitud a través del punto de conexión REST y devuelve una predicción en tiempo real. Esto contrasta con el procesamiento por lotes, que procesa varios valores a la vez y guarda los resultados en un almacén de datos después de la finalización.

#### <a name="pipeline-endpoints"></a>Puntos de conexión de canalización

Los puntos de conexión de canalización permiten llamar a las [canalizaciones de ML](#ml-pipelines) mediante programación a través de un punto de conexión REST. Los puntos de conexión de canalización permiten automatizar los flujos de trabajo de canalización.

Un punto de conexión de canalización es una colección de canalizaciones publicadas. Esta organización lógica le permite administrar y llamar a varias canalizaciones mediante el mismo punto de conexión. Cada canalización publicada en un punto de conexión de canalización tiene versiones. Puede seleccionar una canalización predeterminada para el punto de conexión o especificar una versión en la llamada REST.
 

#### <a name="iot-module-endpoints"></a>Puntos de conexión del módulo de IoT

Un punto de conexión del módulo de IoT implementado es un contenedor de Docker que incluye el modelo y el script asociado o la aplicación y las dependencias adicionales. Estos módulos se implementan con Azure IoT Edge en dispositivos Edge.

Si ha habilitado la supervisión, Azure recopila datos de telemetría desde el modelo que está en el módulo de Azure IoT Edge. Solo usted puede obtener acceso a los datos de telemetría, que se almacenan en las instancias de su cuenta de almacenamiento.

Azure IoT Edge garantiza que el módulo se esté ejecutando y supervisa el dispositivo que lo hospeda. 
## <a name="automation"></a>Automation

### <a name="azure-machine-learning-cli"></a>CLI de Azure Machine Learning 

La [CLI de Azure Machine Learning](reference-azure-machine-learning-cli.md) es una extensión de la CLI de Azure, una interfaz de la línea de comandos multiplataforma para la plataforma Azure. Esta extensión proporciona comandos para automatizar las actividades de aprendizaje automático.

### <a name="ml-pipelines"></a>Canalizaciones de Machine Learning

Las [canalizaciones de aprendizaje automático](concept-ml-pipelines.md) se usan para crear y administrar flujos de trabajo que unen las fases de aprendizaje automático. Por ejemplo, una canalización podría incluir las fases de preparación de los datos, entrenamiento del modelo, implementación del modelo e inferencia y puntuación. Cada fase puede estar formada por varios pasos, cada uno de los cuales puede ejecutarse en modo desatendido en varios destinos de proceso. 

Los pasos de canalización se pueden reutilizar y se pueden ejecutar sin volver a ejecutar los pasos anteriores si la salida de esos pasos no ha cambiado. Por ejemplo, puede volver a entrenar un modelo sin volver a ejecutar los costosos pasos de preparación de datos si los datos no han cambiado. Las canalizaciones también permiten a los científicos de datos colaborar mientras trabajan en áreas independientes de un flujo de trabajo de Machine Learning.

## <a name="monitoring-and-logging"></a>Supervisión y registro

Azure Machine Learning proporciona las siguientes funcionalidades de supervisión y registro:

* Como __científico de datos__, puede supervisar los experimentos y la información de registro de las ejecuciones de entrenamiento. Para más información, consulte los siguientes artículos.
   * [Inicio, supervisión y cancelación de las ejecuciones de entrenamiento](how-to-track-monitor-analyze-runs.md)
   * [Métricas de registro de las ejecuciones de entrenamientos](how-to-log-view-metrics.md)
   * [Seguimiento de experimentos con MLflow](how-to-use-mlflow.md)
   * [Visualización de ejecuciones con TensorBoard](how-to-monitor-tensorboard.md)
* Como __administrador__, puede supervisar la información sobre el área de trabajo y los recursos de Azure relacionados, o eventos como la creación y eliminación de recursos mediante Azure Monitor. Para más información, consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).
* Como profesional de __DevOps__ o __MLOps__, puede supervisar la información generada por los modelos implementados como servicios web o módulos de IoT Edge para identificar problemas con las implementaciones y recopilar los datos enviados al servicio. Para más información, consulte [Recopilación de datos de modelos](how-to-enable-data-collection.md) y [Supervisión con Application Insights](how-to-enable-app-insights.md).

## <a name="interacting-with-your-workspace"></a>Interacción con el área de trabajo

### <a name="studio"></a>Estudio

[Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md) proporciona una vista web de todos los artefactos del área de trabajo.  Puede ver los resultados y los detalles de los conjuntos de datos, los experimentos, las canalizaciones, los modelos y los puntos de conexión.  También puede administrar los recursos de proceso y los almacenes de datos en Studio.

Studio también es el lugar desde el que se puede acceder a las herramientas interactivas que forman parte de Azure Machine Learning:

+ [Diseñador de Azure Machine Learning](concept-designer.md) para realizar los pasos del flujo de trabajo sin escribir código
+ Experiencia web para el [aprendizaje automático automatizado](concept-automated-ml.md)
+ [Cuadernos de Azure Machine Learning](how-to-run-jupyter-notebooks.md) para escribir y ejecutar su propio código en servidores integrados de Jupyter Notebook.
+ [Proyectos de etiquetado de datos](how-to-create-labeling-projects.md) para crear, administrar y supervisar proyectos para etiquetar los datos

### <a name="programming-tools"></a>Herramientas de programación

> [!IMPORTANT]
> Las herramientas marcadas (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+  Interactúe con el servicio en cualquier entorno de Python con el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).
+ Interactúe con el servicio en cualquier entorno de R con el [SDK de Azure Machine Learning para R](https://azure.github.io/azureml-sdk-for-r/reference/index.html) (versión preliminar).
+ Use el [diseñador de Azure Machine Learning](concept-designer.md) para seguir los pasos del flujo de trabajo sin escribir código. 
+ Use la [CLI de Azure Machine Learning](./reference-azure-machine-learning-cli.md) para la automatización.
+ El [Acelerador de soluciones Many Models](https://aka.ms/many-models) (versión preliminar) se basa en Azure Machine Learning y permite entrenar, usar y administrar cientos o incluso miles de modelos de Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

Para una introducción a Azure Machine Learning, consulte:

* [¿Qué es Azure Machine Learning?](overview-what-is-azure-ml.md)
* [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md)
* [Tutorial (parte 1): Entrenamiento de un modelo](tutorial-train-models-with-aml.md)