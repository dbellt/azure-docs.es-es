---
title: 'Adición y administración de usuarios para marketplace comercial: Azure Marketplace'
description: Obtenga información sobre cómo administrar usuarios en el programa de marketplace comercial para una cuenta de marketplace comercial de Microsoft en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: varsha-sarah
ms.author: vavargh
ms.custom: contperf-fy21q2
ms.date: 04/07/2021
ms.openlocfilehash: 108beee178cbf65bb4ba4f29479b335ab8d01a1e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141602"
---
# <a name="add-and-manage-users-for-the-commercial-marketplace"></a>Adición y administración de usuarios para el marketplace comercial

**Roles adecuados**

- Propietario
- Manager

La sección **Usuarios** del Centro de partners (en **Configuración de la cuenta**) le permite usar Azure AD para administrar los usuarios, grupos y aplicaciones de Azure AD que tienen acceso a esa cuenta. La cuenta debe tener permisos con nivel de administrador sobre la [cuenta profesional (inquilino de Azure AD)](company-work-accounts.md) en la que desea agregar o editar usuarios. Para administrar los usuarios de una cuenta o inquilino profesional diferente, necesitará cerrar sesión y volver a iniciarla como un usuario con permisos de **administrador** sobre esa cuenta profesional o de inquilino.

Después de haber iniciado sesión con su cuenta profesional (inquilino de Azure AD), puede agregar y administrar usuarios:

## <a name="add-existing-users"></a>Incorporación de usuarios ya existentes

Para agregar usuarios a la cuenta del Centro de partners que ya existen en la [cuenta profesional (inquilino de Azure AD)](company-work-accounts.md) de la empresa:

1. Vaya a **Usuarios** (en **Configuración de la cuenta**) y seleccione **Agregar usuarios**.
1. Seleccione uno o varios usuarios en la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios concretos. *Si selecciona más de un usuario para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varios usuarios con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
1. Cuando haya terminado de elegir los usuarios, seleccione **Agregar selección**.
1. En la sección **Roles**, especifique los roles o permisos personalizados de los usuarios seleccionados.
1. Seleccione **Guardar**.

## <a name="create-new-users"></a>Creación de nuevos usuarios

Para crear nuevas cuentas de usuario, debe tener una cuenta con permisos de [administrador global](../active-directory/roles/permissions-reference.md).

1. Vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar usuarios** y, a continuación, elija **Crear nuevos usuarios**.
1. Escriba un nombre, apellidos y nombre de usuario para cada usuario nuevo.
1. Si desea que el nuevo usuario tenga una cuenta de administrador global en el directorio de la organización, active la casilla denominada **Convertir a este usuario en administrador global de Azure AD, con control total sobre todos los recursos del directorio**. Esto le dará al usuario acceso completo a todas las características administrativas de la instancia de Azure AD de la empresa. Podrá agregar y administrar usuarios en la cuenta profesional de la organización (inquilino de Azure AD), aunque no en el Centro de partners, a menos que otorgue los permisos y roles adecuados para la cuenta de este.
1. Si ha seleccionado la casilla **Convertir a este usuario en administrador global**, deberá proporcionar un *correo electrónico de recuperación de contraseña* para que el usuario pueda recuperar su contraseña si es necesario.
1. En la sección **Pertenencia a grupos**, seleccione los grupos a los que desee que pertenezca el nuevo usuario.
1. En la sección **Roles**, especifique los roles o permisos personalizados del usuario.
1. Seleccione **Guardar**.

La creación de un nuevo usuario en el Centro de partners, también crea una cuenta para ese usuario en la cuenta profesional (inquilino de Azure AD) en la que ha iniciado sesión. Si realiza cambios en el nombre de un usuario del Centro de partners, los mismos cambios se realizarán en la cuenta profesional de la organización (inquilino de Azure AD).

## <a name="invite-new-users-by-email"></a>Invitar a nuevos usuarios por correo electrónico

Para invitar a los usuarios que actualmente no forman parte de la cuenta profesional de la empresa (inquilino de Azure AD) mediante correo electrónico, debe tener una cuenta con permisos de [administrador global](../active-directory/roles/permissions-reference.md).

1. Vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar usuarios** y, a continuación, elija **Invite users by email** (Invitar usuarios por correo electrónico).
1. Escriba una o varias direcciones de correo electrónico (hasta 10), separadas por comas o por puntos y comas.
1. En la sección **Roles**, especifique los roles o permisos personalizados del usuario.
1. Seleccione **Guardar**.

Los usuarios recibirán una invitación por correo electrónico para unirse a la cuenta del Centro de partners. Se creará una nueva cuenta de usuario invitado en su cuenta profesional (inquilino de Azure AD). Cada usuario deberá aceptar la invitación antes de poder acceder a la cuenta.

Si necesita volver a enviar una invitación, vaya a la página *Usuarios*, busque la invitación en la lista de usuarios y seleccione su dirección de correo electrónico (o el texto que dice *invitación pendiente*). A continuación, en la parte inferior de la página, seleccione **Volver a enviar la invitación**.

Si la organización usa [integración de directorios](/previous-versions/azure/azure-services/jj573653(v=azure.100)) para sincronizar el servicio de directorio local con Azure AD, no podrá crear nuevos usuarios, grupos o aplicaciones de Azure AD en el Centro de partners. Usted (u otro administrador del directorio local) deberán crearlos directamente en el directorio local antes de poder verlos y agregarlos en el Centro de partners.

## <a name="remove-a-user"></a>Eliminación de un usuario

Para quitar un usuario de la cuenta profesional (inquilino de Azure AD), vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione el usuario que desee eliminar mediante la casilla de verificación de la columna situada más a la derecha y, a continuación, elija **Quitar** en el listado de acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar los usuarios seleccionados.

## <a name="change-a-user-password"></a>Cambiar una contraseña de usuario

Si uno de los usuarios tiene que cambiar su contraseña, podrá hacerlo él mismo si proporcionó un *correo electrónico de recuperación de contraseña* al crear la cuenta de usuario. También puede actualizar la contraseña de un usuario mediante los pasos siguientes. Para cambiar la contraseña de un usuario en la cuenta profesional de la empresa (inquilino de Azure AD), debe iniciar sesión en una cuenta con permisos de [administrador global](../active-directory/roles/permissions-reference.md). Esta operación cambiará la contraseña del usuario en el inquilino de Azure AD y la contraseña que usa para acceder al Centro de partners.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione el nombre de la cuenta de usuario que desea editar.
1. Seleccione el botón **Restablecer contraseña** situado en la parte inferior de la página.
1. Aparecerá una página de confirmación para mostrar la información de inicio de sesión del usuario, incluida una contraseña temporal. No olvide imprimir o copiar esta información y proporcionársela al usuario, ya que no podrá acceder a la contraseña temporal después de salir de esta página.