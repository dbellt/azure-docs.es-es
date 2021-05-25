---
title: Cuadernos de Jupyter Notebook de ejemplo
titleSuffix: Azure Machine Learning
description: Aprenda a encontrar y usar los cuadernos de Jupyter Notebook diseñados para que le ayuden a explorar el SDK y sirvan como modelos para sus propios proyectos de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: sample
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.date: 03/05/2020
ms.custom: seodec18
ms.openlocfilehash: ac0b1b78291d895e97cb967e047ae36da6ff7469
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108746844"
---
# <a name="explore-azure-machine-learning-with-jupyter-notebooks"></a>Exploración de Azure Machine Learning con cuadernos de Jupyter Notebook

> [!NOTE] 
> Puede encontrar un repositorio de ejemplos controlado por la comunidad en https://github.com/Azure/azureml-examples.

El repositorio de [cuadernos de Azure Machine Learning](https://github.com/azure/machinelearningnotebooks) incluye los ejemplos más recientes del SDK de Python para Azure Machine Learning. Estos cuadernos de Juypter Notebook están diseñados para ayudarle a explorar el SDK y para servir como modelos para sus propios proyectos de aprendizaje automático.

Este artículo le muestra cómo acceder al repositorio desde los siguientes entornos:

- [Instancia de proceso de Azure Machine Learning](#notebookvm)
- [Servidor de Notebook propio](#byo)
- [Data Science Virtual Machine](#dsvm)

> [!NOTE]
> Una vez que haya clonado el repositorio, encontrará los cuadernos del tutorial en la carpeta **tutorials** y los cuadernos de características específicas en la carpeta **how-to-use-azureml**.

<a name="notebookvm"></a>
## <a name="get-samples-on-azure-machine-learning-compute-instance"></a>Obtener ejemplos de la instancia de proceso de Azure Machine Learning

La forma más fácil de empezar con los ejemplos es completar el [inicio rápido: Introducción a Azure Machine Learning](quickstart-create-resources.md). Una vez completado, tendrá un servidor de cuadernos en el que se habrá cargado previamente el SDK y el repositorio de ejemplos. No es necesario realizar ninguna descarga ni instalación.

<a name="byo"></a>

## <a name="get-samples-on-your-notebook-server"></a>Obtener ejemplos del servidor de cuadernos

Si desea usar su propio servidor de Jupyter Notebook para desarrollo local, siga estos pasos:

[!INCLUDE [aml-your-server](../../includes/aml-your-server.md)]

Estas instrucciones le permitirán instalar los paquetes básicos del SDK necesarios para los cuadernos del inicio rápido y el tutorial. Otros cuadernos de ejemplo pueden requerir la instalación de componentes adicionales. Para más información, consulte [Install the Azure Machine Learning SDK for Python](/python/api/overview/azure/ml/install) (Instalación del SDK de Azure Machine Learning para Python).

<a name="dsvm"></a>
## <a name="get-samples-on-dsvm"></a>Obtener ejemplos de DSVM

Data Science Virtual Machine (DSVM) es una imagen de máquina virtual personalizada diseñada específicamente para realizar ciencia de datos. Si [crea una instancia de DSVM](how-to-configure-environment.md#dsvm), el SDK y el servidor de cuadernos se instalarán y configurarán automáticamente. No obstante, tendrá que crear un área de trabajo y clonar el repositorio de ejemplo.

[!INCLUDE [aml-dsvm-server](../../includes/aml-dsvm-server.md)]

## <a name="next-steps"></a>Pasos siguientes

Explore los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks) para descubrir lo que Azure Machine Learning puede hacer.

Puede encontrar más ejemplos y proyectos de ejemplo de GitHub en estos repositorios:
+ [Azure/azureml-examples](https://github.com/Azure/azureml-examples)
+ [Microsoft/MLOps](https://github.com/Microsoft/MLOps)
+ [Microsoft/MLOpsPython](https://github.com/microsoft/MLOpsPython)

Pruebe estos tutoriales:

- [Entrenamiento de un modelo de clasificación de imágenes con el servicio Azure Machine Learning](tutorial-train-models-with-aml.md)

- [Preparación de datos y uso de Machine Learning automatizado para entrenar un modelo de regresión con el conjunto de datos de los taxis de Nueva York](tutorial-auto-train-models.md)
