---
title: Protección de la entrega de webhooks con Azure AD en Azure Event Grid
description: Describe cómo enviar eventos a puntos de conexión HTTPS protegidos por Azure Active Directory mediante Azure Event Grid
ms.topic: how-to
ms.date: 04/13/2021
ms.openlocfilehash: 6a0f9059e17d96d497b425abc9749e69c5ab4d41
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107575554"
---
# <a name="publish-events-to-azure-active-directory-protected-endpoints"></a>Publicación de eventos en puntos de conexión protegidos por Azure Active Directory
En este artículo se describe cómo usar Azure Active Directory (Azure AD) para proteger la conexión entre su **suscripción de eventos** y el **punto de conexión de webhook**. Para una introducción a las entidades de servicio y aplicaciones de Azure AD, consulte [Introducción a la plataforma de identidad de Microsoft (versión 2.0)](../active-directory/develop/v2-overview.md).

En este artículo se usa Azure Portal para la demostración; sin embargo, la característica también se puede habilitar mediante la CLI, PowerShell o los SDK.

> [!IMPORTANT]
> La comprobación de acceso adicional se introdujo como parte de la creación o actualización de la suscripción de eventos el 30 de marzo de 2021 para solucionar una vulnerabilidad de seguridad. La entidad de servicio del cliente de suscriptor debe ser un propietario o tener un rol asignado en la entidad de servicio de la aplicación de destino. Vuelva a configurar la aplicación de AAD siguiendo las nuevas instrucciones que se indican a continuación.


## <a name="create-an-azure-ad-application"></a>Creación de una aplicación de Azure AD
Registre el webhook con Azure AD mediante la creación de una aplicación Azure AD para el punto de conexión protegido. Consulte [Escenario: API web protegida](https://docs.microsoft.com/azure/active-directory/develop/scenario-protected-web-api-overview). Configure la API protegida para que la llame una aplicación de demonio.
    
## <a name="enable-event-grid-to-use-your-azure-ad-application"></a>Habilitación de Event Grid para que use la aplicación de Azure AD
En esta sección se muestra cómo habilitar Event Grid para usar la aplicación Azure AD. 

> [!NOTE]
> Debe ser miembro del [rol Administrador de aplicaciones de Azure AD](../active-directory/roles/permissions-reference.md#all-roles) para ejecutar este script.

### <a name="connect-to-your-azure-tenant"></a>Conéctese a su inquilino de Azure
En primer lugar, conéctese a su inquilino de Azure mediante el comando `Connect-AzureAD`. 

```PowerShell
$myWebhookAadTenantId = "<Your Webhook's Azure AD tenant id>"

Connect-AzureAD -TenantId $myWebhookAadTenantId
```

### <a name="create-microsofteventgrid-service-principal"></a>Cree la entidad de servicio Microsoft.EventGrid
Ejecute el siguiente script para crear la entidad de servicio para **Microsoft.EventGrid** si aún no existe. 

```PowerShell
# This is the "Azure Event Grid" Azure Active Directory (AAD) AppId
$eventGridAppId = "4962773b-9cdb-44cf-a8bf-237846a00ab7"

# Create the "Azure Event Grid" AAD Application service principal if it doesn't exist
$eventGridSP = Get-AzureADServicePrincipal -Filter ("appId eq '" + $eventGridAppId + "'")
if ($eventGridSP -match "Microsoft.EventGrid")
{
    Write-Host "The Service principal is already defined.`n"
} else {
    # Create a service principal for the "Azure Event Grid" AAD Application and add it to the role
    Write-Host "Creating the Azure Event Grid service principal"
    $eventGridSP = New-AzureADServicePrincipal -AppId $eventGridAppId
}
```

### <a name="create-a-role-for-your-application"></a>Cree un rol para la aplicación   
Ejecute el siguiente script para crear un rol para la aplicación Azure AD. En este ejemplo, el nombre del rol es: **AzureEventGridSecureWebhookSubscriber**. Modifique el valor `$myTenantId` del script de PowerShell para usar el id. de inquilino de Azure AD y `$myAzureADApplicationObjectId` con el id. de objeto de la aplicación Azure AD.

```PowerShell
# This is your Webhook's Azure AD Application's ObjectId. 
$myWebhookAadApplicationObjectId = "<Your webhook's aad application object id>"

# This is the name of the new role we will add to your Azure AD Application
$eventGridRoleName = "AzureEventGridSecureWebhookSubscriber"
       
# Create an application role of given name and description
Function CreateAppRole([string] $Name, [string] $Description)
{
    $appRole = New-Object Microsoft.Open.AzureAD.Model.AppRole
    $appRole.AllowedMemberTypes = New-Object System.Collections.Generic.List[string]
    $appRole.AllowedMemberTypes.Add("Application");
    $appRole.AllowedMemberTypes.Add("User");
    $appRole.DisplayName = $Name
    $appRole.Id = New-Guid
    $appRole.IsEnabled = $true
    $appRole.Description = $Description
    $appRole.Value = $Name;
    return $appRole
}
       
# Get my Azure AD Application, it's roles and service principal
$myApp = Get-AzureADApplication -ObjectId $myWebhookAadApplicationObjectId
$myAppRoles = $myApp.AppRoles

Write-Host "App Roles before addition of new role.."
Write-Host $myAppRoles
       
# Create the role if it doesn't exist
if ($myAppRoles -match $eventGridRoleName)
{
    Write-Host "The Azure Event Grid role is already defined.`n"
} else {      
    # Add our new role to the Azure AD Application
    Write-Host "Creating the Azure Event Grid role in Azure Ad Application: " $myWebhookAadApplicationObjectId
    $newRole = CreateAppRole -Name $eventGridRoleName -Description "Azure Event Grid Role"
    $myAppRoles.Add($newRole)
    Set-AzureADApplication -ObjectId $myApp.ObjectId -AppRoles $myAppRoles
}

# print application's roles
Write-Host "My Azure AD Application's Roles: "
Write-Host $myAppRoles

```

### <a name="create-role-assignment-for-the-client-creating-event-subscription"></a>Creación de una asignación de roles para el cliente que crea una suscripción de eventos
La asignación de roles se debe crear en la aplicación de Azure AD Webhook para la aplicación de AAD o el usuario de AAD que crea la suscripción de eventos. Use uno de los scripts siguientes en función de si una aplicación de AAD o un usuario de AAD está creando la suscripción de eventos.

> [!IMPORTANT]
> La comprobación de acceso adicional se introdujo como parte de la creación o actualización de la suscripción de eventos el 30 de marzo de 2021 para solucionar una vulnerabilidad de seguridad. La entidad de servicio del cliente de suscriptor debe ser un propietario o tener un rol asignado en la entidad de servicio de la aplicación de destino. Vuelva a configurar la aplicación de AAD siguiendo las nuevas instrucciones que se indican a continuación.

#### <a name="create-role-assignment-for-an-event-subscription-aad-app"></a>Creación de una asignación de roles para la aplicación de AAD de suscripción de eventos 

```powershell
# This is the app id of the application which will create event subscription. Set to $null if you are not assigning the role to app.
$eventSubscriptionWriterAppId = "<the app id of the application which will create event subscription>"

$myServicePrincipal = Get-AzureADServicePrincipal -Filter ("appId eq '" + $myApp.AppId + "'")

$eventSubscriptionWriterSP = Get-AzureADServicePrincipal -Filter ("appId eq '" + $eventSubscriptionWriterAppId + "'")
if ($eventSubscriptionWriterSP -eq $null)
{
        $eventSubscriptionWriterSP = New-AzureADServicePrincipal -AppId $eventSubscriptionWriterAppId
}

Write-Host "Creating the Azure Ad App Role assignment for application: " $eventSubscriptionWriterAppId
$eventGridAppRole = $myApp.AppRoles | Where-Object -Property "DisplayName" -eq -Value $eventGridRoleName
New-AzureADServiceAppRoleAssignment -Id $eventGridAppRole.Id -ResourceId $myServicePrincipal.ObjectId -ObjectId $eventSubscriptionWriterSP.ObjectId -PrincipalId $eventSubscriptionWriterSP.ObjectId
```

#### <a name="create-role-assignment-for-an-event-subscription-aad-user"></a>Creación de una asignación de roles para el usuario de AAD de suscripción de eventos 

```powershell
# This is the user principal name of the user who will create event subscription. Set to $null if you are not assigning the role to user.
$eventSubscriptionWriterUserPrincipalName = "<the user principal name of the user who will create event subscription>"

$myServicePrincipal = Get-AzureADServicePrincipal -Filter ("appId eq '" + $myApp.AppId + "'")
    
Write-Host "Creating the Azure Ad App Role assignment for user: " $eventSubscriptionWriterUserPrincipalName
$eventSubscriptionWriterUser = Get-AzureAdUser -ObjectId $eventSubscriptionWriterUserPrincipalName
$eventGridAppRole = $myApp.AppRoles | Where-Object -Property "DisplayName" -eq -Value $eventGridRoleName
New-AzureADUserAppRoleAssignment -Id $eventGridAppRole.Id -ResourceId $myServicePrincipal.ObjectId -ObjectId $eventSubscriptionWriterUser.ObjectId -PrincipalId $eventSubscriptionWriterUser.ObjectId
```

### <a name="create-role-assignment-for-event-grid-service-principal"></a>Creación de una asignación de roles para entidad de servicio de Event Grid
Ejecute el comando New-AzureADServiceAppRoleAssignment para asignar la entidad de servicio de Event Grid al rol que creó en el paso anterior.

```powershell
$eventGridAppRole = $myApp.AppRoles | Where-Object -Property "DisplayName" -eq -Value $eventGridRoleName
New-AzureADServiceAppRoleAssignment -Id $eventGridAppRole.Id -ResourceId $myServicePrincipal.ObjectId -ObjectId $eventGridSP.ObjectId -PrincipalId $eventGridSP.ObjectId
```

Ejecute los siguientes comandos para obtener la información que usará más adelante.

```powershell
Write-Host "My Webhook's Azure AD Tenant Id:  $myWebhookAadTenantId"
Write-Host "My Webhook's Azure AD Application Id: $($myApp.AppId)"
Write-Host "My Webhook's Azure AD Application ObjectId Id$($myApp.ObjectId)"
```

    
## <a name="configure-the-event-subscription"></a>Configuración de la suscripción de eventos
Al crear una suscripción de eventos, siga los siguientes pasos:

1. Para el tipo de punto de conexión, seleccione **Webhook**. 
1. Especifique el **identificador URI** del punto de conexión.

    ![Selección del tipo de punto de conexión webhook](./media/secure-webhook-delivery/select-webhook.png)
1. Seleccione la pestaña **Características adicionales** en la parte superior de la página **Crear suscripciones a eventos** .
1. En la pestaña **Características adicionales**, siga estos pasos:
    1. Seleccione **Usar autenticación de AAD** y configure el identificador de inquilino y el identificador de aplicación:
    1. Copie el identificador de inquilino de Azure AD de la salida del script e introdúzcalo en el campo **Identificador de inquilino de AAD**.
    1. Copie el identificador de aplicación de Azure AD de la salida del script e introdúzcalo en el campo **Identificador de aplicación de AAD**. Como alternativa, puede usar el URI del identificador de aplicación de AAD. Para obtener más información sobre la URI del identificador, consulte [este artículo](../app-service/configure-authentication-provider-aad.md).

        ![Acción de webhook seguro](./media/secure-webhook-delivery/aad-configuration.png)



## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para más información sobre la clave de autenticación, vea [Seguridad y autenticación de Event Grid](security-authentication.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
