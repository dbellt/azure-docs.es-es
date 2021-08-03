---
title: Planeamiento de la administración de costos de Azure ExpressRoute
description: Aprenda a planear y administrar los costos de Azure ExpressRoute mediante el análisis de costos de Azure Portal.
author: duongau
ms.author: duau
ms.custom: subject-cost-optimization
ms.service: expressroute
ms.topic: how-to
ms.date: 05/18/2021
ms.openlocfilehash: c4a2113a5aa95c1bacf7be0f4b376b9377412371
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971694"
---
# <a name="plan-and-manage-costs-for-azure-expressroute"></a>Planeamiento y administración de costos de Azure ExpressRoute

En este artículo se describe cómo puede planear y administrar los costos de Azure ExpressRoute. En primer lugar, antes de agregar recursos del servicio para calcular los costos, use la calculadora de precios de Azure para planear los costos de ExpressRoute. Después, a medida que agregue recursos de Azure, revise los costos estimados. 

Una vez que haya empezado a usar recursos de Azure ExpressRoute, utilice las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para determinar las áreas en las que podría querer actuar. Los costos de Azure ExpressRoute son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de Azure ExpressRoute, se le facturarán todos los servicios y recursos de Azure usados en su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="local-vs-standard-vs-premium"></a>Local frente a Estrategias de enrutamiento de máquina virtual Estándar Premium

Azure ExpressRoute tiene tres SKU de circuito diferentes: [*Local*](./expressroute-faqs.md#expressroute-local), *Estándar* y [*Prémium*](./expressroute-faqs.md#expressroute-premium). La forma en que se le cobra el uso de ExpressRoute varía entre estos tres tipos de SKU. Con la SKU local, se le cobra automáticamente un plan de datos ilimitado. Con la SKU Estándar y Premium, puede seleccionar entre un plan de datos de uso medido o ilimitado. Todos los datos de entrada son gratuitos, excepto cuando se usa el complemento Global Reach. Para optimizar mejor el costo y el presupuesto, es importante comprender qué tipos de SKU y plan de datos funcionan mejor con su carga de trabajo.

## <a name="estimate-costs-before-using-azure-expressroute"></a>Cálculo de costos antes de usar Azure ExpressRoute

Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de crear un circuito Azure ExpressRoute. 

1. A la izquierda, seleccione **Redes** y, luego, elija **Azure ExpressRoute** para comenzar. 

1. Seleccione la *zona* adecuada según la ubicación de emparejamiento. Consulte [Proveedores de conectividad de ExpressRoute](./expressroute-locations-providers.md#partners) para seleccionar la *zona* adecuada en la lista desplegable. 

1. Luego, seleccione los valores de *SKU*, *Circuit Speed* (Velocidad de circuito) y *Plan de datos* en los que va a basar su estimación. 

1. En *Additional outbound data transfer* (Transferencia de datos de salida adicionales), escriba una estimación en GB de la cantidad de datos de salida que puede usar en el transcurso de un mes. 

1. Por último, puede agregar el *complemento Global Reach* a la estimación.

En la captura de pantalla siguiente se muestra la estimación de costos mediante la calculadora:

:::image type="content" source="media/plan-manage-cost/capacity-calculator-cost-estimate.png" alt-text="Estimación de costos de ExpressRoute en la calculadora de Azure":::

Para más información, consulte [Precios de Azure ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).

### <a name="expressroute-gateway-estimated-cost"></a>Costo estimado de la puerta de enlace de ExpressRoute

Si usa una puerta de enlace de ExpressRoute para vincular una red virtual al circuito ExpressRoute, siga estos pasos para estimar el costo del uso de la puerta de enlace.

1. A la izquierda, seleccione **Redes** y, luego, elija **VPN Gateway** para comenzar. 

1. Seleccione la *región* de la puerta de enlace y, luego, cambie *Tipo* por **Puertas de enlace de ExpressRoute**.

1. En *Tipo de puerta de enlace*, seleccione un valor en la lista desplegable.

1. En *Horas de puerta de enlace*, especifique un valor. (720 horas = 30 días)

## <a name="understand-the-full-billing-model-for-expressroute"></a>Modelo de facturación completo de ExpressRoute

Azure ExpressRoute se ejecuta en la infraestructura de Azure, donde se acumulan los costos junto con ExpressRoute al implementar el nuevo recurso. Es importante que comprenda que hay otras infraestructuras que pueden generar costos. Este costo se debe administrar al realizar cambios en los recursos implementados. 

### <a name="costs-that-typically-accrue-with-expressroute"></a>Costos que normalmente se acumulan con ExpressRoute

#### <a name="expressroute"></a>ExpressRoute

Al crear un circuito ExpressRoute, puede optar por crear una puerta de enlace de ExpressRoute para vincular la red virtual al circuito. Las puertas de enlace de ExpressRoute se cobran a una tarifa por hora más el costo de un circuito ExpressRoute. Consulte [Precios de ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute) y seleccione Puertas de enlace de ExpressRoute para ver las tarifas de diferentes SKU de puerta de enlace.

La transferencia de datos de entrada se incluye en el costo mensual del circuito ExpressRoute de las tres SKU. La transferencia de datos de salida solo se incluye en los planes de datos ilimitados. En los planes de datos limitados, la transferencia de datos de salida se cobra por GB usado en función del número de zona de la [ubicación de emparejamiento](expressroute-locations-providers.md#partners).

#### <a name="expressroute-direct"></a>ExpressRoute Direct

ExpressRoute Direct tiene una tarifa de puerto mensual que incluye el precio de los circuitos ExpressRoute de SKU Local y Estándar. En el caso de los circuitos de SKU Prémium, hay una tarifa de circuito adicional. La transferencia de datos de salida se cobra por GB usado en función del número de zona de la ubicación de emparejamiento. El cargo de datos de salida solo se aplica a las SKU Estándar y Prémium.
 
#### <a name="expressroute-global-reach"></a>Global Reach de ExpressRoute

Global Reach de ExpressRoute es un complemento que puede habilitar para ExpressRoute y ExpressRoute Direct para vincular circuitos ExpressRoute. La transferencia de datos de entrada y salida se cobra por GB usado en función del número de zona de la ubicación de emparejamiento.

### <a name="costs-might-accrue-after-resource-deletion"></a>Se pueden acumular costos después de la eliminación de recursos

Si tiene una puerta de enlace de ExpressRoute después de eliminar el circuito ExpressRoute, se le seguirá cobrando hasta que la elimine.

### <a name="using-azure-prepayment-credit"></a>Uso del crédito del pago por adelantado de Azure

Los cargos de ExpressRoute se pueden pagar con el crédito del pago por adelantado de Azure (antes conocido como compromiso monetario). Sin embargo, no puede usar el crédito del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los que proceden de Azure Marketplace.

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos de Azure con ExpressRoute, se generan costos. Los costos de unidad de uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto se inicia el uso de ExpressRoute, se generan costos, que puede ver en el [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Al usar el análisis de costos, puede ver los costos del circuito ExpressRoute de diferentes intervalos de tiempo en gráficos y tablas. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de ExpressRoute en el análisis de costos:

1. Inicie sesión en Azure Portal.

1. Vaya a **Suscripciones**, seleccione una suscripción de la lista y, luego, elija **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos. De forma predeterminada, el costo de los servicios se muestra en el primer gráfico de anillos.

    Los costos mensuales reales se muestran cuando se abre inicialmente el análisis de costos. Este es un ejemplo con todos los costos mensuales de uso.

    :::image type="content" source="media/plan-manage-cost/cost-analysis-pane.png" alt-text="Ejemplo que muestra los costos acumulados de una suscripción":::
    

1.  Para limitar la información a los costos de un único servicio, como ExpressRoute, seleccione **Agregar filtro** y, luego, elija **Nombre del servicio**. Luego, seleccione **ExpressRoute**.

    Este es un ejemplo que muestra solo los costos de ExpressRoute.

    :::image type="content" source="media/plan-manage-cost/cost-analysis-pane-expressroute.png" alt-text="Ejemplo que muestra los costos acumulados de ExpressRoute":::

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los costos de ExpressRoute por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.

## <a name="create-budgets-and-alerts"></a>Creación de presupuestos y alertas

Puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando usted u otro usuario necesita hacer un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda más sobre cómo funcionan los precios con Azure Storage. Consulte [Precios de Azure ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).
- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).