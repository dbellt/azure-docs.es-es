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
ms.openlocfilehash: 1bea195d6f08832415a16531212d176ada7402e6
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110612581"
---
# <a name="azure-machine-learning-curated-environments"></a>Entornos mantenidos de Azure Machine Learning

En este artículo se enumeran los entornos mantenidos disponibles en Azure Machine Learning. Los entornos mantenidos los proporciona Azure Machine Learning y están disponibles en el área de trabajo de forma predeterminada. Están respaldados por imágenes de Docker en caché que usan la versión más reciente del SDK de Azure Machine Learning, lo que reduce el costo de preparación de la ejecución y permite un tiempo de implementación más rápido. Use estos entornos para empezar a trabajar rápidamente con varios marcos de aprendizaje automático.

> [!NOTE]
> Esta lista está actualizada a abril de 2021. Use la CLI o el SDK de Python para obtener la lista más actualizada de entornos y sus dependencias. Para más información, consulte el [artículo sobre los entornos](./how-to-use-environments.md#use-a-curated-environment). Después del lanzamiento de este conjunto nuevo, se ocultarán los entornos mantenidos anteriormente, pero se podrán seguir usando. 

## <a name="pytorch"></a>PyTorch
- AzureML-pytorch-1.7-ubuntu18.04-py37-cuda11-gpu
     - Un entorno de aprendizaje profundo con PyTorch que contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de PyTorch: 1.7
     - Versión de Python: 3.7
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - Versión de OpenMPI: 4.1.0
     - Versión de Ubuntu: 18.04

## <a name="sklearn"></a>Sklearn
- AzureML-sklearn-0.24-ubuntu18.04-py37-cuda11-gpu
     - Un entorno para tareas como regresión, agrupación en clústeres y clasificación con Scikit-learn. Contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de Scikit-learn: 24.1
     - Versión de Python: 3.7
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - Versión de OpenMPI: 4.1.0
     - Versión de Ubuntu: 18.04

## <a name="tensorflow"></a>TensorFlow
- AzureML-tensorflow-2.4-ubuntu18.04-py37-cuda11-gpu
     - Un entorno de aprendizaje profundo con Tensorflow que contiene el SDK de Azure ML y paquetes adicionales de Python.
     - Versión de Tensorflow: 2.4
     - Versión de Python: 3.7
     - Imagen base: mcr.microsoft.com/azureml/openmpi4.1.0-cuda11.0.3-cudnn8-ubuntu18.04
     - Versión de CUDA: 11.0.3
     - Versión de OpenMPI: 4.1.0
     - Versión de Ubuntu: 18.04

## <a name="inference-only-curated-environments-and-prebuilt-docker-images"></a>Entornos mantenidos de solo inferencia e imágenes de Docker precompiladas
- Aprenda sobre los entornos mantenidos de solo inferencia y la ruta de acceso MCR de las imágenes Docker precompiladas. Consulte [Imágenes de Docker precompiladas para inferencia](concept-prebuilt-docker-images-inference.md#list-of-prebuilt-docker-images-for-inference).
