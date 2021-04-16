---
title: Configuración del registro y del inicio de sesión con OpenID Connect
titleSuffix: Azure AD B2C
description: Configuración del registro y del inicio de sesión con cualquier proveedor de identidades de OpenID Connect (IdP) en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/05/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 21f31a21921953148ae0438cee04d846706161cc
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382936"
---
# <a name="set-up-sign-up-and-sign-in-with-openid-connect-using-azure-active-directory-b2c"></a>Configuración de la suscripción y el inicio de sesión con OpenID Connect mediante Azure Active Directory B2C

[OpenID Connect](openid-connect.md) es un protocolo de autenticación basado en OAuth 2.0 que se puede usar para el inicio de sesión de usuario seguro. La mayoría de los proveedores de identidades que usan este protocolo se admiten en Azure AD B2C. En este artículo se explica cómo puede agregar proveedores de identidades personalizados de OpenID Connect en sus flujos de usuario.

## <a name="add-the-identity-provider"></a>Agregar el proveedor de identidades

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, haga clic en el filtro **Directorio y suscripción** del menú superior y seleccione el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Nuevo proveedor de OpenID Connect**.
1. Escriba un **nombre**. Por ejemplo, escriba *Contoso*.

## <a name="configure-the-identity-provider"></a>Configurar el proveedor de identidades

Cada proveedor de identidades de OpenID Connect describe un documento de metadatos que contiene la mayor parte de la información necesaria para iniciar sesión. Esta información incluye las direcciones URL que se usan y la ubicación de las claves de firma pública del servicio. El documento de metadatos de OpenID Connect siempre se encuentra en un punto de conexión que termina en `.well-known/openid-configuration`. En cuanto al proveedor de identidades de OpenID Connect que quiere agregar, escriba su URL de metadatos.

## <a name="client-id-and-secret"></a>Id. de cliente y secreto

Para permitir que los usuarios inicien sesión, el proveedor de identidades pide a los desarrolladores que registren una aplicación en su servicio. Esta aplicación tiene un identificador que se denomina **ID de cliente** y un **secreto de cliente**. Copie estos valores desde el proveedor de identidades y escríbalos en los campos correspondientes.

> [!NOTE]
> El secreto del cliente es opcional. Pero debe especificar un secreto de cliente si quiere usar el [flujo de códigos de autorización](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth), que usa el secreto para intercambiar el código del token.

## <a name="scope"></a>Ámbito

El ámbito define la información y los permisos que quiere recopilar del proveedor de identidades, por ejemplo `openid profile`. Para recibir el token de identificación del proveedor de identidades, se debe especificar el ámbito `openid`. Sin ese token de identificador, los usuarios no pueden iniciar sesión en Azure AD B2C mediante el proveedor de identidades personalizado. Pueden anexarse otros ámbitos separados con espacios. Consulte la documentación del proveedor de identidades personalizado para ver otros ámbitos disponibles.

## <a name="response-type"></a>Tipo de respuesta

El tipo de respuesta describe qué tipo de información se envía en la llamada inicial al valor `authorization_endpoint` del proveedor de identidades personalizado. Pueden usarse estos tipos de respuesta:

* `code`: según el [flujo de código de autorización](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth), se devolverá un código a Azure AD B2C. Azure AD B2C llama a `token_endpoint` para intercambiar el código del token.
* `id_token`: se devuelve un token de identificador a Azure AD B2C desde el proveedor de identidades personalizado.

## <a name="response-mode"></a>Modo de respuesta

El modo de respuesta define el método que se debe usar para devolver los datos a Azure AD B2C desde el proveedor de identidades personalizado. Se pueden usar estos modos de respuesta:

* `form_post`: este modo de respuesta se recomienda para mayor seguridad. La respuesta se transmite a través del método HTTP `POST`, con el código o token codificado en el cuerpo mediante el formato `application/x-www-form-urlencoded`.
* `query`: el código o el token se devuelve como un parámetro de consulta.

## <a name="domain-hint"></a>Sugerencia de dominio

La sugerencia de dominio se puede usar para ir directamente a la página de inicio de sesión del proveedor de identidades especificado, en lugar de hacer que el usuario lo seleccione en la lista de proveedores de identidades disponibles. Para permitir este tipo de comportamiento, escriba un valor en la sugerencia de dominio. Para ir al proveedor de identidades personalizado, agregue el parámetro `domain_hint=<domain hint value>` al final de la solicitud cuando llame a Azure AD B2C para iniciar sesión.

## <a name="claims-mapping"></a>Asignación de notificaciones

Una vez que el proveedor de identidades personalizado devuelve un token de id. a Azure AD B2C, Azure AD B2C debe poder asignar las notificaciones del token recibido a aquellas que Azure AD B2C ya reconoce y usa. Para cada una de las asignaciones que se detallan aquí, lea la documentación del proveedor de identidades personalizado para comprender las notificaciones que se devuelven en los tokens del proveedor de identidades:

* **Id. de usuario**: especifique la notificación que proporciona el *identificador único* del usuario con sesión iniciada.
* **Nombre para mostrar**: especifique la notificación que proporciona el *nombre para mostrar* o el *nombre completo* del usuario.
* **Nombre propio**: especifique la notificación que proporciona el *nombre* del usuario.
* **Apellido**: especifique la notificación que proporciona el *apellido* del usuario.
* **Correo electrónico**: especifique la notificación que proporciona la *dirección de correo electrónico* del usuario.

## <a name="add-the-identity-provider-to-a-user-flow"></a>Adición del proveedor de identidades a un flujo de usuario 

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en el flujo de usuario al que quiere agregar el proveedor de identidades. 
1. En **Proveedores de identidades sociales**, seleccione el proveedor de identidades que ha agregado. Por ejemplo, *Contoso*.
1. Seleccione **Guardar**.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar flujo de usuario**.
1. En la página de registro o de inicio de sesión, seleccione el proveedor de identidades en el que desea iniciar sesión. Por ejemplo, *Contoso*.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.
