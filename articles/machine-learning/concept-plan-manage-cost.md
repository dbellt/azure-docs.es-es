---
title: Planificación para administrar costos
titleSuffix: Azure Machine Learning
description: Planeación y administración de los costos de Azure Machine Learning con análisis de costos en Azure Portal. Conozca las sugerencias de ahorro en los costos al crear modelos de aprendizaje automático.
author: sdgilley
ms.author: sgilley
ms.custom: subject-cost-optimization, devx-track-azurecli
ms.reviewer: nigup
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 05/07/2021
ms.openlocfilehash: 0ad5c432cb9ba5742ae4fa2ee2e41dc2ff2b5936
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713535"
---
# <a name="plan-to-manage-costs-for-azure-machine-learning"></a>Planificación para administrar costos de Azure Machine Learning

En este artículo se describe cómo puede planear y administrar los costos de Azure Machine Learning. En primer lugar, use la calculadora de precios de Azure para ayudar a planear los costos antes de agregar recursos. Después, a medida que agregue los recursos de Azure, revise los costos estimados. Por último, use sugerencias de ahorro de costos a medida que entrena el modelo con clústeres de proceso de Azure Machine Learning administrados.

Después de comenzar a usar los recursos de Azure Machine Learning, use las características de administración de costos para establecer presupuestos y supervisar los costos. Revise los costos previstos e identifique las tendencias de gasto para determinar las áreas interesantes para actuar.

Tenga en cuenta que los costos de Azure Machine Learning son solo una parte de los costos mensuales de la factura de Azure. Si usa otros servicios de Azure, se le facturarán todos los servicios y recursos de Azure usados en la suscripción de Azure, incluidos los servicios de terceros. En este artículo se explica cómo planear y administrar los costos de Azure Machine Learning. Una vez que esté familiarizado con la administración de los costos de Azure Machine Learning, puede aplicar métodos similares para administrar los costos de todos los servicios de Azure que se usan en la suscripción.

Al entrenar los modelos de aprendizaje automático, use clústeres de proceso de Azure Machine Learning administrados para aprovechar las ventajas de las sugerencias para el ahorro de costos:

* Configurar los clústeres de entrenamiento para el escalado automático
* Establecer cuotas en las áreas de trabajo y la suscripción
* Establecer directivas de finalización en la ejecución del entrenamiento
* Usar máquinas virtuales de prioridad baja
* Usar una instancia de Azure Reserved Virtual Machine Instances

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). 

Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-azure-machine-learning"></a>Cálculo de costos antes de usar Azure Machine Learning

- Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de crear los recursos en un área de trabajo de Azure Machine Learning.
A la izquierda, seleccione **IA y Machine Learning** y, a continuación, seleccione **Azure Machine Learning** para comenzar.  

En la captura de pantalla siguiente se muestra la estimación de costos mediante la calculadora:

:::image type="content" source="media/concept-plan-manage-cost/capacity-calculator-cost-estimate.png" alt-text="Ejemplo que muestra el costo estimado en la calculadora de precios de Azure.":::

A medida que agregue nuevos recursos al área de trabajo, vuelva a esta calculadora y agregue el mismo recurso aquí para actualizar las estimaciones de costos.

Para más información, consulte [Precios de Azure Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="understand-the-full-billing-model-for-azure-machine-learning"></a>Conozca todo el modelo de facturación de Azure Machine Learning

Azure Machine Learning se ejecuta en una infraestructura de Azure que genera otros costos, además de los de Azure Machine Learning, cuando se implementa el nuevo recurso. Es importante que comprenda que hay otras infraestructuras que pueden generar costos. Estos costos deben administrarse cuando se realizan cambios en los recursos implementados. 






### <a name="costs-that-typically-accrue-with-azure-machine-learning"></a>Costos que suelen generarse con Azure Machine Learning

Cuando se crean recursos para un área de trabajo Azure Machine Learning, también se crean recursos para otros servicios de Azure. Son las siguientes:

* Cuenta básica de [Azure Container Registry](https://azure.microsoft.com/pricing/details/container-registry?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Cuenta de almacenamiento de blobs en bloques de Azure](https://azure.microsoft.com/pricing/details/storage/blobs?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (uso general v1)
* [Key Vault](https://azure.microsoft.com/pricing/details/key-vault?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Application Insights](https://azure.microsoft.com/en-us/pricing/details/monitor?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
 
### <a name="costs-might-accrue-after-resource-deletion"></a>Costos que pueden generarse tras eliminar un recurso

Después de eliminar un área de trabajo de Azure Machine Learning en Azure Portal o con la CLI de Azure, se mantienen los recursos que se indican a continuación. Estos recursos siguen generando costos hasta que se eliminan.

* Azure Container Registry
* Almacenamiento de blobs en bloque de Azure
* Key Vault
* Application Insights

Para eliminar el área de trabajo junto con estos recursos dependientes, use el SDK:

```python
ws.delete(delete_dependent_resources=True)
```

Si crea Azure Kubernetes Service (AKS) en el área de trabajo, o si asocia recursos de proceso al área de trabajo, deberá eliminarlos por separado en [Azure Portal](https://portal.azure.com).

### <a name="using-azure-prepayment-credit-with-azure-machine-learning"></a>Uso del crédito del pago por adelantado de Azure con Azure Machine Learning

Puede pagar los cargos de Azure Machine Learning con el crédito del pago por adelantado de Azure. Sin embargo, no puede usar los créditos del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los de Azure Marketplace.

## <a name="review-estimated-costs-in-the-azure-portal"></a>Revisión de los costos estimados en Azure Portal

<!-- Note for Azure service writer: If your service shows estimated costs when a user is creating resources in the Azure portal, at a minimum, insert this section as a brief walkthrough that steps through creating a Azure Machine Learning resource where the estimated cost is shown to the user, updated for your service. Add a screenshot where the estimated costs or subscription credits are shown.

If your service doesn't show costs as they create a resource or if estimated costs aren't shown to users before they use your service, then omit this section.

For example, you might start with the following (modify for your service):
-->

A medida que cree recursos de proceso para Azure Machine Learning, verá los costos estimados.

Para crear una *instancia de proceso * y ver el precio estimado, haga lo siguiente:

1. Inicie sesión en el [Estudio de Azure Machine Learning](https://ml.azure.com).
1. Seleccione **Proceso** en el lado izquierdo.
1. En la barra de herramientas superior, seleccione **+ Nuevo**.
1. Revise el precio estimado que se muestra para cada tamaño de máquina virtual disponible.
1. Termine de crear el recurso.


:::image type="content" source="media/concept-plan-manage-cost/create-resource.png" alt-text="Ejemplo que muestra los costos estimados durante la creación de una instancia de proceso." lightbox="media/concept-plan-manage-cost/create-resource.png" :::

Si la suscripción de Azure tiene un límite de gasto, Azure le impide gastar por encima del importe del crédito. A medida que crea y usa recursos de Azure, se usan los créditos. Cuando alcanza el límite de crédito, los recursos que ha implementado se deshabilitan para el resto de ese período de facturación. No se puede cambiar el límite de crédito, pero sí puede quitarlo. Para más información sobre los límites de gasto, consulte [Límite de gasto de Azure](../cost-management-billing/manage/spending-limit.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos de Azure con Azure Machine Learning, se generan costos. Los costos de unidad del uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto se comienza a usar Azure Machine Learning, se generan costos, que puede ver en el [análisis de costos](../cost-management/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Al usar el análisis de costos, verá los costos de Azure Machine Learning en gráficos y tablas para diferentes intervalos de tiempo. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de Azure Machine Learning en el análisis de costos, haga lo siguiente:

1. Inicie sesión en Azure Portal.
2. Abra el ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos.
3. De forma predeterminada, el costo de los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta "Azure Machine Learning".

Los costos mensuales reales se muestran cuando se abre inicialmente el análisis de costos. Este es un ejemplo con todos los costos mensuales de uso.

:::image type="content" source="media/concept-plan-manage-cost/all-costs.png" alt-text="Ejemplo que muestra los costos acumulados de una suscripción." lightbox="media/concept-plan-manage-cost/all-costs.png" :::


Para limitar la información a los costos de un único servicio, como Azure Machine Learning, seleccione **Agregar filtro** y, luego, elija **Nombre del servicio**. A continuación, seleccione **Azure Machine Learning**.

Este es un ejemplo que muestra solo los costos de Azure Machine Learning.

:::image type="content" source="media/concept-plan-manage-cost/vm-specific-cost.png" alt-text="Ejemplo que muestra los costos acumulados de ServiceName." lightbox="media/concept-plan-manage-cost/vm-specific-cost.png" :::

<!-- Note to Azure service writer: The image shows an example for Azure Storage. Replace the example image with one that shows costs for your service. -->

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los costos de Azure Machine Learning por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.
## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando usted u otro usuario necesita hacer un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="other-ways-to-manage-and-reduce-costs-for-azure-machine-learning"></a>Otras formas de administrar y reducir los costos de Azure Machine Learning

Utilice estas sugerencias para contener los gastos de los recursos de proceso de aprendizaje automático.

### <a name="use-azure-machine-learning-compute-cluster-amlcompute"></a>Uso del clúster de proceso de Azure Machine Learning (AmlCompute)

Como los datos cambian constantemente, necesita entrenar y volver a entrenar modelos rápidos y simplificados para mantener su precisión. Sin embargo, el entrenamiento continuo conlleva un costo, especialmente en el caso de modelos de aprendizaje profundo en GPU. 

Los usuarios de Azure Machine Learning pueden usar el clúster de proceso de Azure Machine Learning administrado, también denominado AmlCompute. AmlCompute admite una variedad de opciones de GPU y CPU. Azure Machine Learning hospeda internamente AmlCompute en nombre de su suscripción. Proporciona los mismos niveles empresariales de seguridad, cumplimiento y gobernanza en la escala de nube de IaaS de Azure.

Dado que estos grupos de proceso se encuentran dentro de la infraestructura de IaaS de Azure, puede implementar, escalar y administrar el entrenamiento con los mismos requisitos de seguridad y cumplimiento que el resto de la infraestructura.  Estas implementaciones se producen en su suscripción y obedecen sus reglas de gobernanza. Más información sobre [proceso de Azure Machine Learning](how-to-create-attach-compute-cluster.md).

### <a name="configure-training-clusters-for-autoscaling"></a>Configuración de clústeres de entrenamiento para el escalado automático

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

### <a name="set-quotas-on-resources"></a>Establecimiento de cuotas en recursos

AmlCompute incluye una [configuración de cuota (o límite)](how-to-manage-quotas.md#azure-machine-learning-compute). Esta cuota se establece para cada familia de máquinas virtuales (por ejemplo, serie Dv2, serie NCv3) y varía en función de la región de cada suscripción. Las suscripciones se inician con valores predeterminados pequeños para empezar a trabajar, pero esta opción le permite controlar la cantidad de recursos de Amlcompute disponibles para poner en marcha en su suscripción. 

Configure también la [cuota de nivel de área de trabajo por familia de máquinas virtuales](how-to-manage-quotas.md#workspace-level-quotas) para cada área de trabajo de una suscripción. Esto le permite tener un control más detallado sobre los costos que cada área de trabajo podría provocar potencialmente y restringir determinadas familias de VM. 

Para establecer cuotas en el nivel de área de trabajo, empiece en [Azure Portal](https://portal.azure.com).  Seleccione cualquier área de trabajo de la suscripción y seleccione **Usos y cuotas** en el panel izquierdo. A continuación, seleccione la pestaña **Configurar cuotas** para ver las cuotas. Necesita privilegios en el ámbito de la suscripción para establecer esta cuota, ya que se trata de una configuración que afecta a varias áreas de trabajo.

### <a name="set-run-autotermination-policies"></a>Establecimiento de directivas de finalización automática de ejecución 

En algunos casos, debe configurar las ejecuciones de entrenamiento para limitar su duración o terminarlas de forma temprana. Por ejemplo, al usar el aprendizaje automático automatizado o el ajuste de hiperparámetros integrado de Azure Machine Learning.

Estas son algunas de las opciones que tiene:
* Defina un parámetro llamado `max_run_duration_seconds` en RunConfiguration para controlar la duración máxima a la que se puede extender una ejecución en el proceso que elija (proceso local o en la nube remota).
* Para el [ajuste de hiperparámetros](how-to-tune-hyperparameters.md#early-termination), defina una directiva de finalización temprana de una directiva de bandidos, una directiva de mediana de detención o una directiva de selección de truncamiento. Para controlar aún más barridos de hiperparámetros, utilice parámetros como `max_total_runs` o `max_duration_minutes`.
* En el caso del [aprendizaje automático automatizado](how-to-configure-auto-train.md#exit), establezca directivas de terminación similares mediante la marca `enable_early_stopping`. Use también propiedades como `iteration_timeout_minutes` y `experiment_timeout_minutes` para controlar la duración máxima de una ejecución o para todo el experimento.

### <a name="use-low-priority-vms"></a><a id="low-pri-vm"></a> Uso de máquinas virtuales de prioridad baja

Azure le permite usar un exceso de capacidad no utilizada como máquinas virtuales de prioridad baja en conjuntos de escalado de máquinas virtuales, Batch y el servicio de Machine Learning. Estas asignaciones son reemplazables pero tienen un precio reducido en comparación con las máquinas virtuales dedicadas. En general, se recomienda usar VM de prioridad baja para cargas de trabajo de Batch. También debe usarlas cuando las interrupciones se puedan recuperar a través de reenvíos (para la inferencia de lotes) o a través de reinicios (para un entrenamiento de aprendizaje profundo con puntos de comprobación).

Las máquinas virtuales de prioridad baja tienen una cuota única independiente del valor de cuota dedicada, que es por familia de máquinas virtuales. Obtenga [más información sobre las cuotas de AmlCompute](how-to-manage-quotas.md).

 Las máquinas virtuales de prioridad baja no funcionan para las instancias de proceso, ya que necesitan admitir experiencias interactivas de cuaderno.

### <a name="use-reserved-instances"></a>Uso de instancias reservadas

Otra manera de ahorrar dinero en recursos de proceso es la instancia reservada de máquina virtual de Azure. Con esta oferta, se compromete con plazos de uno o tres años. Estos descuentos van hasta el 72 % de los precios de pago por uso y se aplican directamente a su factura mensual de Azure.

Proceso de Azure Machine Learning admite las instancias reservadas de forma inherente. Si compra una instancia reservada de uno o tres años, se aplicará automáticamente el descuento en el proceso administrado de Azure Machine Learning.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
