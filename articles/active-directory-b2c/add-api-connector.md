---
title: Adición de conectores de API a flujos de usuarios (versión preliminar)
description: Configure un conector de API para usarlo en un flujo de usuario.
services: active-directory-b2c
ms.service: active-directory
ms.subservice: B2C
ms.topic: how-to
ms.date: 04/28/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 62de5eff098c467f048ae33cd38e7c730af863bc
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108174621"
---
# <a name="add-an-api-connector-to-a-sign-up-user-flow"></a>Adición de un conector de API a un flujo de usuario de registro

Como desarrollador o administrador de TI, puede usar conectores de API para integrar los flujos de usuario de registro con las API REST a fin de personalizar la experiencia de registro e integrarla con los sistemas externos. Al final de este tutorial podrá crear un flujo de trabajo del usuario de Azure AD B2C que interactúe con [servicios API REST](api-connectors-overview.md). 

::: zone pivot="b2c-user-flow"

En este escenario, la API REST valida si el dominio de la dirección de correo electrónico es fabrikam.com o fabricam.com. El puesto proporcionado por el usuario supera los cinco caracteres. 

> [!IMPORTANT]
> Los conectores de API para el registro son una característica en versión preliminar pública de Azure AD B2C. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

::: zone-end

::: zone pivot="b2c-custom-policy"

En este escenario, se agregará la posibilidad de que los usuarios escriban un número de fidelidad en la página de inicio de sesión de Azure AD B2C. La API REST valida si la combinación de correo electrónico y número de fidelidad está asignada a un código promocional. Si la API REST encuentra un código promocional para este usuario, se devolverá a Azure AD B2C. Por último, el código promocional se insertará en las notificaciones del token para que la aplicación lo consuma.

La interacción también se puede diseñar como un paso de orquestación. Esto es adecuado cuando la API REST no va a validar datos en pantalla y siempre devuelve notificaciones. Para más información, consulte [Tutorial: Integración de intercambios de notificaciones de API REST en los recorridos de usuario de Azure AD B2C como un paso de orquestación](custom-policy-rest-api-claims-exchange.md).

::: zone-end

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="create-an-api-connector"></a>Creación de un conector de API

Para usar un [conector de API](api-connectors-overview.md), primero debe crear el conector de API y, después, habilitarlo en un flujo de usuario.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En **Servicios de Azure**, seleccione **Azure AD B2C**.
4. Seleccione **API connectors (Preview)** (Conectores de API [versión preliminar]) y, después, seleccione **New API connector** (Nuevo conector de API).

   ![Adición de un conector de API nuevo](./media/add-api-connector/api-connector-new.png)

5. Escriba el nombre para mostrar de la llamada. Por ejemplo, **Validar la información de usuario**.
6. Proporcione el valor de **Dirección URL del punto de conexión** de la llamada API.
7. Elija el **tipo de autenticación** y configure la información de autenticación para llamar a la API. Consulte la sección siguiente para ver las opciones de protección de la API.

    ![Configuración de un conector de API](./media/add-api-connector/api-connector-config.png)

8. Seleccione **Guardar**.

## <a name="securing-the-api-endpoint"></a>Protección del punto de conexión de la API
Puede proteger el punto de conexión de la API mediante la autenticación HTTP básica o la autenticación de certificados de cliente HTTPS (versión preliminar). En cualquier caso, debe proporcionar las credenciales que Azure AD B2C usará al llamar a su punto de conexión de la API. A continuación, el punto de conexión de la API comprueba las credenciales y realiza decisiones de autorización.

### <a name="http-basic-authentication"></a>Autenticación HTTP básica
La autenticación HTTP básica se define en [RFC 2617](https://tools.ietf.org/html/rfc2617). Azure AD B2C envía una solicitud HTTP con las credenciales del cliente (`username` y `password`) en el encabezado `Authorization`. Las credenciales tienen el formato de cadena codificada en Base 64 `username:password`. A continuación, la API comprueba estos valores para determinar si se debe rechazar o no una llamada API.

### <a name="https-client-certificate-authentication-preview"></a>Autenticación con certificados de cliente HTTPS (versión preliminar)

> [!IMPORTANT]
> Esta funcionalidad está en versión preliminar y se proporciona sin ningún contrato de nivel de servicio. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La autenticación de certificados de cliente es un método de autenticación basado en certificados mutuo en el que el cliente proporciona un certificado de cliente al servidor para demostrar su identidad. En este caso, Azure AD B2C usará el certificado que se carga como parte de la configuración del conector de la API. Esto sucede como parte del protocolo de enlace TLS/SSL. Después, el servicio de API puede limitar el acceso a los servicios que tienen certificados apropiados. El certificado de cliente es un certificado digital X.509 de PKCS12 (PFX). En entornos de producción, debe estar firmado por una autoridad de certificación. 

Para crear un certificado, puede usar [Azure Key Vault](../key-vault/certificates/create-certificate.md), que tiene opciones para certificados autofirmados e integraciones con proveedores de emisores de certificados para certificados firmados. La configuración recomendada incluye:
- **Asunto**: `CN=<yourapiname>.<tenantname>.onmicrosoft.com`
- **Tipo de contenido**: `PKCS #12`
- **Tipo de Acton de duración**: `Email all contacts at a given percentage lifetime` o `Email all contacts a given number of days before expiry`
- **Tipo de clave**: `RSA`
- **Tamaño de clave**: `2048`
- **Clave privada exportable**: `Yes` (para poder exportar el archivo PFX)

Después, puede [exportar el certificado](../key-vault/certificates/how-to-export-certificate.md). También puede usar el cmdlet [New-SelfSignedCertificate](../active-directory-b2c/secure-rest-api.md#prepare-a-self-signed-certificate-optional) de PowerShell para generar un certificado autofirmado.

Después de tener un certificado, puede cargarlo como parte de la configuración del conector de la API. Tenga en cuenta que la contraseña solo es necesaria para los archivos de certificado protegidos mediante una contraseña.

La API debe implementar la autorización basada en certificados de cliente enviados con el fin de proteger los puntos de conexión de la API. Para obtener Azure App Service y Azure Functions, consulte [Configuración de la autenticación mutua de TLS](../app-service/app-service-web-configure-tls-mutual-auth.md) para obtener información sobre cómo habilitar y *validar el certificado desde el código de la API*.  También puede usar Azure API Management para [comprobar las propiedades](
../api-management/api-management-howto-mutual-certificates-for-clients.md) de los certificados de cliente con los valores deseados mediante expresiones de directiva.

Le recomendamos que establezca alertas de aviso para cuando expire el certificado. Tendrá que generar un nuevo certificado y repetir los pasos anteriores. El servicio de API puede seguir aceptando temporalmente los certificados antiguos y nuevos mientras se implementa el nuevo certificado. Para cargar un nuevo certificado en un conector de API existente, seleccione el conector de API en **Conectores de API** y haga clic en **Cargar certificado nuevo**. Azure Active Directory usará automáticamente el certificado cargado más recientemente que no haya caducado y haya pasado la fecha de inicio.

### <a name="api-key"></a>Clave de API

Algunos servicios utilizan un mecanismo de "clave de API" para ofuscar el acceso a los puntos de conexión HTTP durante el desarrollo. En el caso de [Azure Functions](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys), para ello, incluya `code` como parámetro de consulta en la **dirección URL del punto de conexión**. Por ejemplo, `https://contoso.azurewebsites.net/api/endpoint`<b>`?code=0123456789`</b>. 

No se trata de un mecanismo que se debe usar por sí solo en el entorno producción. Por lo tanto, siempre se requiere la configuración de autenticación básica o de certificado. Si no quiere implementar ningún método de autenticación (opción no recomendada) para fines de desarrollo, puede elegir la autenticación básica y usar valores temporales para `username` y `password` que la API pueda omitir mientras implementa la autorización en la API.

## <a name="the-request-sent-to-your-api"></a>Solicitud enviada a la API
Un conector de API se materializa como una solicitud **HTTP POST** y envía los atributos de usuario ("claims") como pares de clave y valor en un cuerpo JSON. Los atributos se serializan de forma similar a las propiedades de usuario de [Microsoft Graph](/graph/api/resources/user#properties). 

**Solicitud de ejemplo**
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [
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

Solo se pueden enviar en la solicitud las propiedades de usuario y los atributos personalizados que se enumeran en la experiencia **Azure AD B2C** > **Atributos de usuario**.

Los atributos personalizados existen en el formato **extension_\<extensions-app-id>_CustomAttribute** en el directorio. La API esperará recibir las notificaciones con este mismo formato serializado. Para obtener más información sobre los atributos personalizados, consulte [Definición de atributos personalizados en Azure AD B2C](user-flow-custom-attributes.md).

Además, en todas las solicitudes se envía de forma predeterminada la notificación de **configuración regional de UI ("ui_locales")** . Proporciona la configuración o configuraciones regionales de un usuario, tal como están definidas en su dispositivo que la API puede utilizar para devolver respuestas internacionalizadas.

> [!IMPORTANT]
> Si una notificación no tiene un valor en el momento en que se llama al punto de conexión de la API, la notificación no se enviará a la API. La API debe estar diseñada para comprobar y controlar explícitamente el caso en el que una notificación no está en la solicitud.

> [!TIP] 
> La API puede utilizar notificaciones de [**identidades ("identities")**](/graph/api/resources/objectidentity) y de **dirección de correo electrónico ("email")** para identificar a un usuario antes de que tenga una cuenta en el inquilino. 

## <a name="enable-the-api-connector-in-a-user-flow"></a>Habilitación del conector de API en un flujo de usuario

Siga estos pasos para agregar el conector de API a un flujo de usuario de registro.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En **Servicios de Azure**, seleccione **Azure AD B2C**.
4. Seleccione **Flujos de usuario** y, después, seleccione el flujo de usuario para el que desea habilitar el conector de API.
5. Seleccione **Conectores de API** y, después, seleccione los puntos de conexión de API que desea invocar en los pasos siguientes del flujo de usuario:

   - **Después de iniciar sesión con un proveedor de identidades**
   - **Antes de crear el usuario**

   ![Adición de API al flujo de usuario](./media/add-api-connector/api-connectors-user-flow-select.png)

6. Seleccione **Guardar**.

## <a name="after-signing-in-with-an-identity-provider"></a>Después de iniciar sesión con un proveedor de identidades

Inmediatamente después de que el usuario se autentique con un proveedor de identidades (como Google, Facebook o Azure AD), se invoca un conector de API en este paso del proceso de registro. Este paso precede a la ***página de recopilación de atributos***, que es el formulario que se muestra al usuario para recopilar los atributos de usuario. Este paso no se invoca si un usuario se registra con una cuenta local.

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ 
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

Después de la página de recopilación de atributos, se invoca un conector de API en este paso del proceso de registro, si se incluye uno. Este paso siempre se invoca antes de crear una cuenta de usuario.

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [
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

Las notificaciones enviadas a la API dependen de la información recopilada por el usuario o proporcionada por el proveedor de identidades.

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
| action                                             | String            | Sí      | El valor debe ser `Continue`.                                                                                                                                                                                                                                                              |
| \<builtInUserAttribute>                            | \<attribute-type> | No       | Los valores devueltos pueden sobrescribir los valores recopilados de un usuario. También se pueden devolver en el token si están seleccionados como **Application claim** (Notificación de aplicación).                                              |
| \<extension\_{extensions-app-id}\_CustomAttribute> | \<attribute-type> | No       | No es necesario que la notificación contenga `_<extensions-app-id>_`. Los valores devueltos pueden sobrescribir los valores recopilados de un usuario. También se pueden devolver en el token si están seleccionados como **Application claim** (Notificación de aplicación).  |

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

![Página de bloqueo de ejemplo](./media/add-api-connector/blocking-page-response.png)

### <a name="example-of-a-validation-error-response"></a>Ejemplo de una respuesta de error de validación

```http
HTTP/1.1 400 Bad Request
Content-type: application/json

{
    "version": "1.0.0",
    "status": 400,
    "action": "ValidationError",
    "userMessage": "Please enter a valid Postal Code."
}
```

| Parámetro   | Tipo    | Obligatorio | Descripción                                                                |
| ----------- | ------- | -------- | -------------------------------------------------------------------------- |
| version     | String  | Sí      | Versión de la API.                                                    |
| action      | String  | Sí      | El valor debe ser `ValidationError`.                                           |
| status      | Entero/cadena | Sí      | Debe ser un valor `400` o `"400"` para una respuesta ValidationError.  |
| userMessage | String  | Sí      | Mensaje que se va a mostrar al usuario.                                            |

> [!NOTE]
> El código de estado HTTP debe ser "400" además del valor de "status" en el cuerpo de la respuesta.

**Experiencia del usuario final con una respuesta de error de validación**

![Página de validación de ejemplo](./media/add-api-connector/validation-error-postal-code.png)


::: zone-end

::: zone pivot="b2c-custom-policy"


## <a name="prepare-a-rest-api-endpoint"></a>Preparación del punto de conexión de API REST

Para este tutorial, debe tener una API REST que valide si una dirección de correo electrónico está registrada en el sistema de back-end con un identificador de fidelidad. Si lo está, la API REST debe devolver un código de promoción de registro, que el cliente puede usar para comprar mercancías desde la aplicación. De lo contrario, la API REST debería devolver un mensaje de error HTTP 409 similar al siguiente: "El id. de fidelidad "{id. fidelidad}" no está asociado con la dirección de correo "{correo}"".

El siguiente código JSON muestra los datos que Azure AD B2C enviará al punto de conexión de la API REST. 

```json
{
    "email": "User email address",
    "language": "Current UI language",
    "loyaltyId": "User loyalty ID"
}
```

Una vez que la API REST valide los datos, debe devolver un código HTTP 200 (correcto), con los siguientes datos JSON:

```json
{
    "promoCode": "24534"
}
```

Si se produce un error en la validación, la API REST debe devolver un código HTTP 409 (conflicto), con el elemento JSON `userMessage`. IEF espera la notificación `userMessage` que devuelve la API REST. Esta notificación se presentará como una cadena para el usuario si se produce un error en la validación.

```json
{
    "version": "1.0.1",
    "status": 409,
    "userMessage": "LoyaltyId ID '1234' is not associated with 'david@contoso.com' email address."
}
```

La configuración del punto de conexión de API REST está fuera del ámbito de este artículo. Hemos creado una muestra de [Azure Functions](../azure-functions/functions-reference.md). Puede acceder al código completo de la función de Azure en [GitHub](https://github.com/azure-ad-b2c/rest-api/tree/master/source-code/azure-function).

## <a name="define-claims"></a>Definición de notificaciones

Una notificación proporciona un almacenamiento temporal de datos durante la ejecución de una directiva de Azure AD B2C. Puede declarar notificaciones dentro de la sección del [esquema de notificaciones](claimsschema.md). 

1. Abra el archivo de extensiones de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Busque el elemento [BuildingBlocks](buildingblocks.md). Si el elemento no existe, agréguelo.
1. Busque el elemento [ClaimsSchema](claimsschema.md). Si el elemento no existe, agréguelo.
1. Agregue las notificaciones siguientes al elemento **ClaimsSchema**.  

```xml
<ClaimType Id="loyaltyId">
  <DisplayName>Your loyalty ID</DisplayName>
  <DataType>string</DataType>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
<ClaimType Id="promoCode">
  <DisplayName>Your promo code</DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
</ClaimType>
  <ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="add-the-restful-api-technical-profile"></a>Incorporación del perfil técnico de la API RESTful 

Un [perfil técnico de RESTful](restful-technical-profile.md) proporciona compatibilidad para interactuar con su propio servicio RESTful. Azure AD B2C envía datos al servicio RESTful en una colección `InputClaims` y recibe los datos en una colección `OutputClaims`. Busque el elemento **ClaimsProviders** y agregue un nuevo proveedor de notificaciones como se indica a continuación:

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Check loyaltyId Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- Set the ServiceUrl with your own REST API endpoint -->
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/ValidateProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="loyaltyId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

En este ejemplo, `userLanguage` se enviará al servicio REST como `lang` desde la carga de JSON. El valor de la notificación `userLanguage` contiene el identificador de idioma del usuario actual. Para obtener más información, consulte el [solucionador de notificaciones](claim-resolver-overview.md).

### <a name="configure-the-restful-api-technical-profile"></a>Configuración del perfil técnico de la API RESTful 

Después de implementar la API REST, configure los metadatos del perfil técnico `REST-ValidateProfile` para que reflejen su propia API REST, incluidos:

- **ServiceUrl**. Establezca la dirección URL del punto de conexión de la API REST.
- **SendClaimsIn**. Especifique cómo se envían las notificaciones de entrada al proveedor de notificaciones RESTful.
- **AuthenticationType**. Establezca el tipo de autenticación realizada por el proveedor de notificaciones RESTful. 
- **AllowInsecureAuthInProduction**. En un entorno de producción, asegúrese de establecer estos metadatos en `true`.
    
Consulte los [metadatos del perfil técnico de RESTful](restful-technical-profile.md#metadata) para obtener más configuraciones.

Los comentarios anteriores `AuthenticationType` y `AllowInsecureAuthInProduction` especifican los cambios que se deben realizar al pasar a un entorno de producción. Para aprender a proteger las API RESTful para la producción, consulte [Proteger la API RESTful](secure-rest-api.md).

## <a name="validate-the-user-input"></a>Validación de la entrada del usuario

Para obtener el número de fidelidad del usuario durante el registro, debe permitir que el usuario escriba estos datos en la pantalla. Agregue la notificación de salida **loyaltyId** a la página de registro. Para hacerlo, agréguela al elemento `OutputClaims` de la sección del perfil técnico de registro existente. Especifique la lista completa de notificaciones de salida para controlar el orden en que se presentan las notificaciones en la pantalla.  

Agregue la referencia del perfil técnico de validación al perfil técnico de registro, que llama a `REST-ValidateProfile`. El nuevo perfil técnico de validación se agregará al principio de la colección `<ValidationTechnicalProfiles>` definida en la directiva base. Este comportamiento significa que, solo después de la validación correcta, Azure AD B2C pasa a crear la cuenta en el directorio.   

1. Busque el elemento **ClaimsProviders**. Agregue un nuevo proveedor de notificaciones como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <DisplayName>Local Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="newPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surName"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
      <DisplayName>Self Asserted</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SelfAsserted-Social">
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
            <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surname"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile"/>
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

## <a name="include-a-claim-in-the-token"></a>Incorporación de una notificación en el token 

Para devolver la notificación del código de promoción a la aplicación de usuario de confianza, agregue una notificación de salida al archivo <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. La notificación de salida permitirá agregar la notificación al token después de un recorrido del usuario correcto y se enviará a la aplicación. Modifique el elemento de perfil técnico en la sección de usuario de confianza para agregar `promoCode` como una notificación de salida.
 
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <OutputClaim ClaimTypeReferenceId="promoCode" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD. Para ello, seleccione el filtro **Directorio y suscripción** que se encuentra en el menú superior y elija el directorio que contiene el inquilino de Azure AD.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Marco de experiencia de identidad**.
1. Seleccione **Cargar directiva personalizada** y cargue los archivos de directiva modificados: *TrustFrameworkExtensions.xml* y *SignUpOrSignin.xml*. 
1. Seleccione la directiva de registro o inicio de sesión que cargó y haga clic en el botón **Ejecutar ahora**.
1. Debe poder registrarse con una dirección de correo electrónico.
1. Haga clic en el vínculo **Registrarse ahora**.
1. En **Your loyalty ID** (Id. de fidelidad), escriba 1234 y haga clic en **Continuar**. Llegados a este punto, debería obtener un mensaje de error de validación.
1. Cambie a otro valor y haga clic en **Continuar**.
1. El token enviado de vuelta a la aplicación contiene la notificación `promoCode`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584295703,
  "nbf": 1584292103,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584292103,
  "auth_time": 1584292103,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "promoCode": "84362"
  ...
}
```

::: zone-end

## <a name="best-practices-and-how-to-troubleshoot"></a>Procedimientos recomendados y solución de problemas

### <a name="using-serverless-cloud-functions"></a>Uso de funciones de nube sin servidor

Las funciones sin servidor, como los desencadenadores HTTP en Azure Functions, proporcionan una manera de crear puntos de conexión de API para usarlos con el conector de API. Puede usar la función de nube sin servidor para, [por ejemplo](code-samples.md#api-connectors), crear la lógica de validación y limitar los registros a dominios específicos. La función de nube sin servidor también puede llamar e invocar otras API web, almacenes de usuarios y otros servicios en la nube para escenarios más complejos.

### <a name="best-practices"></a>Procedimientos recomendados
Asegúrese de que:
* La API sigue los contratos de solicitud y respuesta de la API, tal y como se describe anteriormente. 
* La **URL del punto de conexión** del conector de API apunta al punto de conexión de API correcto.
* La API comprueba explícitamente si hay valores NULL de las notificaciones recibidas.
* La API responde lo más rápido posible para garantizar una experiencia de usuario fluida.
    * Si usa una función sin servidor o un servicio web escalable, use un plan de hospedaje que mantenga la API "activa" o "caliente" en producción. Para Azure Functions, se recomienda usar el [plan Premium](../azure-functions/functions-scale.md).
 
### <a name="use-logging"></a>Uso del registro

En general, resulta útil usar las herramientas de registro que habilita el servicio de API web, como [Application Insights](../azure-functions/functions-monitoring.md), para supervisar la API en busca de códigos de error inesperados, excepciones y rendimiento deficiente.
* Supervise los códigos de estado HTTP que no sean HTTP 200 ni 400.
* Un código de estado HTTP 401 o 403 suele indicar que hay un problema con la autenticación. Compruebe la capa de autenticación de la API y la configuración correspondiente en el conector de API.
* Use niveles más agresivos de registro (por ejemplo, "trace" o "debug") en el desarrollo, si es necesario.
* Supervise la API en busca de tiempos de respuesta prolongados.

## <a name="next-steps"></a>Pasos siguientes

::: zone pivot="b2c-user-flow"

- Comience a trabajar con nuestros [ejemplos](code-samples.md#api-connectors).
- [Protección de un conector de API](secure-rest-api.md)

::: zone-end

::: zone pivot="b2c-custom-policy"

- [Tutorial: Integración de intercambios de notificaciones de API REST en los recorridos de usuario de Azure AD B2C como un paso de orquestación](custom-policy-rest-api-claims-exchange.md)
- [Protección de un conector de API](secure-rest-api.md)
- [Referencia: Perfil técnico de RESTful](restful-technical-profile.md)

::: zone-end