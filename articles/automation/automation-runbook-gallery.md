---
title: Uso de runbooks y módulos de Azure Automation en la Galería de PowerShell
description: En este artículo se indica cómo usar runbooks y módulos de Microsoft y la comunidad en la Galería de PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 03/04/2021
ms.topic: conceptual
ms.openlocfilehash: b8f1fbdcb3b268c24eb19517a0686c6c72c50842
ms.sourcegitcommit: af6eba1485e6fd99eed39e507896472fa930df4d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/04/2021
ms.locfileid: "106294015"
---
# <a name="use-runbooks-and-modules-in-powershell-gallery"></a>Uso de runbooks y módulos en la Galería de PowerShell

En lugar de crear sus propios runbooks y módulos en Azure Automation, dispone de acceso a escenarios ya compilados por Microsoft y la comunidad. Puede obtener runbooks y [módulos](#modules-in-powershell-gallery) de PowerShell de la Galería de PowerShell y [runbooks de Python](#use-python-runbooks) de la organización GitHub de Azure Automation. También puede contribuir a la comunidad si comparte los [escenarios que desarrolle](#add-a-powershell-runbook-to-the-gallery).

> [!NOTE]
> El Centro de scripts de TechNet se va a retirar. Todos los runbooks del Centro de scripts de la Galería de runbooks se han migrado a la [organización GitHub de Automation](https://github.com/azureautomation). Para obtener más información, vea [aquí](https://techcommunity.microsoft.com/t5/azure-governance-and-management/azure-automation-runbooks-moving-to-github/ba-p/2039337).

## <a name="runbooks-in-powershell-gallery"></a>Runbooks en la Galería de PowerShell

La [Galería de PowerShell](https://www.powershellgallery.com/packages) ofrece diversos runbooks de Microsoft y la comunidad que se pueden importar a Azure Automation. Puede usar uno al descargarlo de la galería, o puede importar runbooks directamente desde la galería o de la cuenta de Automation en Azure Portal.

> [!NOTE]
> Los runbooks gráficos no se admiten en la Galería de PowerShell.

Solo se puede importar directamente desde la Galería de PowerShell mediante Azure Portal. No se puede llevar a cabo esta función mediante PowerShell.

> [!NOTE]
> Debería validar el contenido de los runbooks que obtenga de la Galería de PowerShell y extremar el cuidado al instalarlos y ejecutarlos en un entorno de producción.

## <a name="modules-in-powershell-gallery"></a>Módulos en la Galería de PowerShell

Los módulos de PowerShell contienen cmdlets que puede usar en sus runbooks. Los módulos existentes que se pueden instalar en Azure Automation están disponibles en la [Galería de PowerShell](https://www.powershellgallery.com). Puede iniciar esta galería desde Azure Portal e instalar los módulos directamente en Azure Automation, o puede descargarlos e instalarlos de forma manual.

## <a name="common-scenarios-available-in-powershell-gallery"></a>Escenarios comunes disponibles en la Galería de PowerShell

La lista siguiente contiene algunos runbooks que admiten escenarios comunes. Para obtener una lista completa de los runbooks creados por el equipo de Azure Automation, consulte el [perfil de AzureAutomationTeam](https://www.powershellgallery.com/profiles/AzureAutomationTeam).

   * [Update-ModulesInAutomationToLatestVersion](https://www.powershellgallery.com/packages/Update-ModulesInAutomationToLatestVersion/): Importa la versión más reciente de todos los módulos de una cuenta de Automation de la Galería de PowerShell.
   * [Enable-AzureDiagnostics](https://www.powershellgallery.com/packages/Enable-AzureDiagnostics/): Configura Azure Diagnostics y Log Analytics para recibir registros de Azure Automation que contengan el estado y los flujos de trabajo.
   * [Copy-ItemFromAzureVM](https://www.powershellgallery.com/packages/Copy-ItemFromAzureVM/): Copia un archivo remoto de una máquina virtual Windows de Azure.
   * [Copy-ItemToAzureVM](https://www.powershellgallery.com/packages/Copy-ItemToAzureVM/): copia un archivo local en una máquina virtual de Azure.

## <a name="import-a-powershell-runbook-from-github-with-the-azure-portal"></a>Importación de un runbook de PowerShell desde GitHub con Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
1. Seleccione **Galería de runbooks** en **Automatización de procesos**.
1. Seleccione **Origen: GitHub**.
1. Puede usar los filtros situados encima de la lista para restringir la presentación por publicador, tipo y ordenación. Busque el elemento de la galería que desee y selecciónelo para ver sus detalles.

   :::image type="content" source="media/automation-runbook-gallery/browse-gallery-github-sm.png" alt-text="Examen de la galería de GitHub." lightbox="media/automation-runbook-gallery/browse-gallery-github-lg.png":::

1. Para importar un elemento, haga clic en **Importar** en la hoja de detalles.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-details-blade-github-sm.png" alt-text="Vista detallada de un runbook de la galería de GitHub." lightbox="media/automation-runbook-gallery/gallery-item-details-blade-github-lg.png":::

1. Si lo desea, cambie el nombre del runbook y después haga clic en **Aceptar** para importarlo.
1. El runbook aparecerá en la pestaña **Runbooks** para la cuenta de Automation.

## <a name="import-a-powershell-runbook-from-the-runbook-gallery-with-the-azure-portal"></a>Importación de un runbook de PowerShell de la galería de runbooks con Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
1. Seleccione **Galería de runbooks** en **Automatización de procesos**.
1. Seleccione **Origen: Galería de PowerShell**. Esto muestra una lista de los runbooks disponibles que puede examinar.
1. Puede usar el cuadro de búsqueda situado encima de la lista para restringir la lista, o puede usar los filtros para restringir la presentación por publicador, tipo y ordenación. Busque el elemento de la galería que desee y selecciónelo para ver sus detalles.

   :::image type="content" source="media/automation-runbook-gallery/browse-gallery-sm.png" alt-text="Examen de la galería de runbooks." lightbox="media/automation-runbook-gallery/browse-gallery-lg.png":::

1. Para importar un elemento, haga clic en **Importar** en la hoja de detalles.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-detail-sm.png" alt-text="Mostrar detalles de un elemento de la galería de runbooks." lightbox="media/automation-runbook-gallery/gallery-item-detail-lg.png":::

1. Si lo desea, cambie el nombre del runbook y después haga clic en **Aceptar** para importarlo.
1. El runbook aparecerá en la pestaña **Runbooks** para la cuenta de Automation.

## <a name="add-a-powershell-runbook-to-the-gallery"></a>Adición de un runbook de PowerShell a la galería

Microsoft recomienda agregar a la Galería de PowerShell aquellos runbooks que piense que podrían ser útiles para otros clientes. La Galería de PowerShell acepta módulos y scripts de PowerShell. Puede agregar un runbook [cargándolo en la Galería de PowerShell](/powershell/scripting/gallery/how-to/publishing-packages/publishing-a-package).

## <a name="import-a-module-from-the-module-gallery-with-the-azure-portal"></a>Importación de un módulo de la galería de módulos con Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
1. Seleccione **Módulos** en **Recursos compartidos** para abrir la lista de módulos.
1. Haga clic en **Examinar la galería** en la parte superior de la página.

      :::image type="content" source="media/automation-runbook-gallery/modules-blade-sm.png" alt-text="Vista de la galería de módulos." lightbox="media/automation-runbook-gallery/modules-blade-lg.png":::

1. En la página para examinar la galería, puede usar el cuadro Buscar para buscar coincidencias en cualquiera de los campos siguientes:

   * Nombre del módulo
   * Etiquetas
   * Autor
   * Nombre de cmdlet/recurso de DSC

1. Busque un módulo que le interese y selecciónelo para ver sus detalles.

   Al explorar un módulo específico, puede ver más información. Esta información incluye un vínculo a la Galería de PowerShell, las dependencias necesarias y todos los cmdlets o recursos de DSC que contiene el módulo.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-details-blade-sm.png" alt-text="Vista detallada de un módulo de la galería." lightbox="media/automation-runbook-gallery/gallery-item-details-blade-lg.png":::

1. Para instalar el módulo directamente en Azure Automation, haga clic en **Importar**.
1. En el panel Importar, puede ver el nombre del módulo que se va a importar. Si están instaladas todas las dependencias, el botón **Aceptar** estará activo. Si faltan dependencias, debe importarlas antes de importar este módulo.
1. En la página Importar, haga clic en **Aceptar** para importar el módulo. Cuando Azure Automation importa un módulo en la cuenta, extrae metadatos sobre el módulo y los cmdlets. Esta acción puede tardar un par de minutos, ya que debe extraerse cada actividad.
1. Recibirá una notificación inicial indicando que el módulo se está implementado y otra notificación cuando se haya completado.
1. Una vez importado el módulo, puede ver las actividades disponibles. Puede usar los recursos del módulo en los runbooks y recursos de DSC.

> [!NOTE]
> Los módulos que solo admiten PowerShell Core no son compatibles con Azure Automation y no pueden importarse en Azure Portal, ni implementarse directamente desde la Galería de PowerShell.

## <a name="use-python-runbooks"></a>Uso de runbooks de Python

Los runbooks de Python están disponibles en la [organización GitHub de Azure Automation](https://github.com/azureautomation). Cuando contribuya al repositorio de GitHub, agregue la etiqueta **(Tema de GitHub): Python3** al cargar la contribución.

## <a name="request-a-runbook-or-module"></a>Solicitud de un runbook o módulo

Puede enviar solicitudes a [Voz de usuario](https://feedback.azure.com/forums/246290-azure-automation/).  Si necesita ayuda para escribir un runbook o se plantea preguntas acerca de PowerShell, publique una pregunta en nuestra [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-automation.html).

## <a name="next-steps"></a>Pasos siguientes

* Para empezar a trabajar con un runbook de PowerShell, consulte [Tutorial: Creación de un runbook de PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Para trabajar con runbooks, consulte [Administración de runbooks en Azure Automation](manage-runbooks.md).
* Para más información sobre PowerShell, consulte [PowerShell Docs](/powershell/scripting/overview).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).