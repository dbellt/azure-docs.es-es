---
title: archivo de inclusión
description: archivo de inclusión
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.date: 10/02/2020
ms.openlocfilehash: 22411e5a80f555a3ead05d39466a7a175923d9bc
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105105700"
---
* Después de seleccionar la plataforma adecuada en el paso 4 de [Configuración de la plataforma](../articles/active-directory/develop/quickstart-register-app.md#configure-platform-settings), configure los valores de **URI de redirección** y **Tokens de acceso** en el panel lateral a la derecha de la interfaz de usuario.

    * El valor de **URI de redirección** debe coincidir con la dirección proporcionada en la solicitud de autenticación:

        * Si se trata de aplicaciones hospedadas en un entorno de desarrollo local, seleccione **Cliente público (móvil y escritorio)** . Asegúrese de establecer el **cliente público** en **Sí**.
        * En el caso de aplicaciones de página única hospedadas en Azure App Service, seleccione **Web**.

    * Determine si procede agregar una **URL de cierre de sesión**.

    * Marque **Tokens de acceso** o **Tokens de id.** para habilitar el flujo de concesión implícita.

    [![Creación de URI de redirección](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png)](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png#lightbox)

    Haga clic en **Configurar** y, a continuación, en **Guardar**.

* Asocie la aplicación de Azure Active Directory a Azure Time Series Insights. Seleccione **Permisos de API** > **Agregar un permiso** > **API usadas en mi organización**.

    [![Asociación de una API a la aplicación de Azure Active Directory](media/time-series-insights-aad-registration/active-directory-app-api-permission.png)](media/time-series-insights-aad-registration/active-directory-app-api-permission.png#lightbox)

   Escriba `Azure Time Series Insights` en la barra de búsqueda y, luego, seleccione `Azure Time Series Insights`.

* A continuación, especifique el tipo de permiso de API que requiere la aplicación. De forma predeterminada, se resalta la opción **Permisos delegados**. Elija un tipo de permiso y, luego, seleccione **Agregar permisos**.

    [![Especificación del tipo de permiso de API que requiere la aplicación](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png)](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png#lightbox)

* [Agregue credenciales](../articles/active-directory/develop/quickstart-register-app.md#add-credentials) si la aplicación va a llamar a las API del entorno ella misma. Las credenciales permiten que la aplicación se autentique a sí misma, por lo que no se requiere la interacción del usuario en tiempo de ejecución.