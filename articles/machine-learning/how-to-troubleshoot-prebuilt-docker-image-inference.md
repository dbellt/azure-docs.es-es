---
title: Solución de problemas de imágenes de Docker precompiladas
titleSuffix: Azure Machine Learning
description: Pasos de solución de problemas para usar imágenes de Docker precompiladas para la inferencia.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: ssambare
author: shivanissambare
ms.date: 05/25/2021
ms.topic: how-to
ms.reviewer: larryfr
ms.custom: deploy, docker, prebuilt, troubleshoot
ms.openlocfilehash: 598c578c445cbd7dc7086f22e3c8d4885d67b112
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537337"
---
# <a name="troubleshooting-prebuilt-docker-images-for-inference-preview"></a>Solución de problemas de imágenes de Docker precompiladas para la inferencia (versión preliminar)

Aprenda a solucionar problemas que puede experimentar al usar imágenes de Docker precompiladas para la inferencia [(versión preliminar)](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) con Azure Machine Learning.

## <a name="model-deployment-failed"></a>Error de implementación del modelo

Si se produce un error en la implementación del modelo, no verá registros en [Azure Machine Learning Studio](https://ml.azure.com/) y `service.get_logs()` devolverá None (Ninguno).
Si hay un problema en la función init() de score.py, `service.get_logs()` devolverá registros por lo mismo.

Por lo tanto, deberá ejecutar el contenedor localmente mediante uno de los comandos que se muestran a continuación y reemplazar `<MCR-path>` por una ruta de acceso de imagen. Para obtener una lista de las imágenes y las rutas de acceso, consulte [Imágenes de Docker precompiladas para inferencia](concept-prebuilt-docker-images-inference.md).

### <a name="mounting-extensibility-solution"></a>Montaje de la solución de extensibilidad

Vaya al directorio que contiene `score.py` y ejecute:

```bash
docker run -it -v $(pwd):/var/azureml-app -e AZUREML_EXTRA_PYTHON_LIB_PATH="myenv/lib/python3.7/site-packages" <mcr-path>
```

### <a name="requirementstxt-extensibility-solution"></a>Solución de extensibilidad requirements.txt

Vaya al directorio que contiene `score.py` y ejecute:

```bash
docker run -it -v $(pwd):/var/azureml-app -e AZUREML_EXTRA_REQUIREMENTS_TXT="requirements.txt" <mcr-path>
```

## <a name="enable-local-debugging"></a>Habilitación de la depuración local

El servidor de inferencia local permite depurar rápidamente el script de entrada (`score.py`). En caso de que el script de puntuación subyacente tenga un error, el servidor no podrá inicializar ni atender el modelo. Se producirá una excepción con la ubicación donde se produjeron los problemas. [Más información sobre el servidor HTTP de inferencia de Azure Machine Learning](how-to-inference-server-http.md)

## <a name="for-common-model-deployment-issues"></a>Para problemas comunes de implementación del modelo

Si tiene problemas al implementar un modelo de Azure Machine Learning en Azure Container Instances (ACI) o Azure Kubernetes Service (AKS), consulte [Solución de problemas de implementación de modelos](how-to-troubleshoot-deployment.md).

## <a name="init-or-run-failing-to-write-a-file"></a>Error de init() o run() al escribir un archivo

El servidor HTTP de nuestras imágenes de Docker precompiladas se ejecuta como *usuario no raíz*, así que es posible que no tenga acceso a todos los directorios. Escriba solo en los directorios a los que tenga derechos de acceso. Por ejemplo, el directorio `/tmp` del contenedor.

## <a name="extra-python-packages-not-installed"></a>Paquetes adicionales de Python no instalados

* Compruebe si hay un error tipográfico en la variable de entorno o el nombre de archivo.
* Compruebe el registro de contenedor para ver si `pip install -r <your_requirements.txt>` está o no instalado.
* Compruebe si el directorio de origen está establecido correctamente en el constructor de [configuración de inferencia](/python/api/azureml-core/azureml.core.model.inferenceconfig#constructor).
* Si no se encuentra la instalación y el registro indica "archivo no encontrado", compruebe si el nombre de archivo que se muestra en el registro es correcto.
* Si la instalación se inició pero no se realizó o se agotó el tiempo de espera, intente instalar el mismo archivo `requirements.txt` localmente con la misma versión de Python y PIP en un entorno limpio (es decir, sin directorio de caché: `pip install --no-cache-dir -r requriements.txt`). Vea si el problema se puede reproducir localmente.

## <a name="mounting-solution-failed"></a>Error al montar la solución

* Compruebe si hay un error tipográfico en la variable de entorno o el nombre del directorio.
* La variable de entorno debe establecerse en la ruta de acceso relativa del archivo `score.py`.
* Compruebe si el directorio de origen está establecido correctamente en el constructor de [configuración de inferencia](/python/api/azureml-core/azureml.core.model.inferenceconfig#constructor).
* El directorio debe ser el directorio "site-packages" del entorno.
* Si `score.py` todavía devuelve `ModuleNotFound` y se supone que el módulo está en el directorio montado, intente imprimir `sys.path` en `init()` o `run()` para ver si falta alguna ruta de acceso.

## <a name="building-an-image-based-on-the-prebuilt-docker-image-failed"></a>Error al compilar una imagen basada en la imagen de Docker precompilada

* Si se ha registrado un error durante la instalación del paquete apt, compruebe si el usuario se ha establecido como raíz antes de ejecutar el comando apt. (Asegúrese de volver a cambiar al usuario no raíz). 

## <a name="image-built-based-on-the-prebuilt-docker-image-cant-boot-up"></a>La imagen compilada a partir de la imagen de Docker precompilada no puede iniciarse.

* El usuario no raíz debe ser `dockeruser`. De lo contrario, el propietario de los directorios siguientes debe establecerse en el nombre de usuario que quiera usar al ejecutar la imagen:

    ```bash
    /var/runit
    /var/log
    /var/lib/nginx
    /run
    /opt/miniconda
    /var/azureml-app
    ```

* Si se ha cambiado `ENTRYPOINT` en la nueva imagen compilada, el servidor HTTP y los componentes relacionados deben cargarse mediante `runsvdir /var/runit`.

## <a name="next-steps"></a>Pasos siguientes

* [Agregue paquetes de Python a imágenes precompiladas](how-to-prebuilt-docker-images-inference-python-extensibility.md).
* [Use un paquete precompilado como base para un nuevo Dockerfile](how-to-extend-prebuilt-docker-image-inference.md).