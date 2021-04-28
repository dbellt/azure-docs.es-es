---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 10/16/2019
ms.author: mimart
ms.openlocfilehash: 85883cd465626764d7af0c20af480b6075e13f9e
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070592"
---
Los recursos de API web tienen que registrarse en el inquilino antes de que puedan aceptar y responder a solicitudes de recursos protegidos por aplicaciones cliente que presenten un token de acceso.

Para registrar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](../articles/active-directory-b2c/app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *webapi1*.
1. En **URI de redirección**, seleccione **Web** y escriba el punto de conexión donde Azure AD B2C deberá devolver los tokens que solicite la aplicación. En este tutorial, el ejemplo se ejecuta localmente y escucha en el puerto `http://localhost:5000`.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

A continuación, habilite el flujo de concesión implícita:

1. En **Administrar**, seleccione **Autenticación**.
1. Seleccione **Probar la nueva experiencia** (si se muestra).
1. En **Concesión implícita**, active las casillas **Tokens de acceso** y **Tokens de id.**
1. Seleccione **Guardar**.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Aplicaciones (heredado)** y, a continuación, **Agregar**.
1. Escriba un nombre para la aplicación. Por ejemplo, *webapi1*.
1. En **Incluir aplicación web o API web**, seleccione **Sí**.
1. En **Permitir flujo implícito**, seleccione **Sí**.
1. En **Dirección URL de respuesta**, escriba un punto de conexión donde Azure AD B2C devolverá los tokens que solicite la aplicación. En este tutorial, el ejemplo se ejecuta localmente y escucha en el puerto `https://localhost:5000`.
1. En **URI de id. de aplicación**, agregue un identificador de punto de conexión de API para el identificador URI que se muestra. Para este tutorial, escriba `api`, con el fin de que el identificador URI completo sea similar a `https://contosob2c.onmicrosoft.com/api`.
1. Seleccione **Crear**.
1. Anote el valor de **ID. DE APLICACIÓN** para usarlo en un paso posterior.