---
title: Guía paso a paso de notificaciones por correo electrónico para el ajuste automático
description: Habilite las notificaciones por correo electrónico para la optimización automática de consultas de Azure SQL Database.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: NikaKinska
ms.author: nnikolic
ms.reviewer: mathoma, wiassaf
ms.date: 06/03/2019
ms.openlocfilehash: 7216978845921e4b35c4cb3485379054cbf5cfff
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591823"
---
# <a name="email-notifications-for-automatic-tuning"></a>Notificaciones por correo electrónico para el ajuste automático
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]


Las recomendaciones de ajustes de Azure SQL Database se generan mediante el [ajuste automático](automatic-tuning-overview.md) de Azure SQL Database. Esta solución supervisa y analiza de forma continua las cargas de trabajo de las bases de datos y proporciona recomendaciones de ajustes personalizadas para cada base de datos individual relacionadas con la creación y eliminación de índices y la optimización de los planes de ejecución de consultas.

Las recomendaciones de ajuste automático de Azure SQL Database pueden verse en [Azure Portal](database-advisor-find-recommendations-portal.md) y recuperarse con llamadas a la [API REST](/rest/api/sql/databaserecommendedactions/listbydatabaseadvisor), o mediante los comandos [T-SQL](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management/) y [PowerShell](/powershell/module/az.sql/get-azsqldatabaserecommendedaction). Este artículo se basa en el uso de un script de PowerShell para recuperar las recomendaciones de ajuste automático.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

## <a name="automate-email-notifications-for-automatic-tuning-recommendations"></a>Automatización de las notificaciones por correo electrónico para las recomendaciones del ajuste automático

La siguiente solución automatiza el envío de notificaciones por correo electrónico que contienen recomendaciones sobre el ajuste automático. La solución descrita consiste en automatizar la ejecución de un script de PowerShell para recuperar las recomendaciones de ajustes mediante [Azure Automation](../../automation/automation-intro.md), y automatizar el trabajo de programación de la entrega de correos electrónicos mediante [Microsoft Flow](https://flow.microsoft.com).

## <a name="create-azure-automation-account"></a>Creación de una cuenta de Azure Automation

Para utilizar Azure Automation, primero debe crear una cuenta de automatización y configurarla con los recursos de Azure que se usarán para ejecutar el script de PowerShell. Para obtener más información acerca de Azure Automation, consulte [Introducción a Azure Automation](../../automation/index.yml).

Siga estos pasos para crear la cuenta de Azure Automation mediante el método de selección y configuración de la aplicación Automation de Azure Marketplace:

1. Inicie sesión en Azure Portal.
1. Haga clic en " **+ Crear un recurso**" en la esquina superior izquierda.
1. Busque "**Automation**" (presione ENTRAR).
1. Haga clic en la aplicación Automation en los resultados de la búsqueda.

    ![Agregar Azure Automation](./media/automatic-tuning-email-notifications-configure/howto-email-01.png)

1. Una vez dentro del panel "Crear una cuenta de Automation", haga clic en "**Crear**".
1. Rellene la información necesaria: escriba un nombre para esta cuenta de Automation, y seleccione los recursos de Azure y el id. de suscripción de Azure que se usarán para la ejecución del script de PowerShell.
1. Para la opción "**Crear cuenta de ejecución de Azure**", seleccione **Sí** para configurar el tipo de cuenta en que el script de PowerShell se ejecuta con la ayuda de Azure Automation. Para más información sobre los tipos de cuenta, consulte [Ejecutar como cuenta](../../automation/manage-runas-account.md).
1. Para finalizar la creación de la cuenta de Automation, haga clic en **Crear**.

> [!TIP]
> Registre el nombre de la cuenta de Azure Automation, el identificador de suscripción y los recursos (por ejemplo, copiar y pegar en el Bloc de notas) tal y como se especificó al crear la aplicación Automation. Necesitará esta información más adelante.

Si tiene varias suscripciones a Azure para las que quiere compilar la misma automatización, deberá repetir este proceso para las otras suscripciones.

## <a name="update-azure-automation-modules"></a>Actualización de los módulos de Azure Automation

El script de PowerShell para recuperar la recomendación de ajuste automático usa los comandos [Get-AzResource](/powershell/module/az.Resources/Get-azResource) y [Get-AzSqlDatabaseRecommendedAction](/powershell/module/az.Sql/Get-azSqlDatabaseRecommendedAction) para los que se requiere la actualización de los módulos de Azure a la versión 4 o posterior.

- En caso de que los módulos de Azure deban actualizarse, consulte [Compatibilidad con módulos de Az en Azure Automation](../../automation/shared-resources/modules.md).

## <a name="create-azure-automation-runbook"></a>Creación de un runbook de Azure Automation

El siguiente paso consiste en crear un runbook en Azure Automation en el que se encuentre el script de PowerShell para la recuperación de recomendaciones de ajuste.

Para crear un nuevo runbook de Azure Automation, siga estos pasos:

1. Acceda a la cuenta de Azure Automation que creó en el paso anterior.
1. Una vez esté en el panel de la cuenta de Automation, haga clic en el elemento de menú "**Runbooks**" en el lado izquierdo para crear un runbook de Azure Automation con el script de PowerShell. Para obtener más información sobre la creación de runbooks de automatización, consulte [Creación de un runbook](../../automation/manage-runbooks.md#create-a-runbook).
1. Para agregar un runbook nuevo, haga clic en la opción de menú " **+Agregar un runbook**" y, a continuación, haga clic en "**Creación rápida - Crear un runbook nuevo**".
1. En el panel Runbook, escriba el nombre de su runbook (en este ejemplo, se usa "**AutomaticTuningEmailAutomation**"), seleccione el tipo de runbook como **PowerShell** y escriba una descripción de este runbook para indicar su finalidad.
1. Haga clic en el botón **Crear** para terminar de crear un nuevo runbook.

    ![Agregar un runbook de Azure Automation](./media/automatic-tuning-email-notifications-configure/howto-email-03.png)

Siga estos pasos para cargar un script de PowerShell en el runbook creado:

1. En el panel "**Editar runbook de PowerShell**&quot;, seleccione &quot;**RUNBOOKS**&quot; en el árbol de menú y expanda la vista hasta que vea el nombre del runbook (en este ejemplo, &quot;**AutomaticTuningEmailAutomation**"). Seleccione este runbook.
1. En la primera línea del panel "Editar runbook de PowerShell" (comenzando por el número 1), copie y pegue el siguiente código de script de PowerShell. Este script de PowerShell se proporciona tal cual para ayudarle a comenzar. Modifíquelo para que se adapte a sus necesidades.

En el encabezado del script de PowerShell proporcionado, es necesario reemplazar `<SUBSCRIPTION_ID_WITH_DATABASES>` por su identificador de suscripción de Azure. Para obtener información sobre cómo recuperar el identificador de la suscripción de Azure, consulte [Getting your Azure Subscription GUID](/archive/blogs/mschray/getting-your-azure-subscription-guid-new-portal) (Obtención del GUID de la suscripción a Azure).

Si tiene varias suscripciones, puede agregarlas como una lista delimitada por comas a la propiedad "$subscriptions" del encabezado del script.

```powershell
# PowerShell script to retrieve Azure SQL Database automatic tuning recommendations.
#
# Provided "as-is" with no implied warranties or support.
# The script is released to the public domain.
#
# Replace <SUBSCRIPTION_ID_WITH_DATABASES> in the header with your Azure subscription ID.
#
# Microsoft Azure SQL Database team, 2018-01-22.

# Set subscriptions : IMPORTANT – REPLACE <SUBSCRIPTION_ID_WITH_DATABASES> WITH YOUR SUBSCRIPTION ID
$subscriptions = ("<SUBSCRIPTION_ID_WITH_DATABASES>", "<SECOND_SUBSCRIPTION_ID_WITH_DATABASES>", "<THIRD_SUBSCRIPTION_ID_WITH_DATABASES>")

# Get credentials
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Define the resource types
$resourceTypes = ("Microsoft.Sql/servers/databases")
$advisors = ("CreateIndex", "DropIndex");
$results = @()

# Loop through all subscriptions
foreach($subscriptionId in $subscriptions) {
    Select-AzSubscription -SubscriptionId $subscriptionId
    $rgs = Get-AzResourceGroup

    # Loop through all resource groups
    foreach($rg in $rgs) {
        $rgname = $rg.ResourceGroupName;

        # Loop through all resource types
        foreach($resourceType in $resourceTypes) {
            $resources = Get-AzResource -ResourceGroupName $rgname -ResourceType $resourceType

            # Loop through all databases
            # Extract resource groups, servers and databases
            foreach ($resource in $resources) {
                $resourceId = $resource.ResourceId
                if ($resourceId -match ".*RESOURCEGROUPS/(?<content>.*)/PROVIDERS.*") {
                    $ResourceGroupName = $matches['content']
                } else {
                    continue
                }
                if ($resourceId -match ".*SERVERS/(?<content>.*)/DATABASES.*") {
                    $ServerName = $matches['content']
                } else {
                    continue
                }
                if ($resourceId -match ".*/DATABASES/(?<content>.*)") {
                    $DatabaseName = $matches['content']
                } else {
                    continue
                }

                # Skip if master
                if ($DatabaseName -eq "master") {
                    continue
                }

                # Loop through all automatic tuning recommendation types
                foreach ($advisor in $advisors) {
                    $recs = Get-AzSqlDatabaseRecommendedAction -ResourceGroupName $ResourceGroupName -ServerName $ServerName  -DatabaseName $DatabaseName -AdvisorName $advisor
                    foreach ($r in $recs) {
                        if ($r.State.CurrentValue -eq "Active") {
                            $object = New-Object -TypeName PSObject
                            $object | Add-Member -Name 'SubscriptionId' -MemberType Noteproperty -Value $subscriptionId
                            $object | Add-Member -Name 'ResourceGroupName' -MemberType Noteproperty -Value $r.ResourceGroupName
                            $object | Add-Member -Name 'ServerName' -MemberType Noteproperty -Value $r.ServerName
                            $object | Add-Member -Name 'DatabaseName' -MemberType Noteproperty -Value $r.DatabaseName
                            $object | Add-Member -Name 'Script' -MemberType Noteproperty -Value $r.ImplementationDetails.Script
                            $results += $object
                        }
                    }
                }
            }
        }
    }
}

# Format and output results for the email
$table = $results | Format-List
Write-Output $table
```

Haga clic en el botón "**Guardar**" de la esquina superior derecha para guardar el script. Cuando esté satisfecho con el script, haga clic en el botón "**Publicar**" para publicar este runbook.

En el panel principal del runbook, puede hacer clic en el botón "**Iniciar**" para **probar** el script. Haga clic en la "**Salida**" para ver los resultados del script ejecutado. Esta salida será el contenido de su correo electrónico. La salida de ejemplo del script se puede ver en la captura de pantalla siguiente.

![Ejecutar vista de recomendaciones de ajuste automáticas con Azure Automation](./media/automatic-tuning-email-notifications-configure/howto-email-04.png)

Asegúrese de ajustar el contenido. Para ello, personalice el script de PowerShell según sus necesidades.

Con los pasos anteriores, el script de PowerShell para recuperar las recomendaciones de ajuste automático se carga en Azure Automation. El siguiente paso consiste en automatizar y programar el trabajo de entrega de correos electrónicos.

## <a name="automate-the-email-jobs-with-microsoft-flow"></a>Automatización de las tareas de correo electrónico con Microsoft Flow

Para completar la solución, como último paso, cree un flujo de automatización en Microsoft Flow que conste de tres acciones (trabajos):

- "**Azure Automation - Crear trabajo**": se usa para ejecutar el script de PowerShell a fin de recuperar las recomendaciones de ajuste automático del runbook de Azure Automation.
- "**Azure Automation - Obtener resultado del trabajo**": se usa para recuperar la salida del script de PowerShell ejecutado.
- "**Office 365 Outlook - Enviar un correo electrónico**": se usa para enviar correos electrónicos. Los correos electrónicos se envían mediante la cuenta profesional o educativa de la persona que crea el flujo.

Para obtener más información sobre las funcionalidades de Microsoft Flow, consulte [Introducción a Microsoft Flow](/flow/getting-started).

El requisito previo para este paso es registrarse para obtener una cuenta de [Microsoft Flow](https://flow.microsoft.com) e iniciar sesión. Una vez dentro de la solución, siga estos pasos para configurar un **flujo nuevo**:

1. Acceda al elemento de menú "**Mis flujos**".
1. Dentro de Mis flujos, seleccione el vínculo " **+Crear desde cero**" en la parte superior de la página.
1. Haga clic en el vínculo "**Buscar entre cientos de conectores y desencadenadores**" en la parte inferior de la página.
1. En el campo de búsqueda, escriba "**Periodicidad**" y seleccione "**Programación - Periodicidad**" en los resultados de la búsqueda para programar la ejecución del trabajo de entrega de correos electrónicos.
1. Desde el panel Periodicidad, en el campo Frecuencia, seleccione la frecuencia de programación para la ejecución de este flujo; por ejemplo, enviar un correo electrónico automatizado cada minuto, hora, día, semana, etc.

En el siguiente paso, debe agregar tres trabajos (crear, obtener salida y enviar correo electrónico) al flujo de periodicidad recién creado. Para agregar los trabajos necesarios al flujo, siga estos pasos:

1. Cree una acción para ejecutar el script de PowerShell para recuperar las recomendaciones de ajuste.

   - Seleccione " **+ Nuevo paso**", seguido de "**Agregar una acción**" dentro del panel de flujo Periodicidad.
   - En el campo de búsqueda, escriba "**automation**" y seleccione "**Azure Automation – Crear trabajo**" en los resultados de la búsqueda.
   - En el panel de trabajo Crear, configure las propiedades del trabajo. En esta configuración, necesitará los detalles del identificador de la suscripción de Azure, el grupo de recursos y la cuenta de automatización que se **registraron anteriormente** en el **panel Cuenta de automatización**. Para obtener más información acerca de las opciones disponibles en esta sección, consulte [Azure Automation – Create job](/connectors/azureautomation/#create-job) (Azure Automation - Crear un trabajo).
   - Para completar la creación de esta acción, haga clic en "**Guardar flujo**".

2. Creación de una acción para recuperar la salida del script de PowerShell ejecutado

   - Seleccione " **+ Nuevo paso**", seguido de "**Agregar una acción**" dentro del panel de flujo Periodicidad.
   - En el campo de búsqueda, escriba "**automation**" y seleccione "**Azure Automation – Obtener resultado del trabajo**" en los resultados de la búsqueda. Para obtener más información acerca de las opciones disponibles en esta sección, consulte [Azure Automation – Get job output](/connectors/azureautomation/#get-job-output)" (Azure Automation - Obtener la salida de trabajo).
   - Rellene los campos obligatorios (como cuando creó el trabajo anterior): rellene su id. de suscripción de Azure, el grupo de recursos y la cuenta de Automation (tal y como se especificó en el panel de la cuenta de Automation).
   - Haga clic en el campo "**Id. de trabajo**" para que se muestre el menú "**Contenido dinámico**". En este menú, seleccione la opción "**Id. de trabajo**".
   - Para completar la creación de esta acción, haga clic en "**Guardar flujo**".

3. Creación de una acción para enviar correos electrónicos mediante la integración de Office 365

   - Seleccione " **+ Nuevo paso**", seguido de "**Agregar una acción**" dentro del panel de flujo Periodicidad.
   - En el campo de búsqueda, escriba "**enviar un correo electrónico**" y seleccione "**Office 365 Outlook - Enviar un correo electrónico**" en los resultados de la búsqueda.
   - En el campo "**Para**", escriba la dirección de correo electrónico a la que debe enviar el correo electrónico de notificación.
   - En el campo "**Asunto**", escriba el asunto del correo electrónico; por ejemplo, "Notificación por correo electrónico de las recomendaciones de ajuste automático".
   - Haga clic en el campo "**Cuerpo**" para que se muestre el menú "**Contenido dinámico**". Desde dentro de este menú, en "**Obtener resultado del trabajo**", seleccione "**Contenido**".
   - Para completar la creación de esta acción, haga clic en "**Guardar flujo**".

> [!TIP]
> Para enviar correos electrónicos automatizados a varios destinatarios, cree flujos independientes. En estos flujos adicionales, cambie la dirección de correo electrónico del destinatario del campo "A" y la línea de asunto del correo electrónico del campo "Asunto". La creación de runbooks nuevos en Azure Automation con scripts de PowerShell personalizados (por ejemplo, con cambio de identificador de suscripción de Azure) permite personalizar mejor los escenarios automatizados, como, por ejemplo, el envío de correos electrónicos sobre recomendaciones del ajuste automático a diferentes destinatarios para distintas suscripciones.

Los pasos anteriores proporcionan todo lo que debe hacer para configurar el flujo de trabajo de entrega de correos electrónicos. El flujo completo, que consta de tres acciones integradas, se muestra en la imagen siguiente.

![Vista del flujo de notificaciones por correo electrónico para el ajuste automático](./media/automatic-tuning-email-notifications-configure/howto-email-05.png)

Para probar el flujo, haga clic en "**Ejecutar ahora**" en la esquina superior derecha del panel del flujo.

Las estadísticas de la ejecución de los trabajos automatizados, que muestran si las notificaciones por correo electrónico se enviaron correctamente, se pueden ver desde el panel de análisis de Flow.

![Flujo de ejecución para las notificaciones por correo electrónico del ajuste automático](./media/automatic-tuning-email-notifications-configure/howto-email-06.png)

El panel de análisis de Flow resulta útil para supervisar el éxito de las ejecuciones de trabajo y, si es necesario, para solucionar problemas.  Para solucionar problemas, también puede examinar el registro de ejecución del script de PowerShell accesible a través de la aplicación Azure Automation.

El resultado final del correo electrónico automatizado tiene un aspecto similar al siguiente correo electrónico recibido después de compilar y ejecutar esta solución:

![Salida de correo electrónico de ejemplo de notificaciones por correo electrónico de los ajustes automáticos](./media/automatic-tuning-email-notifications-configure/howto-email-07.png)

Mediante el ajuste del script de PowerShell, puede ajustar la salida y el formato del correo electrónico automatizado según sus necesidades.

También puede personalizar aún más la solución para compilar notificaciones por correo electrónico en función de un evento de ajuste específico, y para varios destinatarios, suscripciones o bases de datos, según sus escenarios personalizados.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre cómo el ajuste automático puede ayudarle a mejorar el rendimiento de la base de datos, consulte [Ajuste automático en Azure SQL Database](automatic-tuning-overview.md).
- Para habilitar la característica de ajuste automático en Azure SQL Database para administrar la carga de trabajo, vea [Habilitación del ajuste automático](automatic-tuning-enable.md).
- Para revisar manualmente y aplicar las recomendaciones de ajuste automático, vea [Búsqueda y aplicación de recomendaciones de rendimiento](database-advisor-find-recommendations-portal.md).
