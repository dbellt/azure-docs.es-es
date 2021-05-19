---
title: 'Salir de una organización como usuario invitado: Azure Active Directory'
description: Muestra cómo un usuario invitado de Azure AD B2B puede salir de una organización a través del Panel de acceso.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/05/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc07e038ef382a44c4672b2d63c49d8f5a961cda
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486342"
---
# <a name="leave-an-organization-as-a-guest-user"></a>Salir de una organización como usuario invitado

Es posible que un usuario invitado de Azure Active Directory (Azure AD) B2B decida salir de una organización en cualquier momento si ya no necesita usar las aplicaciones de dicha organización ni mantener ninguna asociación. Un usuario puede salir de una organización por su cuenta, sin ponerse en contacto con un administrador.

> [!NOTE]
> Un usuario invitado no puede abandonar una organización si su cuenta está deshabilitada en el inquilino principal o el inquilino del recurso. Si su cuenta está deshabilitada, el usuario invitado debe ponerse en contacto con el administrador de inquilinos, que puede eliminar la cuenta de invitado o habilitar la cuenta de invitado para que el usuario pueda abandonar la organización.

## <a name="leave-an-organization"></a>Salir de una organización

Para salir de una organización, siga estos pasos.

1. Siga uno de estos pasos para ir a la página Perfil del Panel de acceso:
   
   - En [Azure Portal](https://portal.azure.com), haga clic en su nombre en la esquina superior derecha y seleccione **Ver cuenta**.
   - Abra el [Panel de acceso](https://myapps.microsoft.com), haga clic en su nombre en la parte superior derecha y, al lado de **Organizaciones**, seleccione **Ver cuenta**.
 
 
2. Seleccione **Administrar organizaciones**.
  ![Captura de pantalla que muestra la configuración de usuario en el panel de acceso](media/leave-the-organization/manage-organizations.png) 
  
3. En **Organizaciones**, busque la organización que quiere abandonar y seleccione **Salir de la organización**.

   ![Captura de pantalla que muestra la opción Salir de la organización en la interfaz de usuario](media/leave-the-organization/leave-org.png)
4. Cuando se le pida que confirme, seleccione **Salir**.
> [!NOTE]
   > No puede abandonar la organización principal.

## <a name="account-removal"></a>Eliminar cuenta

Cuando un usuario sale de una organización, la cuenta de usuario se "elimina temporalmente" en el directorio. De forma predeterminada, el objeto de usuario se mueve al área **Usuarios eliminados** en Azure AD, pero no se elimina permanentemente durante 30 días. Esta eliminación temporal permite que el administrador restaure la cuenta de usuario (incluidos los grupos y permisos) si el usuario lo solicita en el plazo de 30 días.

Si lo desea, un administrador de inquilinos puede eliminar permanentemente la cuenta en cualquier momento durante el período de 30 días. Para ello, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.
2. En **Administrar**, seleccione **Usuarios**.
3. Seleccione **Usuarios eliminados**.
4. Active la casilla de verificación situada junto a un usuario eliminado y, luego, seleccione **Eliminar de forma permanente**.

Si elimina permanentemente un usuario, esta acción es irrevocable.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Pasos siguientes

- Para información general sobre B2B de Azure AD, consulte [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)



