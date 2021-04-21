---
title: 'Tutorial: Introducción a Machine Learning: Python'
titleSuffix: Azure Machine Learning
description: Introducción al SDK de Azure Machine Learning para Python que se ejecuta en el entorno de desarrollo personal.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 02/11/2021
ms.custom: devx-track-python, contperf-fy21q3
adobe-target: true
ms.openlocfilehash: 69340245cdebda570b526afd4f11ac8314384b43
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210625"
---
# <a name="tutorial-get-started-with-azure-machine-learning-in-your-development-environment-part-1-of-4"></a>Tutorial: Introducción a Azure Machine Learning en el entorno de desarrollo (parte 1 de 4)

En esta *serie de tutoriales de cuatro partes*, aprenderá los aspectos básicos de Azure Machine Learning y completará tareas de Machine Learning de Python basadas en trabajos en la plataforma de la nube de Azure. 

En la parte 1 de esta serie de tutoriales, deberá realizar lo siguiente:

> [!div class="checklist"]
> * Instale el SDK de Azure Machine Learning.
> * Configure la estructura de directorio del código.
> * Cree un área de trabajo de Azure Machine Learning.
> * Configure el entorno de desarrollo local.
> * Configure un clúster de proceso, un recurso basado en la nube para entrenar modelos.

Esta serie de tutoriales se centra en los conceptos de Azure Machine Learning necesarios para enviar **trabajos por lotes**: aquí es donde el código se envía a la nube para ejecutarse en segundo plano sin interacción del usuario. Esto resulta útil para scripts terminados o código que se desea ejecutar repetidamente, o para tareas de aprendizaje automático de proceso intensivo.

Otra excelente manera de empezar a usar Azure Machine Learning es con cuadernos de Jupyter Notebook.  Consulte [Tutorial: Introducción a Azure Machine Learning en cuadernos de Jupyter Notebook](tutorial-1st-experiment-sdk-setup.md)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe [Azure Machine Learning](https://aka.ms/AMLFree).
- Se recomienda usar [Anaconda](https://www.anaconda.com/download/) o [Miniconda](https://www.anaconda.com/download/) para administrar los entornos virtuales de Python e instalar los paquetes.  
- Si no está familiarizado con el uso de Conda, consulte [Introducción a Conda](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html).
- Cualquier entorno de desarrollo integrado o editor de texto para crear sus scripts de Python.

## <a name="install-the-azure-machine-learning-sdk"></a>Instalación del SDK de Azure Machine Learning

En este tutorial se usa el SDK de Azure Machine Learning para Python. Para evitar problemas de dependencia de Python, debe crear un entorno aislado. En esta serie de tutoriales se usa Conda para crear ese entorno. Si prefiere usar otras soluciones, como `venv`, `virtualenv` o Docker, asegúrese de que usa una versión de Python >= 3.5 y < 3.9.

Compruebe si tiene Conda instalado en el sistema:
    
```bash
conda --version
```
    
Si este comando devuelve un error `conda not found`, [descargue e instale Miniconda](https://docs.conda.io/en/latest/miniconda.html). 

Una vez instalado Conda, utilice una ventana del símbolo del sistema de terminal o Anaconda para crear un entorno:

```bash
conda create -n tutorial python=3.8
```

A continuación, instale el SDK de Azure Machine Learning en el entorno de Conda que creó:

```bash
conda activate tutorial
pip install azureml-core
```
    
> [!NOTE]
> La instalación del SDK de Azure Machine Learning tarda aproximadamente 2 minutos.
>
> Si recibe un error de tiempo de espera, intente `pip install --default-timeout=100 azureml-core` en su lugar.


> [!div class="nextstepaction"]
> [He instalado el SDK](?success=install-sdk#dir) [He tenido un problema](https://www.research.net/r/7C8Z3DN?issue=install-sdk)

## <a name="create-a-directory-structure-for-code"></a><a name="dir"></a>Creación de una estructura de directorio del código

Se recomienda configurar la siguiente estructura de directorio sencilla para este tutorial:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-1.png" alt-text="Estructura de los directorios: nivel superior del tutorial con el subdirectorio .azureml":::


- `tutorial`: directorio de nivel superior del proyecto.
- `.azureml`: subdirectorio oculto para almacenar archivos de configuración de Azure Machine Learning.

Por ejemplo, para crear esto en una ventana de Bash:

```bash
mkdir tutorial
cd tutorial
mkdir .azureml
```

> [!TIP]
> Para crear o ver la estructura en una ventana gráfica, habilite primero la posibilidad de ver y crear archivos y carpetas ocultos:
>
> * En una ventana de Finder de Mac, use **Comando + Mayús + .** para alternar la visualización de archivos o carpetas ocultos.  
> * En el Explorador de archivos de Windows 10, consulte [Ver carpetas y archivos ocultos en Windows 10](https://support.microsoft.com/en-us/windows/view-hidden-files-and-folders-in-windows-10-97fbc472-c603-9d90-91d0-1166d1d9f4b5). 
> * En la interfaz gráfica de Linux, use **Ctrl + h** o el menú **Ver** y active la casilla **Mostrar archivos ocultos**.




> [!div class="nextstepaction"]
> [He creado un directorio](?success=create-dir#workspace) [He tenido un problema](https://www.research.net/r/7C8Z3DN?issue=create-dir)

## <a name="create-an-azure-machine-learning-workspace"></a><a name="workspace"></a>Creación de un área de trabajo de Azure Machine Learning

Un área de trabajo es un recurso de nivel superior de Azure Machine Learning y es un lugar centralizado para:

- Administrar recursos, como procesos.
- Almacenar recursos, como cuadernos, entornos, conjuntos de datos, canalizaciones, modelos, puntos de conexión, etc.
- Colaborar con otros miembros del equipo.

En el directorio de nivel superior, `tutorial`, agregue un nuevo archivo de Python denominado `01-create-workspace.py` mediante el código siguiente. Adapte los parámetros (nombre, identificador de suscripción, grupo de recursos y [ubicación](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service)) a sus preferencias.

Puede ejecutar el código en una sesión interactiva o como un archivo de Python.

>[!NOTE]
> Cuando se use un entorno de desarrollo local (por ejemplo, un equipo), se le pedirá que se autentique en el área de trabajo mediante un *código de dispositivo* la primera vez que ejecute el código siguiente. Sigue las instrucciones en pantalla.

```python
# tutorial/01-create-workspace.py
from azureml.core import Workspace

ws = Workspace.create(name='<my_workspace_name>', # provide a name for your workspace
                      subscription_id='<azure-subscription-id>', # provide your subscription ID
                      resource_group='<myresourcegroup>', # provide a resource group name
                      create_resource_group=True,
                      location='<NAME_OF_REGION>') # For example: 'westeurope' or 'eastus2' or 'westus2' or 'southeastasia'.

# write out the workspace details to a configuration file: .azureml/config.json
ws.write_config(path='.azureml')
```

En la ventana que tiene activado el entorno de Conda *tutorial1*, ejecute este código desde el directorio `tutorial`.

```bash
cd <path/to/tutorial>
python ./01-create-workspace.py
```

> [!TIP]
> Si la ejecución de este código genera un error que indica que no tiene acceso a la suscripción, consulte [Creación de un área de trabajo](how-to-manage-workspace.md?tab=python#create-multi-tenant) para informarse sobre las opciones de autenticación.


Después de que haya ejecutado correctamente *01-create-workspace.py*, la estructura de carpetas tendrá el siguiente aspecto:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-2.png" alt-text="El archivo config.json aparece en el subdirectorio .azureml después de haber ejecutado 01-create-workspace.py.":::

El archivo `.azureml/config.json` contiene los metadatos necesarios para conectarse al área de trabajo de Azure Machine Learning. En concreto, contiene el identificador de la suscripción, el grupo de recursos y el nombre del área de trabajo. 

> [!NOTE]
> El contenido de `config.json` no es secreto. Estos detalles se pueden compartir.
>
> Todavía se requiere autenticación para interactuar con el área de trabajo de Azure Machine Learning.

> [!div class="nextstepaction"]
> [He creado un área de trabajo](?success=create-workspace#cluster) [He tenido un problema](https://www.research.net/r/7C8Z3DN?issue=create-workspace)

## <a name="create-an-azure-machine-learning-compute-cluster"></a><a name="cluster"></a> Creación de un clúster de proceso de Azure Machine Learning

Cree un script de Python en el directorio de nivel superior `tutorial` denominado `02-create-compute.py`. Rellénelo con el código siguiente para crear un clúster de proceso de Azure Machine Learning que se escalará automáticamente entre cero y cuatro nodos:

```python
# tutorial/02-create-compute.py
from azureml.core import Workspace
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

ws = Workspace.from_config() # This automatically looks for a directory .azureml

# Choose a name for your CPU cluster
cpu_cluster_name = "cpu-cluster"

# Verify that the cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
except ComputeTargetException:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                           idle_seconds_before_scaledown=2400,
                                                           min_nodes=0,
                                                           max_nodes=4)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

cpu_cluster.wait_for_completion(show_output=True)
```

En la ventana que tiene activado el entorno de Conda *tutorial1*, ejecute el archivo de Python:

```bash
python ./02-create-compute.py
```


> [!NOTE]
> Cuando se cree el clúster, no tendrá ningún nodo aprovisionado. El clúster *no* genera costos hasta que envíe un trabajo. Este clúster se reducirá verticalmente cuando haya estado inactivo durante 2400 segundos (40 minutos).

Ahora la estructura de carpetas tendrá el siguiente aspecto:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-3.png" alt-text="Incorporación de 02-create-compute.py al directorio del tutorial.":::

> [!div class="nextstepaction"]
> [He creado un clúster de proceso](?success=create-compute-cluster#next-steps) [He tenido un problema](https://www.research.net/r/7C8Z3DN?issue=create-compute-cluster)

## <a name="view-in-the-studio"></a>Visualización en Studio

Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com) para ver el área de trabajo y la instancia de proceso que ha creado.

1. Seleccione la **suscripción** que usó para crear el área de trabajo.
1. Seleccione el **área de trabajo de Machine Learning** que ha creado, *tutorial-ws*.
1. Una vez cargada el área de trabajo, en el lado izquierdo, seleccione **Compute** (Proceso).
1. En la parte superior, seleccione la pestaña **Compute clusters** (Clústeres de proceso).

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/compute-instance-in-studio.png" alt-text="Captura de pantalla: visualización de la instancia de proceso en el área de trabajo.":::

En esta vista se muestra el clúster de proceso aprovisionado, junto con el número de nodos inactivos, de nodos ocupados y de nodos no aprovisionados.  Dado que aún no ha usado el clúster, los nodos no están aprovisionados actualmente.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de configuración, ha hecho lo siguiente:

- Ha creado un área de trabajo de Azure Machine Learning.
- Ha configurado el entorno de desarrollo local.
- Ha creado un clúster de proceso de Azure Machine Learning.

En las otras partes de este tutorial, aprenderá lo siguiente:

* Parte 2. Ejecución del código en la nube con el SDK de Azure Machine Learning para Python.
* Parte 3. Administración del entorno de Python que se usa para el entrenamiento del modelo.
* Parte 4. Carga de datos en Azure y consumo de esos datos en el entrenamiento.

Continúe con el siguiente tutorial, que le guiará por los pasos para enviar un script al clúster de proceso de Azure Machine Learning.

> [!div class="nextstepaction"]
> [Tutorial: Ejecución del script de Python "Hola mundo" en Azure](tutorial-1st-experiment-hello-world.md)
