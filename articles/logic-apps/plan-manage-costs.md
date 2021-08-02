---
title: Planeamiento de la administración de los costos de Azure Logic Apps
description: Aprenda a planear y administrar los costos de Azure Logic Apps mediante el análisis de costos de Azure Portal.
ms.service: logic-apps
ms.reviewer: estfan, logicappspm, azla
ms.topic: how-to
ms.custom: subject-cost-optimization
ms.date: 05/25/2021
ms.openlocfilehash: 5bbdcd8032fbb4d20af2e681bf703c3d62985fe0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971623"
---
# <a name="plan-and-manage-costs-for-azure-logic-apps"></a>Planeamiento y administración de los costos de Azure Logic Apps

Este artículo le ayuda a planear y administrar los costos de Azure Logic Apps. Antes de crear o agregar recursos mediante este servicio, calcule los costos mediante la calculadora de precios de Azure. Después de empezar a usar recursos de Logic Apps, puede crear presupuestos y supervisar los costos mediante [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn). Para identificar las áreas en las que podría querer actuar, también puede revisar los costos previstos y supervisar las tendencias de gasto.

Tenga en cuenta que los costos de Logic Apps son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo calcular y administrar los costos de Logic Apps, se le facturarán todos los servicios y recursos de Azure utilizados en su suscripción de Azure, incluidos los servicios de cualquier tercero. Una vez que esté familiarizado con la administración de los costos de Logic Apps, puede aplicar métodos similares para administrar los costos de todos los servicios de Azure que se usan en su suscripción.

## <a name="prerequisites"></a>Requisitos previos

<!--Note for Azure service writer: This section covers prerequisites for the Cost Management's Cost Analysis feature. Add other prerequisites needed for your service after the Cost Management prerequisites. -->

[Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) es compatible con la mayoría de los tipos de cuentas de Azure. Para ver todos los tipos de cuentas admitidos, vea [Descripción de los datos de Cost Management](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure.

Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

<!--Note for Azure service writer: If you have other prerequisites for your service, add them here -->

<a name="understand-billing-model"></a>

## <a name="understand-the-billing-model"></a>Descripción del modelo de facturación

Azure Logic Apps se ejecuta en la infraestructura de Azure que [genera los costos](https://azure.microsoft.com/pricing/details/logic-apps/) al implementar recursos nuevos. Asegúrese de que comprende el [modelo de facturación del servicio Logic Apps junto con los recursos de Azure relacionados](logic-apps-pricing.md) y administre los costos generados por estas dependencias al realizar cambios en los recursos implementados.

<a name="typical-costs"></a>

### <a name="costs-that-typically-accrue-with-azure-logic-apps"></a>Costos que suelen generarse con Azure Logic Apps

El servicio Azure Logic Apps aplica distintos modelos de precios, en función de los recursos que se creen y utilicen:

* Los recursos de aplicaciones lógicas creados y ejecutados en el servicio Logic Apps multiinquilino usan un [modelo de precios de consumo (pago por uso)](../logic-apps/logic-apps-pricing.md#consumption-pricing).

* Los recursos de aplicaciones lógicas creados y ejecutados en el servicio Azure Logic Apps de inquilino único usan un [modelo de precios de plan de hospedaje](../logic-apps/logic-apps-pricing.md#standard-pricing).

* Los recursos de aplicaciones lógicas creados y ejecutados en un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) usan un [modelo de precios fijos](../logic-apps/logic-apps-pricing.md#fixed-pricing).

Estos son otros recursos que incurren en costos cuando se crean para usarlos con aplicaciones lógicas:

* Una [cuenta de integración](../logic-apps/logic-apps-pricing.md#integration-accounts) es un recurso independiente que se crea y se vincula a aplicaciones lógicas para crear integraciones de B2B. Las cuentas de integración usan un [modelo de precios fijos](../logic-apps/logic-apps-pricing.md#integration-accounts) en el que la tarifa se basa en el tipo de cuenta de integración o el *nivel* que utilice.

* Un [ISE](../logic-apps/logic-apps-pricing.md#fixed-pricing) es un recurso independiente que se crea como una ubicación de implementación para aplicaciones lógicas que necesitan acceso directo a los recursos de una red virtual. Los ISE usan un [modelo de precios fijos](../logic-apps/logic-apps-pricing.md#fixed-pricing) donde la tarifa se basa en la SKU de ISE creada y en otras opciones. Sin embargo, la retención de datos y el consumo de almacenamiento no generan costos.

* Un [conector personalizado](../logic-apps/logic-apps-pricing.md#consumption-pricing) es un recurso independiente que se crea para una API de REST que no tiene ningún conector precompilado para utilizarlo en las aplicaciones lógicas. Las ejecuciones de conectores personalizados usan un [modelo de precios de consumo](../logic-apps/logic-apps-pricing.md#consumption-pricing), excepto cuando se usan en un ISE.

<a name="storage-operations-costs"></a>

#### <a name="storage-operations-and-costs"></a>Operaciones y costos de almacenamiento

Azure Logic Apps usa [Azure Storage](../storage/index.yml) en las operaciones de almacenamiento. Con Azure Logic Apps multiinquilino, el uso y los costos de almacenamiento se asocian a la aplicación lógica. [La retención de datos y los costos de almacenamiento](../logic-apps/logic-apps-pricing.md#data-retention) acumulan costos mediante un [modelo de precio fijo](../logic-apps/logic-apps-pricing.md#fixed-pricing). Por ejemplo, las entradas y salidas del historial de ejecución se mantienen en el almacenamiento en segundo plano, que difiere de los recursos de almacenamiento creados, administrados y a los que se accede de forma independiente desde la aplicación lógica.

Con Azure Logic Apps de inquilino único, puede usar su propia [cuenta de almacenamiento](../azure-functions/storage-considerations.md#storage-account-requirements) de Azure. Esta funcionalidad proporciona más control y flexibilidad con los datos de Logic Apps. Cuando los flujos de trabajo *con estado* ejecutan sus operaciones, Azure Logic Apps realiza transacciones de almacenamiento en tiempo de ejecución. Por ejemplo, las colas se utilizan para la programación, mientras que las tablas y blobs se utilizan para almacenar los estados de los flujos de trabajo. Los costos de almacenamiento cambian en función del contenido del flujo de trabajo. Los distintos desencadenadores, acciones y cargas tienen como resultado diferentes operaciones y necesidades de almacenamiento. Las transacciones de almacenamiento siguen el [modelo de precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/). Los costos de almacenamiento se enumeran por separado en la factura de Azure.

En el caso de Azure Logic Apps de inquilino único, puede hacerse una idea del número de operaciones de almacenamiento que un flujo de trabajo podría ejecutar y su costo con la [calculadora de almacenamiento de Logic Apps](https://logicapps.azure.com/calculator). Puede seleccionar un flujo de trabajo de ejemplo o usar una definición de flujo de trabajo existente. El primer cálculo estima el número de operaciones de almacenamiento en el flujo de trabajo. Luego, puede usar estos números para calcular los posibles costos mediante la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/). Para más información, revise [Estimación de las necesidades de almacenamiento y los costos de los flujos de trabajo de Azure Logic Apps de inquilino único](estimate-storage-costs.md).

<a name="costs-after-resource-deletion"></a>

### <a name="costs-might-accrue-after-resource-deletion"></a>Se pueden acumular costos después de la eliminación de recursos

<!--Note to Azure service writer: You might need to sync with your product team to identify resources that continue to exist after those ones for your service are deleted. If you're certain that no resources can exist after those for your service are deleted, then omit this section. -->

Después de eliminar una aplicación lógica, el servicio Logic Apps no creará ni ejecutará nuevas instancias de flujo de trabajo. Sin embargo, todas las ejecuciones en curso y pendientes continuarán hasta que finalicen. En función del número de ejecuciones, este proceso puede tardar algún tiempo. Para más información, vea [Administración de aplicaciones lógicas](manage-logic-apps-with-azure-portal.md#delete-logic-apps).

Si tiene estos recursos después de eliminar una aplicación lógica, estos recursos seguirán existiendo y generarán costos hasta que los elimine:

* Recursos de Azure que se crean y administran de forma independiente desde la aplicación lógica que se conecta a esos recursos, por ejemplo, aplicaciones de función de Azure, centros de eventos, cuadrículas de eventos, etc.

* Cuentas de integración

* Entornos del servicio de integración (ISE)

  Si [elimina un ISE](ise-manage-integration-service-environment.md#delete-ise), la red virtual de Azure asociada, las subredes y otros recursos relacionados seguirán existiendo. Después de eliminar el ISE, puede que tenga que esperar hasta un número específico de horas antes de intentar eliminar la red virtual o las subredes.

### <a name="using-monetary-credit-with-azure-logic-apps"></a>Uso de crédito monetario con Azure Logic Apps

Puede pagar los cargos de Azure Logic Apps con el crédito de compromiso monetario de EA. Sin embargo, no puede usar los créditos de compromiso monetario de EA para pagar los gastos de productos y servicios de terceros, como los de Azure Marketplace.

<a name="estimate-costs"></a>

## <a name="estimate-costs"></a>cálculo de los costos

Antes de crear recursos con Azure Logic Apps, calcule los costos con la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/). Para más información, vea [Modelo de precios de Azure Logic Apps](../logic-apps/logic-apps-pricing.md).

1. En el menú de la izquierda de la [página de la calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/), seleccione **Integración** > **Azure Logic Apps**.

   ![Captura de pantalla en la que se muestra la calculadora de precios de Azure con la opción "Azure Logic Apps" seleccionada.](./media/plan-manage-costs/add-azure-logic-apps-pricing-calculator.png)

1. Desplácese hacia abajo en la página hasta que pueda ver la calculadora de precios de Azure Logic Apps. En las distintas secciones de los recursos de Azure que están directamente relacionadas con Azure Logic Apps, escriba los números de recursos que planea usar y el número de intervalos en los que puede usar esos recursos.

   En esta captura de pantalla se muestra un ejemplo de cálculo de costos con la calculadora:

   ![Ejemplo que muestra el costo estimado en la calculadora de precios de Azure](./media/plan-manage-costs/example-logic-apps-pricing-calculator.png)

1. Para actualizar las estimaciones de costos a medida que crea y usa nuevos recursos relacionados, vuelva a esta calculadora y actualice esos recursos aquí.

<a name="create-budgets-alerts"></a>

## <a name="create-budgets-and-alerts"></a>Creación de presupuestos y alertas

Para ayudarle a administrar de forma proactiva los costos de su cuenta o suscripción de Azure, puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) y [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) mediante el servicio [Azure Cost Management and Billing](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) y sus funcionalidades.  Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos.

En función de los gastos comparados con los umbrales de presupuesto y costo, las alertas notifican automáticamente a las partes interesadas de las anomalías de gasto y los riesgos de incurrir en costos excesivos. Si desea más granularidad en la supervisión, también puede crear presupuestos que usen filtros para recursos o servicios específicos de Azure. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro, vea [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

<a name="monitor-costs"></a>

## <a name="monitor-costs"></a>Supervisión de costos

Los costos de unidad de uso de recursos varían según los intervalos de tiempo, como segundos, minutos, horas y días, o según el uso de unidades, como bytes, megabytes, etc. Algunos ejemplos son: por día, mes actual y anterior y año. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto Al usar las características de análisis de costos, puede ver los costos como gráficos y tablas durante varios intervalos de tiempo. Si ha creado presupuestos y previsiones de costos, también puede encontrar fácilmente dónde se superan los presupuestos y se podría haber producido un exceso de gasto.

Después de empezar a incurrir en costos de recursos que se crean o se empiezan a usar en Azure, puede revisar y supervisar estos costos de las siguientes maneras:

* [Supervisar las ejecuciones de aplicaciones lógicas y el uso de almacenamiento](#monitor-billing-metrics) mediante Azure Monitor

* Ejecutar [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) con [Azure Cost Management and Billing](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)

<a name="monitor-billing-metrics"></a>

### <a name="monitor-logic-app-executions-and-storage-usage"></a>Supervisar las ejecuciones de aplicaciones lógicas y el uso de almacenamiento

Con Azure Monitor, puede ver estas métricas para una aplicación lógica específica:

* Ejecuciones de acciones facturables
* Ejecuciones del desencadenador facturable
* Uso de facturación para las ejecuciones de la operación nativa
* Uso de facturación para las ejecuciones del conector estándar
* Uso de facturación para el consumo de almacenamiento
* Total de ejecuciones facturables

<a name="execution-storage-metrics"></a>

#### <a name="view-execution-and-storage-consumption-metrics"></a>Visualización de las métricas de consumo de almacenamiento y ejecuciones

1. En Azure Portal, busque y abra la aplicación lógica. En el menú de la aplicación lógica, en **Supervisión**, seleccione **Métricas**.

1. En el panel de la derecha, en **Título de gráfico**, en la barra de métricas, abra la lista **Métrica** y seleccione la métrica que desee.

   > [!NOTE]
   > El consumo de almacenamiento se mide como el número de unidades de almacenamiento (GB) que usa la aplicación lógica, y se factura. Las ejecuciones que usan menos de 500 MB de almacenamiento podrían no aparecer en la vista de supervisión, aunque se siguen facturando.

   ![Captura de pantalla en la que se muestra el panel Métricas con la lista "Métrica" abierta.](./media/logic-apps-pricing/select-metric.png)

1. En la esquina superior derecha del panel, seleccione el período de tiempo que desee.

1. Para ver otros datos sobre el consumo de almacenamiento, en particular los tamaños de entrada y salida de las acciones, en el historial de ejecución de la aplicación lógica, [siga estos pasos](#view-input-output-sizes).

<a name="view-input-output-sizes"></a>

#### <a name="view-action-input-and-output-sizes-in-run-history"></a>Visualización de los tamaños de entrada y salida de las acciones en el historial de ejecución

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, seleccione **Introducción**.

1. En el panel de la derecha, en **Historial de ejecuciones**, seleccione la ejecución que tiene las entradas y salidas que quiera ver.

1. En **Ejecución de aplicación lógica**, seleccione **Detalles de ejecución**.

1. En el panel **Detalles de ejecución de la aplicación lógica**, seleccione la acción que quiera ver en la tabla de acciones, que muestra el estado y la duración de cada acción.

1. En el panel **Acción de aplicación lógica**, busque los tamaños de las entradas y salidas de esa acción. En **Vínculo de entradas** y **Vínculos salidas**, busque los vínculos a dichas entradas y salidas.

   > [!NOTE]
   > En el caso de los bucles, solo las acciones de nivel superior muestran los tamaños de sus entradas y salidas. En el caso de las acciones dentro de bucles anidados, las entradas y salidas muestran un tamaño cero y la ausencia de vínculos.

<a name="run-cost-analysis"></a>

### <a name="run-cost-analysis-by-using-azure-cost-management-and-billing"></a>Ejecución de análisis de costos con Azure Cost Management and Billing

Para revisar los costos del servicio Logic Apps según un ámbito específico, por ejemplo, una suscripción de Azure, puede usar las funcionalidades del [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) de [Azure Cost Management and Billing](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

1. En Azure Portal, abra el ámbito que desee, como su suscripción de Azure. En el menú de la izquierda, en **Cost Management**, seleccione **Análisis de costos**.

   Al abrir por primera vez el panel Análisis de costos, en el gráfico superior se muestran los costos de uso reales y previstos de todos los servicios de la suscripción correspondientes al mes en curso.

   ![Captura de pantalla en la que se muestran Azure Portal y el panel de análisis de costos, con un ejemplo de costos reales y previstos en una suscripción.](./media/plan-manage-costs/cost-analysis-total-actual-forecasted-costs.png)

   > [!TIP]
   > Para cambiar los ámbitos, en el panel **Análisis de costos** de la barra de filtros, seleccione el filtro **Ámbito**. En el panel **Seleccionar ámbito**, cambie al ámbito que desee.

   Debajo, en los gráficos de anillos se muestran los costos actuales desglosados por servicios de Azure, región de Azure (ubicación) y grupo de recursos.

   ![Captura de pantalla en la que se muestran Azure Portal y el panel de análisis de costos con gráficos de anillos para servicios, regiones y grupos de recursos.](./media/plan-manage-costs/cost-analysis-donut-charts.png)

1. Para filtrar el gráfico en un área específica, como un servicio o un recurso, en la barra de filtros, seleccione **Agregar filtro**.

1. En la lista del lado izquierdo, seleccione el tipo de filtro, por ejemplo, **Nombre de servicio**. En la lista del lado derecho, seleccione el filtro, por ejemplo, **aplicaciones lógicas**. Cuando termine, seleccione la marca de verificación verde.

   ![Captura de pantalla en la que se muestran Azure Portal y el panel de análisis de costos con selecciones de filtro.](./media/plan-manage-costs/cost-analysis-add-service-name-filter.png)

   Por ejemplo, este es el resultado del servicio Logic Apps:

   ![Captura de pantalla en la que se muestran Azure Portal y el panel de análisis de costos con los resultados filtrados por "aplicaciones lógicas".](./media/plan-manage-costs/cost-analysis-total-actual-costs-service.png)

### <a name="export-cost-data"></a>Exportación de datos de costos

Cuando necesite realizar más análisis de datos sobre los costos, puede [exportar los datos de los costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Por ejemplo, un equipo de finanzas puede analizar estos datos con Excel o Power BI. Puede exportar los costos según una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="other-ways-to-manage-and-reduce-costs"></a>Formas alternativas de administrar y reducir costos

<!-- Note to Azure service writer: This section is optional. Other than using the Cost Management methods above, your service probably has other specific ways to minimize costs. For example, you might have best practice advice or specific ways to reduce costs that are specific to your service. If so, try to add that guidance here or at least summarize key points. Try to be as prescriptive as possible. If you have more comprehensive content, add links to your other published articles or sections here.

Add a statement that discusses any recommended settings for your service that might help keep the charges minimal if a service isn't being actively used by the customer. For example: Will turning off a VM help to get no charges for the specific VM resource?

Otherwise, if no other cost-saving recommendations or best practices exist to reduce costs, cut this section.
-->

Para ayudarle a reducir los costos de las aplicaciones lógicas y los recursos relacionados, pruebe estas opciones:

* Si es posible, use [acciones y desencadenadores integrados](../connectors/built-in.md), cuya ejecución cuesta menos que la de [las acciones y los desencadenadores de los conectores administrados](../connectors/managed.md).

  Por ejemplo, puede reducir los costos al acceder a otros recursos mediante la [acción HTTP](../connectors/connectors-native-http.md) o con una llamada a una función que ha creado con el [servicio Azure Functions](../azure-functions/functions-overview.md) y mediante la [acción integrada de Azure Functions](../logic-apps/logic-apps-azure-functions.md). Sin embargo, el uso de Azure Functions también incurre en costos, por lo que debe asegurarse de comparar las opciones.

* [Especifique condiciones de desencadenamiento precisas](logic-apps-workflow-actions-triggers.md#trigger-conditions) para ejecutar un flujo de trabajo.

  Por ejemplo, puede especificar que un desencadenador se active solo cuando un sitio web de destino devuelve un error de servidor interno. En la definición JSON del desencadenador, use la propiedad `conditions` para especificar una condición que haga referencia al código de estado del desencadenador.

* Si un desencadenador tiene una versión de sondeo y una versión de webhook, pruebe la versión de webhook, que espera a que se produzca el evento especificado antes de la activación, en lugar de comprobar periódicamente el evento.

* Llame a la aplicación lógica con otro servicio para que el desencadenador se active solo cuando se deba ejecutar el flujo de trabajo.

  Por ejemplo, puede llamar a la aplicación lógica desde una función que cree y ejecute con el servicio Azure Functions. Por ejemplo, vea [Llamada o desencadenamiento de aplicaciones lógicas con Azure Functions y Azure Service Bus](logic-apps-scenario-function-sb-trigger.md).

* [Deshabilite las aplicaciones lógicas](manage-logic-apps-with-azure-portal.md#disable-or-enable-logic-apps) que no tengan que ejecutarse constantemente o [elimine las aplicaciones lógicas](manage-logic-apps-with-azure-portal.md#delete-logic-apps) que ya no necesite. Si es posible, deshabilite todos los demás recursos que no necesite de forma permanente.

## <a name="next-steps"></a>Pasos siguientes

* [Optimización de la inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Administración de costos mediante análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Prevención de costes inesperados](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* Curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)