---
title: Introducción a la cuenta de facturación de Azure actualizada
description: Empiece a usar su cuenta de facturación de Azure actualizada para conocer los cambios que se han producido en la nueva experiencia de administración de costos y facturación
author: bandersmsft
ms.reviewer: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: banders
ms.openlocfilehash: 4f7179a5ad35b4d3ca9a92119fb7b492e2aff779
ms.sourcegitcommit: 9f4510cb67e566d8dad9a7908fd8b58ade9da3b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106122533"
---
# <a name="get-started-with-your-updated-azure-billing-account"></a>Introducción a la cuenta de facturación de Azure actualizada

La administración de costos y facturas es uno de los componentes clave de la experiencia en la nube, ya que le ayuda a controlar y conocer el gasto que realiza en la nube. Para facilitar la administración de los costos y las facturas, Microsoft va a actualizar su cuenta de facturación de Azure con funcionalidades de facturación y administración de costos mejoradas. En este artículo se describen las actualizaciones que se han realizado en su cuenta de facturación y se explican las nuevas funcionalidades.

> [!IMPORTANT]
> Su cuenta se actualizará cuando reciba un correo electrónico de Microsoft en el que se le informa de las actualizaciones de su cuenta. Este correo se envía 60 días antes de que se actualice la cuenta.

## <a name="more-flexibility-with-your-new-billing-account"></a>Mayor flexibilidad con la nueva cuenta de facturación

En el diagrama siguiente se comparan la cuenta de facturación antigua y la nueva:

:::image type="content" source="./media/mosp-new-customer-experience/comparison-old-new-account.png" alt-text="Diagrama que muestra la comparación entre la jerarquía de facturación de la cuenta antigua y de la nueva." border="false" lightbox="./media/mosp-new-customer-experience/comparison-old-new-account.png":::

La nueva cuenta de facturación contiene uno o varios perfiles de facturación que permiten administrar las facturas y los métodos de pago. Cada perfil de facturación contiene una o más secciones de factura que permiten organizar los costos en la factura del perfil de facturación.

:::image type="content" source="./media/mosp-new-customer-experience/new-billing-account-hierarchy.png" alt-text="Diagrama que muestra la nueva jerarquía de facturación." border="false" lightbox="./media/mosp-new-customer-experience/new-billing-account-hierarchy.png":::

Los roles en la cuenta de facturación tienen el nivel más alto de permisos. Estos roles deben asignarse a los usuarios que necesitan ver las facturas y realizar el seguimiento de los costos de toda una cuenta, como los directores financieros o de TI de una organización, o al usuario que ha registrado una cuenta. Para más información, consulte [Tareas y roles de la cuenta de facturación](../manage/understand-mca-roles.md#billing-account-roles-and-tasks). Cuando se actualiza la cuenta, todo usuario que tenga un rol de administrador de cuenta en la cuenta de facturación antigua recibe un rol de propietario en la nueva cuenta.

## <a name="billing-profiles"></a>Perfiles de facturación

Los perfiles de facturación se usan para administrar facturas y métodos de pago. Una factura mensual se genera al comienzo del mes para cada perfil de facturación de la cuenta. La factura incluye los cargos respectivos del mes anterior de todas las suscripciones asociadas al perfil de facturación.

Cuando la cuenta se actualiza, se crea automáticamente un perfil de facturación para cada suscripción. Los cargos de la suscripción se facturan al perfil de facturación correspondiente y se muestran en su factura.

Los roles en los perfiles de facturación tienen permisos para ver y administrar las facturas y los métodos de pago. Esos roles se deben asignar a los usuarios que pagan facturas, como miembros del equipo de contabilidad de una organización. Para más información, consulte [Tareas y roles del perfil de facturación](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

Cuando se actualiza una cuenta, en todas las suscripciones en las que haya otorgado permisos a otros usuarios para [ver las facturas](download-azure-invoice.md#allow-others-to-download-your-subscription-invoice), a los usuarios que tengan un rol de Azure de propietario, colaborador, lector o lector de facturación se les concede el rol de lector en el perfil de facturación correspondiente.

## <a name="invoice-sections"></a>Secciones de la factura

Las secciones de la factura se usan para organizar los costos en la factura. Por ejemplo, puede que necesite una única factura, pero desea organizar los costos por departamento, equipo o proyecto. Para este escenario, cuenta con un único perfil de facturación en el que se crea una sección de factura para cada departamento, equipo o proyecto.

Cuando se actualiza la cuenta, se crea una sección de la factura para cada perfil de facturación y la suscripción relacionada se asigna a la sección de la factura. Cuando se agregan suscripciones, se pueden crear más secciones y asignar las suscripciones a las secciones de la factura. Las secciones se verán en la factura del perfil de facturación que refleja el uso de cada suscripción que les ha asignado.

Los roles en la sección de factura tienen permisos para controlar quién crea suscripciones a Azure. Los roles se deben asignar a los usuarios que configuren el entorno de Azure para los equipos de una organización, como los jefes de ingeniería y los arquitectos técnicos. Para más información, consulte [Tareas y roles de la sección de factura](../manage/understand-mca-roles.md#invoice-section-roles-and-tasks).

## <a name="enhanced-features-in-your-new-experience"></a>Características que han mejorado en la nueva experiencia

La nueva experiencia incluye las siguientes funcionalidades de administración de costos y facturación que facilitan la administración de los costos y las facturas:

#### <a name="invoice-management"></a>Administración de facturas

Un **período de facturación mensual más predecible**: en la nueva cuenta, el período de facturación comienza el primer día del mes y termina el último día del mes, independientemente del momento en que se registre para usar Azure. Al principio de cada mes se generará una factura que contendrá todos los cargos del mes anterior.

Una **única factura mensual para varias suscripciones**: en su cuenta existente, obtendrá una factura para cada suscripción de Azure. Cuando se actualiza la cuenta, se mantiene el comportamiento existente, pero tiene la opción de consolidar los cargos de las suscripciones en una sola factura. Una vez actualizada la cuenta, siga estos pasos para consolidar los cargos en una sola factura:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Busque **Administración de costos + facturación**.  
   ![Captura de pantalla que muestra la búsqueda en Azure Portal de Cost Management + Billing.](./media/mosp-new-customer-experience/billing-search-cost-management-billing.png)
3. Seleccione **Suscripciones de Azure** en el lado izquierdo de la pantalla. 
4. En la tabla se enumeran las suscripciones de Azure de pago. En la columna del perfil de facturación encontrará el perfil de facturación al que se factura la suscripción. Los cargos de suscripción se muestran en la factura del perfil de facturación. Para consolidar los cargos de todas las suscripciones en una sola factura, debe vincular todas las suscripciones a un único perfil de facturación.  
    :::image type="content" source="./media/mosp-new-customer-experience/list-azure-subscriptions.png" alt-text="Captura de pantalla que muestra la lista de suscripciones de Azure." lightbox="./media/mosp-new-customer-experience/list-azure-subscriptions.png" :::
5. Elija el perfil de facturación que quiera usar. 
6. Seleccione una suscripción que no esté vinculada al perfil de facturación que eligió en el paso 5. Haga clic en los puntos suspensivos (tres puntos) de la suscripción. Seleccione **Cambiar sección de factura**.  
    :::image type="content" source="./media/mosp-new-customer-experience/select-change-invoice-section.png" alt-text="Captura de pantalla que muestra la opción para cambiar la sección de factura." lightbox="./media/mosp-new-customer-experience/select-change-invoice-section-zoomed-in.png" :::
7. Seleccione el perfil de facturación que eligió en el paso 5.  
    :::image type="content" source="./media/mosp-new-customer-experience/change-invoice-section.png" alt-text="Captura de pantalla que muestra cómo cambiar la sección de la factura." lightbox="./media/mosp-new-customer-experience/change-invoice-section-zoomed-in.png" :::
8. Seleccione **Cambiar**.
9. Repita los pasos del 6 al 8 para todas las demás suscripciones. 

**Reciba una sola factura mensual con las suscripciones de Azure, los planes de soporte técnico y los productos de Azure Marketplace**: obtendrá una factura mensual con todos los cargos, incluidos los cargos de uso de las suscripciones de Azure, así como los planes de soporte técnico y las compras de Azure Marketplace.

**Agrupe los costos de la factura en función de las necesidades**: puede agrupar los costos de la factura por departamentos, proyectos o equipos.

**Establezca un número de pedido de compra opcional en la factura**: para asociar su factura a los sistemas financieros internos, establezca un número de orden de compra. Adminístrela y actualícela en cualquier momento desde Azure Portal.

#### <a name="payment-management"></a>Administración de pagos

**Pague facturas de inmediato mediante tarjeta de crédito**: no es preciso esperar a que se cargue el pago automático en su tarjeta de crédito. Puede usar cualquier tarjeta de crédito para pagar un vencimiento o una factura vencida desde Azure Portal.

**Realice un seguimiento de todos los pagos aplicados a la cuenta**: vea todos los pagos que se aplican a su cuenta; la información que se proporciona incluye el método de pago usado, el importe pagado y la fecha de pago en Azure Portal.

#### <a name="cost-management"></a>Administración de costos

**Programe operaciones mensuales de exportación de los datos de uso a una cuenta de almacenamiento**: publique automáticamente los datos de uso y los costos relativos a una cuenta de almacenamiento de forma diaria, semanal o mensual.

#### <a name="account-and-subscription-management"></a>Administración de cuentas y suscripciones

**Asigne varios administradores para realizar operaciones de facturación**: asigne permisos de facturación a varios usuarios para administrar la facturación de su cuenta. Disfrute de mayor flexibilidad proporcionando permisos de lectura, escritura, o ambos, a otros usuarios.

**Cree más suscripciones directamente en Azure Portal**: cree todas sus suscripciones con una sola selección en Azure Portal.

#### <a name="api-support"></a>Compatibilidad de API

**Realice operaciones de administración de costos y facturación mediante las API, SDK y PowerShell**: use las API de administración de costos, facturación y consumo para extraer datos de costos y facturación en las herramientas de análisis de datos que prefiera.

**Realice todas las operaciones de suscripción mediante las API, SDK y PowerShell**: use las API de suscripción de Azure para automatizar la administración de las suscripciones de Azure, lo que incluye la creación, el cambio de nombre y la cancelación de cualquier suscripción.

## <a name="get-prepared-for-your-new-experience"></a>Prepárese para la nueva experiencia

Siga estas recomendaciones para prepararse para su nueva experiencia:

**Período de facturación mensual y fecha de factura diferente**

En la nueva experiencia, la factura se generará en torno al noveno día de cada mes y contendrá todos los cargos del mes anterior. Esta fecha puede diferir de la fecha en que se generó la factura en la cuenta anterior. Si comparte las facturas con otras personas, notifique el cambio de fecha.


**Facturas del primer mes después de la migración**

El día en que se actualice la cuenta, se finalizarán los cargos no facturados existentes y recibirá las facturas de estos cargos el día que normalmente reciba las facturas. Por ejemplo, John tiene dos suscripciones de Azure: Azure sub 01, con un ciclo de facturación del día 5 del mes al día 4 del mes siguiente, y Azure sub 02, con un ciclo de facturación del día 10 del mes al día 9 del mes siguiente. John obtiene facturas de ambas suscripciones de Azure normalmente el día 5 del mes. Ahora, si la cuenta de John se actualiza el 4 de abril, los cargos de Azure sub 01 del 5 de marzo al 4 de abril y los cargos de Azure sub 02 del 10 de marzo al 4 de abril se finalizarán. John recibirá dos facturas, una para cada subscripción, el 5 de abril. Una vez actualizada la cuenta, el ciclo de facturación de John se basará en el mes natural y cubrirá todos los cargos incurridos desde el principio del mes natural hasta el final de este.  La factura de los cargos del mes natural anterior está disponible el día 9 de cada mes. Por lo tanto, en el ejemplo anterior, John recibirá otra factura el 5 de mayo para el período de facturación del 5 de abril al 30 de abril. 


**Nuevas API de administración de costos y facturación**

Si usa las API de administración de costos o facturación de Azure para consultar y actualizar los datos de facturación o de costos, debe usar las nuevas API. En la tabla siguiente se enumeran las API que no funcionarán con la nueva cuenta de facturación y los cambios que debe realizar en ella.

|API | Cambios  |
|---------|---------|
|[Billing Accounts - List](/rest/api/billing/2019-10-01-preview/billingaccounts/list) | En Billing Accounts - List API, la cuenta de facturación antigua tiene el valor **MicrosoftOnlineServiceProgram** en agreementType, mientras que la nueva tiene el valor **MicrosoftCustomerAgreement**. Si tiene una dependencia de agreementType, actualícela. |
|[Invoices - List By Billing Subscription](/rest/api/billing/2019-10-01-preview/invoices/listbybillingsubscription)     | Esta API solo devolverá las facturas que se generaron antes de que se actualizara la cuenta. Tendría que usar [Invoices - List By Billing Account](/rest/api/billing/2019-10-01-preview/invoices/listbybillingaccount) API para obtener las facturas que se generan en la nueva cuenta de facturación. |


## <a name="cost-management-updates-after-account-update"></a>Actualizaciones de Cost Management después de la actualización de la cuenta

Su cuenta de facturación de Azure actualizada del Contrato de cliente de Microsoft le proporciona acceso a nuevas y mejoradas experiencias de Cost Management en Azure Portal que no tenía con su cuenta de pago por uso.

### <a name="new-capabilities"></a>Funcionalidades nuevas

Las siguientes nuevas funcionalidades están disponibles con su cuenta de facturación de Azure.

#### <a name="new-billing-scopes"></a>Nuevos ámbitos de facturación

Como parte de la cuenta actualizada, tiene nuevos ámbitos en Cost Management + Billing. Además de ayudar a la organización jerárquica y la facturación, también son una manera de ver los cargos combinados de varias suscripciones subyacentes. Para más información acerca de los ámbitos de facturación, consulte [Ámbitos de contrato de cliente de Microsoft](../costs/understand-work-scopes.md#microsoft-customer-agreement-scopes).

También puede acceder a las API de Cost Management para obtener vistas combinadas de costos en ámbitos superiores. Todas las API de Cost Management que utilizan el ámbito de la suscripción siguen estando disponibles con algunos cambios menores en el esquema. Para más información sobre las API, consulte [API de Azure Cost Management](/rest/api/cost-management/) y [API de consumo de Azure](/rest/api/consumption/).

#### <a name="cost-allocation"></a>Asignación de costos

Con su cuenta actualizada, puede usar las funcionalidades de asignación de costos para distribuir los costos de los servicios compartidos de su organización. Para más información sobre la asignación de costos, consulte [Creación y administración de reglas de asignación de costos de Azure](../costs/allocate-costs.md).

#### <a name="power-bi"></a>Power BI

El conector de Azure Cost Management para Power BI Desktop le ayuda a crear visualizaciones e informes personalizados sobre el uso y los gastos de Azure. Tendrá acceso a los datos de uso y costo después de conectarse a la cuenta actualizada. Para más información sobre el conector de Azure Cost Management para Power BI Desktop, consulte [Creación de objetos visuales e informes con el conector Azure Cost Management de Power BI Desktop](/power-bi/connect-data/desktop-connect-azure-cost-management).

### <a name="updated-capabilities"></a>Funcionalidades actualizadas

Las siguientes funcionalidades actualizadas están disponibles con su cuenta de facturación de Azure.

#### <a name="cost-analysis"></a>Análisis de costos

Puede seguir viendo y realizando un seguimiento de los costos de consumo mes a mes y ahora también puede ver los costos de compra de reservas y de Marketplace en el análisis de costos.

Con la cuenta actualizada, recibirá una sola factura con todos los cargos de Azure. Ahora también tiene una vista de calendario mensual simplificada que reemplaza a la vista de períodos de facturación que tenía anteriormente.

Por ejemplo, si el período de facturación era del 24 de noviembre al 23 de diciembre para la cuenta anterior, después de la actualización el período pasará a ser del 1 de noviembre al 30 de noviembre, del 1 de diciembre al 31 de diciembre y así sucesivamente.

:::image type="content" source="./media/mosp-new-customer-experience/billing-periods.png" alt-text="Captura de pantalla que muestra una comparación entre los períodos de facturación antiguos y los nuevos." lightbox="./media/mosp-new-customer-experience/billing-periods.png" :::

#### <a name="budgets"></a>Presupuestos

Ahora puede crear presupuestos para la cuenta de facturación, lo que le permite realizar un seguimiento de los costos entre las suscripciones. También puede estar al día en cuanto a la información sobre los cargos de compra mediante los presupuestos. Para más información sobre los presupuestos, consulte [Creación y administración de presupuestos de Azure](../costs/tutorial-acm-create-budgets.md).

#### <a name="exports"></a>Exports

La nueva cuenta de facturación proporciona la funcionalidad de exportación mejorada. Por ejemplo, puede crear exportaciones para costos reales que incluyen compras o costos amortizados (los costos de compra de reserva se reparten a lo largo del plazo de compra). También puede crear una exportación para la cuenta de facturación para obtener los datos de uso y cargos de todas las suscripciones de la cuenta de facturación. Para más información acerca de las exportaciones, consulte [Creación y administración de datos exportados](../costs/tutorial-export-acm-data.md).

> [!NOTE]
> Las exportaciones creadas antes de la actualización de la cuenta con el tipo **Monthly export of last month's costs** (Exportación mensual de los costos del mes pasado) exportarán los datos del último mes natural, no los del último período de facturación.

Por ejemplo, para un período de facturación del 23 de diciembre al 22 de enero, el archivo CSV exportado incluiría los datos de costo y uso para ese período. Después de la actualización, la exportación contendrá datos los datos del mes natural. Por ejemplo, del 1 de enero al 31 de enero, etc.

:::image type="content" source="./media/mosp-new-customer-experience/export-amortized-costs.png" alt-text="Captura de pantalla que muestra una comparación entre los detalles de exportación nuevos y los antiguos." lightbox="./media/mosp-new-customer-experience/export-amortized-costs.png" :::

## <a name="additional-information"></a>Información adicional

En las siguientes secciones se proporciona más información sobre su nueva experiencia.

**Sin tiempo de inactividad en los servicios** Los servicios de Azure de la suscripción se seguirán ejecutando sin ninguna interrupción. Lo único que se actualiza es su experiencia de facturación. Los recursos existentes, los grupos de recursos o los grupos de administración no se verán afectados.

**Sin cambios en los recursos de Azure** El acceso a los recursos de Azure que se establecieron mediante el control de acceso basado en rol (Azure RBAC ) no se ve afectado por la actualización.

**Las facturas anteriores están disponibles en la nueva experiencia** Las facturas generadas antes de que se actualizara la cuenta siguen estando disponibles en Azure Portal.

**Las facturas de la cuenta se actualizan a mediados de mes** Si su cuenta se actualiza a mediados de mes, recibirá una factura por los cargos acumulados hasta el día en que se realice dicha actualización. Recibirá otra factura por los cargos del resto del mes. Por ejemplo, su cuenta tiene una suscripción y se actualiza el 15 de septiembre. Recibirá una factura por los cargos acumulados hasta el 15 de septiembre. Y recibirá otra factura por los cargos acumulados durante el período comprendido entre el 15 y el 30 de septiembre. Después de septiembre, recibirá una factura al mes.

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre su cuenta de facturación, consulte los siguientes artículos.

- [Descripción de los roles administrativos en la nueva cuenta de facturación](../manage/understand-mca-roles.md)
- [Creación de una suscripción de Azure adicional para la cuenta de facturación](../manage/create-subscription.md)
- [Creación de secciones en la factura para organizar los costos](../manage/mca-section-invoice.md)