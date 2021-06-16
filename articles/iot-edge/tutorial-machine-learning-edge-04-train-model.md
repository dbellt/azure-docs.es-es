---
title: 'Tutorial: Entrenamiento e implementación de un modelo: Machine Learning en Azure IoT Edge'
description: En este tutorial, entrenará un modelo de Machine Learning mediante Azure Machine Learning y, después, empaquetará el modelo como una imagen de contenedor que puede implementarse como un módulo de Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 3/24/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: a0b9c96fcd54ab89d5d454b206ece6fce8f0e34e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111969251"
---
# <a name="tutorial-train-and-deploy-an-azure-machine-learning-model"></a>Tutorial: Entrenamiento e implementación de un modelo de Azure Machine Learning

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

En este artículo, realizará las tareas siguientes:

* Uso de Azure Machine Learning Studio para entrenar un modelo de Machine Learning.
* Empaquetará el modelo entrenado como una imagen de contenedor.
* Implementará la imagen de contenedor como un módulo de Azure IoT Edge.

Machine Learning Studio es un bloque fundamental que se usa para experimentar, entrenar e implementar modelos de Machine Learning.

Normalmente, los científicos de datos se encargan de los pasos descritos en este artículo.

En esta sección del tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear cuadernos de Jupyter Notebook en un área de trabajo de Azure Machine Learning para entrenar un modelo de Machine Learning.
> * Contenedorizar el modelo de aprendizaje automático entrenado.
> * Crear un módulo de IoT Edge a partir del modelo de Machine Learning contenedorizado.

## <a name="prerequisites"></a>Requisitos previos

Este artículo forma parte de un tutorial sobre el uso de Machine Learning en IoT Edge. Cada artículo de la serie se basa en el trabajo del artículo anterior. Si ha llegado a este artículo directamente, consulte el [primer artículo](tutorial-machine-learning-edge-01-intro.md) de la serie.

## <a name="set-up-azure-machine-learning"></a>Configuración de Azure Machine Learning

Se usa Machine Learning Studio para hospedar tanto los dos cuadernos de Jupyter Notebook como los archivos auxiliares. A continuación, se crea y configura un proyecto de Machine Learning. Si no ha usado Jupyter o Machine Learning Studio, aquí tiene dos documentos introductorios:

* **Jupyter Notebook**: [Uso de cuadernos de Jupyter Notebook en Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support)
* **Azure Machine Learning**: [Introducción a Azure Machine Learning en cuadernos de Jupyter Notebook](../machine-learning/quickstart-create-resources.md)

> [!NOTE]
> Una vez configurado el servicio, se puede acceder a Azure Machine Learning desde cualquier máquina. Durante el proceso de configuración, debe usar la máquina virtual de desarrollo que tiene todos los archivos que necesitará.

### <a name="install-azure-machine-learning-visual-studio-code-extension"></a>Instalación de la extensión Azure Machine Learning para Visual Studio Code

En la máquina virtual de desarrollo Visual Studio Code debe estar instalada esta extensión. Si la ejecución se realiza en otra instancia, vuelva a instalar la extensión tal como se describe en el artículo sobre la [configuración de la extensión para Visual Studio Code](../machine-learning/how-to-setup-vs-code.md).

### <a name="create-an-azure-machine-learning-account"></a>Creación de una cuenta de Azure Machine Learning

Para aprovisionar recursos y ejecutar cargas de trabajo en Azure, inicie sesión con las credenciales de su cuenta de Azure.

1. En Visual Studio Code, abra la paleta de comandos; para ello, seleccione **View** > **Command Palette** (Ver > Paleta de comandos) en la barra de menús.

1. Escriba el comando `Azure: Sign In` en la paleta de comandos para empezar el proceso de inicio de sesión. Siga las instrucciones para completar el inicio de sesión.

1. Cree una instancia de proceso de Machine Learning para ejecutar la carga de trabajo. En la paleta de comandos, escriba el comando `Azure ML: Create Compute`.
1. Seleccione su suscripción a Azure.
1. Seleccione **+ Create a new Azure ML Workspace** (Crear un área de trabajo de Azure Machine Learning) y escriba el nombre **turbofandemo**.
1. Seleccione el grupo de recursos que ha usado para esta demostración.
1. Debería poder ver el progreso de la creación del área de trabajo en la esquina inferior derecha de la ventana de Visual Studio Code: **Creating Workspace: turobofandemo** (Creando área de trabajo: turbofandemo). Este paso puede tardar uno o dos minutos.
1. Espere a que el área de trabajo se cree correctamente. Debe decir **Azure ML workspace turbofandemo created** (Área de trabajo turbofandemo de Azure Machine Learning creada).

### <a name="upload-jupyter-notebook-files"></a>Carga de archivos de Jupyter Notebook

Cargaremos archivos del cuaderno de ejemplo en un nuevo área de trabajo de Machine Learning.

1. Vaya a ml.azure.com e inicie sesión.
1. Seleccione el directorio de Microsoft, la suscripción de Azure y el área de trabajo de Machine Learning recién creada.

    :::image type="content" source="media/tutorial-machine-learning-edge-04-train-model/select-studio-workspace.png" alt-text="Captura de pantalla que muestra la selección del área de trabajo de Azure Machine Learning." :::

1. Una vez que haya iniciado sesión en el área de trabajo de Machine Learning, vaya a la sección **Notebooks** del menú del lado izquierdo.
1. Seleccione la pestaña **My files** (Mis archivos).

1. Seleccione **Upload** (Cargar) (el icono de flecha hacia arriba).

1. Vaya a **C:\source\IoTEdgeAndMlSample\AzureNotebooks**. Seleccione todos los archivos de la lista y seleccione **Open** (Abrir).

1. Active la casilla **I trust the content of these files** (Confío en el contenido de estos archivos).

1. Seleccione **Upload** (Cargar) para comenzar la carga. Seleccione **Done** (listo) cuando se haya completado el proceso.

### <a name="jupyter-notebook-files"></a>Archivos de Jupyter Notebook

Vamos a examinar los archivos que ha cargado en el área de trabajo de Machine Learning. Las actividades de esta parte del tutorial abarcan dos archivos de cuaderno, que usan algunos archivos auxiliares.

* **01-turbofan\_regression.ipynb**: este cuaderno usa el área de trabajo de Machine Learning para crear y ejecutar un experimento de aprendizaje automático. En líneas generales, el cuaderno realiza los siguientes pasos:

  1. Descarga datos de la cuenta de Azure Storage generada por la herramienta de ejecución de dispositivos.
  1. Explora y prepara los datos, y los usa para entrenar el modelo de clasificación.
  1. Evalúe el modelo desde el experimento mediante un conjunto de datos de prueba (Test\_FD003.txt).
  1. Publica el mejor modelo de clasificación en el área de trabajo de Machine Learning.

* **02-turbofan\_deploy\_model.ipynb**: este cuaderno toma el modelo creado en el cuaderno anterior y lo usa para crear una imagen de contenedor lista para implementarse en un dispositivo IoT Edge. El cuaderno realiza los siguientes pasos:

  1. Crea un script de puntuación para el modelo.
  1. Genera una imagen de contenedor mediante el modelo de clasificación que se guardó en el área de trabajo de Machine Learning.
  1. Implementa la imagen como un servicio web en Azure Container Instances.
  1. Usa el servicio web para validar el modelo y el trabajo de la imagen según lo previsto. La imagen validada se implementará en el dispositivo IoT Edge en la sección [Creación e implementación de módulos personalizados de IoT Edge](tutorial-machine-learning-edge-06-custom-modules.md) de este tutorial.

* **Test\_FD003.txt**: este archivo contiene los datos que usaremos como conjunto de prueba al validar el clasificador entrenado. Decidimos usar los datos de prueba tal, y como se proporcionan para el concurso original, como nuestro conjunto de pruebas por su sencillez.
* **RUL\_FD003.txt**: este archivo contiene la vida útil restante (RUL) del último ciclo de todos los dispositivos en el archivo Test\_FD003.txt. Consulte los archivos readme.txt y Damage Propagation Modeling.pdf de C:\\source\\IoTEdgeAndMlSample\\data\\Turbofan para obtener una explicación detallada de los datos.
* **Utils.py**: este archivo contiene un conjunto de funciones de utilidad de Python para trabajar con datos. El primer cuaderno contiene una explicación detallada de las funciones.
* **README.MD**: este archivo Léame describe el uso de los cuadernos.

## <a name="run-the-jupyter-notebooks"></a>Ejecución de los cuadernos de Jupyter Notebooks

Ahora que se ha creado el área de trabajo, puede ejecutar los cuadernos.

1. En la página **My files** (Mis archivos), seleccione **01-turbofan\_regression.ipynb**.

    :::image type="content" source="media/tutorial-machine-learning-edge-04-train-model/select-turbofan-notebook.png" alt-text="Captura de pantalla que muestra la selección del primer cuaderno que se va a ejecutar.":::

1. Si el cuaderno aparece en la lista como **Not Trusted** (No de confianza), seleccione el widget **Not Trusted** (No de confianza) en la esquina superior derecha del cuaderno. Cuando aparezca el cuadro de diálogo, seleccione **Trust** (Confiar).

1. Para obtener unos mejores óptimos, lea la documentación de cada celda y ejecútela de forma individual. Seleccione **Ejecutar** en la barra de herramientas. Más adelante, le resultará conveniente ejecutar varias celdas. Puede obviar las advertencias de actualización y desuso.

    Cuando se ejecuta una celda, muestra un asterisco entre corchetes ([\*]). Cuando se completa la operación de la celda, el asterisco se reemplaza por un número y es posible que aparezca el resultado pertinente. Las celdas de los cuadernos se compilan secuencialmente y no se pueden ejecutar varias celdas a la vez.

    También se pueden usar las opciones de ejecución del menú **Celda**. Seleccione **Ctrl + Entrar** para ejecutar una celda y seleccione **Mayús + Entrar** para ejecutar una celda y pasar a la siguiente.

    > [!TIP]
    > Para las operaciones de celda coherentes, evite ejecutar el mismo cuaderno desde varias pestañas del explorador.

1. En la celda que sigue a las instrucciones de **Set global properties** (Establecer propiedades globales), escriba los valores de la suscripción, la configuración y los recursos de Azure. Luego, ejecute la celda.

    ![Captura de pantalla que muestra el establecimiento de propiedades globales en el cuaderno.](media/tutorial-machine-learning-edge-04-train-model/set-global-properties.png)

1. En la celda anterior a **Detalles del área de trabajo**, después de que se haya ejecutado, busque el vínculo en que se le solicita que inicie sesión para autenticarse.

    ![Captura de pantalla que muestra el mensaje de inicio de sesión para la autenticación del dispositivo.](media/tutorial-machine-learning-edge-04-train-model/sign-in-prompt.png)

    Abra el vínculo y escriba el código especificado. Este procedimiento de inicio de sesión autentica el cuaderno de Jupyter Notebook para acceder a los recursos de Azure mediante la interfaz de la línea de comandos multiplataforma de Azure.

    ![Captura de pantalla que muestra la confirmación de la aplicación en el dispositivo.](media/tutorial-machine-learning-edge-04-train-model/cross-platform-cli.png)

1. En la celda que precede a **Explorar los resultados**, copie el valor del identificador de ejecución y péguelo en la celda posterior a **Reconstituir una ejecución**.

   ![Captura de pantalla que muestra cómo se copia el identificador de ejecución entre celdas.](media/tutorial-machine-learning-edge-04-train-model/automl-id.png)

1. Ejecute las restantes celdas en el cuaderno.

1. Guarde el cuaderno y vuelva a la página del proyecto.

1. Abra el archivo **02-turbofan\_deploy\_model.ipynb** y ejecute todas las celdas. Tendrá que iniciar sesión para autenticarse en la celda posterior a **Configurar área de trabajo**.

1. Guarde el cuaderno y vuelva a la página del proyecto.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Para confirmar que los cuadernos se han completado correctamente, compruebe si se han creado algunos elementos.

1. En la pestaña **Mis archivos** de los cuadernos de Machine Learning, seleccione **Actualizar**.

1. Compruebe que se hayan creado los archivos siguientes.

    | Archivo | Descripción |
    | --- | --- |
    | ./aml_config/.azureml/config.json | Archivo de configuración que se usa para crear el área de trabajo de Machine Learning. |
    | ./aml_config/model_config.json | Archivo de configuración necesario para implementar el modelo en el área de trabajo de Machine Learning **turbofanDemo** en Azure. |
    | myenv.yml| Proporciona información sobre las dependencias para el modelo de Machine Learning implementado.|

1. Compruebe que se han creado los siguientes recursos de Azure. Algunos nombres de recursos se anexan con caracteres aleatorios.

    | Recurso de Azure | Nombre |
    | --- | --- |
    | Área de trabajo de Azure Machine Learning | turborfanDemo |
    | Azure Container Registry | turbofandemoxxxxxxxx |
    | Application Insights | turbofaninsightxxxxxxxx |
    | Azure Key Vault | turbofankeyvaultbxxxxxxxx |
    | Azure Storage | turbofanstoragexxxxxxxxx |

### <a name="debugging"></a>Depuración

Puede insertar instrucciones de Python en el cuaderno para su depuración, como por ejemplo, el comando `print()` para mostrar valores. Si ve variables u objetos que no están definidos, ejecute las celdas en las que se hayan declarado o se hayan creado instancias de ellos primero.

Si necesita poner al día los cuadernos, es posible que tenga que eliminar tanto los archivos como los recursos de Azure.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este tutorial forma parte de una serie en la que cada artículo complementa el trabajo realizado en los anteriores. No borre ningún recurso hasta que complete el tutorial final.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se han usado dos cuadernos de Jupyter Notebook que se ejecutan en Machine Learning Studio para utilizar los datos de los dispositivos turbofan para:

- Entrenar un clasificador de RUL.
- Guardar el clasificador como modelo.
- Crear una imagen de contenedor.
- Implementar y probar la imagen como servicio web.

Consulte el artículo siguiente para crear un dispositivo IoT Edge.

> [!div class="nextstepaction"]
> [Configure un dispositivo IoT Edge](tutorial-machine-learning-edge-05-configure-edge-device.md)