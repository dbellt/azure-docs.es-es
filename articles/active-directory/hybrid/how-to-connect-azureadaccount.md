---
title: Cambio de la contraseña de la cuenta del conector de Azure AD | Microsoft Docs
description: En este tema se describe cómo restaurar la cuenta del conector de Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 6077043a-27f1-4304-a44b-81dc46620f24
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 5a16e571478428ef1eb068236369b6e9af9f19e1
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111853701"
---
# <a name="change-the-azure-ad-connector-account-password"></a>Cambio de la contraseña de la cuenta del conector de Azure AD
Se supone que la cuenta del conector de Azure AD está libre de servicio. Si necesita restablecer sus credenciales, este tema le interesa. Por ejemplo, si un administrador global ha restablecido la contraseña por error en la cuenta mediante PowerShell.

## <a name="reset-the-credentials"></a>Restablecimiento de las credenciales
Si la cuenta del conector de Azure AD no puede ponerse en contacto con Azure AD debido a problemas de autenticación, se puede restablecer la contraseña.

1. Inicie sesión en el servidor de sincronización de Azure AD Connect y abra PowerShell.
2. Para proporcionar las credenciales de administrador global de Azure AD, ejecute `$credential = Get-Credential`.
3. Ejecute el cmdlet `Add-ADSyncAADServiceAccount -AADCredential $credential`.

   Si el cmdlet se ejecuta correctamente, aparece el símbolo del sistema de PowerShell. 
   
El cmdlet restablecerá la contraseña de la cuenta de servicio y la actualizará en Azure AD y en el motor de sincronización.

## <a name="known-issues-these-steps-can-solve"></a>Problemas conocidos que pueden solucionarse siguiendo estos pasos
Esta sección es una lista de los errores notificados por clientes que se solucionaron mediante un restablecimiento de las credenciales de la cuenta del conector de Azure AD.

---
Evento 6900 El servidor encontró un error inesperado al procesar una notificación de cambio de contraseña: AADSTS70002: error al validar las credenciales. AADSTS50054: Se utilizó una contraseña antigua para realizar la autenticación.

---
Evento 659 Error al recuperar la configuración de sincronización de directivas de contraseña. Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException: AADSTS70002: error al validar las credenciales. AADSTS50054: Se utilizó una contraseña antigua para realizar la autenticación.

## <a name="next-steps"></a>Pasos siguientes
**Temas de introducción**

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)
