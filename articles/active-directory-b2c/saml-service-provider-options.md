---
title: Configuración de las opciones del proveedor de servicios SAML
title-suffix: Azure Active Directory B2C
description: Configuración de las opciones del proveedor de servicios SAML de Azure Active Directory B2C
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/05/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: fea39388b6b4387dfc4fe95d1cdfb3e523a8089c
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382443"
---
# <a name="options-for-registering-a-saml-application-in-azure-ad-b2c"></a>Opciones para registrar una aplicación SAML en Azure AD B2C

En este artículo se describen las opciones de configuración que están disponibles al conectar Azure Active Directory (Azure AD B2C) con la aplicación SAML.

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"


## <a name="saml-response-signature"></a>Firma de respuesta en SAML

Puede especificar un certificado que se utilizará para firmar los mensajes SAML. El mensaje es el elemento `<samlp:Response>` dentro de la respuesta SAML enviada a la aplicación.

Si aún no tiene una clave de directiva, [créela](saml-service-provider.md#create-a-policy-key). A continuación, configure el elemento de metadatos `SamlMessageSigning` en el perfil técnico del emisor del token de SAML. `StorageReferenceId` debe hacer referencia al nombre de la clave de la directiva.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
        ...
      <CryptographicKeys>
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SamlMessageCert"/>
        ...
      </CryptographicKeys>
    ...
    </TechnicalProfile>
```

### <a name="saml-response-signature-algorithm"></a>Algoritmo de firma de respuesta SAML

Puede configurar el algoritmo de firma utilizado para firmar la aserción SAML. Los valores posibles son `Sha256`, `Sha384`, `Sha512` o `Sha1`. Asegúrese de que el perfil técnico y la aplicación usan el mismo algoritmo de firma. Use solo el algoritmo que admite el certificado.

Configure el algoritmo de firma mediante la clave de metadatos `XmlSignatureAlgorithm` en el elemento de metadatos del usuario de confianza.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="XmlSignatureAlgorithm">Sha256</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

## <a name="saml-assertions-signature"></a>Firma de aserciones en SAML

Cuando una aplicación espere que se firme la sección de aserciones de SAML, asegúrese de que el proveedor de servicios de SAML haya establecido `WantAssertionsSigned` en `true`. Si lo ha establecido en `false`, o no existe, la sección de aserciones no se firmará. En el ejemplo siguiente se muestran los metadatos del proveedor de servicios SAML con `WantAssertionsSigned` establecido en `true`.

```xml
<EntityDescriptor ID="id123456789" entityID="https://samltestapp2.azurewebsites.net" validUntil="2099-12-31T23:59:59Z" xmlns="urn:oasis:names:tc:SAML:2.0:metadata">
  <SPSSODescriptor  WantAssertionsSigned="true" AuthnRequestsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
  ...
  </SPSSODescriptor>
</EntityDescriptor>
```  

### <a name="saml-assertions-signature-certificate"></a>Certificado de firma de aserciones de SAML

La directiva debe especificar que se use un certificado para firmar la sección de aserciones de SAML de la respuesta de SAML. Si aún no tiene una clave de directiva, [créela](saml-service-provider.md#create-a-policy-key). A continuación, configure el elemento de metadatos `SamlAssertionSigning` en el perfil técnico del emisor del token de SAML. `StorageReferenceId` debe hacer referencia al nombre de la clave de la directiva.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
        ...
      <CryptographicKeys>
        <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SamlMessageCert"/>
        ...
      </CryptographicKeys>
    ...
    </TechnicalProfile>
```

## <a name="saml-assertions-encryption"></a>Cifrado de aserciones de SAML

Si la aplicación espera que las aserciones SAML estén en un formato cifrado, debe asegurarse de que el cifrado esté habilitado en la directiva de Azure AD B2C.

Azure AD B2C utiliza el certificado de clave pública del proveedor de servicios para cifrar la aserción SAML. La clave pública debe existir en el punto de conexión de metadatos de la aplicación SAML con la etiqueta KeyDescriptor 'use' establecida en 'Encryption', como se muestra en el ejemplo siguiente:

```xml
<KeyDescriptor use="encryption">
  <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>valid certificate</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

Para habilitar Azure AD B2C para enviar aserciones cifradas, establezca el elemento de los metadatos **WantsEncryptedAssertion** en `true` en el [perfil técnico del usuario de confianza](relyingparty.md#technicalprofile). También puede configurar el algoritmo utilizado para cifrar la aserción de SAML.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="WantsEncryptedAssertions">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

### <a name="encryption-method"></a>Encryption method

Para configurar el método que se usa para cifrar los datos de la aserción SAML, establezca la clave de metadatos `DataEncryptionMethod` en el usuario de confianza. Los valores posibles son: `Aes256` (predeterminado), `Aes192`, `Sha512` o `Aes128`. Los metadatos controlan el valor del elemento `<EncryptedData>` en la respuesta de SAML.

Para configurar el método que se usa para cifrar la copia de la clave, usada para cifrar los datos de la aserción SAML, establezca la clave de metadatos `KeyEncryptionMethod` en el usuario de confianza. Los valores posibles son `Rsa15` (predeterminado), el algoritmo del estándar de criptografía de clave pública (PKCS) versión 1.5, y `RsaOaep`, el algoritmo de cifrado de relleno óptimo de cifrado asimétrico (OAEP) de RSA.  Los metadatos controlan el valor del elemento `<EncryptedKey>` en la respuesta de SAML.

En el ejemplo siguiente se muestra la sección `EncryptedAssertion` de una aserción SAML. El método de cifrado de datos es `Aes128` y el método de cifrado de clave es `Rsa15`.

```xml
<saml:EncryptedAssertion>
  <xenc:EncryptedData xmlns:xenc="http://www.w3.org/2001/04/xmlenc#"
    xmlns:dsig="http://www.w3.org/2000/09/xmldsig#" Type="http://www.w3.org/2001/04/xmlenc#Element">
    <xenc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes128-cbc" />
    <dsig:KeyInfo>
      <xenc:EncryptedKey>
        <xenc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#rsa-1_5" />
        <xenc:CipherData>
          <xenc:CipherValue>...</xenc:CipherValue>
        </xenc:CipherData>
      </xenc:EncryptedKey>
    </dsig:KeyInfo>
    <xenc:CipherData>
      <xenc:CipherValue>...</xenc:CipherValue>
    </xenc:CipherData>
  </xenc:EncryptedData>
</saml:EncryptedAssertion>
```

Puede cambiar el formato de las aserciones cifradas. Para configurar el formato de cifrado, establezca la clave de metadatos `UseDetachedKeys` en el usuario de confianza. Valores posibles: `true` o `false` (valor predeterminado). Cuando el valor se establece en `true`, las claves desasociadas agregan la aserción cifrada como un elemento secundario de `EncrytedAssertion` y no de `EncryptedData`.

Para configurar el método y el formato de cifrado, use las claves de metadatos en el [perfil técnico del usuario de confianza](relyingparty.md#technicalprofile):

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="DataEncryptionMethod">Aes128</Item>
      <Item Key="KeyEncryptionMethod">Rsa15</Item>
      <Item Key="UseDetachedKeys">false</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

## <a name="identity-provider-initiated-flow"></a>Flujo iniciado por el proveedor de identidades

Si la aplicación espera recibir una aserción SAML sin enviar primero una solicitud de autenticación SAML al proveedor de identidades, debe configurar Azure AD B2C para el flujo Iniciado por el proveedor de identidades.

En el flujo iniciado por el proveedor de identidades, el proveedor de identidades (Azure AD B2C) inicia el proceso de inicio de sesión y envía una respuesta SAML no solicitada al proveedor de servicios (la aplicación de usuario de confianza).

Actualmente no se admiten escenarios en los que el proveedor de identidades que inicia el proceso sea un proveedor de identidades externo federado con Azure AD B2C; por ejemplo [AD-FS](identity-provider-adfs.md) o [Salesforce](identity-provider-salesforce-saml.md). Solo se admite en la autenticación de cuentas locales de Azure AD B2C.

Para habilitar el flujo iniciado por el proveedor de identidades, establezca el elemento de metadatos **IdpInitiatedProfileEnabled** en `true`, en el [perfil técnico del usuario de confianza](relyingparty.md#technicalprofile).

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="IdpInitiatedProfileEnabled">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

Para iniciar sesión o registrar un usuario mediante el flujo iniciado por el proveedor de identidades, use la siguiente dirección URL:

```
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/generic/login?EntityId=app-identifier-uri 
```

Reemplace los siguientes valores:

* **tenant-name** por el nombre de inquilino
* **policy-name** por el nombre de la directiva de usuario de confianza de SAML
* **app-identifier-uri** con `identifierUris` en el archivo de metadatos, por ejemplo `https://contoso.onmicrosoft.com/app-name`

### <a name="sample-policy"></a>Directiva de ejemplo

Se proporciona una directiva de ejemplo completa que se puede usar para realizar pruebas con la aplicación SAML de prueba.

1. Descargue la [directiva de ejemplo de inicio de sesión iniciada por el proveedor de servicios SAML](https://github.com/azure-ad-b2c/saml-sp/tree/master/policy/SAML-SP-Initiated).
1. Actualice `TenantId` para que coincida con el nombre de su inquilino; por ejemplo,  *contoso.b2clogin.com*.
1. Mantenga el nombre de la directiva *B2C_1A_signup_signin_saml*.

## <a name="saml-response-lifetime"></a>Duración de respuesta SAML

Puede configurar el período de tiempo durante el que la respuesta SAML sigue siendo válida. Establezca la duración mediante el elemento de metadatos `TokenLifeTimeInSeconds` en el perfil técnico del emisor del token de SAML. Este valor es el número de segundos que pueden transcurrir desde la marca de tiempo `NotBefore` calculada en el momento de emitir el token. La duración predeterminada es de 300 segundos (5 minutos).

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="TokenLifeTimeInSeconds">400</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="saml-response-valid-from-skew"></a>Respuesta SAML válida con desfase

Puede configurar el desfase horario que se aplica a la marca de tiempo `NotBefore` de la respuesta SAML. Esta configuración garantiza que, si las horas entre dos plataformas no están sincronizadas, la aserción SAML seguirá considerándose válida cuando se encuentre dentro de este desfase horario.

Establezca el desfase horario mediante el elemento de metadatos `TokenNotBeforeSkewInSeconds` en el perfil técnico del emisor del token de SAML. El valor de desfase se proporciona en segundos, con un valor predeterminado de 0. El valor máximo es 3600 (una hora).

Por ejemplo, si `TokenNotBeforeSkewInSeconds` se establece en `120` segundos:

- El token se emite a las 13:05:10 UTC
- El token es válido a partir de las 13:03:10 UTC

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="TokenNotBeforeSkewInSeconds">120</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="remove-milliseconds-from-date-and-time"></a>Eliminación de milisegundos de la fecha y hora

Puede especificar si se quitarán los milisegundos de los valores datetime de la respuesta SAML (es decir, de IssueInstant, NotBefore, NotOnOrAfter y AuthnInstant). Para quitar los milisegundos, establezca la clave de metadatos `RemoveMillisecondsFromDateTime
` en el usuario de confianza. Valores posibles: `false` (predeterminado) o `true`.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="RemoveMillisecondsFromDateTime">true</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="azure-ad-b2c-issuer-id"></a>Identificador de emisor de Azure AD B2C

Si tiene varias aplicaciones SAML que dependen de distintos valores de `entityID`, puede reemplazar el valor de `issueruri` del archivo de usuario de confianza. Para reemplazar el identificador URI del emisor, copie el perfil técnico con el identificador "Saml2AssertionIssuer" del archivo base y reemplace el valor `issueruri`.

> [!TIP]
> Copie la sección `<ClaimsProviders>` de la base y conserve estos elementos en el proveedor de notificaciones: `<DisplayName>Token Issuer</DisplayName>`, `<TechnicalProfile Id="Saml2AssertionIssuer">` y `<DisplayName>Token Issuer</DisplayName>`.
 
Ejemplo:

```xml
   <ClaimsProviders>   
    <ClaimsProvider>
      <DisplayName>Token Issuer</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Saml2AssertionIssuer">
          <DisplayName>Token Issuer</DisplayName>
          <Metadata>
            <Item Key="IssuerUri">customURI</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
  </ClaimsProviders>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpInSAML" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
     …
```

## <a name="session-management"></a>Administración de sesiones

Puede administrar la sesión entre Azure AD B2C y la aplicación de usuario de confianza SAML mediante los elementos `UseTechnicalProfileForSessionManagement` y [SamlSSOSessionProvider](custom-policy-reference-sso.md#samlssosessionprovider).

## <a name="force-users-to-reauthenticate"></a>Obligatoriedad de que los usuarios vuelvan a autenticarse 

Para obligar a los usuarios a volver a autenticarse, la aplicación puede incluir el atributo `ForceAuthn` en la solicitud de autenticación de SAML. El atributo `ForceAuthn` es un valor booleano. Cuando se establece en true, la sesión de los usuarios se invalidará en Azure AD B2C y el usuario tendrá que volver a autenticarse. La siguiente solicitud de autenticación de SAML muestra cómo establecer el atributo `ForceAuthn` en true. 


```xml
<samlp:AuthnRequest 
       Destination="https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1A_SAML2_signup_signin/samlp/sso/login"
       ForceAuthn="true" ...>
    ...
</samlp:AuthnRequest>
```

## <a name="sign-the-azure-ad-b2c-idp-saml-metadata"></a>Firma de los metadatos de SAML del IdP de Azure AD B2C

Puede indicar Azure AD B2C para firmar su documento de metadatos de IdP de SAML, si lo requiere la aplicación. Si aún no tiene una clave de directiva, [créela](saml-service-provider.md#create-a-policy-key). A continuación, configure el elemento de metadatos `MetadataSigning` en el perfil técnico del emisor del token de SAML. `StorageReferenceId` debe hacer referencia al nombre de la clave de la directiva.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
        ...
      <CryptographicKeys>
        <Key Id="MetadataSigning" StorageReferenceId="B2C_1A_SamlMetadataCert"/>
        ...
      </CryptographicKeys>
    ...
    </TechnicalProfile>
```

## <a name="debug-the-saml-protocol"></a>Depuración del protocolo SAML

Para ayudar a configurar y depurar la integración con el proveedor de servicios, puede usar una extensión del navegador para el protocolo SAML; por ejemplo, la [extensión SAML DevTools](https://chrome.google.com/webstore/detail/saml-devtools-extension/jndllhgbinhiiddokbeoeepbppdnhhio) para Chrome, [SAML-Tracer](https://addons.mozilla.org/es/firefox/addon/saml-tracer/) para Firefox o las [herramientas para desarrolladores de Edge o IE](https://techcommunity.microsoft.com/t5/microsoft-sharepoint-blog/gathering-a-saml-token-using-edge-or-ie-developer-tools/ba-p/320957).

Con estas herramientas, puede comprobar la integración entre la aplicación y Azure AD B2C. Por ejemplo:

* Puede comprobar si la solicitud SAML contiene una firma y determinar qué algoritmo se usa para iniciar sesión en la solicitud de autorización.
* Puede comprobar si Azure AD B2C devuelve un mensaje de error.
* Puede comprobar si la sección de aserción está cifrada.

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar más información sobre el [protocolo SAML en el sitio web de OASIS](https://www.oasis-open.org/).
- Obtenga la aplicación web de prueba de SAML en el [repositorio de la comunidad de Azure AD B2C de GitHub](https://github.com/azure-ad-b2c/saml-sp-tester).

<!-- LINKS - External -->
[samltest]: https://aka.ms/samltestapp

::: zone-end
