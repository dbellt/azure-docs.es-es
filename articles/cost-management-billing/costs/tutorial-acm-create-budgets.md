---
title: 'Tutorial: Creación y administración de presupuestos de Azure'
description: Este tutorial le ayuda a planear y tener en cuenta los costos de los servicios de Azure que usted consume.
author: bandersmsft
ms.author: banders
ms.date: 04/26/2021
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18
ms.openlocfilehash: 4669edd4b812eaf1dba4324c46ba97ab71392592
ms.sourcegitcommit: c1b0d0b61ef7635d008954a0d247a2c94c1a876f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109628273"
---
# <a name="tutorial-create-and-manage-azure-budgets"></a>Tutorial: Creación y administración de presupuestos de Azure

Los presupuestos en Cost Management le ayudan a planear y dirigir la presentación de cuentas de la organización. Le ayudan a informar a otros usuarios sobre sus gastos a fin de administrar de manera proactiva los costos y supervisar cómo avanza el gasto a lo largo del tiempo. Puede configurar alertas en función del costo real o el costo previsto para asegurarse de que el gasto se incluye en los márgenes de la organización. Cuando se superan los umbrales presupuestarios que ha creado, solo se desencadenan las notificaciones. Ninguno de los recursos se ve afectado y no se detiene el consumo. Puede usar los presupuestos para comparar y realizar un seguimiento de gastos para analizar los costos.

Normalmente, la información sobre los costos y el uso está disponible en un plazo que oscila entre 8 y 24 horas, y los presupuestos se evalúan con arreglo a estos costos cada 24 horas. Asegúrese de familiarizarse con la información específica de [Actualizaciones de los datos de uso y costo](./understand-cost-mgt-data.md#cost-and-usage-data-updates-and-retention). Normalmente, cuando se alcanza el umbral del presupuesto, las notificaciones por correo electrónico se envían en el plazo de una hora de la evaluación.

Los presupuestos se restablecen automáticamente al final de un período (mensual, trimestral o anualmente) para el mismo importe presupuestario al seleccionar una fecha de expiración futura. Dado que se restablecen con el mismo importe presupuestario, deberá crear presupuestos independientes cuando los importes presupuestarios en moneda difieran para períodos futuros. Cuando un presupuesto expira, se elimina automáticamente.

Los ejemplos de este tutorial le guiarán a través de la creación y edición de un presupuesto para una suscripción de Contrato Enterprise (EA) de Azure.

Vea el vídeo [Aplicar presupuestos a las suscripciones mediante Azure Portal](https://www.youtube.com/watch?v=UrkHiUx19Po) si desea ver cómo puede crear presupuestos en Azure para supervisar los gastos. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/UrkHiUx19Po]

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un presupuesto en Azure Portal
> * Creación y edición de presupuestos con PowerShell
> * Crear un presupuesto con una plantilla de Azure Resource Manager

## <a name="prerequisites"></a>Requisitos previos

Se admiten los presupuestos para los siguientes tipos de cuentas y ámbitos de Azure:

- Ámbitos del control de acceso basado en roles de Azure (RBAC de Azure)
    - Grupos de administración
    - Subscription
- Ámbitos del Contrato Enterprise
    - Cuenta de facturación
    - department
    - Cuenta de inscripción
- Acuerdos individuales
    - Cuenta de facturación
- Ámbitos de contrato de cliente de Microsoft
    - Cuenta de facturación
    - Perfil de facturación
    - Sección de factura
    - Customer
- Ámbitos de AWS
    - Cuenta externa
    - Suscripción externa

Para ver los presupuestos, se necesita al menos acceso de lectura en la cuenta de Azure.

Si tiene una suscripción nueva, no puede crear un presupuesto ni usar las características de Cost Management de inmediato. Para poder hacerlo deberán transcurrir un máximo de 48 horas.

En el caso de las suscripciones con contrato Enterprise de Azure, debe tener acceso de lectura para ver los presupuestos. Para crear y administrar presupuestos, debe tener permiso de colaborador.

Se admiten los siguientes permisos o ámbitos de Azure por suscripción para los presupuestos por usuario y grupo.

- Propietario: puede crear, modificar o eliminar los presupuestos para una suscripción.
- Colaborador y Colaborador de Cost Management: puede crear, modificar o eliminar sus propios presupuestos. Puede modificar el importe presupuestario para los presupuestos creados por otros usuarios.
- Lector y Lector de Cost Management: puede ver los presupuestos para los que tiene permiso.

**Para más información sobre los ámbitos, incluido el acceso necesario para configurar las exportaciones de Contrato Enterprise y los ámbitos del contrato de cliente de Microsoft, consulte [Descripción y uso de ámbitos](understand-work-scopes.md)** . Para más información sobre cómo asignar permisos a los datos de Cost Management, consulte [Asignación del acceso a los datos de Cost Management](./assign-access-acm-data.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

- Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-budget-in-the-azure-portal"></a>Crear un presupuesto en Azure Portal

Puede crear un presupuesto de suscripción de Azure durante un período mensual, trimestral o anual.

Para crear o ver un presupuesto, abra el ámbito en Azure Portal y seleccione **Presupuestos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y elija **Presupuestos** en el menú. Use la píldora **Ámbito** para cambiar a un ámbito diferente, como un grupo de administración, en los presupuestos. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

Después de crear los presupuestos, muestran a un lado una vista sencilla de su gasto actual.

Seleccione **Agregar**.

:::image type="content" source="./media/tutorial-acm-create-budgets/budgets-cost-management.png" alt-text="Captura de pantalla que muestra una lista de presupuestos ya creada." lightbox="./media/tutorial-acm-create-budgets/budgets-cost-management.png" :::

En la ventana **Crear presupuesto**, asegúrese de que el ámbito que se muestra es correcto. Elija los filtros que quiera agregar. Los filtros le permiten crear presupuestos en para costos específicos, como los grupos de recursos de una suscripción o un servicio, como las máquinas virtuales. Todos los filtros que puede usar en el análisis de costos también se pueden aplicar a un presupuesto.

Después de haber identificado el ámbito y los filtros, escriba un nombre de presupuesto. A continuación, elija un período de restablecimiento mensual, trimestral o anual. El período de restablecimiento determina el período de tiempo analizado en el presupuesto. El costo evaluado por el presupuesto comienza en cero al principio de cada nuevo período. Cuando crea un presupuesto trimestral, funciona de la misma manera que un presupuesto mensual. La diferencia es que el importe presupuestario para el trimestre se divide de manera uniforme entre los tres meses del trimestre. Un importe presupuestario anual se divide de manera uniforme entre los 12 meses del año natural.

Si tiene una suscripción de pago por uso, MSDN o Visual Studio, es posible que el período de facturación no esté alineado con el mes natural. En el caso de esos tipos de suscripciones y grupos de recursos, puede crear un presupuesto que se adapte al período de su factura o a los meses naturales. Para crear un presupuesto adaptado al período de facturación, seleccione el período de restablecimiento **Mes de facturación**, **Trimestre de facturación** o **Año de facturación**. Para crear un presupuesto adaptado al mes natural, seleccione un período de restablecimiento **Mensual**, **Trimestral** o **Anual**.

A continuación, identifique la fecha de expiración en la que presupuesto dejará de ser válido y ya no evaluará los costos.

En función de los campos elegidos en el presupuesto hasta el momento, se muestra un gráfico para ayudarle a seleccionar el umbral que se usará para el presupuesto. El presupuesto sugerido se basa en el costo pronosticado más alto que podría producirse en períodos futuros. Puede cambiar la cantidad del presupuesto.

:::image type="content" source="./media/tutorial-acm-create-budgets/create-monthly-budget.png" alt-text="Captura de pantalla que muestra la creación de un presupuesto con datos de costos mensuales." lightbox="./media/tutorial-acm-create-budgets/create-monthly-budget.png" :::

Después de configurar el importe del presupuesto, seleccione **Siguiente** para configurar las alertas de presupuesto para el costo real y el previsto.

## <a name="configure-actual-costs-budget-alerts"></a>Configuración de alertas de presupuesto de costos reales

Los presupuestos requieren al menos un umbral de costos (% del presupuesto) y una dirección de correo electrónico correspondiente. De manera opcional, puede incluir hasta cinco umbrales y cinco direcciones de correo electrónico en un único presupuesto. Normalmente, cuando se alcanza el umbral del presupuesto, las notificaciones por correo electrónico se envían en el plazo de una hora después de la evaluación. Las alertas de presupuesto de costos reales se generan para el costo real acumulado en relación con los umbrales de presupuesto configurados.

## <a name="configure-forecasted-budget-alerts"></a>Configuración de alertas de presupuesto de costos previstos

Las alertas de costos previstos proporcionan una notificación avanzada que indica que la tendencia de gasto podría superar el presupuesto. Las alertas usan [predicciones de costos previstos](quick-acm-cost-analysis.md#understand-forecast). Las alertas se generan cuando la proyección de costos previstos supera el umbral establecido. El umbral previsto (% del presupuesto) se puede configurar. Normalmente, cuando se alcanza, se envían notificaciones por correo electrónico una hora después de la evaluación.

Para alternar entre configurar una alerta de costo real o de previsión, use el campo `Type` al configurar la alerta como se muestra en la siguiente imagen.

Si desea recibir correos electrónicos, agregue azure-noreply@microsoft.com a la lista de remitentes aprobados, con el fin de que los correos electrónicos no vayan a la carpeta de correo no deseado. Para más información acerca de las notificaciones, consulte [Use cost alerts](./cost-mgt-alerts-monitor-usage-spending.md) (Uso de alertas de costos).

En el siguiente ejemplo se genera una alerta por correo electrónico cuando se alcanza el 90 % del presupuesto. Si crea un presupuesto con API Budgets, también puede asignar roles a los usuarios para que reciban alertas. No se admite la asignación de roles a personas en Azure Portal. Para más información sobre la API de presupuestos de Azure, consulte [API Budgets](/rest/api/consumption/budgets). Si desea recibir una alerta por correo electrónico en otro idioma, consulte [Configuraciones regionales admitidas para los correos electrónicos de alertas de presupuesto](manage-automation.md#supported-locales-for-budget-alert-emails).

Los límites de alerta admiten un intervalo de 0,01 a 1000 % del umbral del presupuesto proporcionado.

:::image type="content" source="./media/tutorial-acm-create-budgets/budget-set-alert.png" alt-text="Captura de pantalla que muestra las condiciones de alerta." lightbox="./media/tutorial-acm-create-budgets/budget-set-alert.png" :::

Después de crear un presupuesto, se muestra en el análisis de costos. Ver el presupuesto con respecto a la tendencia del gasto es uno de los primeros pasos para empezar a [analizar los costos y gastos](./quick-acm-cost-analysis.md).

:::image type="content" source="./media/tutorial-acm-create-budgets/cost-analysis.png" alt-text="Captura de pantalla que muestra un presupuesto de ejemplo con los gastos indicados en el análisis de costos." lightbox="./media/tutorial-acm-create-budgets/cost-analysis.png" :::

En el ejemplo anterior, creó un presupuesto para una suscripción. También puede crear un presupuesto para un grupo de recursos. Si quiere crear un presupuesto para un grupo de recursos, vaya a **Cost Management + Billing** &gt; **Suscripciones** &gt; seleccione una suscripción > **Grupos de recursos** > seleccione un grupo de recursos > **Presupuestos** > y **Agregar** un presupuesto.

### <a name="create-a-budget-for-combined-azure-and-aws-costs"></a>Creación de un presupuesto para los costos combinados de Azure y AWS

Para agrupar los costos de Azure y AWS, asigne un grupo de administración al conector junto con las cuentas consolidadas y vinculadas. Asigne las suscripciones de Azure al mismo grupo de administración. A continuación, cree un presupuesto para los costos combinados.

1. En Cost Management, seleccione **Presupuestos**.
1. Seleccione **Agregar**.
1. Seleccione **Cambiar ámbito** y, a continuación, seleccione el grupo de administración.
1. Siga creando el presupuesto hasta que finalice.

## <a name="costs-in-budget-evaluations"></a>Costos de las evaluaciones de presupuesto

Las evaluaciones de costos del presupuesto ahora incluyen datos de instancias reservadas y de compras. Si se le aplican cargos, es posible que reciba alertas a medida que se incorporen cargos a las evaluaciones. Se recomienda iniciar sesión en [Azure Portal](https://portal.azure.com) para verificar que los umbrales del presupuesto estén configurados correctamente para considerar los nuevos costos. Los cargos facturados de Azure no cambian. Los presupuestos ahora se evalúan frente a un conjunto más completo de costos. Si no se le aplican los cargos, el comportamiento del presupuesto permanece inalterado.

Si desea filtrar los nuevos costos de modo que los presupuestos se evalúen únicamente frente a los cargos de consumo propio de Azure, agregue los siguientes filtros al presupuesto:

- Tipo de publicador: Azure
- Tipo de cargo: Uso

Las evaluaciones de los costos del presupuesto se basan en el costo real. No incluyen la amortización. Para obtener más información sobre las opciones de filtrado disponibles en los presupuestos, consulte [Descripción de las opciones de agrupación y filtrado](group-filter.md).

## <a name="trigger-an-action-group"></a>Activación de un grupo de acciones

Al crear o editar un presupuesto para un ámbito de suscripción o grupo de recursos, puede configurarlo para que llame a un grupo de acciones. El grupo de acciones puede realizar varias acciones cuando se alcanza el umbral del presupuesto. 

Los grupos de acciones solo se admiten actualmente para ámbitos de suscripción y de grupo de recursos. Para más información sobre la creación de grupos de acciones, consulte [Configuración básica del grupo de acciones](../../azure-monitor/alerts/action-groups.md#configure-basic-action-group-settings). 

Para obtener más información sobre el uso de la automatización basada en presupuestos con grupos de acciones, vea [Administración de costos con presupuestos de Azure](../manage/cost-management-budget-scenario.md).

Para crear o actualizar los grupos de acciones, seleccione **Administrar grupo de acciones** al crear o editar un presupuesto.

:::image type="content" source="./media/tutorial-acm-create-budgets/trigger-action-group.png" alt-text="Captura de pantalla que muestra un ejemplo de creación de un presupuesto para mostrar la opción Administrar grupos de acciones." lightbox="./media/tutorial-acm-create-budgets/trigger-action-group.png" :::

Después, seleccione **Agregar grupo de acciones** y cree el grupo de acciones.

La integración de presupuesto con los grupos de acciones solo funciona para los grupos de acciones que tienen deshabilitado el esquema de alertas común. Para más información sobre cómo deshabilitar el esquema, consulte [¿Cómo habilito el esquema de alertas comunes?](../../azure-monitor/alerts/alerts-common-schema.md#how-do-i-enable-the-common-alert-schema)

## <a name="create-and-edit-budgets-with-powershell"></a>Creación y edición de presupuestos con PowerShell

Los clientes de EA pueden crear y editar los presupuestos mediante programación con el módulo de Azure PowerShell. Para descargar la versión más reciente de Azure PowerShell ejecute el siguiente comando:

```azurepowershell-interactive
install-module -name Az
```

Los comandos del siguiente ejemplo crean un presupuesto.

```azurepowershell-interactive
#Sign into Azure Powershell with your account

Connect-AzAccount

#Select a subscription to to monitor with a budget

select-AzSubscription -Subscription "Your Subscription"

#Create an action group email receiver and corresponding action group

$email1 = New-AzActionGroupReceiver -EmailAddress test@test.com -Name EmailReceiver1
$ActionGroupId = (Set-AzActionGroup -ResourceGroupName YourResourceGroup -Name TestAG -ShortName TestAG -Receiver $email1).Id

#Create a monthly budget that sends an email and triggers an Action Group to send a second email. Make sure the StartDate for your monthly budget is set to the first day of the current month. Note that Action Groups can also be used to trigger automation such as Azure Functions or Webhooks.

New-AzConsumptionBudget -Amount 100 -Name TestPSBudget -Category Cost -StartDate 2020-02-01 -TimeGrain Monthly -EndDate 2022-12-31 -ContactEmail test@test.com -NotificationKey Key1 -NotificationThreshold 0.8 -NotificationEnabled -ContactGroup $ActionGroupId
```

## <a name="create-a-budget-with-an-azure-resource-manager-template"></a>Creación de un presupuesto con una plantilla de Azure Resource Manager

Puede crear un presupuesto mediante una plantilla de Azure Resource Manager. Para utilizar la plantilla, consulte [Inicio rápido: Creación de un presupuesto con una plantilla de Azure Resource Manager](quick-create-budget-template.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un presupuesto y ya no lo necesita, vea sus detalles y elimínelo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un presupuesto en Azure Portal
> * Creación y edición de presupuestos con PowerShell
> * Crear un presupuesto con una plantilla de Azure Resource Manager

Pase al siguiente tutorial para crear una exportación periódica de los datos de administración de costos.

> [!div class="nextstepaction"]
> [Creación y administración de datos exportados](tutorial-export-acm-data.md)
