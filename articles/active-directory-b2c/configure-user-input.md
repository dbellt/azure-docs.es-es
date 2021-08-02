---
title: Adición de atributos de usuario y personalización de entradas de usuario
titleSuffix: Azure AD B2C
description: Aprenda a personalizar entradas de usuario y a agregar atributos de usuario en el proceso de registro o inicio de sesión en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/03/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: eb700a4432082f75cf1ddf1ce007cee801597948
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111409458"
---
#  <a name="add-user-attributes-and-customize-user-input-in-azure-active-directory-b2c"></a>Adición de atributos de usuario y personalización de entradas de usuario en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

En este artículo, recopilará un nuevo atributo durante el recorrido de registro en Azure Active Directory B2C (Azure AD B2C). Obtendrá la ciudad de los usuarios, la configurará como una lista desplegable y definirá si es necesario proporcionarla.

> [!IMPORTANT]
> En este ejemplo se usa una notificación "city" integrada. En su lugar, puede elegir uno de los [atributos integrados de Azure AD B2C](user-profile-attributes.md) admitidos o un atributo personalizado. Para usar un atributo personalizado, [habilite los atributos personalizados](user-flow-custom-attributes.md). Para usar otro atributo integrado o personalizado, reemplace "city" por el atributo que prefiera; por ejemplo, el atributo integrado *jobTitle* o uno personalizado como *extension_loyaltyId*.  

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="add-user-attributes-your-user-flow"></a>Adición de atributos de usuario al flujo de usuario

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Atributos de usuario** y, después, seleccione el atributo que desee (por ejemplo, "City"). 
1. Seleccione **Guardar**.

## <a name="provide-optional-claims-to-your-app"></a>Proporcionar notificaciones opcionales a la aplicación

Las notificaciones de aplicación son valores que se devuelven a la aplicación. Actualice el flujo de usuario para que contenga las notificaciones que necesite.

1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Notificaciones de aplicación**.
1. Seleccione los atributos que quiera enviar de nuevo a la aplicación (por ejemplo, "City").
1. Seleccione **Guardar**.
 
## <a name="configure-user-attributes-input-type"></a>Configuración del tipo de entrada de los atributos de usuario

1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Diseños de página**.
1. Seleccione **Página de suscripción de cuenta local**.
1. En **Atributos de usuario**, seleccione **City** (Ciudad).
    1. En la lista desplegable **Tipo de entrada de usuario**, seleccione **DropdownSingleSelect**. Opcional: Use los botones "Subir/Bajar" para organizar el orden de texto en la página de registro.
    1. En la lista desplegable **Opcional**, seleccione **No**.
1. Seleccione **Guardar**. 

### <a name="provide-a-list-of-values-by-using-localized-collections"></a>Entrega de una lista de valores mediante colecciones localizadas

Para proporcionar una lista establecida de valores para el atributo de ciudad: 

1. [Habilite la personalización del idioma en el flujo del usuario](language-customization.md#support-requested-languages-for-ui_locales)
1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. En la página **Idiomas** del flujo de usuario, seleccione un idioma que desee personalizar.
1. En **Page-level-resources files** (Archivos de recursos de nivel de página), seleccione **Página de registro de cuenta local**.
1. Seleccione **Descargar valores predeterminados** (o **Descargar invalidaciones** si ha editado anteriormente este idioma).
1. Cree un atributo `LocalizedCollections`.

`LocalizedCollections` es una matriz de pares de `Name` y `Value`. El orden de los elementos será el orden en el que se muestran. 

* `ElementId` es el atributo de usuario para el que este atributo `LocalizedCollections` es una respuesta.
* `Name` es el valor que se muestra al usuario.
* `Value` es lo que se devuelve en la notificación cuando se selecciona esta opción.

```json
{
  "LocalizedStrings": [...],
  "LocalizedCollections": [
    {
      "ElementType": "ClaimType",
      "ElementId": "city",
      "TargetCollection": "Restriction",
      "Override": true,
      "Items": [
        {
          "Name": "Berlin",
          "Value": "Berlin"
        },
        {
          "Name": "London",
          "Value": "London"
        },
        {
          "Name": "Seattle",
          "Value": "Seattle"
        }
      ]
    }
  ]
}
```

#### <a name="upload-your-changes"></a>Carga de los cambios

1. Una vez completados los cambios en el archivo JSON, vuelva al inquilino B2C.
1. Haga clic en **Flujos de usuario** y seleccione la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Idiomas**.
1. Seleccione el idioma al que quiere traducir.
1. Seleccione **Página de registro de cuenta local**.
1. Seleccione el icono de carpeta y el archivo JSON para cargar. Este cambio se guarda en el flujo de usuario automáticamente.

## <a name="test-your-user-flow"></a>Prueba del flujo de usuario

1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Haga clic en **Ejecutar flujo de usuario**.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="overview"></a>Información general

Para recopilar los datos iniciales de los usuarios, utilice el recorrido del usuario de registro o inicio de sesión. Las reclamaciones posteriores se pueden recopilar mediante el recorrido de edición de perfil del usuario. Siempre que Azure AD B2C recopila información del usuario de forma directa e interactiva, utiliza el [perfil técnico autoafirmado](self-asserted-technical-profile.md). En este ejemplo:

1. Define una notificación de "ciudad". 
1. Pide al usuario que especifique su ciudad.
1. Conserva la ciudad en el perfil de usuario en el directorio de Azure AD B2C.
1. Lee la notificación de ciudad en el directorio de Azure AD B2C en cada inicio de sesión.
1. Devuelve la ciudad a la aplicación de usuario de confianza después de iniciar sesión o registrarse.  

## <a name="define-a-claim"></a>Definición de una notificación

Una notificación proporciona un almacenamiento temporal de datos durante la ejecución de una directiva de Azure AD B2C. El [esquema de notificaciones](claimsschema.md) es el lugar en el que se declaran las notificaciones. Los elementos siguientes se usan para definir la notificación:

- **DisplayName**: cadena que define la etiqueta que se muestra al usuario.
- [DataType](claimsschema.md#datatype): tipo de notificación.
- **UserHelpText**: ayuda al usuario a entender los requisitos.
- [UserInputType](claimsschema.md#userinputtype): tipo de control de entrada, como cuadro de texto, botón de selección, lista desplegable o selecciones múltiples.

Abra el archivo de extensiones de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Busque el elemento [BuildingBlocks](buildingblocks.md). Si el elemento no existe, agréguelo.
1. Busque el elemento [ClaimsSchema](claimsschema.md). Si el elemento no existe, agréguelo.
1. Agregue la notificación de ciudad al elemento **ClaimsSchema**.  

```xml
<!-- 
<BuildingBlocks>
  <ClaimsSchema> -->
    <ClaimType Id="city">
      <DisplayName>City where you work</DisplayName>
      <DataType>string</DataType>
      <UserInputType>DropdownSingleSelect</UserInputType>
      <Restriction>
        <Enumeration Text="Berlin" Value="berlin" />
        <Enumeration Text="London" Value="bondon" />
        <Enumeration Text="Seattle" Value="seattle" />
      </Restriction>
    </ClaimType>
  <!-- 
  </ClaimsSchema>
</BuildingBlocks>-->
```

Incluya el atributo [SelectByDefault](claimsschema.md#enumeration) en un elemento `Enumeration` para que se seleccione de forma predeterminada cuando se cargue la página por primera vez. Por ejemplo, para seleccionar previamente el elemento *London*, cambie el elemento `Enumeration` como el ejemplo siguiente:

```xml
<Restriction>
  <Enumeration Text="Berlin" Value="berlin" />
  <Enumeration Text="London" Value="bondon" SelectByDefault="true" />
  <Enumeration Text="Seattle" Value="seattle" />
</Restriction>
```

## <a name="add-a-claim-to-the-user-interface"></a>Incorporación de una notificación a la interfaz de usuario

Los siguientes perfiles técnicos son [autoafirmados](self-asserted-technical-profile.md) y se invocan cuando se espera que un usuario proporcione una entrada:

- **LocalAccountSignUpWithLogonEmail**: flujo de registro de la cuenta local.
- **SelfAsserted-Social**: primer inicio de sesión de un usuario de una cuenta federada.
- **SelfAsserted-ProfileUpdate**: flujo de perfil de edición.

Para recopilar la notificación de ciudad durante el registro, se debe agregar como notificación de salida al perfil técnico `LocalAccountSignUpWithLogonEmail`. Reemplace este perfil técnico en el archivo de extensión. Especifique la lista completa de notificaciones de salida para controlar el orden en que se presentan las notificaciones en la pantalla. Busque el elemento **ClaimsProviders**. Agregue un nuevo elemento ClaimsProviders tal como se muestra a continuación:

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <!--Local account sign-up page-->
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <OutputClaims>
       <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true" />
       <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="displayName" />
       <OutputClaim ClaimTypeReferenceId="givenName" />
       <OutputClaim ClaimTypeReferenceId="surName" />
       <OutputClaim ClaimTypeReferenceId="city"/>
     </OutputClaims>
   </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Para recopilar la notificación de ciudad después del inicio de sesión inicial con una cuenta federada, se debe agregar como notificación de salida al perfil técnico `SelfAsserted-Social`. Para que los usuarios de cuentas locales y federadas puedan editar sus datos de perfil posteriormente, agregue las notificaciones de entrada y salida al perfil técnico `SelfAsserted-ProfileUpdate`. Reemplace estos perfiles técnicos en el archivo de extensión. Especifique la lista completa de notificaciones de salida para controlar el orden en que se presentan las notificaciones en la pantalla. Busque el elemento **ClaimsProviders**. Agregue un nuevo elemento ClaimsProviders tal como se muestra a continuación:

```xml
<ClaimsProvider>
  <DisplayName>Self Asserted</DisplayName>
  <TechnicalProfiles>
    <!--Federated account first-time sign-in page-->
    <TechnicalProfile Id="SelfAsserted-Social">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="city" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName"/>
        <OutputClaim ClaimTypeReferenceId="surname"/>
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
    <!--Edit profile page-->
    <TechnicalProfile Id="SelfAsserted-ProfileUpdate">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="city" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="read-and-write-a-claim"></a>Lectura y escritura de una notificación

Los siguientes perfiles técnicos son [perfiles técnicos de Active Directory](active-directory-technical-profile.md), que leen y escriben datos en Azure Active Directory.  
Use `PersistedClaims` para escribir datos en el perfil de usuario y `OutputClaims` para leer datos del perfil de usuario en los perfiles técnicos de Active Directory correspondientes.

Reemplace estos perfiles técnicos en el archivo de extensión. Busque el elemento **ClaimsProviders**.  Agregue un nuevo elemento ClaimsProviders tal como se muestra a continuación:

```xml
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <!-- Write data during a local account sign-up flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during a federated account first-time sign-in flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during edit profile flow. -->
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Read data after user resets the password. -->
    <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a local account. -->
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a federated account. -->
    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="include-a-claim-in-the-token"></a>Incorporación de una notificación en el token 

Para devolver la notificación de ciudad a la aplicación de usuario de confianza, agregue una notificación de salida al archivo <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. La notificación de salida se agregará al token después de un recorrido del usuario correcto y se enviará a la aplicación. Modifique el elemento de perfil técnico en la sección de usuario de confianza para agregar la ciudad como una notificación de salida.
 
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
      <OutputClaim ClaimTypeReferenceId="city" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="upload-and-test-your-updated-custom-policy"></a>Carga y prueba de la directiva personalizada actualizada

1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Cargar directiva personalizada**.
1. Cargue los archivos de directiva que ha cambiado anteriormente.

### <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. Desde la página de registro o inicio de sesión, seleccione **Registrarse ahora** para registrarse. Termine de escribir la información del usuario, incluido el nombre de la ciudad, y haga clic en **Crear**. Debería ver el contenido del token que se devolvió.

::: zone-end

La pantalla de registro debe ser similar a la captura de pantalla siguiente:

![Captura de pantalla de la opción de registro modificada](./media/configure-user-input/signup-with-city-claim-drop-down-example.png)

El token enviado de vuelta a la aplicación contiene la notificación `city`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1583500140,
  "nbf": 1583496540,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1583496540,
  "auth_time": 1583496540,
  "name": "Emily Smith",
  "email": "joe@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "city": "Berlin"
  ...
}
```

::: zone pivot="b2c-custom-policy"

## <a name="optional-localize-the-ui"></a>[Opcional] Localización de la interfaz de usuario

Azure AD B2C le permite adaptar su directiva a diferentes idiomas. Para más información, [obtenga información sobre cómo personalizar la experiencia de idioma](language-customization.md). Para encontrar la página de registro, [configure la lista de idiomas admitidos](language-customization.md#set-up-the-list-of-supported-languages) y [proporcione etiquetas específicas del idioma](language-customization.md#provide-language-specific-labels).

> [!NOTE]
> Cuando `LocalizedCollection` se usa con las etiquetas específicas del idioma, puede quitar la colección `Restriction` de la [definición de notificación](#define-a-claim).

En el ejemplo siguiente se muestra cómo proporcionar la lista de ciudades para inglés y español. Ambos establecen la colección `Restriction` de la notificación *city* con una lista de elementos para inglés y español. [SelectByDefault](claimsschema.md#enumeration) selecciona un elemento de forma predeterminada cuando se carga la página por primera vez.
   
```xml
<!-- 
<BuildingBlocks>-->
  <Localization Enabled="true">
    <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
      <SupportedLanguage>en</SupportedLanguage>
      <SupportedLanguage>es</SupportedLanguage>
    </SupportedLanguages>
    <LocalizedResources Id="api.localaccountsignup.en">
      <LocalizedCollections>
        <LocalizedCollection ElementType="ClaimType" ElementId="city" TargetCollection="Restriction">
          <Item Text="Berlin" Value="Berlin"></Item>
          <Item Text="London" Value="London" SelectByDefault="true"></Item>
          <Item Text="Seattle" Value="Seattle"></Item>
        </LocalizedCollection>
      </LocalizedCollections>
    </LocalizedResources>
    <LocalizedResources Id="api.localaccountsignup.es">
      <LocalizedCollections>
        <LocalizedCollection ElementType="ClaimType" ElementId="city" TargetCollection="Restriction">
          <Item Text="Berlina" Value="Berlin"></Item>
          <Item Text="Londres" Value="London" SelectByDefault="true"></Item>
          <Item Text="Seattle" Value="Seattle"></Item>
        </LocalizedCollection>
      </LocalizedCollections>
    </LocalizedResources>
  </Localization>
<!-- 
</BuildingBlocks>-->
```

Después de agregar el elemento de localización, [edite la definición de contenido con la localización](language-customization.md#edit-the-content-definition-with-the-localization). En el ejemplo siguiente, los recursos localizados personalizados inglés (en) y español (es) se agregan a la página de registro:
   
```xml
<!-- 
<BuildingBlocks>
  <ContentDefinitions> -->
   <ContentDefinition Id="api.localaccountsignup">
    <LocalizedResourcesReferences MergeBehavior="Prepend">
        <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
        <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
    </LocalizedResourcesReferences>
   </ContentDefinition>
  <!-- 
  </ContentDefinitions>
</BuildingBlocks>-->
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el elemento [ClaimsSchema](claimsschema.md) en la referencia de IEF.
- Aprenda a [usar atributos personalizados en Azure AD B2C](user-flow-custom-attributes.md).

::: zone-end
