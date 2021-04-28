---
title: Depuración interactiva con Visual Studio Code
titleSuffix: Azure Machine Learning
description: Depure de manera interactiva el código, las canalizaciones y las implementaciones de Azure Machine Learning mediante Visual Studio Code.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: luisquintanilla
ms.author: luquinta
ms.date: 09/30/2020
ms.openlocfilehash: d3c4438cdfa8ddd41cb306e84623ccd569ac1680
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107885585"
---
# <a name="interactive-debugging-with-visual-studio-code"></a>Depuración interactiva con Visual Studio Code



Aprenda a depurar de manera interactiva experimentos, canalizaciones e implementaciones de Azure Machine Learning con Visual Studio Code (VS Code) y [depugpy](https://github.com/microsoft/debugpy/).

## <a name="run-and-debug-experiments-locally"></a>Ejecución y depuración de experimentos de manera local

Use la extensión de Azure Machine Learning para validar, ejecutar y depurar los experimentos de aprendizaje automático antes de enviarlos a la nube.

### <a name="prerequisites"></a>Requisitos previos

* Extensión de Azure Machine Learning para VS Code (versión preliminar) Para más información, consulte [Configuración de la extensión de Azure Machine Learning para VS Code](tutorial-setup-vscode-extension.md).
* [Docker](https://www.docker.com/get-started)
  * Docker Desktop para Mac y Windows
  * Motor de Docker para Linux.
* [Python 3](https://www.python.org/downloads/)

> [!NOTE]
> En Windows, asegúrese de [configurar Docker para usar contenedores de Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

> [!TIP]
> En Windows, aunque no es necesario, se recomienda encarecidamente [usar Docker con el subsistema de Windows para Linux (WSL) 2](/windows/wsl/tutorials/wsl-containers#install-docker-desktop).

> [!IMPORTANT]
> Antes de ejecutar el experimento de manera local, asegúrese de que Docker se está ejecutando.

### <a name="debug-experiment-locally"></a>Depuración del experimento de manera local

1. En VS Code, abra la vista de la extensión de Azure Machine Learning.
1. Expanda el nodo de suscripción que contiene el área de trabajo. Si aún no tiene una, puede [crear un área de trabajo de Azure Machine Learning](how-to-manage-resources-vscode.md#create-a-workspace) mediante la extensión.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Experiments** (Experimentos) y seleccione **Create experiment** (Crear experimento). Cuando se le pida, proporcione un nombre para el experimento.
1. Expanda el nodo **Experiments** (Experimentos), haga clic con el botón derecho en el experimento que quiera ejecutar y seleccione **Run Experiment** (Ejecutar experimento).
1. En la lista de opciones para ejecutar el experimento, seleccione **Locally** (Localmente).
1. **Uso por primera vez solo en Windows**. Cuando se le pregunte si desea permitir recursos compartidos de archivos, seleccione **Yes** (Sí). Cuando habilita los recursos compartidos de archivos, permite que Docker monte el directorio que contiene el script en el contenedor. Además, también permite que Docker almacene los registros y los resultados de la ejecución en un directorio temporal del sistema.
1. Seleccione **Yes** (Sí) para depurar el experimento. De lo contrario, seleccione **No**. Si selecciona No, el experimento se ejecutará de manera local sin asociarse al depurador.
1. Seleccione **Create new Run Configuration** (Crear configuración de ejecución) para crear la configuración de ejecución. La configuración de ejecución define el script que quiere ejecutar, las dependencias y los conjuntos de datos usados. Como alternativa, si ya tiene una, selecciónela en la lista desplegable.
    1. Elija el entorno. Puede elegir cualquiera de los [mantenidos por Azure Machine Learning](resource-curated-environments.md) o crear el suyo propio.
    1. Proporcione el nombre del script que quiere ejecutar. La ruta de acceso es relativa al directorio abierto en VS Code.
    1. Elija si quiere usar un conjunto de datos de Azure Machine Learning o no. Puede crear [conjuntos de datos de Azure Machine Learning](how-to-manage-resources-vscode.md#create-dataset) mediante la extensión.
    1. La herramienta debugpy es necesaria para asociar el depurador al contenedor que ejecuta el experimento. Para agregar debugpy como dependencia, seleccione **Add Debugpy** (Agregar debugpy). De lo contrario, seleccione **Skip** (Omitir). Si no se agrega debugpy como dependencia, el experimento se ejecuta sin asociarse al depurador.
    1. En el editor, se abre un archivo de configuración que contiene los valores de configuración de ejecución. Si está satisfecho con la configuración, seleccione **Submit experiment** (Enviar experimento). Como alternativa, abra la paleta de comandos (**View > Command Palette** [Ver > Paleta de comandos]) en la barra de menús y escriba el comando `Azure ML: Submit experiment` en el cuadro de texto.
1. Una vez enviado el experimento, se crea una imagen de Docker que contiene el script y las configuraciones especificadas en la configuración de ejecución.

    Cuando se inicia el proceso de compilación de la imagen de Docker, el contenido del archivo `60_control_log.txt` se transmite a la consola de salida en VS Code.

    > [!NOTE]
    > La primera vez que se crea la imagen de Docker, esta operación puede tardar varios minutos.

1. Después de que se ha creado la imagen, aparece un aviso para iniciar el depurador. Establezca los puntos de interrupción en el script y, cuando esté preparado para comenzar la depuración, seleccione **Iniciar depuración**. Al hacerlo, se asocia el depurador de VS Code al contenedor que ejecuta el experimento. Como alternativa, en la extensión de Azure Machine Learning, mantenga el mouse sobre el nodo de la ejecución actual y seleccione el icono de reproducción para iniciar el depurador.

    > [!IMPORTANT]
    > No puede tener varias sesiones de depuración para un solo experimento. Sin embargo, puede depurar dos o más experimentos mediante varias instancias de VS Code.

Llegados a este punto, debería poder recorrer el código y depurarlo mediante VS Code.

Si en algún momento quiere cancelar la ejecución, haga clic con el botón derecho en el nodo de ejecución y seleccione **Cancelar ejecución**.

De forma similar a las ejecuciones de experimentos remotas, puede expandir el nodo de ejecución para inspeccionar los registros y las salidas.

> [!TIP]
> Las imágenes de Docker que usan las mismas dependencias definidas en el entorno se reutilizan entre ejecuciones. Sin embargo, si ejecuta un experimento con un entorno nuevo o diferente, se crea otra imagen. Dado que estas imágenes se guardan en el almacenamiento local, se recomienda quitar las imágenes de Docker antiguas o sin usar. Para quitar imágenes del sistema, use la [CLI de Docker](https://docs.docker.com/engine/reference/commandline/rmi/) o la [extensión de Docker para VS Code](https://code.visualstudio.com/docs/containers/overview).

## <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Depuración y solución de problemas de canalizaciones de aprendizaje automático

En algunos casos, es posible que tenga que depurar interactivamente el código de Python usado en la canalización de ML. Mediante VS Code y debugpy, se puede conectar al código que se ejecuta en el entorno de entrenamiento.

### <a name="prerequisites"></a>Prerrequisitos

* Un __área de trabajo de Azure Machine Learning__ configurada para usar __Azure Virtual Network__.
* Una __canalización de Azure Machine Learning__ que use scripts de Python como parte de los pasos de la canalización. Por ejemplo, PythonScriptStep.
* Un clúster de Proceso de Azure Machine Learning, que se encuentre __en la red virtual__ y lo __use la canalización para el aprendizaje__.
* Un __entorno de desarrollo__ que se encuentre __en la red virtual__. El entorno de desarrollo puede ser uno de los siguientes:

  * Una máquina virtual de Azure en la red virtual
  * Una instancia de proceso de VM de cuadernos en la red virtual
  * Una máquina cliente que tenga conectividad de red privada con la red virtual, ya sea por VPN o a través de ExpressRoute.

Para más información sobre el uso de Azure Virtual Network con Azure Machine Learning, consulte [Información general sobre la privacidad y el aislamiento de la red virtual](how-to-network-security-overview.md).

> [!TIP]
> Aunque puede trabajar con recursos de Azure Machine Learning que no están detrás de una red virtual, se recomienda usar una red virtual.

### <a name="how-it-works"></a>Funcionamiento

Los pasos de la canalización de ML ejecutan scripts de Python. Estos scripts se modifican para realizar las siguientes acciones:

1. Registrar la dirección IP del host en el que se ejecutan. Se usa la dirección IP para conectar el depurador al script.

2. Iniciar el componente de depuración debugpy y esperar a que se conecte un depurador.

3. En el entorno de desarrollo, se supervisan los registros creados por el proceso de aprendizaje para encontrar la dirección IP en la que se ejecuta el script.

4. Se indica a VS Code la dirección IP a la que se debe conectar el depurador mediante un archivo `launch.json`.

5. Se conecta el depurador y se recorre el script de forma interactiva.

### <a name="configure-python-scripts"></a>Configuración de scripts de Python

Para habilitar la depuración, realice los cambios siguientes en los scripts de Python que se usan en los pasos de la canalización de ML:

1. Agregue las siguientes instrucciones import:

    ```python
    import argparse
    import os
    import debugpy
    import socket
    from azureml.core import Run
    ```

1. Agregue los siguientes argumentos. Estos argumentos permiten habilitar el depurador según sea necesario y establecer el tiempo de espera para la conexión del depurador:

    ```python
    parser.add_argument('--remote_debug', action='store_true')
    parser.add_argument('--remote_debug_connection_timeout', type=int,
                        default=300,
                        help=f'Defines how much time the AML compute target '
                        f'will await a connection from a debugger client (VSCODE).')
    parser.add_argument('--remote_debug_client_ip', type=str,
                        help=f'Defines IP Address of VS Code client')
    parser.add_argument('--remote_debug_port', type=int,
                        default=5678,
                        help=f'Defines Port of VS Code client')
    ```

1. Agregue las siguientes instrucciones. Estas instrucciones cargan el contexto de ejecución actual para que pueda registrar la dirección IP del nodo en el que se está ejecutando el código:

    ```python
    global run
    run = Run.get_context()
    ```

1. Agregue una instrucción `if` que inicie debugpy y espere a que se conecte un depurador. Si no se conecta ningún depurador antes de que finalice el tiempo de espera, el script continúa de la forma habitual. Asegúrese de reemplazar los valores `HOST` y `PORT` de la función `listen` por los propios.

    ```python
    if args.remote_debug:
        print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
        # Log the IP and port
        try:
            ip = args.remote_debug_client_ip
        except:
            print("Need to supply IP address for VS Code client")
        print(f'ip_address: {ip}')
        debugpy.listen(address=(ip, args.remote_debug_port))
        # Wait for the timeout for debugger to attach
        debugpy.wait_for_client()
        print(f'Debugger attached = {debugpy.is_client_connected()}')
    ```

En el siguiente ejemplo de Python se muestra un archivo `train.py` de ejemplo que habilita la depuración:

```python
# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license.

import argparse
import os
import debugpy
import socket
from azureml.core import Run

print("In train.py")
print("As a data scientist, this is where I use my training code.")

parser = argparse.ArgumentParser("train")

parser.add_argument("--input_data", type=str, help="input data")
parser.add_argument("--output_train", type=str, help="output_train directory")

# Argument check for remote debugging
parser.add_argument('--remote_debug', action='store_true')
parser.add_argument('--remote_debug_connection_timeout', type=int,
                    default=300,
                    help=f'Defines how much time the AML compute target '
                    f'will await a connection from a debugger client (VSCODE).')
parser.add_argument('--remote_debug_client_ip', type=str,
                    help=f'Defines IP Address of VS Code client')
parser.add_argument('--remote_debug_port', type=int,
                    default=5678,
                    help=f'Defines Port of VS Code client')

# Get run object, so we can find and log the IP of the host instance
global run
run = Run.get_context()

args = parser.parse_args()

# Start debugger if remote_debug is enabled
if args.remote_debug:
    print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
    # Log the IP and port
    # ip = socket.gethostbyname(socket.gethostname())
    try:
        ip = args.remote_debug_client_ip
    except:
        print("Need to supply IP address for VS Code client")
    print(f'ip_address: {ip}')
    debugpy.listen(address=(ip, args.remote_debug_port))
    # Wait for the timeout for debugger to attach
    debugpy.wait_for_client()
    print(f'Debugger attached = {debugpy.is_client_connected()}')

print("Argument 1: %s" % args.input_data)
print("Argument 2: %s" % args.output_train)

if not (args.output_train is None):
    os.makedirs(args.output_train, exist_ok=True)
    print("%s created" % args.output_train)
```

### <a name="configure-ml-pipeline"></a>Configuración de la canalización de ML

Para proporcionar los paquetes de Python necesarios para iniciar debugpy y obtener el contexto de ejecución, cree un entorno y establezca `pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>']`. Cambie la versión del SDK para que coincida con la que está usando. El fragmento de código siguiente muestra cómo crear un entorno:

```python
# Use a RunConfiguration to specify some additional requirements for this step.
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_CPU_IMAGE

# create a new runconfig object
run_config = RunConfiguration()

# enable Docker 
run_config.environment.docker.enabled = True

# set Docker base image to the default CPU-based image
run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE

# use conda_dependencies.yml to create a conda environment in the Docker image for execution
run_config.environment.python.user_managed_dependencies = False

# specify CondaDependencies obj
run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
                                                                           pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>'])
```

En la sección [Configuración de scripts de Python](#configure-python-scripts), se agregaron nuevos argumentos a los scripts usados por los pasos de la canalización de ML. En el fragmento de código siguiente se muestra cómo usar estos argumentos para habilitar la depuración del componente y establecer un tiempo de espera. También se muestra cómo usar el entorno creado anteriormente estableciendo `runconfig=run_config`:

```python
# Use RunConfig from a pipeline step
step1 = PythonScriptStep(name="train_step",
                         script_name="train.py",
                         arguments=['--remote_debug', '--remote_debug_connection_timeout', 300,'--remote_debug_client_ip','<VS-CODE-CLIENT-IP>','--remote_debug_port',5678],
                         compute_target=aml_compute,
                         source_directory=source_directory,
                         runconfig=run_config,
                         allow_reuse=False)
```

Cuando se ejecuta la canalización, cada paso crea una ejecución secundaria. Si está habilitada la depuración, el script modificado registra información similar al siguiente texto en `70_driver_log.txt` para la ejecución secundaria:

```text
Timeout for debug connection: 300
ip_address: 10.3.0.5
```

Guarde el valor de `ip_address`. Se usa en la siguiente sección.

> [!TIP]
> También puede encontrar la dirección IP en los registros de la ejecución secundaria para este paso de canalización. Para obtener más información sobre cómo ver esta información, vea [Supervisión de métricas y ejecuciones de experimentos de Azure ML](how-to-log-view-metrics.md).

### <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

1. Para instalar debugpy en el entorno de desarrollo de VS Code, use el comando siguiente:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Para obtener más información sobre cómo usar debugpy con VS Code, consulte [Depuración remota](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Para configurar VS Code para comunicarse con el proceso de Azure Machine Learning que ejecuta el depurador, cree una nueva configuración de depuración:

    1. En VS Code, seleccione el menú __Depurar__ y, luego, seleccione __Abrir configuraciones__. Se abre un archivo denominado __launch.json__.

    1. En el archivo __launch.json__, busque la línea que contiene `"configurations": [` e inserte el texto siguiente después de ella. Cambie la entrada `"host": "<IP-ADDRESS>"` a la dirección IP devuelta en los registros de la sección anterior. Cambie la entrada `"localRoot": "${workspaceFolder}/code/step"` a un directorio local que contenga una copia del script que se está depurando:

        ```json
        {
            "name": "Azure Machine Learning Compute: remote debug",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "<IP-ADDRESS>",
            "redirectOutput": true,
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}/code/step1",
                    "remoteRoot": "."
                }
            ]
        }
        ```

        > [!IMPORTANT]
        > Si ya hay otras entradas en la sección de configuraciones, agregue una coma (,) después del código que insertó.

        > [!TIP]
        > El procedimiento recomendado, especialmente en el caso de las canalizaciones, es mantener los recursos de los scripts en directorios independientes para que el código sea pertinente solo para cada uno de los pasos. En este ejemplo, el valor de ejemplo `localRoot` hace referencia a `/code/step1`.
        >
        > Si va a depurar varios scripts en directorios diferentes, cree una sección de configuración independiente para cada script.

    1. Guarde el archivo __launch.json__.

### <a name="connect-the-debugger"></a>Conexión del depurador

1. Abra VS Code y abra una copia local del script.
2. Establezca puntos de interrupción donde desee que se detenga el script una vez que se haya conectado.
3. Mientras el proceso secundario ejecuta el script y se muestra `Timeout for debug connection` en los registros, use la tecla F5 o seleccione __Depurar__. Cuando se le solicite, seleccione la configuración __Azure Machine Learning Compute: remote debug__ (Proceso de Azure Machine Learning: depuración remota). También puede seleccionar el icono de depuración en la barra lateral, la entrada __Azure Machine Learning: remote debug__ (Azure Machine Learning: depuración remota) en el menú desplegable Depurar y, a continuación, usar la flecha verde para conectar el depurador.

    En este punto, VS Code se conecta a debugpy en el nodo de ejecución y se detiene en el punto de interrupción que se estableció anteriormente. Ahora puede recorrer el código a medida que se ejecuta, ver variables, etc.

    > [!NOTE]
    > Si el registro muestra una entrada que indica `Debugger attached = False`, el tiempo de espera ha expirado y el script ha continuado sin el depurador. Vuelva a enviar la canalización y conecte el depurador después del mensaje de `Timeout for debug connection` y antes de que expire el tiempo de espera.

## <a name="debug-and-troubleshoot-deployments"></a>Depuración y solución de problemas de implementaciones

En algunos casos, es posible que tenga que depurar interactivamente el código de Python incluido en la implementación de modelo. Por ejemplo, si el script de entrada presenta errores y no se puede determinar el motivo mediante un registro adicional. Mediante VS Code y debugpy, puede conectarse al código que se ejecuta en el contenedor de Docker.

> [!IMPORTANT]
> Este método de depuración no funciona cuando se usa `Model.deploy()` y `LocalWebservice.deploy_configuration` para implementar un modelo de manera local. En su lugar, debe crear una imagen con el método [Model.package()](/python/api/azureml-core/azureml.core.model.model#package-workspace--models--inference-config-none--generate-dockerfile-false-).

Las implementaciones de servicios web locales requieren una instalación de Docker en funcionamiento en el sistema local. Para obtener más información sobre el uso de Docker, consulte la [Documentación de Docker](https://docs.docker.com/). Tenga en cuenta que, al trabajar con instancias de proceso, Docker ya está instalado.

### <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

1. Para instalar debugpy en el entorno de desarrollo de VS Code local, use el comando siguiente:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Para obtener más información sobre cómo usar debugpy con VS Code, consulte [Depuración remota](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Para configurar VS Code para comunicarse con la imagen de Docker, cree una configuración de depuración nueva:

    1. En VS Code, seleccione el menú __Depurar__ en la extensión __Ejecutar__ y, luego, seleccione __Abrir configuraciones__. Se abre un archivo denominado __launch.json__.

    1. En el archivo __launch.json__, busque el elemento __"configuraciones"__ (la línea que contiene `"configurations": [`) e inserte el texto siguiente después de ella. 

        ```json
        {
            "name": "Azure Machine Learning Deployment: Docker Debug",
            "type": "python",
            "request": "attach",
            "connect": {
                "port": 5678,
                "host": "0.0.0.0",
            },
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}",
                    "remoteRoot": "/var/azureml-app"
                }
            ]
        }
        ```
        Después de la inserción, el archivo __launch.json__ debe ser similar al siguiente:
        ```json
        {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Python: Current File",
                "type": "python",
                "request": "launch",
                "program": "${file}",
                "console": "integratedTerminal"
            },
            {
                "name": "Azure Machine Learning Deployment: Docker Debug",
                "type": "python",
                "request": "attach",
                "connect": {
                    "port": 5678,
                    "host": "0.0.0.0"
                    },
                "pathMappings": [
                    {
                        "localRoot": "${workspaceFolder}",
                        "remoteRoot": "/var/azureml-app"
                    }
                ]
            }
            ]
        }
        ```

        > [!IMPORTANT]
        > Si ya hay otras entradas en la sección de configuraciones, agregue una coma ( __,__ ) después del código que insertó.

        Esta sección se adjunta al contenedor de Docker mediante el puerto __5678__.

    1. Guarde el archivo __launch.json__.

### <a name="create-an-image-that-includes-debugpy"></a>Creación de una imagen que incluye debugpy

1. Modifique el entorno de conda para la implementación de manera que incluya debugpy. En el ejemplo siguiente se muestra cómo se agrega con el parámetro `pip_packages`:

    ```python
    from azureml.core.conda_dependencies import CondaDependencies 


    # Usually a good idea to choose specific version numbers
    # so training is made on same packages as scoring
    myenv = CondaDependencies.create(conda_packages=['numpy==1.15.4',
                                'scikit-learn==0.19.1', 'pandas==0.23.4'],
                                 pip_packages = ['azureml-defaults==1.0.83', 'debugpy'])

    with open("myenv.yml","w") as f:
        f.write(myenv.serialize_to_string())
    ```

1. Para iniciar debugpy y esperar una conexión cuando se inicia el servicio, agregue lo siguiente en la parte superior del archivo `score.py`:

    ```python
    import debugpy
    # Allows other computers to attach to debugpy on this IP address and port.
    debugpy.listen(('0.0.0.0', 5678))
    # Wait 30 seconds for a debugger to attach. If none attaches, the script continues as normal.
    debugpy.wait_for_client()
    print("Debugger attached...")
    ```

1. Cree una imagen basada en la definición de entorno y extráigala en el Registro local. 

    > [!NOTE]
    > En este ejemplo se da por supuesto que `ws` apunta al área de trabajo de Azure Machine Learning y que `model` es el modelo que se está implementando. El archivo `myenv.yml` contiene las dependencias de conda creadas en el paso 1.

    ```python
    from azureml.core.conda_dependencies import CondaDependencies
    from azureml.core.model import InferenceConfig
    from azureml.core.environment import Environment


    myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
    myenv.docker.base_image = None
    myenv.docker.base_dockerfile = "FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
    inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
    package = Model.package(ws, [model], inference_config)
    package.wait_for_creation(show_output=True)  # Or show_output=False to hide the Docker build logs.
    package.pull()
    ```

    Una vez que se haya creado y descargado la imagen (este proceso puede tardar más de 10 minutos, tenga paciencia), la ruta de acceso de la misma (incluye el repositorio, el nombre y la etiqueta, que en este caso también es su resumen) se mostrará finalmente en un mensaje similar al siguiente:

    ```text
    Status: Downloaded newer image for myregistry.azurecr.io/package@sha256:<image-digest>
    ```

1. Para facilitar el trabajo con la imagen localmente, puede usar el comando siguiente para agregar una etiqueta para esta imagen. Reemplace `myimagepath` en el siguiente comando por el valor de la ubicación del paso anterior.

    ```bash
    docker tag myimagepath debug:1
    ```

    Para el resto de los pasos, puede hacer referencia a la imagen local como `debug:1` en lugar del valor de la ruta de acceso completa a la imagen.

### <a name="debug-the-service"></a>Depuración del servicio

> [!TIP]
> Si establece un tiempo de expiración para la conexión de debugpy en el archivo `score.py`, debe conectar VS Code a la sesión de depuración antes de que se cumpla el tiempo de expiración. Inicie VS Code, abra la copia local de `score.py`, establezca un punto de interrupción y prepárelo antes de seguir los pasos de esta sección.
>
> Para más información sobre la depuración y el establecimiento de puntos de interrupción, consulte [Depuración](https://code.visualstudio.com/Docs/editor/debugging).

1. Para iniciar un contenedor de Docker con la imagen, use el comando siguiente:

    ```bash
    docker run -it --name debug -p 8000:5001 -p 5678:5678 -v <my_local_path_to_score.py>:/var/azureml-app/score.py debug:1 /bin/bash
    ```

    Esto asocia el archivo `score.py` localmente al del contenedor. Por lo tanto, los cambios realizados en el editor se verán reflejados automáticamente en el contenedor.

2. Para una mejor experiencia, puede entrar en el contenedor con una nueva interfaz de VS Code. Seleccione la extensión `Docker` en la barra lateral de VS Code, busque el contenedor local creado que, en esta documentación, se denomina `debug:1`. Haga clic con el botón derecho en este contenedor y seleccione `"Attach Visual Studio Code"`, se abrirá automáticamente una nueva interfaz de VS Code que mostrará el interior del contenedor creado.

    ![Interfaz del contenedor de VS Code](./media/how-to-troubleshoot-deployment/container-interface.png)

3. En el contenedor, ejecute el comando siguiente en el shell.

    ```bash
    runsvdir /var/runit
    ```
    A continuación, puede ver la siguiente salida en el shell dentro del contenedor:

    ![La salida de la consola de ejecución del contenedor](./media/how-to-troubleshoot-deployment/container-run.png)

4. Para adjuntar VS Code a debugpy dentro del contenedor, abra VS Code y use la tecla F5 o seleccione __Depurar__. Cuando se le solicite, seleccione la configuración __Azure Machine Learning Deployment: Docker Debug__ (Azure Machine Learning Service: depuración de Docker). También puede seleccionar el icono de la extensión __Ejecutar__ en la barra lateral, la entrada __Azure Machine Learning Deployment: Docker Debug__ (Azure Machine Learning Service: depuración de Docker) en el menú desplegable Depurar y, luego, use la flecha verde para adjuntar el depurador.

    ![El icono de depuración, el botón de inicio de la depuración y el selector de configuración](./media/how-to-troubleshoot-deployment/start-debugging.png)
    
    Después de hacer clic en la flecha verde y asociar el depurador, en la interfaz de VS Code del contenedor puede ver información nueva:
    
    ![Información asociada del depurador del contenedor](./media/how-to-troubleshoot-deployment/debugger-attached.png)
    
    Además, en la interfaz principal de VS Code, lo que puede ver es lo siguiente:

    ![El punto de interrupción de VS Code en score.py](./media/how-to-troubleshoot-deployment/local-debugger.png)

Y ahora, el archivo `score.py` local que está asociado al contenedor ya se ha detenido en los puntos de interrupción establecidos. En este punto, VS Code se conecta a debugpy dentro del contenedor de Docker y detiene a este en el punto de interrupción que se estableció anteriormente. Ahora puede recorrer el código a medida que se ejecuta, ver variables, etc.

Para más información sobre cómo usar VS Code para implementar Python, consulte [Depurar el código de Python](https://code.visualstudio.com/docs/python/debugging).

### <a name="stop-the-container"></a>Detención del contenedor

Para detener el contenedor, use el comando siguiente:

```bash
docker stop debug
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado VS Code remoto, puede usar una instancia de proceso como proceso remoto desde VS Code para depurar el código de forma interactiva. 

Más información sobre solución de problemas:

* [Implementación del modelo local](how-to-troubleshoot-deployment-local.md)
* [Implementación del modelo remoto](how-to-troubleshoot-deployment.md)
* [Canalizaciones de aprendizaje automático](how-to-debug-pipelines.md)
* [ParallelRunStep](how-to-debug-parallel-run-step.md)

