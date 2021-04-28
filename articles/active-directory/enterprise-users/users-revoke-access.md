---
title: Revocación del acceso de usuario en un emergencia de Azure Active Directory | Microsoft Docs
description: Revocación de todo el acceso de un usuario en Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
author: curtand
ms.author: curtand
manager: daveba
ms.reviewer: krbain
ms.date: 03/29/2021
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 75547b50289f9fa73b6fc2a28fe7ec53b5c5367a
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137984"
---
# <a name="revoke-user-access-in-azure-active-directory"></a>Revocación del acceso de usuario en Azure Active Directory

Entre los escenarios que podrían requerir que un administrador revoque todo el acceso de un usuario, se incluyen cuentas en peligro, rescisión de empleados y otras amenazas internas. En función de la complejidad del entorno, los administradores pueden realizar varios pasos para garantizar la revocación del acceso. En algunos escenarios, puede haber un período entre el inicio de la revocación del acceso y el momento en que se revoca realmente.

Para mitigar los riesgos, debe comprender cómo funcionan los tokens. Hay muchos tipos de tokens, que se encuentran en uno de los patrones que se mencionan en las secciones siguientes.

## <a name="access-tokens-and-refresh-tokens"></a>Tokens de acceso y tokens de actualización

Los tokens de acceso y los tokens de actualización se usan con frecuencia con aplicaciones cliente pesado y también se usan en aplicaciones basadas en explorador, como las aplicaciones de una sola página.

- Cuando los usuarios se autentican en Azure AD, se evalúan las directivas de autorización para determinar si se puede conceder acceso al usuario para un recurso específico.  

- Si se autoriza, Azure AD emite un token de acceso y un token de actualización para el recurso.  

- Los tokens de acceso que emite Azure AD duran 1 hora de forma predeterminada. Si el protocolo de autenticación lo permite, la aplicación puede pasar el token de actualización a Azure AD cuando expire el token de acceso con la finalidad de volver a autenticar al usuario de forma silenciosa.

A continuación, Azure AD vuelve a evaluar sus directivas de autorización. Si el usuario sigue autorizado, Azure AD emite un nuevo token de acceso y un token de actualización.

Los tokens de acceso pueden ser un problema de seguridad si el acceso se debe revocar en un período de tiempo menor que la duración del token, que suele ser de una hora aproximadamente. Por esta razón, Microsoft trabaja activamente para incorporar la [evaluación del acceso continua](../conditional-access/concept-continuous-access-evaluation.md) a las aplicaciones de Office 365, lo que ayuda a garantizar la invalidación de los tokens de acceso casi en tiempo real.  

## <a name="session-tokens-cookies"></a>Tokens de sesión (cookies)

La mayoría de las aplicaciones basadas en explorador usan tokens de sesión en lugar de tokens de acceso y de actualización.  

- Cuando un usuario abre un explorador y se autentica en una aplicación a través de Azure AD, recibe dos tokens de sesión. Uno de Azure AD y otro de la aplicación.  

- Una vez que una aplicación emite su propio token de sesión, el acceso a la aplicación se rige por la sesión de la aplicación. En este momento, el usuario solo se ve afectado por las directivas de autorización de las que la aplicación es consciente.

- Las directivas de autorización de Azure AD se vuelven a evaluar con la frecuencia con la que la aplicación vuelve a enviar al usuario a Azure AD. Normalmente, la reevaluación se produce de forma silenciosa, aunque la frecuencia depende de cómo esté configurada la aplicación. Es posible que la aplicación nunca vuelva a enviar al usuario a Azure AD siempre y cuando el token de sesión sea válido.

- Para poder revocar un token de sesión, la aplicación debe revocar el acceso en función de sus propias directivas de autorización. Azure AD no puede revocar directamente un token de sesión emitido por una aplicación.  

## <a name="revoke-access-for-a-user-in-the-hybrid-environment"></a>Revocar el acceso de un usuario en el entorno híbrido

Para un entorno híbrido con Active Directory local sincronizado con Azure Active Directory, Microsoft recomienda a los administradores de TI que realicen las siguientes acciones.  

### <a name="on-premises-active-directory-environment"></a>Entorno de Active Directory local

Como administrador de Active Directory, conéctese a la red local, abra PowerShell y realice las siguientes acciones:

1. Deshabilite al usuario en Active Directory. Consulte [Disable-ADAccount](/powershell/module/activedirectory/disable-adaccount?view=win10-ps).

    ```PowerShell
    Disable-ADAccount -Identity johndoe  
    ```

2. Restablezca la contraseña del usuario dos veces en Active Directory. Consulte [Set-ADAccountPassword](/powershell/module/activedirectory/set-adaccountpassword?view=win10-ps).

    > [!NOTE]
    > La razón para cambiar la contraseña de un usuario dos veces es mitigar el riesgo de ataques Pass-the-hash, especialmente si hay retrasos en la replicación de contraseñas local. Si puede suponer con seguridad que esta cuenta no está en peligro, puede restablecer la contraseña solo una vez.

    > [!IMPORTANT]
    > No use las contraseñas de ejemplo en los siguientes cmdlets. Asegúrese de cambiar las contraseñas por una cadena aleatoria.

    ```PowerShell
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd1" -Force)
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd2" -Force)
    ```

### <a name="azure-active-directory-environment"></a>Entorno de Azure Active Directory

Como administrador de Azure Active Directory, abra PowerShell, ejecute ``Connect-AzureAD`` y realice las siguientes acciones:

1. Deshabilite al usuario en Azure AD. Consulte [Set-AzureADUser](/powershell/module/azuread/Set-AzureADUser?view=azureadps-2.0).

    ```PowerShell
    Set-AzureADUser -ObjectId johndoe@contoso.com -AccountEnabled $false
    ```

2. Revoque los tokens de actualización de Azure AD del usuario. Consulte [Revoke-AzureADUserAllRefreshToken](/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0).

    ```PowerShell
    Revoke-AzureADUserAllRefreshToken -ObjectId johndoe@contoso.com
    ```

3. Deshabilite los dispositivos del usuario. Consulte [Get-AzureADUserRegisteredDevice](/powershell/module/azuread/get-azureaduserregistereddevice?view=azureadps-2.0).

    ```PowerShell
    Get-AzureADUserRegisteredDevice -ObjectId johndoe@contoso.com | Set-AzureADDevice -AccountEnabled $false
    ```
## <a name="when-access-is-revoked"></a>Cuando se revoca el acceso

Una vez que los administradores han realizado los pasos anteriores, el usuario no puede obtener nuevos tokens para ninguna aplicación asociada a Azure Active Directory. El tiempo transcurrido entre la revocación y el momento en que el usuario pierde el acceso depende del modo en que la aplicación concede el acceso:

- Para las **aplicaciones que usan tokens de acceso**, el usuario pierde el acceso cuando expira el token de acceso.

- En el caso de las **aplicaciones que usan tokens de sesión**, las sesiones existentes finalizan cuando expira el token. Si el estado deshabilitado del usuario se sincroniza con la aplicación, la aplicación puede revocar automáticamente las sesiones existentes del usuario si está configurada para hacerlo.  El tiempo que se tarda depende de la frecuencia de sincronización entre la aplicación y Azure AD.

## <a name="best-practices"></a>Procedimientos recomendados

- Implementación de una solución de aprovisionamiento y desaprovisionamiento automatizada. El desaprovisionamiento de usuarios de las aplicaciones es una manera eficaz de revocar el acceso, especialmente en el caso de las aplicaciones que usan tokens de sesión. Desarrolle un proceso para desaprovisionar a los usuarios de las aplicaciones que no admiten el aprovisionamiento y desaprovisionamiento automáticos. Asegúrese de que las aplicaciones revocan sus propios tokens de sesión y deje de aceptar tokens de acceso de Azure AD aunque sigan siendo válidos.

  - Use el [aprovisionamiento de aplicaciones SaaS de Azure AD](../app-provisioning/user-provisioning.md). El aprovisionamiento de aplicaciones SaaS de Azure AD normalmente se ejecuta automáticamente cada 20-40 minutos. [Configuración del aprovisionamiento de Azure AD](../saas-apps/tutorial-list.md) para desaprovisionar o desactivar los usuarios deshabilitados en las aplicaciones.
  
  - En el caso de las aplicaciones que no usan el aprovisionamiento de aplicaciones SaaS de Azure AD, use [Identity Manager (MIM)](/microsoft-identity-manager/mim-how-provision-users-adds) o una solución de terceros para automatizar el desaprovisionamiento de usuarios.  
  - Identifique y desarrolle un proceso para las aplicaciones que requieren realizar el desaprovisionamiento manual. Asegúrese de que los administradores puedan ejecutar rápidamente las tareas manuales necesarias para desaprovisionar al usuario de estas aplicaciones cuando sea necesario.
  
- [Administración de dispositivos y aplicaciones con Microsoft Intune](/mem/intune/remote-actions/device-management). Los dispositivos que administra Intune [se pueden restablecer a la configuración de fábrica](/mem/intune/remote-actions/devices-wipe). Si el dispositivo no está administrado, puede [borrar los datos corporativos de las aplicaciones administradas](/mem/intune/apps/apps-selective-wipe). Estos procesos son eficaces para quitar datos potencialmente confidenciales de los dispositivos de los usuarios finales. Sin embargo, para cualquier proceso que se desencadene, el dispositivo debe estar conectado a Internet. Si el dispositivo está sin conexión, este seguirá teniendo acceso a los datos almacenados localmente.

> [!NOTE]
> Los datos del dispositivo no se pueden recuperar después de un borrado.

- Use [Microsoft Cloud App Security (MCAS) para bloquear la descarga de datos](/cloud-app-security/use-case-proxy-block-session-aad) cuando sea necesario. Si solo se puede obtener acceso a los datos en línea, las organizaciones pueden supervisar las sesiones y lograr el cumplimiento de las directivas en tiempo real.

- Habilite la [Evaluación de acceso continuo (CAE) en Azure AD](../conditional-access/concept-continuous-access-evaluation.md). CAE permite a los administradores revocar los tokens de sesión y los tokens de acceso para las aplicaciones que son compatibles con CAE.  

## <a name="next-steps"></a>Pasos siguientes

- [Procedimientos de acceso seguro para administradores de Azure AD](../roles/security-planning.md)
- [Agregar o actualizar la información del perfil de usuario](../fundamentals/active-directory-users-profile-azure-portal.md)