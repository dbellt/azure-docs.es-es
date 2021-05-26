---
title: Configuración del entorno de desarrollo de Python
titleSuffix: Azure Machine Learning
description: Configure entornos de desarrollo de Python de Azure Machine Learning en Jupyter Notebook, Visual Studio Code, Azure Databricks y Data Science Virtual Machine.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.date: 03/22/2021
ms.topic: how-to
ms.custom: devx-track-python, contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: dde8291bce52e8f36aa712e294109dfe156fb426
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367209"
---
# <a name="set-up-a-python-development-environment-for-azure-machine-learning"></a>Configuración de un entorno de desarrollo de Python para Azure Machine Learning

Descubra cómo configurar un entorno de desarrollo de Python para Azure Machine Learning.

En la tabla siguiente se muestra cada entorno de desarrollo que se trata en este artículo, junto con las ventajas y desventajas.

| Entorno | Ventajas | Desventajas |
| --- | --- | --- |
| [Entorno local](#local) | Control total del entorno de desarrollo y las dependencias. Funciona con cualquier herramienta de compilación, entorno o IDE de su elección. | Tarda más tiempo en comenzar. Se deben instalar los paquetes de SDK necesarios y también debe instalarse un entorno si aún no tiene uno. |
| [Data Science Virtual Machine (DSVM)](#dsvm) | De forma parecida a la instancia de proceso basada en la nube (con Python y el SDK preinstalados), pero con herramientas adicionales de ciencia de datos y aprendizaje automático conocidas instaladas previamente. Fácil de escalar y combinar con otras herramientas y flujos de trabajo personalizados. | Una experiencia de inicio más lenta en comparación con la instancia de proceso basada en la nube. |
| [Instancia de proceso de Azure Machine Learning](#compute-instance) | Manera más fácil de empezar. El SDK completo ya está instalado en la máquina virtual del área de trabajo y los tutoriales de cuaderno están clonados previamente y listos para ejecutarse. | Falta de control sobre el entorno de desarrollo y las dependencias. Costo adicional por la máquina virtual Linux (la máquina virtual se puede detener cuando no se use para evitar cargos). Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). |
| [Azure Databricks](how-to-configure-databricks-automl-environment.md) | Perfecto para ejecutar flujos de trabajo de aprendizaje automático intensivos y a gran escala en la plataforma escalable de Apache Spark. | Excesivo para aprendizaje automático experimental o experimentos y flujos de trabajo a pequeña escala. Costo adicional por Azure Databricks. Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/databricks/). |

En este artículo también se proporcionan sugerencias de uso adicionales para las siguientes herramientas:

* Jupyter Notebook: si ya usa instancias de Jupyter Notebook, el SDK incluye algunos elementos adicionales que debe instalar.

* Visual Studio Code: Si usa Visual Studio Code, la [extensión de Azure Machine Learning](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai) incluye un amplio soporte de lenguaje para Python, así como características para que trabajar con Azure Machine Learning sea mucho más cómodo y productivo.

## <a name="prerequisites"></a>Requisitos previos

* Área de trabajo de Azure Machine Learning. Si no tiene ninguna, puede crear un área de trabajo de Azure Machine Learning a través de [Azure Portal](how-to-manage-workspace.md), la [CLI de Azure](how-to-manage-workspace-cli.md#create-a-workspace) y las [plantillas de Azure Resource Manager](how-to-create-workspace-template.md).

### <a name="local-and-dsvm-only-create-a-workspace-configuration-file"></a><a id="workspace"></a> Solo en el entorno local y DSVM: Crear un archivo de configuración del área de trabajo

El archivo de configuración del área de trabajo es un archivo JSON que le indica al SDK cómo comunicarse con el área de trabajo de Azure Machine Learning. El archivo se denomina *config.json* y tiene el formato siguiente:

```json
{
    "subscription_id": "<subscription-id>",
    "resource_group": "<resource-group>",
    "workspace_name": "<workspace-name>"
}
```

Este archivo JSON debe estar en la estructura de directorios que contiene los scripts de Python o las instancias de Jupyter Notebook. Puede estar en el mismo directorio, en un subdirectorio denominado *.azureml* o en un directorio principal.

Para utilizar este archivo desde el código, utilice el método [`Workspace.from_config`](/python/api/azureml-core/azureml.core.workspace.workspace#from-config-path-none--auth-none---logger-none---file-name-none-). Este código carga la información del archivo y se conecta a su área de trabajo.

Cree un archivo de configuración del área de trabajo con uno de los métodos siguientes:

* Azure portal

    **Descargue el archivo**: En [Azure Portal](https://ms.portal.azure.com), seleccione **Descargar config.json** desde la sección **Información general** del área de trabajo.

    ![Azure portal](./media/how-to-configure-environment/configure.png)

* SDK de Python de Azure Machine Learning

    Cree un script para conectarse a su área de trabajo de Azure Machine Learning y use el método [`write_config`](/python/api/azureml-core/azureml.core.workspace.workspace#write-config-path-none--file-name-none-) para generar el archivo y guárdelo como *.azureml/config.json*. Asegúrese de reemplazar `subscription_id`, `resource_group` y `workspace_name` por sus propios valores.

    ```python
    from azureml.core import Workspace

    subscription_id = '<subscription-id>'
    resource_group  = '<resource-group>'
    workspace_name  = '<workspace-name>'

    try:
        ws = Workspace(subscription_id = subscription_id, resource_group = resource_group, workspace_name = workspace_name)
        ws.write_config()
        print('Library configuration succeeded')
    except:
        print('Workspace not found')
    ```

## <a name="local-computer-or-remote-vm-environment"></a><a id="local"></a>Equipo local o entorno de máquina virtual remoto

Puede configurar un entorno en un equipo local o una máquina virtual remota, como una instancia de proceso de Azure Machine Learning o Data Science VM. 

Para configurar un entorno de desarrollo local o una máquina virtual remota:

1. Cree un entorno virtual de Python (virtualenv, conda).

    > [!NOTE]
    > Aunque no es necesario, se recomienda usar [Anaconda](https://www.anaconda.com/download/) o [Miniconda](https://www.anaconda.com/download/) para administrar entornos virtuales de Python e instalar paquetes.

    > [!IMPORTANT]
    > Si usa Linux o Mac OS y un shell distinto de bash (por ejemplo, zsh), es posible que reciba errores al ejecutar algunos comandos. Para solucionar este problema, use el comando `bash` para iniciar un nuevo shell de bash y ejecutar sus comandos.

1. Active el entorno virtual de Python recién creado.
1. Instale el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/install).
1. Con el fin de configurar el entorno local para usar el área de trabajo de Azure Machine Learning, [cree un archivo de configuración de área de trabajo](#workspace) o use uno que ya tenga.

Ahora que ha configurado el entorno local, ya podrá empezar a trabajar con Azure Machine Learning. Consulte la [guía de introducción de Azure Machine Learning para Python](tutorial-1st-experiment-hello-world.md) para comenzar.

### <a name="jupyter-notebooks"></a><a id="jupyter"></a>Jupyter Notebook

Al ejecutar un servidor de Jupyter Notebook local, se recomienda crear un kernel de IPython para el entorno virtual de Python. Esto ayuda a garantizar el comportamiento esperado de la importación del paquete y el kernel.

1. Habilitación de los kernels de IPython específicos del entorno

    ```bash
    conda install notebook ipykernel
    ```

1. Cree un kernel para el entorno virtual de Python. Asegúrese de reemplazar `<myenv>` por el nombre de su entorno virtual de Python.

    ```bash
    ipython kernel install --user --name <myenv> --display-name "Python (myenv)"
    ```

1. Inicio del servidor de Jupyter Notebook

Consulte el [repositorio de cuadernos de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks) para empezar a trabajar con Azure Machine Learning y Jupyter Notebook.

> [!NOTE]
> Puede encontrar un repositorio de ejemplos administrado por la comunidad en https://github.com/Azure/azureml-examples.

### <a name="visual-studio-code"></a><a id="vscode"></a>Visual Studio Code

Para usar Visual Studio Code para el desarrollo:

1. Instale [Visual Studio Code](https://code.visualstudio.com/Download).
1. Instale la [extensión Azure Machine Learning para Visual Studio Code](how-to-setup-vs-code.md) (versión preliminar).

Una vez que tenga instalada la extensión de Visual Studio Code, úsela para:

* [Administrar los recursos de Azure Machine Learning](how-to-manage-resources-vscode.md)
* [Conectarse a una instancia de proceso de Azure Machine Learning](how-to-set-up-vs-code-remote.md)
* [Ejecutar y depurar experimentos](how-to-debug-visual-studio-code.md)
* [Implementar modelos entrenados](tutorial-train-deploy-image-classification-model-vscode.md).

## <a name="azure-machine-learning-compute-instance"></a><a id="compute-instance"></a>Instancia de Proceso de Azure Machine Learning

La [instancia de proceso](concept-compute-instance.md) de Azure Machine Learning es una estación de trabajo de Azure segura y basada en la nube que proporciona a los científicos de datos un servidor de Jupyter Notebook, JupyterLab y un entorno de aprendizaje automático totalmente administrado.

No hay nada que instalar o configurar para una instancia de proceso.  

Cree una en cualquier momento desde el área de trabajo de Azure Machine Learning. Basta con que proporcione un nombre y especifique un tipo de máquina virtual de Azure. Pruébelo ahora con este [Tutorial: Configuración del entorno y el área de trabajo](quickstart-create-resources.md).

Para obtener más información acerca de las instancias de proceso, incluyendo cómo instalar paquetes, consulte [Creación y administración de una instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md).

> [!TIP]
> Para evitar incurrir en cargos por una instancia de proceso no usada, [detenga la instancia de proceso](how-to-create-manage-compute-instance.md#manage).

Además de un servidor de Jupyter Notebook y JupyterLab, puede usar instancias de proceso en la [característica de cuadernos integrada en Estudio de Azure Machine Learning](how-to-run-jupyter-notebooks.md).

También puede usar la extensión de Visual Studio Code para Azure Machine Learning para [conectarse a una instancia de proceso remota mediante VS Code](how-to-set-up-vs-code-remote.md).

## <a name="data-science-virtual-machine"></a><a id="dsvm"></a>Data Science Virtual Machine

Data Science VM es una imagen de máquina virtual personalizada que puede usar como entorno de desarrollo. Está diseñada para realizar trabajos de ciencia de datos que estén preconfigurados con herramientas y software como:

  - Paquetes, como TensorFlow, PyTorch, Scikit-learn, XGBoost y el SDK de Azure Machine Learning
  - Herramientas de ciencia de datos populares, como Spark Standalone o Drill
  - Herramientas de Azure, como la CLI de Azure, AzCopy y el Explorador de Storage
  - Entornos de desarrollo integrado (IDE), como Visual Studio Code y PyCharm
  - Servidor de Jupyter Notebook

Para ver una lista más completa de las herramientas, consulte la [guía de herramientas de Data Science VM](data-science-virtual-machine/tools-included.md).

> [!IMPORTANT]
> Si planea usar la instancia de Data Science VM como [destino de proceso](concept-compute-target.md) para sus trabajos de entrenamiento o inferencia, solo se admite Ubuntu.

Para usar la instancia de Data Science VM como entorno de desarrollo, haga lo siguiente:

1. Cree una instancia de Data Science VM con uno de los métodos siguientes:

    * Use Azure Portal para crear una instancia de DSVM de [Ubuntu](data-science-virtual-machine/dsvm-ubuntu-intro.md) o [Windows](data-science-virtual-machine/provision-vm.md).
    * [Cree una instancia de Data Science VM con plantillas de Resource Manager](data-science-virtual-machine/dsvm-tutorial-resource-manager.md).
    * Uso de la CLI de Azure

        Para crear una instancia de Data Science VM de Ubuntu, use el siguiente comando:

        ```azurecli-interactive
        # create a Ubuntu Data Science VM in your resource group
        # note you need to be at least a contributor to the resource group in order to execute this command successfully
        # If you need to create a new resource group use: "az group create --name YOUR-RESOURCE-GROUP-NAME --location YOUR-REGION (For example: westus2)"
        az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:linux-data-science-vm-ubuntu:linuxdsvmubuntu:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --generate-ssh-keys --authentication-type password
        ```

        Para crear una instancia de DSVM de Windows, use el siguiente comando:

        ```azurecli-interactive
        # create a Windows Server 2016 DSVM in your resource group
        # note you need to be at least a contributor to the resource group in order to execute this command successfully
        az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:dsvm-windows:server-2016:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --authentication-type password
        ```

1. Active el entorno de conda que contiene el SDK de Azure Machine Learning.

    * Para una instancia de Data Science VM de Ubuntu:

        ```bash
        conda activate py36
        ```

    * Para una instancia de Data Science VM de Windows:

        ```bash
        conda activate AzureML
        ```

1. Para configurar la instancia de Data Science VM que usará el área de trabajo de Azure Machine Learning, [cree un archivo de configuración del área de trabajo](#workspace) o use uno que ya tenga.

De forma similar a los entornos locales, puede usar Visual Studio Code y la [extensión de Azure Machine Learning para Visual Studio Code](#vscode) con el fin de interactuar con Azure Machine Learning.

Consulte [Data Science Virtual Machine](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/) para obtener más información.


## <a name="next-steps"></a>Pasos siguientes

- [Entrenar un modelo](tutorial-train-models-with-aml.md) en Azure Machine Learning con el conjunto de datos de MNIST.
- Consultar la [referencia del SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro). 
