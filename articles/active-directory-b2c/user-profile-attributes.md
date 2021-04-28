---
title: Atributos de perfil de usuario en Azure Active Directory B2C
description: Obtenga información sobre los atributos de tipo de recurso de usuario que son compatibles con el perfil de usuario del directorio de Azure AD B2C. Descubra los atributos integrados, las extensiones y cómo se asignan los atributos a Microsoft Graph.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 42bc87259265b45dabb631507ce6956e07077a7d
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070486"
---
# <a name="user-profile-attributes"></a>Atributos de perfil de usuario

El perfil de usuario del directorio de Azure Active Directory (Azure AD) B2C incluye un conjunto integrado de atributos, por ejemplo, givenName, surname, city, postalCode y telephoneNumber. Puede extender el perfil de usuario con sus propios datos de aplicación sin necesitar un almacén de datos externo. 

La mayoría de los atributos que se pueden utilizar con los perfiles de usuario de Azure AD B2C también se admiten en Microsoft Graph. En este artículo se describen los atributos de perfil de usuario que se admiten en Azure AD B2C. También se indican los atributos que no son compatibles con Microsoft Graph, así como los atributos de Microsoft Graph que no se deben usar con Azure AD B2C.

> [!IMPORTANT]
> No debe utilizar atributos de extensión o integrados para almacenar datos personales confidenciales, como credenciales de cuenta, números de identificación gubernamental, datos de titulares de tarjetas, datos de cuentas financieras, información sanitaria o la información básica confidencial.

También pueden integrarse con sistemas externos. Por ejemplo, puede usar Azure AD B2C para la autenticación, pero delegar en una base de datos externa de administración de las relaciones con el cliente (CRM) o de fidelización de clientes como el origen de autoridad de los datos de los clientes. Para obtener más información, consulte la solución de [perfil remoto](https://github.com/azure-ad-b2c/samples/tree/master/policies/remote-profile).

## <a name="azure-ad-user-resource-type"></a>Tipo de recurso de usuario de Azure AD

En la tabla siguiente se enumeran los atributos de [tipo de recurso de usuario](/graph/api/resources/user) que son compatibles con el perfil de usuario del directorio de Azure AD B2C. En la tabla siguiente se proporciona información sobre cada atributo:

- El nombre de atributo que usa Azure AD B2C (seguido del nombre de Microsoft Graph entre paréntesis, si es diferente).
- El tipo de datos del atributo.
- La descripción del atributo.
- Si el atributo está disponible en Azure Portal.
- Si el atributo se puede usar en un flujo de usuario.
- Si el atributo se puede usar en un [perfil técnico de Azure AD](active-directory-technical-profile.md) de una directiva personalizada, y en qué sección (&lt;InputClaims&gt;, &lt;OutputClaims&gt;o &lt;PersistedClaims&gt;).

|Nombre     |Tipo     |Descripción|Azure Portal|Flujos de usuario|Directiva personalizada|
|---------|---------|----------|------------|----------|-------------|
|accountEnabled  |Boolean|Si la cuenta de usuario está habilitada o deshabilitada: **true** si la cuenta está habilitada; en caso contrario, **false**.|Sí|No|Persistente, salida|
|ageGroup        |String|Grupo de edad del usuario. Valores posibles: null, Undefined, Minor, Adult y NotAdult.|Sí|No|Persistente, salida|
|alternativeSecurityId ([Identidades](#identities-attribute))|String|Una única identidad de usuario del proveedor de identidades externo.|No|No|Entrada, persistente, salida|
|alternativeSecurityIds ([Identidades](#identities-attribute))|Colección de identidades de seguridad alternativa|Una colección de identidades de usuario de proveedores de identidades externos.|No|No|Persistente, salida|
|city            |String|La ciudad en la que se encuentra el usuario. La longitud máxima es 128.|Sí|Sí|Persistente, salida|
|consentProvidedForMinor|String|Indica si se ha proporcionado el consentimiento para un menor. Valores permitidos: null, granted, denied o notRequired.|Sí|No|Persistente, salida|
|country         |String|El país o la región donde se encuentra el usuario. Ejemplo: "EE. UU." o "Reino Unido". La longitud máxima es 128.|Sí|Sí|Persistente, salida|
|createdDateTime|DateTime|La fecha de creación del objeto de usuario. Solo lectura.|No|No|Persistente, salida|
|creationType    |String|Si la cuenta de usuario se creó como una cuenta local para un inquilino de Azure Active Directory B2C, el valor es LocalAccount o nameCoexistence. Solo lectura.|No|No|Persistente, salida|
|dateOfBirth     |Date|La fecha de nacimiento.|No|No|Persistente, salida|
|department      |String|El nombre del departamento en el que trabaja el usuario. La longitud máxima es 64.|Sí|No|Persistente, salida|
|DisplayName     |String|El nombre para mostrar del usuario. La longitud máxima es 256.|Sí|Sí|Persistente, salida|
|facsimileTelephoneNumber<sup>1</sup>|String|El número de teléfono del equipo de fax del trabajo del usuario.|Sí|No|Persistente, salida|
|givenName       |String|EL nombre del usuario. La longitud máxima es 64.|Sí|Sí|Persistente, salida|
|jobTitle        |String|El puesto del usuario. La longitud máxima es 128.|Sí|Sí|Persistente, salida|
|immutableId     |String|Un identificador que se usa normalmente para los usuarios migrados desde Active Directory local.|No|No|Persistente, salida|
|legalAgeGroupClassification|String|La clasificación de grupo de edad legal. Valor de solo lectura que se calcula en función de las propiedades ageGroup y consentProvidedForMinor. Valores permitidos: null, minorWithOutParentalConsent, minorWithParentalConsent, minorNoParentalConsentRequired, notAdult y adult.|Sí|No|Persistente, salida|
|legalCountry<sup>1</sup>  |String|País o región para fines legales.|No|No|Persistente, salida|
|mailNickName    |String|El alias de correo electrónico del usuario. La longitud máxima es 64.|No|No|Persistente, salida|
|mobile (mobilePhone) |String|EL número de teléfono móvil principal del usuario. La longitud máxima es 64.|Sí|No|Persistente, salida|
|netId           |String|El identificador de red.|No|No|Persistente, salida|
|objectId        |String|El identificador único global (GUID) que es el identificador único del usuario. Ejemplo: 12345678-9abc-def0-1234-56789abcde. Valor de solo lectura e inmutable.|Solo lectura|Sí|Entrada, persistente, salida|
|otherMails      |Colección de cadenas|Una lista de otras direcciones de correo electrónico del usuario. Ejemplo: ["bob@contoso.com", "Robert@fabrikam.com"].|Sí (correo electrónico alternativo)|No|Persistente, salida|
|password        |String|La contraseña de la cuenta local durante la creación del usuario.|No|No|Guardado|
|passwordPolicies     |String|La directiva de la contraseña. Es una cadena que consta de un nombre de directiva diferente separado por una coma. Por ejemplo, "DisablePasswordExpiration, DisableStrongPassword".|No|No|Persistente, salida|
|physicalDeliveryOfficeName (officeLocation)|String|La ubicación del lugar de trabajo del usuario. La longitud máxima es 128.|Sí|No|Persistente, salida|
|postalCode      |String|El código postal de la dirección del usuario. El código postal es específico del país o la región del usuario. En Estados Unidos de América, este atributo contiene el código ZIP. La longitud máxima es 40.|Sí|No|Persistente, salida|
|preferredLanguage    |String|El idioma preferido del usuario. El formato de idioma preferido se basa en RFC 4646. El nombre es una combinación de un código de referencia cultural ISO 639 de dos letras en minúsculas asociado con un idioma y un código de referencia cultural secundaria ISO 3166 de dos letras en mayúsculas asociado con un país o región. Ejemplo: "en-US" o "es-ES".|No|No|Persistente, salida|
|refreshTokensValidFromDateTime (signInSessionsValidFromDateTime)|DateTime|Los tokens de actualización emitidos antes de esta hora no son válidos. Las aplicaciones obtendrán un error al usar un token de actualización no válido para adquirir un nuevo token de acceso. Si esto ocurre, la aplicación tendrá que adquirir un nuevo token de actualización realizando una solicitud al punto de conexión de autorización. Solo lectura.|No|No|Output|
|signInNames ([Identidades](#identities-attribute)) |String|El nombre de inicio de sesión único del usuario de cuenta local de cualquier tipo en el directorio. Use este atributo para obtener un usuario con el valor de inicio de sesión sin especificar el tipo de cuenta local.|No|No|Entrada|
|signInNames.userName ([Identidades](#identities-attribute)) |String|El nombre de inicio de sesión único del usuario de cuenta local en el directorio. Use este atributo para crear u obtener un usuario con un nombre de usuario de inicio de sesión específico. Si se especifica en PersistedClaims solo durante la operación de revisión, se quitarán otros tipos de signInNames. Si desea agregar un nuevo tipo de signInNames, también debe conservar los atributos signInNames que ya existan.|No|No|Entrada, persistente, salida|
|signInNames.phoneNumber ([Identidades](#identities-attribute)) |String|El número de teléfono único del usuario de cuenta local en el directorio. Use este atributo para crear u obtener un usuario con un número de teléfono de inicio de sesión específico. Si se especifica este atributo en PersistedClaims solo durante la operación de revisión, se quitarán otros tipos de signInNames. Si desea agregar un nuevo tipo de signInNames, también debe conservar los atributos signInNames que ya existan.|No|No|Entrada, persistente, salida|
|signInNames.emailAddress ([Identidades](#identities-attribute))|String|La dirección de correo electrónico única del usuario de cuenta local en el directorio. Se usa para crear u obtener un usuario con una dirección de correo electrónico de inicio de sesión específica. Si se especifica este atributo en PersistedClaims solo durante la operación de revisión, se quitarán otros tipos de signInNames. Si desea agregar un nuevo tipo de signInNames, también debe conservar los atributos signInNames que ya existan.|No|No|Entrada, persistente, salida|
|state           |String|El estado o provincia de la dirección del usuario. La longitud máxima es 128.|Sí|Sí|Persistente, salida|
|streetAddress   |String|La dirección postal del lugar de trabajo del usuario. La longitud máxima es 1024.|Sí|Sí|Persistente, salida|
|strongAuthentication AlternativePhoneNumber<sup>1</sup>|String|El número de teléfono secundario del usuario que se usa para la autenticación multifactor.|Sí|No|Persistente, salida|
|strongAuthenticationEmailAddress<sup>1</sup>|String|La dirección SMTP del usuario. Ejemplo: "bob@contoso.com". Este atributo se usa para el inicio de sesión con la directiva de nombre de usuario, para almacenar la dirección de correo electrónico del usuario. La dirección de correo electrónico que se usa en un flujo de restablecimiento de contraseña.|Sí|No|Persistente, salida|
|strongAuthenticationPhoneNumber<sup>2</sup>|String|El número de teléfono principal del usuario que se usa para la autenticación multifactor.|Sí|No|Persistente, salida|
|surname         |String|Los apellidos del usuario. La longitud máxima es 64.|Sí|Sí|Persistente, salida|
|telephoneNumber (primera entrada de businessPhones)|String|El número de teléfono principal del lugar de trabajo del usuario.|Sí|No|Persistente, salida|
|userPrincipalName    |String|El nombre principal de usuario (UPN) del usuario. El UPN es un nombre de inicio de sesión del estilo de Internet del usuario basado en el estándar de Internet RFC 822. El dominio debe encontrarse en la colección de dominios comprobados del inquilino. Esta propiedad es necesaria cuando se crea una cuenta. El valor es inmutable.|No|No|Entrada, persistente, salida|
|usageLocation   |String|Es necesario para los usuarios a los que se asignarán licencias debido al requisito legal de comprobar la disponibilidad de los servicios en los países o regiones. No acepta valores NULL. Código de país o región de dos letras (norma ISO 3166). Ejemplos: "US", "JP" y "GB".|Sí|No|Persistente, salida|
|userType        |String|Un valor de cadena que se puede usar para clasificar tipos de usuario en el directorio. El valor debe ser Miembro. Solo lectura.|Solo lectura|No|Persistente, salida|
|userState (externalUserState)<sup>3</sup>|String|Solo para la cuenta de Azure AD B2B; indica si la invitación está en el estado PendingAcceptance o Accepted.|No|No|Persistente, salida|
|userStateChangedOn (externalUserStateChangeDateTime)<sup>2</sup>|DateTime|Muestra la marca de tiempo del cambio más reciente de la propiedad UserState.|No|No|Persistente, salida|

<sup>1 </sup>No es compatible con Microsoft Graph.<br><sup>2 </sup>Para más información, consulte [Atributo de número de teléfono para MFA](#mfa-phone-number-attribute)<br><sup>3 </sup>No debe utilizarse con Azure AD B2C

## <a name="display-name-attribute"></a>Atributo de nombre para mostrar

`displayName` es el nombre que se va a mostrar en la administración de usuarios de Azure Portal para el usuario y, en el token de acceso que Azure AD B2C devuelve a la aplicación. Esta propiedad es obligatoria.

## <a name="identities-attribute"></a>Atributo de identidades

Una cuenta de cliente, que podría ser un consumidor, un asociado o un ciudadano, puede asociarse con estos tipos de identidad:

- Identidad **Local**: el nombre de usuario y la contraseña almacenados localmente en el directorio de Azure AD B2C. A menudo se hace referencia a estas identidades como "cuentas locales".
- Identidad **Federado**: también denominada cuenta *social* o de *empresa*, la identidad del usuario se administra mediante un proveedor de identidades federado, como Facebook, Microsoft, ADFS o Salesforce.

Un usuario con una cuenta de cliente puede iniciar sesión con varias identidades. Por ejemplo, el nombre de usuario, el correo electrónico, el Id. de empleado, el Id. oficial y otros. Una sola cuenta puede tener varias identidades, tanto locales como sociales, con la misma contraseña. 

En Microsoft Graph API, las identidades locales y federadas se almacenan en el atributo `identities` de usuario, que es de tipo [objectIdentity](/graph/api/resources/objectidentity). La colección `identities` representa un conjunto de identidades que se usan para iniciar sesión en una cuenta de usuario. Esta colección permite al usuario iniciar sesión en la cuenta de usuario con cualquiera de sus identidades asociadas. El atributo Identities puede contener hasta diez objetos [objectIdentity](/graph/api/resources/objectidentity). Cada objeto contiene las siguientes propiedades:

| Nombre   | Tipo |Descripción|
|:---------------|:--------|:----------|
|signInType|string| Especifica los tipos de inicio de sesión de usuario del directorio. Para la cuenta local: `emailAddress`, `emailAddress1`, `emailAddress2`, `emailAddress3`, `userName` o cualquier otro tipo que desee. La cuenta social debe estar establecida en `federated`.|
|issuer|string|Especifica el emisor de la identidad. En el caso de las cuentas locales (donde **signInType** no es `federated`), esta propiedad es el nombre de dominio predeterminado del inquilino de B2C local, por ejemplo `contoso.onmicrosoft.com`. En el caso de la identidad social (donde **signInType** es `federated`), el valor es el nombre del emisor; por ejemplo, `facebook.com`|
|issuerAssignedId|string|Especifica el identificador único asignado al usuario por el emisor. La combinación de **issuer** e **issuerAssignedId** debe ser única dentro del inquilino. En el caso de la cuenta local, cuando **signInType** se establece en `emailAddress` o `userName`, representa el nombre de inicio de sesión del usuario.<br>Cuando **signInType** se establece en: <ul><li>`emailAddress` (o empieza por `emailAddress` como `emailAddress1`) **issuerAssignedId** debe ser una dirección de correo electrónico válida.</li><li>`userName` (o cualquier otro valor), **issuerAssignedId** debe ser una [parte local válida de una dirección de correo electrónico](https://tools.ietf.org/html/rfc3696#section-3).</li><li>`federated`, **issuerAssignedId** representa el identificador único de la cuenta federada.</li></ul>|

El siguiente atributo **Identities**, con una identidad de cuenta local con un nombre de inicio de sesión, una dirección de correo electrónico como inicio de sesión y con una identidad social. 

 ```json
 "identities": [
     {
       "signInType": "userName",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "johnsmith"
     },
     {
       "signInType": "emailAddress",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "jsmith@yahoo.com"
     },
     {
       "signInType": "federated",
       "issuer": "facebook.com",
       "issuerAssignedId": "5eecb0cd"
     }
   ]
 ```

En las identidades federadas, en función del proveedor de identidades, **issuerAssignedId** es un valor único para un usuario determinado por aplicación o cuenta de desarrollo. Configure la directiva de Azure AD B2C con el mismo identificador de aplicación que asignó previamente el proveedor de redes sociales u otra aplicación con la misma cuenta de desarrollo. 

## <a name="password-profile-property"></a>Propiedad de perfil de contraseña

En el caso de una identidad local, se necesita el atributo **passwordProfile**, que contiene la contraseña del usuario. El atributo `forceChangePasswordNextSignIn` indica si un usuario debe restablecer la contraseña en el siguiente inicio de sesión. Para administrar un restablecimiento de contraseña forzado, [configure el flujo de restablecimiento de contraseña forzado](force-password-reset.md).

En el caso de una identidad federada (social), no se necesita el atributo **passwordProfile**.

```json
"passwordProfile" : {
    "password": "password-value",
    "forceChangePasswordNextSignIn": false
  }
```

## <a name="password-policy-attribute"></a>Atributo de directiva de contraseñas

La directiva de contraseñas de Azure AD B2C (para cuentas locales) se basa en la directiva de [seguridad de contraseña segura](../active-directory/authentication/concept-sspr-policy.md) de Azure Active Directory. Las directivas de restablecimiento de la contraseña, registro e inicio de sesión de Azure AD B2C requieren esta seguridad de contraseña segura y las contraseñas no expiran.

En los escenarios de migración de usuarios, si las cuentas que se van a migrar cuentan con una seguridad de contraseña inferior a la [seguridad de contraseña segura](../active-directory/authentication/concept-sspr-policy.md) que exige Azure AD B2C, puede deshabilitar el requisito de contraseña segura. Para cambiar la directiva de contraseñas predeterminada, establezca el atributo `passwordPolicies` en `DisableStrongPassword`. Por ejemplo, puede modificar la solicitud de creación de usuario de la siguiente manera:

```json
"passwordPolicies": "DisablePasswordExpiration, DisableStrongPassword"
```

## <a name="mfa-phone-number-attribute"></a>Atributo de número de teléfono para MFA

Cuando se usa un teléfono para autenticación multifactor (MFA), el teléfono móvil se usa para comprobar la identidad del usuario. Para [agregar](/graph/api/authentication-post-phonemethods) un número de teléfono nuevo mediante programación, [actualizar](/graph/api/b2cauthenticationmethodspolicy-update), [obtener](/graph/api/b2cauthenticationmethodspolicy-get)o [eliminar](/graph/api/phoneauthenticationmethod-delete) el número de teléfono, use el [método de autenticación de teléfono](/graph/api/resources/phoneauthenticationmethod) de Graph API de MS.

En las [directivas personalizadas](custom-policy-overview.md) de Azure AD B2C, el número de teléfono está disponible a través del tipo de notificación `strongAuthenticationPhoneNumber`.

## <a name="extension-attributes"></a>Atributos de extensión

Todas las aplicaciones orientadas al cliente tienen requisitos únicos para la información que debe recopilarse. El inquilino de Azure AD B2C incluye un conjunto integrado de información almacenada en propiedades, como el nombre propio, los apellidos y el código postal. Con Azure AD B2C, puede ampliar el conjunto de propiedades que se almacenan en cada cuenta de cliente. Para más información, consulte [Adición de atributos de usuario y personalización de entradas de usuario en Azure Active Directory B2C](configure-user-input.md).

Los atributos de extensión [extienden el esquema](/graph/extensibility-overview#schema-extensions) de los objetos de usuario en el directorio. Los atributos de extensión solo se pueden registrar en un objeto de aplicación, aunque podrían contener datos de un usuario. El atributo de extensión está conectado a la aplicación denominada `b2c-extensions-app`. No modifique esta aplicación, ya que la usa Azure AD B2C para almacenar los datos de usuario. Puede encontrar esta aplicación en los registros de aplicaciones de Azure Active Directory.

> [!NOTE]
> - Se pueden escribir hasta 100 atributos de extensión en cualquier cuenta de usuario.
> - Si se elimina la aplicación b2c-extensions-app, estos atributos de extensión se quitan de todos los usuarios, junto con los datos que contienen.
> - Si la aplicación elimina un atributo de extensión, se quita de todas las cuentas de usuario y los valores se eliminan.

Los atributos de extensión de Graph API se denominan mediante la convención `extension_ApplicationClientID_AttributeName`, donde `ApplicationClientID` es el **id. de la aplicación (cliente)** de la aplicación `b2c-extensions-app` (se encuentra en la opción **Registros de aplicaciones** > **Todas las aplicaciones** en Azure Portal). Tenga en cuenta que el **id. de la aplicación (cliente)** , tal como se representa en el nombre del atributo de extensión, no incluye guiones. Por ejemplo:

```json
"extension_831374b3bd5041bfaa54263ec9e050fc_loyaltyNumber": "212342"
```

Al definir un atributo en una extensión de esquema, se admiten los siguientes tipos de datos:

|Tipo |Observaciones  |
|--------------|---------|
|Boolean    | Valores posibles: **true** o **false**. |
|DateTime   | Debe especificarse en formato ISO 8601. Se almacenará en UTC.   |
|Entero    | Valor de 32 bits.               |
|String     | 256 caracteres como máximo.     |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los atributos de extensión:

- [Extensiones de esquema](/graph/extensibility-overview#schema-extensions)
- [Definición de atributos personalizados](user-flow-custom-attributes.md)
