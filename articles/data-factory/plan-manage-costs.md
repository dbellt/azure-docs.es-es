---
title: Planeamiento para administrar los costos de Azure Data Factory
description: Aprenda a planear y administrar los costos de Azure Data Factory mediante el análisis de costos de Azure Portal.
author: shirleywangmsft
ms.author: shwang
ms.service: data-factory
ms.topic: how-to
ms.date: 04/28/2021
ms.openlocfilehash: 4e401886d37ec4221a0498863ec50ece8bffb984
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108292407"
---
# <a name="plan-to-manage-costs-for-azure-data-factory"></a>Planeamiento para administrar los costos de Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo planear y administrar los costos de Azure Data Factory. 

En primer lugar, al principio del proyecto de ETL se usa una combinación de las calculadoras de precios de Azure y de precios y consumo por canalización para planear los costos de Azure Data Factory antes de agregar recursos para que el servicio calcule los costos. Después, a medida que agregue recursos de Azure, revise los costos estimados. Después de comenzar a usar los recursos de Azure Data Factory, use las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para identificar las áreas en las que debería tomar medidas. Los costos de Azure Data Factory son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de Data Factory, se le facturarán todos los servicios y recursos de Azure usados para su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-azure-data-factory"></a>Cálculo de costos antes de usar Azure Data Factory
 
Use la [calculadora de precios de ADF](https://azure.microsoft.com/pricing/calculator/?service=data-factory) para obtener una cotización del costo por ejecutar la carga de trabajo ETL en Azure Data Factory.  Para usar la calculadora, debe especificar detalles como el número de ejecuciones de actividad, el número de horas por unidad de integración de datos, el tipo de proceso que se usará para Data Flow, el número de núcleos, el recuento de instancias y la duración de la ejecución, entre otros.

Una de las preguntas más frecuentes sobre la calculadora de precios es qué valores se deben usar como entradas.  Durante la fase de prueba de concepto, puede realizar ejecuciones de prueba con conjuntos de datos de ejemplo para comprender el consumo de varios medidores de ADF.  Después, en función del consumo para el conjunto de datos de ejemplo, puede realizar una proyección del consumo de todo el conjunto de datos y de la programación de la operacionalización.

> [!NOTE]
> Los precios usados en los ejemplos siguientes son hipotéticos y no pretenden reflejar precios reales.

Por ejemplo, supongamos que necesita trasladar 1 TB de datos diariamente de AWS S3 a Azure Data Lake Gen2.  Puede realizar una prueba de concepto en la que se mueven 100 GB de datos para medir el rendimiento de la ingesta de datos y comprender el consumo de facturación correspondiente.

Este es un ejemplo sobre los detalles de ejecución de una actividad de copia (su consumo real variará en función de la forma de su conjunto de datos específico, las velocidades de red, los límites de salida de la cuenta de S3, los límites de entrada en ADLS Gen2 y otros factores).

:::image type="content" source="media/plan-manage-costs/s3-copy-run-details.png" alt-text="Ejecución de copia en S3":::

Gracias a la [supervisión del consumo de en el nivel de ejecución de canalización](#monitor-consumption-at-pipeline-run-level), puede ver las cantidades de consumo de las mediciones de movimiento de datos correspondientes:

:::image type="content" source="media/plan-manage-costs/s3-copy-pipeline-consumption.png" alt-text="Consumo de la canalización de copia en S3":::

Por lo tanto, el número total de horas por DIU necesarias para mover 1 TB al día durante todo el mes es:

1,2667 (horas por DIU) * (1 TB/100 GB) * 30 (días en un mes) = 380 DIU por hora

Ahora puede introducir 30 ejecuciones de actividad y 380 horas por DIU a la calculadora de precios de ADF para obtener una estimación de la factura mensual:

:::image type="content" source="media/plan-manage-costs/s3-copy-pricing-calculator.png" alt-text="Calculadora de precios de una copia de S3":::

## <a name="understand-the-full-billing-model-for-azure-data-factory"></a>Modelo de facturación completa de Azure Data Factory

Azure Data Factory se ejecuta en la infraestructura de Azure que genera los costos al implementar recursos nuevos. Es importante entender que podrían generarse otros costos de infraestructura adicionales.

### <a name="how-youre-charged-for-azure-data-factory"></a>Cómo se le cobra por Azure Data Factory

Azure Data Factory es un servicio de integración de datos elástico y sin servidor diseñado para el escalado en la nube.  Esto significa que no es necesario planear ningún proceso de tamaño fijo para las cargas máximas; en su lugar, se especifica la cantidad de recursos que se asignarán a petición por operación, lo que permite diseñar los procesos ETL de una manera mucho más escalable. Además, el plan de ADF se factura según el consumo, por lo que solo paga por lo que usa.

Al crear o usar recursos de Azure Data Factory, es posible que se le cobre por los siguientes medidores:

- Ejecuciones de actividad de orquestación: se le cobra según el número de ejecuciones de actividad que se hayan orquestado.
- Horas de unidad de integración de datos (DIU): para las actividades de copia que se ejecutan en Azure Integration Runtime, se le cobra según el número de DIU usadas y la duración de la ejecución.
- Horas de núcleo virtual: para la ejecución y depuración del flujo de datos, se le cobra según el tipo de proceso, el número de núcleos virtuales y la duración de la ejecución.


Al final del ciclo de facturación, se suman los cargos de cada medidor. La factura muestra una sección para todos los costos de Azure Data Factory. Hay un elemento de línea independiente para cada medidor.

### <a name="other-costs-that-might-accrue-with-azure-data-factory"></a>Otros costos que pueden generarse con Azure Data Factory

Cuando se crean recursos para Azure Data Factory, también se crean recursos para otros servicios de Azure. Entre ellas, las siguientes:

- Ejecución de la actividad de canalización
- Ejecución de la actividad de canalización externa
- Creación, edición, recuperación y supervisión de artefactos de factoría de datos
- Duración de SSIS Integration Runtime en función del tipo de instancia y la duración

### <a name="using-azure-prepayment-with-azure-data-factory"></a>Uso del pago por adelantado de Azure con Azure Data Factory

Puede pagar los cargos de Azure Data Factory con el crédito del pago por adelantado de Azure. Sin embargo, no puede usar los créditos del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los de Azure Marketplace.

## <a name="monitor-costs"></a>Supervisión de costos

Los costos de Azure Data Factory se pueden supervisar en los niveles de factoría, ejecución de canalización y ejecución de actividad.

### <a name="monitor-costs-at-factory-level"></a>Supervisión de costos en el nivel de factoría

A medida que se usan recursos de Azure con Data Factory, se incurre en costos. Los costos de unidad de uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto se inicia el uso de Data Factory, se generan costos, que puede ver en el [análisis de costos](../cost-management/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Al usar el análisis de costos, puede ver los costos de Data Factory de diferentes intervalos de tiempo en gráficos y tablas. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de Data Factory en el análisis de costos:

1. Inicie sesión en Azure Portal.
2. Abra el ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos.
3. De forma predeterminada, el costo de los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta "Azure Data Factory v2".

Los costos mensuales reales se muestran cuando se abre inicialmente el análisis de costos. Este es un ejemplo con todos los costos mensuales de uso.

:::image type="content" source="media/all-costs.png" alt-text="Ejemplo que muestra los costos acumulados de una suscripción":::

- Para restringir los costos de un único servicio, como Data Factory, seleccione **Agregar filtro** y, a continuación, seleccione **Nombre del servicio**. A continuación, seleccione **Azure Data Factory v2**.

Este es un ejemplo que muestra solo los costos de Data Factory.

:::image type="content" source="media/service-specific-cost.png" alt-text="Ejemplo que muestra los costos acumulados del servicio":::

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los costos de Data Factory por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.

### <a name="monitor-consumption-at-pipeline-run-level"></a>Supervisión del consumo en el nivel de ejecución de canalización

En función de los tipos de actividades que tenga en la canalización, la cantidad de datos que se mueven y transforman, y la complejidad de la transformación, la ejecución de una canalización activará distintos medidores de facturación en Azure Data Factory.

Puede ver la cantidad de consumo de medidores distintos para ejecuciones de canalización individuales en la experiencia de usuario de Azure Data Factory. Para abrir la experiencia de supervisión, seleccione el icono **Supervisar y administrar** de la hoja de la factoría de datos de [Azure Portal](https://portal.azure.com/). Si ya está en ADF UX, haga clic en el icono **Supervisar** de la barra lateral izquierda. La vista de supervisión predeterminada es una lista de ejecuciones de la canalización.

Al hacer clic en el botón **Consumo** situado junto al nombre de la canalización, se mostrará una ventana emergente con el consumo de la ejecución de canalización agregada de todas las actividades de la canalización.

:::image type="content" source="media/plan-manage-costs/pipeline-run-consumption.png" alt-text="Consumo de la ejecución de canalización":::

:::image type="content" source="media/plan-manage-costs/pipeline-consumption-details.png" alt-text="Detalles de consumo de la canalización":::

La vista del consumo de la ejecución de canalización muestra el la cantidad de cada medidor de ADF que consumió la ejecución de canalización específica, pero no muestra el precio real que se cobrará, ya que la cantidad facturada depende del tipo de cuenta de Azure y del tipo de moneda que se usará.  Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md) (Información sobre los datos de Cost Management).

### <a name="monitor-consumption-at-activity-run-level"></a>Supervisión del consumo en el nivel de ejecución de actividad
Una vez que comprenda el consumo agregado en el nivel de ejecución de la canalización, existen escenarios en los que es necesario profundizar e identificar cuál es la actividad más costosa de la canalización.

Para ver el consumo en el nivel de la ejecución de la actividad, vaya a la opción **Crear y supervisar** de la interfaz de usuario de su factoría de datos. Desde la pestaña **Supervisión** en la que se muestra una lista de ejecuciones de canalización, haga clic en el vínculo del **nombre de canalización** para acceder a la lista de ejecuciones de actividad en la ejecución de canalización.  Haga clic en el botón **Salida** situado junto al nombre de la actividad y busque la propiedad **billableDuration** en la salida JSON:

El siguiente es un ejemplo de la ejecución de una actividad de copia:

:::image type="content" source="media/plan-manage-costs/copy-output.png" alt-text="Salida de copia":::

Y este es un ejemplo de la ejecución de una actividad de flujo de datos de asignación:

:::image type="content" source="media/plan-manage-costs/dataflow-output.png" alt-text="Salida de flujo de datos":::

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro disponibles al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando usted u otro usuario necesita hacer un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- [Página de precios de Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/ssis/)
- [Información sobre Azure Data Factory a través de ejemplos](./pricing-concepts.md)
- [Calculadora de precios de Azure Data Factory](https://azure.microsoft.com/pricing/calculator/?service=data-factory)
