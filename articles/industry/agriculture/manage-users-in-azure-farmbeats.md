---
title: Administración de usuarios en Azure FarmBeats
description: En este artículo se describe cómo administrar usuarios en Azure FarmBeats.
author: RiyazPishori
ms.topic: article
ms.date: 12/02/2019
ms.author: riyazp
ms.openlocfilehash: cb9a46dcfe3a99717c78e2401b2d5b69702ffdaf
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759732"
---
# <a name="manage-users"></a>Administrar usuarios

Azure FarmBeats incluye la administración de usuarios para personas que forman parte de su instancia de Azure Active Directory (Azure AD). Puede agregar usuarios a la instancia de Azure FarmBeats para acceder a las API, ver los mapas generados y acceder a la telemetría del sensor desde la granja de servidores.

## <a name="prerequisites"></a>Requisitos previos

- Se requiere la instalación de Azure FarmBeats. Para obtener más información, consulte [Instalación de Azure FarmBeats](install-azure-farmbeats.md).
- Identificadores de correo electrónico de los usuarios que quiere agregar o quitar de la instancia de Azure FarmBeats.

## <a name="manage-azure-farmbeats-users"></a>Administración de los usuarios de Azure FarmBeats

Azure FarmBeats usa Azure AD para la autenticación, el control de acceso y los roles. Puede agregar usuarios al inquilino de Azure AD como usuarios en Azure FarmBeats.

> [!NOTE]
> Si un usuario no es un usuario de inquilino de Azure AD, siga las instrucciones de la sección **Add Azure AD users** (Agregar usuarios de Azure AD) para completar la instalación.

Azure FarmBeats admite dos tipos de roles de usuario:

 - **Administrador**: acceso completo a las API del centro de datos de Azure FarmBeats. Los usuarios con este rol pueden consultar todos los objetos del centro de datos de Azure FarmBeats y hacer todas las operaciones desde el acelerador de FarmBeats.
 - **Solo lectura**: acceso de solo lectura a las API del centro de datos de FarmBeats. Los usuarios pueden ver las API del centro de datos, los paneles del acelerador y las asignaciones. Los usuarios con acceso de solo lectura no pueden realizar operaciones como la generación de asignaciones, la asociación de dispositivos o la creación de granjas de servidores.

## <a name="add-users-to-azure-farmbeats"></a>Adición de usuarios a Azure FarmBeats

Para agregar usuarios a Azure FarmBeats:

1. Inicie sesión en el acelerador y, luego, seleccione el icono **Configuración**.
2. Seleccione **Control de acceso**.

    ![Panel Configuración de granjas de servidores](./media/create-farms-in-azure-farmbeats/settings-users-1.png)

3. Escriba el identificador de correo electrónico del usuario al que quiere conceder acceso.
4. Seleccione el rol que quiera, **Administrador** o **Solo lectura**.
5. Seleccione **Agregar rol**.

El usuario agregado ahora podrá acceder a Azure FarmBeats (tanto al acelerador como al centro de datos).

## <a name="delete-users-from-azure-farmbeats"></a>Eliminación de usuarios de Azure FarmBeats

Para quitar usuarios del sistema Azure FarmBeats:

1. Inicie sesión en el acelerador y, luego, seleccione el icono **Configuración**.
2. Seleccione **Control de acceso**.
3. Seleccione **Eliminar**.

   El usuario se elimina del sistema. Obtendrá el siguiente mensaje de confirmación:

   ![Mensaje de confirmación de Azure FarmBeats](./media/create-farms-in-azure-farmbeats/manage-users-2.png)

## <a name="add-azure-ad-users"></a>Adición de usuarios de Azure AD

> [!NOTE]
> Los usuarios de Azure FarmBeats deben existir en el inquilino de Azure AD antes de asignarlos a aplicaciones y roles. Si no existe un usuario en el inquilino de Azure AD, siga las instrucciones de esta sección. Omita las instrucciones si ya existe un usuario en el inquilino de Azure AD.

Para agregar usuarios a Azure AD, realice los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En la esquina superior derecha, seleccione su cuenta y cambie al inquilino de Azure AD asociado a FarmBeats.
3. Seleccione **Azure Active Directory** > **Usuarios**.

    Se muestra una lista de usuarios de Azure AD.

4. Para agregar un usuario al directorio, seleccione **Nuevo usuario**. Para agregar un usuario externo, seleccione **Nuevo usuario invitado**.

    ![Panel "Todos los usuarios"](./media/create-farms-in-azure-farmbeats/manage-users-3.png)

5. Seleccione el nombre del nuevo usuario y, a continuación, complete los campos obligatorios para ese usuario.
6. Seleccione **Crear**.

Para obtener información acerca de cómo administrar usuarios de Azure AD, consulte [Incorporación o eliminación de usuarios en Azure AD](../../active-directory/fundamentals/add-users-azure-active-directory.md).

## <a name="next-steps"></a>Pasos siguientes

Agregó correctamente usuarios a la instancia de Azure FarmBeats. Ahora, obtenga información sobre cómo [crear y administrar granjas de servidores](manage-farms-in-azure-farmbeats.md#create-farms).