---
title: Planeamiento de la administración de costos de Azure Synapse Analytics
description: Aprenda a planear y administrar los costos de Azure Synapse Analytics mediante el análisis de costos de Azure Portal.
author: julieMSFT
ms.author: jrasnick
ms.custom: subject-cost-optimization
ms.service: synapse-analytics
ms.subservice: overview
ms.topic: how-to
ms.date: 06/08/2021
ms.openlocfilehash: 7a5b9efce1c24aecb54d3f473b7227357c8fb99f
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111890193"
---
# <a name="plan-and-manage-costs-for-azure-synapse-analytics"></a>Planeamiento y administración de costos de Azure Synapse Analytics

En este artículo se describe cómo planear y administrar los costos de Azure Synapse Analytics. Antes de agregar recursos en Azure Synapse Analytics, use la calculadora de precios de Azure para estimar y planear los costos del servicio. Después, a medida que agregue recursos de Azure, revise los costos estimados. Después de comenzar a usar los recursos de Azure Synapse Analytics, utilice las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para identificar las áreas en las que podría querer actuar. Los costos de Azure Synapse Analytics son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de Azure Synapse Analytics, se le facturarán todos los servicios y recursos de Azure que use en su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-azure-synapse-analytics"></a>Cálculo de costos antes de usar Azure Synapse Analytics

Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de agregar Azure Synapse Analytics.

Azure Synapse incluye varios recursos que tienen distintos cargos, tal como se muestra en la estimación de costos que aparece a continuación. 

![Ejemplo que muestra el costo estimado en la calculadora de precios de Azure](./media/plan-manage-costs/cost-estimate.png)

## <a name="understand-the-full-billing-model-for-azure-synapse-analytics"></a>Modelo de facturación completo de Azure Synapse Analytics

Azure Synapse se ejecuta en una infraestructura de Azure que genera otros costos, además de los de Azure Synapse, cuando se implementa el nuevo recurso. Es importante que comprenda que hay otras infraestructuras que pueden generar costos. 

### <a name="how-youre-charged-for-azure-synapse-analytics"></a>¿Cómo se le cobra por Azure Synapse Analytics?

Al crear o usar recursos de Azure Synapse Analytics, es posible que se le cobre por los siguientes medidores:

- Exploración de datos y almacenamiento de datos 
    - Grupo de SQL dedicado: se le cobra según el número de bloques de DWU y las horas de ejecución.
    - Almacenamiento: se le cobra según el número de TB almacenados.
    - Grupo de SQL sin servidor: se le cobra según los TB de datos procesados.
- Grupo de Apache Spark: se le cobra según el número de instancias y las horas de ejecución.
- Integración de datos 
    - Ejecuciones de actividad de orquestación: se le cobra según el número de ejecuciones de actividad.
    - Movimiento de datos: para las actividades de copia que se ejecutan en Azure Integration Runtime, se le cobra según el número de DIU usadas y la duración de la ejecución.
    - Horas de núcleo virtual para flujos de datos: para la ejecución y depuración del flujo de datos, se le cobra según el tipo de proceso, el número de núcleos virtuales y la duración de la ejecución.

Al final del ciclo de facturación, se suman los cargos de cada medidor. La factura muestra una sección para todos los costos de Azure Synapse Analytics. Hay un elemento de línea independiente para cada medidor.

### <a name="other-costs-that-might-accrue-with-azure-synapse-analytics"></a>Otros costos que pueden generarse con Azure Synapse Analytics

Cuando se crean recursos para Azure Synapse, también se crean recursos para otros servicios de Azure. Incluyen:

- Data Lake Storage Gen2

 ### <a name="costs-might-accrue-after-resource-deletion"></a>Costos que pueden generarse tras eliminar un recurso

Después de eliminar los recursos de Azure Synapse, es posible que los siguientes recursos sigan existiendo. Estos recursos siguen generando costos hasta que se eliminan.

- Data Lake Storage Gen2

### <a name="using-azure-prepayment-credit-with-azure-synapse"></a>Uso del crédito del pago por adelantado de Azure con Azure Synapse 

Puede pagar los cargos de Azure Synapse con el crédito del pago por adelantado de Azure. Sin embargo, no puede usar el crédito del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los que proceden de Azure Marketplace.


## <a name="review-estimated-costs-in-the-azure-portal"></a>Revisión de los costos estimados en Azure Portal

A medida que cree recursos para Azure Synapse Analytics, verá los costos estimados. Con un área de trabajo se crea un grupo de SQL sin servidor. Este grupo de SQL sin servidor no incurrirá en cargos hasta que ejecute consultas. Además, deberán crearse otros recursos, como los grupos de SQL dedicados y los grupos de Apache Spark sin servidor, en el área de trabajo.

Para crear un área de trabajo de Azure Synapse Analytics y ver el precio estimado:

1. Navegue hasta el servicio en Azure Portal.
2. Cree el recurso.
3. Revise el precio estimado que se muestra en el resumen.
4. Termine de crear el recurso.

![Ejemplo que muestra los costos estimados durante la creación de un recurso](./media/plan-manage-costs/create-workspace-cost.png)


Si la suscripción de Azure tiene un límite de gasto, Azure le impide gastar por encima del importe del crédito. A medida que crea y usa recursos de Azure, se usan los créditos. Cuando alcanza el límite de crédito, los recursos que ha implementado se deshabilitan para el resto de ese período de facturación. No se puede cambiar el límite de crédito, pero sí puede quitarlo. Para más información sobre los límites de gasto, consulte [Límite de gasto de Azure](../cost-management-billing/manage/spending-limit.md).

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos de Azure Synapse, se incurre en costos. Los costos de unidad de uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto empiece a usar los recursos en Azure Synapse, incurrirá en costos, que podrá ver en el [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Al usar el análisis de costos, puede ver los costos de Azure Synapse Analytics de diferentes intervalos de tiempo en forma de gráficos y tablas. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de Azure Synapse en el análisis de costos:

1. Inicie sesión en Azure Portal.
2. Abra el ámbito, ya sea la suscripción o el grupo de recursos, en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos.
3. De forma predeterminada, el costo de los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta "Azure Synapse".

Los costos mensuales reales se muestran cuando se abre inicialmente el análisis de costos. Este es un ejemplo con todos los costos mensuales de uso.

![Ejemplo que muestra los costos acumulados de una suscripción](./media/plan-manage-costs/actual-monthly-costs.png)

- Para limitar la información a los costos de un único servicio, como Azure Synapse, seleccione **Agregar filtro** y, luego, elija **Nombre del servicio**. Después, seleccione **Azure Synapse Analytics**.

Este es un ejemplo que muestra solo los costos de Azure Synapse.

![Ejemplo que muestra los costos acumulados del servicio](./media/plan-manage-costs/filtered-monthly-costs.png)

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los costos de Azure Synapse por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro disponibles al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando usted u otro usuario necesita hacer un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="other-ways-to-manage-and-reduce-costs-for-azure-synapse"></a>Otras formas de administrar y reducir los costos de Azure Synapse 

### <a name="serverless-sql-pool"></a>Grupo de SQL sin servidor

Para más información sobre los costos del grupo SQL sin servidor, consulte [Administración de costos del grupo de SQL sin servidor en Azure Synapse Analytics](./sql/data-processed.md).

### <a name="dedicated-sql-pool"></a>Grupo de SQL dedicado

Para controlar los costos de un grupo de SQL dedicado, puede pausar el recurso cuando no se use. Por ejemplo, si no va a usar la base de datos durante la noche y los fines de semana, puede pausarla durante esas horas y reanudarla durante el día. Para más información, vea [Pausa y reanudación del proceso en un grupo de SQL dedicado mediante Azure Portal](./sql-data-warehouse/pause-and-resume-compute-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="serverless-apache-spark-pool"></a>Grupo de Apache Spark sin servidor

Para controlar los costos del grupo de Apache Spark sin servidor, habilite la característica de pausa automática de Apache Spark sin servidor y establezca el valor de tiempo de espera en consecuencia.  Al usar Synapse Studio para el desarrollo, Studio envía un mensaje de mantenimiento de conexión para mantener activa la sesión, que también es configurable, por lo que debe establecer un breve valor de tiempo de espera para la pausa automática.  Cuando haya terminado, cierre la sesión y el grupo de Apache Spark se pausará automáticamente una vez que se alcance el valor de tiempo de espera.
 
Durante el desarrollo, cree varias definiciones de grupos de Apache Spark de varios tamaños.  La creación de definiciones de grupo de Apache Spark es gratuita y solo se le cobrará por el uso.  El uso de Apache Spark en Azure Synapse se cobra por hora de núcleo virtual y se prorratea por minuto.  Por ejemplo, use tamaños de grupo pequeños para el desarrollo y la validación de código al tiempo que usa tamaños de grupo mayores para las pruebas de rendimiento.

### <a name="data-integration---pipelines-and-data-flows"></a>Integración de datos: canalizaciones y flujos de datos 

Para más información sobre los costos de la integración de datos, vea [Planeamiento y administración de los costos de Azure Data Factory](../data-factory/plan-manage-costs.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo planear y administrar los costos de [Azure Machine Learning](../machine-learning/concept-plan-manage-cost.md).