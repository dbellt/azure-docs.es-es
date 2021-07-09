---
title: Administración y optimización de costos
titleSuffix: Azure Machine Learning
description: Obtenga sugerencias para optimizar el costo durante la creación de modelos de aprendizaje automático en Azure Machine Learning
author: luisquintanilla
ms.author: luquinta
ms.custom: subject-cost-optimization
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 06/08/2021
ms.openlocfilehash: f5f0351e21588d6e01a633a11d5638358e4d706b
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112008275"
---
# <a name="manage-and-optimize-azure-machine-learning-costs"></a>Administración y optimización de costos de Azure Machine Learning

Aprenda a administrar y optimizar los costos al entrenar e implementar modelos de aprendizaje automático para Azure Machine Learning.

Use las sugerencias siguientes para ayudarle a administrar y optimizar los costos de los recursos de proceso.

- Configurar los clústeres de entrenamiento para el escalado automático
- Establecer cuotas en las áreas de trabajo y la suscripción
- Establecer directivas de finalización en la ejecución del entrenamiento
- Usar máquinas virtuales de prioridad baja
- Usar una instancia de Azure Reserved Virtual Machine Instances
- Entrenamiento local
- Paralelización del entrenamiento
- Establecimiento de directivas de retención y eliminación de datos
- Implementación de recursos en la misma región

Para más información sobre el planeamiento y la supervisión de los costos, consulte la guía [Planificación para administrar costos de Azure Machine Learning](concept-plan-manage-cost.md).

## <a name="use-azure-machine-learning-compute-cluster-amlcompute"></a>Uso del clúster de proceso de Azure Machine Learning (AmlCompute)

Como los datos cambian constantemente, necesita entrenar y volver a entrenar modelos rápidos y simplificados para mantener su precisión. Sin embargo, el entrenamiento continuo conlleva un costo, especialmente en el caso de modelos de aprendizaje profundo en GPU. 

Los usuarios de Azure Machine Learning pueden usar el clúster de proceso de Azure Machine Learning administrado, también denominado AmlCompute. AmlCompute admite una variedad de opciones de GPU y CPU. Azure Machine Learning hospeda internamente AmlCompute en nombre de su suscripción. Proporciona los mismos niveles empresariales de seguridad, cumplimiento y gobernanza en la escala de nube de IaaS de Azure.

Dado que estos grupos de proceso se encuentran dentro de la infraestructura de IaaS de Azure, puede implementar, escalar y administrar el entrenamiento con los mismos requisitos de seguridad y cumplimiento que el resto de la infraestructura.  Estas implementaciones se producen en su suscripción y obedecen sus reglas de gobernanza. Más información sobre [proceso de Azure Machine Learning](how-to-create-attach-compute-cluster.md).

## <a name="configure-training-clusters-for-autoscaling"></a>Configuración de clústeres de entrenamiento para el escalado automático

El escalado automático de clústeres en función de los requisitos de la carga de trabajo ayuda a reducir los costos de manera que solo use lo que necesita.

Los clústeres de AmlCompute están diseñados para escalarse de forma dinámica en función de la carga de trabajo. El clúster se puede escalar verticalmente hasta el número máximo de nodos que configure. A medida que se complete cada ejecución, el clúster liberará los nodos y se escalará al número mínimo de nodos configurado.

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

También puede configurar la cantidad de tiempo que el nodo está inactivo antes de reducirse verticalmente. De forma predeterminada, el tiempo de inactividad antes de la reducción vertical se establece en 120 segundos.

+ Si realiza una experimentación menos iterativa, reduzca este tiempo para ahorrar costos.
+ Si realiza experimentación de desarrollo y pruebas de gran iteración, es posible que tenga que aumentar este tiempo para no pagar por el escalado y la reducción vertical constante después de cada cambio en el entorno o el script de entrenamiento.

Los clústeres de AmlCompute se pueden configurar para los cambios en los requisitos de carga de trabajo de Azure Portal, mediante la [clase SDK de AmlCompute](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute), [la CLI de AmlCompute](/cli/azure/ml/computetarget/create#az_ml_computetarget_create_amlcompute), con las [API REST](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/machinelearningservices/resource-manager/Microsoft.MachineLearningServices/stable).

```azurecli
az ml computetarget create amlcompute --name testcluster --vm-size Standard_NC6 --min-nodes 0 --max-nodes 5 --idle-seconds-before-scaledown 300
```

## <a name="set-quotas-on-resources"></a>Establecimiento de cuotas en recursos

AmlCompute incluye una [configuración de cuota (o límite)](how-to-manage-quotas.md#azure-machine-learning-compute). Esta cuota se establece para cada familia de máquinas virtuales (por ejemplo, serie Dv2, serie NCv3) y varía en función de la región de cada suscripción. Las suscripciones se inician con valores predeterminados pequeños para empezar a trabajar, pero esta opción le permite controlar la cantidad de recursos de Amlcompute disponibles para poner en marcha en su suscripción. 

Configure también la [cuota de nivel de área de trabajo por familia de máquinas virtuales](how-to-manage-quotas.md#workspace-level-quotas) para cada área de trabajo de una suscripción. Esto le permite tener un control más detallado sobre los costos que cada área de trabajo podría provocar potencialmente y restringir determinadas familias de VM. 

Para establecer cuotas en el nivel de área de trabajo, empiece en [Azure Portal](https://portal.azure.com).  Seleccione cualquier área de trabajo de la suscripción y seleccione **Usos y cuotas** en el panel izquierdo. A continuación, seleccione la pestaña **Configurar cuotas** para ver las cuotas. Necesita privilegios en el ámbito de la suscripción para establecer esta cuota, ya que se trata de una configuración que afecta a varias áreas de trabajo.

## <a name="set-run-autotermination-policies"></a>Establecimiento de directivas de finalización automática de ejecución 

En algunos casos, debe configurar las ejecuciones de entrenamiento para limitar su duración o terminarlas de forma temprana. Por ejemplo, al usar el aprendizaje automático automatizado o el ajuste de hiperparámetros integrado de Azure Machine Learning.

Estas son algunas de las opciones que tiene:
* Defina un parámetro llamado `max_run_duration_seconds` en RunConfiguration para controlar la duración máxima a la que se puede extender una ejecución en el proceso que elija (proceso local o en la nube remota).
* Para el [ajuste de hiperparámetros](how-to-tune-hyperparameters.md#early-termination), defina una directiva de finalización temprana de una directiva de bandidos, una directiva de mediana de detención o una directiva de selección de truncamiento. Para controlar aún más barridos de hiperparámetros, utilice parámetros como `max_total_runs` o `max_duration_minutes`.
* En el caso del [aprendizaje automático automatizado](how-to-configure-auto-train.md#exit), establezca directivas de terminación similares mediante la marca `enable_early_stopping`. Use también propiedades como `iteration_timeout_minutes` y `experiment_timeout_minutes` para controlar la duración máxima de una ejecución o para todo el experimento.

## <a name="use-low-priority-vms"></a><a id="low-pri-vm"></a> Uso de máquinas virtuales de prioridad baja

Azure le permite usar un exceso de capacidad no utilizada como máquinas virtuales de prioridad baja en conjuntos de escalado de máquinas virtuales, Batch y el servicio de Machine Learning. Estas asignaciones son reemplazables pero tienen un precio reducido en comparación con las máquinas virtuales dedicadas. En general, se recomienda usar VM de prioridad baja para cargas de trabajo de Batch. También debe usarlas cuando las interrupciones se puedan recuperar a través de reenvíos (para la inferencia de lotes) o a través de reinicios (para un entrenamiento de aprendizaje profundo con puntos de comprobación).

Las máquinas virtuales de prioridad baja tienen una cuota única independiente del valor de cuota dedicada, que es por familia de máquinas virtuales. Obtenga [más información sobre las cuotas de AmlCompute](how-to-manage-quotas.md).

 Las máquinas virtuales de prioridad baja no funcionan para las instancias de proceso, ya que necesitan admitir experiencias interactivas de cuaderno.

## <a name="use-reserved-instances"></a>Uso de instancias reservadas

Otra manera de ahorrar dinero en recursos de proceso es la instancia reservada de máquina virtual de Azure. Con esta oferta, se compromete con plazos de uno o tres años. Estos descuentos van hasta el 72 % de los precios de pago por uso y se aplican directamente a su factura mensual de Azure.

Proceso de Azure Machine Learning admite las instancias reservadas de forma inherente. Si compra una instancia reservada de uno o tres años, se aplicará automáticamente el descuento en el proceso administrado de Azure Machine Learning.

## <a name="train-locally"></a>Entrenamiento local

Al crear prototipos y ejecutar trabajos de entrenamiento lo suficientemente pequeños como para ejecutarse en el equipo local, considere la posibilidad de realizar el entrenamiento localmente. Con el SDK de Python, al establecer el destino de proceso `local` se ejecutará el script localmente. Para más información, consulte [Configuración y envío de ejecuciones de entrenamiento](how-to-set-up-training-targets.md#select-a-compute-target).

Visual Studio Code proporciona un entorno completo para desarrollar aplicaciones de aprendizaje automático. Mediante la extensión de Visual Studio Code para Azure Machine Learning y Docker, puede ejecutar y depurar localmente. Para más información, consulte [Depuración interactiva con Visual Studio Code](how-to-debug-visual-studio-code.md).

## <a name="parallelize-training"></a>Paralelización del entrenamiento

Uno de los métodos clave para optimizar el costo y el rendimiento es paralelizar la carga de trabajo con la ayuda de un paso de ejecución en paralelo en Azure Machine Learning. Este paso le permite usar muchos nodos más pequeños para ejecutar la tarea en paralelo, lo que le permite escalar horizontalmente. Hay una sobrecarga en la paralelización. En función de la carga de trabajo y del grado de paralelismo que se pueda lograr, esta puede ser o no una opción. Para más información, consulte la documentación de [ParallelRunStep](xref:azureml.contrib.pipeline.steps.ParallelRunStep).

## <a name="set-data-retention--deletion-policies"></a>Establecimiento de directivas de retención y eliminación de datos

Cada vez que se ejecuta una canalización, se generan conjuntos de datos intermedios en cada paso. Con el tiempo, estos conjuntos de datos intermedios ocupan espacio en la cuenta de almacenamiento. Considere la posibilidad de configurar directivas para administrar los datos a lo largo de su ciclo de vida para archivar y eliminar los conjuntos de datos. Para más información, consulte [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](/storage/blobs/storage-lifecycle-management-concepts.md).

## <a name="deploy-resources-to-the-same-region"></a>Implementación de recursos en la misma región

Los procesos ubicados en regiones diferentes pueden experimentar latencia de red y mayores costos de transferencia de datos. Los costos de red de Azure se incurren en el ancho de banda saliente de los centros de datos de Azure. Para ayudar a reducir los costos de red, implemente todos los recursos de la región. El aprovisionamiento del área de trabajo de Azure Machine Learning y de los recursos dependientes en la misma región que los datos puede ayudar a reducir el costo y mejorar el rendimiento.

En escenarios de nube híbrida como los que usan ExpressRoute, a veces puede ser más rentable mover todos los recursos a Azure para optimizar los costos de red y la latencia.

## <a name="next-steps"></a>Pasos siguientes

- [Planificación para administrar costos de Azure Machine Learning](concept-plan-manage-cost.md)
- [Administración de presupuestos, costos y cuota de Azure Machine Learning a escala organizativa](/azure/cloud-adoption-framework/ready/azure-best-practices/optimize-ai-machine-learning-cost)