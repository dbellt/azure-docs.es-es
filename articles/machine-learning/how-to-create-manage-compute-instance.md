---
title: Creación y administración de una instancia de proceso
titleSuffix: Azure Machine Learning
description: Información sobre la creación y administración de una instancia de proceso de Azure Machine Learning. Se puede usar como entorno de desarrollo o como destino de proceso para desarrollo y pruebas.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: devx-track-azurecli, references_regions
ms.author: sgilley
author: sdgilley
ms.reviewer: sgilley
ms.date: 10/02/2020
ms.openlocfilehash: db6414ecf4b1b5fcbdf52d59c0c79b72998e610a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375222"
---
# <a name="create-and-manage-an-azure-machine-learning-compute-instance"></a>Creación y administración de una instancia de proceso de Azure Machine Learning

Aprenda a crear y administrar una [instancia de proceso](concept-compute-instance.md) en el área de trabajo de Azure Machine Learning.

Use una instancia de proceso como entorno de desarrollo completamente configurado y administrado en la nube. Para el desarrollo y las pruebas, también puede usar la instancia como un [destino de proceso de entrenamiento](concept-compute-target.md#train) o para un [destino de inferencia](concept-compute-target.md#deploy).   Una instancia de proceso puede ejecutar varios trabajos en paralelo y tiene una cola de trabajos. Como entorno de desarrollo, una instancia de proceso no se puede compartir con otros usuarios del área de trabajo.

En este artículo aprenderá a:

* Crear una instancia de proceso
* Administrar (iniciar, detener, reiniciar y eliminar) una instancia de proceso
* Acceder a la ventana de terminal
* Instalar paquetes de R o Python
* Crear entornos o kernels de Jupyter

Las instancias de proceso pueden ejecutar trabajos de manera segura en un [entorno de red virtual](how-to-secure-training-vnet.md), sin necesidad de que las empresas abran puertos SSH. El trabajo se ejecuta en un entorno con contenedores y empaqueta las dependencias del modelo en un contenedor de Docker.

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](/python/api/overview/azure/ml/intro) o la [extensión de Visual Studio Code para Azure Machine Learning](how-to-setup-vs-code.md).

## <a name="create"></a>Crear

> [!IMPORTANT]
> Los elementos marcados (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

**Tiempo estimado**: Aproximadamente 5 minutos.

La creación de una instancia de proceso es un proceso único en el área de trabajo. Puede volver a usar el proceso como una estación de trabajo de desarrollo o como un destino de proceso para el entrenamiento. Puede tener varias instancias de proceso asociadas al área de trabajo.

Los núcleos dedicados por región por cuota de familia de máquinas virtuales y cuota regional total, que se aplica a la creación de instancias de proceso, se unifica y comparte con la cuota de clúster de proceso de Azure Machine Learning. La detención de la instancia de proceso no libera la cuota para garantizar que pueda reiniciar la instancia de proceso. Tenga en cuenta que no es posible cambiar el tamaño de la máquina virtual de la instancia de proceso una vez creada.

En el siguiente ejemplo se muestra cómo crear una instancia de proceso:

# <a name="python"></a>[Python](#tab/python)

```python
import datetime
import time

from azureml.core.compute import ComputeTarget, ComputeInstance
from azureml.core.compute_target import ComputeTargetException

# Choose a name for your instance
# Compute instance name should be unique across the azure region
compute_name = "ci{}".format(ws._workspace_id)[:10]

# Verify that instance does not exist already
try:
    instance = ComputeInstance(workspace=ws, name=compute_name)
    print('Found existing instance, use it.')
except ComputeTargetException:
    compute_config = ComputeInstance.provisioning_configuration(
        vm_size='STANDARD_D3_V2',
        ssh_public_access=False,
        # vnet_resourcegroup_name='<my-resource-group>',
        # vnet_name='<my-vnet-name>',
        # subnet_name='default',
        # admin_user_ssh_public_key='<my-sshkey>'
    )
    instance = ComputeInstance.create(ws, compute_name, compute_config)
    instance.wait_for_completion(show_output=True)
```

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [Clase ComputeInstance](/python/api/azureml-core/azureml.core.compute.computeinstance.computeinstance)
* [ComputeTarget.create](/python/api/azureml-core/azureml.core.compute.computetarget#create-workspace--name--provisioning-configuration-)
* [ComputeInstance.wait_for_completion](/python/api/azureml-core/azureml.core.compute.computeinstance(class)#wait-for-completion-show-output-false--is-delete-operation-false-)


# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az ml computetarget create computeinstance  -n instance -s "STANDARD_D3_V2" -v
```

Para más información, vea la referencia de [az ml computetarget create computeinstance](/cli/azure/ml/computetarget/create#az_ml_computetarget_create_computeinstance).

# <a name="studio"></a>[Estudio](#tab/azure-studio)

En el área de trabajo de Azure Machine Learning Studio, cree una nueva instancia de proceso desde la sección **Proceso** o en la sección **Notebooks** cuando esté listo para ejecutar uno de sus cuadernos.

Para obtener información sobre cómo crear una instancia de proceso en Estudio, vea [Creación de destinos de proceso en Estudio de Azure Machine Learning](how-to-create-attach-compute-studio.md#compute-instance).

---

También puede crear una instancia de proceso con una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/machine-learning-compute-create-computeinstance).



## <a name="create-on-behalf-of-preview"></a><a name="on-behalf"></a> Creación en nombre de alguien (versión preliminar)

Como administrador, puede crear una instancia de proceso en nombre de un científico de datos y asignarle la instancia con:

* [Plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/machine-learning-compute-create-computeinstance)  Para más información sobre cómo buscar los valores de TenantID y ObjectID necesarios en esta plantilla, consulte [Encontrar identificadores de objeto de identidad para la configuración de autenticación](../healthcare-apis/fhir/find-identity-object-ids.md).  También puede encontrar estos valores en el portal de Azure Active Directory.

* API DE REST

El científico de datos para el que se crea la instancia de proceso necesita los siguientes permisos de [control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/overview.md):
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*
* *Microsoft.MachineLearningServices/workspaces/computes/applicationaccess/action*

El científico de datos puede iniciar, detener y reiniciar la instancia de proceso. Puede utilizar la instancia de proceso para:
* Jupyter
* JupyterLab
* RStudio
* Cuadernos integrados

## <a name="customize-the-compute-instance-with-a-script-preview"></a><a name="setup-script"></a> Personalización de la instancia de proceso con un script (versión preliminar)

Use un script de configuración para una manera automatizada de personalizar y configurar la instancia de proceso en el momento del aprovisionamiento. Como administrador, puede escribir un script de personalización que se usará para aprovisionar todas las instancias de proceso del área de trabajo según sus requisitos.

A continuación se muestran algunos ejemplos de lo que puede hacer en un script de configuración:

* Instalar paquetes, herramientas y software
* Montar datos
* Crear un entorno de Conda personalizado y kernels de Jupyter
* Clonar repositorios de Git y configurar opciones de Git
* Establecer servidores proxy de red
* Establecimiento de variables de entorno
* Instalar extensiones de JupyterLab

### <a name="create-the-setup-script"></a>Creación del script de configuración

El script de configuración es un script de shell que se ejecuta como *rootuser*.  Cree o cargue el script en los archivos de **Notebooks**:

1. Inicie sesión en [Studio](https://ml.azure.com) y vaya al área de trabajo.
2. Seleccione **Notebooks** en la parte izquierda.
3. Use la herramienta **Agregar archivos** para crear o cargar el script del shell de configuración.  Asegúrese de que el nombre de archivo del script finaliza en ".sh".  Al crear un nuevo archivo, cambie también el **tipo de archivo** a *bash(.sh)* .

:::image type="content" source="media/how-to-create-manage-compute-instance/create-or-upload-file.png" alt-text="Creación o carga del script de configuración en el archivo de Notebooks en Studio":::

Cuando se ejecuta el script, el directorio de trabajo actual del script es el directorio donde se cargó. Por ejemplo, si carga el script en **Users>admin**, la ubicación del script en la instancia de proceso y el directorio de trabajo actual cuando se ejecuta el script es */home/azureuser/cloudfiles/code/Users/admin*. Esto le permitiría usar rutas de acceso relativas en el script.

En el script se pueden hacer referencia a los argumentos del script como $1, $2, etc.

Si el script estaba haciendo algo específico de azureuser, como instalar el entorno de Conda o el kernel de Jupyter, tendrá que colocarlo en el bloque *sudo -u azureuser* de este modo.

```shell
sudo -u azureuser -i <<'EOF'

EOF
```
Tenga en cuenta que *sudo -u azureuser* sí cambia el directorio de trabajo actual a */home/azureuser*. Tampoco puede acceder a los argumentos del script de este bloque.

También puede usar las siguientes variables de entorno en el script:

1. CI_RESOURCE_GROUP
2. CI_WORKSPACE
3. CI_NAME
4. CI_LOCAL_UBUNTU_USER. Esta apunta a azureuser.

### <a name="use-the-script-in-the-studio"></a>Uso del script en Studio

Una vez que almacene el script, especifíquelo durante la creación de la instancia de proceso:

1. Inicie sesión en [Studio](https://ml.azure.com/) y seleccione el área de trabajo.
1. Seleccione **Proceso** a la izquierda.
1. Seleccione **+Nuevo** para crear una nueva instancia de proceso.
1. [Rellene el formulario](how-to-create-attach-compute-studio.md#compute-instance).
1. En la segunda página del formulario, abra **Mostrar configuración avanzada**.
1. Active la opción **Aprovisionar con un script de configuración**.
1. Vaya al script de shell que guardó.  O bien, cargue un script desde el equipo.
1. Agregue argumentos de comando según sea necesario.

:::image type="content" source="media/how-to-create-manage-compute-instance/setup-script.png" alt-text="Aprovisionamiento de una instancia de proceso con un script de configuración en Studio.":::

Tenga en cuenta que si el almacenamiento del área de trabajo está conectado a una red virtual, es posible que no pueda acceder al archivo de script de instalación a menos que acceda a Studio desde la red virtual.

### <a name="use-script-in-a-resource-manager-template"></a>Uso de una plantilla de Resource Manager

En una [plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/machine-learning-compute-create-computeinstance)de Resource Manager, agregue `setupScripts` para invocar el script de configuración cuando se aprovisione la instancia de proceso. Por ejemplo:

```json
"setupScripts":{
    "scripts":{
        "creationScript":{
        "scriptSource":"workspaceStorage",
        "scriptData":"[parameters('creationScript.location')]",
        "scriptArguments":"[parameters('creationScript.cmdArguments')]"
        }
    }
}
```

En su lugar, podría proporcionar el script en línea para una plantilla de Resource Manager.  El comando de shell puede hacer referencia a las dependencias cargadas en el recurso compartido de archivos de Notebooks.  Cuando se usa una cadena insertada, el directorio de trabajo para el script es */mnt/batch/tasks/shared/LS_root/mounts/clusters/**ciname**/code/Users*.

Por ejemplo, especifique una cadena de comando codificada en Base64 para `scriptData`:

```json
"setupScripts":{
    "scripts":{
        "creationScript":{
        "scriptSource":"inline",
        "scriptData":"[base64(parameters('inlineCommand'))]",
        "scriptArguments":"[parameters('creationScript.cmdArguments')]"
        }
    }
}
```

### <a name="setup-script-logs"></a>Registros del script de configuración

Los registros de la ejecución del script de configuración aparecen en la carpeta de registros de la página de detalles de la instancia de proceso. Los registros se almacenan de nuevo en el recurso compartido de archivos de Notebooks en la carpeta Registros\<compute instance name>. Los argumentos de archivo de script y comando para una instancia de proceso determinada se muestran en la página de detalles.

## <a name="manage"></a>Administrar

Inicie, detenga, reinicie y elimine una instancia de proceso. Una instancia de proceso no se reduce verticalmente de forma automática, por lo que debe asegurarse de detener el recurso para evitar cargos continuos. Al detener una instancia de proceso, se cancela su asignación. A continuación, inícielo de nuevo cuando lo necesite. Si bien detener la instancia de proceso detiene la facturación de las horas de proceso, se le seguirá facturando por el disco, la dirección IP pública y el equilibrador de carga estándar.

> [!TIP]
> La instancia de proceso tiene un disco de SO de 120 GB. Si se queda sin espacio en disco, [use el terminal](how-to-access-terminal.md) para borrar al menos entre 1 y 2 GB antes de detener o reiniciar la instancia de proceso.

# <a name="python"></a>[Python](#tab/python)

En todos los siguientes ejemplos, el nombre de la instancia de proceso es **instance**.

* Obtener estado

    ```python
    # get_status() gets the latest status of the ComputeInstance target
    instance.get_status()
    ```

* Stop

    ```python
    # stop() is used to stop the ComputeInstance
    # Stopping ComputeInstance will stop the billing meter and persist the state on the disk.
    # Available Quota will not be changed with this operation.
    instance.stop(wait_for_completion=True, show_output=True)
    ```

* Start

    ```python
    # start() is used to start the ComputeInstance if it is in stopped state
    instance.start(wait_for_completion=True, show_output=True)
    ```

* Reinicio

    ```python
    # restart() is used to restart the ComputeInstance
    instance.restart(wait_for_completion=True, show_output=True)
    ```

* Eliminar

    ```python
    # delete() is used to delete the ComputeInstance target. Useful if you want to re-use the compute name
    instance.delete(wait_for_completion=True, show_output=True)
    ```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En todos los siguientes ejemplos, el nombre de la instancia de proceso es **instance**.

* Stop

    ```azurecli-interactive
    az ml computetarget stop computeinstance -n instance -v
    ```

    Para más información, consulte [az ml computetarget stop computeinstance](/cli/azure/ml/computetarget/computeinstance#az_ml_computetarget_computeinstance_stop).

* Start

    ```azurecli-interactive
    az ml computetarget start computeinstance -n instance -v
    ```

    Para más información, consulte [az ml computetarget start computeinstance](/cli/azure/ml/computetarget/computeinstance#az_ml_computetarget_computeinstance_start).

* Reinicio

    ```azurecli-interactive
    az ml computetarget restart computeinstance -n instance -v
    ```

    Para más información, consulte[az ml computetarget restart computeinstance](/cli/azure/ml/computetarget/computeinstance#az_ml_computetarget_computeinstance_restart).

* Eliminar

    ```azurecli-interactive
    az ml computetarget delete -n instance -v
    ```

    Para más información, consulte [az ml computetarget delete computeinstance](/cli/azure/ml/computetarget#az_ml_computetarget_delete).

# <a name="studio"></a>[Estudio](#tab/azure-studio)

En el área de trabajo de Azure Machine Learning Studio, seleccione **Compute** (Proceso) y, luego, **Compute Instance** (Instancia de proceso) en la parte superior.

![Administración de una instancia de proceso](./media/concept-compute-instance/manage-compute-instance.png)

Puede realizar las siguientes acciones:

* Crear una nueva instancia de proceso
* Actualizar la pestaña instancias de proceso.
* Iniciar, detener y reiniciar una instancia de proceso.  Se paga por la instancia cada vez que se ejecuta. Detenga la instancia de proceso cuando no la use para reducir el costo. Al detener una instancia de proceso, se cancela su asignación. A continuación, inícielo de nuevo cuando lo necesite.
* Elimine una instancia de proceso.
* Filtre la lista de instancias de proceso para mostrar solo las que haya creado.

Para cada instancia de proceso del área de trabajo que creó, o que se creó automáticamente, puede:

* Acceder a Jupyter, JupyterLab y RStudio en la instancia de proceso.
* SSH en la instancia de proceso. El acceso SSH está deshabilitado de forma predeterminada, pero se puede habilitar en el momento de la creación de la instancia de proceso. El acceso SSH se realiza a través del mecanismo de claves pública y privada. La pestaña le proporcionará detalles sobre la conexión SSH, como la dirección IP, el nombre de usuario y el número de puerto.
* Más detalles sobre una instancia de proceso específica, como la dirección IP y la región.

---

[RBAC de Azure](../role-based-access-control/overview.md) permite controlar qué usuarios del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar una instancia de proceso. Todos los usuarios del rol colaborador y propietario del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar las instancias de proceso en el área de trabajo. Sin embargo, solo el creador de una instancia de proceso específica o el usuario asignado, si se creó en su nombre, tienen permiso para acceder a Jupyter, JupyterLab y RStudio en esa instancia de proceso. Una instancia de proceso está dedicada a un solo usuario que tiene acceso raíz y puede pasar por el terminal a través de Jupyter, JupyterLab o RStudio. La instancia de proceso incluirá el usuario que ha iniciado sesión y todas las acciones usarán la identidad de ese usuario para RBAC de Azure y la atribución de ejecuciones de experimentos. El acceso SSH se controla mediante un mecanismo de clave pública-privada.

RBAC de Azure puede controlar estas acciones:
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*

Para crear una instancia de proceso, debe tener permisos para realizar las siguientes acciones:
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/checkComputeNameAvailability/action*


## <a name="next-steps"></a>Pasos siguientes

* [Acceso al terminal de la instancia de proceso](how-to-access-terminal.md)
* [Creación y administración de archivos](how-to-manage-files.md)
* [Envío de una ejecución de entrenamiento](how-to-set-up-training-targets.md)
