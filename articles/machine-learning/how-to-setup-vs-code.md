---
title: Configuración de la extensión de Visual Studio Code (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a configurar la extensión Azure Machine Learning para Visual Studio Code.
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.service: machine-learning
ms.subservice: core
ms.date: 05/25/2021
ms.topic: how-to
ms.openlocfilehash: 80f6849bb4989692e8cd29a3288f9b104bf250bb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110460092"
---
# <a name="set-up-the-visual-studio-code-azure-machine-learning-extension-preview"></a>Configuración de la extensión Azure Machine Learning para Visual Studio Code (versión preliminar)

Aprenda a configurar la extensión Azure Machine Learning para Visual Studio Code para los flujos de trabajo de aprendizaje automático.

> [!div class="mx-imgBorder"]
> ![Uso de la extensión para VS Code](./media/how-to-setup-vs-code/vs-code-extension.PNG)

La extensión Azure Machine Learning para VS Code proporciona una interfaz de usuario para:

- La administración de los recursos de Azure Machine Learning (experimentos, máquinas virtuales, modelos, implementaciones, etc.)
- El desarrollo de forma local mediante instancias de proceso remotas
- Entrenamiento de modelos de Machine Learning
- La depuración local de experimentos de aprendizaje automático
- Compatibilidad con lenguajes basados en esquemas, autocompletado y diagnóstico para la creación de archivos de especificación
- Fragmentos de código para tareas comunes

## <a name="prerequisites"></a>Requisitos previos

- Suscripción de Azure. Si no tiene una, regístrese para probar la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
- Código de Visual Studio. Si no lo tiene, [instálelo](https://code.visualstudio.com/docs/setup/setup-overview).
- [Python](https://www.python.org/downloads/)
- (Opcional) Para crear recursos mediante la extensión, debe instalar la CLI 2.0. Para obtener instrucciones, consulte [Instalación, configuración y uso de la CLI 2.0 (versión preliminar).](how-to-configure-cli.md)

## <a name="install-the-extension"></a>Instalación de la extensión

1. Abra Visual Studio Code.
1. Seleccione el icono **Extensiones** de la **barra de actividades** para abrir la vista Extensiones.
1. En la vista Extensiones, busque "Azure Machine Learning".
1. Seleccione **Instalar**.

    > [!div class="mx-imgBorder"]
    > ![Instalación de la extensión Azure Machine Learning para VS Code](./media/how-to-setup-vs-code/install-aml-vscode-extension.PNG)

> [!NOTE]
> Como alternativa, puede instalar la extensión Azure Machine Learning al [descargar el instalador directamente](https://aka.ms/vscodetoolsforai) desde Visual Studio Marketplace.

El resto de los pasos de este tutorial se han probado con la versión más reciente de la extensión.

> [!NOTE]
> La extensión Azure Machine Learning para VS Code usa la CLI 2.0 de forma predeterminada. Para cambiar a la experiencia de la CLI 1.0, defina la opción `azureML.CLI Compatibility Mode` en Visual Studio Code como `1.0`. Para más información sobre cómo modificar la configuración en Visual Studio, consulte la [documentación de la configuración de usuario y área de trabajo](https://code.visualstudio.com/docs/getstarted/settings).

## <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure

Para aprovisionar recursos y ejecutar cargas de trabajo en Azure, tiene que iniciar sesión con las credenciales de su cuenta de Azure. Para ayudar con la administración de cuentas, Azure Machine Learning instala automáticamente la extensión Azure Account. Visite el sitio siguiente para [más información sobre la extensión Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

Para iniciar sesión en su cuenta de Azure, seleccione el botón **Azure: Iniciar sesión** en la barra de estado de Visual Studio Code para iniciar el proceso de inicio de sesión.

Como alternativa, utilice la paleta de comandos:

1. Seleccione **Ver > Paleta de comandos** en la barra de menús para abrir la paleta de comandos.
1. Escriba el comando "> Azure: Iniciar sesión" en la paleta de comandos para empezar el proceso de inicio de sesión.

## <a name="choose-your-default-workspace"></a>Elección del área de trabajo predeterminada

La elección de un área de trabajo de Azure Machine Learning predeterminada permite lo siguiente al crear archivos de especificación YAML de la CLI 2.0:

- Validación de esquema
- Autocompletar
- Diagnóstico

Si aún no tiene un área de trabajo, créela ahora: Para más información, consulte [Administración de recursos de Azure Machine Learning con la extensión de VS Code (versión preliminar)](how-to-manage-resources-vscode.md).

Para elegir el área de trabajo predeterminada, seleccione el botón **Set Azure ML Workspace** (Establecer área de trabajo de Azure Machine Learning) en la barra de estado de Visual Studio Code y siga las indicaciones para establecer el área de trabajo.

Como alternativa, use el comando `> Azure ML: Set Default Workspace` de la paleta de comandos y siga las indicaciones para establecer el área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

- [Administrar los recursos de Azure Machine Learning](how-to-manage-resources-vscode.md)
- [Conexión a una instancia de proceso remoto de manera local](how-to-set-up-vs-code-remote.md)
- [Uso de instancias de proceso como servidor remoto de Jupyter](how-to-set-up-vs-code-remote.md)
- [Entrenamiento de un modelo de clasificación de imágenes mediante la extensión de Visual Studio Code](tutorial-train-deploy-image-classification-model-vscode.md)
- [Ejecución y depuración local de experimentos de aprendizaje automático](how-to-debug-visual-studio-code.md)