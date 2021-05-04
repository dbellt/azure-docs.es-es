---
title: Asignación de costos de Azure
description: En este artículo se explica cómo crear reglas de asignación de costos para distribuir los costos de las suscripciones, grupos de recursos o etiquetas a otros usuarios.
author: bandersmsft
ms.author: banders
ms.date: 03/23/2021
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: benshy
ms.openlocfilehash: e7afef7e0a10bb4be3c30112fc207467167e4a17
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726527"
---
# <a name="create-and-manage-azure-cost-allocation-rules-preview"></a>Creación y administración de reglas de asignación de costos de Azure (versión preliminar)

A menudo, las grandes empresas tienen servicios o recursos de Azure que se administran centralmente, pero que los usan diferentes departamentos internos o unidades de negocio. Lo habitual es que el equipo que los administra de forma centralizada quiera repercutir el costo de los servicios compartidos a los departamentos internos o a las unidades de negocio de la organización que los usan de forma activa. Este artículo le ayudará no solo a conocer el concepto de asignación de costos en Azure Cost Management, sino también a usar dicha asignación.

Con la asignación de costos, es posible reasignar o distribuir los costos de los servicios compartidos de unas suscripciones, grupos de recursos o etiquetas a otras suscripciones, grupos de recursos o etiquetas de la organización. La asignación de costos traslada los costos de los servicios compartidos a otra suscripción, otros grupos de recursos u otras etiquetas, que son propiedad de los departamentos internos o las unidades de negocio que los consumen. En otras palabras, la asignación de costos ayuda a administrar y mostrar la _responsabilidad de los costos_ de un lugar en otro.

La asignación de costos no afecta a su facturación. Las responsabilidades de facturación no cambian. El objetivo principal de la asignación de costos es ayudarle a repercutir los costos a otros usuarios. Todos los procesos de contracargo se producen en su organización fuera de Azure. La asignación de costos le ayuda a repercutir los costos, ya que los muestra a medida que se reasignan o distribuyen.

Los costos asignados se muestran en el análisis de costos. Se muestran como elementos adicionales asociados a las suscripciones de destino, los grupos de recursos o las etiquetas que se especifiquen al crear una regla de asignación de costos.

> [!NOTE]
> La característica de asignación de costos de Azure Cost Management está actualmente en versión preliminar pública. Es posible que algunas características de Cost Management no sean compatibles o que sus funcionalidades estén limitadas.

## <a name="prerequisites"></a>Requisitos previos

- En la actualidad, la asignación de costos solo pueden usarla los clientes con un [Contrato de cliente de Microsoft](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) o un [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/).
- Para crear o administrar una regla de asignación de costos, se debe usar una cuenta de administrador de empresa para los [Contratos Enterprise](../manage/understand-ea-roles.md). O bien se debe ser propietario de una [cuenta de facturación](../manage/understand-mca-roles.md) para los Contratos de cliente de Microsoft.

## <a name="create-a-cost-allocation-rule"></a>Creación de una regla de asignación de costos

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).
2. Vaya a **Administración de costos + facturación** > **Administración de costos**.
3. En **Configuración** > **Configuración**, seleccione **Asignación de costos (versión preliminar)** .
4. Asegúrese de que se selecciona la cuenta de facturación o inscripción de EA correctas.
5. Seleccione **+Agregar**.
6. Escriba un texto descriptivo para el nombre de la regla de asignación de costos.

:::image type="content" source="./media/allocate-costs/rule-name.png" alt-text="Ejemplo que muestra la creación de un nombre de regla" lightbox="./media/allocate-costs/rule-name.png" :::

La fecha de inicio de la evaluación de la regla se usa para generar porcentajes de asignación de costos que se rellenan previamente.

1. Seleccione **Agregar orígenes** y, después, seleccione las suscripciones, los grupos de recursos o las etiquetas en la que se eligen los costos que se van a distribuir.
2. Seleccione **Agregar destinos** y, después, seleccione las suscripciones, los grupos de recursos o las etiquetas que van a recibir los costos asignados.
3. Si necesita crear más reglas de asignación de costos, repita este proceso.

## <a name="configure-the-allocation-percentage"></a>Configuración del porcentaje de asignación

Configure el porcentaje de asignación para definir la forma en que los costos se dividen proporcionalmente entre los destinos especificados. Puede definir de forma manual los porcentajes de números enteros para crear una regla de asignación. O bien, puede dividir los costos de manera proporcional en función del uso actual del proceso, almacenamiento o red entre los destinos especificados.

Al distribuir los costos por costo de proceso, costo del almacenamiento o costo de red, el porcentaje proporcional se deduce mediante la evaluación de los costos del destino seleccionado. Los costos se asocian con el tipo de recurso del mes de facturación actual.

Al distribuir los costos de forma proporcional al costo total, el porcentaje proporcional se asigna por la suma o costo total de los destinos seleccionados en el mes de facturación actual.

:::image type="content" source="./media/allocate-costs/cost-distribution.png" alt-text="Ejemplo que muestra el porcentaje de asignación" lightbox="./media/allocate-costs/cost-distribution.png" :::

Una vez que se establecen, los porcentajes rellenados previamente se quedan fijos. Se usan para todas las asignaciones en curso. Los porcentajes solo cambian cuando la regla se actualiza manualmente.

1. Seleccione una de las siguientes opciones en la lista **Prerrellenar porcentaje**.
    - **Distribuir uniformemente**: cada uno de los destinos recibe una proporción igual del costo total.
    - **Costo total**: crea una relación proporcional a los destinos en función de su costo total. La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de proceso**: crea una relación proporcional a los destinos en función de su costo de proceso de Azure (los tipos de recursos del espacio de nombres [Microsoft.Compute](/azure/templates/microsoft.compute/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de almacenamiento**: crea una relación proporcional a los destinos en función de su costo de proceso de Azure (los tipos de recursos del espacio de nombres [Microsoft.Storage](/azure/templates/microsoft.storage/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de red**: crea una relación proporcional a los destinos en función de su costo de red de Azure (los tipos de recursos del espacio de nombres [Microsoft.Network](/azure/templates/microsoft.network/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Personalizado**: permite especificar manualmente un porcentaje de un número entero. El total especificado debe sumar un 100 %.
1. Cuando la regla se haya configurado, seleccione **Crear**.

La regla de asignación empieza a procesarse. Cuando la regla está activa, todos los costos del origen seleccionado se asignan a los destinos especificados.

> [!NOTE] 
> El procesamiento de la regla nueva puede tardar hasta dos horas en completarse y en que la regla esté activa.

## <a name="verify-the-cost-allocation-rule"></a>Verificación de la regla de asignación de costos

Cuando la regla de asignación de costos está activa, los costos de los orígenes seleccionados se distribuyen a los destinos de asignación especificados. Use la siguiente información para verificar que el costo se ha asignado correctamente a los destinos.

### <a name="view-cost-allocation-for-a-subscription"></a>Visualización de la asignación de costos de una suscripción

Vea el impacto de la regla de asignación en el análisis de costos. En Azure Portal, vaya a [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). Seleccione en la lista una suscripción que sea el destino de una regla de asignación de costos activa. Luego seleccione **Análisis de costos** en el menú. En Análisis de costos, seleccione **Agrupar por** y luego seleccione **Asignación de costos**. La vista resultante muestra un rápido desglose de costos generado por la suscripción. También se muestran los costos asignados a la suscripción, como en la siguiente imagen.

:::image type="content" source="./media/allocate-costs/cost-breakdown.png" alt-text="Ejemplo que muestra el desglose de costos" lightbox="./media/allocate-costs/cost-breakdown.png" :::

### <a name="view-cost-allocation-for-a-resource-group"></a>Visualización de la asignación de costos de un grupo de recursos

Use un proceso similar para ver el impacto de una regla de asignación de costos en un grupo de recursos. En Azure Portal, vaya a [Grupo de recursos](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups). Seleccione un grupo de recursos en la lista una suscripción que sea el destino de una regla de asignación de costos activa. Luego seleccione **Análisis de costos** en el menú. En Análisis de costos, seleccione **Agrupar por** y luego seleccione **Asignación de costos**. La vista muestra un rápido desglose de costos generado por el grupo de recursos. También se muestra el costo asignado al grupo de recursos.

### <a name="view-cost-allocation-for-tags"></a>Visualización de la asignación de costos de etiquetas

En Azure Portal, vaya a **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**. En Análisis de costos, seleccione **Agregar filtro**. Seleccione **Etiqueta**, elija la clave de la etiqueta y los valores de la etiqueta a los que se haya asignado algún costo.

:::image type="content" source="./media/allocate-costs/tagged-costs.png" alt-text="Ejemplo que muestra los costos de los elementos con etiqueta" lightbox="./media/allocate-costs/tagged-costs.png" :::

Este es un vídeo que muestra cómo crear una regla de asignación de costos.

>[!VIDEO https://www.youtube.com/embed/nYzIIs2mx9Q]


## <a name="edit-an-existing-cost-allocation-rule"></a>Edición de una regla de asignación existente

Puede editar una regla de asignación de costos para cambiar el origen o el destino, si desea actualizar el porcentaje rellenado en las opciones de proceso, almacenamiento o red. Las reglas se editan de la misma forma que se crean. La modificación de las reglas existentes puede tardar hasta dos horas en reprocesarse.

## <a name="current-limitations"></a>Limitaciones actuales

Actualmente, la asignación de costos se admite en Cost Management por análisis de costo, presupuesto y vistas de previsión. Los costos asignados también se muestran en la lista de suscripciones y en la página de información general de Suscripciones.

Los elementos siguientes no son compatibles actualmente con la versión preliminar pública de asignación de costos:

- [Exportaciones](tutorial-export-acm-data.md) programadas
- Datos que expone [Usage Details](/rest/api/consumption/usagedetails/list) API
- Área de suscripciones de facturación
- [Aplicación Azure Cost Management para Power BI](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp)
- [Conector de Power BI Desktop](/power-bi/connect-data/desktop-connect-azure-cost-management)


## <a name="next-steps"></a>Pasos siguientes

- Lea las [preguntas frecuentes de Cost Management + Billing](../cost-management-billing-faq.yml) para más información sobre la asignación de costos.
- Cree o actualice reglas de asignación mediante la [API REST de asignación de costos](/rest/api/cost-management/costallocationrules)
- Obtenga más información sobre la [optimización de la inversión en la nube con Azure Cost Management](cost-mgt-best-practices.md)