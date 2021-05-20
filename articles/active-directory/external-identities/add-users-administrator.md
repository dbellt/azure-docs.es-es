---
title: 'Incorporación de usuarios de colaboración B2B en Azure Portal: Azure AD'
description: Muestra cómo un administrador puede agregar usuarios invitados a su directorio desde una organización asociada con la colaboración B2B de Azure Active Directory (Azure AD).
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 04/23/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5e538be52f21d169aefd350aa2781de0065f04f
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632953"
---
# <a name="add-azure-active-directory-b2b-collaboration-users-in-the-azure-portal"></a>Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal

Como usuario al que se le asignan cualquiera de los roles limitados de directorio de administradores, puede usar Azure Portal para invitar a los usuarios de colaboración B2B. Puede invitar a usuarios invitados al directorio, a un grupo o a una aplicación. Después de invitar a un usuario mediante cualquiera de estos métodos, la cuenta del usuario invitado se agrega a Azure Active Directory (Azure AD), con *invitado* como tipo de usuario. El usuario invitado deberá canjear después su invitación para acceder a los recursos. Una invitación de un usuario no expira.

Después de agregar un usuario invitado al directorio, puede enviarle un vínculo directo a una aplicación compartida o bien, el propio usuario invitado puede seleccionar la dirección URL de canje del correo electrónico de invitación. Para más información sobre el proceso de canje, consulte [Canje de invitación de colaboración B2B](redemption-experience.md).

> [!IMPORTANT]
> Debe seguir los pasos descritos en [Procedimiento: Agregar información de privacidad con Azure Active Directory](../fundamentals/active-directory-properties-area.md) para agregar la dirección URL de la declaración de privacidad de su organización. Como parte del proceso de canje de invitación por primera vez, el usuario invitado debe indicar su consentimiento con los términos de privacidad para poder continuar. 

## <a name="before-you-begin"></a>Antes de empezar

Asegúrese de que la configuración de colaboración externa de la organización se configure de forma que le permita invitar a otros usuarios. De manera predeterminada, todos los usuarios y administradores pueden invitar a otros usuarios. Sin embargo, las directivas de colaboración externa de la organización pueden estar configuradas para impedir que determinados tipos de usuarios o administradores inviten a otros usuarios. Para obtener información sobre cómo ver y establecer estas directivas, consulte [Enable B2B external collaboration and manage who can invite guests](delegate-invitations.md) (Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios).

## <a name="add-guest-users-to-the-directory"></a>Adición de usuarios invitados al directorio

Para agregar usuarios de colaboración B2B al directorio, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como usuario que tiene asignado un rol de directorio administrador limitado o el rol de invitador de usuarios invitados.
2. Busque y seleccione **Azure Active Directory** en cualquier página.
3. En **Administrar**, seleccione **Usuarios**.
4. Seleccione **Nuevo usuario invitado**.
5. En la página **Nuevo usuario**, seleccione **Invitar usuario** y, después, agregue la información del usuario invitado.

   ![Imagen de tipo de usuario invitado](media/add-users-administrator/invite-user.png)

      - **Nombre**. Nombre y apellidos del nuevo usuario.
   - **Dirección de correo (obligatorio)** . La dirección de correo del usuario invitado.
   - **Mensaje personal (opcional)** Incluye un mensaje de bienvenida personal al usuario invitado.
   - **Grupos**: Puede agregar al usuario invitado a uno o varios de los grupos existentes, o puede hacerlo después.
   -  **Roles**: Si necesita permisos administrativos de Azure AD para el usuario, puede agregarlos a un rol de Azure AD seleccionando **Usuario** junto a **Roles**.  

   > [!NOTE]
   > No se admiten direcciones de correo electrónico de grupos; escriba la dirección de un usuario individual. Además, algunos proveedores de correo electrónico permiten a los usuarios agregar un signo más (+) y texto adicional a sus direcciones de correo electrónico, ya que ello sirve de ayuda a algunas funciones como el filtrado de la bandeja de entrada. Sin embargo, Azure AD no admite actualmente estos símbolos más (+) en las direcciones de correo electrónico. Para evitar problemas de entrega, omita el signo más y los caracteres siguientes hasta el símbolo @.
6. Seleccione **Invitar** para enviar automáticamente la invitación al usuario invitado. 
 
Después de enviar la invitación, la cuenta de usuario se agrega automáticamente al directorio como invitado.

 ![Imagen de usuario invitado](media/add-users-administrator//guest-user-type.png)

## <a name="add-guest-users-to-a-group"></a>Adición de usuarios invitados a un grupo
Si necesita agregar manualmente usuarios de colaboración B2B a un grupo, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. Busque y seleccione **Azure Active Directory** en cualquier página.
3. En **Administrar**, seleccione **Grupos**.
4. Seleccione un grupo (o seleccione **Nuevo grupo** para crear uno). Es una buena idea incluir en la descripción del grupo que el grupo contiene los usuarios invitados B2B.
5. Seleccione el vínculo debajo de **Miembros**.
6. Realice una de las siguientes acciones:

     - Si el usuario invitado ya existe en el directorio, busque el usuario B2B. Seleccione el usuario y, luego, seleccione **Seleccionar** para agregar el usuario al grupo.
     - Si el usuario invitado no existe ya en el directorio, invítelo al grupo; para ello, escriba su dirección de correo electrónico en el cuadro de búsqueda, escriba un mensaje personal opcional y, a continuación, elija **Invitar**. La invitación se dirige automáticamente al usuario invitado.
   
![Adición del botón Invitar para agregar miembros invitados](./media/add-users-administrator/group-invite.png)
 
También puede utilizar grupos dinámicos con colaboración B2B de Azure AD. Para más información, consulte [Grupos dinámicos y colaboración B2B de Azure Active Directory](use-dynamic-groups.md).

## <a name="add-guest-users-to-an-application"></a>Adición de usuarios invitados a una aplicación

Para agregar usuarios de colaboración B2B a una aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. Busque y seleccione **Azure Active Directory** en cualquier página.
3. En **Administrar**, seleccione **Aplicaciones empresariales**.
4. En la página **Todas las aplicaciones**, seleccione la aplicación a la que quiere agregar usuarios invitados.
5. En **Administrar**, seleccione **Usuarios y grupos**.
6. Seleccione **Agregar usuario**.
7. En **Agregar asignación**, seleccione **Usuarios y grupos**.
8. Realice una de las siguientes acciones:
   - Si el usuario invitado ya existe en el directorio, busque el usuario B2B. Seleccione el usuario, elija **Seleccionar** y luego seleccione **Asignar** para agregar el usuario a la aplicación.
   - Si el usuario invitado no existe en el directorio, en **Seleccione un miembro o invite a un usuario externo**, escriba la dirección de correo electrónico del usuario. En el cuadro de mensaje, escriba un mensaje personal opcional. Debajo del cuadro de mensaje, seleccione **Invitar**.
               ![Captura de pantalla que resalta dónde agregar la dirección de correo electrónico del usuario, el mensaje personalizado y también resalta el botón Invitar.](./media/add-users-administrator/app-invite-users.png)

10.  El usuario invitado aparece en la lista **Usuarios y grupos** de la aplicación con el rol asignado de **Acceso predeterminado**. Si la aplicación proporciona roles diferentes, y usted quiere cambiar el rol del usuario, haga lo siguiente:
   - Active la casilla situada junto al usuario invitado y, a continuación, seleccione el botón **Editar**. 
   - En la página **Editar asignación**, elija el vínculo en **Seleccionar un rol** y seleccione el rol que quiere asignar al usuario.
   - Elija **Seleccionar**.
   - Seleccione **Asignar**.
 
## <a name="resend-invitations-to-guest-users"></a>Reenvío de invitaciones a usuarios invitados

Si un usuario invitado todavía no ha canjeado su invitación, puede volver a enviarle el correo electrónico de invitación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. Busque y seleccione **Azure Active Directory** en cualquier página.
3. En **Administrar**, seleccione **Usuarios**.
4. Seleccione la cuenta del usuario.
5. En la sección **Identidad**, en **Invitación aceptada**, seleccione el vínculo **(administrar)** . 
6. Si el usuario aún no ha aceptado la invitación, seleccione la opción **Sí** para volver a enviarla.

    ![Reenvío de una invitación](./media/add-users-administrator/resend-invitation.png)

> [!NOTE]
> Se generará una dirección URL de invitación.

## <a name="next-steps"></a>Pasos siguientes

- Para aprender cómo los administradores que no son de Azure AD pueden agregar usuarios invitados B2B, consulte [¿Cómo agregan los trabajadores de la información usuarios de colaboración B2B a Azure Active Directory?](add-users-information-worker.md)
- Para más información sobre el correo electrónico de invitación, consulte [Los elementos del correo electrónico de invitación de colaboración B2B](invitation-email-elements.md).