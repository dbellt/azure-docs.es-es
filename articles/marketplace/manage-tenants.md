---
title: Administración de inquilinos en el marketplace comercial - Azure Marketplace
description: Obtenga información acerca de cómo administrar inquilinos para el programa Marketplace comercial en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: varsha-sarah
ms.author: vavargh
ms.custom: contperf-fy21q2
ms.date: 04/07/2021
ms.openlocfilehash: 446792b26527126a4db99da14a2585c17cf8610c
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108072"
---
# <a name="manage-tenants-in-the-commercial-marketplace"></a>Administración de inquilinos en el marketplace comercial

**Roles adecuados**

- Manager

Un inquilino de Azure Active Directory (AD), también denominado *cuenta profesional* en esta documentación, es una representación de la configuración de la organización en Azure Portal y le ayuda a administrar una instancia específica de los servicios en la nube de Microsoft para los usuarios internos y externos. Si la organización se suscribió a un servicio en la nube de Microsoft, como Azure, Microsoft Intune u Microsoft 365, ya se estableció automáticamente un inquilino de Azure AD.

Puede configurar varios inquilinos para su uso con el Centro de partners. Se recomienda hacerlo si su empresa tiene varios inquilinos (por ejemplo, contoso.com, contoso.uk, entre otros) y puede vincular los inquilinos adicionales aquí. Esta asociación le permitiría agregar y administrar usuarios de los inquilinos adicionales en la cuenta de marketplace comercial.

Cualquier usuario con el rol Administrador en la cuenta del Centro de partners tendrá la opción de agregar y quitar inquilinos de Azure AD de la cuenta.

## <a name="add-an-existing-tenant"></a>Agregar un inquilino existente

Para asociar otro inquilino de Azure AD con la cuenta del Centro de partners:

1. En la parte superior derecha del Centro de partners, seleccione **Configuración** > **Configuración de la cuenta**.
1. En **Perfil de la organización**, seleccione **Inquilinos**. Se muestran las asociaciones de inquilino actuales.
1. En la pestaña **Desarrollador**, seleccione **Asociar**.
1. Escriba sus credenciales de Azure AD para el inquilino que desea asociar.
1. Revise el nombre de dominio y de la organización para el inquilino de Azure AD. Para completar la asociación, seleccione **Confirmar**.

Si la asociación se realiza correctamente, estará listo para agregar y administrar los usuarios de la cuenta en la sección Usuarios del Centro de partners. Para obtener información acerca de cómo agregar usuarios, consulte [Administración de usuarios](add-manage-users.md).

## <a name="create-a-new-tenant"></a>Creación de un nuevo inquilino

Para crear un nuevo inquilino de Azure AD con la cuenta del Centro de partners:

1. En la parte superior derecha del Centro de partners, seleccione **Configuración** > **Configuración de la cuenta**.
1. En **Perfil de la organización**, seleccione **Inquilinos**. Se muestran las asociaciones de inquilino actuales.
1. En la pestaña Desarrollador, seleccione **Crear**.
1. Escriba la información del directorio para la nueva instancia de Azure AD:
    - **Nombre de dominio**: nombre único que se va a usar para el dominio de Azure AD, junto con ".onmicrosoft.com". Por ejemplo, si ha escrito "ejemplo", el dominio de Azure AD sería "ejemplo.onmicrosoft.com".
    - **Dirección de correo electrónico de contacto**: una dirección de correo electrónico para ponernos en contacto con información sobre la cuenta si es necesario.
    - **Información de la cuenta de usuario del administrador global**: nombre, apellidos, nombre de usuario y contraseña que desea utilizar para la nueva cuenta de administrador global.
1. Seleccione Crear para confirmar la nueva información de dominio y de cuenta.
1. Inicie sesión con el nuevo nombre de usuario y contraseña de administrador global de Azure AD para empezar a [agregar y administrar usuarios](add-manage-users.md).

Para más información acerca de la creación de nuevos inquilinos en Azure Portal, en lugar de consultar el portal del Centro de partners, consulte el artículo [Creación de un inquilino en Azure Active Directory](/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).

## <a name="remove-a-tenant"></a>Eliminación de un inquilino

Para eliminar un inquilino de su cuenta del Centro de partners, busque su nombre en la página **Inquilinos** (en **Configuración de la cuenta**) y, a continuación, seleccione **Eliminar**. Se le pide que confirme que quiere eliminar el inquilino. Después de hacerlo, ningún usuario de ese inquilino podrá iniciar sesión en la cuenta del Centro de partners y se eliminarán los permisos que haya configurado para esos usuarios.

> [!TIP]
> No se puede eliminar un inquilino si se ha iniciado sesión en el Centro de partners con una cuenta del mismo inquilino. Para eliminar un inquilino, debe iniciar sesión en el Centro de partners como **Administrador** de otro inquilino que esté asociado con la cuenta. Si solo hay un inquilino asociado con la cuenta, ese inquilino solo se podrá eliminar después de iniciar sesión con la cuenta Microsoft que abrió la cuenta.
