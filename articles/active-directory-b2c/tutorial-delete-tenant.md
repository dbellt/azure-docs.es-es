---
title: 'Limpieza de recursos y eliminación de un inquilino: Azure Active Directory B2C'
description: Pasos que describen cómo eliminar un inquilino de Azure AD B2C. Obtenga información sobre cómo eliminar todos los recursos de inquilino y, a continuación, eliminar el inquilino.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/23/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5935847c4abdf3bc9da60937096a19d44fb2d09d
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107982063"
---
# <a name="clean-up-resources-and-delete-the-tenant"></a>Limpieza de recursos y eliminación del inquilino

Cuando haya terminado los tutoriales de Azure AD B2C, puede eliminar el inquilino que usó para las pruebas o el entrenamiento. Para eliminar el inquilino, primero debe eliminar todos los recursos del inquilino. En este artículo, hará lo siguiente:

> [!div class="checklist"]
> * Usar la opción **Eliminar inquilino** para identificar las tareas de limpieza.
> * Eliminar recursos de inquilino (flujos de usuario, proveedores de identidades, aplicaciones, usuarios).
> * Eliminar el inquilino.

## <a name="identify-cleanup-tasks"></a>Identificación de las tareas de limpieza

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un rol de administrador global o de administrador de suscripciones. Use la misma cuenta profesional o educativa o la misma cuenta de Microsoft que ha usado para registrarse en Azure.

2. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

3. Seleccione el servicio **Azure Active Directory**.

4. En **Administrar**, seleccione **Propiedades**.

5. En **Administración del acceso para los recursos de Azure**, seleccione **Sí** y luego **Guardar**.

6. Cierre la sesión de Azure Portal y vuelva a iniciar sesión para actualizar el acceso. Use el filtro **Directorio y suscripción** del menú superior para volver a seleccionar el directorio que contiene el inquilino de Azure AD B2C.

7. Seleccione el servicio **Azure Active Directory**.

8. En la página **Información general**, seleccione **Eliminar inquilino**. La columna **Acción necesaria** indica los recursos que debe quitar para poder eliminar el inquilino.

   ![Eliminación de tareas de inquilino](media/tutorial-delete-tenant/delete-tenant-tasks.png)

## <a name="delete-tenant-resources"></a>Eliminación de recursos de inquilino

Si tiene abierta la página de confirmación de la sección anterior, puede usar los vínculos de la columna **Acción necesaria** para abrir las páginas de Azure Portal donde puede quitar estos recursos. O bien, puede quitar recursos de inquilino desde el servicio Azure AD B2C mediante los pasos siguientes.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un rol de administrador global o de administrador de suscripciones. Use la misma cuenta profesional o educativa o la misma cuenta de Microsoft que ha usado para registrarse en Azure.

2. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

3. Seleccione el servicio **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.

4. Elimine todos los usuarios, *excepto* la cuenta de administrador en la que ha iniciado sesión actualmente: en **Administrar**, seleccione **Usuarios**. En la página **Todos los usuarios**, active la casilla situada junto a cada usuario (excepto la cuenta de administrador con la que ha iniciado sesión actualmente). Seleccione **Aceptar** y, después, seleccione **Sí** cuando se le solicite.

   ![Eliminación de usuarios](media/tutorial-delete-tenant/delete-users.png)
   
5. Elimine registros de aplicaciones y *b2c-extensions-app*: en **Administrar**, seleccione **Registros de aplicaciones**. Seleccione la pestaña **Todas las aplicaciones**. Seleccione una aplicación y, a continuación, seleccione **Eliminar**. Repita la acción para todas las aplicaciones, incluida la aplicación **b2c-extensions-app**.

   ![Eliminar aplicación](media/tutorial-delete-tenant/delete-applications.png)

6. Elimine los proveedores de identidades que configuró: en **Administrar**, seleccione **Proveedores de identidades**. Seleccione el proveedor de identidades que configuró y, a continuación, seleccione **Quitar**.

   ![Eliminación del proveedor de identidades](media/tutorial-delete-tenant/identity-providers.png)

8. Elimine flujos de usuario: en **Directivas,** seleccione **Flujos de usuario**. Junto a cada flujo de usuario, seleccione los puntos suspensivos (...) y, a continuación, seleccione **Eliminar**.

   ![Eliminación de flujos de usuario](media/tutorial-delete-tenant/user-flow.png)

9. Elimine claves de directiva: en **Directivas**, seleccione **Identity Experience Framework** y, posteriormente, **Claves de directiva**. Junto a cada clave de directiva, seleccione los puntos suspensivos (...) y, a continuación, seleccione **Eliminar**.

10. Elimine directivas personalizadas: en **Directivas**, seleccione **Identity Experience Framework**, seleccione **Directivas personalizadas** y, a continuación, elimine todas las directivas.

## <a name="delete-the-tenant"></a>Eliminación del inquilino

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un rol de administrador global o de administrador de suscripciones. Use la misma cuenta profesional o educativa o la misma cuenta de Microsoft que ha usado para registrarse en Azure.

2. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

3. Seleccione el servicio **Azure Active Directory**.

4. Si aún no se ha concedido permisos de administración de acceso, haga lo siguiente:

   * En **Administrar**, seleccione **Propiedades**.
   * En **Administración del acceso para los recursos de Azure**, seleccione **Sí** y luego **Guardar**.
   * Cierre la sesión de Azure Portal y vuelva a iniciar sesión para actualizar el acceso y seleccione el servicio **Azure Active Directory**.

5. En la página **Información general**, seleccione **Eliminar inquilino**.

   ![Eliminación del inquilino](media/tutorial-delete-tenant/delete-tenant.png)

6. Siga las instrucciones que aparecen en pantalla para completar el proceso.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Eliminar los recursos de inquilino
> * Eliminar el inquilino

A continuación, obtenga más información sobre cómo empezar a trabajar con los [flujos de usuario y las directivas personalizadas](user-flow-overview.md) de Azure AD B2C.
