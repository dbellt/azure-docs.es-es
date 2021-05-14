---
title: Acceso a un terminal de instancia de proceso en el área de trabajo
titleSuffix: Azure Machine Learning
description: Use el terminal en una instancia de proceso para las operaciones de Git, para instalar paquetes y para agregar kernels.
services: machine-learning
author: abeomor
ms.author: osomorog
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 02/05/2021
ms.openlocfilehash: bff14ddd6a361c93c416aa746640b774afc43961
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897242"
---
# <a name="access-a-compute-instance-terminal-in-your-workspace"></a>Acceso a un terminal de instancia de proceso en el área de trabajo

Acceda al terminal de una instancia de proceso en el área de trabajo para hacer lo siguiente:

* Usar los archivos de Git y los archivos de versión. Estos archivos se almacenan en el sistema de archivos del área de trabajo, no se restringen a una única instancia de proceso.
* Instalar los paquetes en la instancia de proceso.
* Crear kernels adicionales en la instancia de proceso.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="access-a-terminal"></a>Acceso a un terminal

Para tener acceso al terminal:

1. En [Azure Machine Learning Studio](https://ml.azure.com), abra el área de trabajo.
1. En el lado izquierdo, seleccione **Notebooks**.
1. Seleccione la imagen de **Abrir terminal**.

    :::image type="content" source="media/how-to-use-terminal/open-terminal-window.png" alt-text="Abrir una ventana de terminal":::

1. Cuando se está ejecutando una instancia de proceso, aparece la ventana de terminal correspondiente.
1. Si no se está ejecutando ninguna instancia de proceso, use la sección **Proceso** de la derecha para iniciar o crear una instancia de proceso.
    :::image type="content" source="media/how-to-use-terminal/start-or-create-compute.png" alt-text="Inicio o creación de una instancia de proceso":::

Además de los pasos anteriores, también puede acceder al terminal desde:

* RStudio: seleccione la pestaña **Terminal**, situada en la parte superior izquierda.
* Jupyter Lab:  seleccione el icono **Terminal** en el encabezado **Otros** de la pestaña Iniciador.
* Jupyter:  seleccione **Nuevo>Terminal** en la parte superior derecha de la pestaña Archivos.
* SSH en la máquina, si habilitó el acceso SSH cuando se creó la instancia de proceso.

## <a name="copy-and-paste-in-the-terminal"></a>Copiado y pegado en el terminal

> * Windows: `Ctrl-Insert` para copiar y `Ctrl-Shift-v` o `Shift-Insert` para pegar.
> * Mac OS: `Cmd-c` para copiar y `Cmd-v` para pegar.
> * Es posible que Firefox o IE no admitan los permisos del Portapapeles correctamente.

## <a name="use-files-from-git-and-version-files"></a><a name=git></a> Uso de archivos de Git y archivos de versión

Acceda a todas las operaciones de Git desde el terminal. Todos los archivos y carpetas de Git se almacenarán en el sistema de archivos del área de trabajo. Este almacenamiento le permite usar estos archivos desde cualquier instancia de proceso en el área de trabajo.

> [!NOTE]
> Agregue los archivos y las carpetas en cualquier parte dentro de la carpeta **~/cloudfiles/code/Users** para que estén visibles en todos los entornos de Jupyter.

Obtenga más información sobre [la clonación de repositorios de GIT de en el sistema de archivos del área de trabajo](concept-train-model-git-integration.md#clone-git-repositories-into-your-workspace-file-system).

## <a name="install-packages"></a>Instalar paquetes

 Instale paquetes desde una ventana del terminal. Instale los paquetes de Python en el entorno de **Python 3.6: AzureML**.  Instale los paquetes de R en el entorno de **R**.

O puede instalar paquetes directamente en Jupyter Notebook o en RStudio:

* RStudio: use la pestaña **Paquetes**, situada en la parte inferior derecha, o la pestaña **Consola**, situada en la parte superior izquierda.  
* Python: agregue el código de instalación y ejecútelo en una celda de Jupyter Notebook.

> [!NOTE]
> Para la administración de paquetes en un cuaderno, use las funciones magic **%pip** o **%conda** para instalar paquetes automáticamente en el **kernel que actualmente está en ejecución**, en lugar de **!pip** o **!conda** que hacen referencia a todos los paquetes (incluidos los paquetes que se encuentran fuera del kernel que actualmente está en ejecución)

## <a name="add-new-kernels"></a>Incorporación de nuevos kernels

> [!WARNING]
>  Durante la personalización de la instancia de proceso, asegúrese de no eliminar el entorno de Conda **azureml_py36** ni el kernel de **Python 3.6: AzureML**. Esto es necesario para la funcionalidad de Jupyter/JupyterLab.

Para agregar un kernel nuevo de Jupyter a la instancia de proceso:

1. Use la ventana de terminal para crear un nuevo entorno.  Por ejemplo, el código siguiente crea `newenv`:

    ```shell
    conda create --name newenv
    ```

1. Active el entorno.  Por ejemplo, después de crear `newenv`:

    ```shell
    conda activate newenv
    ```

1. Instalación del paquete PIP e ipykernel en el entorno nuevo y creación de un kernel para ese entorno conda

    ```shell
    conda install pip
    conda install ipykernel
    python -m ipykernel install --user --name newenv --display-name "Python (newenv)"
    ```

Se puede instalar cualquiera de los [kernels de Jupyter](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels) disponibles.

## <a name="manage-terminal-sessions"></a>Administración de las sesiones del terminal

 Seleccione **Ver sesiones activas** en la barra de herramientas del terminal para ver una lista de todas las sesiones de terminal activas. Si no hay sesiones activas, esta pestaña se deshabilitará.

Cierre todas las sesiones no utilizadas para conservar los recursos de la instancia de proceso.