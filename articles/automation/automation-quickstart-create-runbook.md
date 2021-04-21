---
title: 'Inicio rápido de Azure: Creación de un runbook de Azure Automation | Microsoft Docs'
description: Este artículo le ayudará a comenzar a crear un runbook de Azure Automation.
services: automation
ms.date: 02/05/2019
ms.topic: quickstart
ms.subservice: process-automation
ms.custom:
- mvc
- mode-api
ms.openlocfilehash: 151199d8389130b85e917a09fd8e8d7b5f56d90c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107536544"
---
# <a name="create-an-azure-automation-runbook"></a>Creación de un runbook de Azure Automation

Los runbooks de Azure Automation se pueden crear a través de Azure. Este método proporciona una interfaz de usuario basada en explorador para crear runbooks de Automation. Esta guía de inicio rápido le guía en las operaciones de creación, edición, prueba y publicación de un runbook de Automation PowerShell.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure en https://portal.azure.com.

## <a name="create-the-runbook"></a>Creación del runbook

En primer lugar, cree un runbook. El runbook de ejemplo creado en esta guía de inicio rápido genera `Hello World` de forma predeterminada.

1. Abra su cuenta de Automation.

1. Haga clic en **Runbooks** en **Automatización de procesos**. Se muestra la lista de runbooks.

1. Haga clic en **Crear un Runbook** en la parte superior de la lista.

1. Escriba `Hello-World` como nombre de runbook en el campo **Nombre**  y seleccione **PowerShell** en el campo **Tipo de Runbook**. 

   ![Escriba la información sobre su runbook de Automation en la página](./media/automation-quickstart-create-runbook/automation-create-runbook-configure.png)

1. Haga clic en **Crear**. Se crea el runbook y se abre la página Editar Runbook de PowerShell.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-edit-runbook-empty.png" alt-text="Captura de pantalla de la página Editar Runbook de PowerShell.":::

1. Escriba o copie y pegue el código siguiente en el panel de edición. Crea un parámetro de entrada opcional denominado `Name` con el valor predeterminado `World` y genera una cadena que utiliza este valor de entrada:

   ```powershell-interactive
   param
   (
       [Parameter(Mandatory=$false)]
       [String] $Name = "World"
   )

   "Hello $Name!"
   ```

1. Haga clic en **Guardar** para guardar una copia en borrador del runbook.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-edit-runbook.png" alt-text="Captura de pantalla de la página Editar Runbook de PowerShell con un ejemplo de código en la ventana derecha.":::

## <a name="test-the-runbook"></a>Probar el runbook

Una vez que se ha creado el runbook, debe probarlo para validar que funciona.

1. Haga clic en **Panel Prueba** para abrir el panel de prueba.

1. Escriba un valor en **Nombre** y haga clic en **Iniciar**. Se inicia el trabajo de prueba y se muestran la salida y el estado del trabajo.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-test-runbook.png" alt-text="Captura de pantalla del panel Prueba con un valor de ejemplo en el campo Nombre.":::

1. Cierre la página Probar, para lo que debe hacer clic en la **X** de la esquina superior derecha. Seleccione **Aceptar** en el menú emergente que aparece.

1. En la página Editar Runbook de PowerShell, haga clic en **Publicar** para publicar el runbook como la versión oficial del runbook en la cuenta.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-job.png" alt-text="Captura de pantalla de la página Editar Runbook de PowerShell que muestra el botón Publicar seleccionado.":::

## <a name="run-the-runbook"></a>Ejecución del runbook

Una vez publicado el runbook, se muestra la página de información general.

1. En la página de información general del runbook, haga clic en **Iniciar** para abrir la página de configuración Iniciar runbook de este runbook.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-start.png" alt-text="Captura de pantalla de la página de configuración Iniciar Runbook.":::

1. Deje el valor de **Nombre** en blanco para que se use el valor predeterminado y haga clic en **Aceptar**. Se envía el trabajo del runbook y aparece la página Trabajo.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-job-page.png" alt-text="Captura de pantalla de la página del trabajo que muestra el botón Salida seleccionado.":::

1. Cuando el estado del trabajo sea `Running` o `Completed`, haga clic en **Salida** para abrir el panel Salida y ver la salida del runbook.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-job-output.png" alt-text="Captura de pantalla del panel Salida que muestra la salida del runbook.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el runbook, elimínelo. Para ello, seleccione el runbook en la lista de runbooks y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado, editado, probado y publicado un runbook e iniciado un trabajo de runbook. Para obtener más información sobre los runbooks de Automation, consulte el artículo sobre los tipos de runbook diferentes que puede crear y usar en Automation.

> [!div class="nextstepaction"]
> [Tipos de runbooks de Azure Automation](./automation-runbook-types.md)
