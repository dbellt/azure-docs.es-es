---
title: Seguimiento, supervisión y análisis de ejecuciones
titleSuffix: Azure Machine Learning
description: Aprenda a iniciar, supervisar y seguir las ejecuciones de los experimentos de aprendizaje automático con el SDK de Azure Machine Learning para Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: swinner95
ms.author: shwinne
ms.reviewer: sgilley
ms.date: 04/19/2021
ms.topic: how-to
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: d50820e954c1a34f1ccffe133a338538bf0abd18
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888537"
---
# <a name="start-monitor-and-track-run-history"></a>Inicio, supervisión y seguimiento del historial de ejecución

El [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro), la [CLI de Machine Learning](reference-azure-machine-learning-cli.md) y [Estudio de Azure Machine Learning](https://ml.azure.com) proporcionan varios métodos para supervisar, organizar y seguir las ejecuciones de entrenamiento y experimentación. El historial de ejecución de ML es una parte importante de un proceso de desarrollo de ML que se pueda explicar y repetir.

En este artículo se muestra cómo realizar las tareas siguientes:

* Supervisión del rendimiento de la ejecución.
* Creación de una vista personalizada. 
* Incorporación de la descripción de una ejecución. 
* Etiquetado y búsqueda de ejecuciones.
* Ejecución de búsquedas en el historial de ejecución. 
* Cancelación o ejecuciones erróneas.
* Creación de ejecuciones secundarias.
* Supervisión del estado de la ejecución mediante notificaciones por correo electrónico.
 

> [!TIP]
> Si busca información sobre cómo supervisar Azure Machine Learning Service y los servicios de Azure asociados, consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).
> Si está buscando información sobre la supervisión de los modelos implementados como servicios web o módulos de IoT Edge, consulte [Recopilación de datos de modelos](how-to-enable-data-collection.md) y [Supervisión con Application Insights](how-to-enable-app-insights.md).

## <a name="prerequisites"></a>Prerrequisitos

Necesitará los siguientes elementos:

* Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* El SDK de Azure Machine Learning para Python (versión 1.0.21 o posterior). Para instalar o actualizar a la versión más reciente del SDK, consulte [Instalación o actualización del SDK](/python/api/overview/azure/ml/install).

    Para comprobar su versión del SDK de Azure Machine Learning, use el siguiente código:

    ```python
    print(azureml.core.VERSION)
    ```

* La [CLI de Azure](/cli/azure/?preserve-view=true&view=azure-cli-latest) y la [extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).


## <a name="monitor-run-performance"></a>Supervisión del rendimiento de la ejecución

* Inicio de una ejecución y su proceso de registro

    # <a name="python"></a>[Python](#tab/python)
    
    1. Para configurar el experimento, importe las clases [Workspace](/python/api/azureml-core/azureml.core.workspace.workspace), [Experiment](/python/api/azureml-core/azureml.core.experiment.experiment), [Run](/python/api/azureml-core/azureml.core.run%28class%29) y [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig) del paquete [azureml.core](/python/api/azureml-core/azureml.core).
    
        ```python
        import azureml.core
        from azureml.core import Workspace, Experiment, Run
        from azureml.core import ScriptRunConfig
        
        ws = Workspace.from_config()
        exp = Experiment(workspace=ws, name="explore-runs")
        ```
    
    1. Inicie una ejecución y su proceso de registro con el método [`start_logging()`](/python/api/azureml-core/azureml.core.experiment%28class%29#start-logging--args----kwargs-).
    
        ```python
        notebook_run = exp.start_logging()
        notebook_run.log(name="message", value="Hello from run!")
        ```
        
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    Para iniciar una ejecución del experimento, use los pasos siguientes:
    
    1. En el shell o el símbolo del sistema, use la CLI de Azure para autenticarse en la suscripción de Azure:
    
        ```azurecli-interactive
        az login
        ```
        
        [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 
    
    1. Adjunte una configuración de área de trabajo a la carpeta que contiene el script de entrenamiento. Reemplace `myworkspace` por el área de trabajo de Azure Machine Learning. Reemplace `myresourcegroup` por el grupo de recursos de Azure que contiene el área de trabajo:
    
        ```azurecli-interactive
        az ml folder attach -w myworkspace -g myresourcegroup
        ```
    
        Este comando crea un subdirectorio `.azureml` que contiene archivos de entorno de conda y runconfig de ejemplo. También contiene un archivo `config.json` que se usa para comunicarse con el área de trabajo de Azure Machine Learning.
    
        Para obtener más información, consulte [az ml folder attach](/cli/azure/ml/folder?preserve-view=true&view=azure-cli-latest#az_ml_folder_attach).
    
    2. Para iniciar la ejecución, use el comando siguiente. Cuando use este comando, especifique el nombre del archivo runconfig (el texto antes de \*.runconfig si está mirando el sistema de archivos) en el parámetro -c.
    
        ```azurecli-interactive
        az ml run submit-script -c sklearn -e testexperiment train.py
        ```
    
        > [!TIP]
        > El comando `az ml folder attach` ha creado un subdirectorio `.azureml`, que contiene dos archivos runconfig de ejemplo.
        >
        > Si tiene un script de Python que crea un objeto de configuración de ejecución mediante programación, puede usar [RunConfig.save()](/python/api/azureml-core/azureml.core.runconfiguration#save-path-none--name-none--separate-environment-yaml-false-) para guardarlo como un archivo runconfig.
        >
        > Para ver más archivos runconfig de ejemplo, consulte [https://github.com/MicrosoftDocs/pipelines-azureml/](https://github.com/MicrosoftDocs/pipelines-azureml/).
    
        Para obtener más información, consulte [az ml run submit-script](/cli/azure/ml/run?preserve-view=true&view=azure-cli-latest#az_ml_run_submit-script).

    # <a name="studio"></a>[Estudio](#tab/azure-studio)

    Para obtener un ejemplo del entrenamiento de un modelo en el diseñador de Azure Machine Learning, consulte [Tutorial: Predicción del precio de un automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md).

    ---

* Supervisión del estado de una ejecución

    # <a name="python"></a>[Python](#tab/python)
    
    * Obtenga el estado de una ejecución con el método [`get_status()`](/python/api/azureml-core/azureml.core.run%28class%29#get-status--).
    
        ```python
        print(notebook_run.get_status())
        ```
    
    * Para obtener el identificador de ejecución, la hora de ejecución y otros detalles sobre esta, use el método [`get_details()`](/python/api/azureml-core/azureml.core.workspace.workspace#get-details--).
    
        ```python
        print(notebook_run.get_details())
        ```
    
    * Cuando la ejecución finalice correctamente, use el método [`complete()`](/python/api/azureml-core/azureml.core.run%28class%29#complete--set-status-true-) para marcarla como completada.
    
        ```python
        notebook_run.complete()
        print(notebook_run.get_status())
        ```
    
    * Si usa el modelo de diseño `with...as` de Python, la ejecución se marcará automáticamente como completada cuando esta quede fuera del ámbito. No tiene que marcar la ejecución como completada de forma manual.
        
        ```python
        with exp.start_logging() as notebook_run:
            notebook_run.log(name="message", value="Hello from run!")
            print(notebook_run.get_status())
        
        print(notebook_run.get_status())
        ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    * Para ver una lista de ejecuciones del experimento, use el siguiente comando. Reemplace `experiment` por el nombre del experimento:
    
        ```azurecli-interactive
        az ml run list --experiment-name experiment
        ```
    
        Este comando devuelve un documento JSON que muestra información sobre las ejecuciones de este experimento.
    
        Para obtener más información, consulte [az ml experiment list](/cli/azure/ml/experiment?preserve-view=true&view=azure-cli-latest#az_ml_experiment_list).
    
    * Para ver información sobre una ejecución concreta, use el siguiente comando. Reemplace `runid` por el identificador de la ejecución:
    
        ```azurecli-interactive
        az ml run show -r runid
        ```
    
        Este comando devuelve un documento JSON que muestra información sobre la ejecución.
    
        Para obtener más información, consulte [az ml run show](/cli/azure/ml/run?preserve-view=true&view=azure-cli-latest#az_ml_run_show).
    
    
    # <a name="studio"></a>[Estudio](#tab/azure-studio)
    
    ---
## <a name="custom-view"></a>Vista personalizada 
    
Para ver las ejecuciones en el estudio: 
    
1. Vaya a la sección **Experimentos**.
    
1. Seleccione **Todos los experimentos** para ver todas las ejecuciones de un experimento o seleccione **Todas las ejecuciones** para ver todas las ejecuciones enviadas en el área de trabajo.
    
En la página **Todas las ejecuciones**, puede filtrar la lista de ejecuciones por etiquetas, experimentos y destino de proceso, entre otros criterios, para organizar mejor el ámbito del trabajo.  
    
1. Para hacer personalizaciones en la página, seleccione ejecuciones para comparar, agregue gráficos o aplique filtros. Estos cambios se pueden guardar como una **vista personalizada** para que pueda volver fácilmente a su trabajo. Los usuarios con permisos en el área de trabajo pueden editar o ver la vista personalizada. Además, comparta la vista personalizada con los miembros del equipo para mejorar la colaboración seleccionando **Compartir vista**.   
    
:::image type="content" source="media/how-to-track-monitor-analyze-runs/custom-views.gif" alt-text="Captura de pantalla: creación de una vista personalizada":::
    
1. Para ver los registros de ejecución, seleccione una ejecución específica y, en la pestaña **Resultados y registros**, puede encontrar registros de diagnóstico y errores para la ejecución.

## <a name="run-description"></a>Descripción de la ejecución 

Se puede agregar una descripción a una ejecución para proporcionar más contexto e información a la ejecución. También puede buscar en estas descripciones desde la lista de ejecuciones y agregar la descripción de la ejecución como una columna en la lista de ejecuciones. 

Vaya a la página **Detalles de ejecución** de la ejecución y seleccione el icono de edición o de lápiz para agregar, editar o eliminar descripciones de la ejecución. Para conservar los cambios en la lista de ejecuciones, guarde los cambios en la vista personalizada existente o en una nueva vista personalizada. Se admite el formato Markdown para las descripciones de ejecución, lo que permite la inserción de imágenes y la vinculación en profundidad, como se muestra a continuación.

:::image type="content" source="media/how-to-track-monitor-analyze-runs/run-description.gif" alt-text="Captura de pantalla: crear una descripción de ejecución"::: 

## <a name="tag-and-find-runs"></a>Etiquetado y búsqueda de ejecuciones

En Azure Machine Learning, puede usar etiquetas y propiedades para ayudar a organizar y consultar las ejecuciones a fin de obtener información importante.

* Adición de etiquetas y propiedades

    # <a name="python"></a>[Python](#tab/python)
    
    Para agregar metadatos de búsqueda a las ejecuciones, use el método [`add_properties()`](/python/api/azureml-core/azureml.core.run%28class%29#add-properties-properties-). Por ejemplo, el código siguiente agrega la propiedad `"author"` a la ejecución:
    
    ```Python
    local_run.add_properties({"author":"azureml-user"})
    print(local_run.get_properties())
    ```
    
    Las propiedades son inmutables, por lo que crean un registro permanente con fines de auditoría. El siguiente ejemplo de código da como resultado un error porque ya hemos agregado `"azureml-user"` como el valor de propiedad `"author"` en el código anterior:
    
    ```Python
    try:
        local_run.add_properties({"author":"different-user"})
    except Exception as e:
        print(e)
    ```
    
    A diferencia de las propiedades, las etiquetas son mutables. Para agregar información que permite realizar búsquedas y que es significativa para los consumidores del experimento, use el método [`tag()`](/python/api/azureml-core/azureml.core.run%28class%29#tag-key--value-none-).
    
    ```Python
    local_run.tag("quality", "great run")
    print(local_run.get_tags())
    
    local_run.tag("quality", "fantastic run")
    print(local_run.get_tags())
    ```
    
    También puede agregar etiquetas de cadena simples. Cuando estas etiquetas aparecen en el diccionario de etiquetas como claves, tienen un valor de `None`.
    
    ```Python
    local_run.tag("worth another look")
    print(local_run.get_tags())
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    > [!NOTE]
    > Con la CLI, solo puede agregar o actualizar las etiquetas.
    
    Para agregar o actualizar una etiqueta, use el siguiente comando:
    
    ```azurecli-interactive
    az ml run update -r runid --add-tag quality='fantastic run'
    ```
    
    Para obtener más información, consulte [az ml run update](/cli/azure/ml/run?preserve-view=true&view=azure-cli-latest#az_ml_run_update).
    
    # <a name="studio"></a>[Estudio](#tab/azure-studio)
    
    Puede agregar, editar o eliminar etiquetas de ejecución desde el estudio. Vaya a la página **Detalles de ejecución** de la ejecución y seleccione el icono de edición o de lápiz para agregar, editar o eliminar etiquetas de las ejecuciones. También puede buscar y filtrar en estas etiquetas desde la página de la lista de ejecuciones.
    
    :::image type="content" source="media/how-to-track-monitor-analyze-runs/run-tags.gif" alt-text="Captura de pantalla: adición, edición o eliminación de etiquetas de ejecución":::
    
    ---

* Consulta de etiquetas y propiedades

    Puede consultar las ejecuciones de un experimento para devolver una lista de ejecuciones que coinciden con etiquetas y propiedades específicas.

    # <a name="python"></a>[Python](#tab/python)
    
    ```Python
    list(exp.get_runs(properties={"author":"azureml-user"},tags={"quality":"fantastic run"}))
    list(exp.get_runs(properties={"author":"azureml-user"},tags="worth another look"))
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    La CLI de Azure admite las consultas [JMESPath](http://jmespath.org), que se pueden usar para filtrar las ejecuciones en función de etiquetas y propiedades. Para usar una consulta JMESPath con la CLI de Azure, especifíquela con el parámetro `--query`. En los ejemplos siguientes se muestran algunas consultas que usan etiquetas y propiedades:
    
    ```azurecli-interactive
    # list runs where the author property = 'azureml-user'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user']
    # list runs where the tag contains a key that starts with 'worth another look'
    az ml run list --experiment-name experiment [?tags.keys(@)[?starts_with(@, 'worth another look')]]
    # list runs where the author property = 'azureml-user' and the 'quality' tag starts with 'fantastic run'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user' && tags.quality=='fantastic run']
    ```
    
    Para obtener más información sobre la consulta de resultados de la CLI de Azure, vea [Resultados de los comandos de consulta de la CLI de Azure](/cli/azure/query-azure-cli?preserve-view=true&view=azure-cli-latest).
    
    # <a name="studio"></a>[Estudio](#tab/azure-studio)
    
    Para buscar ejecuciones específicas, vaya a la lista **Todas las ejecuciones**. Cuenta con dos opciones:
    
    1. Puede usar el botón **Agregar filtro** y seleccione filtrar por etiquetas a fin de filtrar las ejecuciones en función de la etiqueta que se les asignó. <br><br>
    O BIEN
    
    1. Puede usar la barra de búsqueda para filtrar rápidamente las ejecuciones mediante la búsqueda de metadatos de la ejecución, como su estado, las descripciones, los nombres de los experimentos y el nombre del remitente. 
    
## <a name="cancel-or-fail-runs"></a>Cancelación o ejecuciones erróneas

Si detecta un error o si la ejecución tarda demasiado en finalizar, puede cancelarla.

# <a name="python"></a>[Python](#tab/python)

Para cancelar una ejecución mediante el SDK, use el método [`cancel()`](/python/api/azureml-core/azureml.core.run%28class%29#cancel--):

```python
src = ScriptRunConfig(source_directory='.', script='hello_with_delay.py')
local_run = exp.submit(src)
print(local_run.get_status())

local_run.cancel()
print(local_run.get_status())
```

Si la ejecución finaliza, pero contiene un error (por ejemplo, se ha usado el script de entrenamiento incorrecto), puede usar el método [`fail()`](/python/api/azureml-core/azureml.core.run%28class%29#fail-error-details-none--error-code-none---set-status-true-) para marcarla como errónea.

```python
local_run = exp.submit(src)
local_run.fail()
print(local_run.get_status())
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para cancelar una ejecución con la CLI, use el siguiente comando. Reemplace `runid` por el identificador de la ejecución.

```azurecli-interactive
az ml run cancel -r runid -w workspace_name -e experiment_name
```

Para obtener más información, consulte [az ml run cancel](/cli/azure/ml/run?preserve-view=true&view=azure-cli-latest#az_ml_run_cancel).

# <a name="studio"></a>[Estudio](#tab/azure-studio)

Para cancelar una ejecución en Studio, siga estos pasos:

1. Vaya a la canalización en ejecución en la sección **Experimentos** o **Canalizaciones**. 

1. Seleccione el número de ejecución de la canalización que quiere cancelar.

1. En la barra de herramientas, seleccione **Cancelar**.

---

## <a name="create-child-runs"></a>Creación de ejecuciones secundarias

Cree ejecuciones secundarias para agrupar ejecuciones relacionadas (por ejemplo, para diferentes iteraciones de ajuste de hiperparámetros).

> [!NOTE]
> Las ejecuciones secundarias solo se pueden crear mediante el SDK.

En este ejemplo de código se usa el script `hello_with_children.py` para crear un lote de cinco ejecuciones secundarias desde una ejecución enviada mediante el método [`child_run()`](/python/api/azureml-core/azureml.core.run%28class%29#child-run-name-none--run-id-none--outputs-none-):

```python
!more hello_with_children.py
src = ScriptRunConfig(source_directory='.', script='hello_with_children.py')

local_run = exp.submit(src)
local_run.wait_for_completion(show_output=True)
print(local_run.get_status())

with exp.start_logging() as parent_run:
    for c,count in enumerate(range(5)):
        with parent_run.child_run() as child:
            child.log(name="Hello from child run", value=c)
```

> [!NOTE]
> A medida que quedan fuera del ámbito, las ejecuciones secundarias se marcan automáticamente como completadas.

Para crear muchas ejecuciones secundarias de forma eficaz, use el método [`create_children()`](/python/api/azureml-core/azureml.core.run.run#create-children-count-none--tag-key-none--tag-values-none-). Dado que cada creación da lugar a una llamada de red, la creación de un lote de ejecuciones es más eficaz que hacerlo una a una.

### <a name="submit-child-runs"></a>Envío de ejecuciones secundarias

Las ejecuciones secundarias también se pueden enviar desde una ejecución principal. Esto le permite crear jerarquías de ejecuciones primarias y secundarias. No se puede crear una ejecución secundaria sin primaria: aunque la ejecución primaria no haga más que iniciar ejecuciones secundarias, sigue siendo necesaria para crear la jerarquía. Los estados de todas las ejecuciones son independientes: una ejecución primaria puede tener el estado `"Completed"` correcto aunque una o varias ejecuciones secundarias se hayan cancelado o hayan dado error.  

Es posible que desee que las ejecuciones secundarias utilicen una configuración de ejecución diferente de la ejecución primaria. Por ejemplo, puede utilizar una configuración menos potente basada en CPU para el elemento primario, mientras usa configuraciones basadas en GPU para los elementos secundarios. Otro deseo habitual es pasar a cada ejecución secundaria argumentos y datos distintos. Para personalizar una ejecución secundaria, cree un objeto de `ScriptRunConfig` para la ejecución secundaria. 

> [!IMPORTANT]
> Para enviar una ejecución secundaria desde una ejecución primaria en un proceso remoto, primero debe iniciar sesión en el área de trabajo, en el código de ejecución principal. De forma predeterminada, el objeto de contexto de una ejecución remota no tiene credenciales para enviar ejecuciones secundarias. Use credenciales de identidad administrada o una entidad de servicio para iniciar sesión. Para obtener más información sobre la autenticación, consulte el documento sobre la [configuración de la autenticación](how-to-setup-authentication.md).

El código siguiente:

- Recupera un recurso de proceso denominado `"gpu-cluster"` del área de trabajo `ws`
- Recorre en iteración los diferentes valores de los argumentos que se van a pasar a los objetos `ScriptRunConfig` secundarios
- Crea y envía una nueva ejecución secundaria mediante el recurso de proceso personalizado y el argumento
- Se bloquea hasta que se completan todas las ejecuciones secundarias

```python
# parent.py
# This script controls the launching of child scripts
from azureml.core import Run, ScriptRunConfig

compute_target = ws.compute_targets["gpu-cluster"]

run = Run.get_context()

child_args = ['Apple', 'Banana', 'Orange']
for arg in child_args: 
    run.log('Status', f'Launching {arg}')
    child_config = ScriptRunConfig(source_directory=".", script='child.py', arguments=['--fruit', arg], compute_target=compute_target)
    # Starts the run asynchronously
    run.submit_child(child_config)

# Experiment will "complete" successfully at this point. 
# Instead of returning immediately, block until child runs complete

for child in run.get_children():
    child.wait_for_completion()
```

Para crear muchas ejecuciones secundarias con configuraciones, argumentos y entradas idénticas de forma eficaz, use el método [`create_children()`](/python/api/azureml-core/azureml.core.run.run#create-children-count-none--tag-key-none--tag-values-none-). Dado que cada creación da lugar a una llamada de red, la creación de un lote de ejecuciones es más eficaz que hacerlo una a una.

Desde la ejecución secundaria, puede ver el identificador de la ejecución principal:

```python
## In child run script
child_run = Run.get_context()
child_run.parent.id
```

### <a name="query-child-runs"></a>Consulta de ejecuciones secundarias

Para consultar las ejecuciones secundarias de un elemento primario específico, use el método [`get_children()`](/python/api/azureml-core/azureml.core.run%28class%29#get-children-recursive-false--tags-none--properties-none--type-none--status-none---rehydrate-runs-true-). El argumento ``recursive = True`` permite consultar un árbol anidado de elementos secundarios y descendientes.

```python
print(parent_run.get_children())
```

### <a name="log-to-parent-or-root-run"></a>Registro en la ejecución primaria o raíz

Puede usar el campo `Run.parent` para acceder a la ejecución que inició la ejecución secundaria actual. Un caso de uso habitual de `Run.parent` es combinar resultados de registro en una única ubicación. Las ejecuciones secundarias se ejecutan de forma asincrónica y no hay ninguna garantía de ordenación o sincronización más allá de la capacidad del elemento primario de esperar a que se completen sus ejecuciones secundarias.

```python
# in child (or even grandchild) run

def root_run(self : Run) -> Run :
    if self.parent is None : 
        return self
    return root_run(self.parent)

current_child_run = Run.get_context()
root_run(current_child_run).log("MyMetric", f"Data from child run {current_child_run.id}")

```

## <a name="monitor-the-run-status-by-email-notification"></a>Supervisión del estado de la ejecución mediante notificaciones por correo electrónico

1. En la barra de navegación de la izquierda de [Azure Portal](https://ms.portal.azure.com/), seleccione la pestaña **Supervisar**. 

1. Seleccione **Configuración de diagnóstico** y, luego, seleccione **+ Agregar configuración de diagnóstico**.

    ![Captura de pantalla de la configuración de diagnóstico para la notificación por correo electrónico](./media/how-to-track-monitor-analyze-runs/diagnostic-setting.png)

1. En la Configuración de diagnóstico, 
    1. en **Detalles de categoría**, seleccione **AmlRunStatusChangedEvent**. 
    1. En **Detalles de categoría**, seleccione **Enviar a área de trabajo de Log Analytics** y especifique la **suscripción** y el **área de trabajo de Log Analytics**. 

    > [!NOTE]
    > El **área de trabajo de Azure Log Analytics** es un tipo de recurso de Azure distinto del **área de trabajo de Azure Machine Learning Service**. Si no hay opciones en esa lista, puede [crear un área de trabajo de Log Analytics](../azure-monitor/logs/quick-create-workspace.md). 
    
    ![Dónde guardar la notificación por correo electrónico](./media/how-to-track-monitor-analyze-runs/log-location.png)

1. En la pestaña **Registros**, agregue una **Nueva regla de alertas**. 

    ![Nueva alerta de reglas](./media/how-to-track-monitor-analyze-runs/new-alert-rule.png)

1. Consulte [cómo crear y administrar las alertas de registro mediante Azure Monitor](../azure-monitor/alerts/alerts-log.md).

## <a name="example-notebooks"></a>Cuadernos de ejemplo

Los siguientes cuadernos muestran los conceptos de este artículo:

* Para obtener más información sobre las API de registro, vea el [cuaderno de API de registro](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api/logging-api.ipynb).

* Para obtener más información sobre cómo administrar ejecuciones con el SDK de Azure Machine Learning, consulte el [cuaderno de administración de ejecuciones](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/manage-runs/manage-runs.ipynb).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre cómo registrar las métricas de sus experimentos, consulte [Registrar métricas durante las ejecuciones de entrenamiento](how-to-log-view-metrics.md).
* Para aprender a supervisar los recursos y los registros desde Azure Machine Learning, consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).