---
title: Entornos mantenidos
titleSuffix: Azure Machine Learning
description: Obtenga información acerca de los entornos mantenidos de Azure Machine Learning, un conjunto de entornos preconfigurados que ayudan a reducir los tiempos de preparación del experimento y la implementación.
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.date: 4/2/2021
ms.openlocfilehash: 16d5cbc1d48b4559ac34314582b9c01b7c6bb58c
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166656"
---
# <a name="azure-machine-learning-curated-environments"></a>Entornos mantenidos de Azure Machine Learning

En este artículo se enumeran los entornos mantenidos disponibles en Azure Machine Learning. Los entornos mantenidos los proporciona Azure Machine Learning y están disponibles en el área de trabajo de forma predeterminada. Están respaldados por imágenes de Docker en caché que usan la versión más reciente del SDK de Azure Machine Learning, lo que reduce el costo de preparación de la ejecución y permite un tiempo de implementación más rápido. Use estos entornos para empezar a trabajar rápidamente con varios marcos de aprendizaje automático.

> [!NOTE]
> Esta lista está actualizada a abril de 2021. Use la CLI o el SDK de Python para obtener la lista más actualizada de entornos y sus dependencias. Para más información, consulte el [artículo sobre los entornos](./how-to-use-environments.md#use-a-curated-environment). Después del lanzamiento de este conjunto nuevo, se ocultarán los entornos mantenidos anteriormente, pero se podrán seguir usando. 

## <a name="pytorch"></a>PyTorch
- AzureML-Pytorch1.7-Cuda11-OpenMpi4.1.0-py36
     - Un entorno de aprendizaje profundo con PyTorch que contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de PyTorch: 1.7
     - Versión de Python: 3.6.9
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - OpenMPI: 4.1.0

## <a name="scikit"></a>Scikit
- AzureML-Scikit-learn0.20.4-Cuda11-OpenMpi4.1.0-py36
     - Un entorno para tareas como regresión, agrupación en clústeres y clasificación con Scikit-learn. Contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de Scikit-learn: 20.4
     - Versión de Python: 3.6.9
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - OpenMPI: 4.1.0

## <a name="tensorflow"></a>TensorFlow
- AzureML-TensorFlow2.4-Cuda11-OpenMpi4.1.0-py36
     - Un entorno de aprendizaje profundo con Tensorflow que contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de Tensorflow: 2.4
     - Versión de Python: 3.6.9
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - OpenMPI: 4.1.0
