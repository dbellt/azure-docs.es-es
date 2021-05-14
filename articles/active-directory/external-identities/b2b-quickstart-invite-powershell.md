---
title: 'Inicio rápido: Incorporación de un usuario invitado con PowerShell en Azure AD'
description: En este tutorial, obtendrá información sobre cómo usar PowerShell para enviar una invitación a un usuario externo de colaboración de Azure AD B2B.
services: active-directory
author: msmimart
ms.author: mimart
manager: celestedg
ms.reviewer: mal
ms.date: 08/28/2018
ms.topic: quickstart
ms.service: active-directory
ms.subservice: B2B
ms.custom:
- it-pro
- seo-update-azuread-jan
- mode-api
ms.collection: M365-identity-device-management
ms.openlocfilehash: f3541698e4a02f82f54d84eabf1bf7bb285cda10
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987585"
---
# <a name="quickstart-add-a-guest-user-with-powershell"></a>Guía de inicio rápido: Incorporación de un usuario invitado con PowerShell

Hay muchas maneras de invitar a los asociados externos a sus aplicaciones y servicios con la colaboración de Azure Active Directory B2B. En el tutorial anterior, vimos cómo agregar usuarios invitados directamente en el portal de administración de Azure Active Directory. También puede usar PowerShell para agregar usuarios invitados, uno a uno o de forma masiva. En este tutorial, usará el comando New-AzureADMSInvitation para agregar un usuario invitado a su inquilino de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

## <a name="prerequisites"></a>Requisitos previos

### <a name="powershell-module"></a>Módulo de PowerShell
Instale el [módulo de Azure AD V2 PowerShell para Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) o la [versión preliminar del módulo de Azure AD V2 PowerShell para Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

### <a name="get-a-test-email-account"></a>Obtención de una cuenta de correo electrónico de prueba

Necesita una cuenta de correo electrónico de prueba a la que poder enviar la invitación. La cuenta debe estar fuera de su organización. Puede usar cualquier tipo de cuenta, incluida una cuenta social como una dirección de gmail.com o outlook.com.

## <a name="sign-in-to-your-tenant"></a>Inicio de sesión en su inquilino

Ejecute el siguiente comando para conectarse al dominio del inquilino:

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
Por ejemplo, `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

Cuando se le solicite, escriba las credenciales.

## <a name="send-an-invitation"></a>Envío de una invitación

1. Para enviar una invitación a su cuenta de correo electrónico de prueba, ejecute el siguiente comando de PowerShell (reemplace **"Sanda"** y **sanda\@fabrikam.com** por la dirección de correo electrónico y el nombre de la cuenta de correo electrónico de prueba): 

   ```powershell
   New-AzureADMSInvitation -InvitedUserDisplayName "Sanda" -InvitedUserEmailAddress sanda@fabrikam.com -InviteRedirectURL https://myapps.microsoft.com -SendInvitationMessage $true
   ```
2. El comando envía una invitación a la dirección de correo electrónico especificada. Consulte la salida, que debería ser similar a la siguiente:

   ![Salida de PowerShell que muestra la aceptación pendiente del usuario](media/quickstart-invite-powershell/powershell-azureadmsinvitation-result.png)

## <a name="verify-the-user-exists-in-the-directory"></a>Verificación de la existencia del usuario en el directorio

1. Para verificar que se agregó el usuario invitado a Azure AD, ejecute el siguiente comando:
 
   ```powershell
   Get-AzureADUser -Filter "UserType eq 'Guest'"
   ```
3. Consulte la salida para asegurarse de que el usuario invitado aparece en la lista, con un nombre principal de usuario (UPN) con el formato *emailaddress*#EXT#\@*domain*. Por ejemplo, *sanda_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, donde contoso.onmicrosoft.com es la organización desde la que se enviaron las invitaciones.

   ![Salida de PowerShell que muestra que se agregó el usuario invitado](media/quickstart-invite-powershell/powershell-guest-user-added.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesaria, puede eliminar la cuenta de usuario de prueba del directorio. Ejecute el siguiente comando para eliminar una cuenta de usuario:

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
Por ejemplo: `Remove-AzureADUser -ObjectId "sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, invitó y agregó a un único usuario invitado a su directorio con PowerShell. A continuación, obtenga información sobre cómo invitar a usuarios en masa mediante PowerShell.

> [!div class="nextstepaction"]
> [Tutorial: Invitación en masa a usuarios de colaboración de Azure AD B2B](tutorial-bulk-invite.md)
