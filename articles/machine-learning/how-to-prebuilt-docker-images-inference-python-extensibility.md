---
title: Extensibilidad de Python de la imagen de Docker precompilada
titleSuffix: Azure Machine Learning
description: Extensión de imágenes de Docker precompiladas con la solución de extensibilidad de paquetes de Python
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: ssambare
author: shivanissambare
ms.date: 05/25/2021
ms.topic: how-to
ms.reviewer: larryfr
ms.custom: deploy, docker, prebuilt
ms.openlocfilehash: 8af27f876f1c325cf99214e36f680e012e86c98d
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536332"
---
# <a name="python-package-extensibility-for-prebuilt-docker-images-preview"></a>Extensibilidad de paquetes de Python para imágenes de Docker precompiladas (versión preliminar)

Las [imágenes de Docker precompiladas para la inferencia de modelos](concept-prebuilt-docker-images-inference.md) contienen paquetes para marcos de aprendizaje automático populares. Hay dos métodos que se pueden usar para agregar paquetes de Python __sin volver a recompilar la imagen de Docker__:

* [Instalación dinámica](#dynamic): este enfoque usa un archivo de [requisitos](https://pip.pypa.io/en/stable/cli/pip_install/#requirements-file-format) para restaurar automáticamente los paquetes de Python cuando se inicia el contenedor de Docker.

    Considere la posibilidad de usar este método para la __creación rápida de prototipos__. Cuando se inicia la imagen, los paquetes se restauran mediante el archivo `requirements.txt`. Este método aumenta el inicio de la imagen y debe esperar más tiempo para que la implementación pueda controlar las solicitudes.

* [Paquetes de Python preinstalados](#preinstalled): puede proporcionar un directorio que contiene paquetes de Python preinstalados. Durante la implementación, este directorio se monta en el contenedor para que lo use el script de entrada (`score.py`).

    Use este enfoque __para las implementaciones de producción__. Puesto que el directorio que contiene los paquetes se monta en la imagen, se puede usar incluso cuando las implementaciones no tienen acceso público a Internet. Por ejemplo, cuando se implementa en una instancia de Azure Virtual Network.

> [!IMPORTANT]
> El uso de imágenes de Docker precompiladas con Azure Machine Learning está actualmente en versión preliminar. La funcionalidad de versión preliminar se proporciona "tal cual", sin ninguna garantía de soporte técnico ni contrato de nivel de servicio. Para obtener más información, consulte [Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para ver un tutorial sobre la creación de un área de trabajo, consulte [Introducción a Azure Machine Learning](quickstart-create-resources.md).
* Familiaridad con el uso de [entornos](how-to-use-environments.md) de Azure Machine Learning.
* Familiaridad con [dónde y cómo se implementan los modelos](how-to-deploy-and-where.md) con Azure Machine Learning.

<a id="dynamic"></a>

## <a name="dynamic-installation"></a>Instalación dinámica

Este enfoque usa un archivo de [requisitos](https://pip.pypa.io/en/stable/cli/pip_install/#requirements-file-format) para restaurar automáticamente los paquetes de Python cuando se inicia la imagen.

Para ampliar la imagen de contenedor de Docker precompilada a través de un archivo requirements.txt, siga estos pasos:

1. Cree un archivo `requirements.txt` junto con el script `score.py`.
2. Agregue **todos** los paquetes necesarios al archivo `requirements.txt`.
3. Establezca la variable de entorno `AZUREML_EXTRA_REQUIREMENTS_TXT` del [entorno](how-to-use-environments.md) de Azure Machine Learning en la ubicación del archivo `requirements.txt`.

Una vez implementados, los paquetes se restaurarán automáticamente para el script de puntuación.

> [!TIP]
> Incluso durante la creación de prototipos, se recomienda anclar cada versión del paquete en `requirements.txt`.
> Por ejemplo, use `scipy == 1.2.3` en lugar de solo `scipy` o incluso `scipy > 1.2.3`.
> Si no ancla una versión exacta y `scipy` publica una nueva versión, esto podría interrumpir el script de puntuación y provocar errores durante la implementación y el escalado.

En el siguiente ejemplo de código se muestra cómo se establece la variable de entorno `AZUREML_EXTRA_REQUIRMENTS_TXT`:

```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies 

myenv = Environment(name="my_azureml_env")
myenv.docker.enabled = True
myenv.docker.base_image = <MCR-path>
myenv.python.user_managed_dependencies = True

myenv.environment_variables = {
    "AZUREML_EXTRA_REQUIREMENTS_TXT": "requirements.txt"
}
```

El diagrama siguiente es una representación visual del proceso de instalación dinámica:

:::image type="content" source="./media/how-to-prebuilt-docker-images-inference-python-extensibility/dynamic-install-python-extend.svg" alt-text="Diagrama del proceso de instalación dinámica":::

<a id="preinstalled"></a>

## <a name="pre-installed-python-packages"></a>Paquetes preinstalados de Python

Este enfoque monta el directorio que se proporciona en la imagen. A continuación, el script de entrada (`score.py`) puede usar los paquetes de Python de este directorio.

Para ampliar la imagen de contenedor de Docker precompilada a través de paquetes de Python preinstalados, siga estos pasos:

> [!IMPORTANT]
> Debe usar paquetes compatibles con Python 3.7. Todas las imágenes actuales están ancladas a Python 3.7.

1. Cree un entorno virtual mediante [virtualenv](https://virtualenv.pypa.io/).

1. Instale las dependencias. Si tiene una lista de dependencias en un archivo `requirements.txt`, por ejemplo, puede usarla para instalar cada una de las dependencias con `pip install -r requirements.txt` o simplemente con `pip install`.

1. Al especificar la variable de entorno `AZUREML_EXTRA_PYTHON_LIB_PATH`, asegúrese de que apunta al directorio de paquetes del sitio correcto, el cual variará según el nombre del entorno y la versión de Python. En el código siguiente se muestra cómo establecer la ruta de acceso para un entorno virtual denominado `myenv` y Python 3.7:


    ```python
    from azureml.core import Environment
    from azureml.core.conda_dependencies import CondaDependencies 

    myenv = Environment(name='my_azureml_env')
    myenv.docker.enabled = True
    myenv.docker.base_image = <MCR-path>
    myenv.python.user_managed_dependencies = True

    myenv.environment_variables = {
        "AZUREML_EXTRA_PYTHON_LIB_PATH": "myenv/lib/python3.7/site-packages"
    }
    ```

El diagrama siguiente es una representación visual del proceso de paquetes preinstalados:

:::image type="content" source="./media/how-to-prebuilt-docker-images-inference-python-extensibility/pre-install-python-extend.svg" alt-text="Diagrama del proceso mediante paquetes preinstalados":::

### <a name="common-problems"></a>Problemas comunes

La solución de montaje solo funcionará cuando el directorio de paquetes del sitio `myenv` contenga todas las dependencias. Si el entorno local usa dependencias instaladas en una ubicación diferente, no estarán disponibles en la imagen.

Estos son algunos de los aspectos que pueden provocar este problema:

* `virtualenv` crea un entorno aislado de forma predeterminada. Una vez que se activa el entorno virtual, __no se pueden usar dependencias globales__.
* Si tiene una variable de entorno `PYTHONPATH` que apunta a las dependencias globales, __puede interferir con el entorno virtual__. Ejecute `pip list` y `pip freeze` después de activar el entorno para asegurarse de que no haya dependencias no deseadas en el entorno.
* __Los entornos de Conda y los entornos `virtualenv` pueden interferir__. Asegúrese de no usar el [entorno de Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) y `virtualenv` al mismo tiempo.

## <a name="limitations"></a>Limitaciones

### <a name="modelpackage"></a>Model.package()

* El método [Model.package()](/python/api/azureml-core/azureml.core.model(class)) permite crear un paquete de modelos como una imagen de Docker o un contexto de compilación de Dockerfile. El uso de Model.package() con imágenes de Docker de inferencia precompiladas desencadena una compilación de imagen intermedia que cambia el usuario no raíz por el usuario raíz.

* Le recomendamos que use nuestras soluciones de extensibilidad de paquetes de Python. Si se requieren otras dependencias (como paquetes `apt`), cree su propio archivo [Dockerfile mediante su extensión a partir de la imagen de inferencia](how-to-extend-prebuilt-docker-image-inference.md#buildmodel).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* En el enfoque de extensibilidad de requirements.txt, ¿es obligatorio que el nombre de archivo sea `requirements.txt`?

        
    ```python
    myenv.environment_variables = {
        "AZUREML_EXTRA_REQUIREMENTS_TXT": "name of your pip requirements file goes here"
    }
    ```

* ¿Puede resumir las características del enfoque con `requirements.txt` en comparación con el *enfoque de montaje*?

    Comience la creación de prototipos mediante el enfoque con *requirements.txt*.
    Después de algunas iteraciones, cuando esté seguro de qué paquetes (y versiones) necesita para una implementación correcta del modelo, cambie a la *solución de montaje*.
        
    Aquí se muestra una comparación detallada.

    | Elemento comparado | Requirements.txt (instalación dinámica) | Montaje de paquetes |
    | ----- | ----- | ------ |
    | Solución  | Cree un archivo `requirements.txt` que instale los paquetes especificados cuando se inicie el contenedor. | Cree un entorno de Python local con todas las dependencias. Monte este directorio en el contenedor en tiempo de ejecución. |
    | Instalación del paquete           | Sin instalación adicional (suponiendo que pip ya esté instalado)                                                                                                          | Instalación de entorno virtual o entorno de Conda.                                                                                   |
    | Configuración del entorno virtual              | No se requiere ninguna configuración adicional del entorno virtual, ya que los usuarios pueden extraer el entorno de usuario local actual con pip freeze según sea necesario para crear `requirements.txt`. | Es necesario para configurar un entorno virtual limpio, puede realizar pasos adicionales en función del entorno local del usuario actual.                        |
    | [Depuración](how-to-inference-server-http.md)                 | Servidor fácil de configurar y depurar, ya que las dependencias se muestran claramente. | Un entorno virtual no limpio podría causar problemas al depurar el servidor. Por ejemplo, puede que no esté claro si los errores proceden del entorno o del código de usuario. |
    | Coherencia durante el escalado horizontal | No es coherente, ya que depende de los paquetes PyPi externos y de los usuarios que anclan sus dependencias. Estas descargas externas podrían ser no confiables.                                 | Se basa únicamente en el entorno de usuario, por lo que no hay problemas de coherencia.                                                                             |

* ¿Por qué mi archivo `requirements.txt` y el directorio de dependencias montadas no se encuentran en el contenedor?

    Localmente, compruebe que las variables de entorno están establecidas correctamente. A continuación, compruebe que las rutas de acceso especificadas estén escritas correctamente y que existen.
    Compruebe si ha establecido correctamente el directorio de origen en el constructor de [configuración de inferencias](/python/api/azureml-core/azureml.core.model.inferenceconfig#constructor).

* ¿Puedo invalidar las dependencias de paquetes de Python en la imagen de Docker de la inferencia precompilada?

    Sí. Si desea usar otra versión del paquete de Python que ya está instalada en una imagen de inferencia, nuestra solución de extensibilidad respetará su versión. Asegúrese de que no haya ningún conflicto entre las dos versiones.

## <a name="best-practices"></a>Prácticas recomendadas

* Consulte la documentación sobre la [carga de modelos registrados](how-to-deploy-advanced-entry-script.md#load-registered-models). Al registrar un directorio de modelos, no incluya el script de puntuación, el directorio de dependencias montadas ni el archivo `requirements.txt` dentro de ese directorio.


* Para más información sobre cómo cargar un modelo registrado o local, consulte [Dónde y cómo implementar](how-to-deploy-and-where.md?tabs=azcli#define-a-dummy-entry-script).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la implementación de un modelo, consulte [Implementación de un modelo](how-to-deploy-and-where.md).

Para obtener información sobre cómo solucionar problemas de implementaciones de imágenes de Docker precompiladas, consulte [Solución de problemas de implementaciones de imágenes de Docker precompiladas](how-to-troubleshoot-prebuilt-docker-image-inference.md).