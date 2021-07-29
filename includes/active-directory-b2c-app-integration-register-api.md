---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: 59d65673c425523fae46a85db18ca76db5d420eb
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073185"
---
Siga estos pasos para crear el registro de la aplicación de API web:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *my-api1*. Deje los valores predeterminados para el **URI de redirección**. 
1. Seleccione **Registrar**.
1. Una vez completado el registro de la aplicación, seleccione **Información general.**
1. Anote el **Id. de aplicación (cliente)** ; lo usará en un paso posterior al configurar la aplicación web.

    ![Captura de pantalla que muestra cómo obtener un identificador de la aplicación de la API web.](./media/active-directory-b2c-app-integration-register-api/get-azure-ad-b2c-web-api-app-id.png)