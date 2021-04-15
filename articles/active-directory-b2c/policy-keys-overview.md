---
title: 'Información general sobre las claves de directiva: Azure Active Directory B2C'
description: Obtenga información sobre los tipos de claves de directiva de cifrado que se pueden usar en Azure Active Directory B2C para firmar y validar tokens, secretos de cliente, certificados y contraseñas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 37ff3300935e90304754b5f2b375df0adecdaefa
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256249"
---
# <a name="overview-of-policy-keys-in-azure-active-directory-b2c"></a>Introducción a las claves de directiva en Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) almacena secretos y certificados en forma de claves de directiva para establecer la confianza con los servicios con los que se integra. Estas confianzas constan de los siguientes componentes:

- Proveedores de identidades externos
- Conexión con los [servicios de API REST](restful-technical-profile.md)
- Firma y cifrado de tokens

 En este artículo se explica lo que necesita saber sobre las claves de directiva que usa Azure AD B2C.

> [!NOTE]
> Actualmente, la configuración de las claves de directiva solo se limita a las [directivas personalizadas](./user-flow-overview.md).

Puede configurar secretos y certificados para establecer la confianza entre servicios en Azure Portal, en el menú **Claves de directiva**. Las claves pueden ser simétricas o asimétricas. La criptografía *simétrica*, o criptografía de clave privada, es donde se usa un secreto compartido para cifrar y descifrar los datos. La criptografía *asimétrica*, o la criptografía de clave pública, es un sistema de cifrado que usa pares de claves, que se componen de claves públicas que se comparten con la aplicación de usuario de confianza y las claves privadas que solo conoce Azure AD B2C.

## <a name="policy-keyset-and-keys"></a>Claves y conjunto de claves de directiva

El recurso de nivel superior de las claves de directiva en Azure AD B2C es el contenedor **Keyset**. Cada conjunto de claves contiene al menos una **clave**. Una clave tiene los siguientes atributos:

| Atributo |  Obligatorio | Comentarios |
| --- | --- |--- |
| `use` | Sí | Uso: Identificar el uso previsto de la clave pública Cifrar los datos `enc` o comprobar la firma en datos `sig`|
| `nbf`| No | Fecha y hora de activación. |
| `exp`| No | Fecha y hora de expiración. |

Se recomienda establecer los valores de activación y expiración de claves de acuerdo con los estándares de PKI. Puede que necesite rotar esos certificados por motivos de seguridad o de directivas. Por ejemplo, puede establecer una directiva para rotar todos los certificados cada año.

Para crear una clave, puede elegir uno de los métodos siguientes:

- **Manual**: cree un secreto con una cadena que defina. El secreto es una clave simétrica. Puede establecer las fechas de activación y expiración.
- **Generado**: se genera automáticamente una clave. Puede establecer fechas de activación y de expiración. Hay dos opciones:
  - **Secreto**: genera una clave simétrica.
  - **RSA**: genera un par de claves (claves asimétricas).
- **Cargar**: cargue un certificado o una clave PKCS12. El certificado debe contener las claves públicas y privadas (claves asimétricas).

## <a name="key-rollover"></a>Sustitución de claves

Por motivos de seguridad, Azure AD B2C puede sustituir las claves periódicamente, o inmediatamente en caso de emergencia. Cualquier aplicación, proveedor de identidades o API REST que se integra con Azure AD B2C debe estar preparada para controlar un evento de sustitución de claves, con independencia de la frecuencia con que se produzca. De lo contrario, si la aplicación o Azure AD B2C intenta usar una clave expirada para realizar una operación criptográfica, se producirá un error en la solicitud de inicio de sesión.

Si un conjunto de claves de Azure AD B2C tiene varias claves, solo una de ellas está activa al mismo tiempo, en función de los criterios siguientes:

- La activación de claves se basa en la **fecha de activación**.
  - Las claves se ordenan por fecha de activación en orden ascendente. Las claves con fechas de activación en el futuro aparecen en la parte inferior de la lista. Las claves sin una fecha de activación se encuentran en la parte inferior de la lista.
  - Cuando la fecha y hora actuales es mayor que la fecha de activación de una clave, Azure AD B2C activará la clave y dejará de usar la clave activa anterior.
- Cuando ha transcurrido el tiempo de expiración de la clave actual y el contenedor de claves contiene una nueva clave con tiempos *no anteriores a una fecha* y de *expiración* válidos, la nueva clave se activará automáticamente.
- Cuando ha transcurrido el tiempo de expiración de la clave actual y el contenedor de claves *no* contiene una nueva clave con *tiempos no anteriores a una fecha* y de *expiración* válidos, Azure AD B2C no podrá usar la clave expirada. Azure AD B2C generará un mensaje de error en un componente dependiente de la directiva personalizada. Para evitar este problema, puede crear una clave predeterminada sin fechas de activación ni expiración a modo de red de seguridad.
- El punto de conexión de la clave (URI de JWKS) del punto de conexión de configuración conocido de OpenId Connect refleja las claves configuradas en el contenedor de claves, cuando se hace referencia a la clave en el [perfil técnico de JwtIssuer](./jwt-issuer-technical-profile.md). Una aplicación que use una biblioteca OIDC capturará automáticamente estos metadatos para asegurarse de que use las claves correctas para validar los tokens. Para obtener más información, aprenda a usar la [Biblioteca de autenticación de Microsoft](../active-directory/develop/msal-b2c-overview.md), que siempre captura las claves de firma de tokens más recientes automáticamente.

## <a name="policy-key-management"></a>Administración de claves de directiva

Para obtener la clave activa actual dentro de un contenedor de claves, use el punto de conexión [getActiveKey](/graph/api/trustframeworkkeyset-getactivekey) de Microsoft Graph API.

Para agregar o eliminar claves de cifrado y firma, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En la página de información general, en **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva**. 
    1. Para agregar una nueva clave, seleccione **Agregar**.
    1. Para quitar una nueva clave, seleccione la clave y, a continuación, seleccione **Eliminar**. Para eliminar la clave, escriba el nombre del contenedor de claves que se va a eliminar. Azure AD B2C eliminará la clave y creará una copia de la clave con el sufijo .bak.

### <a name="replace-a-key"></a>Reemplazo de una clave

Las claves de un conjunto de claves no se pueden reemplazar ni quitar. Si tiene que cambiar una clave existente:

- Se recomienda agregar una nueva clave con la **fecha de activación** establecida en la fecha y hora actuales. Azure AD B2C activará la nueva clave y dejará de usar la clave activa anterior.
- También puede crear un conjunto de claves con las claves correctas. Actualice la directiva para usar el nuevo conjunto de claves y, después, quite el conjunto de claves anterior. 

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo usar Microsoft Graph para automatizar una implementación de [claves de directiva](microsoft-graph-operations.md#trust-framework-policy-key) y [conjuntos de claves](microsoft-graph-operations.md#trust-framework-policy-keyset).