---
title: Planeamiento y administración de los costos de Azure Cosmos DB
description: Obtenga información acerca de cómo planear y administrar los costos de Azure Cosmos DB mediante el análisis de costos en Azure Portal.
author: SnehaGunda
ms.author: sngun
ms.custom: subject-cost-optimization
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/05/2021
ms.openlocfilehash: 98e849791acd71ea8bf3ac9cb1949da9f562e749
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106490835"
---
# <a name="plan-and-manage-costs-for-azure-cosmos-db"></a>Planeamiento y administración de los costos de Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

En este artículo se describe cómo puede planear y administrar los costos de Azure Cosmos DB. En primer lugar, puede usar la calculadora de capacidad de Azure Cosmos DB para calcular el costo de la carga de trabajo antes de crear los recursos. Luego puede revisar el costo estimado y empezar a crear los recursos.

Después de comenzar a usar los recursos de Azure Cosmos DB, use las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para identificar las áreas en las que podría querer actuar. Los costos de Azure Cosmos DB son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de Azure Cosmos DB, se le facturarán todos los servicios y recursos de Azure usados para su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

### <a name="provisioned-throughput-or-serverless"></a>Rendimiento aprovisionado o sin servidor

Azure Cosmos DB admite dos tipos de modos de capacidad: [rendimiento aprovisionado](set-throughput.md) y [sin servidor](serverless.md). La forma de cobrar por el uso de Azure Cosmos DB varía mucho entre estos dos modos, por lo que es importante elegir el que mejor se adapte a su carga de trabajo. Consulte el artículo sobre [cómo elegir entre el modo de rendimiento aprovisionado y el modo sin servidor](throughput-serverless.md) para obtener instrucciones y recomendaciones sobre cómo tomar esta decisión.

### <a name="cost-analysis"></a>Análisis de costos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-azure-cosmos-db"></a>Cálculo de costos antes de usar Azure Cosmos DB

Azure Cosmos DB admite dos tipos de modos de capacidad: rendimiento aprovisionado y sin servidor. Puede realizar exactamente las mismas operaciones de base de datos en ambos modos, pero la forma de facturar estas operaciones es diferente.

### <a name="estimate-provisioned-throughput-costs"></a>Cálculo de costos del rendimiento aprovisionado

Si tiene previsto usar Azure Cosmos DB en el modo de rendimiento aprovisionado, use la [calculadora de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) para calcular los costos antes de crear los recursos en una cuenta de Azure Cosmos. La calculadora de capacidad se usa para obtener una estimación del rendimiento necesario y el costo de la carga de trabajo. La configuración de las bases de datos y los contenedores de Azure Cosmos con la cantidad adecuada de rendimiento aprovisionado, o [unidades de solicitud (RU/s)](request-units.md), para la carga de trabajo es esencial para optimizar el costo y el rendimiento. Tiene que escribir detalles como el tipo de API, el número de regiones, el tamaño de los elementos, las solicitudes de lectura/escritura por segundo y el total de datos almacenados para obtener una estimación de costos. Para más información acerca de la calculadora de capacidad, consulte el artículo [Estimación](estimate-ru-with-capacity-planner.md).

En la captura de pantalla siguiente se muestra el rendimiento y la estimación de costos mediante la calculadora de capacidad:

:::image type="content" source="./media/plan-manage-costs/capacity-calculator-cost-estimate.png" alt-text="Estimación de costos en la calculadora de capacidad de Azure Cosmos DB":::

### <a name="estimate-serverless-costs"></a><a id="estimating-serverless-costs"></a>Cálculo de los costos sin servidor

Si tiene previsto usar Azure Cosmos DB en el modo sin servidor, debe calcular el número de [unidades de solicitud](request-units.md) y GB de almacenamiento que puede consumir mensualmente. Puede calcular la cantidad de unidades de solicitud necesaria mediante la evaluación del número de operaciones de base de datos que se emitirán en un mes y la multiplicación de su importe por el costo de unidades de solicitud correspondientes. En la tabla siguiente se enumeran los cargos de unidades de solicitud estimados para las operaciones de base de datos comunes:

| Operación | Costo estimado | Notas |
| --- | --- | --- |
| Crear un elemento | 5 RU | Costo medio de un elemento de 1 KB con menos de cinco propiedades para indexar |
| Actualizar un elemento | 10 RU | Costo medio de un elemento de 1 KB con menos de cinco propiedades para indexar |
| Lectura de un elemento individual por su identificador y la clave de partición (lectura de punto) | 1 RU | Costo medio de un elemento de 1 KB |
| Eliminación de un elemento | 5 RU | |
| Ejecutar una consulta | 10 RU | Costo medio de una consulta que saca el máximo partido de la [indexación](index-overview.md) y devuelve 100 resultados o menos. |

> [!IMPORTANT] 
> Preste atención a las notas de la tabla anterior. Para una estimación más precisa de los costos reales de las operaciones, puede usar el [emulador de Azure Cosmos DB](local-emulator.md) y [medir el costo de unidades de solicitud exacto de las operaciones](find-request-unit-charge.md). Aunque el emulador de Azure Cosmos DB no admite el uso de servidores, informa de un cargo estándar de unidades de solicitud para las operaciones de base de datos y se puede usar para esta estimación.

Una vez que haya calculado el número total de unidades de solicitud y los GB de almacenamiento que probablemente consumirá durante un mes, la siguiente fórmula devolverá la estimación de costos: **([Número de unidades de solicitud]/1 millón * 0,25 USD) + ([GB de almacenamiento] * 0,25 USD)** .

> [!NOTE]
> Los costos que se muestran en el ejemplo anterior son solo para fines de demostración. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener la información más reciente sobre los precios.

## <a name="understand-the-full-billing-model"></a>Descripción del modelo de facturación completo

Azure Cosmos DB se ejecuta en la infraestructura de Azure que genera los costos al implementar recursos nuevos. Es importante entender que podrían generarse otros costos de infraestructura adicionales.

### <a name="how-youre-charged-for-azure-cosmos-db"></a>Cómo se le cobra por Azure Cosmos DB

Al crear o usar recursos de Azure Cosmos DB, es posible que se le cobre por los siguientes medidores:

* **Operaciones de base de datos**: se le cobrará en función de las unidades de solicitud (RU/s) aprovisionadas o consumidas:
  * Rendimiento aprovisionado estándar (manual): se le cobrará una tarifa por hora por las RU/s aprovisionadas en el contenedor o la base de datos.
  * Rendimiento aprovisionado de escalabilidad automática: se le facturará en función del número máximo de RU/s a las que el sistema escaló verticalmente en cada hora.

* **Almacenamiento consumido** : se le cobra en función de la cantidad total de almacenamiento (en GB) consumida por los datos y los índices en una hora determinada.

Se aplicará un cargo adicional si usa las características de Azure Cosmos DB, como el almacenamiento de copia de seguridad, almacenamiento analítico, zonas de disponibilidad y escrituras en varias regiones. Al final del ciclo de facturación, se suman los cargos de cada medidor. La factura muestra una sección para todos los costos de Azure Cosmos DB. Hay un elemento de línea independiente para cada medidor. Para más información, consulte el artículo sobre el [Modelo de precios](how-pricing-works.md).

### <a name="using-azure-prepayment"></a>Uso del pago por adelantado de Azure

Puede pagar los cargos de Azure Cosmos DB con el crédito del pago por adelantado de Azure. Sin embargo, no puede usar los créditos del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los de Azure Marketplace.

## <a name="review-estimated-costs-in-the-azure-portal"></a>Revisión de los costos estimados en Azure Portal

Al comenzar a usar los recursos de Azure Cosmos DB desde Azure Portal, puede ver los costos estimados. Siga estos pasos para revisar la estimación de costos:

1. Inicie sesión en Azure Portal y vaya a la cuenta de Azure Cosmos.
1. Vaya a la sección de **información general**.
1. Compruebe el gráfico de **costos** en la parte inferior. En este gráfico se muestra una estimación del costo actual durante un período de tiempo configurable:
1. Cree un nuevo contenedor; por ejemplo, un contenedor de gráficos.
1. Especifique el rendimiento necesario para la carga de trabajo; por ejemplo, 400 RU/s. Después de especificar el valor del rendimiento, puede ver la estimación de precios, tal como se muestra en la siguiente captura de pantalla:

   :::image type="content" source="./media/plan-manage-costs/cost-estimate-portal.png" alt-text="Estimación de costos en Azure Portal":::

Si la suscripción de Azure tiene un límite de gasto, Azure le impide gastar por encima del importe del crédito. A medida que crea y usa recursos de Azure, se usan los créditos. Cuando alcanza el límite de crédito, los recursos que ha implementado se deshabilitan para el resto de ese período de facturación. No se puede cambiar el límite de crédito, pero sí puede quitarlo. Para más información sobre los límites de gasto, consulte [Límite de gasto de Azure](../cost-management-billing/manage/spending-limit.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Puede pagar los cargos de Azure Cosmos DB con el crédito del pago por adelantado de Azure (antes conocido como compromiso monetario). Sin embargo, no puede usar el crédito del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, incluidos los que proceden de Azure Marketplace.

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos con Azure Cosmos DB, se incurre en costos. Los costos de la unidad de uso de recursos varían en función de intervalos de tiempo (segundos, minutos, horas y días) o en función del uso de unidades de solicitud. En cuanto se inicia el uso de Azure Cosmos DB, se incurre en costos y puede verlos en el panel [Análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) de Azure Portal.

Al usar el análisis de costos, puede ver los costos de Azure Cosmos DB en gráficos y tablas para diferentes intervalos de tiempo. Algunos ejemplos son: por día, actual, mes anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Cambiar a vistas más largas en el tiempo puede ayudarle a identificar las tendencias de gasto y ver dónde podría haber ocurrido un gasto excesivo. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de Azure Cosmos DB en el análisis de costos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Abra el ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos.

1. De forma predeterminada, el costo de todos los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta "Azure Cosmos DB".

1. Para restringir los costos de un único servicio, como Azure Cosmos DB, seleccione **Agregar filtro** y, a continuación, seleccione **Nombre del servicio**. A continuación, elija **Azure Cosmos DB** en la lista. Este es un ejemplo que muestra solo los costos de Azure Cosmos DB:

   :::image type="content" source="./media/plan-manage-costs/cost-analysis-pane.png" alt-text="Supervisión de costos con el panel Análisis de costos":::

En el ejemplo anterior, hemos visto el costo actual de Azure Cosmos DB del mes de febrero. Los gráficos también contienen los costos de Azure Cosmos DB por ubicación y por grupo de recursos.

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos nuevos con un costo adicional. Para más información sobre las opciones de filtro al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando usted u otro usuario necesita hacer un análisis de datos adicional para los costos. Por ejemplo, un equipo de finanzas puede analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="other-ways-to-manage-and-reduce-costs"></a>Formas alternativas de administrar y reducir costos

A continuación se muestran algunos procedimientos recomendados que puede usar para reducir los costos:

* [Optimizar el costo del rendimiento aprovisionado](optimize-cost-throughput.md): en este artículo se explican detalladamente los procedimientos recomendados para optimizar el costo de rendimiento. Describe cuándo aprovisionar el rendimiento en el nivel de contenedor y cuándo hacerlo en el nivel de base de datos en función del tipo de carga de trabajo.

* [Optimizar el costo de solicitudes](optimize-cost-reads-writes.md): en este artículo se describe cómo se traducen las solicitudes de lectura y escritura en unidades de solicitud y cómo optimizar el costo de estas solicitudes.

* [Optimizar el costo de almacenamiento](optimize-cost-storage.md): el costo de almacenamiento se factura según el consumo. Aprenda a optimizar el costo de almacenamiento con el tamaño de los elementos y la directiva de indexación, mediante el uso de características como la fuente de cambios y el período de vida.

* [Optimizar el costo de varias regiones](optimize-cost-regions.md): si tiene una o varias regiones de lectura infrautilizadas, puede tomar medidas para optimizar el uso de las RU en las regiones de lectura mediante la fuente de cambios de la región de lectura, o bien moverlas a otra secundaria si se están sobreutilizando.

* [Optimizar el costo de desarrollo y pruebas](optimize-dev-test.md): obtenga información sobre cómo optimizar el costo de desarrollo mediante el emulador local, el nivel Gratis de Azure Cosmos DB, la cuenta gratuita de Azure y otras opciones.

* [Optimizar el costo con capacidad reservada](cosmos-db-reserved-capacity.md): obtenga información sobre cómo usar la capacidad reservada para ahorrar dinero mediante una reserva de recursos de Azure Cosmos DB durante un año o tres.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre cómo funcionan los precios en Azure Cosmos DB:

* [Modelo de precios de Azure Cosmos DB](how-pricing-works.md)
* Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
* Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
* Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
* Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).