---
title: Procedimiento para personalizar las credenciales verificables de Azure Active Directory (versión preliminar)
description: En este artículo se muestra cómo crear su propia credencial verificable personalizada.
services: active-directory
author: barclayn
manager: davba
ms.service: identity
ms.subservice: verifiable-credentials
ms.topic: how-to
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: a43e734c0a5bfa7c3698dcde5cb5b17f15575d90
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222935"
---
# <a name="how-to-customize-your-verifiable-credentials-preview"></a>Procedimiento para personalizar las credenciales verificables (versión preliminar)

Las credenciales verificables constan de dos componentes, los archivos de reglas y de visualización. El archivo de reglas determina lo que el usuario debe proporcionar antes de recibir una credencial verificable. El archivo de visualización controla la personalización de marca de la credencial y el estilo de las notificaciones. En esta guía, explicaremos cómo modificar ambos archivos para cumplir los requisitos de su organización. 

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="rules-file-requirements-from-the-user"></a>Archivo de reglas: requisitos del usuario

El archivo de reglas es un archivo JSON simple que describe propiedades importantes de las credenciales verificables. En concreto, describe cómo se usan las notificaciones para rellenar la credencial verificable.

Actualmente hay tres tipos de entrada que están disponibles para configurar en el archivo de reglas. El servicio de emisión de credenciales verificables usa estos tipos para insertar notificaciones en una credencial verificable y atestiguar esa información con su DID. A continuación, se muestran los tres tipos con sus correspondientes explicaciones.

- Token de id.
- Credenciales verificables a través de una presentación comprobable.
- Notificaciones con autoatestación

**Token de id.:** la aplicación de ejemplo y el tutorial usan el token de id. Cuando se configura esta opción, tendrá que proporcionar un URI de configuración de Open ID Connect e incluir las notificaciones que debe contener la credencial verificable. Se le pedirá al usuario que inicie sesión en la aplicación Authenticator para cumplir este requisito y que agregue las notificaciones asociadas de su cuenta. 

**Credenciales verificables:** el resultado final de un flujo de emisión es generar una credencial verificable, pero también puede pedir al usuario que presente una credencial verificable para emitir una. El archivo de reglas puede tomar notificaciones específicas de la credencial verificable presentada e incluirlas en la credencial verificable recién emitida de la organización. 

**Notificaciones con autoatestación:** cuando se selecciona esta opción, el usuario podrá escribir información directamente en Authenticator. En este momento, las cadenas son la única entrada admitida para las notificaciones con autoatestación. 

![Vista detallada de la tarjeta de credencial verificable](media/credential-design/issuance-doc.png) 

**Notificaciones estáticas:** además, se puede declarar una notificación estática en el archivo de reglas, pero esta entrada no procede del usuario. El emisor define una notificación estática en el archivo de reglas y tendrá un aspecto similar a cualquier otra notificación de la credencial verificable. Basta con agregar un elemento credentialSubject después de vc.type y declarar el atributo y la notificación. 

```json
"vc": {
    "type": [ "StaticClaimCredential" ],
    "credentialSubject": {
      "staticClaim": true,
      "anotherClaim": "Your Claim Here"
    },
  }
}
```


## <a name="input-type-id-token"></a>Tipo de entrada: token de id.

Para obtener el token de id. como entrada, el archivo de reglas debe configurar el punto de conexión conocido del sistema de identidad compatible con OIDC. En ese sistema, debe registrar una aplicación con la información correcta de los [ejemplos de comunicación del servicio de emisor](issuer-openid.md). Además, el elemento client_id debe colocarse en el archivo de reglas, y un parámetro de ámbito debe rellenarse con los ámbitos correctos. Por ejemplo, Azure Active Directory necesita el ámbito de correo electrónico si desea que se devuelva una notificación por correo electrónico en el token de id.
```json
    {
      "attestations": {
        "idTokens": [
          {
            "mapping": {
              "firstName": { "claim": "given_name" },
              "lastName": { "claim": "family_name" }
            },
            "configuration": "https://dIdPlayground.b2clogin.com/dIdPlayground.onmicrosoft.com/B2C_1_sisu/v2.0/.well-known/openid-configuration",
            "client_id": "8d5b446e-22b2-4e01-bb2e-9070f6b20c90",
            "redirect_uri": "vcclient://openid/",
            "scope": "openid profile"
          }
        ]
      },
      "validityInterval": 2592000,
      "vc": {
        "type": ["https://schema.org/EducationalCredential", "https://schemas.ed.gov/universityDiploma2020", "https://schemas.contoso.edu/diploma2020" ]
      }
    }
```

| Propiedad | Descripción |
| -------- | ----------- |
| `attestations.idTokens` | Una matriz de proveedores de identidades de OpenID Connect que se admiten para la información de usuario de origen. |
| `...mapping` | Un objeto que describe cómo las notificaciones de cada token de id. se asignan a los atributos de la credencial verificable resultante. |
| `...mapping.{attribute-name}` | El atributo que se debe rellenar en la credencial verificable resultante. |
| `...mapping.{attribute-name}.claim` | La notificación en los tokens de id. cuyo valor debe usarse para rellenar el atributo. |
| `...configuration` | La ubicación del documento de configuración del proveedor de identidades. Esta dirección URL debe cumplir el [estándar OpenID Connect para los metadatos del proveedor de identidades](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata). El documento de configuración debe incluir los campos `issuer`, `authorization_endpoint`, `token_endpoint` y `jwks_uri`. |
| `...client_id` | El id. de cliente obtenido durante el proceso de registro del cliente. |
| `...scope` | Una lista delimitada por espacios de ámbitos que el IDP necesita para poder devolver las notificaciones correctas en el token de id. |
| `...redirect_uri` | Siempre debe usar el valor `vcclient://openid/`. |
| `validityInterval` | El período de tiempo, en segundos, que representa la duración de las credenciales verificables. Una vez transcurrido este período de tiempo, la credencial verificable ya no será válida. Si se omite este valor, las credenciales verificables seguirán siendo válidas hasta que se revoquen explícitamente. |
| `vc.type` | Una matriz de cadenas que indica los esquemas que la credencial verificable satisface. Vea la sección siguiente para obtener más detalles. |

### <a name="vctype-choose-credential-types"></a>vc.type: elección de tipos de credenciales 

Todas las credenciales verificables deben declarar su "tipo" en el archivo de reglas. El tipo de una credencial distingue entre sus credenciales verificables y las credenciales emitidas por otras organizaciones, y garantiza la interoperabilidad entre emisores y comprobadores. Para indicar un tipo de credencial, debe proporcionar uno o varios tipos de credenciales que cumpla la credencial. Cada tipo se representa mediante una cadena única; a menudo, se usará un URI para garantizar la exclusividad global. No es necesario que el URI sea direccionable; se trata como una cadena. 

Por ejemplo, una credencial de diploma emitida por Contoso University puede declarar los siguientes tipos:

| Tipo | Propósito |
| ---- | ------- |
| `https://schema.org/EducationalCredential` | Declara que los diplomas emitidos por Contoso University contienen atributos definidos por el objeto `EducationaCredential` de schema.org. |
| `https://schemas.ed.gov/universityDiploma2020` | Declara que los diplomas emitidos por Contoso University contienen atributos definidos por el Departamento de educación de Estados Unidos. |
| `https://schemas.contoso.edu/diploma2020` | Declara que los diplomas emitidos por Contoso University contienen atributos definidos por Contoso University. |

Al declarar los tres tipos, los diplomas de Contoso University se pueden usar para satisfacer diferentes solicitudes de los comprobadores. Un banco puede solicitar un conjunto de objetos `EducationCredential` de un usuario y se puede usar el diploma para satisfacer la solicitud. Pero la Asociación de alumnos de Contoso University puede solicitar una credencial de tipo `https://schemas.contoso.edu/diploma2020`, y el diploma también satisfará la solicitud.

Para garantizar la interoperabilidad de sus credenciales, se recomienda trabajar estrechamente con las organizaciones conexas para definir los tipos de credenciales, los esquemas y los URI para su uso en su sector. Muchos organismo del sector proporcionan instrucciones sobre la estructura de los documentos oficiales que se pueden readaptar para definir el contenido de las credenciales verificables. También debe trabajar en estrecha colaboración con los comprobadores de sus credenciales para comprender cómo pretenden solicitar y consumir las credenciales verificables.

## <a name="input-type-verifiable-credential"></a>Tipo de entrada: credencial verificable

>[!NOTE]
>Los archivos de reglas que solicitan una credencial verificable no usan el formato de intercambio de presentación para solicitar credenciales. Se actualizará cuando el servicio de emisión admita el manifiesto estándar de credencial. 

```json
{
    "attestations": {
      "presentations": [
        {
          "mapping": {
            "first_name": {
              "claim": "$.vc.credentialSubject.firstName",
            },
            "last_name": {
              "claim": "$.vc.credentialSubject.lastName",
              "indexed": true
            }
          },
          "credentialType": "VerifiedCredentialNinja",
          "contracts": [
            "https://beta.did.msidentity.com/v1.0/3c32ed40-8a10-465b-8ba4-0b1e86882668/verifiableCredential/contracts/VerifiedCredentialNinja"
          ],
          "issuers": [
            {
              "iss": "did:ion:123"
            }
          ]
        }
      ]
    },
    "validityInterval": 25920000,
    "vc": {
      "type": [
        "ProofOfNinjaNinja"
      ],
    }
  }
  ```

| Propiedad | Descripción |
| -------- | ----------- |
| `attestations.presentations` | Una matriz de credenciales verificables que se solicitan como entradas. |
| `...mapping` | Un objeto que describe cómo las notificaciones de cada credencial verificable se asignan a los atributos de la credencial verificable resultante. |
| `...mapping.{attribute-name}` | El atributo que se debe rellenar en la credencial verificable resultante. |
| `...mapping.{attribute-name}.claim` | La notificación de la credencial verificable cuyo valor debe usarse para rellenar el atributo. |
| `...mapping.{attribute-name}.indexed` | Solo se puede habilitar una por cada credencial verificable que se debe guardar para la revocación. Consulte el [artículo sobre cómo revocar una credencial](how-to-issuer-revoke.md) para más información. |
| `credentialType` | El elemento credentialType de la credencial verificable que se pide al usuario que presente. |
| `contracts` | El URI del contrato en el portal del servicio de credenciales verificables. |
| `issuers.iss` | El DID del emisor de la credencial verificable que se pide del usuario. |
| `validityInterval` | El período de tiempo, en segundos, que representa la duración de las credenciales verificables. Una vez transcurrido este período de tiempo, la credencial verificable ya no será válida. Si se omite este valor, las credenciales verificables seguirán siendo válidas hasta que se revoquen explícitamente. |
| `vc.type` | Una matriz de cadenas que indica los esquemas que la credencial verificable satisface. |


## <a name="input-type-self-attested-claims"></a>Tipo de entrada: notificaciones con autoatestación

Durante el flujo de emisión, se puede pedir al usuario que escriba información con autoatestación. Actualmente, el único tipo de entrada es una "cadena". 
```json
{
  "attestations": {
    "selfIssued": {
      "mapping": {
        "alias": {
          "claim": "name"
        }
      },
    },
    "validityInterval": 25920000,
    "vc": {
      "type": [
        "ProofOfNinjaNinja"
      ],
    }
  }



```
| Propiedad | Descripción |
| -------- | ----------- |
| `attestations.selfIssued` | Una matriz de notificaciones autoemitidas que requieren entrada del usuario. |
| `...mapping` | Un objeto que describe cómo las notificaciones autoemitidas se asignan a los atributos de la credencial verificable resultante. |
| `...mapping.alias` | El atributo que se debe rellenar en la credencial verificable resultante. |
| `...mapping.alias.claim` | La notificación de la credencial verificable cuyo valor debe usarse para rellenar el atributo. |
| `validityInterval` | El período de tiempo, en segundos, que representa la duración de las credenciales verificables. Una vez transcurrido este período de tiempo, la credencial verificable ya no será válida. Si se omite este valor, las credenciales verificables seguirán siendo válidas hasta que se revoquen explícitamente. |
| `vc.type` | Una matriz de cadenas que indica los esquemas que la credencial verificable satisface. |


## <a name="display-file-verifiable-credentials-in-microsoft-authenticator"></a>Archivo de visualización: credenciales verificables en Microsoft Authenticator

Las credenciales verificables ofrecen un conjunto limitado de opciones que se pueden usar para reflejar su marca. En este artículo se proporcionan instrucciones sobre cómo personalizar las credenciales y los procedimientos recomendados para diseñar credenciales que tengan un aspecto excelente una vez emitidas a los usuarios.

Las credenciales verificables emitidas a los usuarios se muestran como tarjetas en Microsoft Authenticator. Como administrador, puede elegir el color, el icono y el texto de la tarjeta para que coincida con la marca de su organización.

![Documentación de emisión](media/credential-design/detailed-view.png) 

Las tarjetas también contienen campos personalizables que puede usar para que los usuarios sepan el propósito de la tarjeta, los atributos que contiene, etc.

## <a name="create-a-credential-display-file"></a>Creación de un archivo de visualización de credenciales

Al igual que el archivo de reglas, el archivo de visualización es un archivo JSON simple que describe cómo se debe mostrar el contenido de las credenciales verificables en la aplicación Microsoft Authenticator. 

>[!NOTE]
> En este momento, solo Microsoft Authenticator usa este modelo de presentación.

El archivo de visualización tiene la siguiente estructura.

```json
{
    "default": {
      "locale": "en-US",
      "card": {
        "title": "University Graduate",
        "issuedBy": "Contoso University",
        "backgroundColor": "#212121",
        "textColor": "#FFFFFF",
        "logo": {
          "uri": "https://contoso.edu/images/logo.png",
          "description": "Contoso University Logo"
        },
        "description": "This digital diploma is issued to students and alumni of Contoso University."
      },
      "consent": {
        "title": "Do you want to get your digital diploma from Contoso U?",
        "instructions": "Please log in with your Contoso U account to receive your digital diploma."
      },
      "claims": {
        "vc.credentialSubject.name": {
          "type": "String",
          "label": "Name"
        }
      }
    }
}
```

| Propiedad | Descripción |
| -------- | ----------- |
| `locale` | El idioma de la credencial verificable. Reservado para uso futuro. | 
| `card.title` | Muestra el tipo de credencial al usuario. La longitud máxima recomendada es de 25 caracteres. | 
| `card.issuedBy` | Muestra el nombre de la organización emisora al usuario. La longitud máxima recomendada es de 40 caracteres. |
| `card.backgroundColor` | Determina el color de fondo de la tarjeta, en formato hexadecimal. Se aplicará un gradiente sutil a todas las tarjetas. |
| `card.textColor` | Determina el color del texto de la tarjeta, en formato hexadecimal. Se recomienda usar negro o blanco. |
| `card.logo` | Un logotipo que se muestra en la tarjeta. La dirección URL proporcionada debe ser direccionable públicamente. El alto máximo recomendado es 36 px y el ancho máximo es 100 px, independientemente del tamaño del teléfono. Se recomienda PNG con fondo transparente. | 
| `card.description` | Texto complementario que se muestra junto a cada tarjeta. Se puede usar para cualquier fin. La longitud máxima recomendada es de 100 caracteres. |
| `consent.title` | Texto complementario que se muestra cuando se emite una tarjeta. Se utiliza para proporcionar detalles sobre el proceso de emisión. La longitud recomendada es de 100 caracteres. |
| `consent.instructions` | Texto complementario que se muestra cuando se emite una tarjeta. Se utiliza para proporcionar detalles sobre el proceso de emisión. La longitud recomendada es de 100 caracteres. |
| `claims` | Permite proporcionar etiquetas para los atributos incluidos en cada credencial. |
| `claims.{attribute}` | Indica el atributo de la credencial a la que se aplica la etiqueta. |
| `claims.{attribute}.type` | Indica el tipo de atributo. Actualmente solo admitimos "cadena". |
| `claims.{attribute}.label` | El valor que se debe usar como etiqueta para el atributo, que se mostrará en Authenticator. Puede ser diferente de la etiqueta que se usó en el archivo de reglas. La longitud máxima recomendada es de 40 caracteres. |

>[!NOTE]
  >Si se incluye una notificación en el archivo de reglas y, a continuación, se omite en el archivo de visualización, hay dos tipos diferentes de experiencias. En iOS, la notificación no aparecerá en la sección de detalles que se muestra en la imagen anterior, mientras que en Android sí se mostrará.  

## <a name="next-steps"></a>Pasos siguientes

Ahora tiene una mejor comprensión del diseño de las credenciales verificables y de cómo puede crear las suyas propias para satisfacer sus necesidades.

- [Ejemplos de comunicación del servicio emisor](issuer-openid.md)
