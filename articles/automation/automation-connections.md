---
title: Administración de conexiones en Azure Automation
description: En este artículo se explica cómo administrar las conexiones de Azure Automation a aplicaciones o servicios externos y cómo trabajar con ellas en runbooks.
services: automation
ms.subservice: shared-capabilities
ms.date: 12/22/2020
ms.topic: conceptual
ms.custom: has-adal-ref, devx-track-azurepowershell
ms.openlocfilehash: 4b5983d6ea4ea9065d1292a2c5ee0c094cea093b
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834901"
---
# <a name="manage-connections-in-azure-automation"></a>Administración de conexiones en Azure Automation

Un recurso de conexión de Azure Automation contiene la información que se muestra a continuación. Esta información es necesaria para conectarse a aplicaciones o servicios externos desde un runbook o la configuración de DSC. 

* Información necesaria para la autenticación, como el nombre de usuario y la contraseña
* Información de conexión, como la dirección URL o el puerto

El recurso de conexión mantiene juntas todas las propiedades para conectarse a una aplicación determinada, por lo que no es necesario crear varias variables. Puede editar los valores de una conexión en un lugar y pasar el nombre de una conexión a un runbook o una configuración de DSC en un solo parámetro. El runbook o la configuración acceden a las propiedades de una conexión mediante el cmdlet `Get-AutomationConnection` interno.

Cuando crea una conexión, debe especificar un tipo de conexión. El tipo de conexión es una plantilla que define un conjunto de propiedades. Puede agregar un tipo de conexión a Azure Automation mediante un módulo de integración con un archivo de metadatos. Se puede crear un tipo de conexión mediante la [API de Azure Automation](/previous-versions/azure/reference/mt163818(v=azure.100)) si el módulo de integración incluye un tipo de conexión y se importa en su cuenta de Automation. 

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Azure Automation con una clave única que se genera para cada cuenta de Automation. Azure Automation almacena la clave en la instancia de Key Vault administrada por el sistema. Antes de almacenar un recurso seguro, Azure Automation carga la clave desde Key Vault y después la usa para cifrar el recurso. 

## <a name="connection-types"></a>Tipos de conexión

Azure Automation pone a disposición de los usuarios los siguientes tipos de conexión integrados:

* `Azure`: representa una conexión que se usa para administrar recursos clásicos.
* `AzureServicePrincipal`: representa una conexión que se usa en la cuenta de ejecución de Azure.
* `AzureClassicCertificate`: representa una conexión que se usa en la cuenta de ejecución de Azure clásico.

En la mayoría de los casos no es necesario crear un recurso de conexión, ya que se crea con la creación de una [cuenta de ejecución](automation-security-overview.md).

## <a name="powershell-cmdlets-to-access-connections"></a>Cmdlets de PowerShell para acceder a las conexiones

Los cmdlets de la tabla siguiente permiten crear y administrar conexiones de Automation con PowerShell. Se suministran como un componente de los [módulos Az](shared-resources/modules.md#az-modules).

|Cmdlet|Descripción|
|---|---|
|[Get-AzAutomationConnection](/powershell/module/az.automation/get-azautomationconnection)|Recupera información acerca de una conexión.|
|[New-AzAutomationConnection](/powershell/module/az.automation/new-azautomationconnection)|Crea una conexión nueva.|
|[Remove-AzAutomationConnection](/powershell/module/Az.Automation/Remove-AzAutomationConnection)|Quita una conexión existente.|
|[Set-AzAutomationConnectionFieldValue](/powershell/module/Az.Automation/Set-AzAutomationConnectionFieldValue)|Establece el valor de un campo determinado para una conexión existente.|

## <a name="internal-cmdlets-to-access-connections"></a>Cmdlets internos para acceder a las conexiones

El cmdlet interno de la tabla siguiente se usa para acceder a las conexiones de los runbooks y las configuraciones de DSC. Este cmdlet se incluye en el módulo global `Orchestrator.AssetManagement.Cmdlets`. Para obtener más información, consulte [Cmdlets internos](shared-resources/modules.md#internal-cmdlets).

|Cmdlet interno|Descripción|
|---|---|
|`Get-AutomationConnection` | Recupera los valores de los distintos campos de la conexión y los devuelve como una [tabla hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables). Esta tabla hash se puede usar después con los comandos adecuados en el runbook o la configuración de DSC.|

>[!NOTE]
>Evite el uso de variables con el parámetro `Name` de `Get-AutomationConnection`. El uso de variables en este caso puede complicar la detección de dependencias entre runbooks o configuraciones de DSC y los recursos de conexión durante el tiempo de diseño.

## <a name="python-functions-to-access-connections"></a>Funciones de Python para acceder a las conexiones

La función de la tabla siguiente se usa para acceder a las conexiones de un runbook de Python 2 y 3. Los runbooks de Python 3 se encuentran actualmente en versión preliminar.

| Función | Descripción |
|:---|:---|
| `automationassets.get_automation_connection` | Recupera una conexión. Devuelve un diccionario con las propiedades de la conexión. |

> [!NOTE]
> Para acceder a las funciones del activo, debe importar el módulo `automationassets` en la parte superior del runbook de Python.

## <a name="create-a-new-connection"></a>Crear una conexión

### <a name="create-a-new-connection-with-the-azure-portal"></a>Creación de una conexión con Azure Portal

Para crear una conexión nueva en Azure Portal:

1. En la cuenta de Automation, haga clic en **Conexiones** en **Recursos compartidos**.
2. En la página Conexiones, haga clic en **+ Agregar una conexión**.
4. En el campo **Tipo** del panel Nueva conexión, seleccione el tipo de conexión que desea crear. Las opciones son `Azure`, `AzureServicePrincipal` y `AzureClassicCertificate`. 
5. El formulario presenta las propiedades del tipo de conexión que ha elegido. Complete el formulario y haga clic en **Crear** para guardar la conexión nueva.

### <a name="create-a-new-connection-with-windows-powershell"></a>Creación de una conexión con Windows PowerShell

Cree una conexión con Windows PowerShell mediante el cmdlet `New-AzAutomationConnection`. Este cmdlet tiene un parámetro `ConnectionFieldValues`que espera una tabla hash que defina los valores de cada una de las propiedades definidas por el tipo de conexión.

Puede usar los siguientes comandos de ejemplo como alternativa a la creación de la cuenta de ejecución desde el portal para crear un recurso de conexión.

```powershell
$ConnectionAssetName = "AzureRunAsConnection"
$ConnectionFieldValues = @{"ApplicationId" = $Application.ApplicationId; "TenantId" = $TenantID.TenantId; "CertificateThumbprint" = $Cert.Thumbprint; "SubscriptionId" = $SubscriptionId}
New-AzAutomationConnection -ResourceGroupName $ResourceGroup -AutomationAccountName $AutomationAccountName -Name $ConnectionAssetName -ConnectionTypeName AzureServicePrincipal -ConnectionFieldValues $ConnectionFieldValues
```

Al crear la cuenta de Automation, se incluyen varios módulos globales de forma predeterminada, junto con el tipo de conexión `AzureServicePrincipal` para crear el recurso de conexión `AzureRunAsConnection`. Si intenta crear un recurso de conexión para conectarse a un servicio o a una aplicación con un método de autenticación diferente, la operación generará un error porque el tipo de conexión no se ha definido todavía en la cuenta de Automation. Para más información sobre cómo crear su propio tipo de conexión para un módulo personalizado, consulte [Agregar un tipo de conexión](#add-a-connection-type).

## <a name="add-a-connection-type"></a>Agregar un tipo de conexión

Si el runbook o la configuración de DSC se conectan a un servicio externo, debe definir un tipo de conexión en un [módulo personalizado](shared-resources/modules.md#custom-modules) denominado "módulo de integración". Este módulo incluye un archivo de metadatos que especifica las propiedades del tipo de conexión, se denomina **&lt;ModuleName&gt;-Automation.json** y se encuentra en la carpeta del módulo del archivo **.zip** comprimido. Este archivo contiene los campos de una conexión que se necesitan para conectarse al sistema o servicio que representa el módulo. Con este archivo, puede establecer los nombres de campo, los tipos de datos, el estado de cifrado y el estado opcional del tipo de conexión. 

El ejemplo siguiente es una plantilla en el formato de archivo **.json** que define propiedades de nombre de usuario y contraseña de un tipo de conexión predeterminado denominado `MyModuleConnection`:

```json
{
   "ConnectionFields": [
   {
      "IsEncrypted":  false,
      "IsOptional":  true,
      "Name":  "Username",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  true,
      "IsOptional":  false,
      "Name":  "Password",
      "TypeName":  "System.String"
   }
   ],
   "ConnectionTypeName":  "MyModuleConnection",
   "IntegrationModuleName":  "MyModule"
}
```

## <a name="get-a-connection-in-a-runbook-or-dsc-configuration"></a>Obtención de una conexión en un runbook o una configuración de DSC

Recupere una conexión de un runbook o una configuración de DSC con el cmdlet `Get-AutomationConnection` interno. Este cmdlet es preferible al cmdlet `Get-AzAutomationConnection`, ya que recupera los valores de conexión en lugar de la información sobre la conexión.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

El ejemplo siguiente muestra cómo utilizar la cuenta de ejecución para la autenticación con recursos de Azure Resource Manager en su runbook. Utiliza el recurso de conexión que representa la cuenta de ejecución, que hace referencia a la entidad de servicio basada en certificado.

```powershell
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
```

# <a name="python"></a>[Python](#tab/python2)

En el ejemplo siguiente se muestra cómo autenticarse mediante la conexión de ejecución en un runbook de Python 2 y 3.

```python
""" Tutorial to show how to authenticate against Azure resource manager resources """
import azure.mgmt.resource
import automationassets

def get_automation_runas_credential(runas_connection):
    """ Returns credentials to authenticate against Azure resource manager """
    from OpenSSL import crypto
    from msrestazure import azure_active_directory
    import adal

    # Get the Azure Automation Run As service principal certificate
    cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
    pks12_cert = crypto.load_pkcs12(cert)
    pem_pkey = crypto.dump_privatekey(
        crypto.FILETYPE_PEM, pks12_cert.get_privatekey())

    # Get Run As connection information for the Azure Automation service principal
    application_id = runas_connection["ApplicationId"]
    thumbprint = runas_connection["CertificateThumbprint"]
    tenant_id = runas_connection["TenantId"]

    # Authenticate with service principal certificate
    resource = "https://management.core.windows.net/"
    authority_url = ("https://login.microsoftonline.com/" + tenant_id)
    context = adal.AuthenticationContext(authority_url)
    return azure_active_directory.AdalAuthentication(
        lambda: context.acquire_token_with_client_certificate(
            resource,
            application_id,
            pem_pkey,
            thumbprint)
    )


# Authenticate to Azure using the Azure Automation Run As service principal
runas_connection = automationassets.get_automation_connection(
    "AzureRunAsConnection")
azure_credential = get_automation_runas_credential(runas_connection)
```

---

### <a name="graphical-runbook-examples"></a>Ejemplos de runbook gráfico

Puede agregar una actividad para el cmdlet `Get-AutomationConnection` interno a un runbook gráfico. Haga clic con el botón derecho en la conexión en el panel Biblioteca del editor gráfico y seleccione **Agregar al lienzo**.

![agregar a lienzo](media/automation-connections/connection-add-canvas.png)

La imagen siguiente muestra un ejemplo de cómo usar un objeto de conexión en un runbook gráfico. Este ejemplo utiliza el conjunto de datos `Constant value` para la actividad `Get RunAs Connection`, que usa un objeto de conexión para la autenticación. Aquí se usa un [vínculo de canalización](automation-graphical-authoring-intro.md#use-links-for-workflow), debido a que el parámetro `ServicePrincipalCertificate` establecido espera un solo objeto.

![obtener conexiones](media/automation-connections/automation-get-connection-object.png)

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre los cmdlets que se usan para acceder a las conexiones, consulte [Administración de módulos en Azure Automation](shared-resources/modules.md).
* Para obtener información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](automation-runbook-execution.md).
* Para más información sobre las configuraciones de DSC, consulte [Introducción a State Configuration](automation-dsc-overview.md).
