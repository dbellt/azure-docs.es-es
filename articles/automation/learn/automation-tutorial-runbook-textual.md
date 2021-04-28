---
title: Creación de un runbook de flujo de trabajo de PowerShell en Azure Automation
description: Este artículo le enseña a crear, probar y publicar un sencillo runbook de flujo de trabajo de PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 45c7bb19e3f48fedf93fbb147a3014e09914a475
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831283"
---
# <a name="tutorial-create-a-powershell-workflow-runbook"></a>Tutorial: Creación de un runbook de flujo de trabajo de PowerShell

Este tutorial le guiará para crear un [runbook de flujo de trabajo de PowerShell](../automation-runbook-types.md#powershell-workflow-runbooks) en Azure Automation. Los runbooks de flujo de trabajo de PowerShell son runbooks de texto basados en el flujo de trabajo de Windows PowerShell. Puede crear y modificar directamente el código del runbook con el editor de texto en Azure Portal. 

> [!div class="checklist"]
> * Creación de un runbook de flujo de trabajo simple de PowerShell
> * Prueba y publicación del runbook
> * Ejecución y seguimiento del estado del trabajo de runbook
> * Actualización del runbook para iniciar una máquina virtual de Azure con parámetros de runbook

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Cuenta de Automation](../index.yml) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual.
* Una máquina virtual de Azure. Dado que va a detener e iniciar esta máquina, no debería ser una máquina virtual de producción.

## <a name="step-1---create-new-runbook"></a>Paso 1: crear nuevo runbook

Empiece por crear un runbook simple que genere el texto `Hello World`.

1. En Azure Portal, abra su cuenta de Automation.

   La página de la cuenta de Automation proporciona una vista rápida de los recursos que hay en esa cuenta. Ya debería tener algunos recursos. La mayoría son los módulos que se incluyen automáticamente en una cuenta nueva de Automation. También debe tener el recurso de credencial asociado a su suscripción.
 
2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.

3. Seleccione **Crear un Runbook** para crear uno.

4. Asigne al runbook el nombre **MyFirstRunbook-Workflow**.

5. En este caso, va a crear un [runbook de flujo de trabajo de PowerShell](../automation-runbook-types.md#powershell-workflow-runbooks). Seleccione **Flujo de trabajo de PowerShell** en **Tipo de Runbook**.

6. Haga clic en **Crear** para crear el runbook y abra el editor de texto.

## <a name="step-2---add-code-to-the-runbook"></a>Paso 2: Incorporación de código al runbook

Puede escribir el código directamente en el runbook o seleccionar los cmdlets, runbooks y recursos desde el control Biblioteca y agregarlos al runbook con los parámetros relacionados. En este tutorial, escribirá el código directamente en el runbook.

1. El runbook está vacío actualmente, solo con la palabra clave `Workflow` necesaria, el nombre del runbook y las llaves que encierran el flujo de trabajo completo.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   }
   ```

2. Escriba `Write-Output "Hello World"` entre las llaves.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   Write-Output "Hello World"
   }
   ```

3. Guarde el Runbook, para lo que debe hacer clic en **Guardar**.

## <a name="step-3---test-the-runbook"></a>Paso 3: probar el runbook

Antes de publicar el runbook para que esté disponible en producción, debe probarlo para asegurarse de que funciona correctamente. La prueba de un runbook ejecuta su versión de borrador y permite ver la salida de forma interactiva.

1. Seleccione **Panel de prueba** para abrir el panel de prueba.

2. Haga clic en **Iniciar** para iniciar la prueba, en la que se probará la única opción habilitada.

3. Observe que se crea un [trabajo de runbook](../automation-runbook-execution.md) y su estado se muestra en el panel.

   El estado del trabajo se inicia como En cola, que indica que está esperando a que haya disponible un trabajo de runbook en la nube. El estado cambia a Iniciando cuando un trabajo lo solicita. Por último, el estado pasa a En ejecución cuando el runbook comienza a ejecutarse.

4. Cuando se completa el trabajo del runbook, el panel de prueba muestra su salida. En este caso, verá `Hello World`.

   ![Hola mundo](../media/automation-tutorial-runbook-textual/test-output-hello-world.png)

5. Cierre el panel Prueba para volver al lienzo.

## <a name="step-4---publish-and-start-the-runbook"></a>Paso 4: publicar e iniciar el runbook

El runbook que ha creado aún está en modo Borrador. Tiene que publicarlo para poder ejecutarlo en producción. Al publicar un runbook, se sobrescribe la versión publicada existente con la versión de borrador. En este caso, no tiene una versión publicada aún porque acaba de crear el runbook.

1. Haga clic en **Publicar** para publicar el runbook y en **Sí** cuando se le solicite.

2. Desplácese hacia la izquierda para ver el runbook en la página **Runbooks** y observe que el campo **Estado de creación** está establecido en **Publicado**.

3. Desplácese de nuevo a la derecha para ver el panel de **MyFirstRunbook-Workflow**.

   Las opciones que se encuentran en la parte superior permiten iniciar el runbook ahora, programar una hora de inicio futura o crear un [webhook](../automation-webhooks.md) para que el runbook pueda iniciarse mediante una llamada HTTP.

4. Para iniciar el runbook, seleccione **Iniciar** y **Sí** cuando se le solicite.

   ![Iniciar runbook](../media/automation-tutorial-runbook-textual/automation-runbook-controls-start.png)

5. Se abre un panel Trabajo para el trabajo de runbook que se ha creado. En este caso, deje abierto el panel para poder ver el progreso del trabajo.

6. Observe que el estado del trabajo se muestra en **Resumen del trabajo**. Este estado coincide con los estados que vio al probar el runbook.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-jobsummary.png" alt-text="Captura de pantalla del panel de trabajo de runbook que muestra la sección de resumen de trabajo resaltada.":::

7. Cuando el estado del runbook aparezca como Completado, haga clic en **Salida**. Se abre la página Salida, donde puede ver el mensaje `Hello World`.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-outputtile.png" alt-text="Captura de pantalla del panel de trabajo de runbook que muestra el botón de resultados resaltado.":::

8. Cierre la página Salida.

9. Haga clic en **Todos los registros** para abrir el panel Transmisiones para el trabajo de Runbook. Solo debería ver `Hello World` en el flujo de salida. Tenga en cuenta que el panel Transmisiones puede mostrar otras transmisiones de un trabajo de runbook, como las transmisiones de error y detalladas, si el runbook escribe en ellas.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-alllogstile.png" alt-text="Captura de pantalla del panel de trabajo de runbook que muestra el botón de todos los registros resaltado.":::

10. Cierre el panel Transmisiones y el panel Trabajo para volver a la página de MyFirstRunbook.

11. Haga clic en **Trabajos** bajo **Recursos** para abrir la página Trabajos de este runbook. Esta página enumera todos los trabajos creados por el runbook. Solo debería ver un trabajo en la lista ya que solo ejecutó el trabajo una vez.

   ![Trabajos](../media/automation-tutorial-runbook-textual/runbook-control-job-tile.png)

12. Haga clic en el nombre del trabajo para abrir el mismo panel Trabajo que vio cuando inició el runbook. Use este panel para ver los detalles de cualquier trabajo creado para el runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Paso 5: agregar autenticación para administrar recursos de Azure

Ha probado y publicado su runbook, pero hasta ahora no hace nada útil. Quiere que administre recursos de Azure. Sin embargo, no puede hacerlo a menos que se autentique con las credenciales de la suscripción. La autenticación usa el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

>[!NOTE]
>En el caso de los runbooks de PowerShell, `Add-AzAccount` y `Add-AzureRMAccount` son alias de `Connect-AzAccount`. Puede usar estos cmdlets o bien [actualizar los módulos](../automation-update-azure-modules.md) de la cuenta de Automation a las versiones más recientes. Es posible que deba actualizar los módulos incluso si acaba de crear una nueva cuenta de Automation.

1. Vaya a la página de MyFirstRunbook-Workflow y abra el editor de texto haciendo clic en **Editar**.

2. Elimine la línea `Write-Output`.

3. Coloque el cursor en una línea en blanco entre las llaves.

4. Escriba o copie y pegue el siguiente código, que controla la autenticación con la cuenta de ejecución de Automation.

   ```powershell-interactive
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
   -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Select-AzSubscription -SubscriptionId $Conn.SubscriptionID
   ```

5. Haga clic en el **panel de prueba** para poder probar el runbook.

6. Haga clic en **Iniciar** para iniciar la prueba. Cuando termine, verá una salida similar a la siguiente, que muestra información básica de su cuenta. Esta acción confirma que la credencial es válida.

   ![Authenticate](../media/automation-tutorial-runbook-textual/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Paso 6: Incorporación de una actividad para iniciar una máquina virtual

Ahora que el runbook está autenticado en la suscripción a Azure, puede administrar los recursos. Agreguemos un comando para iniciar una máquina virtual. Puede seleccionar cualquier máquina virtual de su suscripción de Azure. Por ahora, codificará ese nombre en el runbook. Si va a administrar recursos en varias suscripciones, debe usar el parámetro `AzContext`Get-AzContext[ con el cmdlet ](/powershell/module/az.accounts/get-azcontext).

1. Indique el nombre y el nombre del grupo de recursos de la máquina virtual que se va a iniciar; para ello, escriba una llamada al cmdlet [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM) tal como se muestra a continuación. 

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
   # Ensures that you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Get-AzSubscription -SubscriptionId $Conn.SubscriptionID

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName' -AzContext $AzureContext
   }
   ```

2. Guarde el runbook y haga clic en **Panel de prueba** para probarlo.

3. Haga clic en **Iniciar** para iniciar la prueba. Cuando termine, compruebe que se ha iniciado la máquina virtual.

## <a name="step-7---add-an-input-parameter-to-the-runbook"></a>Paso 7: agregar un parámetro de entrada al runbook

Actualmente, el runbook inicia la máquina virtual que ha codificado en el runbook. Será más útil si puede especificar la máquina virtual cuando se inicia el runbook. Agreguemos parámetros de entrada al runbook para incluir esta funcionalidad.

1. Agregue las variables para los parámetros `VMName` y `ResourceGroupName` al runbook, y use las variables con el cmdlet `Start-AzVM` como se muestra a continuación.

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
    Param(
     [string]$VMName,
     [string]$ResourceGroupName
    )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   }
   ```

2. Guarde el runbook y abra el panel Prueba. Ahora puede proporcionar valores para las dos variables de entrada que se usarán en la prueba.

3. Cierre el panel Prueba.

4. Haga clic en **Publicar** para publicar la nueva versión del runbook.

5. Detenga la máquina virtual que ha iniciado.

6. Haga clic en **Iniciar** para iniciar el runbook. 

7. Escriba los valores de **VMNAME** y **RESOURCEGROUPNAME** correspondientes a la máquina virtual que va a iniciar.

   ![Inicio del runbook](../media/automation-tutorial-runbook-textual/automation-pass-params.png)

8. Cuando se complete el runbook, compruebe que se ha iniciado la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre PowerShell, incluidos los módulos de referencia de lenguaje y aprendizaje, consulte la [documentación de PowerShell](/powershell/scripting/overview).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
* Para empezar a trabajar con runbooks gráficos, consulte [Creación de un runbook gráfico](automation-tutorial-runbook-graphical.md).
* Para empezar a trabajar con Runbooks de PowerShell, consulte [Creación de un runbook de PowerShell](automation-tutorial-runbook-textual-powershell.md).
* Para aprender más sobre los tipos de runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Azure Automation](../automation-runbook-types.md).
* Para más información sobre las características de compatibilidad con scripts de PowerShell, consulte [Anuncio de compatibilidad con scripts de PowerShell nativo en Azure Automation](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
