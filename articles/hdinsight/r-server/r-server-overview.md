---
title: Introducción a ML Services en Azure HDInsight
description: Aprenda a usar ML Services en HDInsight para crear aplicaciones para al análisis de macrodatos.
ms.service: hdinsight
ms.topic: overview
ms.date: 04/20/2020
ROBOTS: NOINDEX
ms.openlocfilehash: 0baf100d7d3a3e6a0667ed00176fa533eafe6d57
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112295964"
---
# <a name="what-is-ml-services-in-azure-hdinsight"></a>¿Qué es ML Services en Azure HDInsight?

[!INCLUDE [retirement banner](../includes/ml-services-retirement.md)]

Microsoft Machine Learning Server está disponible como opción de implementación al crear clústeres de HDInsight en Azure. El tipo de clúster que proporciona esta opción se denomina **ML Services**. Esta funcionalidad proporciona acceso a petición a métodos de análisis adaptables y distribuidos en HDInsight.

ML Services en HDInsight proporciona las funcionalidades más recientes para el análisis basado en R en conjuntos de datos de prácticamente cualquier tamaño. Los conjuntos de datos se pueden cargar en un blob de Azure o en el almacenamiento de Data Lake. Las aplicaciones basadas en R pueden usar los más de 8000 paquetes de R de código abierto. También están disponibles rutinas de ScaleR, un paquete de análisis de macrodatos de Microsoft.

El nodo perimetral proporciona un lugar conveniente para conectarse al clúster y ejecutar los scripts de R. El nodo perimetral permite ejecutar funciones distribuidas en paralelo de ScaleR en los núcleos del servidor. También puede ejecutarlas en los nodos del clúster mediante Hadoop MapReduce de ScaleR. Asimismo, puede usar contextos de proceso de Apache Spark.

Los modelos o las predicciones que se generen a partir de los análisis pueden descargarse y utilizarse en el entorno local. También se puede aplicar la operación `operationalized` en otra área de Azure. En concreto, mediante [Azure Machine Learning Studio (clásico)](https://studio.azureml.net) y [servicios web](../../machine-learning/classic/deploy-a-machine-learning-web-service.md).

## <a name="get-started-with-ml-services-on-hdinsight"></a>Introducción a ML Services en HDInsight

Para crear un clúster de ML Services en HDInsight, seleccione el tipo de clúster **ML Services**. El tipo de clúster de ML Services incluye ML Server en los nodos de datos y en el nodo perimetral. El nodo perimetral actúa como zona de aterrizaje para los análisis basados en ML Services. Consulte [Creación de clústeres basados en Linux en HDInsight mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md) para ver un tutorial sobre cómo crear el clúster.

## <a name="why-choose-ml-services-in-hdinsight"></a>Razones para elegir ML Services en HDInsight

ML Services en HDInsight proporciona las siguientes ventajas:

### <a name="ai-innovation-from-microsoft-and-open-source"></a>Las innovaciones en inteligencia artificial que ofrecen tanto Microsoft como el código abierto

  ML Services incluye un conjunto de algoritmos muy adaptable y distribuido, como [RevoscaleR](/machine-learning-server/r-reference/revoscaler/revoscaler), [revoscalepy](/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) y [microsoftML](/machine-learning-server/python-reference/microsoftml/microsoftml-package). Estos algoritmos pueden funcionar con tamaños de datos mayores que el tamaño de la memoria física. También se ejecutan en una amplia variedad de plataformas de una manera distribuida. Obtenga más información acerca de la colección de [paquetes de R](/machine-learning-server/r-reference/introducing-r-server-r-package-reference) y [paquetes de Python](/machine-learning-server/python-reference/introducing-python-package-reference) personalizados de Microsoft que se incluyen con el producto.
  
  ML Services aúna las innovaciones de Microsoft y las contribuciones de la comunidad de código abierto (kits de herramientas de R, Python e inteligencia artificial). Y todo sobre una única plataforma de clase empresarial. Cualquier paquete de aprendizaje automático de código abierto de R o Python puede utilizarse simultáneamente con las innovaciones de Microsoft.

### <a name="simple-secure-and-high-scale-operationalization-and-administration"></a>Operacionalización y administración simples, seguras y a gran escala

  Las empresas que utilizan entornos y paradigmas tradicionales invierten mucho tiempo y esfuerzo en la operacionalización, lo que aumenta los costos y retrasos en la conversión de modelos, iteraciones necesarias para mantener su validez y vigencia, aprobaciones normativas y administración de permisos.

  ML Services ofrece [operacionalización](/machine-learning-server/what-is-operationalization) de clase empresarial de ML Services. Una vez que se completa un modelo de aprendizaje automático, solo se necesitan unos pocos clics para generar las API de servicios web. Estos [servicios web](/machine-learning-server/operationalize/concept-what-are-web-services) se hospedan en una malla de servidores de la nube y pueden integrarse con aplicaciones de línea de negocio. Disponer de una malla flexible en la que realizar las implementaciones le brinda la capacidad de escalar el sistema fácilmente en función de las necesidades del negocio, tanto en las puntuaciones por lotes como en las puntuaciones en tiempo real. Para obtener instrucciones, consulte [Operationalize ML Services on HDInsight](r-server-operationalize.md) (Operacionalización de ML Services en HDInsight).

<!---
* **Deep ecosystem engagements to deliver customer success with optimal total cost of ownership**

  Individuals embarking on the journey of making their applications intelligent or simply wanting to learn the new world of AI and machine learning, need the right resources to help them get started. In addition to this documentation, Microsoft provides several learning resources and has engaged several training partners to help you ramp up and become productive quickly.
--->

> [!NOTE]  
> El tipo de clúster de ML Services en HDInsight solo se admite en HDInsight 3.6. La retirada de HDInsight 3.6 está programada para el 31 de diciembre de 2020.

## <a name="key-features-of-ml-services-on-hdinsight"></a>Características principales de ML Services en HDInsight

Las siguientes características se incluyen con ML Services en HDInsight.

| Categoría de la característica | Descripción |
|------------------|-------------|
| Habilitado para R | [Paquetes de R](/machine-learning-server/r-reference/introducing-r-server-r-package-reference) para soluciones escritas en R, con una distribución de código abierto de R y una infraestructura en tiempo de ejecución para la ejecución de scripts. |
| Habilitado para Python | [Módulos de Python](/machine-learning-server/python-reference/introducing-python-package-reference) para soluciones escritas en Python, con una distribución de código abierto de Python y una infraestructura en tiempo de ejecución para la ejecución de scripts.
| [Modelos previamente entrenados](/machine-learning-server/install/microsoftml-install-pretrained-models) | Para análisis visuales y análisis textuales de sentimientos, preparados para puntuar los datos proporcionados. |
| [Implementación y consumo](r-server-operationalize.md) | Aplique la operación `Operationalize` al servidor e implemente las soluciones como un servicio web. |
| [Ejecución remota](r-server-hdinsight-manage.md#connect-remotely-to-microsoft-ml-services) | Inicie sesiones remotas en el clúster ML Services de la red desde la estación de trabajo del cliente. |

## <a name="data-storage-options-for-ml-services-on-hdinsight"></a>Opciones de almacenamiento de datos para ML Services en HDInsight

El almacenamiento predeterminado del sistema de archivos HDFS puede ser una cuenta de Azure Storage o Azure Data Lake Storage. Los datos cargados en el almacenamiento del clúster durante el análisis se hacen persistentes. Los datos están disponibles incluso después de que se elimina el clúster. Varias herramientas pueden controlar la transferencia de datos al almacenamiento. Las herramientas incluyen el servicio de carga basado en el portal de la cuenta de almacenamiento y la utilidad AzCopy.

Puede habilitar el acceso a almacenes de blobs y Data Lake adicionales durante la creación del clúster. No está limitado por la opción de almacenamiento principal en uso.  Para más información acerca del uso de varias cuentas de almacenamiento, consulte el artículo [Opciones de Azure Storage para ML Services en HDInsight](./r-server-storage.md).

También se puede usar Azure Files como opción de almacenamiento para el nodo perimetral. Azure Files habilita los recursos compartidos de archivos creados en Azure Storage en el sistema de archivos de Linux. Para más información, consulte [Opciones de Azure Storage para ML Services en HDInsight](r-server-storage.md).

## <a name="access-ml-services-edge-node"></a>Acceder a un nodo perimetral de ML Services

Puede conectarse a Microsoft ML Server en el nodo perimetral mediante un explorador o SSH/PuTTY. La consola R se instala de forma predeterminada durante la creación del clúster.  

## <a name="develop-and-run-r-scripts"></a>Desarrollo y ejecución de scripts de R

Los scripts de R pueden usar cualquiera de los más de 8000 paquetes de R de código abierto. También puede usar las rutinas en paralelo y distribuidas de la biblioteca de ScaleR. Los scripts se ejecutan en el nodo perimetral que se ejecuta dentro del intérprete de R en ese nodo, excepto en el caso de los pasos que llaman a funciones de ScaleR con un contexto de proceso de Map Reduce (RxHadoopMR) o Spark (RxSpark). Las funciones se ejecutan de manera distribuida en los nodos de datos que están asociados a los datos. Para más información sobre las opciones de contexto, consulte [Opciones de contexto de proceso para ML Services en HDInsight](r-server-compute-contexts.md).

## <a name="operationalize-a-model"></a>Aplicación de la operación `Operationalize` a un modelo

Cuando se complete el modelado de datos, podrá aplicar la operación `operationalize` al modelo para realizar predicciones con nuevos datos en Azure o en el entorno local. Este proceso se conoce como "puntuación". La puntuación puede realizarse en HDInsight, Azure Machine Learning o de forma local.

### <a name="score-in-hdinsight"></a>Puntuación en HDInsight

Para puntuar en HDInsight, escriba una función de R. La función llama al modelo para hacer predicciones con respecto a un nuevo archivo de datos que ha cargado en la cuenta de almacenamiento. Después, vuelva a guardar las predicciones en la cuenta de almacenamiento. Puede ejecutar esta rutina a petición en el nodo perimetral del clúster o mediante un trabajo programado.

### <a name="score-in-azure-machine-learning-aml"></a>Puntuación en Azure Machine Learning (AML)

Si desea realizar la puntuación mediante Azure Machine Learning, utilice el paquete de R de Azure Machine Learning de código abierto, conocido como [AzureML](https://cran.r-project.org/src/contrib/Archive/AzureML/), para publicar el modelo como un servicio web de Azure. Para mayor comodidad, este paquete está preinstalado en el nodo perimetral. Después, utilizando los recursos de Azure Machine Learning, cree una interfaz de usuario para el servicio web y, si es necesario, llame a dicho servicio web para efectuar las puntuaciones. Luego, convierta los objetos del modelo de ScaleR en objetos del modelo de código abierto equivalentes para usarlos con el servicio web. Para esta conversión, utilice las funciones de coerción de ScaleR, como `as.randomForest()`, para los modelos basados en conjuntos.

### <a name="score-on-premises"></a>Puntuación en un entorno local

Para puntuar en el entorno local después de crear el modelo, serialice el modelo en R, descárguelo, deserialícelo y, luego, úselo para puntuar nuevos datos. Puede puntuar nuevos datos por medio del enfoque descrito antes en Puntuación en HDInsight o mediante [servicios web](/machine-learning-server/operationalize/concept-what-are-web-services).

## <a name="maintain-the-cluster"></a>Mantenimiento del clúster

### <a name="install-and-maintain-r-packages"></a>Instalación y mantenimiento de paquetes de R

La mayoría de los paquetes de R que utiliza se necesitan en el nodo perimetral, puesto que es donde se ejecutan la mayoría de los scripts de R. Para instalar más paquetes de R en el nodo perimetral, puede usar el método `install.packages()` de R.

Si solo usa rutinas de la biblioteca de ScaleR, normalmente no necesita paquetes de R adicionales. Puede que necesite paquetes adicionales para la ejecución de **rxExec** o **RxDataStep** en los nodos de datos.

Los paquetes adicionales pueden instalarse con una acción de script después de crear el clúster. Para más información, consulte [Administración de ML Services en un clúster de HDInsight](r-server-hdinsight-manage.md).

### <a name="change-apache-hadoop-mapreduce-memory-settings"></a>Cambio de la configuración de memoria de Apache Hadoop MapReduce

La memoria disponible para ML Services se puede modificar cuando se ejecuta un trabajo de MapReduce. Para modificar un clúster, use la interfaz de usuario de Apache Ambari para el clúster. Para instrucciones sobre la interfaz de usuario de Ambari, consulte [Administración de clústeres de HDInsight con la interfaz de usuario web de Apache Ambari](../hdinsight-hadoop-manage-ambari.md).

La memoria disponible para ML Services se puede cambiar mediante conmutadores de Hadoop en la llamada a **RxHadoopMR**:

```r
hadoopSwitches = "-libjars /etc/hadoop/conf -Dmapred.job.map.memory.mb=6656"
```

### <a name="scale-your-cluster"></a>Escalado del clúster

Los clústeres de ML Services que existen en HDInsight se pueden escalar o reducir verticalmente desde el portal. Mediante el escalado vertical, se obtiene capacidad adicional para tareas de procesamiento de mayor tamaño. Puede volver a escalar un clúster cuando esté inactivo. Para ver instrucciones sobre cómo escalar un clúster, consulte [Administración de clústeres de HDInsight](../hdinsight-administer-use-portal-linux.md).

### <a name="maintain-the-system"></a>Mantenimiento del sistema

El mantenimiento del sistema operativo se realiza en las máquinas virtuales Linux subyacentes de un clúster de HDInsight fuera del horario laboral. Normalmente, el mantenimiento se realiza a las 3:30 de la madrugada (hora local de la máquina virtual) todos los lunes y jueves. Las actualizaciones no afectan a más de una cuarta parte del clúster a la vez.

Los trabajos en ejecución pueden ralentizarse durante el mantenimiento. No obstante, deben seguir ejecutándose hasta que finalicen. Cualquier software personalizado o dato local que haya instalado se conserva en estos eventos de mantenimiento, salvo que se produzca un error irrecuperable que requiera recompilar el clúster.

## <a name="ide-options-for-ml-services-on-hdinsight"></a>Opciones IDE para ML Services en HDInsight

El nodo perimetral de Linux de un clúster de HDInsight es la zona de aterrizaje de los análisis basados en R. Las versiones recientes de HDInsight proporcionan un IDE basado en el explorador en el nodo perimetral. El servidor de RStudio es más productivo que la consola de R para las tareas de desarrollo y ejecución.

Un IDE de escritorio puede acceder al clúster a través de un contexto de proceso remoto de MapReduce o Spark. Las opciones incluyen: [Herramientas de R para Visual Studio](https://marketplace.visualstudio.com/items?itemName=MikhailArkhipov007.RTVS2019) (RTVS) de Microsoft, RStudio y StatET basado en Eclipse de Walware.

Para acceder a la consola de R en el nodo perimetral, escriba **R** en el símbolo del sistema. Cuando se usa la interfaz de la consola, es conveniente desarrollar el script de R en un editor de texto. A continuación, corte y pegue secciones del script en la consola de R según sea necesario.

## <a name="pricing"></a>Precios

Los precios asociados a un clúster de HDInsight para ML Services están estructurados de forma similar a otros tipos de clúster de HDInsight. Se basan en el tamaño de las máquinas virtuales subyacentes en los nodos de nombre, datos y perimetral. También se eleva la hora del núcleo. Para más información, vea los [precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar clústeres de ML Services en HDInsight, consulte los artículos siguientes:

* [Ejecución de un script de R en un clúster de Machine Learning Services en Azure HDInsight con RStudio Server](machine-learning-services-quickstart-job-rstudio.md)
* [Opciones de contexto de proceso para un clúster de ML Services en HDInsight](r-server-compute-contexts.md)
* [Opciones de Storage para un clúster ML Services en HDInsight](r-server-storage.md)