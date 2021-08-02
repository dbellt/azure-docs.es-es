---
title: Marcos de aprendizaje profundo y AI
titleSuffix: Azure Data Science Virtual Machine
description: Disponibilidad de los marcos de aprendizaje profundo y las herramientas en Azure Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: data-science-vm
ms.custom: devx-track-python
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: 3272c2584d03c6f13d2ffb85eb0b37bf3c29000a
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110081629"
---
# <a name="deep-learning-and-ai-frameworks-for-the-azure-data-science-vm"></a>Marcos de aprendizaje profundo e inteligencia artificial para Azure Data Science VM
La lista a continuación enumera los marcos de aprendizaje profundo en DSVM.


## <a name="cuda-cudnn-nvidia-driver"></a>[CUDA, cuDNN, NVIDIA Driver](https://developer.nvidia.com/cuda-toolkit)

| Category | Value |
|--|--|
| Versiones compatibles | 11 |
| Ediciones de DSVM admitidas | Windows Server 2019<br>Ubuntu 18.04 |
| ¿Cómo se configura/instala en DSVM? | _nvidia-smi_ está disponible en la ruta de acceso del sistema. |
| Cómo ejecutarlo | Abra un símbolo del sistema (en Windows) o un terminal (en Linux) y ejecute _nvidia-smi_. |
## <a name="horovod"></a>[Horovod](https://github.com/uber/horovod)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 0.21.3|
| Ediciones de DSVM admitidas      | Ubuntu 18.04 |
| ¿Cómo se configura/instala en DSVM?  | Horovod se instala en Python 3.5 |
| Cómo ejecutarlo      | Active el entorno correcto en el terminal y, luego, ejecute Python. |


## <a name="nvidia-system-management-interface-nvidia-smi"></a>[NVidia System Management Interface (nvidia-smi)](https://developer.nvidia.com/nvidia-system-management-interface)

| Category | Value |
|--|--|
| Versiones compatibles |  |
| Ediciones de DSVM admitidas | Windows Server 2019<br>Ubuntu 18.04 |
| ¿Para qué sirven? | Herramienta NVIDIA para consultar la actividad GPU |
| ¿Cómo se configura/instala en DSVM? | `nvidia-smi` está en la ruta de acceso del sistema. |
| Cómo ejecutarlo | En una máquina virtual **con GPU**, abra un símbolo del sistema (en Windows) o un terminal (en Linux), y ejecute `nvidia-smi`. |

## <a name="pytorch"></a>[PyTorch](https://pytorch.org/)

| Category | Value |
|--|--|
| Versiones compatibles | 1.8.1 (Ubuntu 18.04, Windows 2019) |
| Ediciones de DSVM admitidas | Windows Server 2019<br>Ubuntu 18.04 |
| ¿Cómo se configura/instala en DSVM? | Instalado en Python, el entorno de Conda "py38_pytorch" |
| Cómo ejecutarlo | Terminal: active el entorno correcto y ejecute Python.<br/>* [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine): conéctese y abra el directorio de PyTorch para obtener ejemplos. |

## <a name="tensorflow"></a>[TensorFlow](https://www.tensorflow.org/)

| Category | Value |
|--|--|
| Versiones compatibles | 2.4 |
| Ediciones de DSVM admitidas | Windows Server 2019<br>Ubuntu 18.04 |
| ¿Cómo se configura/instala en DSVM? | Instalado en Python, el entorno de Conda "py38_tensorflow" |
| Cómo ejecutarlo | Terminal: active el entorno correcto y ejecute Python. <br/> *Jupyter: conéctese a [Jupyter](provision-vm.md) o [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y abra el directorio TensorFlow para acceder a los ejemplos. |
