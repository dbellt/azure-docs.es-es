---
title: ¿Qué es una instancia de proceso de Azure Machine Learning?
titleSuffix: Azure Machine Learning
description: Más información sobre la instancia de proceso Azure Machine Learning, una estación de trabajo totalmente administrada basada en la nube.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 10/02/2020
ms.openlocfilehash: a3677e50d9dab99eaedc88cdd61e8e2ed9a3761b
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321760"
---
# <a name="what-is-an-azure-machine-learning-compute-instance"></a>¿Qué es una instancia de proceso de Azure Machine Learning?

Una instancia de proceso de Azure Machine Learning es una estación de trabajo administrada basada en la nube para científicos de datos.

Las instancias de proceso permiten que sea fácil comenzar a desarrollar con Azure Machine Learning y proporcionan funcionalidades de administración y preparación empresarial para los administradores de TI.  

Use una instancia de proceso como el entorno de desarrollo completamente configurado y administrado en la nube para el aprendizaje automático. También se pueden usar como destino de proceso para el entrenamiento y la inferencia con fines de desarrollo y pruebas.  

Para el entrenamiento del modelo de nivel de producción, use un [clúster de proceso de Azure Machine Learning](how-to-create-attach-compute-cluster.md) con funcionalidades de escalado de varios nodos. Para la implementación del modelo de calidad de producción, use el [clúster de Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

Para que la funcionalidad de Jupyter de instancia de proceso haga su trabajo, asegúrese de que la comunicación de socket web no esté deshabilitada. Asegúrese de que la red permita las conexiones WebSocket a *.instances.azureml.net e *.instances.azureml.ms.

## <a name="why-use-a-compute-instance"></a>¿Por qué usar una instancia de proceso?

Una instancia de proceso es una estación de trabajo totalmente administrada basada en la nube optimizada para el entorno de desarrollo de aprendizaje automático. Proporciona las siguientes ventajas:

|Ventajas principales|Descripción|
|----|----|
|Productividad|Puede compilar e implementar modelos con cuadernos integrados y las siguientes herramientas en Azure Machine Learning Studio:<br/>-  Jupyter<br/>-  JupyterLab<br/>-  VS Code (versión preliminar)<br/>-RStudio (versión preliminar)<br/>La instancia de proceso está totalmente integrada con el área de trabajo y estudio de Azure Machine Learning. Puede compartir cuadernos y datos con otros científicos de datos en el área de trabajo.<br/> También puede usar [VS Code](https://techcommunity.microsoft.com/t5/azure-ai/power-your-vs-code-notebooks-with-azml-compute-instances/ba-p/1629630) con las instancias de proceso.
|Administrada y segura|Reduzca la superficie de seguridad y agregue compatibilidad con los requisitos de seguridad empresariales. Las instancias de proceso proporcionan directivas de administración sólidas y configuraciones de red seguras, como:<br/><br/>- Aprovisionamiento automático a partir de plantillas de Resource Manager o del SDK de Azure Machine Learning<br/>- [Control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md)<br/>- [Compatibilidad con redes virtuales](./how-to-secure-training-vnet.md#compute-instance)<br/>- Directiva SSH para habilitar o deshabilitar el acceso SSH<br/>TLS 1.2 habilitado |
|Preconfigurado &nbsp;para&nbsp;ML|Ahorre tiempo en las tareas de configuración con paquetes de ML preconfigurados y actualizados, marcos de aprendizaje profundo y controladores de GPU.|
|Totalmente personalizable|La amplia compatibilidad con los tipos de máquina virtual de Azure, incluidas las GPU y la personalización de bajo nivel persistente, como la instalación de paquetes y controladores, hace que los escenarios avanzados sean muy sencillos. |

Puede [crear una instancia de proceso](how-to-create-manage-compute-instance.md?tabs=python#create), o bien puede [crearla un administrador por usted](how-to-create-manage-compute-instance.md?tabs=python#create-on-behalf-of-preview).

## <a name="tools-and-environments"></a><a name="contents"></a>Herramientas y entornos

> [!IMPORTANT]
> Los elementos marcados (versión preliminar) en este artículo se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La estancia de proceso Azure Machine Learning permite crear, formar e implementar modelos en una experiencia de cuaderno completamente integrada en el área de trabajo.

Puede ejecutar cuadernos de Jupyter en [VS Code](https://techcommunity.microsoft.com/t5/azure-ai/power-your-vs-code-notebooks-with-azml-compute-instances/ba-p/1629630) mediante la instancia de proceso como el servidor remoto sin necesidad de SSH. También puede habilitar la integración de VS Code a través de la [extensión de SSH remota](https://devblogs.microsoft.com/python/enhance-your-azure-machine-learning-experience-with-the-vs-code-extension/).

Puede [instalar paquetes](how-to-access-terminal.md#install-packages) y [agregar kernels](how-to-access-terminal.md#add-new-kernels) a su instancia de proceso.  

Las herramientas y los entornos siguientes ya están instalados en la instancia de proceso: 

|Herramientas y entornos generales|Detalles|
|----|:----:|
|Controladores|`CUDA`</br>`cuDNN`</br>`NVIDIA`</br>`Blob FUSE` |
|Intel MPI Library||
|Azure CLI ||
|Ejemplos de Azure Machine Learning ||
|Docker||
|Nginx||
|NCCL 2.0 ||
|Protobuf|| 

|Herramientas y entornos de **R**|Detalles|
|----|:----:|
|Edición de código abierto de RStudio Server (versión preliminar)||
|R kernel||
|SDK de Azure Machine Learning para R|[azuremlsdk](https://azure.github.io/azureml-sdk-for-r/reference/index.html)</br>Ejemplos del SDK|

|Herramientas y entornos de **PYTHON**|Detalles|
|----|----|
|Anaconda Python||
|Jupyter y extensiones||
|Jupyterlab y extensiones||
[SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro)</br>desde PyPI|Incluye la mayoría de los paquetes adicionales de azureml.  Para ver la lista completa, [abra una ventana de terminal en la instancia de proceso](how-to-access-terminal.md) y ejecute <br/> `conda list -n azureml_py36 azureml*` |
|Otros paquetes de PyPI|`jupytext`</br>`tensorboard`</br>`nbconvert`</br>`notebook`</br>`Pillow`|
|Paquetes Conda|`cython`</br>`numpy`</br>`ipykernel`</br>`scikit-learn`</br>`matplotlib`</br>`tqdm`</br>`joblib`</br>`nodejs`</br>`nb_conda_kernels`|
|Paquetes de aprendizaje profundo|`PyTorch`</br>`TensorFlow`</br>`Keras`</br>`Horovod`</br>`MLFlow`</br>`pandas-ml`</br>`scrapbook`|
|Paquetes de ONNX|`keras2onnx`</br>`onnx`</br>`onnxconverter-common`</br>`skl2onnx`</br>`onnxmltools`|
|Ejemplos del SDK de Python y R para Azure Machine Learning||

Todos los paquetes de Python se instalan en el entorno de **Python 3.6: AzureML**.  

## <a name="accessing-files"></a>Acceso a archivos

Los cuadernos y los scripts de R se almacenan en la cuenta de almacenamiento predeterminada del área de trabajo en el recurso compartido de archivos de Azure.  Estos archivos se encuentran en el directorio "Archivos de usuario". Este almacenamiento facilita el uso compartido de cuadernos entre instancias de proceso. La cuenta de almacenamiento también evita que los cuadernos se conserven de forma segura al detener o eliminar una instancia de proceso.

La cuenta de recurso compartido de archivos de Azure del área de trabajo se monta como una unidad en la instancia de proceso. Esta unidad es el directorio de trabajo predeterminado para Jupyter, Jupyter Labs y RStudio. Esto significa que los cuadernos y otros archivos creados en Jupyter, JupyterLab o RStudio se almacenan automáticamente en el recurso compartido de archivos y están disponibles para usarse también en otras instancias de proceso.

Se puede tener acceso a los archivos del recurso compartido de archivos desde todas las instancias de proceso de la misma área de trabajo. Los cambios que se realicen en estos archivos en la instancia de proceso se conservarán de forma confiable en el recurso compartido de archivos.

También puede clonar los ejemplos de Azure Machine Learning más recientes en la carpeta situada en el directorio de archivos de usuario del recurso compartido de archivos del área de trabajo.

La escritura de archivos pequeños puede ser más lenta en las unidades de red que la escritura en el propio disco local de la instancia de proceso.  Si escribe muchos archivos pequeños, pruebe usar un directorio directamente en la instancia de proceso, como un directorio`/tmp`. Tenga en cuenta que estos archivos no serán accesibles desde otras instancias de proceso. 

Puede usar el directorio `/tmp` en la instancia de proceso para los datos temporales.  Pero no escriba archivos de datos muy grandes en el disco del sistema operativo de la instancia de proceso. El disco del sistema operativo de la instancia de proceso tiene una capacidad de 128 GB. Además, no almacene un conjunto grande de datos de entrenamiento en el recurso compartido de archivos de los cuadernos. Use [almacenes y conjuntos de datos](concept-azure-machine-learning-architecture.md#datasets-and-datastores) en su lugar. 

## <a name="managing-a-compute-instance"></a>Administración de una instancia de proceso

En el área de trabajo de Azure Machine Learning Studio, seleccione **Compute** (Proceso) y, luego, **Compute Instance** (Instancia de proceso) en la parte superior.

![Administración de una instancia de proceso](./media/concept-compute-instance/manage-compute-instance.png)

Puede realizar las siguientes acciones:

* [Crear una instancia de proceso](#create). 
* Actualizar la pestaña instancias de proceso.
* Iniciar, detener y reiniciar una instancia de proceso.  Se paga por la instancia cada vez que se ejecuta. Detenga la instancia de proceso cuando no la use para reducir el costo. Al detener una instancia de proceso, se cancela su asignación. A continuación, inícielo de nuevo cuando lo necesite. Tenga en cuenta que, al detenerse la instancia de proceso, se detiene la facturación de las horas de proceso, pero se le seguirá facturando por el disco, la dirección IP pública y el equilibrador de carga estándar.
* Elimine una instancia de proceso.
* Filtre la lista de instancias de proceso para mostrar solo las que haya creado.

Con cada instancia de proceso del área de trabajo a la que tenga acceso, puede:

* Acceder a Jupyter, JupyterLab y RStudio en la instancia de proceso.
* SSH en la instancia de proceso. El acceso SSH está deshabilitado de forma predeterminada, pero se puede habilitar en el momento de la creación de la instancia de proceso. El acceso SSH se realiza a través del mecanismo de claves pública y privada. La pestaña le proporcionará detalles sobre la conexión SSH, como la dirección IP, el nombre de usuario y el número de puerto.
* Más detalles sobre una instancia de proceso específica, como la dirección IP y la región.

[RBAC de Azure](../role-based-access-control/overview.md) permite controlar qué usuarios del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar una instancia de proceso. Todos los usuarios del rol colaborador y propietario del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar las instancias de proceso en el área de trabajo. Sin embargo, solo el creador de una instancia de proceso específica o el usuario asignado, si se creó en su nombre, tienen permiso para acceder a Jupyter, JupyterLab y RStudio en esa instancia de proceso. Una instancia de proceso está dedicada a un solo usuario que tiene acceso raíz y puede pasar por el terminal a través de Jupyter, JupyterLab o RStudio. La instancia de proceso incluirá el usuario que ha iniciado sesión y todas las acciones usarán la identidad de ese usuario para RBAC de Azure y la atribución de ejecuciones de experimentos. El acceso SSH se controla mediante un mecanismo de clave pública-privada.

RBAC de Azure puede controlar estas acciones:
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*

Para crear una instancia de proceso, debe tener permisos para realizar las siguientes acciones:
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/checkComputeNameAvailability/action*


### <a name="create-a-compute-instance"></a><a name="create"></a>Crear de una instancia de proceso

En el área de trabajo de Azure Machine Learning Studio, [cree una nueva instancia de proceso](how-to-create-attach-compute-studio.md#compute-instance) desde la sección **Proceso** o en la sección **Notebooks** cuando esté listo para ejecutar uno de sus cuadernos. 

Usted también puede crear una instancia
* Directamente de la [experiencia de cuadernos integrados](tutorial-1st-experiment-sdk-setup.md#azure)
* En Azure Portal
* A partir de una plantilla de Azure Resource Manager. Para ver una plantilla de ejemplo, consulte [Creación una plantilla de instancia de proceso de Azure Machine Learning](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance).
* Con [SDK de Azure Machine Learning](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/machine-learning/concept-compute-instance.md)
* Desde la extensión de la [CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md#computeinstance)

Los núcleos dedicados por región por cuota de familia de máquinas virtuales y cuota regional total, que se aplica a la creación de instancias de proceso, se unifica y comparte con la cuota de clúster de proceso de Azure Machine Learning. La detención de la instancia de proceso no libera la cuota para garantizar que pueda reiniciar la instancia de proceso.

La instancia de proceso viene con el disco del sistema operativo P10. El tipo de disco temporal depende del tamaño de VM elegido. Actualmente, no es posible cambiar el tipo de disco del sistema operativo.


### <a name="create-on-behalf-of-preview"></a>Creación en nombre de alguien (versión preliminar)

Como administrador, puede crear una instancia de proceso en nombre de un científico de datos y asignarle la instancia con:
* [Plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance)  Para más información sobre cómo buscar los valores de TenantID y ObjectID necesarios en esta plantilla, consulte [Encontrar identificadores de objeto de identidad para la configuración de autenticación](../healthcare-apis/fhir/find-identity-object-ids.md).  También puede encontrar estos valores en el portal de Azure Active Directory.
* API DE REST

El científico de datos para el que se crea la instancia de proceso necesita los siguientes permisos de RBAC de Azure: 
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*
* *Microsoft.MachineLearningServices/workspaces/computes/applicationaccess/action*

El científico de datos puede iniciar, detener y reiniciar la instancia de proceso. Puede utilizar la instancia de proceso para:
* Jupyter
* JupyterLab
* RStudio
* Cuadernos integrados

## <a name="compute-target"></a>Destino de proceso

Las instancias de proceso se pueden usar como [destino de proceso de entrenamiento](concept-compute-target.md#train) de forma similar a los clústeres de entrenamiento de Azure Machine Learning. 

Una instancia de proceso:
* Tiene una cola de trabajo.
* Ejecuta trabajos de forma segura en un entorno de red virtual, sin necesidad de que las empresas abran el puerto SSH. El trabajo se ejecuta en un entorno en contenedor y empaqueta las dependencias del modelo en un contenedor de Docker.
* Puede ejecutar varios trabajos pequeños en paralelo (versión preliminar).  Se pueden ejecutar dos trabajos por núcleo en paralelo mientras el resto de los trabajos se ponen en cola.
* Admite trabajos de entrenamiento distribuidos con varias GPU de un solo nodo.

Puede usar la instancia de proceso como destino de implementación de inferencia local para escenarios de prueba o depuración.

> [!TIP]
> La instancia de proceso tiene un disco de SO de 120 GB. Si se queda sin espacio en el disco y entra en un estado inutilizable, borre al menos 5 GB de espacio en el disco del sistema operativo (/dev/sda1/ sistema de archivo en el que está montado /) a través del terminal de JupyterLab eliminando archivos o carpetas; a continuación, reinicie sudo. Para acceder al terminal de JupyterLab, vaya a https://ComputeInstanceName.AzureRegion.instances.azureml.ms/lab y reemplace el nombre de la instancia de proceso y la región de Azure y, a continuación, haga clic en Archivo->Nuevo->Terminal. Borre al menos 5 GB antes de [detener o reiniciar](how-to-create-manage-compute-instance.md#manage) la instancia de proceso. Para comprobar el espacio en disco disponible, ejecute df -h en el terminal.

## <a name="next-steps"></a>Pasos siguientes

* [Creación y administración de una instancia de proceso](how-to-create-manage-compute-instance.md)
* [Tutorial: Formar su primer modelo de ML](tutorial-1st-experiment-sdk-train.md) muestra cómo usar una instancia de proceso con un cuaderno integrado.
