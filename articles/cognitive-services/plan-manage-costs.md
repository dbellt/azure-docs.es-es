---
title: Planeamiento para administrar los costos de Azure Cognitive Services
description: Aprenda a planear y administrar los costos de Azure Cognitive Services mediante el análisis de costos de Azure Portal.
author: erhopf
ms.author: erhopf
ms.custom: subject-cost-optimization
ms.service: cognitive-services
ms.topic: how-to
ms.date: 12/15/2020
ms.openlocfilehash: d686ea40b08139b9d16ade16d6e401db8300db0f
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081126"
---
# <a name="plan-and-manage-costs-for-azure-cognitive-services"></a>Planeamiento y administración de los costos de Azure Cognitive Services

En este artículo se describe cómo puede planear y administrar los costos de Azure Cognitive Services. En primer lugar, antes de agregar recursos del servicio para calcular los costos, use la calculadora de precios de Azure para planear los costos de Cognitive Services. Después, a medida que agregue recursos de Azure, revise los costos estimados. Una vez que ha comenzado a usar los recursos de Cognitive Services (por ejemplo, Voz, Computer Vision, LUIS, Text Analytics, Translator, etc.), use las características de Cost Management para establecer presupuestos y supervisar los costos. También puede revisar los costos previstos e identificar las tendencias de gasto para identificar las áreas en las que podría querer actuar. Los costos de Cognitive Services son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo planear y administrar los costos de Cognitive Services, se le facturarán todos los servicios y recursos de Azure usados para su suscripción de Azure, incluidos los servicios de terceros.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos de Cost Management admite la mayoría de los tipos de cuenta de Azure, pero no todos. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

<!--Note for Azure service writer: If you have other prerequisites for your service, insert them here -->

## <a name="estimate-costs-before-using-cognitive-services"></a>Estimación de costos antes de usar Cognitive Services

Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de agregar Cognitive Services.

:::image type="content" source="media/cognitive-services-pricing-calculator.png" alt-text="Calculadora de precios de Azure para Cognitive Services" border="true":::

A medida que agregue nuevos recursos al área de trabajo, vuelva a esta calculadora y agregue el mismo recurso aquí para actualizar las estimaciones de costos.

Para más información, consulte [Precios de Azure Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).

## <a name="understand-the-full-billing-model-for-cognitive-services"></a>Modelo de facturación completo de Cognitive Services

Cognitive Services se ejecuta en la infraestructura de Azure que [genera los costos](https://azure.microsoft.com/pricing/details/cognitive-services/) al implementar el nuevo recurso. Es importante que sepa que cuanto mayor sea la infraestructura más costos se pueden generar. Estos costos deben administrarse cuando se realizan cambios en los recursos implementados. 

### <a name="how-youre-charged-for-cognitive-services"></a>¿Cómo se cobra Cognitive Services?

Al crear o usar recursos de Cognitive Services, es posible que se le cobrarán los medidores siguientes en función de los servicios que use:

| Servicio | Medidor(es) | Información de facturación | 
|---------|-------|---------------------|
| **Visión** | | |
| [Computer Vision](https://azure.microsoft.com/pricing/details/cognitive-services/computer-vision/) | Gratis, Estándar (S1) | Facturado por el número de transacciones. El precio por transacción varía en función de la característica (Lectura, OCR, Análisis espacial). Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/computer-vision/). |
| [Custom Vision](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) | Gratis, Estándar | <li>Las predicciones se facturan en función del número de transacciones.</li><li>El entrenamiento se factura por horas de proceso.</li><li>El almacenamiento de imágenes se factura por número de imágenes (hasta 6 MB por imagen).</li>|
| [Face](https://azure.microsoft.com/pricing/details/cognitive-services/face-api/) | Gratis, Estándar | Facturado por el número de transacciones. |
| **Voz** | | |
| [Servicio Voz](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) | Gratis, Estándar | La facturación varía según la característica (conversión de voz en texto, conversión de texto en voz, traducción de voz y reconocimiento del hablante). Principalmente, la facturación se hace por recuento de transacciones o recuento de caracteres. Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/). |
| **Lenguaje** | | |
| [Language Understanding (LUIS)](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/) | Creación gratuita, Predicción gratuita, Estándar | Se factura por el número de transacciones. El precio por transacción varía en función de la característica (solicitudes de voz, solicitudes de texto). Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/). |
| [QnA Maker](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/) | Gratis, Estándar | El precio de la suscripción se factura mensualmente. Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/). | 
| [Text Analytics](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) | Gratis, Estándar | Se factura por número de registros de texto. | 
| [Traductor](https://azure.microsoft.com/pricing/details/cognitive-services/translator/) | Gratis, pago por uso (S1), descuento por volumen (S2, S3, S4, C2, C3, C4, D3) | Los precios varían según el medidor y la característica. Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator/). <li>La traducción de texto se factura por número de caracteres traducidos.</li><li>La traducción de documentos se factura por número de caracteres traducidos.</li><li>La traducción personalizada se factura por caracteres de datos de entrenamiento de origen y destino.</li> |  
| **Decisión** | | |
| [Anomaly Detector](https://azure.microsoft.com/pricing/details/cognitive-services/anomaly-detector/) | Gratis, Estándar | Facturado por el número de transacciones. | 
| [Content Moderator](https://azure.microsoft.com/pricing/details/cognitive-services/content-moderator/) | Gratis, Estándar | Facturado por el número de transacciones. |
| [Personalizer](https://azure.microsoft.com/pricing/details/cognitive-services/personalizer/) | Gratis, Estándar (S0) | Se factura por transacciones al mes. Hay cuotas de almacenamiento y transacciones. Para conocer todos los detalles al respecto, consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/personalizer/). | 


### <a name="costs-that-typically-accrue-with-cognitive-services"></a>Costos que normalmente se generan con Cognitive Services

Normalmente, después de implementar un recurso de Azure, los costos vienen determinados por el plan de tarifa y las llamadas API que realice en el punto de conexión. Si el servicio que está usando tiene un nivel de compromiso, puede que se produzca un cargo por uso por encima del límite sobre las llamadas asignadas del nivel.

Se pueden generar costos adicionales al usar estos servicios:

#### <a name="qna-maker"></a>QnA Maker

Cuando se crean recursos para QnA Maker, también se pueden crear recursos para otros servicios de Azure. Incluyen:

- [Azure App Service (por el tiempo de ejecución)](https://azure.microsoft.com/pricing/details/app-service/)
- [Azure Cognitive Search (por los datos)](https://azure.microsoft.com/pricing/details/search/)
 
### <a name="costs-that-might-accrue-after-resource-deletion"></a>Costos que pueden generarse tras eliminar un recurso

#### <a name="qna-maker"></a>QnA Maker

Después de eliminar los recursos de QnA Maker, es posible que los siguientes recursos sigan existiendo. Estos recursos siguen generando costos hasta que se eliminan.

- [Azure App Service (por el tiempo de ejecución)](https://azure.microsoft.com/pricing/details/app-service/)
- [Azure Cognitive Search (por los datos)](https://azure.microsoft.com/pricing/details/search/)

### <a name="using-azure-prepayment-credit-with-cognitive-services"></a>Uso del crédito del pago por adelantado de Azure con Cognitive Services

Puede pagar los cargos de Cognitive Services con el crédito del pago por adelantado de Azure (antes conocido como compromiso monetario). Sin embargo, no puede usar el crédito del pago por adelantado de Azure para pagar los gastos de productos y servicios de terceros, como los que proceden de Azure Marketplace.

## <a name="monitor-costs"></a>Supervisión de costos

<!-- Note to Azure service writer: Modify the following as needed for your service. Replace example screenshots with ones taken for your service. If you need assistance capturing screenshots, ask banders for help. -->

A medida que se usan recursos de Azure con Cognitive Services, se incurre en costos. Los costos de unidad de uso de recursos de Azure varían según el intervalo de tiempo (segundos, minutos, horas y días) o el uso de unidades (bytes, megabytes, etc.). En cuanto se inicia el uso de Cognitive Services, se generan costos, que puede ver en el [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

Al usar el análisis de costos, puede ver los costos de Cognitive Services de diferentes intervalos de tiempo en gráficos y tablas. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Con el tiempo, cambiar a vistas más largas puede ayudarle a identificar las tendencias de gasto y comprobar dónde este se ha sobrepasado. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

Para ver los costos de Cognitive Services en el análisis de costos:

1. Inicie sesión en Azure Portal.
2. Abra el ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Seleccione **Ámbito** para cambiar a otro ámbito del análisis de costos.
3. De forma predeterminada, el costo de los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta Cognitive Services.

Los costos mensuales reales se muestran cuando se abre inicialmente el análisis de costos. Este es un ejemplo con todos los costos mensuales de uso.

:::image type="content" source="./media/cost-management/all-costs.png" alt-text="Ejemplo que muestra los costos acumulados de una suscripción":::

- Para limitar los costos a un único servicio, como Cognitive Services, seleccione **Agregar filtro** y luego **Nombre del servicio**. Después, seleccione **Cognitive Services**.

Este es un ejemplo en el que se muestran solo los costos de Cognitive Services.

:::image type="content" source="./media/cost-management/cognitive-services-costs.png" alt-text="Ejemplo en el que se muestran los costos acumulados para Cognitive Services":::

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y los de Cognitive Services por grupo de recursos. A partir de aquí, puede explorar los costos por su cuenta.

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) para administrar los costos y crear [alertas](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. 

Los presupuestos se pueden crear con filtros para recursos o servicios específicos de Azure si quiere disponer de más granularidad en la supervisión. Los filtros ayudan a garantizar que no se crean accidentalmente recursos, que suponen un mayor costo. Para más información sobre las opciones de filtro al crear un presupuesto, consulte [Opciones de agrupación y filtrado](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exportación de datos de costos

También puede [exportar los datos de costos](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) a una cuenta de almacenamiento. Esto resulta útil cuando se necesita hacer análisis de datos adicionales para los costos. Por ejemplo, los equipos de finanzas pueden analizar los datos con Excel o Power BI. Puede exportar los costos en una programación diaria, semanal o mensual y establecer un intervalo de fechas personalizado. La exportación de los datos de costos es la forma recomendada de recuperar conjuntos de datos de costos.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [optimizar su inversión en la nube con Azure Cost Management](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga más información sobre la administración de costos con los [análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Obtenga información sobre cómo [evitar los costos inesperados](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Haga el curso de aprendizaje guiado sobre [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).