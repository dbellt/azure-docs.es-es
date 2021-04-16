---
title: 'Adición de conectores de API a flujos de autoservicio de registro: Azure AD'
description: Configurar una API Web para usarla en un flujo de usuario.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: article
ms.date: 03/02/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c9bbdb831df9c51c6d80e6c441ac7bdd2778428
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105044556"
---
# <a name="add-an-api-connector-to-a-user-flow"></a>Adición de un conector de API a un flujo de usuario

Para usar un [conector de API](api-connectors-overview.md), primero debe crear el conector de API y, después, habilitarlo en un flujo de usuario.

> [!IMPORTANT]
>**A partir del 4 de enero de 2021**, Google deja [en desuso el soporte de inicio de sesión de WebView](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Si usa Google Federation o el registro de autoservicio con Gmail, debería [comprobar la compatibilidad de las aplicaciones nativas de línea de negocio](google-federation.md#deprecation-of-webview-sign-in-support).

## <a name="create-an-api-connector"></a>Creación de un conector de API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **All API connectors** (Todos los conectores de API) y **New API connector** (Nuevo conector de API).

   ![Adición de un conector de API nuevo](./media/self-service-sign-up-add-api-connector/api-connector-new.png)

5. Escriba el nombre para mostrar de la llamada. Por ejemplo, **Comprobar el estado de aprobación**.
6. Proporcione el valor de **Dirección URL del punto de conexión** de la llamada API.
7. Elija el **tipo de autenticación** y configure la información de autenticación para llamar a la API. Consulte la sección siguiente para ver las opciones de protección de la API.

    ![Configuración de un conector de API](./media/self-service-sign-up-add-api-connector/api-connector-config.png)

8. Seleccione **Guardar**.

## <a name="securing-the-api-endpoint"></a>Protección del punto de conexión de la API
Puede proteger el punto de conexión de la API mediante la autenticación HTTP básica o la autenticación de certificados de cliente HTTPS (versión preliminar). En cualquier caso, debe proporcionar las credenciales que Azure Active Directory usará al llamar a su punto de conexión de la API. A continuación, el punto de conexión de la API comprueba las credenciales y realiza decisiones de autorización.

### <a name="http-basic-authentication"></a>Autenticación HTTP básica
La autenticación HTTP básica se define en [RFC 2617](https://tools.ietf.org/html/rfc2617). Azure Active Directory envía una solicitud HTTP con las credenciales del cliente (`username` y `password`) en el encabezado `Authorization`. Las credenciales tienen el formato de cadena codificada en Base 64 `username:password`. A continuación, la API comprueba estos valores para determinar si se debe rechazar o no una llamada API.

### <a name="https-client-certificate-authentication-preview"></a>Autenticación con certificados de cliente HTTPS (versión preliminar)

> [!IMPORTANT]
> Esta funcionalidad está en versión preliminar y se proporciona sin ningún contrato de nivel de servicio. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La autenticación de certificados de cliente es un método de autenticación basado en certificados mutuo en el que el cliente proporciona un certificado de cliente al servidor para demostrar su identidad. En este caso, Azure Active Directory usará el certificado que carga como parte de la configuración del conector de la API. Esto sucede como parte del protocolo de enlace SSL. Después, el servicio de API puede limitar el acceso a los servicios que tienen certificados apropiados. El certificado de cliente es un certificado digital de la X.509 de PKCS12 (PFX). En entornos de producción, debe estar firmado por una autoridad de certificación. 

Para crear un certificado, puede usar [Azure Key Vault](../../key-vault/certificates/create-certificate.md), que tiene opciones para certificados autofirmados e integraciones con proveedores de emisores de certificados para certificados firmados. La configuración recomendada incluye:
- **Asunto**: `CN=<yourapiname>.<tenantname>.onmicrosoft.com`
- **Tipo de contenido**: `PKCS #12`
- **Tipo de Acton de duración**: `Email all contacts at a given percentage lifetime` o `Email all contacts a given number of days before expiry`
- **Tipo de clave**: `RSA`
- **Tamaño de clave**: `2048`
- **Clave privada exportable**: `Yes` (para poder exportar el archivo PFX)

Después, puede [exportar el certificado](../../key-vault/certificates/how-to-export-certificate.md). También puede usar el cmdlet [New-SelfSignedCertificate](../../active-directory-b2c/secure-rest-api.md#prepare-a-self-signed-certificate-optional) de PowerShell para generar un certificado autofirmado.

Después de tener un certificado, puede cargarlo como parte de la configuración del conector de la API. Tenga en cuenta que la contraseña solo es necesaria para los archivos de certificado protegidos mediante una contraseña.

La API debe implementar la autorización basada en certificados de cliente enviados con el fin de proteger los puntos de conexión de la API. Para obtener Azure App Service y Azure Functions, consulte [Configuración de la autenticación mutua de TLS](../../app-service/app-service-web-configure-tls-mutual-auth.md) para obtener información sobre cómo habilitar y *validar el certificado desde el código de la API*.  También puede usar Azure API Management para proteger la API y [comprobar las propiedades de los certificados de cliente](
../../api-management/api-management-howto-mutual-certificates-for-clients.md) con los valores deseados mediante expresiones de directiva.
 
Le recomendamos que establezca alertas de aviso para cuando expire el certificado. Tendrá que generar un nuevo certificado y repetir los pasos anteriores. El servicio de API puede seguir aceptando temporalmente los certificados antiguos y nuevos mientras se implementa el nuevo certificado. Para cargar un nuevo certificado en un conector de API existente, seleccione el conector de API en **All API connectors** (Todos los conectores de API) y haga clic en **Upload new certificate** (Cargar nuevo certificado). Azure Active Directory usará automáticamente el certificado cargado más recientemente que no haya caducado y haya pasado la fecha de inicio.

### <a name="api-key"></a>Clave de API
Algunos servicios utilizan un mecanismo de "clave de API" para ofuscar el acceso a los puntos de conexión HTTP durante el desarrollo. En el caso de [Azure Functions](../../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys), para ello, incluya `code` como parámetro de consulta en la **dirección URL del punto de conexión**. Por ejemplo, `https://contoso.azurewebsites.net/api/endpoint`<b>`?code=0123456789`</b>. 

No se trata de un mecanismo que se debe usar por sí solo en el entorno producción. Por lo tanto, siempre se requiere la configuración de autenticación básica o de certificado. Si no quiere implementar ningún método de autenticación (opción no recomendada) para fines de desarrollo, puede elegir la autenticación básica y usar valores temporales para `username` y `password` que la API pueda omitir mientras implementa la autorización en la API.

## <a name="the-request-sent-to-your-api"></a>Solicitud enviada a la API
Un conector de API se materializa como una solicitud **HTTP POST** y envía los atributos de usuario ("claims") como pares de clave y valor en un cuerpo JSON. Los atributos se serializan de forma similar a las propiedades de usuario de [Microsoft Graph](/graph/api/resources/user#properties). 

**Solicitud de ejemplo**
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ // Sent for Google, Facebook, and Email One Time Passcode identity providers 
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```

Solo se pueden enviar en la solicitud las propiedades de usuario y los atributos personalizados que se enumeran en la experiencia **Azure Active Directory** > **Identidades externas** > **Atributos de usuario personalizados**.

Los atributos personalizados existen en el formato **extension_\<extensions-app-id>_AttributeName** en el directorio. La API esperará recibir las notificaciones con este mismo formato serializado. Para más información acerca de los atributos personalizados, consulte cómo [definir atributos personalizados para flujos de autoservicio de registro](user-flow-add-custom-attributes.md).

Además, en todas las solicitudes se envía de forma predeterminada la notificación de **configuración regional de UI ("ui_locales")** . Proporciona la configuración o configuraciones regionales de un usuario, tal como están definidas en su dispositivo que la API puede utilizar para devolver respuestas internacionalizadas.

> [!IMPORTANT]
> Si una notificación no tiene un valor en el momento en que se llama al punto de conexión de la API, la notificación no se enviará a la API. La API debe estar diseñada para comprobar y controlar explícitamente el caso en el que una notificación no está en la solicitud.

> [!TIP] 
> La API puede utilizar notificaciones de [**identidades ("identities")**](/graph/api/resources/objectidentity) y de **dirección de correo electrónico ("email")** para identificar a un usuario antes de que tenga una cuenta en el inquilino. La notificación "identities" se envía cuando un usuario se autentica con un proveedor de identidades como Google o Facebook. Siempre se envía "email".

## <a name="enable-the-api-connector-in-a-user-flow"></a>Habilitación del conector de API en un flujo de usuario

Siga estos pasos para agregar el conector de API a un flujo de usuario de autoservicio de registro.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **Flujos de usuario** y, después, seleccione el flujo de usuario para el que desea habilitar el conector de API.
5. Seleccione **Conectores de API** y, después, seleccione los puntos de conexión de API que desea invocar en los pasos siguientes del flujo de usuario:

   - **Después de iniciar sesión con un proveedor de identidades**
   - **Antes de crear el usuario**

   ![Adición de API al flujo de usuario](./media/self-service-sign-up-add-api-connector/api-connectors-user-flow-select.png)

6. Seleccione **Guardar**.

## <a name="after-signing-in-with-an-identity-provider"></a>Después de iniciar sesión con un proveedor de identidades

Inmediatamente después de que el usuario se autentique con un proveedor de identidades (como Google, Facebook o Azure AD), se invoca un conector de API en este paso del proceso de registro. Este paso precede a la ***página de recopilación de atributos***, que es el formulario que se muestra al usuario para recopilar los atributos de usuario. Este paso no se invoca si un usuario se registra con una cuenta local.

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ // Sent for Google, Facebook, and Email One Time Passcode identity providers 
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "lastName":"Smith",
 "ui_locales":"en-US"
}
```

Las notificaciones exactas enviadas a la API dependen de la información proporcionada por el proveedor de identidades. Siempre se envía "email".

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Tipos de respuesta esperados de la API web en este paso

Cuando la API web recibe una solicitud HTTP de Azure AD durante un flujo de usuario, puede devolver estas respuestas:

- Respuesta de continuación
- Respuesta de bloqueo

#### <a name="continuation-response"></a>Respuesta de continuación

Una respuesta de continuación indica que el flujo de usuario debe continuar en el paso siguiente: la página de colección de atributos.

En una respuesta de continuación, la API puede devolver notificaciones. Si la API devuelve una notificación, esta realiza lo siguiente:

- Rellena previamente el campo de entrada en la página de colección de atributos.

Vea un ejemplo de una [respuesta de continuación](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Respuesta de bloqueo

Una respuesta de bloqueo termina el flujo de usuario. La API puede emitirla intencionadamente para detener el flujo de usuario y mostrar una página de bloqueo al usuario. La página de bloqueo muestra el mensaje `userMessage` proporcionado por la API.

Vea un ejemplo de una [respuesta de bloqueo](#example-of-a-blocking-response).

## <a name="before-creating-the-user"></a>Antes de crear el usuario

Después de la página de recopilación de atributos, se invoca un conector de API en este paso del proceso de registro, si se incluye uno. Este paso siempre se invoca antes de crear una cuenta de usuario en Azure AD. 

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ // Sent for Google, Facebook, and Email One Time Passcode identity providers 
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```
Las notificaciones exactas enviadas a la API dependen de la información recopilada por el usuario o proporcionada por el proveedor de identidades.

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Tipos de respuesta esperados de la API web en este paso

Cuando la API web recibe una solicitud HTTP de Azure AD durante un flujo de usuario, puede devolver estas respuestas:

- Respuesta de continuación
- Respuesta de bloqueo
- Respuesta de validación

#### <a name="continuation-response"></a>Respuesta de continuación
Una respuesta de continuación indica que el flujo de usuario debe continuar en el paso siguiente: creación del usuario en el directorio.

En una respuesta de continuación, la API puede devolver notificaciones. Si la API devuelve una notificación, esta realiza lo siguiente:

- Invalida cualquier valor que ya se haya asignado a la notificación de la página de colección de atributos.

Vea un ejemplo de una [respuesta de continuación](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Respuesta de bloqueo
Una respuesta de bloqueo termina el flujo de usuario. La API puede emitirla intencionadamente para detener el flujo de usuario y mostrar una página de bloqueo al usuario. La página de bloqueo muestra el mensaje `userMessage` proporcionado por la API.

Vea un ejemplo de una [respuesta de bloqueo](#example-of-a-blocking-response).

### <a name="validation-error-response"></a>Respuesta de error de validación
 Cuando la API responde con una respuesta de error de validación, el flujo de usuario permanece en la página de colección de atributos y se muestra `userMessage` al usuario. El usuario puede editar el formulario y volver a enviarlo. Este tipo de respuesta se puede usar para la validación de datos de entrada.

Vea un ejemplo de una [respuesta de error de validación](#example-of-a-validation-error-response).

## <a name="example-responses"></a>Respuestas de ejemplo

### <a name="example-of-a-continuation-response"></a>Ejemplo de una respuesta de continuación

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue",
    "postalCode": "12349", // return claim
    "extension_<extensions-app-id>_CustomAttribute": "value" // return claim
}
```

| Parámetro                                          | Tipo              | Obligatorio | Descripción                                                                                                                                                                                                                                                                            |
| -------------------------------------------------- | ----------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| version                                            | String            | Sí      | La versión de la API.                                                                                                                                                                                                                                                                |
| action                                             | String            | Sí      | El valor debe ser `Continue`.                                                                                                                                                                                                                                                              |
| \<builtInUserAttribute>                            | \<attribute-type> | No       | Los valores se pueden almacenar en el directorio si están seleccionados como **Claim to receive** (Notificación para recibir) en la configuración del conector de API y **User attribute** (Atributo de usuario) de un flujo de usuario. Los valores se pueden devolver en el token si están seleccionados como **Application claim** (Notificación de aplicación).                                              |
| \<extension\_{extensions-app-id}\_CustomAttribute> | \<attribute-type> | No       | No es necesario que la notificación devuelta contenga `_<extensions-app-id>_`. Los valores devueltos pueden sobrescribir los valores recopilados de un usuario. También se pueden devolver en el token si se configuran como parte de la aplicación.  |

### <a name="example-of-a-blocking-response"></a>Ejemplo de una respuesta de bloqueo

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "There was a problem with your request. You are not able to sign up at this time.",
}

```

| Parámetro   | Tipo   | Obligatorio | Descripción                                                                |
| ----------- | ------ | -------- | -------------------------------------------------------------------------- |
| version     | String | Sí      | La versión de la API.                                                    |
| action      | String | Sí      | El valor debe ser `ShowBlockPage`.                                              |
| userMessage | String | Sí      | Mensaje que se va a mostrar al usuario.                                            |

**Experiencia del usuario final con una respuesta de bloqueo**

![Página de bloqueo de ejemplo](./media/api-connectors-overview/blocking-page-response.png)

### <a name="example-of-a-validation-error-response"></a>Ejemplo de una respuesta de error de validación

```http
HTTP/1.1 400 Bad Request
Content-type: application/json

{
    "version": "1.0.0",
    "status": 400,
    "action": "ValidationError",
    "userMessage": "Please enter a valid Postal Code.",
}
```

| Parámetro   | Tipo    | Obligatorio | Descripción                                                                |
| ----------- | ------- | -------- | -------------------------------------------------------------------------- |
| version     | String  | Sí      | Versión de la API.                                                    |
| action      | String  | Sí      | El valor debe ser `ValidationError`.                                           |
| status      | Entero | Sí      | Debe ser un valor `400` para una respuesta ValidationError.                        |
| userMessage | String  | Sí      | Mensaje que se va a mostrar al usuario.                                            |

> [!NOTE]
> El código de estado HTTP debe ser "400" además del valor de "status" en el cuerpo de la respuesta.

**Experiencia del usuario final con una respuesta de error de validación**

![Página de validación de ejemplo](./media/api-connectors-overview/validation-error-postal-code.png)


## <a name="best-practices-and-how-to-troubleshoot"></a>Procedimientos recomendados y solución de problemas

### <a name="using-serverless-cloud-functions"></a>Uso de funciones de nube sin servidor
Las funciones sin servidor, como los desencadenadores HTTP en Azure Functions, proporcionan una manera sencilla de crear puntos de conexión de API para usarlos con el conector de API. Puede usar la función de nube sin servidor para, [por ejemplo](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts), crear la lógica de validación y limitar los registros a dominios específicos. La función de nube sin servidor también puede llamar e invocar otras API web, almacenes de usuarios y otros servicios en la nube para escenarios más complejos.

### <a name="best-practices"></a>Procedimientos recomendados
Asegúrese de que:
* La API sigue los contratos de solicitud y respuesta de la API, tal y como se describe anteriormente. 
* La **URL del punto de conexión** del conector de API apunta al punto de conexión de API correcto.
* La API comprueba explícitamente si hay valores NULL de las notificaciones recibidas.
* La API responde lo más rápido posible para garantizar una experiencia de usuario fluida.
    * Si usa una función sin servidor o un servicio web escalable, use un plan de hospedaje que mantenga la API "activa" o "caliente" en producción. Para Azure Functions, se recomienda usar el [plan Premium](../../azure-functions/functions-scale.md).

### <a name="use-logging"></a>Uso del registro
En general, resulta útil usar las herramientas de registro que habilita el servicio de API web, como [Application Insights](../../azure-functions/functions-monitoring.md), para supervisar la API en busca de códigos de error inesperados, excepciones y rendimiento deficiente.
* Supervise los códigos de estado HTTP que no sean HTTP 200 ni 400.
* Un código de estado HTTP 401 o 403 suele indicar que hay un problema con la autenticación. Compruebe la capa de autenticación de la API y la configuración correspondiente en el conector de API.
* Use niveles más agresivos de registro (por ejemplo, "trace" o "debug") en el desarrollo, si es necesario.
* Supervise la API en busca de tiempos de respuesta prolongados.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre cómo [agregar un flujo de trabajo personalizado de aprobación al autoservicio de registro](self-service-sign-up-add-approvals.md).
- Comience con nuestros [ejemplos de inicio rápido](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts).