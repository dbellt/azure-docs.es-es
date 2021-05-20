---
title: 'Invitar a usuarios internos a la colaboración B2B: Azure AD'
description: Si tiene cuentas de usuario internas para asociados, distribuidores, proveedores y otros invitados, puede cambiar a la colaboración B2B de Azure AD invitándolos a que inicien sesión con sus propias credenciales externas o su inicio de sesión. Use PowerShell o la API de invitación de Microsoft Graph.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/03/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.collection: M365-identity-device-management
ms.openlocfilehash: aa2356cf45c1b1a04fac14bff2888d031ca5a423
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109750994"
---
# <a name="invite-internal-users-to-b2b-collaboration"></a>Invitar a usuarios internos a la colaboración B2B

Antes de la disponibilidad de la colaboración B2B de Azure AD, las organizaciones pueden colaborar con distribuidores, proveedores y otros usuarios invitados mediante la configuración de credenciales internas para ellos. Si tiene usuarios de invitado internos como estos, puede invitarlos a usar la colaboración B2B en su lugar. Estos usuarios invitados de B2B podrán usar sus propias identidades y credenciales para iniciar sesión, y no tendrá que mantener contraseñas ni administrar los ciclos de vida de las cuentas.

El envío de una invitación a una cuenta interna existente le permite conservar el identificador de objeto, el UPN, la pertenencia a grupos y las asignaciones de aplicaciones de ese usuario. No es necesario eliminar y volver a invitar al usuario manualmente ni reasignar los recursos. Para invitar al usuario, use la API de invitación para pasar el objeto de usuario interno y la dirección de correo electrónico del usuario invitado junto con la invitación. Cuando el usuario acepta la invitación, el servicio B2B cambia el objeto de usuario interno existente a un usuario de B2B. A partir de ahí, el usuario debe iniciar sesión en los servicios de recursos de nube con sus credenciales de B2B.

## <a name="things-to-consider"></a>Aspectos que se deben tener en cuenta:

- **Acceso a recursos locales**: Una vez que se ha invitado al usuario a la colaboración B2B, todavía puede usar sus credenciales internas para acceder a los recursos locales. Para evitarlo, puede restablecer o cambiar la contraseña en la cuenta interna. La excepción es [Autenticación con código de acceso de un solo uso por correo electrónico](one-time-passcode.md). Si el método de autenticación del usuario se cambia a código de acceso de un solo uso, ya no podrá usar sus credenciales internas.

- **Facturación**: Esta característica no cambia el UserType del usuario, por lo que no cambia automáticamente el modelo de facturación del usuario a los [precios de usuarios activos mensuales (MAU) de External Identities](external-identities-pricing.md). Para activar los precios de MAU para el usuario, cambie UserType para el usuario a `guest`. Además tenga en cuenta que el inquilino de Azure AD debe estar [vinculado a una suscripción de Azure](external-identities-pricing.md#link-your-azure-ad-tenant-to-a-subscription) para activar la facturación de MAU.

- **La invitación es unidireccional**: Puede invitar a usuarios internos a usar la colaboración B2B, pero no puede quitar las credenciales de B2B una vez agregadas. Para volver a cambiar el usuario a un usuario solo interno, deberá eliminar el objeto de usuario y crear otro.

- **Teams**: Cuando el usuario accede a Teams con sus credenciales externas, su inquilino no estará disponible inicialmente en el selector de inquilinos de Teams. El usuario puede acceder a Teams mediante una dirección URL que contiene el contexto del inquilino, por ejemplo: `https://teams.microsoft.com/?tenantId=<TenantId>`. Después de esto, el inquilino estará disponible en el selector de inquilinos de Teams.

- **Usuarios sincronizados locales**: para las cuentas de usuario sincronizadas entre el entorno local y la nube, el directorio local sigue siendo el origen de autoridad después de haber invitado a los usuarios a usar la colaboración B2B. Los cambios que realice en la cuenta local se sincronizarán con la cuenta en la nube, incluida la deshabilitación o eliminación de la cuenta. Por lo tanto, solo mediante la eliminación de la cuenta local, no puede evitar que el usuario inicie sesión en su cuenta local mientras conserva su cuenta en la nube. En su lugar, puede establecer la contraseña de la cuenta local en un GUID aleatorio u otro valor desconocido.

> [!NOTE]
> Si usa Azure AD Connect Cloud Sync, hay una regla de forma predeterminada que escribe OnPremUserPrincipalName en el usuario. Esta regla debe modificarse para que no escriba esta propiedad en el caso de aquellos usuarios que desee convertir de usuarios internos a externos.

## <a name="how-to-invite-internal-users-to-b2b-collaboration"></a>Invitación de usuarios internos a la colaboración B2B

Puede usar PowerShell o la API de invitación para enviar una invitación a B2B al usuario interno. Asegúrese de que la dirección de correo electrónico que desea usar para la invitación está establecida como la dirección de correo electrónico externa en el objeto de usuario interno.

- Debe usar la dirección de correo electrónico de la propiedad User.Mail en la invitación.
- El dominio de la propiedad Mail del usuario debe coincidir con la cuenta que usan para iniciar sesión. De lo contrario, algunos servicios, como Teams, no podrán autenticar al usuario.

De forma predeterminada, se enviará un correo electrónico al usuario para notificarle que se le ha invitado, pero puede suprimir el mensaje y escribir uno personalizado.

> [!NOTE]
> Para enviar su propio correo electrónico u otra comunicación, puede usar `New-AzureADMSInvitation` con `-SendInvitationMessage:$false` para invitar a los usuarios de forma silenciosa y, a continuación, enviar su propio mensaje de correo electrónico al usuario convertido. Vea [Personalización y API de colaboración B2B de Active Directory Azure](customize-invitation-api.md).

## <a name="use-powershell-to-send-a-b2b-invitation"></a>Uso de PowerShell para enviar una invitación a B2B

Necesitará el módulo Azure AD PowerShell, versión 2.0.2.130 o posterior. Use el siguiente comando para actualizar al módulo de AzureAD PowerShell más reciente e invitar al usuario interno a la colaboración B2B:

```powershell
Uninstall-Module AzureAD
Install-Module AzureAD
$ADGraphUser = Get-AzureADUser -objectID "UPN of Internal User"
$msGraphUser = New-Object Microsoft.Open.MSGraph.Model.User -ArgumentList $ADGraphUser.ObjectId
New-AzureADMSInvitation -InvitedUserEmailAddress <<external email>> -SendInvitationMessage $True -InviteRedirectUrl "http://myapps.microsoft.com" -InvitedUser $msGraphUser
```

## <a name="use-the-invitation-api-to-send-a-b2b-invitation"></a>Uso de la API de invitación para enviar una invitación a B2B

En el ejemplo siguiente se muestra cómo llamar a la API de invitación para invitar a un usuario interno como un usuario de B2B.

```json
POST https://graph.microsoft.com/v1.0/invitations
Authorization: Bearer eyJ0eX...
ContentType: application/json
{
    "invitedUserEmailAddress": "<<external email>>",
    "sendInvitationMessage": true,
    "invitedUserMessageInfo": {
        "messageLanguage": "en-US",
        "ccRecipients": [
            {
                "emailAddress": {
                    "name": null,
                    "address": "<<optional additional notification email>>"
                }
            }
        ],
        "customizedMessageBody": "<<custom message>>"
    },
    "inviteRedirectUrl": "https://myapps.microsoft.com?tenantId=",
    "invitedUser": {
        "id": "<<ID for the user you want to convert>>"
    }
}
```

La respuesta a la API es la misma respuesta que se obtiene al invitar a un nuevo usuario invitado al directorio.
## <a name="next-steps"></a>Pasos siguientes

- [Canje de invitación de colaboración B2B](redemption-experience.md)
