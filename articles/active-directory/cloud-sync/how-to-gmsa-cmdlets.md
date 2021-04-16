---
title: Cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube de Azure AD
description: Aprenda a usar los cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube de Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 46b5c94c84bdbd0e5bb0d7b08ba9296ebfcd1838
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107306065"
---
# <a name="azure-ad-connect-cloud-provisioning-agent-gmsa-powershell-cmdlets"></a>Cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube de Azure AD

El objetivo de este documento es describir los cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube de Azure AD. Estos cmdlets le permiten tener un mayor control sobre los permisos que se aplican a la cuenta de servicio (GMSA). De forma predeterminada, la sincronización en la nube de Azure AD Connect aplica todos los permisos similares a Azure AD Connect a la cuenta de GMSA predeterminada o a una cuenta de GMSA personalizada. 

En este documento se tratan los siguientes cmdlets:  

`Set-AADCloudSyncRestrictedPermissions`

`Set-AADCloudSyncPermissions` 

## <a name="how-to-use-the-cmdlets"></a>Uso de los cmdlets  

Los siguientes requisitos previos son indispensables para usar estos cmdlets:

1. Instale el agente de aprovisionamiento. 
2. Importe el módulo PS del agente de aprovisionamiento en una sesión de PowerShell. 

 ```PowerShell
 Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Microsoft.CloudSync.Powershell.dll"  
 ```
3. Quite los permisos existentes.  Para quitar todos los permisos existentes de la cuenta de servicio, excepto SELF, use `Set-AADCloudSyncRestrictedPermission`.  

    Este cmdlet requiere un parámetro denominado `Credential` que se puede pasar o que se le solicitará si se realiza una llamada sin él.

    Para crear una variable, use:  

   `$credential = Get-Credential` 

   Se le pedirá al usuario que escriba el nombre de usuario y la contraseña. Las credenciales deben ser como mínimo las de un administrador de dominio (del dominio en el que esté instalado el agente), aunque también sirven las de un administrador de empresa. 

4.  Después, puede llamar al cmdlet para quitar los permisos adicionales: 
   ```PowerShell
   Set-AADCloudSyncRestrictedPermissions -Credential $credential 
   ```
5. O bien, puede llamar simplemente a 

   `Set-AADCloudSyncRestrictedPermissions`, que le solicitará las credenciales. 

 6.  Agregue un tipo de permiso específico.  Los permisos agregados son los mismos que los de Azure AD Connect.  Consulte a continuación [Uso de Set-AADCloudSyncPermissions](#using-set-aadcloudsyncpermissions) para ver ejemplos de cómo establecer los permisos.

## <a name="using-set-aadcloudsyncpermissions"></a>Uso de Set-AADCloudSyncPermissions 
`Set-AADCloudSyncPermissions` admite los siguientes tipos de permisos que son idénticos a los que usa Azure AD Connect. Se admiten los siguientes tipos de permisos: 

|Tipo de permiso|Descripción|
|-----|-----|
|BasicRead| Consulte los permisos [BasicRead](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#configure-basic-read-only-permissions) de Azure AD Connect.|
|PasswordHashSync|Consulte los permisos [PasswordHashSync](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-password-hash-synchronization) de Azure AD Connect.|
|PasswordWriteBack|Consulte los permisos [PasswordWriteBack](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-password-writeback) de Azure AD Connect.|
|HybridExchangePermissions|Consulte los permisos [HybridExchangePermissions](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-exchange-hybrid-deployment) de Azure AD Connect.| 
|ExchangeMailPublicFolderPermissions| Consulte los permisos [ExchangeMailPublicFolderPermissions](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-exchange-mail-public-folders-preview) de Azure AD Connect.| 
|CloudHR| Aplica "Control total" a "Descendant User objects" (Objetos de usuario descendientes) y "Creación o eliminación de objetos de usuario" a "Este objeto y todos los descendientes"| 
|All|Agrega todos los permisos anteriores.| 



Puede usar AADCloudSyncPermissions de dos formas:
- [Conceder un permiso determinado a todos los dominios configurados](#grant-a-certain-permission-to-all-configured-domains) 
- [Conceder un permiso determinado a un dominio específico](#grant-a-certain-permission-to-a-specific-domain) 
## <a name="grant-a-certain-permission-to-all-configured-domains"></a>Conceder un permiso determinado a todos los dominios configurados 
La concesión de determinados permisos a todos los dominios configurados requiere el uso de una cuenta de administrador de empresa.


 ```PowerShell
Set-AADCloudSyncPermissions -PermissionType “Any mentioned above” -EACredential $credential (prepopulated same as above [$credential = Get-Credential]) 
```

## <a name="grant-a-certain-permission-to-a-specific-domain"></a>Conceder un permiso determinado a un dominio específico 
La concesión de determinados permisos a un dominio específico requerirá el uso de, como mínimo, una cuenta de administrador de dominio del dominio que intenta agregar.


 ```PowerShell
Set-AADCloidSyncPermissions -PermissionType “Any mentioned above” -TargetDomain “FQDN of domain” (has to be already configured through wizard) -TargetDomaincredential $credential(same as above) 
```
 

Nota para el caso 1: Las credenciales deben ser las de un administrador de empresa como mínimo. 

En 2. Las credenciales pueden ser las de un administrador de dominio o las de un administrador de empresa. 

  

