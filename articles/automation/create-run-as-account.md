---
title: Creación de una cuenta de ejecución de Azure Automation
description: En este artículo se describe cómo crear una cuenta de ejecución de Azure Automation con PowerShell o desde Azure Portal.
services: automation
ms.subservice: process-automation
ms.date: 04/29/2021
ms.topic: conceptual
ms.openlocfilehash: 368bbfd0bc4dd0a3c8c2792487db52e8585c2ea1
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108277631"
---
# <a name="how-to-create-an-azure-automation-run-as-account"></a>Creación de una cuenta de ejecución de Azure Automation

Las cuentas de ejecución de Azure Automation proporcionan autenticación para administrar recursos en el modelo de implementación clásico de Azure o Azure Resource Manager mediante runbooks de Automation y otras características de Automation. En este artículo se describe cómo crear una cuenta de ejecución o de ejecución clásica desde Azure Portal o Azure PowerShell.

Al crear la cuenta de ejecución o la cuenta de ejecución clásica en Azure Portal, se usa de forma predeterminada un certificado autofirmado. Si quiere usar un certificado emitido por su entidad de certificación (CA) empresarial o de terceros, puede usar el [script de PowerShell para crear una cuenta de ejecución](#powershell-script-to-create-a-run-as-account).

## <a name="create-account-in-azure-portal"></a>Creación de una cuenta en Azure Portal

Realice los pasos que se describen a continuación para actualizar su cuenta de Azure Automation en Azure Portal. Las cuentas de ejecución y de ejecución clásica se crean por separado. Si no es necesario administrar los recursos clásicos, basta con crear la cuenta de ejecución de Azure.

1. Inicie sesión en Azure Portal con una cuenta que sea miembro del rol Administradores de suscripciones y coadministrador de la suscripción.

2. Busque y seleccione **Cuentas de Automation**.

3. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.

4. En el panel izquierdo, seleccione **Cuentas de ejecución** en la sección **Configuración de la cuenta**.

    :::image type="content" source="media/create-run-as-account/automation-account-properties-pane.png" alt-text="Seleccione la opción Cuenta de ejecución.":::

5. En función de la cuenta que necesite, utilice el panel **+Cuenta de ejecución de Azure** o **+Cuenta de ejecución de Azure clásica**. Después de revisar la información general, haga clic en **Crear**.

    :::image type="content" source="media/create-run-as-account/automation-account-create-run-as.png" alt-text="Selección de la opción para crear una cuenta de ejecución":::

6. Mientras Azure crea la cuenta de ejecución, se puede seguir el progreso en **Notificaciones** en el menú. También se muestra un banner que indica que se está creando la cuenta. El proceso puede tardar unos minutos en completarse.

## <a name="create-account-using-powershell"></a>Creación de una cuenta con PowerShell

En la lista siguiente se proporcionan los requisitos para crear una cuenta de ejecución en PowerShell mediante un script proporcionado. Estos requisitos se aplican a ambos tipos de cuentas de ejecución.

* Windows 10 o Windows Server 2016 con módulos de Azure Resource Manager 3.4.1 y versiones posteriores. El script de PowerShell no admite versiones anteriores de Windows.
* Azure PowerShell 6.2.4 o posterior. Para más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-az-ps).
* Una cuenta de Automation, a la que se hace referencia como el valor de los parámetros `AutomationAccountName` y `ApplicationDisplayName`.
* Permisos equivalentes a los que se muestran en [Permisos para configurar cuentas de ejecución](automation-security-overview.md#permissions).

Para obtener los valores de `AutomationAccountName`, `SubscriptionId` y `ResourceGroupName`, que son los parámetros necesarios para el script de PowerShell, complete los pasos siguientes.

1. Inicie sesión en Azure Portal.

1. Busque y seleccione **Cuentas de Automation**.

1. En la página Cuentas de Automation, seleccione su cuenta de Automation en la lista.

1. Seleccione **Propiedades** en el panel izquierdo.

1. Anote los valores de **Nombre**, **Id. de suscripción** y **Grupo de recursos** en la página **Propiedades**.

   ![Página de propiedades de la cuenta de Automation](media/create-run-as-account/automation-account-properties.png)

### <a name="powershell-script-to-create-a-run-as-account"></a>Script de PowerShell para crear una cuenta de ejecución

El script de PowerShell incluye compatibilidad con varias configuraciones.

* Creación de una cuenta de ejecución o una cuenta de ejecución clásica mediante un certificado autofirmado.
* Cree una cuenta de ejecución o una cuenta de ejecución clásica mediante un certificado emitido por su entidad de certificación (CA) empresarial o de terceros.
* Creación de una cuenta de ejecución o una cuenta de ejecución clásica mediante un certificado autofirmado en la nube de Azure Government.

1. Descargue y guarde el script en una carpeta local con el siguiente comando.

    ```powershell
    wget https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/AzRunAs/Create-RunAsAccount.ps1 -outfile Create-RunAsAccount.ps1
    ```

2. Inicie PowerShell con permisos de usuario elevados y navegue hasta la carpeta que contiene el script.

3. Ejecute uno de los siguientes comandos para crear una cuenta de ejecución o de ejecución clásica en función de sus requisitos.

    * Cree una cuenta de ejecución mediante un certificado autofirmado.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $false
        ```

    * Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true
        ```

    * Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado de empresa.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication>  -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnterpriseCertPathForRunAsAccount <EnterpriseCertPfxPathForRunAsAccount> -EnterpriseCertPlainPasswordForRunAsAccount <StrongPassword> -EnterpriseCertPathForClassicRunAsAccount <EnterpriseCertPfxPathForClassicRunAsAccount> -EnterpriseCertPlainPasswordForClassicRunAsAccount <StrongPassword>
        ```

        Si creó una cuenta de ejecución clásica con un certificado público de empresa (archivo .cer) , use este certificado. El script la crea y la guarda en la carpeta de archivos temporales del equipo, en el perfil de usuario `%USERPROFILE%\AppData\Local\Temp` que usó para ejecutar la sesión de PowerShell. Consulte [Carga de un certificado de API de administración en Azure Portal](../cloud-services/cloud-services-configure-ssl-certificate-portal.md).

    * Creación de una cuenta de ejecución y una cuenta de ejecución clásica mediante un certificado autofirmado en la nube de Azure Government

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnvironmentName AzureUSGovernment
        ```

4. Después de ejecutar el script, se le pedirá que se autentique en Azure. Inicie sesión con una cuenta que sea miembro del rol de administradores de la suscripción. Si va a crear una cuenta de ejecución clásica, su cuenta debe ser un coadministrador de la suscripción.

## <a name="next-steps"></a>Pasos siguientes

* Para empezar a trabajar con runbooks de PowerShell, vea [Tutorial: Creación de un runbook de PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).

* Para empezar a trabajar con un runbook de Python 3, consulte [Tutorial: Creación de un runbook de Python 3](learn/automation-tutorial-runbook-textual-python-3.md).
