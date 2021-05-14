---
title: Desarrollo con AutoML y Azure Databricks
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo configurar un entorno de desarrollo en Azure Machine Learning y Azure Databricks. Use los SDK de Azure ML para Databricks y Databricks con AutoML.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.date: 10/21/2020
ms.topic: how-to
ms.custom: devx-track-python
ms.openlocfilehash: 960cbbac9a1fc8c3a87296ddcb0711564417c647
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107885909"
---
# <a name="set-up-a-development-environment-with-azure-databricks-and-automl-in-azure-machine-learning"></a>Configuración de un entorno de desarrollo con Azure Databricks y AutoML en Azure Machine Learning 

Aprenda a configurar un entorno de desarrollo en Azure Machine Learning que usa Azure Databricks y ML automatizado.

Azure Databricks es idóneo para ejecutar flujos de trabajo de aprendizaje automático intensivos y a gran escala en la plataforma escalable de Apache Spark en la nube de Azure. Proporciona un entorno de colaboración basado en cuadernos con un clúster de proceso basado en CPU o GPU.

Para obtener información sobre otros entornos de desarrollo de aprendizaje automático, consulte [Configuración de un entorno de desarrollo para Azure Machine Learning](how-to-configure-environment.md).


## <a name="prerequisite"></a>Requisito previo

Área de trabajo de Azure Machine Learning. Si no tiene ninguna, puede crear un área de trabajo de Azure Machine Learning a través de [Azure Portal](how-to-manage-workspace.md), la [CLI de Azure](how-to-manage-workspace-cli.md#create-a-workspace) y las [plantillas de Azure Resource Manager](how-to-create-workspace-template.md).


## <a name="azure-databricks-with-azure-machine-learning-and-automl"></a>Azure Databricks con Azure Machine Learning y AutoML

Azure Databricks se integra con Azure Machine Learning y sus funcionalidades de AutoML. 

Puede usar Azure Databricks:

+ Para entrenar un modelo con Spark MLlib e implementar el modelo en ACI o AKS.
+ Con funcionalidades de [aprendizaje automático automatizado](concept-automated-ml.md) mediante un SDK de Azure ML.
+ Como destino de proceso desde una [canalización de Azure Machine Learning](concept-ml-pipelines.md).

## <a name="set-up-a-databricks-cluster"></a>Configuración de un clúster de Databricks

Cree un [clúster de Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Algunas opciones solo se aplican si instala el SDK para el aprendizaje automático automatizado en Databricks.

**Se tardan unos minutos en crear el clúster.**

Use estos valores de configuración:

| Configuración |Se aplica a| Value |
|----|---|---|
| Cluster Name |Siempre| nombredelclúster |
| Versión de Databricks Runtime |Siempre| Runtime 7.1 (scala 2.21, spark 3.0.0): no ML|
| Versión de Python |Siempre| 3 |
| Tipo de trabajo <br>(determina el número máximo de iteraciones simultáneas) |Machine Learning Automatizado<br>solo| Se prefieren las máquinas virtuales optimizadas para memoria |
| Trabajos |Siempre| 2 o más |
| Habilitar escalado automático |Machine Learning Automatizado<br>solo| Desactivar |

Espere hasta que el clúster se ejecute antes de continuar.

## <a name="add-the-azure-ml-sdk-to-databricks"></a>Incorporación del SDK de Azure ML a Databricks

Una vez que se esté ejecutando el clúster, [cree una biblioteca](https://docs.databricks.com/user-guide/libraries.html#create-a-library) para adjuntar al clúster el paquete del SDK adecuado de Azure Machine Learning. 

Para usar ML automatizado, vaya a [Incorporación del SDK de Azure ML con ML automatizado](#add-the-azure-ml-sdk-with-automl-to-databricks).


1. Haga clic con el botón derecho en la carpeta del área de trabajo actual en la que quiere almacenar la biblioteca. Seleccione **Crear** > **Biblioteca**.
    
    > [!TIP]
    > Si tiene una versión anterior del SDK, anule la selección de este desde las bibliotecas instaladas del clúster y muévalo a la Papelera. Instale la nueva versión del SDK y reinicie el clúster. Si hay un problema después del reinicio, desasocie y asocie nuevamente el clúster.

1. Elija la siguiente opción (no se admite ninguna otra instalación de SDK).

   |Elementos adicionales del&nbsp;paquete&nbsp;del SDK|Source|Nombre de&nbsp;PyPi&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
   |----|---|---|
   |Para Databricks| Cargar un huevo o PyPi de Python | azureml-sdk[databricks]|

   > [!WARNING]
   > No se pueden instalar otros elementos adicionales del SDK. Elija solo la opción [`databricks`].

   * No seleccione **Attach automatically to all clusters** (Asociar automáticamente a todos los clústeres).
   * Seleccione **Asociar** junto al nombre del clúster.

1. Supervise los errores hasta que el estado cambie a **Asociado**, lo que podría tardar varios minutos.  Si se produce un error en este paso:

   Pruebe a reiniciar el clúster de la manera siguiente:
   1. En el panel izquierdo, seleccione **Clústeres**.
   1. En la tabla, seleccione el nombre del clúster.
   1. En la ficha **Bibliotecas**, seleccione **Reiniciar**.

   Una instalación correcta tiene un aspecto similar al siguiente: 

  ![SDK de Azure Machine Learning para Databricks](./media/how-to-configure-environment/amlsdk-withoutautoml.jpg) 

## <a name="add-the-azure-ml-sdk-with-automl-to-databricks"></a>Incorporación del SDK de Azure ML con AutoML a Databricks
Si el clúster se creó con Databricks Runtime 7.1 o superior (*no* ML), ejecute el siguiente comando en la primera celda del cuaderno para instalar el SDK de AML.

```
%pip install --upgrade --force-reinstall -r https://aka.ms/automl_linux_requirements.txt
```
Para Databricks Runtime 7.0 o anterior, instale el SDK de Azure Machine Learning mediante el [script init](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks/automl/README.md).

### <a name="automl-config-settings"></a>Opciones de configuración de AutoML

En la configuración de AutoML, al usar Azure Databricks, agregue los siguientes parámetros:

- ```max_concurrent_iterations``` se basa en el número de nodos de trabajo del clúster.
- ```spark_context=sc``` se basa en el contexto de Spark predeterminado.

## <a name="ml-notebooks-that-work-with-azure-databricks"></a>Cuadernos de ML que funcionan con Azure Databricks

Pruebe lo siguiente:
+ Aunque hay muchos cuadernos de ejemplo disponibles, **solo [estos cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks) funcionan con Azure Databricks.**

+ Importe estos ejemplos directamente desde el área de trabajo. Observe a continuación: ![Selección de Importar](./media/how-to-configure-environment/azure-db-screenshot.png)
![Panel Importar](./media/how-to-configure-environment/azure-db-import.png)

+ Obtenga información sobre cómo [crear una canalización con Databricks como proceso de entrenamiento](./how-to-create-machine-learning-pipelines.md).

## <a name="troubleshooting"></a>Solución de problemas

* **Cancelación de Databricks de una ejecución de aprendizaje automático automatizado**: Al usar las funcionalidades de aprendizaje automático automatizado en Azure Databricks, para cancelar una ejecución e iniciar una nueva ejecución de un experimento, reinicie el clúster de Azure Databricks.

* **> 10 iteraciones de Databricks para aprendizaje automático automatizado**: En la configuración del aprendizaje automático automatizado, si tiene más de 10 iteraciones, establezca `show_output` en `False` cuando envíe la ejecución.

* **Widget de Databricks para el SDK de Azure Machine Learning y aprendizaje automático automatizado**: El widget del SDK de Azure Machine Learning no se admite en un cuaderno de Databricks porque los cuadernos no pueden analizar los widgets HTML. Para ver el widget en el portal, use este código de Python en la celda del cuaderno de Azure Databricks:

    ```
    displayHTML("<a href={} target='_blank'>Azure Portal: {}</a>".format(local_run.get_portal_url(), local_run.id))
    ```

* **Error al instalar paquetes**

    No es posible instalar el SDK de Azure Machine Learning en Azure Databricks cuando se instalan más paquetes. Algunos paquetes, como `psutil`, pueden provocar conflictos. Para evitar errores de instalación, inmovilice la versión de la biblioteca para instalar los paquetes. Este problema está relacionado con Databricks y no con el SDK de Azure Machine Learning. También puede experimentar este problema con otras bibliotecas. Ejemplo:
    
    ```python
    psutil cryptography==1.5 pyopenssl==16.0.0 ipython==2.2.0
    ```

    Como alternativa, puede usar scripts de init si sigue experimentando problemas de instalación con las bibliotecas de Python. Este enfoque no se admite oficialmente. Para más información, consulte [Cluster-scoped init scripts](/azure/databricks/clusters/init-scripts#cluster-scoped-init-scripts) (Script de init del ámbito de clúster).

* **Error de importación: no se puede importar el nombre `Timedelta` de `pandas._libs.tslibs`** : Si ve este error al usar el aprendizaje automático automatizado, ejecute las dos líneas siguientes en el cuaderno:
    ```
    %sh rm -rf /databricks/python/lib/python3.7/site-packages/pandas-0.23.4.dist-info /databricks/python/lib/python3.7/site-packages/pandas
    %sh /databricks/python/bin/pip install pandas==0.23.4
    ```

* **Error de importación: No hay ningún módulo denominado "pandas.core.indexes"** : Si ve este error al usar el aprendizaje automático automatizado:

    1. Ejecute este comando para instalar dos paquetes en el clúster de Azure Databricks:
    
       ```bash
       scikit-learn==0.19.1
       pandas==0.22.0
       ```
    
    1. Desasocie y, luego, vuelva a conectar el clúster al cuaderno.
    
    Si estos pasos no resuelven el problema, pruebe a reiniciar el clúster.

* **FailToSendFeather**: Si ve un error `FailToSendFeather` al leer datos en un clúster de Azure Databricks, consulte las soluciones siguientes:
    
    * Actualice el paquete `azureml-sdk[automl]` a la versión más reciente.
    * Agregue `azureml-dataprep` versión 1.1.8 o superior.
    * Agregue `pyarrow` versión 0.11 o superior.
  

## <a name="next-steps"></a>Pasos siguientes

- [Entrenar un modelo](tutorial-train-models-with-aml.md) en Azure Machine Learning con el conjunto de datos de MNIST.
- Consultar la [referencia del SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).