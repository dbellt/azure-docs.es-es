---
title: Servidor HTTP de inferencia de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Aprenda a habilitar el desarrollo local con el servidor HTTP de inferencia de Azure Machine Learning.
author: shivanissambare
ms.author: ssambare
ms.reviewer: larryfr
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: inference server, local development, local debugging
ms.date: 05/14/2021
ms.openlocfilehash: d54195829c4f4734d135e3468897711bdaa0421f
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110538455"
---
# <a name="azure-machine-learning-inference-http-server-preview"></a>Servidor HTTP de inferencia de Azure Machine Learning (versión preliminar)

El servidor HTTP de inferencia de Azure Machine Learning [(versión preliminar)](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) es un paquete de Python que permite validar fácilmente el script de entrada (`score.py`) en un entorno de desarrollo local. Si hay un problema con el script de puntuación, el servidor devolverá un error. También devolverá la ubicación en la que se produjo el error.

El servidor también se puede usar al crear puertas de validación en una canalización de integración e implementación continuas. Puede, por ejemplo, iniciar el servidor con el script candidato y ejecutar el conjunto de pruebas en el punto de conexión local.

## <a name="prerequisites"></a>Prerrequisitos

- Versión de Python 3.7

## <a name="installation"></a>Instalación

> [!NOTE]
> Para evitar conflictos con los paquetes, instale el servidor en un entorno virtual.

Para instalar `azureml-inference-server-http package`, ejecute el siguiente comando en cmd/terminal:

```bash
python -m pip install azureml-inference-server-http
```

## <a name="use-the-server"></a>Uso del servidor

1. Cree un directorio para almacenar los archivos:

    ```bash
    mkdir server_quickstart
    cd server_quickstart
    ```

1. Para evitar conflictos con los paquetes, cree un entorno virtual y actívelo:

    ```bash
    virtualenv myenv
    source myenv/bin/activate
    ```

1. Instale el paquete `azureml-inference-server-http` desde la fuente [pypi](https://pypi.org/):

    ```bash
    python -m pip install azureml-inference-server-http
    ```

1. Creación del script de entrada (`score.py`). En el ejemplo siguiente se crea un script de entrada básico:

    ```bash
    echo '
    import time

    def init():
        time.sleep(1)

    def run(input_data):
        return {"message":"Hello, World!"}
    ' > score.py
    ```

1. Inicie el servidor y establezca `score.py` como script de entrada:

    ```bash
    azmlinfsrv --entry_script score.py
    ```

    > [!NOTE]
    > El servidor se hospeda en 0.0.0.0, lo que significa que escuchará todas las direcciones IP del equipo host.

    El servidor escucha en el puerto 5001 en estas rutas.

    | Nombre | Ruta|
    | --- | --- |
    | Sondeo de ejecución | 127.0.0.1:5001/|
    | Puntuación | 127.0.0.1:5001/score|

1. Envíe una solicitud de puntuación al servidor mediante `curl`:

    ```bash
    curl -p 127.0.0.1:5001/score
    ```

    El servidor debe responder de esta manera.

    ```bash
    {"message": "Hello, World!"}
    ```

Ahora podrá modificar el script de puntuación y probar los cambios mediante la ejecución del servidor de nuevo.

## <a name="server-parameters"></a>Parámetros del servidor

La tabla siguiente contiene los parámetros que acepta el servidor:

| Parámetro | Obligatorio | Valor predeterminado | Descripción |
| ---- | --- | ---- | ----|
| entry_script | True | N/D | Ruta de acceso relativa o absoluta al script de puntuación.|
| model_dir | Falso | N/D | Ruta de acceso relativa o absoluta al directorio que contiene el modelo utilizado para la inferencia.
| port | False | 5001 | Puerto de servicio del servidor.|
| worker_count | False | 1 | Número de subprocesos de trabajo que procesarán solicitudes simultáneas. |

## <a name="request-flow"></a>Flujo de las solicitudes

En los pasos siguientes se explica cómo el servidor HTTP de inferencia de Azure Machine Learning controla las solicitudes entrantes:

1. Alrededor de la pila de red del servidor, un contenedor de la CLI de Python inicia el servidor.
1. Un cliente envía una solicitud al servidor.
1. Cuando se recibe una solicitud, pasa por el servidor [WSGI](https://www.fullstackpython.com/wsgi-servers.html) y se envía a uno de los trabajadores.
    - En __Linux__ se usa [Gunicorn](https://docs.gunicorn.org/).
    - En __Windows__ se usa [Waitress](https://docs.pylonsproject.org/projects/waitress/).
1. Las solicitudes se controlan a continuación mediante una aplicación de [Flask](https://flask.palletsprojects.com/), que carga el script de entrada y las dependencias.
1. Por último, la solicitud se envía al script de entrada. A continuación, el script de entrada realiza una llamada de inferencia al modelo cargado y devuelve una respuesta.

:::image type="content" source="./media/how-to-inference-server-http/inference-server-architecture.png" alt-text="Diagrama del proceso del servidor HTTP":::
## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="do-i-need-to-reload-the-server-when-changing-the-score-script"></a>¿Es necesario volver a cargar el servidor al cambiar el script de puntuación?

Después de cambiar el script de puntuación (`score.py`), detenga el servidor con `ctrl + c`. A continuación, reinícielo con `azmlinfsrv --entry_script score.py`.

### <a name="which-os-is-supported"></a>¿Qué sistema operativo se admite?

El servidor de inferencia de Azure Machine Learning se ejecuta en Windows y en los sistemas operativos basados en Linux.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo crear un script de entrada e implementar modelos, consulte [Implementación de un modelo mediante Azure Machine Learning](how-to-deploy-and-where.md).