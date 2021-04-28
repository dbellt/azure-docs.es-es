---
title: Conexión a una instancia de proceso en Visual Studio Code (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo conectarse a una instancia de proceso de Azure Machine Learning en Visual Studio Code para ejecutar cargas de trabajo interactivas de Jupyter Notebook y desarrollo remoto.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: luquinta
author: luisquintanilla
ms.date: 04/08/2021
ms.openlocfilehash: 7199534fa581ccb235bc8091d9459c640d53b74f
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884631"
---
# <a name="connect-to-an-azure-machine-learning-compute-instance-in-visual-studio-code-preview"></a>Conexión a una instancia de proceso de Azure Machine Learning en Visual Studio Code (versión preliminar)

En este artículo, aprenderá a conectarse a una instancia de proceso de Azure Machine Learning mediante Visual Studio Code.

Una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) es una estación de trabajo basada en la nube totalmente administrada para científicos de datos que proporciona funciones de administración y preparación para empresas a los administradores de TI.

Hay dos maneras de conectarse a una instancia de proceso desde Visual Studio Code:

* Instancia de proceso remoto. Esta opción proporciona un entorno de desarrollo completo para compilar los proyectos de aprendizaje automático.
* Servidor remoto de Jupyter Notebook. Esta opción permite establecer una instancia de proceso como servidor remoto de Jupyter Notebook.

## <a name="configure-a-remote-compute-instance"></a>Configuración de una instancia de proceso remoto

Para configurar una instancia de proceso remoto para el desarrollo, necesitará algunos requisitos previos.

* Extensión Azure Machine Learning para Visual Studio Code. Para más información, consulte la [guía de instalación de la extensión Azure Machine Learning para Visual Studio Code](tutorial-setup-vscode-extension.md).
* Área de trabajo de Azure Machine Learning. [Use la extensión Azure Machine Learning para Visual Studio Code para crear un área de trabajo](how-to-manage-resources-vscode.md#create-a-workspace) si aún no tiene una.
* Instancia de proceso de Azure Machine Learning. [Use la extensión Azure Machine Learning para Visual Studio Code para crear una instancia de proceso](how-to-manage-resources-vscode.md#create-compute-instance) si no tiene una.

Para conectarse a la instancia de proceso remoto:

# <a name="vs-code"></a>[Código de VS](#tab/extension)

### <a name="azure-machine-learning-extension"></a>Extensión de Azure Machine Learning

1. En VS Code, inicie la extensión de Azure Machine Learning.
1. Expanda el nodo **Instancias de proceso** en la extensión.
1. Haga clic con el botón derecho en la instancia de proceso a la que desea conectarse y seleccione **Conectarse a la instancia de proceso**.

:::image type="content" source="media/how-to-set-up-vs-code-remote/vs-code-compute-instance-launch.png" alt-text="Conexión a la extensión de Azure ML de Visual Studio Code de la instancia de proceso" lightbox="media/how-to-set-up-vs-code-remote/vs-code-compute-instance-launch.png":::

### <a name="command-palette"></a>Paleta de comandos

1. En VS Code, seleccione **Ver > Paleta de comandos** para abrir la paleta de comandos.
1. Escriba en el cuadro de texto **Azure ML: Conectarse a la instancia de proceso**.
1. Seleccione su suscripción.
1. Seleccione su área de trabajo.
1. Seleccione la instancia de proceso o cree una nueva.

# <a name="studio"></a>[Estudio](#tab/studio)

Vaya a [ml.azure.com](https://ml.azure.com).

> [!IMPORTANT]
> Para conectarse a la instancia de proceso remoto desde Visual Studio Code, asegúrese de que la cuenta en la que ha iniciado sesión en Estudio de Azure Machine Learning es la misma que usa en Visual Studio Code.

### <a name="compute"></a>Proceso

1. Seleccione la pestaña **Proceso**.
1. En la columna *URI de la aplicación*, seleccione **VS Code** para la instancia de proceso a la que desea conectarse.

:::image type="content" source="media/how-to-set-up-vs-code-remote/studio-compute-instance-vs-code-launch.png" alt-text="Conexión a Estudio de Azure ML de VS Code de la instancia de proceso" lightbox="media/how-to-set-up-vs-code-remote/studio-compute-instance-vs-code-launch.png":::

### <a name="notebook"></a>Notebook

1. Seleccione la pestaña **Cuaderno**.
1. En la pestaña *Cuaderno,* seleccione el archivo que desea editar.
1. Seleccione **Editores > Editar en VS Code (versión preliminar)** .

:::image type="content" source="media/how-to-set-up-vs-code-remote/studio-notebook-compute-instance-vs-code-launch.png" alt-text="Conexión al cuaderno de Estudio de Azure ML de VS Code de la instancia de proceso" lightbox="media/how-to-set-up-vs-code-remote/studio-notebook-compute-instance-vs-code-launch.png":::

---

Se inicia una nueva ventana para la instancia de proceso remoto. Al intentar realizar una conexión a una instancia de proceso remoto, se llevan a cabo las siguientes tareas:

1. Autorización. Se realizan algunas comprobaciones para asegurarse de que el usuario que intenta realizar una conexión está autorizado para usar la instancia de proceso.
1. El servidor remoto de VS Code está instalado en la instancia de proceso.
1. Se establece una conexión de WebSocket para la interacción en tiempo real.

Una vez establecida la conexión, se conserva. Al principio de la sesión se emite un token que se actualiza automáticamente para mantener la conexión con la instancia de proceso.

Después de conectarse a la instancia de proceso remoto, use el editor para:

* [Crear y administrar archivos en la instancia de proceso remoto o el recurso compartido de archivos](https://code.visualstudio.com/docs/editor/codebasics).
* Usar el [terminal integrado de VS Code](https://code.visualstudio.com/docs/editor/integrated-terminal) para [ejecutar comandos y aplicaciones en la instancia de proceso remoto](how-to-access-terminal.md).
* [Depurar scripts y aplicaciones](https://code.visualstudio.com/Docs/editor/debugging).
* [Usar VS Code para administrar los repositorios de Git](concept-train-model-git-integration.md).

## <a name="configure-compute-instance-as-remote-notebook-server"></a>Configuración de la instancia de proceso como servidor de cuadernos remoto

Para configurar una instancia de proceso como servidor remoto de Jupyter Notebook, necesitará cumplir algunos requisitos previos:

* Extensión Azure Machine Learning para Visual Studio Code. Para más información, consulte la [guía de instalación de la extensión Azure Machine Learning para Visual Studio Code](tutorial-setup-vscode-extension.md).
* Área de trabajo de Azure Machine Learning. [Use la extensión Azure Machine Learning para Visual Studio Code para crear un área de trabajo](how-to-manage-resources-vscode.md#create-a-workspace) si aún no tiene una.

Para conectarse a una instancia de proceso:

1. Abra una instancia de Jupyter Notebook en Visual Studio Code.
1. Cuando se cargue la experiencia integrada de Notebook, seleccione **Jupyter Server** (Servidor de Jupyter).

    > [!div class="mx-imgBorder"]
    > ![Inicio de Azure Machine Learning: lista desplegable de servidores remotos de Jupyter Notebook](media/how-to-set-up-vs-code-remote/launch-server-selection-dropdown.png)

    Como alternativa, también puede usar la paleta de comandos:

    1. Seleccione **Ver > Paleta de comandos** en la barra de menús para abrir la paleta de comandos.
    1. Escriba `Azure ML: Connect to Compute instance Jupyter server` en el cuadro de texto.

1. Elija `Azure ML Compute Instances` en la lista de opciones de servidores de Jupyter.
1. En la lista de suscripciones, seleccione la suya. Si ha configurado previamente el área de trabajo de Azure Machine Learning predeterminada, este paso se omitirá.
1. Seleccione su área de trabajo.
1. Seleccione la instancia de proceso de la lista. Si no tiene una, seleccione **Create new Azure ML Compute Instance** (Crear una instancia de proceso de Azute ML) y siga las indicaciones para crearla.
1. Para que los cambios surtan efecto, tiene que volver a cargar Visual Studio Code.
1. Abra un cuaderno de Jupyter Notebook y ejecute una celda.

> [!IMPORTANT]
> Para establecer la conexión, **DEBE** ejecutar una celda.

Llegados a este punto, puede continuar ejecutando las celdas en el cuaderno de Jupyter Notebook.

> [!TIP]
> También puede trabajar con archivos de script de Python (.py) que contengan celdas de código similares a Jupyter. Para más información, consulte la [documentación interactiva de Python para Visual Studio](https://code.visualstudio.com/docs/python/jupyter-support-py).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado Visual Studio Code remoto, puede usar una instancia de proceso como proceso remoto desde Visual Studio Code para [depurar el código de forma interactiva](how-to-debug-visual-studio-code.md).

[Tutorial: Formar su primer modelo de ML](tutorial-1st-experiment-sdk-train.md) muestra cómo usar una instancia de proceso con un cuaderno integrado.
