---
title: Creación y administración de archivos en el área de trabajo
titleSuffix: Azure Machine Learning
description: Aprenda a crear y administrar archivos en el área de trabajo de Azure Machine Learning Studio.
services: machine-learning
author: abeomor
ms.author: osomorog
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 02/05/2021
ms.openlocfilehash: e1868794f7e47ccce3efa36b7954a1a2a704e8d6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108739032"
---
# <a name="how-to-create-and-manage-files-in-your-workspace"></a>Cómo crear y administrar archivos en el área de trabajo

Aprenda a crear y administrar los archivos en el área de trabajo de Azure Machine Learning.  Estos archivos se almacenan en el almacenamiento del área de trabajo predeterminado. Los archivos y las carpetas se pueden compartir con otros usuarios con acceso de lectura en el área de trabajo y se pueden usar desde cualquier instancia de proceso en el área de trabajo.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="create-files"></a><a name="create"></a> Creación de archivos

Para crear un archivo en la carpeta predeterminada (`Users > yourname`):

1. En [Azure Machine Learning Studio](https://ml.azure.com), abra el área de trabajo.
1. En el lado izquierdo, seleccione **Notebooks**.
1. Seleccione la imagen **+** .
1. Seleccione la imagen **Crear archivo**.

    :::image type="content" source="media/how-to-run-jupyter-notebooks/create-new-file.png" alt-text="Crear un nuevo archivo":::

1. Dele un nombre al archivo.
1. Seleccione un tipo de archivo.
1. Seleccione **Crear**.

Los cuadernos y la mayoría de los tipos de archivo de texto se muestran en la sección de vista previa.  La mayoría de los demás tipos de archivo no tienen una vista previa.

Para crear un archivo en una carpeta diferente:
1. Seleccione los puntos suspensivos ("...") al final de la carpeta
1. Seleccione **Create new file** (Crear archivo).

> [!IMPORTANT]
> El contenido de los cuadernos y scripts puede leer los datos de las sesiones y acceder a los datos sin su organización en Azure.  Cargar solo archivos de fuentes de confianza. Para más información, vea [Prácticas recomendadas de código seguro](concept-secure-code-best-practice.md#azure-ml-studio-notebooks).

## <a name="manage-files-with-git"></a>Administración de archivos con Git

[Use un terminal de instancias de proceso](how-to-access-terminal.md#git) para clonar y administrar repositorios de Git.

## <a name="clone-samples"></a>Clonación de ejemplos

El área de trabajo contiene una carpeta **Sample notebooks** con cuadernos diseñados para ayudarle a explorar el SDK y que sirven de ejemplo para sus propios proyectos de aprendizaje automático.   Clone estos cuadernos en su propia carpeta para ejecutarlos y editarlos.  

Para obtener un ejemplo, vea [Tutorial: Creación del primer experimento de Machine Learning](tutorial-train-models-with-aml.md#azure).

## <a name="share-files"></a>Uso compartido de archivos

Copie y pegue la dirección URL para compartir un archivo.  Solo otros usuarios del área de trabajo pueden acceder a esta dirección URL.  Obtenga más información sobre la [concesión de acceso a su área de trabajo](how-to-assign-roles.md).

## <a name="delete-a-file"></a>Eliminación de un archivo

*No puede* eliminar los archivos de **Sample notebooks**.  Estos cuadernos forman parte de Studio y se actualizan cada vez que se publica un nuevo SDK.  

*Puede* eliminar los archivos que se encuentran en la sección **Archivos** de cualquiera de estas maneras:

* En Studio, seleccione los puntos **...** al final de una carpeta o un archivo.  Asegúrese de usar un explorador compatible (Microsoft Edge, Chrome o Firefox).
* [Use un terminal](how-to-access-terminal.md) desde cualquier instancia de proceso del área de trabajo. La carpeta **~/cloudfiles** está asignada a almacenamiento en la cuenta de almacenamiento del área de trabajo.
* En Jupyter o JupyterLab con sus herramientas.
