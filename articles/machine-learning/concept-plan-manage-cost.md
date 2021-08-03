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
ms.date: 06/08/2021
ms.openlocfilehash: 01c985b0554fe5955010c1c8c286f81f8de6d3ee
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006012"
---
# <a name="plan-to-manage-costs-for-azure-machine-learning"></a>Planificación para administrar costos de Azure Machine Learning

En este artículo se describe cómo puede planear y administrar los costos de Azure Machine Learning. En primer lugar, use la calculadora de precios de Azure para ayudar a planear los costos antes de agregar recursos. Después, a medida que agregue los recursos de Azure, revise los costos estimados. 

Después de comenzar a usar los recursos de Azure Machine Learning, use las características de administración de costos para establecer presupuestos y supervisar los costos. Revise los costos previstos e identifique las tendencias de gasto para determinar las áreas interesantes para actuar.

Tenga en cuenta que los costos de Azure Machine Learning son solo una parte de los costos mensuales de la factura de Azure. Si usa otros servicios de Azure, se le facturarán todos los servicios y recursos de Azure usados en la suscripción de Azure, incluidos los servicios de terceros. En este artículo se explica cómo planear y administrar los costos de Azure Machine Learning. Una vez que esté familiarizado con la administración de los costos de Azure Machine Learning, puede aplicar métodos similares para administrar los costos de todos los servicios de Azure que se usan en la suscripción.

Para obtener más información sobre cómo optimizar los costos, consulte [cómo administrar y optimizar los costos en Azure Machine Learning](how-to-manage-optimize-cost.md).

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). 

Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

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

A medida que se usan recursos de Azure con Azure Machine Learning, se generan costos. Los costos de unidad del uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto se comienza a usar Azure Machine Learning, se generan costos, que puede ver en el [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

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

Para obtener más información, consulte [Administración y optimización de los costos en Azure Machine Learning](how-to-manage-optimize-cost.md).

## <a name="next-steps"></a>Pasos siguientes

- [Administre y optimice los costos en Azure Machine Learning](how-to-manage-optimize-cost.md).
- [Administración de presupuestos, costos y cuota de Azure Machine Learning a escala organizativa](/azure/cloud-adoption-framework/ready/azure-best-practices/optimize-ai-machine-learning-cost)
- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).