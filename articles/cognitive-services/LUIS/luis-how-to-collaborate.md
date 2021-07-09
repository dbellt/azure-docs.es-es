---
title: 'Colaboración con otros usuarios: LUIS'
titleSuffix: Azure Cognitive Services
description: Un propietario de la aplicación puede agregar colaboradores al recurso de creación. Estos colaboradores pueden modificar el modelo, entrenar y publicar la aplicación.
services: cognitive-services
author: aahill
ms.author: aahi
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2021
ms.openlocfilehash: bb56ce1b5be30cd36b8368e8ad7b886e81296f43
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110083663"
---
# <a name="add-contributors-to-your-app"></a>Adición de colaboradores a su aplicación

Un propietario de la aplicación puede agregar colaboradores a las aplicaciones. Estos colaboradores pueden modificar el modelo, entrenar y publicar la aplicación. Una vez haya [migrado](luis-migration-authoring.md) la cuenta, los _colaboradores_ se administran en Azure Portal para poder usar el recurso de creación, mediante la página del **control de acceso (IAM)** . Agregue un usuario mediante la dirección de correo electrónico del colaborador y del rol _Colaborador_.

## <a name="add-contributor-to-azure-authoring-resource"></a>Adición de un colaborador al recurso de creación de Azure

Ha migrado si su experiencia de creación de LUIS está asociada a un recurso de creación en la página **Administrar -> Recursos de Azure** del portal de LUIS.

1. En Azure Portal, busque el recurso de creación de Language Understanding (LUIS). Dispone del tipo `LUIS.Authoring`.
1. En la página **Control de acceso (IAM)**, seleccione **+Agregar** y, a continuación, **Agregar asignación de roles**.

    ![En Azure Portal, agregue la asignación de roles en el recurso de creación.](./media/luis-how-to-collaborate/authoring-resource-access-control-add-role.png)

1. En la ventana **Agregar asignación de roles**, seleccione el **rol** de colaborador. En la opción **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD**. En la opción **Seleccionar**, escriba la dirección de correo electrónico del usuario. Si se conoce al usuario por más de una dirección de correo electrónico para el mismo dominio, asegúrese de que se escribe la cuenta de correo electrónico _principal_.

    ![Adición del correo electrónico del usuario al rol de colaborador para Azure AD](./media/luis-how-to-collaborate/add-role-assignment-for-contributor.png)

    Cuando se encuentre el correo electrónico del usuario, seleccione la cuenta y **Guardar**.

    Si tiene problemas con esta asignación de roles, revise [Asignaciones de roles de Azure](../../role-based-access-control/role-assignments-portal.md) y [Solución de problemas del control de acceso de Azure](../../role-based-access-control/troubleshooting.md#problems-with-azure-role-assignments).

## <a name="view-the-app-as-a-contributor"></a>Visualización de la aplicación como colaborador

Una vez que se haya agregado como colaborador, [inicie sesión en el portal LUIS](sign-in-luis-portal.md).

[!INCLUDE [switch azure directories](includes/switch-azure-directories.md)]

### <a name="users-with-multiple-emails"></a>Usuarios con varios correos electrónicos

Si agrega colaboradores a una aplicación de LUIS, debe especificar la dirección de correo electrónico exacta. Si bien Azure Active Directory (Azure AD) permite que un solo usuario tenga más de una cuenta de correo electrónico intercambiable, LUIS requiere que el usuario inicie sesión con la dirección de correo especificada al agregar el colaborador.

<a name="owner-and-collaborators"></a>

### <a name="azure-active-directory-resources"></a>Recursos de Azure Active Directory

Si usa [Azure Active Directory](../../active-directory/index.yml) (Azure AD) en su organización, Language Understanding (LUIS) necesita permiso para obtener acceso a la información sobre sus usuarios cuando quieran usar LUIS. Los recursos que requiere LUIS son mínimos.

Podrá ver la descripción detallada cuando intente registrarse con una cuenta que tenga consentimiento del administrador o no requiera consentimiento del administrador: Ejemplos de consentimiento del administrador serían los siguientes:

* Permite iniciar sesión en la aplicación con su cuenta profesional y dejar que la aplicación lea su perfil. También permite que la aplicación lea información básica de la empresa. Esta opción concede permiso de LUIS para leer datos de perfil básicos, como el id. de usuario, el correo electrónico o el nombre.
* Permite que la aplicación vea y actualice los datos, incluso cuando no se esté usando la aplicación. El permiso es necesario para actualizar el token de acceso del usuario.


### <a name="azure-active-directory-tenant-user"></a>Usuario inquilino de Azure Active Directory

LUIS usa el flujo de consentimiento de Azure Active Directory (Azure AD) estándar.

El administrador de inquilinos debe trabajar directamente con el usuario que requiera que se le conceda acceso para usar LUIS en Azure AD.

* En primer lugar, el usuario inicia sesión en LUIS y ve el cuadro de diálogo emergente que necesita aprobación de administrador. El usuario se pone en contacto con el administrador de inquilinos antes de continuar.
* En segundo lugar, el administrador de inquilinos inicia sesión en LUIS y ve un cuadro de diálogo emergente de flujo de consentimiento. Este es el cuadro de diálogo que debe usar el administrador para conceder permiso al usuario. Una vez que el administrador acepte el permiso, el usuario podrá continuar con LUIS. Si el administrador de inquilinos no inicia sesión en LUIS, el administrador puede acceder a [consentimiento](https://account.activedirectory.windowsazure.com/r#/applications) para LUIS. En esta página puede filtrar la lista por elementos que incluyan el nombre `LUIS`.

Si el administrador de inquilinos desea solo que determinados usuarios puedan usar LUIS, hay un par de soluciones posibles:
* dar el "consentimiento del administrador" (dar el consentimiento a todos los usuarios de Azure AD), pero, luego, establecer "Asignación de usuarios necesaria" en "Sí", en Propiedades de la aplicación de empresa y, por último, asignar o agregar solo los usuarios deseados a la aplicación. Con este método, el administrador aún está proporcionando "consentimiento del administrador" a la aplicación; sin embargo, es posible controlar los usuarios que pueden acceder a ella.
* Una segunda solución consiste en usar la [API de administración de identidades y acceso de Azure AD en Microsoft Graph](/graph/azuread-identity-access-management-concept-overview) para dar consentimiento a cada usuario específico.

Más información sobre el consentimiento y los usuarios de Azure Active Directory:
* [Restricción de la aplicación](../../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md) a un conjunto de usuarios

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [cómo usar versiones](luis-how-to-manage-versions.md) para controlar el ciclo de vida de la aplicación.
* Entienda cómo funcionan los [recursos de creación](luis-how-to-azure-subscription.md) y la [incorporación de colaboradores](luis-how-to-collaborate.md) en este recurso.
* Obtenga información sobre [cómo crear](luis-how-to-azure-subscription.md) los recursos de creación y de tiempo de ejecución.
* Migre al nuevo [recurso de creación](luis-migration-authoring.md).
