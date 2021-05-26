---
title: Extensión de una imagen de Docker precompilada
titleSuffix: Azure Machine Learning
description: Extensión de imágenes de Docker precompiladas en Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: ssambare
author: shivanissambare
ms.date: 05/25/2021
ms.topic: how-to
ms.reviewer: larryfr
ms.custom: deploy, docker, prebuilt
ms.openlocfilehash: 2c410d6c310c8f9b0c50ff0073f4dda9a35c8f9c
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382925"
---
# <a name="extend-a-prebuilt-docker-image-preview"></a>Extensión de una imagen de Docker precompilada (versión preliminar)

En algunos casos, es posible que las [imágenes de Docker precompiladas para la inferencia de modelos](concept-prebuilt-docker-images-inference.md) y las soluciones de [extensibilidad](./how-to-prebuilt-docker-images-inference-python-extensibility.md) para Azure Machine Learning no satisfagan sus necesidades de servicio de inferencia.

En tal caso, puede usar un Dockerfile para crear una imagen nueva, usando una de las imágenes precompiladas como punto de partida. Al extender desde una imagen de Docker precompilada existente, puede usar la pila de red y las bibliotecas de Azure Machine Learning sin crear una imagen desde cero.

> [!IMPORTANT]
> El uso de imágenes de Docker precompiladas con Azure Machine Learning está actualmente en versión preliminar. La funcionalidad de versión preliminar se proporciona "tal cual", sin ninguna garantía de soporte técnico ni contrato de nivel de servicio. Para obtener más información, consulte [Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

**Ventajas e inconvenientes**

El uso de un Dockerfile permite la personalización completa de la imagen antes de la implementación. Permite tener el máximo control sobre qué dependencias o variables de entorno, entre otras cosas, se establecen en el contenedor.

El principal inconveniente de este enfoque es que se realizará una compilación de imágenes adicional durante la implementación, lo que ralentiza el proceso de implementación. Si puede usar el método de [extensibilidad de paquetes de Python](./how-to-prebuilt-docker-images-inference-python-extensibility.md), la implementación será más rápida.
## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para ver un tutorial sobre la creación de un área de trabajo, consulte [Introducción a Azure Machine Learning](quickstart-create-resources.md).
* Familiaridad con la creación de un [Dockerfile](https://docs.docker.com/engine/reference/builder/).
* Una instalación de trabajo local de [Docker](https://www.docker.com/), incluida la CLI `docker`, **O** una instancia de Azure Container Registry (ACR) asociada con el área de trabajo de Azure Machine Learning.

    > [!WARNING]
    > La instancia de Azure Container Registry para el área de trabajo se crea la primera vez que entrena o implementa un modelo con el área de trabajo. Si ha creado una nueva área de trabajo, pero no ha entrenado ni ha creado ningún modelo, no existirá ninguna instancia de Azure Container Registry para el área de trabajo.
## <a name="create-and-build-dockerfile"></a>Creación y compilación de Dockerfile

A continuación se muestra un Dockerfile de ejemplo que usa una imagen de Docker precompilada de Azure Machine Learning como una imagen base:

```Dockerfile
FROM mcr.microsoft.com/azureml/<image_name>:<tag>

COPY requirements.txt /tmp/requirements.txt

RUN pip install –r /tmp/requirements.txt
```

A continuación, coloque el Dockerfile anterior en el directorio con todos los archivos necesarios y ejecute el siguiente comando para compilar la imagen:

```bash
docker build -f <above dockerfile> -t <image_name>:<tag> .
```

> [!TIP]
> Se puede encontrar más información acerca de `docker build` en la [documentación de Docker](https://docs.docker.com/engine/reference/commandline/build/).

Si el comando `docker build` no está disponible localmente, use Azure Container Registry (ACR) para que el área de trabajo de Azure Machine Learning compile la imagen de Docker en la nube. Para más información, consulte [Tutorial: Compilación e implementación de imágenes de contenedor con Azure Container Registry](/azure/container-registry/container-registry-tutorial-quick-task).

> [!IMPORTANT]
> Microsoft recomienda que primero valide que el Dockerfile funciona localmente antes de intentar crear una imagen base personalizada mediante Azure Container Registry.

Las secciones siguientes contienen detalles más específicos sobre el Dockerfile.

## <a name="install-extra-packages"></a>Instalación de paquetes adicionales

Si es necesario instalar más paquetes `apt` en el contenedor de Ubuntu, puede agregarlos al Dockerfile. En el ejemplo siguiente se muestra cómo usar el comando `apt-get` desde un Dockerfile:

```Dockerfile
FROM <prebuilt docker image from MCR>

# Switch to root to install apt packages
USER root:root

RUN apt-get update && \
    apt-get install -y \
    <package-1> \
    ... 
    <package-n> && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/*

# Switch back to non-root user
USER dockeruser
```

También puede instalar paquetes pip adicionales desde un Dockerfile. El ejemplo siguiente muestra cómo usar `pip install`:

```Dockerfile
RUN pip install <library>
```

<a id="buildmodel"></a>

## <a name="build-model-and-code-into-images"></a>Compilación del modelo y el código en imágenes

Si el modelo y el código deben integrarse en la imagen, se deben establecer las siguientes variables de entorno en el Dockerfile:

* `AZUREML_ENTRY_SCRIPT`: script de entrada del código. Este archivo contiene los métodos `init()` y `run()`.
* `AZUREML_MODEL_DIR`: directorio que contiene los archivos del modelo. El script de entrada debe usar este directorio como directorio raíz del modelo.

En el ejemplo siguiente se muestra cómo establecer estas variables de entorno en el Dockerfile:

```Dockerfile
FROM <prebuilt docker image from MCR>

# Code
COPY <local_code_directory> /var/azureml-app
ENV AZUREML_ENTRY_SCRIPT=<entryscript_file_name>

# Model
COPY <model_directory> /var/azureml-app/azureml-models
ENV AZUREML_MODEL_DIR=/var/azureml-app/azureml-models
```

## <a name="example-dockerfile"></a>Dockerfile de ejemplo

En el ejemplo siguiente se muestra cómo instalar paquetes `apt`, establecer variables de entorno e incluir código y modelos como parte del Dockerfile:

```Dockerfile
FROM mcr.microsoft.com/azureml/pytorch-1.6-ubuntu18.04-py37-cpu-inference:latest 

USER root:root

# Install libpng-tools and opencv
RUN apt-get update && \
    apt-get install -y \
    libpng-tools \
    python3-opencv && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/*

# Switch back to non-root user
USER dockeruser

# Code
COPY code /var/azureml-app
ENV AZUREML_ENTRY_SCRIPT=score.py

# Model
COPY model /var/azureml-app/azureml-models
ENV AZUREML_MODEL_DIR=/var/azureml-app/azureml-models
```

## <a name="next-steps"></a>Pasos siguientes

Para usar un Dockerfile con el SDK de Azure Machine Learning de Python, consulte los siguientes documentos:

* [Uso de un Dockerfile local propio](how-to-use-environments.md#use-your-own-dockerfile)
* [Uso de una imagen de Docker precompilada y creación de una imagen base personalizada](how-to-use-environments.md#use-a-prebuilt-docker-image)

Para más información sobre la implementación de un modelo, consulte [Implementación de un modelo](how-to-deploy-and-where.md).

Para obtener información sobre cómo solucionar problemas de implementaciones de imágenes de Docker precompiladas, consulte [Solución de problemas de implementaciones de imágenes de Docker precompiladas](how-to-troubleshoot-prebuilt-docker-image-inference.md).