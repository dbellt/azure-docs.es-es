---
title: Creación y administración de recursos con la extensión de VS Code (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo crear y administrar recursos de Azure Machine Learning mediante la extensión de Visual Studio Code para Azure Machine Learning.
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: 4ebdb5d092a64213727bd35d923c335c8e82210b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479917"
---
# <a name="manage-azure-machine-learning-resources-with-the-vs-code-extension-preview"></a>Administración de recursos de Azure Machine Learning con la extensión de VS Code (versión preliminar)

Aprenda a administrar los recursos de Azure Machine Learning con la extensión de VS Code.

![Extensión Azure Machine Learning para VS Code](media/how-to-manage-resources-vscode/azure-machine-learning-vscode-extension.png)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción de Azure. Si no tiene una, regístrese para probar la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
- Código de Visual Studio. Si no lo tiene, [instálelo](https://code.visualstudio.com/docs/setup/setup-overview).
- Extensión de Azure Machine Learning. Siga la [guía de instalación de la extensión Azure Machine Learning para VS Code](how-to-setup-vs-code.md) para configurar la extensión.

## <a name="create-resources"></a>Crear recursos

La manera más rápida de crear recursos es mediante la barra de herramientas de la extensión.

1. Abra la vista de Azure Machine Learning.
1. Seleccione **+** en la barra de actividades.
1. Elija el recurso en la lista desplegable.
1. Configure el archivo de especificación. La información necesaria depende del tipo de recurso que desee crear.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede crear un recurso mediante la paleta de comandos:

1. Abra la paleta de comandos **Ver > Paleta de comandos**.
1. Escriba `> Azure ML: Create <RESOURCE-TYPE>` en el cuadro de texto. Reemplace `RESOURCE-TYPE` por el tipo de recurso que desee crear.
1. Configure el archivo de especificación.
1. Abra la paleta de comandos **Ver > Paleta de comandos**.
1. Escriba `> Azure ML: Create Resource` en el cuadro de texto.

## <a name="version-resources"></a>Recursos de versión

Algunos recursos (como entornos, conjuntos de datos y modelos) permiten realizar cambios en un recurso y almacenar las distintas versiones.

Para crear una versión de un recurso:

1. Use el archivo de especificación existente que creó el recurso o siga el proceso de creación de recursos para crear un nuevo archivo de especificación.
1. Incremente el número de versión en la plantilla.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

Siempre que el nombre del recurso actualizado sea el mismo que el de la versión anterior, Azure Machine Learning recoge los cambios y crea una nueva versión.

## <a name="workspaces"></a>Áreas de trabajo

Para más información, consulte el artículo sobre las [áreas de trabajo](concept-workspace.md).

### <a name="create-a-workspace"></a>Crear un área de trabajo

1. En la vista de Azure Machine Learning, haga clic con el botón derecho en el nodo de suscripción y seleccione **Crear área de trabajo**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Workspace` en la paleta de comandos.

### <a name="remove-workspace"></a>Eliminación del área de trabajo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Haga clic con el botón derecho en el área de trabajo que quiere quitar.
1. Seleccione si quiere quitar:
    - *Solo el área de trabajo*: esta opción **solo** elimina el recurso de área de trabajo de Azure. El grupo de recursos, las cuentas de almacenamiento y otros recursos a los que se adjuntó el área de trabajo siguen en Azure.
    - *Con recursos asociados*: esta opción elimina el área de trabajo **y** todos los recursos que tiene asociados.

También puede usar el comando `> Azure ML: Remove Workspace` en la paleta de comandos.

## <a name="datastores"></a>Almacenes de datos

Actualmente, la extensión admite estos tipos de almacenes de datos:

- Blob de Azure
- Azure Data Lake Gen 1
- Azure Data Lake Gen 2
- Archivo de Azure

Para más información, consulte la sección sobre los [almacenes de datos](concept-data.md#datastores).

### <a name="create-a-datastore"></a>Creación de un almacén de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el almacén de datos.
1. Haga clic con el botón derecho en el nodo **Almacenes de datos** y seleccione **Crear almacén de datos**.
1. Elija el tipo de almacén de datos.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Datastore` en la paleta de comandos.

### <a name="manage-a-datastore"></a>Administración de un almacén de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Almacenes de datos** dentro del área de trabajo.
1. Haga clic con el botón derecho en el almacén de datos para el que desee realizar alguna de las siguientes operaciones:
    - *Anular registro de almacén de datos*. Quita el almacén de datos del área de trabajo.
    - *Ver el almacén de datos*. Se muestra la configuración del almacén de datos de solo lectura.

También puede usar los comandos `> Azure ML: Unregister Datastore` y `> Azure ML: View Datastore` respectivamente en la paleta de comandos.

## <a name="datasets"></a>Conjuntos de datos

Actualmente, la extensión admite estos tipos de conjunto de datos:

- *Tabular*: permite materializar los datos en un dataframe.
- *Archivo*: un archivo o una colección de archivos. Permite descargar o montar archivos en el proceso.

Para más información, consulte la sección sobre [conjuntos de datos](concept-data.md#datasets).

### <a name="create-dataset"></a>Creación de un conjunto de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde desee crear el conjunto de datos.
1. Haga clic con el botón derecho en el nodo **Conjuntos de datos** y seleccione **Crear conjunto de datos**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Dataset` en la paleta de comandos.

### <a name="manage-a-dataset"></a>Administración de un conjunto de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Conjuntos de datos**.
1. Haga clic con el botón derecho en el conjunto de datos en el que desee realizar alguna de las siguientes operaciones:
    - **Ver las propiedades del conjunto de datos**. Permite ver los metadatos asociados a un conjunto de datos específico. Si tiene varias versiones de un conjunto de datos, puede optar por ver solo las propiedades del conjunto de datos de una versión específica al expandir el nodo del conjunto de datos y realizar los mismos pasos descritos en esta sección sobre la versión correspondiente.
    - **Obtener una vista previa del conjunto de datos**. Vea el conjunto de datos directamente en el visor de datos de VS Code. Tenga en cuenta que esta opción solo está disponible para los conjuntos de datos tabulares.
    - **Anular el registro de un conjunto de dato**. Quita un conjunto de datos y todas sus versiones del área de trabajo.

También puede usar los comandos `> Azure ML: View Dataset Properties` y `> Azure ML: Unregister Dataset` respectivamente en la paleta de comandos.

## <a name="environments"></a>Entornos

Para más información, consulte la sección sobre los [entornos](concept-environments.md).

### <a name="create-environment"></a>Creación del entorno

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el almacén de datos.
1. Haga clic con el botón derecho en el nodo **Entornos** y seleccione **Crear entorno**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Environment` en la paleta de comandos.

### <a name="view-environment-configurations"></a>Visualización de configuraciones de entorno

Para ver las dependencias y configuraciones de un entorno específico en la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Entornos**.
1. Haga clic con el botón derecho en el entorno que quiere ver y seleccione **Ver entorno**.

También puede usar el comando `> Azure ML: View Environment` en la paleta de comandos.

## <a name="experiments"></a>Experimentos

Para más información, consulte la sección sobre los [experimentos](concept-azure-machine-learning-architecture.md#experiments).

### <a name="create-job"></a>Creación del trabajo

La manera más rápida de crear un trabajo es hacer clic en el icono **Crear trabajo** de la barra de actividades de la extensión.

Mediante los nodos de recursos de la vista de Azure Machine Learning:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Experimentos** en el área de trabajo y seleccione **Crear trabajo**.
1. Elija el tipo de trabajo.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Job` en la paleta de comandos.

### <a name="view-job"></a>Ver trabajo

Para ver el trabajo en Azure Machine Learning Studio:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Haga clic con el botón derecho en el experimento que desee ver y seleccione **View Experiment in Studio** (Ver experimento en Studio).
1. Aparece una solicitud para que abra la dirección URL del experimento en Azure Machine Learning Studio. seleccione **Open**(Abrir).

También puede usar el comando `> Azure ML: View Experiment in Studio` en la paleta de comandos.

### <a name="track-run-progress"></a>Seguimiento del progreso de la ejecución

Mientras ejecuta el trabajo, puede que quiera ver el progreso. Para hacer un seguimiento del progreso de una ejecución en Azure Machine Learning Studio desde la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Expanda el nodo del trabajo para el que quiere hacer el seguimiento.
1. Haga clic con el botón derecho en la ejecución y seleccione **View Run in Studio** (Ver ejecución en Studio).
1. Aparece una solicitud para que abra la dirección URL de la ejecución en Azure Machine Learning Studio. seleccione **Open**(Abrir).

### <a name="download-run-logs--outputs"></a>Descarga de registros y salidas de la ejecución

Una vez que se completa una ejecución, puede que quiera descargar los registros y recursos, como el modelo generado como parte de la ejecución.

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Expanda el nodo de trabajo para el que desea descargar registros y salidas.
1. Haga clic con el botón derecho en la ejecución:
    - Para descargar las salidas, seleccione **Download outputs** (Descargar salidas).
    - Para descargar los registros, seleccione **Download logs** (Descargar registros).

También puede usar los comandos `> Azure ML: Download Outputs` y `> Azure ML: Download Logs` respectivamente en la paleta de comandos.

## <a name="compute-instances"></a>Instancias de proceso

Para obtener más información, vea [instancias de proceso](concept-compute-instance.md).

### <a name="create-compute-instance"></a>Creación de una instancia de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Proceso**.
1. Haga clic con el botón derecho en el nodo **Instancias de Compute** en el área de trabajo y seleccione **Creación de un proceso**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Compute` en la paleta de comandos.

### <a name="connect-to-compute-instance"></a>Conexión a una instancia de proceso

Para usar una instancia de proceso como un entorno de desarrollo o un servidor de Jupyter remoto, consulte [Conexión a una instancia de proceso](how-to-set-up-vs-code-remote.md?tabs=extension).

### <a name="stop-or-restart-compute-instance"></a>Detención o reinicio de la instancia de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Instancias de Compute** dentro del nodo **Proceso**.
1. Haga clic con el botón secundario en la instancia de proceso que desee detener o reiniciar y seleccione **Detener instancia de proceso** o **Restart compute instance** (Reiniciar instancia de proceso), respectivamente.

También puede usar los comandos `> Azure ML: Stop Compute instance` y `Restart Compute instance` respectivamente en la paleta de comandos.

### <a name="view-compute-instance-configuration"></a>Configuración de visualización de la instancia de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Instancias de Compute** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en la instancia de proceso que quiere inspeccionar y seleccione **View Compute instance Properties** (Ver propiedades de la instancia de proceso).

También puede usar el comando `Azure ML: View Compute instance Properties` en la paleta de comandos.

### <a name="delete-compute-instance"></a>Eliminación de una instancia de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Instancias de Compute** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en la instancia de proceso que quiere eliminar y seleccione **Delete compute instance** (Eliminar instancia de proceso).

También puede usar el comando `Azure ML: Delete Compute instance` en la paleta de comandos.

## <a name="compute-clusters"></a>Clústeres de proceso

Para más información, consulte el artículo sobre el [entrenamiento de destinos de proceso](concept-compute-target.md#train).

### <a name="create-compute-cluster"></a>Creación de un clúster de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Proceso**.
1. Haga clic con el botón derecho en el nodo **Clústeres de proceso** en el área de trabajo y seleccione **Creación de un proceso**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Compute` en la paleta de comandos.

### <a name="view-compute-configuration"></a>Vista de la configuración de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Clústeres de proceso** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en el proceso que quiere ver y seleccione **View Compute Properties** (Ver propiedades del proceso).

También puede usar el comando `> Azure ML: View Compute Properties` en la paleta de comandos.

### <a name="delete-compute-cluster"></a>Eliminación de un clúster de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Clústeres de proceso** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en el proceso que desee eliminar y seleccione **Remove compute** (Quitar proceso).

También puede usar el comando `> Azure ML: Remove Compute` en la paleta de comandos.

## <a name="inference-clusters"></a>Clústeres de inferencia

Para más información, consulte la sección sobre los [destinos de proceso para inferencia](concept-compute-target.md#deploy).

### <a name="manage-inference-clusters"></a>Administración de clústeres de inferencia

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Inference clusters** (Clústeres de inferencia) dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en el proceso en el que desee realizar alguna de las siguientes operaciones:
    - **Ver las propiedades del proceso**. Muestra datos de configuración de solo lectura sobre el proceso asociado.
    - **Desasociar el proceso**. Desasocia el proceso del área de trabajo.

También puede usar los comandos `> Azure ML: View Compute Properties` y `> Azure ML: Detach Compute` respectivamente en la paleta de comandos.

### <a name="delete-inference-clusters"></a>Eliminación de clústeres de inferencia

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Procesos asociados** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en el proceso que desee eliminar y seleccione **Remove compute** (Quitar proceso).

También puede usar el comando `> Azure ML: Remove Compute` en la paleta de comandos.

## <a name="attached-compute"></a>Proceso asociado

Para más información, consulte [Proceso no administrado](concept-compute-target.md#unmanaged-compute).

### <a name="manage-attached-compute"></a>Administración de procesos asociados

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Procesos asociados** dentro del nodo **Proceso**.
1. Haga clic con el botón derecho en el proceso en el que desee realizar alguna de las siguientes operaciones:
    - **Ver las propiedades del proceso**. Muestra datos de configuración de solo lectura sobre el proceso asociado.
    - **Desasociar el proceso**. Desasocia el proceso del área de trabajo.

También puede usar los comandos `> Azure ML: View Compute Properties` y `> Azure ML: Detach Compute` respectivamente en la paleta de comandos.

## <a name="models"></a>Modelos

Para más información, consulte la sección sobre los [modelos](concept-azure-machine-learning-architecture.md#models).

### <a name="create-model"></a>Crear modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Modelos** en el área de trabajo y seleccione **Crear modelo**.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Model` en la paleta de comandos.

### <a name="view-model-properties"></a>Visualización de las propiedades del modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo cuyas propiedades quiere ver y seleccione **View Model Properties** (Ver propiedades del modelo). En el editor se abre un archivo que contiene las propiedades del modelo.

También puede usar el comando `> Azure ML: View Model Properties` en la paleta de comandos.

### <a name="download-model"></a>Descarga del modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo que va a descargar y seleccione **Download Model File** (Descargar archivo del modelo).

También puede usar el comando `> Azure ML: Download Model File` en la paleta de comandos.

### <a name="delete-a-model"></a>Eliminación de un modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo que quiere eliminar y seleccione **Eliminar modelo**.
1. Aparece un mensaje para confirmar que quiere quitar el modelo. Seleccione **Aceptar**.

También puede usar el comando `> Azure ML: Remove Model` en la paleta de comandos.

## <a name="endpoints"></a>Puntos de conexión

Para obtener más información, consulte [Puntos de conexión](concept-azure-machine-learning-architecture.md#endpoints).

### <a name="create-endpoint"></a>Crear punto de conexión

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Modelos** en el área de trabajo y seleccione **Creación de un punto de conexión**.
1. Elija el tipo de punto de conexión.
1. Aparecerá un archivo de especificación. Configure el archivo de especificación.
1. Haga clic con el botón derecho en el archivo de especificación y seleccione **Azure ML: Crear recurso**.

También puede usar el comando `> Azure ML: Create Endpoint` en la paleta de comandos.

### <a name="delete-endpoint"></a>Eliminación de un punto de conexión

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Puntos de conexión** en el área de trabajo.
1. Haga clic con el botón derecho en la implementación que desee quitar y seleccione **Quitar servicio**.
1. Aparece un mensaje para confirmar que quiere quitar el servicio. Seleccione **Aceptar**.

También puede usar el comando `> Azure ML: Remove Service` en la paleta de comandos.

### <a name="view-service-properties"></a>Visualización de las propiedades del servicio

Además de crear y eliminar implementaciones, también puede ver y editar la configuración asociada con la implementación.

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Puntos de conexión** en el área de trabajo.
1. Haga clic con el botón derecho en la implementación que quiere administrar:
    - Para ver la configuración de implementación, seleccione **View Service Properties** (Ver propiedades del servicio).

También puede usar el comando `> Azure ML: View Service Properties` en la paleta de comandos.

## <a name="next-steps"></a>Pasos siguientes

[Entrenamiento de un modelo de clasificación de imágenes](tutorial-train-deploy-image-classification-model-vscode.md) con la extensión de VS Code.