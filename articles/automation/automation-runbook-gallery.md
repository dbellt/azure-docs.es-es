---
title: Uso de runbooks y módulos de Azure Automation en la Galería de PowerShell
description: En este artículo se indica cómo usar runbooks y módulos de repositorios de GitHub de Microsoft y la Galería de PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 04/07/2021
ms.topic: conceptual
ms.openlocfilehash: 2df019888d293cd8a25a34e6f0f4e7dd215c6a41
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107030686"
---
# <a name="use-existing-runbooks-and-modules"></a>Uso de los runbooks y módulos existentes

En lugar de crear sus propios runbooks y módulos en Azure Automation, dispone de acceso a escenarios ya compilados por Microsoft y la comunidad. Puede obtener runbooks de Python y PowerShell relacionados con Azure en la galería de runbooks de Azure Portal, y [módulos](#modules-in-the-powershell-gallery) y [runbooks](#runbooks-in-the-powershell-gallery) (que pueden ser específicos de Azure o no) en la galería de PowerShell. También puede contribuir a la comunidad si comparte los [escenarios que desarrolle](#contribute-to-the-community).

> [!NOTE]
> El Centro de scripts de TechNet se va a retirar. Todos los runbooks del centro de scripts de la galería de runbooks se han migrado a la [organización GitHub de Automation](https://github.com/azureautomation). Para más información, consulte[Migración de runbooks de Azure Automation a GitHub](https://techcommunity.microsoft.com/t5/azure-governance-and-management/azure-automation-runbooks-moving-to-github/ba-p/2039337).

## <a name="import-runbooks-from-github-with-the-azure-portal"></a>Importación de runbooks desde GitHub con Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
2. Seleccione **Galería de runbooks** en **Automatización de procesos**.
3. Seleccione **Origen: GitHub**.
4. Puede usar los filtros situados encima de la lista para restringir la presentación por publicador, tipo y ordenación. Busque el elemento de la galería que desee y selecciónelo para ver sus detalles.

   :::image type="content" source="./media/automation-runbook-gallery/browse-gallery-github.png" alt-text="Examen de la galería de runbooks." lightbox="./media/automation-runbook-gallery/browse-gallery-github-expanded.png":::

5. Para importar un elemento, haga clic en **Importar** en la página de detalles.

   :::image type="content" source="./media/automation-runbook-gallery/gallery-item-import.png" alt-text="Importación de elementos de la galería.":::

6. Si lo desea, cambie el nombre del runbook en la hoja de importación y después haga clic en **Aceptar** para importarlo.

   :::image type="content" source="./media/automation-runbook-gallery/gallery-item-import-blade.png" alt-text="Hoja de importación de elementos de la galería.":::

7. El runbook aparecerá en la pestaña **Runbooks** para la cuenta de Automation.
 
## <a name="runbooks-in-the-powershell-gallery"></a>Runbooks de la galería de PowerShell

> [!IMPORTANT]
> Debe comprobar el contenido de cualquier runbook que obtenga de la galería de PowerShell. Tenga mucho cuidado al instalarlos y ejecutarlos en un entorno de producción.

La [galería de PowerShell](https://www.powershellgallery.com/packages) ofrece diversos runbooks de Microsoft y de la comunidad que se pueden importar a Azure Automation. Puede usar uno al descargarlo de la galería, o puede importar runbooks directamente desde la galería o de la cuenta de Automation en Azure Portal.

> [!NOTE]
> Los runbooks gráficos no se admiten en la Galería de PowerShell.

Solo se puede importar directamente desde la Galería de PowerShell mediante Azure Portal. No se puede llevar a cabo esta función mediante PowerShell. El procedimiento es el mismo que se indica en [Importación de runbooks desde GitHub con Azure Portal](#import-runbooks-from-github-with-the-azure-portal), excepto en que el **origen** será la **galería de PowerShell**.

:::image type="content" source="./media/automation-runbook-gallery/source-runbook-gallery-small.png" alt-text="Selección de origen de la galería de runbooks." lightbox="./media/automation-runbook-gallery/source-runbook-gallery-large.png":::

## <a name="modules-in-the-powershell-gallery"></a>Módulos de la galería de PowerShell

Los módulos de PowerShell contienen cmdlets que puede usar en sus runbooks. Los módulos existentes que se pueden instalar en Azure Automation están disponibles en la [Galería de PowerShell](https://www.powershellgallery.com). Puede iniciar esta galería desde Azure Portal e instalar los módulos directamente en Azure Automation, o puede descargarlos e instalarlos de forma manual.

También puede encontrar módulos para importar en Azure Portal. Se muestran para la cuenta de Automation en **Galería de módulos** en **Recursos compartidos**.

## <a name="common-scenarios-available-in-the-powershell-gallery"></a>Escenarios comunes disponibles en la galería de PowerShell

La lista siguiente contiene algunos runbooks que admiten escenarios comunes. Para obtener una lista completa de los runbooks creados por el equipo de Azure Automation, consulte el [perfil de AzureAutomationTeam](https://www.powershellgallery.com/profiles/AzureAutomationTeam).

   * [Update-ModulesInAutomationToLatestVersion](https://www.powershellgallery.com/packages/Update-ModulesInAutomationToLatestVersion/): Importa la versión más reciente de todos los módulos de una cuenta de Automation de la Galería de PowerShell.
   * [Enable-AzureDiagnostics](https://www.powershellgallery.com/packages/Enable-AzureDiagnostics/): Configura Azure Diagnostics y Log Analytics para recibir registros de Azure Automation que contengan el estado y los flujos de trabajo.
   * [Copy-ItemFromAzureVM](https://www.powershellgallery.com/packages/Copy-ItemFromAzureVM/): Copia un archivo remoto de una máquina virtual Windows de Azure.
   * [Copy-ItemToAzureVM](https://www.powershellgallery.com/packages/Copy-ItemToAzureVM/): copia un archivo local en una máquina virtual de Azure.

## <a name="contribute-to-the-community"></a>Contribución a la comunidad

Le recomendamos encarecidamente que contribuya y ayude a crecer a la comunidad de Azure Automation. Comparta los runbooks increíbles que haya creado con la comunidad. Se agradecen sus contribuciones.

### <a name="add-a-runbook-to-the-github-runbook-gallery"></a>Adición de un runbook a la galería de runbooks de GitHub

Puede agregar nuevos runbooks de PowerShell o Python a la galería de runbooks con este flujo de trabajo de GitHub.

1. Cree un repositorio público en GitHub y agregue el runbook y cualquier otro archivo necesario (como readme.md, description, etc.).
1. Agregue el tema `azureautomationrunbookgallery` para asegurarse de que nuestro servicio detecta el repositorio, de modo que se pueda mostrar en la galería de runbooks de Automation.
1. Si el runbook que creó es un flujo de trabajo de PowerShell, agregue el tema `PowerShellWorkflow`. Si se trata de un runbook de Python 3, agregue `Python3`. No se requiere ningún otro tema específico para runbooks de Azure, pero le recomendamos que agregue otros temas que se puedan usar para la categorización y la búsqueda en la galería de runbooks.

   >[!NOTE]
   >Consulte los runbooks existentes en la galería para ver aspectos como el formato, los encabezados y las etiquetas existentes que puede usar (como `Azure Automation` o `Linux Azure Virtual Machines`).

Para sugerir cambios en un runbook existente, realice una solicitud de incorporación de cambios en él. 

Si decide clonar y editar un runbook existente, el procedimiento recomendado es darle un nombre diferente. Si vuelve a usar el nombre anterior, aparecerá dos veces en la lista de la galería de runbooks.

>[!NOTE]
>Espere al menos 12 horas para la sincronización entre GitHub y la galería de runbooks de Automation, tanto para runbooks nuevos como actualizados.

### <a name="add-a-powershell-runbook-to-the-powershell-gallery"></a>Adición de un runbook de PowerShell a la galería de PowerShell

Microsoft recomienda agregar a la Galería de PowerShell aquellos runbooks que piense que podrían ser útiles para otros clientes. La Galería de PowerShell acepta módulos y scripts de PowerShell. Puede agregar un runbook [cargándolo en la Galería de PowerShell](/powershell/scripting/gallery/how-to/publishing-packages/publishing-a-package).

## <a name="import-a-module-from-the-modules-gallery-in-the-azure-portal"></a>Importación de un módulo de la galería de módulos de Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
1. En **Recursos compartidos**, seleccione **Galería de módulos** para que se abra la lista de módulos.

      :::image type="content" source="media/automation-runbook-gallery/modules-blade-sm.png" alt-text="Vista de la galería de módulos." lightbox="media/automation-runbook-gallery/modules-blade-lg.png":::

1. En la página Examinar galería, puede buscar por los campos siguientes:

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

## <a name="request-a-runbook-or-module"></a>Solicitud de un runbook o módulo

Puede enviar solicitudes a [Voz de usuario](https://feedback.azure.com/forums/246290-azure-automation/).  Si necesita ayuda para escribir un runbook o se plantea preguntas acerca de PowerShell, publique una pregunta en nuestra [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-automation.html).

## <a name="next-steps"></a>Pasos siguientes

* Para empezar a trabajar con runbooks de PowerShell, vea [Tutorial: Creación de un runbook de PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Para trabajar con runbooks, consulte [Administración de runbooks en Azure Automation](manage-runbooks.md).
* Para más información sobre el scripting de PowerShell, consulte la [documentación de PowerShell](/powershell/scripting/overview).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
