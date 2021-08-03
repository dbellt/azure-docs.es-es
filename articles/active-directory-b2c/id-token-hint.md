---
title: Definición de un perfil técnico de sugerencias de token de id. en una directiva personalizada
titleSuffix: Azure AD B2C
description: Defina un perfil técnico de sugerencias de token de id. en una directiva personalizada de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/30/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6477283b3eb96579b943baf0aa34c2737bf43a58
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110059612"
---
# <a name="define-an-id-token-hint-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de sugerencias de token de id. en una directiva personalizada de Azure Active Directory B2C

Azure AD B2C permite a las aplicaciones de usuario de confianza enviar un token JWT de entrada como parte de la solicitud de autorización de OAuth2. El token JWT se puede emitir mediante una aplicación de usuario de confianza o un proveedor de identidades, y puede pasar una sugerencia sobre el usuario o la solicitud de autorización. Azure AD B2C valida la firma, el nombre del emisor y la audiencia del token, y extrae la notificación del token de entrada.

## <a name="use-cases"></a>Casos de uso

Puede usar esta solución para enviar datos a Azure AD B2C, encapsulados en un único token JWT. La [solución de registro con una invitación por correo electrónico](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md), donde el administrador del sistema puede enviar una invitación firmada a los usuarios, se basa en id_token_hint. Solo los usuarios con acceso al correo electrónico de la invitación pueden crear la cuenta en el directorio.

## <a name="token-signing-approach"></a>Enfoque de firma de token

Con id_token_hint, el emisor del token (una aplicación de usuario de confianza o un proveedor de identidades) crea el token y luego lo firma mediante una clave de firma para probar que el token proviene de un origen de confianza. La clave de firma puede ser simétrica o asimétrica. La criptografía simétrica, o criptografía de clave privada, usa un secreto compartido para firmar y validar la firma. La criptografía asimétrica, o criptografía de clave pública, es un sistema criptográfico que usa una clave privada y una clave pública. Solo el emisor del token conoce la clave privada que se usa para firmar el token. La clave pública se comparte con la directiva de Azure AD B2C para validar la firma del token.

## <a name="token-format"></a>Formato de tokens

id_token_hint debe ser un token JWT válido. En la siguiente tabla se enumeran las notificaciones que son obligatorias. Las demás notificaciones son opcionales.

| Nombre | Notificación | Valor de ejemplo | Descripción |
| ---- | ----- | ------------- | ----------- |
| Público | `aud` | `a489fc44-3cc0-4a78-92f6-e413cd853eae` | Identifica al destinatario previsto del token. El público es una cadena arbitraria definida por el emisor de tokens. Azure AD B2C comprueba este valor y rechaza el token si no coincide.  |
| Emisor | `iss` |`https://localhost` | Identifica el servicio de token de seguridad (emisor del token). El emisor es un URI arbitrario definido por el emisor de tokens. Azure AD B2C comprueba este valor y rechaza el token si no coincide.  |
| Fecha de expiración | `exp` | `1600087315` | La hora en que el token deja de ser válido, que se representa en tiempo de época. Azure AD B2C comprueba este valor y rechaza el token si ha expirado.|
| No antes de | `nbf` | `1599482515` | Hora a la que el token pasa a ser válido, representada en tiempo de época. Suele ser la misma hora a la que se emitió el token. Azure AD B2C comprueba este valor y rechaza el token si su duración no es válida. |

 El siguiente es un ejemplo de un token de id. válido:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}.{
  "displayName": " John Smith",
  "userId": "john.s@contoso.com",
  "nbf": 1599482515,
  "exp": 1600087315,
  "iss": "https://localhost",
  "aud": "a489fc44-3cc0-4a78-92f6-e413cd853eae"
}
```

## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento **Protocol** tiene que establecerse en `None`. Por ejemplo, el protocolo para el perfil técnico **IdTokenHint_ExtractClaims** es `None`:

```xml
<TechnicalProfile Id="IdTokenHint_ExtractClaims">
  <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
  <Protocol Name="None" />
  ...
```

Se llama al perfil técnico desde un paso de orquestación con el tipo `GetClaims`.

```xml
<OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
``` 

## <a name="output-claims"></a>Notificaciones de salida

El elemento **OutputClaims** contiene una lista de las notificaciones que se van a extraer del token JWT. Es posible que tenga que asignar el nombre de la notificación definida en la directiva al nombre definido en el token JWT. También puede incluir notificaciones que el token JWT no devuelve, siempre que establezca el atributo `DefaultValue`.

## <a name="metadata"></a>Metadatos

Los metadatos siguientes son relevantes cuando se usa una clave simétrica. 

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| issuer | Sí | Identifica el servicio de token de seguridad (emisor del token). Este valor debe ser idéntico a la notificación `iss` dentro de la notificación del token JWT. | 
| IdTokenAudience | Sí | Identifica al destinatario previsto del token. Debe coincidir con la notificación `aud` dentro de la notificación del token JWT. | 

Los metadatos siguientes son pertinentes cuando se usa una clave asimétrica. 

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| METADATOS| Sí | Una dirección URL que apunta a un documento de configuración del emisor del token, que es un punto de conexión de configuración conocido de OpenID.   |
| issuer | No | Identifica el servicio de token de seguridad (emisor del token). Este valor se puede usar para sobrescribir el valor configurado en los metadatos y debe ser idéntico a la notificación `iss` dentro de la notificación del token JWT. |  
| IdTokenAudience | No | Identifica al destinatario previsto del token. Debe coincidir con la notificación `aud` dentro de la notificación del token JWT. |  

[!INCLUDE [active-directory-b2c-https-cipher-tls-requirements](../../includes/active-directory-b2c-https-cipher-tls-requirements.md)]

## <a name="cryptographic-keys"></a>Claves criptográficas

Cuando se usa una clave simétrica, el elemento **CryptographicKeys** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| client_secret | Sí | La clave criptográfica que se usa para validar la firma del token JWT.|


## <a name="how-to-guide"></a>Guía paso a paso

### <a name="issue-a-token-with-symmetric-keys"></a>Emisión de un token con claves simétricas

#### <a name="step-1-create-a-shared-key"></a>Paso 1. Creación de una clave compartida 

Cree una clave que se pueda usar para firmar el token. Por ejemplo, use el siguiente código de PowerShell para generar una clave.

```powershell
$bytes = New-Object Byte[] 32
$rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rand.GetBytes($bytes)
$rand.Dispose()
$newClientSecret = [System.Convert]::ToBase64String($bytes)
$newClientSecret
```

Este código crea una cadena de secreto como `VK62QTn0m1hMcn0DQ3RPYDAr6yIiSvYgdRwjZtU5QhI=`.

#### <a name="step-2-add-the-signing-key-to-azure-ad-b2c"></a>Paso 2. Adición de la clave de firma a Azure AD B2C

Debe crearse la misma clave que usa el emisor del token, en las claves de la directiva de Azure AD B2C.  

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En la página de información general, en **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva**. 
1. Seleccione **Manual**.
1. En **Nombre**, use `IdTokenHintKey`.  
   Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
1. En el cuadro **Secreto**, escriba la clave de inicio de sesión que ha generado anteriormente.
1. En **Uso de claves**, use **Cifrado**.
1. Seleccione **Crear**.
1. Confirme que ha creado la clave `B2C_1A_IdTokenHintKey`.


#### <a name="step-3-add-the-id-token-hint-technical-profile"></a>Paso 3. Adición del perfil técnico de sugerencias de token de id.

El siguiente perfil técnico valida el token y extrae las notificaciones. 

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <Item Key="IdTokenAudience">a489fc44-3cc0-4a78-92f6-e413cd853eae</Item>
        <Item Key="issuer">https://localhost</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_IdTokenHintKey" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-4-prepare-your-policy"></a>Paso 4. Preparar la directiva

Complete el paso [Configuración de la directiva](#configure-your-policy).

#### <a name="step-5-prepare-the-code"></a>Paso 5. Preparación del código  

El [ejemplo de GitHub](https://github.com/azure-ad-b2c/id_token_hint/tree/master/dotnet_core_symmetric_key) es una aplicación web de ASP.NET y una aplicación de consola que genera un token de id. que se firma con una clave simétrica. 


### <a name="issue-a-token-with-asymmetric-keys"></a>Emisión de un token con claves asimétricas

Con una clave asimétrica, el token se firma mediante certificados RSA. Esta aplicación hospeda un punto de conexión de metadatos de Open ID Connect y un punto de conexión de claves web de JSON (JWK) que usa Azure AD B2C para validar la firma del token de id.

El emisor del token debe proporcionar los puntos de conexión siguientes:

* `/.well-known/openid-configuration`: un punto de conexión de configuración conocido con la información pertinente sobre el token, como el nombre del emisor del token y el vínculo al punto de conexión JWK. 
* `/.well-known/keys`: el punto de conexión de la clave web de JSON (JWK) con la clave pública que se usa para firmar la clave (con la parte de la clave privada del certificado).

Vea el ejemplo de controlador MVC de .NET [TokenMetadataController.cs](https://github.com/azure-ad-b2c/id-token-builder/blob/master/source-code/B2CIdTokenBuilder/Controllers/TokenMetadataController.cs).

#### <a name="step-1-prepare-a-self-signed-certificate"></a>Paso 1. Preparación de un certificado autofirmado

Si aún no tiene un certificado, puede usar un certificado autofirmado para esta guía paso a paso. En Windows, puede usar el cmdlet [New-SelfSignedCertificate](/powershell/module/pki/new-selfsignedcertificate) de PowerShell para generar un certificado.

Ejecute este comando de PowerShell para generar un certificado autofirmado. Modifique el argumento `-Subject` según corresponda para su aplicación y el nombre del inquilino de Azure AD B2C. También puede ajustar la fecha de `-NotAfter` para especificar una expiración diferente para el certificado.

```PowerShell
New-SelfSignedCertificate `
    -KeyExportPolicy Exportable `
    -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -KeyUsage DigitalSignature `
    -NotAfter (Get-Date).AddMonths(12) `
    -CertStoreLocation "Cert:\CurrentUser\My"
```


#### <a name="step-2-add-the-id-token-hint-technical-profile"></a>Paso 2. Adición del perfil técnico de sugerencias de token de id. 

El siguiente perfil técnico valida el token y extrae las notificaciones. Cambie el URI de metadatos al punto de conexión de configuración conocido del emisor del token.  

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <!-- Replace with your endpoint location -->
        <Item Key="METADATA">https://your-app.azurewebsites.net/.well-known/openid-configuration</Item>
        <Item Key="IdTokenAudience">your_optional_audience</Item>
        <!-- <Item Key="issuer">your_optional_token_issuer_override</Item> -->
      </Metadata>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-3-prepare-your-policy"></a>Paso 3. Preparar la directiva

Complete el paso [Configuración de la directiva](#configure-your-policy).

#### <a name="step-4-prepare-the-code"></a>Paso 4. Preparación del código 

En este [ejemplo de GitHub](https://github.com/azure-ad-b2c/id-token-builder), la aplicación web ASP.NET genera tokens de id. y hospeda los puntos de conexión de metadatos necesarios para usar el parámetro "id_token_hint" en Azure AD B2C.


### <a name="configure-your-policy"></a>Configuración de la directiva

En ambos enfoques, simétrico y asimétrico, se llama al perfil técnico `id_token_hint` desde un paso de orquestación con el tipo `GetClaims` y es necesario especificar las notificaciones de entrada de la directiva de usuario de confianza.

1. Agregue el perfil técnico IdTokenHint_ExtractClaims a la directiva de extensión.
1. Agregue el siguiente paso de orquestación al recorrido del usuario como primer elemento.  

    ```xml
    <OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
    ``` 
1. En la directiva de usuario de confianza, repita las mismas notificaciones de entrada que configuró en el perfil técnico IdTokenHint_ExtractClaims. Por ejemplo:
    ```xml
   <RelyingParty>
     <DefaultUserJourney ReferenceId="SignUp" />
     <TechnicalProfile Id="PolicyProfile">
       <DisplayName>PolicyProfile</DisplayName>
       <Protocol Name="OpenIdConnect" />
       <InputClaims>
         <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
        </InputClaims>
       <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
       </OutputClaims>
       <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```

En función de los requisitos de su empresa, es posible que deba agregar validaciones de tokens (por ejemplo, comprobar el formato de la dirección de correo electrónico). Para ello, agregue pasos de orquestación que invoquen un [perfil técnico de transformación de notificaciones](claims-transformation-technical-profile.md). Agregue también un [perfil técnico autoafirmado](self-asserted-technical-profile.md) para presentar un mensaje de error. 

### <a name="create-and-sign-a-token"></a>Creación y firma de un token

Los ejemplos de GitHub muestran cómo crear el tipo de token JWT, que posteriormente se enviará como un parámetro de cadena de consulta `id_token_hint`. A continuación, se proporciona un ejemplo de una solicitud de autorización con el parámetro id_token_hint
 
```html
https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/B2C_1A_signup_signin/oauth2/v2.0/authorize?client_id=63ba0d17-c4ba-47fd-89e9-31b3c2734339&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login&id_token_hint=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkaXNwbGF5TmFtZSI6IiBKb2huIFNtaXRoIiwidXNlcklkIjoiam9obi5zQGNvbnRvc28uY29tIiwibmJmIjoxNTk5NDgyNTE1LCJleHAiOjE2MDAwODczMTUsImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwiYXVkIjoiYTQ4OWZjNDQtM2NjMC00YTc4LTkyZjYtZTQxM2NkODUzZWFlIn0.nPmLXydI83PQCk5lRBYUZRu_aX58pL1khahHyQuupig
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte la solución de [registro con invitación por correo electrónico](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md) en el repositorio de GitHub de la comunidad de Azure AD B2C.
