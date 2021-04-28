---
title: Creación y exploración de conjuntos de datos con etiquetas
titleSuffix: Azure Machine Learning
description: Aprenda a exportar etiquetas de datos desde los proyectos de etiquetado de Azure Machine Learning y a usarlas para tareas de aprendizaje automático.
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: data4ml
ms.date: 05/14/2020
ms.openlocfilehash: 7fc4a36cc8fedaa5de704f668f8b15aecac98cd7
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108140522"
---
# <a name="create-and-explore-azure-machine-learning-dataset-with-labels"></a>Creación y exploración de un conjunto de datos de Azure Machine Learning con etiquetas

En este artículo, aprenderá a exportar las etiquetas de datos desde un proyecto de etiquetado de datos de Azure Machine Learning y a cargarlos en formatos populares como tramas de datos de Pandas para la exploración de datos o conjuntos de datos de Torchvision para la transformación de imágenes. 

## <a name="what-are-datasets-with-labels"></a>Qué son los conjuntos de datos con etiquetas 

Los conjuntos de datos de Azure Machine Learning se conocen como conjuntos de datos con etiquetas. Estos tipos específicos de conjuntos de datos con etiquetas solo se crean como salida de los proyectos de etiquetado de datos de Azure Machine Learning. Cree un proyecto de etiquetado de datos con [estos pasos](how-to-create-labeling-projects.md). Machine Learning admite proyectos de etiquetado de datos para la clasificación de imágenes (de varias etiquetas o varias clases) y la identificación de objetos mediante rectángulos de selección.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro) o acceso a [Azure Machine Learning Studio](https://ml.azure.com/).
    * Instalación del paquete [azure-contrib-dataset](/python/api/azureml-contrib-dataset/)
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
* Acceda a un proyecto de etiquetado de datos de Azure Machine Learning. Si no tiene un proyecto de etiquetado, siga [estos pasos](how-to-create-labeling-projects.md) para crearlo.

## <a name="export-data-labels"></a>Exportación de etiquetas de datos 

Al completar un proyecto de etiquetado de datos, puede exportar los datos de las etiquetas de un proyecto de etiquetado. Esto le permite capturar tanto la referencia a los datos como sus etiquetas, y exportarlas en [formato de COCO](http://cocodataset.org/#format-data) o como un conjunto de datos de Azure Machine Learning. Use el botón **Exportar** en la página **Detalles del proyecto** del proyecto de etiquetado.

### <a name="coco"></a>COCO 

 El archivo COCO se crea en el almacén de blobs predeterminado del área de trabajo de Azure Machine Learning, en una carpeta dentro de *export/coco*. 
 
>[!NOTE]
>En los proyectos de detección de objetos, los valores exportados de "bbox: [x,y,width,height] "del archivo COCO se normalizan. Se modifica su escala a 1. Ejemplo: un rectángulo delimitador que se encuentre en la ubicación (10, 10) y tenga 30 píxeles de ancho y 60 de alto, en una imagen de 640 x 480 píxeles se anotará como (0,015625, 0,02083, 0,046875, 0,125). Como las coordenadas están normalizadas, se mostrarán como "0,0" como "width" (ancho) y "height" (alto) en todas las imágenes. El ancho y el alto reales se pueden obtener mediante una biblioteca de Python, como OpenCV o Pillow (PIL).

### <a name="azure-machine-learning-dataset"></a>Conjunto de datos de Azure Machine Learning

Puede acceder al conjunto de datos de Azure Machine Learning exportado en la sección **Conjuntos de datos** de Azure Machine Learning Studio. La página **Details** (Detalles) del conjunto de datos también proporciona código de ejemplo para acceder a las etiquetas desde Python.

![Conjunto de datos exportado](./media/how-to-create-labeling-projects/exported-dataset.png)

## <a name="explore-labeled-datasets"></a>Exploración de conjuntos de datos etiquetados

Cargue los conjuntos de datos con etiquetas en una trama de datos de Pandas o un conjunto de datos de Torchvision para aprovechar las conocidas bibliotecas de código abierto para la exploración de datos, así como las bibliotecas proporcionadas por PyTorch para la transformación y el entrenamiento de imágenes.

### <a name="pandas-dataframe"></a>Dataframe de Pandas

Puede cargar conjuntos de datos con etiqueta en una trama de datos de Pandas con el método [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) de la clase `azureml-contrib-dataset`. Instale la clase con el siguiente comando de shell: 

```shell
pip install azureml-contrib-dataset
```

>[!NOTE]
>El espacio de nombres azureml.contrib cambia con frecuencia, ya que estamos trabajando para mejorar el servicio. Por lo tanto, todo el contenido de este espacio de nombres debe considerarse como una versión preliminar, no completamente compatible con Microsoft.

Azure Machine Learning ofrece las siguientes opciones de administración de archivos para flujos de archivos al convertirlos en una trama de datos de Pandas.
* Descargar: descargue sus archivos de datos en una ruta de acceso local.
* Montar: monte los archivos de datos en un punto de montaje. El montaje solo funciona para el proceso basado en Linux, lo que incluye la máquina virtual de cuadernos de Azure Machine Learning y Proceso de Azure Machine Learning.

En el código siguiente, el conjunto de datos `animal_labels` es la salida de un proyecto de etiquetado guardado anteriormente en el área de trabajo.

```Python
import azureml.core
import azureml.contrib.dataset
from azureml.core import Dataset, Workspace
from azureml.contrib.dataset import FileHandlingOption

# get animal_labels dataset from the workspace
animal_labels = Dataset.get_by_name(workspace, 'animal_labels')
animal_pd = animal_labels.to_pandas_dataframe(file_handling_option=FileHandlingOption.DOWNLOAD, target_path='./download/', overwrite_download=True)

import matplotlib.pyplot as plt
import matplotlib.image as mpimg

#read images from downloaded path
img = mpimg.imread(animal_pd.loc[0,'image_url'])
imgplot = plt.imshow(img)
```

### <a name="torchvision-datasets"></a>Conjuntos de datos de Torchvision

Puede cargar conjuntos de datos con etiquetas en el conjunto de datos de Torchvision con el método [to_torchvision()](/python/api/azureml-contrib-dataset/azureml.contrib.dataset.tabulardataset#to-torchvision--) también desde la clase `azureml-contrib-dataset`. Para usar este método, debe tener [PyTorch](https://pytorch.org/) instalado. 

En el código siguiente, el conjunto de datos `animal_labels` es la salida de un proyecto de etiquetado guardado anteriormente en el área de trabajo.

```python
import azureml.core
import azureml.contrib.dataset
from azureml.core import Dataset, Workspace
from azureml.contrib.dataset import FileHandlingOption

from torchvision.transforms import functional as F

# get animal_labels dataset from the workspace
animal_labels = Dataset.get_by_name(workspace, 'animal_labels')

# load animal_labels dataset into torchvision dataset
pytorch_dataset = animal_labels.to_torchvision()
img = pytorch_dataset[0][0]
print(type(img))

# use methods from torchvision to transform the img into grayscale
pil_image = F.to_pil_image(img)
gray_image = F.to_grayscale(pil_image, num_output_channels=3)

imgplot = plt.imshow(gray_image)
```

## <a name="next-steps"></a>Pasos siguientes
