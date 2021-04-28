---
title: Uso de la integración del control de código fuente en Azure Automation
description: En este artículo se describe cómo sincronizar el control de código fuente de Azure Automation con otros repositorios.
services: automation
ms.subservice: process-automation
ms.date: 03/10/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d94da9792d40a389e3981163e565d85d82a9cdc9
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831247"
---
# <a name="use-source-control-integration"></a>Uso de la integración del control de código fuente

 La integración del control de código fuente en Azure Automation permite la sincronización en una sola dirección desde el repositorio de control de código fuente. El control de código fuente permite mantener actualizados los runbooks de la cuenta de Automation con los scripts del repositorio de control de código fuente de GitHub o de Azure Repos. Esta característica facilita la promoción de código que se ha probado en el entorno de desarrollo a la cuenta de Automation de producción.

 La integración del control de código fuente permite colaborar fácilmente con el equipo, realizar el seguimiento de los cambios y revertir a versiones anteriores de los runbooks. Por ejemplo, el control de código fuente permite sincronizar distintas ramas de control de código fuente con las cuentas de Automation de desarrollo, prueba o producción.

## <a name="source-control-types"></a>Tipos de control de código fuente

Azure Automation admite tres tipos de control de código fuente:

* GitHub
* Azure Repos (Git)
* Azure Repos (TFVC)

## <a name="prerequisites"></a>Prerrequisitos

* Un repositorio de control de código fuente (GitHub o Azure Repos).
* Una [cuenta de ejecución](automation-security-overview.md#run-as-accounts)
* El [módulo `AzureRM.Profile`](/powershell/module/azurerm.profile/) se debe importar en la cuenta de Automation. Tenga en cuenta que el módulo Az (`Az.Accounts`) equivalente no funcionará con el control de código fuente de Automation.

> [!NOTE]
> Los trabajos de sincronización de control de código fuente se ejecutan en la cuenta de Automation del usuario y se facturan con la misma tarifa que otros trabajos de Automation.

## <a name="configure-source-control"></a>Configurar el control de código fuente

En esta sección se explica cómo configurar el control de código fuente para la cuenta de Automation. Puede usar Azure Portal o PowerShell.

### <a name="configure-source-control-in-azure-portal"></a>Configuración del control de código fuente en Azure Portal

Utilice este procedimiento para configurar el control de código fuente mediante Azure Portal.

1. En la cuenta de Automation, seleccione **Control de código fuente** y haga clic en **Agregar**.

    ![Seleccionar control de código fuente](./media/source-control-integration/select-source-control.png)

2. Elija **Tipo de control de código fuente** y, a continuación, haga clic en **Autenticar**.

3. Se abre una ventana del explorador y se le pide que inicie sesión. Siga las indicaciones para completar la autenticación.

4. En la página Resumen del control de código fuente, utilice los campos para rellenar las propiedades del control de código fuente definidas a continuación. Haga clic en **Guardar** cuando termine.

    |Propiedad  |Descripción  |
    |---------|---------|
    |Nombre del control de código fuente     | Nombre descriptivo del control de código fuente. Este nombre solo puede contener letras y números.        |
    |Tipo de control de código fuente     | Tipo de mecanismo de control de código fuente. Las opciones disponibles son la siguientes:</br> * GitHub</br>* Azure Repos (Git)</br> * Azure Repos (TFVC)        |
    |Repositorio     | Nombre del repositorio o proyecto. Se recuperan los primeros 200 repositorios. Para buscar un repositorio, escriba el nombre en el campo y haga clic en **Search on GitHub** (Buscar en GitHub).|
    |Rama     | Rama de la que se van a extraer los archivos de código fuente. No se permite especificar una rama para el tipo de control de código fuente TFVC.          |
    |Ruta de acceso a la carpeta     | Carpeta que contiene los runbooks que se van a sincronizar; por ejemplo, **/Runbooks**. Solo se sincronizan los runbooks de la carpeta especificada. No se admite la recursividad.        |
    |Sincronización automática<sup>1</sup>     | Opción que activa o desactiva la sincronización automática cuando se realiza una confirmación en el repositorio de control de código fuente.        |
    |Publicar runbook     | Se debe establecer en Activado si los runbooks se publican automáticamente después de la sincronización desde el control de código fuente y en Desactivado de otro modo.           |
    |Descripción     | Texto que especifica información adicional sobre el control de código fuente.        |

    <sup>1</sup> Para habilitar la sincronización automática al configurar la integración del control de código fuente con Azure Repos, debe ser un administrador del proyecto.

   ![Resumen de control de código fuente](./media/source-control-integration/source-control-summary.png)

> [!NOTE]
> El inicio de sesión en el repositorio del control de código fuente podría ser diferente al inicio de sesión en Azure Portal. Al configurar el control de código fuente, asegúrese de que ha iniciado sesión con la cuenta correcta en el repositorio del control de código fuente. Si tiene dudas, abra una nueva pestaña en el explorador, cierre la sesión de **dev.azure.com**, **visualstudio.com** o **github.com**, y vuelva a intentar establecer la conexión con el control de código fuente.

### <a name="configure-source-control-in-powershell"></a>Configuración del control de código fuente en PowerShell

También puede usar PowerShell para configurar el control de código fuente en Azure Automation. Para usar cmdlets de PowerShell para esta operación, necesita un token de acceso personal (PAT). Utilice el cmdlet [New-AzAutomationSourceControl](/powershell/module/az.automation/new-azautomationsourcecontrol) para crear la conexión al control de código fuente. Este cmdlet requiere una cadena segura para el PAT. Para obtener información sobre cómo crear una cadena segura, consulte [ConvertTo-SecureString](/powershell/module/microsoft.powershell.security/convertto-securestring).

En las subsecciones siguientes se muestra la creación de PowerShell de la conexión del control de código fuente para GitHub, Azure Repos (Git) y Azure Repos (TFVC).

#### <a name="create-source-control-connection-for-github"></a>Creación de la conexión del control de código fuente para GitHub

```powershell-interactive
New-AzAutomationSourceControl -Name SCGitHub -RepoUrl https://github.com/<accountname>/<reponame>.git -SourceType GitHub -FolderPath "/MyRunbooks" -Branch master -AccessToken <secureStringofPAT> -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName>
```

#### <a name="create-source-control-connection-for-azure-repos-git"></a>Creación de la conexión del control de código fuente para Azure Repos (Git)

> [!NOTE]
> Azure Repos (Git) usa una dirección URL que accede a **dev.azure.com** en lugar de **visualstudio.com**, que se usaba en formatos anteriores. El formato de dirección URL anterior `https://<accountname>.visualstudio.com/<projectname>/_git/<repositoryname>` está en desuso pero todavía se admite, aunque se prefiere el nuevo formato.


```powershell-interactive
New-AzAutomationSourceControl -Name SCReposGit -RepoUrl https://dev.azure.com/<accountname>/<adoprojectname>/_git/<repositoryname> -SourceType VsoGit -AccessToken <secureStringofPAT> -Branch master -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName> -FolderPath "/Runbooks"
```

#### <a name="create-source-control-connection-for-azure-repos-tfvc"></a>Creación de la conexión del control de código fuente para Azure Repos (TFVC)

> [!NOTE]
> Azure Repos (TFVC) usa una dirección URL que accede a **dev.azure.com** en lugar de **visualstudio.com**, que se usaba en formatos anteriores. El formato de dirección URL anterior `https://<accountname>.visualstudio.com/<projectname>/_versionControl` está en desuso pero todavía se admite, aunque se prefiere el nuevo formato.

```powershell-interactive
New-AzAutomationSourceControl -Name SCReposTFVC -RepoUrl https://dev.azure.com/<accountname>/<adoprojectname>/_git/<repositoryname> -SourceType VsoTfvc -AccessToken <secureStringofPAT> -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName> -FolderPath "/Runbooks"
```

#### <a name="personal-access-token-pat-permissions"></a>Permisos del token de acceso personal (PAT)

El control de código fuente requiere algunos permisos mínimos para los tokens de acceso personal. Las subsecciones siguientes contienen los permisos mínimos necesarios para GitHub y Azure Repos.

##### <a name="minimum-pat-permissions-for-github"></a>Permisos mínimos de PAT para GitHub

En la tabla siguiente se definen los permisos mínimos necesarios de PAT para GitHub. Para más información sobre cómo crear un token de acceso personal en GitHub, consulte [Creación de un token de acceso personal para la línea de comandos](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/).

|Ámbito  |Descripción  |
|---------|---------|
|**`repo`**     |         |
|`repo:status`     | Acceder al estado de confirmación         |
|`repo_deployment`      | Acceder al estado de implementación         |
|`public_repo`     | Acceder a los repositorios públicos         |
|`repo:invite` | Acceder a invitaciones de repositorio |
|`security_events` | Leer y escribir eventos de seguridad |
|**`admin:repo_hook`**     |         |
|`write:repo_hook`     | Escribir los enlaces de repositorio         |
|`read:repo_hook`|Leer los enlaces de repositorio|

##### <a name="minimum-pat-permissions-for-azure-repos"></a>Permisos mínimos de PAT para Azure Repos

En la tabla siguiente se definen los permisos mínimos necesarios de PAT para Azure Repos. Para más información sobre cómo crear un token de acceso personal en Azure Repos, consulte [Autenticación del acceso con tokens de acceso personal](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate).

| Ámbito  |  Tipo de acceso  |
|---------| ----------|
| `Code`      | Lectura  |
| `Project and team` | Lectura |
| `Identity` | Lectura     |
| `User profile` | Lectura     |
| `Work items` | Lectura    |
| `Service connections` | Leer, consultar, administrar<sup>1</sup>    |

<sup>1</sup> El permiso `Service connections` solo es necesario si ha habilitado la sincronización automática.

## <a name="synchronize-with-source-control"></a>Sincronización con el control de código fuente

Siga estos pasos para sincronizar con el control de código fuente.

1. Seleccione el código fuente en la tabla de la página Control de código fuente.

2. Haga clic en **Iniciar sincronización** para iniciar el proceso de sincronización.

3. Para ver el estado del trabajo de sincronización actual o de los anteriores, haga clic en la pestaña **Trabajos de sincronización**.

4. En el menú desplegable **Control de código fuente**, seleccione un mecanismo de control de código fuente.

    ![Estado de sincronización](./media/source-control-integration/sync-status.png)

5. Al hacer clic en un trabajo, verá la salida del trabajo. El siguiente es un ejemplo de la salida de un trabajo de sincronización de control de código fuente.

    ```output
    ===================================================================

    Azure Automation Source Control.
    Supported runbooks to sync: PowerShell Workflow, PowerShell Scripts, DSC Configurations, Graphical, and Python 2.

    Setting AzEnvironment.

    Getting AzureRunAsConnection.

    Logging in to Azure...

    Source control information for syncing:

    [Url = https://ContosoExample.visualstudio.com/ContosoFinanceTFVCExample/_versionControl] [FolderPath = /Runbooks]

    Verifying url: https://ContosoExample.visualstudio.com/ContosoFinanceTFVCExample/_versionControl

    Connecting to VSTS...

    Source Control Sync Summary:

    2 files synced:
     - ExampleRunbook1.ps1
     - ExampleRunbook2.ps1

    ==================================================================

    ```

6. Puede encontrar opciones de registro adicionales si selecciona **Todos los registros** en la página Resumen del trabajo de sincronización del control de código fuente. Estas entradas de registro adicionales pueden ayudar a solucionar los problemas que pueden surgir al usar el control de código fuente.

## <a name="disconnect-source-control"></a>Desconexión del control de código fuente

Para desconectarse de un repositorio de control de código fuente:

1. Abra **Control de código fuente** en **Configuración de la cuenta** de la cuenta de Automation.

2. Seleccione el mecanismo de control de código fuente que desea eliminar.

3. En la página Resumen del trabajo de sincronización del control de código fuente, haga clic en **Eliminar**.

## <a name="handle-encoding-issues"></a>Control de los problemas de codificación

Si varias personas editan runbooks en el repositorio de control de código fuente con diferentes editores, pueden darse problemas de codificación. Para más información sobre esta situación, consulte [Causas comunes de problemas de codificación](/powershell/scripting/components/vscode/understanding-file-encoding#common-causes-of-encoding-issues).

## <a name="update-the-pat"></a>Actualización del token de acceso personal

Actualmente, no puede usar Azure Portal para actualizar el token de acceso personal en el control de código fuente. Después de que el token de acceso personal haya expirado o se haya revocado, podrá actualizar el control de código fuente con un nuevo token de acceso de las siguientes maneras:

* Mediante la [API REST](/rest/api/automation/sourcecontrol/update).
* Mediante el cmdlet [Update-AzAutomationSourceControl](/powershell/module/az.automation/update-azautomationsourcecontrol).

## <a name="next-steps"></a>Pasos siguientes

* Para integrar el control de código fuente en Azure Automation, vea [Azure Automation: integración del control de código fuente en Azure Automation](https://azure.microsoft.com/blog/azure-automation-source-control-13/).  
* Para integrar el control de código fuente del runbook con Visual Studio Codespaces, consulte [Azure Automation: Integración del control de código fuente del runbook mediante Visual Studio Codespaces](https://azure.microsoft.com/blog/azure-automation-integrating-runbook-source-control-using-visual-studio-online/).
