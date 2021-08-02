---
title: Planeamiento y administración de costos de Azure API Management
description: Aprenda a planear y administrar los costos de Azure API Management mediante el análisis de costos de Azure Portal.
author: dlepow
ms.author: apimpm
ms.custom: subject-cost-optimization
ms.service: api-management
ms.topic: how-to
ms.date: 06/11/2021
ms.openlocfilehash: ec15f11c6aee82d117210402e4cc4fda114e86a2
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112034614"
---
# <a name="plan-and-manage-costs-for-api-management"></a>Planeamiento y administración de costos de API Management

En este artículo se describe cómo puede planear y administrar los costos de Azure API Management. En primer lugar, antes de agregar recursos del servicio para calcular los costos, use la calculadora de precios de Azure para planear los costos de API Management. Después de comenzar a usar los recursos de API Management, utilice las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para identificar las áreas en las que podría querer actuar. 

Los costos de API Management son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de API Management, se le facturarán todos los servicios y recursos de Azure usados para su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-api-management"></a>Cálculo de los costos antes de usar API Management

Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de agregar API Management. 

1. Busque *API Management* o seleccione **Integración** > **API Management**.
1. Seleccione **Ver** para agregar una estimación de costos predeterminada para una instancia de servicio de API Management.

> [!NOTE]
> Los costos que se muestran en este ejemplo son solo para fines de demostración. Para obtener información sobre los precios más recientes, consulte [Precios de API Management](https://azure.microsoft.com/pricing/details/api-management/).

:::image type="content" source="media/plan-manage-costs/pricing-calculator-developer-tier.png" alt-text="Cálculo de los costos del nivel Desarrollador":::

* La estimación de costos predeterminada se basa en una instancia de servicio de API Management en el [nivel de servicio](api-management-features.md) **Desarrollador** con una [unidad de capacidad](api-management-capacity.md). El nivel Desarrollador no es para evaluaciones ni casos de uso que no son de producción. No está respaldado por un Acuerdo de Nivel de Servicio.

* Para calcular los costos de las unidades de capacidad adicionales o de un nivel de servicio diferente, seleccione otras opciones en las listas desplegables **Unidades** y **Nivel**.

* Según la disponibilidad de características y el nivel de servicio, se pueden aplicar cargos adicionales para el uso de [puertas de enlace autohospedadas](self-hosted-gateway-overview.md).

Para obtener más información sobre los precios y las características, consulte:

* [Precios de API Management](https://azure.microsoft.com/pricing/details/api-management/)
* [Comparación de características de los planes de tarifa de Azure API Management](api-management-features.md)

### <a name="using-monetary-credit-with-api-management"></a>Uso de crédito monetario con API Management

Puede pagar los cargos de API Management con el pago por adelantado de Azure (antes conocido como compromiso monetario). Sin embargo, no puede usar el crédito del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los que proceden de Azure Marketplace.

## <a name="understand-the-full-billing-model"></a>Descripción del modelo de facturación completo

El uso de recursos de Azure con API Management incurre en costos. Los costos unitarios del uso de los recursos de Azure varían según lo siguiente:
* Intervalos de tiempo (segundos, minutos, horas y días)
* Uso de unidades (bytes, megabytes, etc)
* Número de transacciones

### <a name="how-youre-charged-for-api-management"></a>Cómo se le cobra por API Management

Al crear o usar recursos de Azure con API Management, se le cobrará en función de los niveles en los que esté trabajando. Obtenga más información sobre [cómo elegir el mejor nivel](./api-management-features.md).

| Niveles | Descripción |
| ----- | ----------- |
| Consumo | No incurre en costos fijos. Se le facturará en función del número de llamadas API al servicio por encima de un umbral determinado. |
| Desarrollador, Básico, Estándar y Prémium | Incurre en costos mensuales, en función del número de [unidades](./api-management-capacity.md) y [puertas de enlace auto-hospedadas](./self-hosted-gateway-overview.md). Las puertas de enlace auto hospedadas son gratuitas para el nivel Desarrollador. [Actualice](./upgrade-and-scale.md) a un nivel de servicio diferente en cualquier momento. |

También puede incurrir en cargos adicionales al usar otros recursos de Azure con API Management, como redes virtuales, zonas de disponibilidad y escrituras en varias regiones. Al final del ciclo de facturación, se suman los cargos de cada medidor. La factura muestra una sección con todos los costos de API Management. Hay un elemento de línea independiente para cada medidor.

## <a name="monitor-costs"></a>Supervisión de costos

Los costos se generan en cuanto se empieza a usar API Management. Puede ver los costos en el [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) o con la Calculadora de precios de Azure.

Al usar el análisis de costos, puede ver en gráficos y tablas los costos de API Management de diferentes intervalos de tiempo (día, mes actual y anterior, y año). También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

> [!NOTE]
> Los costos que se muestran en este ejemplo son solo para fines de demostración. Los costos variarán en función del uso de recursos y del precio actual.

Para ver los costos de API Management en el análisis de costos:

1. Inicie sesión en [Azure Portal](https://azure.microsoft.com).
1. Abra la ventana **Administración de costos y facturación**, seleccione **Administración de costos** en el menú y, a continuación, seleccione **Ámbito de facturación**. Por ejemplo, seleccione una suscripción en la lista.
1. Seleccione **Administración de costos** en el menú y, a continuación, seleccione **Análisis de costos**.
1. De manera predeterminada, los costos mensuales de todos los servicios se muestran en el primer gráfico de anillos. 

    :::image type="content" source="media/plan-manage-costs/api-management-cost-analysis.png" alt-text="Costos mensuales de la suscripción":::

1. Para restringir los costos de un único servicio, como API Management, seleccione **Agregar filtro** y, a continuación, seleccione **Nombre del servicio**. A continuación, seleccione **API Management**.

    :::image type="content" source="media/plan-manage-costs/api-management-apim-cost-analysis.png" alt-text="Ejemplo que muestra los costos acumulados de API Management":::

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los costos de API Management por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando necesita que otros usuarios hagan un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="other-ways-to-manage-and-reduce-costs-for-api-management"></a>Otras formas de administrar y reducir los costos de API Management

### <a name="scale-using-capacity-units"></a>Escalado mediante unidades de capacidad

Excepto en el nivel de servicio Consumo, API Management admite el escalado mediante la adición o eliminación de [*unidades de capacidad*](api-management-capacity.md). A medida que aumenta la carga en una instancia de API Management, la adición de unidades de capacidad puede ser más económica que la actualización a un nivel de servicio superior. El número máximo de unidades depende del nivel de servicio.

Cada unidad de capacidad tiene una determinada funcionalidad de procesamiento de solicitudes que depende del nivel de servicio. Por ejemplo, una unidad del nivel básico tiene un rendimiento máximo estimado de aproximadamente 1000 solicitudes por segundo. 

A medida que se agregan o quitan unidades, la capacidad y el costo se escalan de forma proporcional. Por ejemplo, dos unidades del nivel estándar proporcionan un rendimiento estimado de aproximadamente 2000 solicitudes por segundo. El rendimiento real puede ser diferente debido al tamaño de las solicitudes o respuestas, los patrones de conexión, el número de clientes que realizan solicitudes y otros factores.

[Supervise](api-management-howto-use-azure-monitor.md) la métrica de capacidad de la instancia de API Management para ayudar a tomar decisiones sobre el escalado o la actualización de una instancia de API Management para dar cabida a más carga.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la [capacidad](api-management-capacity.md) de API Management.
- Vea los pasos para escalar y actualizar API Management mediante [Azure Portal](upgrade-and-scale.md) y obtenga información sobre el [escalado automático](api-management-howto-autoscale.md).
