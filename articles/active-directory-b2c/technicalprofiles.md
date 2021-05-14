---
title: Perfiles técnicos
titleSuffix: Azure AD B2C
description: Especifique el elemento TechnicalProfiles de una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/10/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: c555f86113b9a4557e3b5ebc356f63a5e5dc1eb1
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109664373"
---
# <a name="technical-profiles"></a>Perfiles técnicos

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Un *perfil técnico* proporciona un marco con un mecanismo integrado para comunicarse con distintos tipos de partes. Los perfiles técnicos se usan para comunicarse con su inquilino de Azure Active Directory B2C (Azure AD B2C) a fin de crear un usuario o leer un perfil de usuario. Un perfil técnico puede ser autoafirmado para habilitar la interacción con el usuario. Por ejemplo, un perfil técnico puede recopilar las credenciales del usuario para iniciar sesión y, a continuación, representar la página de registro o de restablecimiento de contraseña.

## <a name="types-of-technical-profiles"></a>Tipos de perfiles técnicos

Un perfil técnico posibilita estos tipos de escenarios:

- [Application Insights](analytics-with-application-insights.md): envía datos de eventos a [Application Insights](../azure-monitor/app/app-insights-overview.md).
- [Azure AD](active-directory-technical-profile.md): proporciona compatibilidad con la administración de usuarios de Azure AD B2C.
- [Autenticación multifactor de Azure AD](multi-factor-auth-technical-profile.md): proporciona compatibilidad para verificar un número de teléfono mediante la autenticación multifactor de Azure AD.
- [Transformación de notificaciones](claims-transformation-technical-profile.md): llamada a transformaciones de notificaciones de salida para manipular los valores de notificaciones, validar las notificaciones o establecer valores predeterminados para un conjunto de notificaciones de salida.
- [Sugerencia de token de id.](id-token-hint.md): valida la firma del token de JWT `id_token_hint`, el nombre del emisor y la audiencia del token y extrae la notificación del token de entrada.
- [Emisor del token de JWT](jwt-issuer-technical-profile.md): emite un token de JWT que se devuelve a la aplicación de usuario de confianza.
- [OAuth1](oauth1-technical-profile.md) : federación con cualquier proveedor de identidades del protocolo de OAuth 1.0.
- [OAuth2](oauth2-technical-profile.md) : federación con cualquier proveedor de identidades del protocolo de OAuth 2.0.
- [Contraseña de un solo uso](one-time-password-technical-profile.md): proporciona compatibilidad para administrar la generación y verificación de una contraseña de un solo uso.
- [OpenID Connect](openid-connect-technical-profile.md): federación con cualquier proveedor de identidades del protocolo OpenID Connect.
- [Factor de teléfono](phone-factor-technical-profile.md): admite la inscripción y verificación de números de teléfono.
- [Proveedor de RESTful](restful-technical-profile.md): llama a los servicios de la API REST y, por ejemplo, valida la entrada de usuario, enriquece los datos de usuario o realiza la integración con aplicaciones de línea de negocio.
- [Proveedor de identidades SAML](identity-provider-generic-saml.md): federación con cualquier proveedor de identidades del protocolo SAML.
- [Emisor de tokens SAML](saml-service-provider.md): emite un token SAML que se devuelve a la aplicación de usuario de confianza.
- [Autoaserción](self-asserted-technical-profile.md): interactúa con el usuario. Por ejemplo, recopila las credenciales del usuario para iniciar sesión, representar la página de registro o restablecer la contraseña.
- [Administración de sesiones](custom-policy-reference-sso.md): administra distintos tipos de sesiones.

## <a name="technical-profile-flow"></a>Flujo del perfil técnico

Todos los tipos de perfiles técnicos comparten el mismo concepto. Empiezan leyendo las notificaciones de entrada y ejecutan las transformaciones de notificaciones. A continuación, se ponen en contacto con la entidad configurada, como un proveedor de identidades, la API REST o los servicios de directorio de Azure AD. Una vez finalizado el proceso, el perfil técnico devuelve las notificaciones de salida y puede ejecutar transformaciones de notificaciones de salida. El diagrama siguiente muestra cómo se procesan las transformaciones y las asignaciones a las que se hace referencia en el perfil técnico. Una vez que se ejecute la transformación de las notificaciones, las notificaciones de salida se almacenan de inmediato en el contenedor de notificaciones, independientemente de la parte con la que interactúe el perfil técnico.

![Diagrama que muestra el flujo del perfil técnico.](./media/technical-profiles/technical-profile-flow.png)

1. **Administración de la sesión de inicio de sesión único (SSO)** : restaura el estado de sesión del perfil técnico mediante la [administración de la sesión de inicio de sesión único](custom-policy-reference-sso.md).
1. **Transformación de notificaciones de entrada**: antes de iniciarse el perfil técnico, Azure AD B2C ejecuta la entrada de [transformación de notificaciones](claimstransformations.md).
1. **Notificaciones de entrada**: las notificaciones se toman del contenedor de notificaciones y se usan para el perfil técnico.
1. **Ejecución de perfil técnico**: el perfil técnico intercambia las notificaciones con la parte configurada. Por ejemplo:
    - Redirige el usuario al proveedor de identidades para completar el inicio de sesión. Después de iniciar sesión correctamente, el usuario regresa y la ejecución del perfil técnico continúa.
    - Llama a una API REST mientras envía parámetros como InputClaims y recupera información como OutputClaims.
    - Crea o actualiza los usuarios la cuenta de usuario.
    - Envía y verifica el mensaje de texto de autenticación multifactor.
1. **Perfiles técnicos de validación**: un [perfil técnico autoafirmado](self-asserted-technical-profile.md) puede llamar a [perfiles técnicos de validación](validation-technical-profile.md) para validar los datos generados por el usuario.
1. **Notificaciones de salida**: las notificaciones se devuelven al contenedor de notificaciones. Puede usar esas notificaciones en el siguiente paso de orquestaciones o generar transformaciones de notificaciones.
1. **Transformaciones de notificaciones de salida**: una vez completado el perfil técnico, Azure AD B2C ejecuta las [transformaciones de notificaciones](claimstransformations.md) de salida.
1. **Administración de la sesión de SSO**: conserva los datos del perfil técnico en la sesión mediante la [administración de la sesión de SSO](custom-policy-reference-sso.md).

Un elemento **TechnicalProfiles** contiene un conjunto de perfiles técnicos admitidos por el proveedor de notificaciones. Cada proveedor de notificaciones debe tener al menos un perfil técnico. El perfil técnico determina los puntos de conexión y los protocolos necesarios para establecer comunicación con el proveedor de notificaciones. Un proveedor de notificaciones puede tener varios perfiles técnicos.

```xml
<ClaimsProvider>
  <DisplayName>Display name</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Technical profile identifier">
      <DisplayName>Display name of technical profile</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        ...
      </Metadata>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

El elemento **TechnicalProfile** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
|---------|---------|---------|
| Identificador | Sí | Un identificador único del perfil técnico. Se puede hacer referencia al perfil técnico con este identificador desde otros elementos del archivo de directiva. Algunos ejemplos son **OrchestrationSteps** y **ValidationTechnicalProfile**. |

El elemento **TechnicalProfile** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Domain | 0:1 | Nombre de dominio del perfil técnico. Por ejemplo, si el perfil técnico especifica el proveedor de identidades de Facebook, el nombre de dominio será Facebook.com. |
| DisplayName | 1:1 | Nombre para mostrar del perfil técnico. |
| Descripción | 0:1 | Descripción del perfil técnico. |
| Protocolo | 1:1 | Protocolo usado para la comunicación con la otra parte. |
| Metadatos | 0:1 | Conjunto de claves y valores que controla el comportamiento del perfil técnico. |
| InputTokenFormat | 0:1 | Formato del token de entrada. Los valores posibles son `JSON`, `JWT`, `SAML11` o `SAML2`. El valor `JWT` representa un token de JSON Web Token según la especificación IETF. El valor `SAML11` representa un token de seguridad SAML 1.1 según la especificación OASIS. El valor `SAML2` representa un token de seguridad SAML 2.0 según la especificación OASIS. |
| OutputTokenFormat | 0:1 | Formato del token de salida. Los valores posibles son `JSON`, `JWT`, `SAML11` o `SAML2`. |
| CryptographicKeys | 0:1 | Lista de claves criptográficas que se usan en el perfil técnico. |
| InputClaimsTransformations | 0:1 | Lista de referencias (definidas anteriormente) a transformaciones de notificaciones que tienen que ejecutarse antes de enviar notificaciones al proveedor de notificaciones o al usuario de confianza. |
| InputClaims | 0:1 | Lista de referencias (definidas anteriormente) a tipos de notificación que se usan como entrada en el perfil técnico. |
| PersistedClaims | 0:1 | Lista de referencias (definidas anteriormente) a tipos de notificación que serán persistentes en el perfil técnico. |
| DisplayClaims | 0:1 | Lista de referencias (definidas anteriormente) a tipos de notificación que el [perfil técnico autoafirmado](self-asserted-technical-profile.md) presenta. La característica DisplayClaims se encuentra actualmente en versión preliminar. |
| OutputClaims | 0:1 | Lista de referencias (definidas anteriormente) a tipos de notificación que se usan como salida en el perfil técnico. |
| OutputClaimsTransformations | 0:1 | Lista de referencias (definidas anteriormente) a transformaciones de notificaciones que tienen que ejecutarse cuando el proveedor de notificaciones reciba las notificaciones. |
| ValidationTechnicalProfiles | 0:n | Lista de referencias a otros perfiles técnicos que el perfil técnico usa con fines de validación. Para obtener más información, vea [Validación del perfil técnico](validation-technical-profile.md).|
| SubjectNamingInfo | 0:1 | Controla la producción del nombre de firmante en los tokens donde el nombre de firmante se especifica por separado de las notificaciones. Algunos ejemplos son OAuth o SAML. |
| IncludeInSso | 0:1 | Indica si el uso de este perfil técnico debe aplicar el comportamiento de SSO para la sesión o, en su lugar, requerir una interacción explícita. Este elemento solo es válido en los perfiles de SelfAsserted usados dentro de un perfil técnico de validación. Los valores posibles son `true` (valor predeterminado) o `false`. |
| IncludeClaimsFromTechnicalProfile | 0:1 | Identificador de un perfil técnico cuyas notificaciones de entrada o salida quiere que se agreguen a este perfil técnico. El perfil técnico al que se haga referencia tiene que definirse en el mismo archivo de directiva. |
| IncludeTechnicalProfile |0:1 | Identificador de un perfil técnico cuyos datos quiere agregar a este perfil técnico. |
| UseTechnicalProfileForSessionManagement | 0:1 | Perfil técnico distinto que se usará para la administración de sesiones. |
|EnabledForUserJourneys| 0:1 |Controla si el perfil técnico se ejecuta en un recorrido del usuario. |

## <a name="protocol"></a>Protocolo

El elemento **Protocol** especifica el protocolo que se va a usar para la comunicación con la otra parte. El elemento **Protocol** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Nombre | Sí | Nombre de un protocolo válido admitido por Azure AD B2C que se usará como parte del perfil técnico. Los valores posibles son `OAuth1`, `OAuth2`, `SAML2`, `OpenIdConnect`, `Proprietary` o `None`. |
| Controlador | No | Cuando el nombre del protocolo se establece en `Proprietary`, especifica el nombre del ensamblado que usará Azure AD B2C para determinar el controlador de protocolo. |

## <a name="metadata"></a>Metadatos

El elemento **Metadata** contiene las opciones de configuración pertinentes de un protocolo específico. La lista de metadatos admitidos se documenta en la especificación del [perfil técnico](#types-of-technical-profiles) correspondiente. Un elemento **Metadata** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Elemento | 0:n | Los metadatos relacionados con el perfil técnico. Cada tipo de perfil técnico tiene un conjunto distinto de elementos de metadatos. Para más información, vea la sección sobre los tipos de perfil técnico. |

### <a name="item"></a>Elemento

El elemento **Item** del elemento **Metadata** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Clave | Sí | La clave de metadatos. Puede ver la lista de elementos de metadatos en cada [tipo de perfil técnico](#types-of-technical-profiles). |

En el ejemplo siguiente se muestra el uso de metadatos pertinentes para el [perfil técnico de OAuth2](oauth2-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  ...
  <Metadata>
    <Item Key="ProviderName">facebook</Item>
    <Item Key="authorization_endpoint">https://www.facebook.com/dialog/oauth</Item>
    <Item Key="AccessTokenEndpoint">https://graph.facebook.com/oauth/access_token</Item>
    <Item Key="HttpBinding">GET</Item>
    <Item Key="UsePolicyInRedirectUri">0</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

En el ejemplo siguiente se muestra el uso de metadatos pertinentes para el [perfil técnico de la API REST](restful-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="REST-Validate-Email">
  ...
  <Metadata>
    <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
    <Item Key="AuthenticationType">Bearer</Item>
    <Item Key="SendClaimsIn">Body</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

## <a name="cryptographic-keys"></a>Claves de cifrado

Para establecer la confianza con los servicios con los que se integra, Azure AD B2C almacena secretos y certificados en forma de [claves de directiva](policy-keys-overview.md). Durante la ejecución del perfil técnico, Azure AD B2C recupera las claves criptográficas de las claves de directiva de Azure AD B2C. A continuación, las usa para establecer la confianza, cifrar o firmar un token. Estas confianzas constan de los siguientes componentes:

- Federación con los proveedores de identidades [OAuth1](oauth1-technical-profile.md#cryptographic-keys), [OAuth2](oauth2-technical-profile.md#cryptographic-keys) y [SAML](identity-provider-generic-saml.md)
- Protección de la conexión con los [servicios de la API REST](secure-rest-api.md)
- Firma y cifrado de los tokens [JWT](jwt-issuer-technical-profile.md#cryptographic-keys) y [SAML](saml-service-provider.md)

El elemento **CryptographicKeys** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Clave | 1:n | Una clave criptográfica usada en este perfil técnico. |

### <a name="key"></a>Clave

El elemento **Key** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | No | Un identificador único de un par de claves específico al que se hace referencia desde otros elementos en el archivo de directiva. |
| StorageReferenceId | Sí | Un identificador del contenedor de claves de almacenamiento al que se hace referencia desde otros elementos en el archivo de directiva. |

## <a name="input-claims-transformations"></a>Transformaciones de notificaciones de entrada

El elemento **InputClaimsTransformations** puede contener una colección de elementos de transformación de notificaciones de entrada que se usan para modificar las notificaciones de entrada o generar otras nuevas.

Las notificaciones de salida de una transformación de notificaciones anterior de la colección de estas transformaciones pueden ser notificaciones de entrada de una transformación de notificaciones de entrada subsiguiente. De esta manera, puede tener una secuencia de transformaciones de notificaciones que dependen unas de otras.

El elemento **InputClaimsTransformations** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| InputClaimsTransformation | 1:n | El identificador de una transformación de notificaciones que tiene que ejecutarse antes de enviar notificaciones al proveedor de notificaciones o al usuario de confianza. Una transformación de notificaciones puede usarse para modificar notificaciones ClaimsSchema existentes o para generar nuevas. |

### <a name="inputclaimstransformation"></a>InputClaimsTransformation

El elemento **InputClaimsTransformation** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Un identificador de una transformación de notificaciones que ya se ha definido en el archivo de directiva o en el archivo de directiva principal. |

Los siguientes perfiles técnicos hacen referencia a la transformación de notificaciones **CreateOtherMailsFromEmail**. La transformación de notificaciones agrega el valor de la notificación `email` a la colección `otherMails` antes de conservar los datos en el directorio.

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  ...
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="CreateOtherMailsFromEmail" />
  </InputClaimsTransformations>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <PersistedClaim ClaimTypeReferenceId="userPrincipalName" />
    <PersistedClaim ClaimTypeReferenceId="mailNickName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="otherMails" />
    <PersistedClaim ClaimTypeReferenceId="givenName" />
    <PersistedClaim ClaimTypeReferenceId="surname" />
  </PersistedClaims>
  ...
</TechnicalProfile>
```

## <a name="input-claims"></a>Notificaciones de entrada

El elemento **InputClaims** toma notificaciones del contenedor de notificaciones que  se usan para el perfil técnico. Por ejemplo, un [perfil técnico autoafirmado](self-asserted-technical-profile.md) usa las notificaciones de entrada para rellenar previamente las notificaciones de salida que proporciona el usuario. Un perfil técnico de la API REST usa las notificaciones de entrada para enviar los parámetros de entrada al punto de conexión de la API REST. Azure AD usa la notificación de entrada como un identificador único para leer, actualizar o eliminar una cuenta.

El elemento **InputClaims** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | Un tipo de notificación de entrada esperado. |

### <a name="inputclaim"></a>InputClaim

El elemento **InputClaim** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Sí | Identificador de un tipo de notificación. La notificación ya se ha definido en la sección del esquema de notificaciones del archivo de directiva o del archivo de directiva primario. |
| DefaultValue | No | Valor predeterminado que se usará para crear una notificación si la que indicó ClaimTypeReferenceId no existe, por lo que el perfil técnico puede usar la notificación resultante como un elemento InputClaim. |
| PartnerClaimType | No | El identificador del tipo de notificación del socio externo al que se asigna el tipo de notificación de directiva especificado. Si no se especifica el atributo PartnerClaimType, el tipo de notificación de directiva especificado se asignará al tipo de notificación del asociado que tenga el mismo nombre. Use esta propiedad cuando el nombre del tipo de notificación sea distinto de la otra entidad. Por ejemplo, el nombre de la primera notificación es *givenName*, mientras que el asociado usa una notificación denominada *first_name*. |

## <a name="display-claims"></a>Notificaciones de visualización

El elemento **DisplayClaims** contiene una lista de las notificaciones que se van a presentar en la pantalla para recopilar datos del usuario. En la colección de notificaciones de visualización, puede incluir una referencia a un [tipo de notificación](claimsschema.md) o a un [control de visualización](display-controls.md) que haya creado.

- Un tipo de notificación es una referencia a una notificación que se va a mostrar en la pantalla.
  - Para forzar que el usuario proporcione un valor para una notificación específica, establezca el atributo **Required** del elemento **DisplayClaim** en `true`.
  - Para rellenar previamente los valores de las notificaciones de visualización, use las notificaciones de entrada descritas anteriormente. El elemento también puede incluir un valor predeterminado.
  - El elemento **ClaimType** de la colección **DisplayClaims** necesita establecer el elemento **UserInputType** en cualquier tipo de entrada de usuario admitido por Azure AD B2C. Algunos ejemplos son `TextBox` o `DropdownSingleSelect`.
- Un control de visualización es un elemento de la interfaz de usuario que tiene una funcionalidad especial e interactúa con el servicio de back-end de Azure AD B2C. Permite al usuario realizar acciones en la página que invocan un perfil técnico de validación en el back-end. Por ejemplo, la verificación de una dirección de correo electrónico, un número de teléfono o un número de fidelidad del cliente.

El orden de los elementos en **DisplayClaims** especifica el orden en que Azure AD B2C presenta las notificaciones en la pantalla.

El elemento **DisplayClaims** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| DisplayClaim | 1:n | Un tipo de notificación de entrada esperado. |

### <a name="displayclaim"></a>DisplayClaim

El elemento **DisplayClaim** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | No | El identificador de un tipo de notificación que ya se ha definido en la sección ClaimsSchema del archivo de directiva o del archivo de directiva principal. |
| DisplayControlReferenceId | No | Identificador de un [control de visualización](display-controls.md) ya definido en la sección ClaimsSchema del archivo de directiva o del archivo de directiva principal. |
| Obligatorio | No | Indica si la notificación de visualización es obligatoria. |

En el ejemplo siguiente se muestra el uso de notificaciones y controles de visualización en un perfil técnico autoafirmado.

![Captura de pantalla que muestra un perfil técnico autoafirmado con notificaciones de visualización.](./media/technical-profiles/display-claims.png)

En el siguiente perfil técnico:

- La primera notificación de visualización hace referencia al control de visualización `emailVerificationControl`, que recopila y comprueba la dirección de correo.
- La quinta notificación de visualización hace referencia al control de visualización `phoneVerificationControl`, que recopila y comprueba un número de teléfono.
- Las demás notificaciones de visualización son elementos ClaimTypes que se recopilarán del usuario.

```xml
<TechnicalProfile Id="Id">
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
    <DisplayClaim DisplayControlReferenceId="phoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
  </DisplayClaims>
</TechnicalProfile>
```

## <a name="persisted-claims"></a>Notificaciones persistentes

El elemento **PersistedClaims** contiene todos los valores que un [perfil técnico de Azure AD](active-directory-technical-profile.md) debe conservar con la información de asignaciones posibles entre un tipo de notificación ya definido en la sección [ClaimsSchema](claimsschema.md) de la directiva y el nombre de atributo de Azure AD.

El nombre de la notificación es el del [atributo de Azure AD](user-profile-attributes.md), a menos que se especifique el atributo **PartnerClaimType**, que contiene el nombre de atributo de Azure AD.

El elemento **PersistedClaims** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| PersistedClaim | 1:n | El tipo de notificación que persistirá. |

### <a name="persistedclaim"></a>PersistedClaim

El elemento **PersistedClaim** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Sí | El identificador de un tipo de notificación que ya se ha definido en la sección ClaimsSchema del archivo de directiva o del archivo de directiva principal. |
| DefaultValue | No | Un valor predeterminado que se usará para crear una notificación si esta no existe. |
| PartnerClaimType | No | El identificador del tipo de notificación del socio externo al que se asigna el tipo de notificación de directiva especificado. Si no se especifica el atributo PartnerClaimType, el tipo de notificación de directiva especificado se asignará al tipo de notificación del asociado que tenga el mismo nombre. Use esta propiedad cuando el nombre del tipo de notificación sea distinto de la otra entidad. Por ejemplo, el nombre de la primera notificación es *givenName*, mientras que el asociado usa una notificación denominada *first_name*. |

En el siguiente ejemplo, el perfil técnico **AAD-UserWriteUsingLogonEmail** o el [paquete de inicio](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccounts) crea una cuenta local y conserva las notificaciones siguientes:

```xml
<PersistedClaims>
  <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" />
  <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password"/>
  <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
  <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration" />
  <PersistedClaim ClaimTypeReferenceId="givenName" />
  <PersistedClaim ClaimTypeReferenceId="surname" />
</PersistedClaims>
```

## <a name="output-claims"></a>Notificaciones de salida

El elemento **OutputClaims** es una colección de notificaciones que se devuelven al contenedor de notificaciones una vez completado el perfil técnico. Puede usar esas notificaciones en el siguiente paso de orquestaciones o generar transformaciones de notificaciones. El elemento **OutputClaims** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| OutputClaim | 1:n | Un tipo de notificación de salida esperado. |

### <a name="outputclaim"></a>OutputClaim

El elemento **OutputClaim** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Sí | El identificador de un tipo de notificación que ya se ha definido en la sección ClaimsSchema del archivo de directiva o del archivo de directiva principal. |
| DefaultValue | No | Un valor predeterminado que se usará para crear una notificación si esta no existe. |
|AlwaysUseDefaultValue |No |Obliga a usar el valor predeterminado. |
| PartnerClaimType | No | El identificador del tipo de notificación del socio externo al que se asigna el tipo de notificación de directiva especificado. Si no se especifica el atributo de tipo de notificación de asociado, el tipo de notificación de directiva especificado se asignará al tipo de notificación del asociado que tenga el mismo nombre. Use esta propiedad cuando el nombre del tipo de notificación sea distinto de la otra entidad. Por ejemplo, el nombre de la primera notificación es *givenName*, mientras que el asociado usa una notificación denominada *first_name*. |

## <a name="output-claims-transformations"></a>Transformaciones de notificaciones de salida

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation**. Las transformaciones de notificaciones de salida se usan para modificar las notificaciones de salida o generar otras nuevas. Después de la ejecución, las notificaciones de salida se vuelven a poner en el contenedor de notificaciones. Puede usar esas notificaciones en el paso de orquestaciones siguiente.

Las notificaciones de salida de una transformación de notificaciones anterior de la colección de estas transformaciones pueden ser notificaciones de entrada de una transformación de notificaciones de entrada subsiguiente. De esta manera, puede tener una secuencia de transformaciones de notificaciones que dependen unas de otras.

El elemento **OutputClaimsTransformations** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| OutputClaimsTransformation | 1:n | Los identificadores de transformaciones de notificaciones que tienen que ejecutarse antes de enviar notificaciones al proveedor de notificaciones o al usuario de confianza. Una transformación de notificaciones puede usarse para modificar notificaciones ClaimsSchema existentes o para generar nuevas. |

### <a name="outputclaimstransformation"></a>OutputClaimsTransformation

El elemento **OutputClaimsTransformation** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Un identificador de una transformación de notificaciones que ya se ha definido en el archivo de directiva o en el archivo de directiva principal. |

El siguiente perfil técnico hace referencia a la transformación de notificaciones AssertAccountEnabledIsTrue para evaluar si la cuenta se habilita o no después de leer la notificación `accountEnabled` del directorio.

```xml
<TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
  ...
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="accountEnabled" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
  </OutputClaims>
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertAccountEnabledIsTrue" />
  </OutputClaimsTransformations>
  ...
</TechnicalProfile>
```

## <a name="validation-technical-profiles"></a>Perfiles técnicos de validación

Los perfiles técnicos de validación se usan para validar las notificaciones de salida de [perfil técnico autoafirmado](self-asserted-technical-profile.md#validation-technical-profiles). Un perfil técnico de validación es un perfil técnico normal de cualquier protocolo, como [Azure AD](active-directory-technical-profile.md) o una [API REST](restful-technical-profile.md). El perfil técnico de validación devuelve notificaciones de salida o el código de error. El mensaje de error se representa para el usuario en la pantalla, lo que le permite volver a intentarlo.

En el siguiente diagrama se muestra cómo Azure AD B2C usa un perfil técnico de validación para validar las credenciales de usuario.

![Diagrama que muestra un flujo de perfil técnico de validación.](./media/technical-profiles/validation-technical-profile.png)

El elemento **ValidationTechnicalProfiles** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Los identificadores de perfiles técnicos que se usan para validar algunas o todas las notificaciones de salida del perfil técnico al que se hace referencia. Todas las notificaciones de entrada del perfil técnico al que se hace referencia tienen que aparecer en las notificaciones de salida del perfil técnico al que se hace referencia. |

### <a name="validationtechnicalprofile"></a>ValidationTechnicalProfile

El elemento **ValidationTechnicalProfile** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Un identificador de un perfil técnico que ya se ha definido en el archivo de directiva o en el archivo de directiva principal. |

## <a name="subjectnaminginfo"></a>SubjectNamingInfo

El elemento **SubjectNamingInfo** define el nombre del firmante que se usa en los tokens de una [directiva de usuario de confianza](relyingparty.md#subjectnaminginfo). El elemento **SubjectNamingInfo** contiene el siguiente atributo:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimType | Sí | Un identificador de un tipo de notificación que ya se ha definido en la sección ClaimsSchema del archivo de directiva. |

## <a name="include-technical-profile"></a>Inclusión de un perfil técnico

Un perfil técnico puede incluir otro perfil técnico para cambiar la configuración o agregar funcionalidades. El elemento **IncludeTechnicalProfile** es una referencia al perfil técnico común del que se deriva un perfil técnico. Para reducir la redundancia y complejidad de los elementos de la directiva, use la inclusión cuando tenga varios perfiles técnicos que compartan los elementos básicos. Use un perfil técnico común con el conjunto común de configuración, junto con perfiles técnicos de tareas específicos que incluyen el perfil técnico común.

Supongamos que tiene un [perfil técnico de la API REST](restful-technical-profile.md) con un solo punto de conexión en el que debe enviar diferentes conjuntos de notificaciones para diversos escenarios. Cree un perfil técnico común con la funcionalidad compartida, como el URI de punto de conexión de la API de REST, los metadatos, el tipo de autenticación y las claves criptográficas. Cree perfiles técnicos de tareas específicas que incluyan el perfil técnico común. Después, agregue las notificaciones de entrada y salida o sobrescriba el URI del punto de conexión de la API REST pertinente para ese perfil técnico.

El elemento **IncludeTechnicalProfile** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Un identificador de un perfil técnico que ya se ha definido en el archivo de directiva o en el archivo de directiva principal. |

En el siguiente ejemplo se muestra el uso de la instrucción:

- **REST-API-Common**: perfil técnico común con la configuración básica.
- **REST-ValidateProfile**: incluye el perfil técnico **REST-API-Common** y especifica las notificaciones de entrada y salida.
- **REST-UpdateProfile**: incluye el perfil técnico **REST-API-Common**, especifica las notificaciones de entrada y sobrescribe los metadatos `ServiceUrl`.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-API-Common">
      <DisplayName>Base REST API configuration</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="SendClaimsIn">Body</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_B2cRestClientId" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_B2cRestClientSecret" />
      </CryptographicKeys>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Validate the account and return promo code</DisplayName>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Common" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-UpdateProfile">
      <DisplayName>Update the user profile</DisplayName>  
      <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/update</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Common" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

### <a name="multilevel-inclusion"></a>Inclusión multinivel

Un perfil técnico puede incluir un solo perfil técnico. No existe ningún límite en el número de niveles de inclusión. Por ejemplo, el perfil técnico **AAD-UserReadUsingAlternativeSecurityId-NoError** incluye **AAD-UserReadUsingAlternativeSecurityId**. Este perfil técnico establece el elemento de metadatos `RaiseErrorIfClaimsPrincipalDoesNotExist` en `true` y genera un error si no existe una cuenta de redes sociales en el directorio. **AAD-UserReadUsingAlternativeSecurityId-NoError** invalida este comportamiento y deshabilita ese mensaje de error.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId-NoError">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">false</Item>
  </Metadata>
  <IncludeTechnicalProfile ReferenceId="AAD-UserReadUsingAlternativeSecurityId" />
</TechnicalProfile>
```

**AAD UserReadUsingAlternativeSecurityId** incluye el perfil técnico de `AAD-Common`.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="Operation">Read</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">User does not exist. Please sign up before you can sign in.</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityId" PartnerClaimType="alternativeSecurityId" Required="true" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surname" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

Ni **AAD-UserReadUsingAlternativeSecurityId-NoError** ni **AAD-UserReadUsingAlternativeSecurityId** especifican el elemento **Protocol**, obligatorio porque se especifica en el perfil técnico **AAD-Common**.

```xml
<TechnicalProfile Id="AAD-Common">
  <DisplayName>Azure Active Directory</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
</TechnicalProfile>
```

## <a name="use-technical-profile-for-session-management"></a>Uso de un perfil técnico para la administración de sesiones

El elemento **UseTechnicalProfileForSessionManagement** hace referencia al [perfil técnico de sesión de SSO](custom-policy-reference-sso.md). El elemento **UseTechnicalProfileForSessionManagement** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Un identificador de un perfil técnico que ya se ha definido en el archivo de directiva o en el archivo de directiva principal. |

## <a name="enabled-for-user-journeys"></a>Habilitado para recorridos de usuario

El elemento [ClaimsProviderSelections](userjourneys.md#claims-provider-selection) en un recorrido del usuario define la lista de opciones de selección de proveedor de notificaciones y el orden en que se mostrarán. Con el elemento **EnabledForUserJourneys**, puede filtrar los proveedores de notificaciones que estarán disponibles para el usuario. El elemento **EnabledForUserJourneys** contiene uno de los valores siguientes:

- **Always**: ejecuta el perfil técnico.
- **Never**: omite el perfil técnico.
- **OnClaimsExistence**: solo se ejecuta cuando existe una notificación específica indicada en el perfil técnico.
- **OnItemExistenceInStringCollectionClaim**: solo se ejecuta cuando existe un elemento en una notificación de colección de cadenas.
- **OnItemAbsenceInStringCollectionClaim**: solo se ejecuta cuando no existe ningún elemento en una notificación de colección de cadenas.

Para usar **OnClaimsExistence**, **OnItemExistenceInStringCollectionClaim** o **OnItemAbsenceInStringCollectionClaim**, es necesario que proporcione los siguientes metadatos:

- **ClaimTypeOnWhichToEnable**: especifica el tipo de notificación que se va a evaluar.
- **ClaimValueOnWhichToEnable**: especifica el valor que se va a comparar.

El siguiente perfil técnico solo se ejecuta si la colección de cadenas **identityProviders** contiene el valor de `facebook.com`:

```xml
<TechnicalProfile Id="UnLink-Facebook-OAUTH">
  <DisplayName>Unlink Facebook</DisplayName>
...
    <Metadata>
      <Item Key="ClaimTypeOnWhichToEnable">identityProviders</Item>
      <Item Key="ClaimValueOnWhichToEnable">facebook.com</Item>
    </Metadata>
...
  <EnabledForUserJourneys>OnItemExistenceInStringCollectionClaim</EnabledForUserJourneys>
</TechnicalProfile>
```
