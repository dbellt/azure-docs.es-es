---
title: 'Uso de Mi personal para delegar la administración de usuarios: Azure AD | Microsoft Docs'
description: Delegación de la administración de usuarios mediante Mi personal y unidades administrativas
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.topic: how-to
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.date: 03/11/2021
ms.author: rolyon
ms.reviewer: sahenry
ms.custom: oldportal;it-pro;
ms.openlocfilehash: 106f44bd33fff88d93e0a4abfe678df4bdf9ca59
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108761730"
---
# <a name="manage-your-users-with-my-staff"></a>Administración de los usuarios con Mi personal

Mi personal permite delegar en una figura de autoridad, como un administrador de almacenes o un responsable de equipo, los permisos para garantizar que los miembros del personal puedan obtener acceso a sus cuentas de Azure AD. En lugar de depender de un departamento de soporte técnico central, las organizaciones pueden delegar tareas comunes, como el restablecimiento de contraseñas o el cambio de números de teléfono, en un administrador de equipos local. Con Mi personal, un usuario que no puede tener acceso a su cuenta puede recuperar el acceso con tan solo un par de clics, sin necesidad de ayuda por parte del personal de TI o el departamento de soporte técnico.

Antes de configurar Mi personal para su organización, recomendamos que revise esta documentación, así como la [documentación de usuario](../user-help/my-staff-team-manager.md), para asegurarse de que comprende cómo funciona y afecta a los usuarios. Puede aprovechar la documentación de usuario para entrenar y preparar a los usuarios para la nueva experiencia y ayudar a garantizar un lanzamiento satisfactorio.

## <a name="how-my-staff-works"></a>Cómo funciona Mi personal

Mi personal se basa en unidades administrativas, que son un contenedor de recursos que se pueden usar para restringir el ámbito del control administrativo de una asignación de roles. Para más información, consulte [Administración de unidades administrativas en Azure Active Directory](administrative-units.md). En Mi personal, las unidades administrativas se pueden usar para contener un grupo de usuarios de un almacén o departamento. A continuación, se puede asignar un administrador de equipo a un rol administrativo en el ámbito de una o varias unidades.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.

  * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a la suscripción.

  * Si es necesario, [cree un inquilino de Azure Active Directory](../fundamentals/sign-up-organization.md) o [asocie una suscripción a Azure con su cuenta](../fundamentals/active-directory-how-subscriptions-associated-directory.md).
* Necesita privilegios de *Administrador global* en el inquilino de Azure AD para permitir la autenticación basada en SMS.
* Cada usuario que esté habilitado en la directiva de métodos de autenticación con mensaje de texto debe tener licencia, aunque no la usen. Cada usuario habilitado debe tener una de las siguientes licencias de Azure AD o Microsoft 365:

  * [Azure AD Premium P1 o P2](https://azure.microsoft.com/pricing/details/active-directory/)
  * [Microsoft 365 (M365) F1 o F3](https://www.microsoft.com/licensing/news/m365-firstline-workers)
  * [Enterprise Mobility + Security (EMS) E3 o E5](https://www.microsoft.com/microsoft-365/enterprise-mobility-security/compare-plans-and-pricing) o [Microsoft 365 (M365) E3 o E5](https://www.microsoft.com/microsoft-365/compare-microsoft-365-enterprise-plans)

## <a name="how-to-enable-my-staff"></a>Cómo habilitar Mi personal

Una vez que haya configurado las unidades administrativas, puede aplicar este ámbito a los usuarios con acceso a Mi personal. Solo los usuarios a los que se ha asignado un rol administrativo pueden tener acceso a Mi personal. Para habilitar Mi personal, siga los pasos siguientes:

1. Inicie sesión en Azure Portal como administrador de usuarios.
2. Vaya a **Azure Active Directory** > **Configuración de usuario** > **Versiones preliminares de características de usuarios** > **Administrar la configuración de la versión preliminar de características del usuario**.
3. En **Los administradores pueden tener acceso a Mi personal**, puede elegir habilitar para todos los usuarios, para usuarios seleccionados o sin acceso de los usuarios.

> [!Note]
> Solo los usuarios a los que se ha asignado un rol administrativo pueden tener acceso a Mi personal. Si habilita Mi personal para un usuario que no tiene asignado un rol administrativo, no podrá tener acceso a Mi personal.

## <a name="conditional-access"></a>Acceso condicional

Puede proteger el portal Mi personal mediante una directiva de acceso condicional de Azure AD. Úselo para tareas como requerir la autenticación multifactor antes de tener acceso a Mi personal.

Se recomienda que proteja Mi personal mediante [directivas de acceso condicional de Azure AD](../conditional-access/index.yml). Para aplicar una directiva de acceso condicional a Mi personal, primero debe visitar el sitio de Mi personal una vez durante unos minutos para aprovisionar automáticamente la entidad de servicio en el inquilino para que la use el acceso condicional.

Verá la entidad de servicio al crear una directiva de acceso condicional que se aplica a la aplicación en la nube Mi personal.

![Creación de una directiva de acceso condicional para la aplicación Mi personal](./media/my-staff-configure/conditional-access.png)

## <a name="using-my-staff"></a>Uso de Mi personal

Cuando un usuario va a Mi personal, se le muestran los nombres de las [unidades administrativas](administrative-units.md) sobre las que tienen permisos administrativos. En la [documentación de usuario de Mi personal](../user-help/my-staff-team-manager.md), usamos el término "ubicación" para hacer referencia a las unidades administrativas. Si los permisos de un administrador no tienen ningún ámbito de unidad administrativa, estos se aplicarán en toda la organización. Una vez que se ha habilitado Mi personal, los usuarios que están habilitados y a los que se ha asignado un rol administrativo pueden tener acceso al mismo a través de [https://mystaff.microsoft.com](https://mystaff.microsoft.com). Puede seleccionar una unidad administrativa para ver los usuarios de esta y seleccionar un usuario para abrir su perfil.

## <a name="reset-a-users-password"></a>Restablecimiento de la contraseña del usuario

Antes de que pueda restablecer contraseñas para usuarios locales, debe cumplir las siguientes condiciones de requisitos previos. Para obtener instrucciones detalladas, consulte el tutorial [Habilitación del autoservicio de restablecimiento de contraseña](../authentication/tutorial-enable-sspr-writeback.md).

* Configuración de los permisos de escritura diferida de contraseñas
* Habilitación de la Escritura diferida de contraseñas en Azure AD Connect
* Habilitación de la escritura diferida de contraseñas en el autoservicio de restablecimiento de contraseñas (SSPR) de Azure AD

Los roles siguientes tienen permiso para restablecer la contraseña de un usuario:

* [Administrador de autenticación](permissions-reference.md#authentication-administrator)
* [Administrador de autenticación con privilegios](permissions-reference.md#privileged-authentication-administrator)
* [Administrador global](permissions-reference.md#global-administrator)
* [Administrador del departamento de soporte técnico](permissions-reference.md#helpdesk-administrator)
* [Administrador de usuarios](permissions-reference.md#user-administrator)
* [Administrador de contraseñas](permissions-reference.md#password-administrator)

En **Mi personal**, abra el perfil de un usuario. Seleccione **Restablecer contraseña**.

* Si el usuario es solo de nube, puede ver una contraseña temporal que puede proporcionar al usuario.
* Si el usuario se sincroniza desde Active Directory local, puede escribir una contraseña que cumpla las directivas de AD locales. A continuación, puede proporcionar esa contraseña al usuario.

    ![Indicador de progreso de restablecimiento de contraseña y notificación de éxito](./media/my-staff-configure/reset-password.png)

Se obliga al usuario a cambiar su contraseña la próxima vez que inicie sesión.

## <a name="manage-a-phone-number"></a>Administración de un número de teléfono

En **Mi personal**, abra el perfil de un usuario.

* Seleccione la sección **Agregar número de teléfono** para agregar un número de teléfono del usuario.
* Seleccione **Editar número de teléfono** para cambiar el número de teléfono.
* Seleccione **Quitar número de teléfono** para quitar el número de teléfono del usuario.

En función de la configuración, el usuario puede utilizar el número de teléfono configurado para iniciar sesión con SMS, realizar la autenticación multifactor y llevar a cabo el autoservicio de restablecimiento de contraseña.

Para administrar el número de teléfono de un usuario, debe tener asignado uno de los roles siguientes:

* [Administrador de autenticación](permissions-reference.md#authentication-administrator)
* [Administrador de autenticación con privilegios](permissions-reference.md#privileged-authentication-administrator)
* [Administrador global](permissions-reference.md#global-administrator)

## <a name="search"></a>Search

Puede buscar unidades administrativas y usuarios de su organización mediante la barra de búsqueda de Mi personal. Puede buscar en todas las unidades administrativas y los usuarios de su organización, pero solo puede realizar cambios en los usuarios que se encuentran en una unidad administrativa en la que se le han concedido permisos administrativos.

También puede buscar un usuario dentro de una unidad administrativa. Para ello, use la barra de búsqueda de la parte superior de la lista de usuarios.

## <a name="audit-logs"></a>Registros de auditoría

Puede ver registros de auditoría de las acciones realizadas en Mi personal en el portal de Azure Active Directory. Si una acción realizada en Mi personal generó un registro de auditoría, lo verá indicado en DETALLES ADICIONALES en el evento de auditoría.

## <a name="next-steps"></a>Pasos siguientes

[Documentación de usuario de Mi personal](../user-help/my-staff-team-manager.md)
[Documentación de unidades administrativas](administrative-units.md)
