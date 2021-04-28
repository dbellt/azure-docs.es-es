---
title: Solución de problemas de experimentos de aprendizaje automático automatizado
titleSuffix: Azure Machine Learning
description: Aprenda a solucionar los problemas de los experimentos de aprendizaje automático automatizado.
author: nibaccam
ms.author: nibaccam
ms.reviewer: nibaccam
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 03/08/2021
ms.topic: troubleshooting
ms.custom: devx-track-python, automl, references_regions
ms.openlocfilehash: 6f64d3511da8fbf953a50eab44702c7356ad3c5b
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108140486"
---
# <a name="troubleshoot-automated-ml-experiments-in-python"></a>Solución de problemas de experimentos de aprendizaje automático automatizado en Python

En esta guía, aprenderá a identificar y resolver problemas conocidos de los experimentos de aprendizaje automático automatizado con el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/intro).

## <a name="version-dependencies"></a>Dependencias de versiones

Las dependencias de **`AutoML` con respecto a versiones más recientes del paquete rompen la compatibilidad**. Después de la versión 1.13.0 del SDK, los modelos no se cargan en los SDK anteriores debido a la incompatibilidad entre las versiones anteriores ancladas en los paquetes de `AutoML` anteriores, y las versiones más recientes ancladas hoy en día.

Se esperan errores como:

* Errores de módulo no encontrado como,

  `No module named 'sklearn.decomposition._truncated_svd`

* Errores de importación como,

  `ImportError: cannot import name 'RollingOriginValidator'`,
* Errores de atributos como,

  `AttributeError: 'SimpleImputer' object has no attribute 'add_indicator`

Las resoluciones dependen de la versión de entrenamiento del SDK de `AutoML`:

* Si la versión de entrenamiento del SDK de `AutoML` es posterior a la 1.13.0, se necesitan `pandas == 0.25.1` y `scikit-learn==0.22.1`.

    * Si hay una discrepancia de versiones, actualice scikit-learn o Pandas a la versión correcta con lo siguiente:

      ```bash
          pip install --upgrade pandas==0.25.1
          pip install --upgrade scikit-learn==0.22.1
      ```

* Si la versión de entrenamiento del SDK de `AutoML` es la 1.12.0, o anterior, se necesitan `pandas == 0.23.4` y `sckit-learn==0.20.3`.
  * Si hay una discrepancia de versiones, cambie scikit-learn o Pandas a una versión anterior correcta con lo siguiente:
  
    ```bash
      pip install --upgrade pandas==0.23.4
      pip install --upgrade scikit-learn==0.20.3
    ```

## <a name="setup"></a>Configurar

El paquete de `AutoML` cambia, ya que la versión 1.0.76 requiere desinstalar la versión anterior antes de actualizar a la nueva.

* **`ImportError: cannot import name AutoMLConfig`**

    Si se detecta este error después de actualizar a la versión v1.0.76, o posterior, del SDK desde una versión anterior, resuelva el error ejecutando `pip uninstall azureml-train automl` y luego `pip install azureml-train-automl`. El script automl_setup.cmd lo hace automáticamente.

* **Error de automl_setup**

  * En Windows, ejecute automl_setup desde un símbolo del sistema de Anaconda. [Instalación de Miniconda](https://docs.conda.io/en/latest/miniconda.html).

  * Asegúrese de está instalado la versión de 64 bits de Conda 4.4.10, o una versión posterior. Puede comprobar los bits con el comando `conda info`. `platform` debe ser `win-64` para Windows o `osx-64` para Mac. Para comprobar la versión, use el comando `conda -V`. Si tiene instalada una versión anterior, puede actualizarla mediante el comando `conda update conda`. Para comprobar la versión de 32 bits, ejecute 

  * Asegúrese de que Conda esté instalado. 

  * Linux: `gcc: error trying to exec 'cc1plus'`

    1. Si se produce el error `gcc: error trying to exec 'cc1plus': execvp: No such file or directory`, instale las herramientas de creación de GCC para su distribución de Linux. Por ejemplo, en Ubuntu, use el comando `sudo apt-get install build-essential`.

    1. Pase un nuevo nombre como primer parámetro a automl_setup para crear un nuevo entorno de Conda. Vea los entornos de Conda existentes mediante `conda env list` y quítelos con `conda env remove -n <environmentname>`.

* **Error de automl_setup_linux.sh**: si automl_setup_linus.sh produce el error `unable to execute 'gcc': No such file or directory` en Ubuntu Linux:

  1. Asegúrese de que los puertos de salida 53 y 80 estén habilitados. En una máquina virtual de Azure, puede hacerlo desde Azure Portal. Para ello, seleccione la máquina virtual y haga clic en **Redes**.
  1. Ejecute el comando: `sudo apt-get update`
  1. Ejecute el comando: `sudo apt-get install build-essential --fix-missing`
  1. Vuelva a ejecutar `automl_setup_linux.sh`.

* **Error de configuration.ipynb**:

  * En el caso de Conda local, asegúrese primero de que `automl_setup` se haya ejecutado correctamente.
  * Asegúrese de que subscription_id es correcto. Busque subscription_id en Azure Portal. Para ello, seleccione Todos los servicios y, después, Suscripciones. Los caracteres "<" y ">" no deben incluirse en el valor de subscription_id. Por ejemplo, `subscription_id = "12345678-90ab-1234-5678-1234567890abcd"` tiene el formato válido.
  * Garantice el acceso de tipo Colaborador o Propietario a la suscripción.
  * Compruebe que la región sea una de las regiones admitidas: `eastus2`, `eastus`, `westcentralus`, `southeastasia`, `westeurope`, `australiaeast`, `westus2` o `southcentralus`.
  * Asegúrese de acceder a la región mediante Azure Portal.
  
* **Error de workspace.from_config**:

  Si se produce un error en la llamada a `ws = Workspace.from_config()`:

  1. Asegúrese de que el cuaderno configuration.ipynb se ha ejecutado correctamente.
  1. Si el cuaderno se ejecuta desde una carpeta que no está en la carpeta donde se ejecutó `configuration.ipynb`, copie la carpeta aml_config y el archivo config.json que contiene en la nueva carpeta. Workspace.from_config lee el archivo config.json de la carpeta del cuaderno o su carpeta principal.
  1. Si se usa una suscripción, un grupo de recursos, un área de trabajo o una región nuevos, asegúrese de volver a ejecutar el cuaderno `configuration.ipynb`. Cambiar el archivo config.json directamente solo funcionará si el área de trabajo ya existe en el grupo de recursos especificado en la suscripción indicada.
  1. Si quiere cambiar la región, cambie el área de trabajo, el grupo de recursos o la suscripción. `Workspace.create` no creará ni actualizará un área de trabajo si ya existe, aunque la región especificada sea diferente.

## <a name="tensorflow"></a>TensorFlow

A partir de la versión 1.5.0 del SDK, el aprendizaje automático automatizado no instala los modelos de TensorFlow de forma predeterminada. Para instalar TensorFlow y usarlo con los experimentos de aprendizaje automático automatizado, instale `tensorflow==1.12.0` mediante `CondaDependencies`.

```python
  from azureml.core.runconfig import RunConfiguration
  from azureml.core.conda_dependencies import CondaDependencies
  run_config = RunConfiguration()
  run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['tensorflow==1.12.0'])
```

## <a name="numpy-failures"></a>Errores de Numpy

* **Error de `import numpy` en Windows**: algunos entornos Windows ven un error al cargar Numpy con la versión más reciente de Python: 3.6.8. Si ve este problema, pruebe con la versión 3.6.7 de Python.

* **Error de `import numpy`** : compruebe la versión de TensorFlow en el entorno de Conda de aprendizaje automático automatizado. Las versiones compatibles son < 1.13. Desinstale TensorFlow del entorno si la versión es >= 1.13.

Puede comprobar la versión de TensorFlow y desinstalar como se indica a continuación:

  1. Inicie un shell de comandos y active el entorno de Conda donde están instalados los paquetes de aprendizaje automático automatizado.
  1. Escriba `pip freeze` y busque `tensorflow`. Si se encuentra, la versión indicada debe ser < 1,13.
  1. Si la versión indicada no es una versión compatible, escriba `pip uninstall tensorflow` en el shell de comandos y escriba "y" (sí) para confirmar la operación.

## `jwt.exceptions.DecodeError`

mensaje de error exacto: `jwt.exceptions.DecodeError: It is required that you pass in a value for the "algorithms" argument when calling decode()`.

En el caso de las versiones del SDK anteriores o iguales a la versión 1.17.0, la instalación podría dar como resultado una versión no compatible de PyJWT. Compruebe que la versión de PyJWT en el entorno de Conda de aprendizaje automático automatizado es una versión admitida. Es decir, PyJWT versión 2.0.0 o anterior.

Puede comprobar la versión de PyJWT como se indica a continuación:

1. Inicie un shell de comandos y active el entorno de Conda donde están instalados los paquetes de aprendizaje automático automatizado.

1. Escriba `pip freeze` y busque `PyJWT`; si se encuentra, la versión indicada debe ser < 2.0.0.

Si la versión indicada no es una versión compatible:

1. Considere la posibilidad de actualizar a la versión más reciente del SDK de AutoML: `pip install -U azureml-sdk[automl]`.

1. Si eso no es viable, desinstale PyJWT del entorno e instale la versión correcta como se indica a continuación:

    1. Ejecute `pip uninstall PyJWT` en el shell de comandos y escriba `y` para confirmar.
    1. Realice la instalación mediante `pip install 'PyJWT<2.0.0'`.
  
## <a name="databricks"></a>Databricks
Consulte [Configuración de un experimento de aprendizaje automático automatizado con Databricks](how-to-configure-databricks-automl-environment.md#troubleshooting).

## <a name="forecasting-r2-score-is-always-zero"></a>La puntuación R2 de la previsión siempre es cero

 este problema surge si los datos de entrenamiento proporcionados tienen una serie temporal que contiene el mismo valor para los últimos puntos de datos `n_cv_splits` + `forecasting_horizon`.

Si se espera este patrón en la serie temporal, puede cambiar la métrica principal a la **raíz del error cuadrático medio normalizado**.

## <a name="failed-deployment"></a>Implementación con errores

 En el caso de la versión 1.18.0 u otras anteriores del SDK, puede producirse el siguiente error en la imagen base creada para la implementación: `ImportError: cannot import name cached_property from werkzeug`.

  Para solucionar el problema, haga lo siguiente:

  1. Descargue los archivos del modelo.
  1. Descompresión del paquete
  1. Implementación mediante los recursos sin comprimir

## <a name="azure-functions-application"></a>Aplicación de Azure Functions
  
  En la actualidad, el aprendizaje automático automatizado no admite aplicaciones de Azure Functions. 

## <a name="sample-notebook-failures"></a>Errores del cuaderno de ejemplo

 si se produce un error en un cuaderno de ejemplo que indica que la propiedad, el método o la biblioteca no existen:

* Asegúrese de que se ha seleccionado el kernel correcto en el cuaderno de Jupyter Notebook. El kernel se muestra en la parte superior derecha de la página del cuaderno. El valor predeterminado es *azure_automl*. El kernel se guarda como parte del cuaderno. Si cambia a un nuevo entorno de Conda, tendrá que seleccionar el nuevo kernel en el cuaderno.

  * Para Azure Notebooks, debe ser Python 3.6.
  * En el caso de entornos de Conda locales, debe ser el nombre del entorno de Conda que especificó en automl_setup.

* Para asegurarse de que el cuaderno sea para la versión del SDK que está usando:
  * Para comprobar la versión del SDK, ejecute `azureml.core.VERSION` en una celda de Jupyter Notebook.
  * Puede descargar la versión anterior de los cuadernos de ejemplo de GitHub con estos pasos:
    1. Seleccione el botón `Branch`.
    1. Vaya a la pestaña `Tags`.
    1. Selección de la versión

## <a name="next-steps"></a>Pasos siguientes

+ Obtenga más información sobre [cómo entrenar un modelo de regresión con aprendizaje automático automatizado](tutorial-auto-train-models.md) o [cómo entrenar con aprendizaje automático automatizado en un recurso remoto](concept-automated-ml.md#local-remote).

+ Obtenga más información sobre [cómo y dónde implementar un modelo](how-to-deploy-and-where.md).
