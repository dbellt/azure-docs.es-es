---
title: 'Tutorial: Introducción a los cuadernos de Jupyter Notebook (Python)'
titleSuffix: Azure Machine Learning
description: Tutoriales para la configuración de Jupyter Notebook. Cree un área de trabajo, clone los cuadernos en ella y cree una instancia de proceso donde ejecutarlos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 02/10/2020
ms.custom: devx-track-python, contperf-fy21q3
adobe-target: true
ms.openlocfilehash: ed7a95ea83cacc0eaccfa23cd87403fb9be615ee
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210676"
---
# <a name="tutorial-get-started-with-azure-machine-learning-in-jupyter-notebooks"></a>Tutorial: Introducción a Azure Machine Learning en cuadernos de Jupyter Notebook

En este tutorial completará los pasos para empezar a trabajar con Azure Machine Learning mediante cuadernos de Jupyter Notebook en una [estación de trabajo (instancia de proceso) administrada basada en la nube](concept-compute-instance.md). Este tutorial es un precursor de todos los demás tutoriales de Jupyter Notebook.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Crear un [área de trabajo de Azure Machine Learning](concept-workspace.md) para usarla en otros tutoriales de Jupyter Notebook.
> * Clone el cuaderno de tutoriales en su carpeta en el área de trabajo.
> * Cree una instancia de proceso basada en la nube, que le proporciona un entorno con el SDK de Python de Azure Machine Learning ya instalado y configurado automáticamente.

Este tutorial le prepara para ejecutar cuadernos de Jupyter Notebook en un recurso de proceso del área de trabajo.  

Otra excelente manera de empezar a usar Azure Machine Learning es mediante el envío de trabajos por lotes.  Consulte [Tutorial: Introducción a Azure Machine Learning en el entorno de desarrollo](tutorial-1st-experiment-sdk-setup-local.md).

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

## <a name="create-a-workspace"></a>Crear un área de trabajo

Un área de trabajo de Azure Machine Learning es un recurso básico de la nube que se usa para experimentar, entrenar e implementar modelos de aprendizaje automático. Vincula la suscripción y el grupo de recursos de Azure con un objeto fácilmente consumido del servicio.

Si ya tiene un área de trabajo de Azure Machine Learning, vaya a [Clone a notebook folder](#clone) (Clonar una carpeta de cuaderno).  

Existen muchas [maneras de crear un área de trabajo](how-to-manage-workspace.md).  En este tutorial, creará un área de trabajo mediante Azure Portal, una consola basada en web para administrar los recursos de Azure.

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

>[!IMPORTANT]
> Tome nota del *área de trabajo* y de la *suscripción*. Necesitará esta información para asegurarse de que crea el experimento en el lugar correcto.

## <a name="run-a-notebook-in-your-workspace"></a><a name="azure"></a>Ejecución de un cuaderno en el área de trabajo

Azure Machine Learning incluye un servidor de cuadernos en la nube del área de trabajo para obtener una experiencia sin instalación y configurada previamente. Si prefiere tener control sobre su entorno, los paquetes y las dependencias, use [su propio entorno](tutorial-1st-experiment-sdk-setup-local.md).

 Siga este vídeo o use los pasos detallados para clonar y ejecutar el cuaderno del tutorial desde el área de trabajo.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4mTUr]

### <a name="clone-a-notebook-folder"></a><a name="clone"></a> Clonación de una carpeta de cuaderno

Complete la configuración del experimento siguiente y ejecute los pasos de Azure Machine Learning Studio. Esta interfaz consolidada incluye herramientas de aprendizaje automático para realizar escenarios de ciencia de datos para los profesionales de ciencia de datos con conocimientos de todos los niveles.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).

1. Seleccione la suscripción y el área de trabajo que ha creado.

1. Seleccione **Notebooks** en la parte izquierda.

1. Seleccione la pestaña **Ejemplos** en la parte superior.

1. Abra la carpeta **Python**.

1. Abra la carpeta con un número de versión. Este número representa la versión actual del SDK de Python.

1. Seleccione el botón **"..."** a la derecha de la carpeta **tutorials** (tutoriales) y **Clone** (Clonar).

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/clone-tutorials.png" alt-text="Captura de pantalla que muestra la carpeta tutorials > Clone.":::

1. En una lista de carpetas se muestran los usuarios que acceden al área de trabajo. Seleccione la carpeta donde se va a clonar la carpeta **tutorials**.

### <a name="open-the-cloned-notebook"></a><a name="open"></a>Apertura del cuaderno clonado

1. Abra la carpeta **tutorials** que se acaba de cerrar en la sección **User files** (Archivos de usuario).

    > [!IMPORTANT]
    > Verá los cuadernos en la carpeta **samples**, pero no puede ejecutar cuadernos desde aquí. Para ejecutar un cuaderno, asegúrese de que abre la versión clonada de este en la sección **User Files** (Archivos de usuario).
    
1. Seleccione el archivo **img-classification-part1-training.ipynb** en la carpeta **tutorials/image-classification-mnist-data**.

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/expand-user-folder.png" alt-text="Captura de pantalla que muestra la carpeta tutorials abierta.":::

1. En la barra superior, seleccione una instancia de proceso para usarla para ejecutar el cuaderno. Estas máquinas virtuales están configuradas previamente con [todo lo necesario para ejecutar Azure Machine Learning](concept-compute-instance.md#contents).

1. Si no encuentra ninguna máquina virtual, seleccione **+ Add** (+ Agregar) para crear la máquina virtual de instancia de proceso.

    1. Al crear una máquina virtual, siga estas reglas:
 
        + El nombre es obligatorio y el campo no puede estar vacío.
        + El nombre debe ser único (no se distinguen mayúsculas de minúsculas) en todas las instancias de proceso existentes en la región de Azure del área de trabajo o la instancia de proceso. En caso contrario, recibirá una alerta.
        + Los caracteres válidos son las letras mayúsculas y minúsculas, los números (del 0 al 9) y los guiones (-).
        + El nombre debe tener entre 3 y 24 caracteres.
        + El nombre debe empezar por una letra (no un número ni un guion).
        + Si se usa el guion, debe ir seguido al menos de una letra. Por ejemplo, test-, test-0, test-01 no son válidos, mientras que test-a0 y test-0a son ejemplos válidos.

    1. Seleccione el tamaño de la máquina virtual de entre las opciones disponibles. En los tutoriales, la máquina virtual predeterminada es una buena elección.

    1. Seleccione **Crear**. La configuración de la máquina virtual puede tardar aproximadamente cinco minutos.

1. Cuando la máquina virtual está disponible, aparece en la barra de herramientas superior. Ahora puede ejecutar el cuaderno con **Run all** (Ejecutar todo) de la barra de herramientas o **Mayús + Entrar** en las celdas de código del cuaderno.

Si tiene widgets personalizados o prefiere usar Jupyter o JupyterLab, seleccione la lista desplegable **Jupyter** en el extremo derecho y, después, seleccione **Jupyter** o **JupyterLab**. Se abre la nueva ventana del explorador.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un entorno de desarrollo configurado, continúe con el entrenamiento de un modelo en un cuaderno de Jupyter Notebook.

> [!div class="nextstepaction"]
> [Tutorial: Entrenamiento de modelos de clasificación de imágenes con los datos MNIST y scikit-learn](tutorial-train-models-with-aml.md)

<a name="stop-compute-instance"></a> Si no pretende seguir ningún otro tutorial ahora, detenga la máquina virtual del servidor de cuadernos en la nube cuando no la use para reducir el costo.

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]
