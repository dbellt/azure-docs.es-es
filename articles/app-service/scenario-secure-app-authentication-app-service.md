---
title: 'Tutorial: Incorporación de la autenticación a una aplicación web en Azure App Service | Azure'
description: En este tutorial aprenderá a habilitar la autenticación y la autorización para una aplicación web que se ejecuta en Azure App Service. También limitará el acceso a la aplicación web solo a los usuarios de su organización.
services: active-directory, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 04/02/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: a99b017bf619bb7b3312f71a156b4d535b4067b3
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075869"
---
# <a name="tutorial-add-authentication-to-your-web-app-running-on-azure-app-service"></a>Tutorial: Incorporación de la autenticación a una aplicación web que se ejecuta en Azure App Service

Aprenda a habilitar la autenticación para la aplicación web que se ejecuta en Azure App Service y a limitar el acceso solo a los usuarios de su organización.

:::image type="content" source="./media/scenario-secure-app-authentication-app-service/web-app-sign-in.svg" alt-text="Diagrama que muestra el inicio de sesión del usuario." border="false":::

App Service proporciona compatibilidad integrada con la autenticación y la autorización, para que pueda proporcionar inicio de sesión a los usuarios y acceso a los datos escribiendo una cantidad mínima de código, o directamente sin código, en la aplicación web. El uso del módulo de autenticación y autorización de App Service no es obligatorio, pero ayuda a simplificar estos procesos en la aplicación. En este artículo se muestra cómo proteger una aplicación web con el módulo de autenticación y autorización de App Service, utilizando Azure Active Directory (Azure AD) como proveedor de identidades.

El módulo de autenticación y autorización se habilita y se configura mediante la configuración de Azure Portal y de la aplicación. No se necesitan SDK, lenguajes específicos o cambios en el código de la aplicación. Se admiten diversos proveedores de identidades, entre los que se incluyen Azure AD, cuenta de Microsoft, Facebook, Google y Twitter. Cuando está habilitado el módulo de autenticación y autorización, cada solicitud HTTP entrante pasa a través de él antes de que el código de aplicación lo controle. Para obtener más información, vea [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Configurar la autenticación de una aplicación web
> * Limitar el acceso a la aplicación web solo a los usuarios de su organización

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-and-publish-a-web-app-on-app-service"></a>Crear y publicar una aplicación web en App Service

Para este tutorial, necesita una aplicación web implementada en App Service. Puede usar una aplicación web existente, o puede seguir las indicaciones del [inicio rápido de ASP.NET Core](quickstart-dotnetcore.md) para crear y publicar una nueva aplicación web en App Service.

Si va a usar una aplicación web existente o a crear una nueva, tome nota del nombre de esta y del nombre del grupo de recursos en el que está implementada. Necesitará estos nombres en este tutorial. 

## <a name="configure-authentication-and-authorization"></a>Configuración de la autenticación y la autorización

Ahora tiene una aplicación web que se ejecuta en App Service. A continuación, habilitará la autenticación y la autorización para la aplicación web. En este caso, usa Azure AD como proveedor de identidades. Para más información, consulte [Configuración de una autenticación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](configure-authentication-provider-aad.md).

En el menú de [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos** o busque y seleccione **Grupos de recursos** desde cualquier página.

En **Grupos de recursos**, busque y seleccione el grupo de recursos. En **Información general**, seleccione la página de administración de la aplicación.

:::image type="content" alt-text="Captura de pantalla que muestra la selección de la página de administración de la aplicación." source="./media/scenario-secure-app-authentication-app-service/select-app-service.png":::

En el menú de la izquierda de la aplicación, seleccione **Autenticación** y, luego, haga clic en **Agregar proveedor de identidades**.

En la página **Agregar un proveedor de identidades**, seleccione **Microsoft** en **Proveedor de identidades** para iniciar sesión en las identidades de Microsoft y Azure AD.

En **Registro de aplicación** > **App registration type** (Tipo de registro de aplicación), seleccione **Create new app registration** (Crear registro de aplicación).

En **Registro de aplicación** > **Supported account types** (Tipos de cuenta admitidos), seleccione **Current tenant-single tenant** (Inquilino de inquilino único actual).

En la sección **App Service authentication settings** (Configuración de autenticación de App Service), deje la opción **Autenticación** establecida en **Requerir autenticación** y la opción **Unauthenticated requests** (Solicitudes no autenticadas) establecida en **HTTP 302 Found redirect: recommended for websites** (HTTP 302 Redirección no encontrada: recomendado para sitios web).

En la parte inferior de la página **Agregar un proveedor de identidades**, haga clic en **Agregar** para permitir la autenticación de la aplicación web.

:::image type="content" alt-text="Captura de pantalla que muestra la configuración de la autenticación." source="./media/scenario-secure-app-authentication-app-service/configure-authentication.png":::

Ahora tiene una aplicación que está protegida por la autenticación y la autorización de App Service.

> [!NOTE]
> Para permitir cuentas de otros inquilinos, cambie el "URL del emisor" a "https://login.microsoftonline.com/common/v2.0" editando el "Proveedor de identidades" en la hoja "Autenticación".
>

## <a name="verify-limited-access-to-the-web-app"></a>Comprobación del acceso limitado a la aplicación web

Al habilitar el módulo de autenticación y autorización de App Service, se crea un registro de aplicaciones en el inquilino de Azure AD. El registro de aplicaciones tiene el mismo nombre para mostrar que la aplicación web. Para comprobar la configuración, seleccione **Azure Active Directory** en el menú del portal y **Registros de aplicaciones**. Seleccione el registro de aplicaciones que creó. En la información general, compruebe que la opción **Tipos de cuenta compatibles** está establecida en **Solo mi organización**.

:::image type="content" alt-text="Captura de pantalla que muestra cómo comprobar el acceso." source="./media/scenario-secure-app-authentication-app-service/verify-access.png":::

Para comprobar que el acceso a la aplicación está limitado a los usuarios de la organización, inicie un explorador en el modo de incógnito o privado y vaya a `https://<app-name>.azurewebsites.net`. Se le debería dirigir a una página de inicio de sesión segura, lo que confirma que no se permite el acceso de los usuarios no autenticados al sitio. Inicie sesión como un usuario de la organización para obtener acceso al sitio. También puede iniciar un nueva ventana del explorador e intentar iniciar sesión con una cuenta personal para comprobar que los usuarios ajenos a la organización no tienen acceso.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya ha terminado con este tutorial y no necesita la aplicación web ni los recursos asociados, [elimine los recursos que creó](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a:

> [!div class="checklist"]
>
> * Configurar la autenticación de una aplicación web
> * Limitar el acceso a la aplicación web solo a los usuarios de su organización

> [!div class="nextstepaction"]
> [Accesos de App Service a Storage](scenario-secure-app-access-storage.md)
