---
title: TypingDNA con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar la autenticación de Azure AD B2C con TypingDNA para ayudar a la revisión y verificación de identidad basadas en el patrón de escritura del usuario, ya que proporciona soluciones de verificación de identidad que fuerzan la autenticación multifactor y ayudan a cumplir con los requisitos de SCA de la directiva de servicios de pago 2 (PSD2).
author: gargi-sinha
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: afaac48d263082a17c7ba58996f4000366de0211
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256555"
---
# <a name="tutorial-for-configuring-typingdna-with-azure-active-directory-b2c"></a>Tutorial para configurar TypingDNA con Azure Active Directory B2C

En este tutorial, aprenderá a integrar una aplicación de pago en línea de ejemplo en Azure Active Directory B2C con la aplicación TypingDNA. Mediante el uso de la aplicación TypingDNA, los clientes de Azure AD B2C pueden cumplir los requisitos para las transacciones de la [Directiva de servicios de pago 2](https://www.typingdna.com/use-cases/sca-strong-customer-authentication) (PSD2) a través de una dinámica de pulsaciones de teclas y la sólida autenticación del cliente. Obtenga más información sobre TypingDNA [aquí](https://www.typingdna.com/).

 Azure AD B2C usa las tecnologías de TypingDNA para capturar las características de escritura de los usuarios, y luego grabarlas y analizarlas para familiarizarse con cada autenticación. De este modo se agrega una capa de protección frente a los riesgos de una autenticación y se evalúan los niveles de los mismos. Azure AD B2C puede recurrir a otros mecanismos para aumentar la confianza de que el usuario es quien dice ser: invocar MFA de Azure AD, forzar la verificación del correo electrónico o cualquier otra lógica personalizada para el escenario.

>[!NOTE]
> Esta directiva de ejemplo se basa en el paquete de inicio [SocialAndLocalAccountsWithMfa](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccountsWithMfa).

## <a name="scenario-description"></a>Descripción del escenario

![Captura de pantalla del diagrama de la arquitectura de TypingDNA](./media/partner-typingdna/typingdna-architecture-diagram.png)

### <a name="sign-up"></a>Registro

1. Las páginas de Azure AD B2C usan la biblioteca JavaScript de TypingDNA para grabar el patrón de escritura del usuario. Por ejemplo, el nombre de usuario y la contraseña se graban al realizar el registro en la inscripción inicial y luego en cada inicio de sesión para la comprobación.

2. Cuando el usuario envía la página, la biblioteca de TypingDNA procesará las características de la escritura del usuario. Después, inserte la información en un campo de texto oculto que Azure AD B2C ha representado. Este campo se oculta con CSS.  

    El [ejemplo contiene archivos HTML](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignUp.cshtml) con las modificaciones de JavaScript y CSS, y se hace referencia a ellos en las definiciones de contenido de `api.selfasserted.tdnasignin` y `api.selfasserted.tdnasignup`. Consulte [Hospedaje del contenido de la página](./customize-ui-with-html.md#hosting-the-page-content) para hospedar los archivos HTML.

3. Azure AD B2C tiene ahora el patrón de escritura en el contenedor de notificaciones cuando el usuario envía sus credenciales. Debe llamar a una API (suya) para pasar estos datos al punto de conexión de la API de REST de TypingDNA. Esta API se incluye en el [ejemplo (typingDNA-API-Interface)](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
4. A continuación, la API de capa intermedia pasa los datos del patrón de escritura a la API de REST de TypingDNA. Al registrarse, se llama al punto de conexión de [comprobar usuario](https://api.typingdna.com/index.html#api-API_Services-GetUser) para confirmar que el usuario no existe y, a continuación, se llama al punto de conexión de [guardar patrón](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern) para guardar el primer patrón de escritura del usuario.

> [!NOTE]
> Todas las llamadas a puntos de conexión de la API de REST de TypingDNA envían un elemento UserId. Debe ser un valor con hash. Azure AD B2C usa la transformación de notificaciones de `HashObjectIdWithEmail` para aplicar un algoritmo hash al correo electrónico con sal y secreto aleatorios.  

Las llamadas a la API de REST se modelan con `validationTechnicalProfiles` dentro de `LocalAccountSignUpWithLogonEmail-TDNA`:

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail-TDNA" />
    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="true"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser"/>

</ValidationTechnicalProfiles>

```

### <a name="sign-in"></a>Inicio de sesión

En los inicios de sesión posteriores, el patrón de escritura del usuario se procesa de la misma manera que al registrarse con [HTML personalizado](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignIn.cshtml). Una vez que el perfil de escritura se encuentra dentro del contenedor de notificaciones de Azure AD B2C, Azure AD B2C llamará a la API para llamar al punto de conexión de la API de REST de TypingDNA. Se llama al punto de conexión de [comprobar usuario](https://api.typingdna.com/index.html#api-API_Services-GetUser) para confirmar que el usuario existe. Después, se llama al punto de conexión de [comprobar patrón](https://api.typingdna.com/index.html#api-API_Services-verifyTypingPattern) para devolver el elemento `net_score`. Este elemento `net_score` es una indicación de la proximidad del patrón de escritura al registro original.

Este patrón de escritura se modela con `validationTechnicalProfiles` dentro de `SelfAsserted-LocalAccountSignin-Email-TDNA`:

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="login-NonInteractive"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="false"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-VerifyUser"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser">

        <Preconditions>

            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>saveTypingPattern</Value>

            <Value>False</Value>

            <Action>SkipThisValidationTechnicalProfile</Action>

            </Precondition>

        </Preconditions>

    </ValidationTechnicalProfile>

</ValidationTechnicalProfiles>

```

 Si el usuario obtiene un patrón de escritura con un valor alto de `net_score`, puede guardarlo mediante el punto de conexión de [guardar patrón de escritura](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern).  

La API debe devolver una notificación `saveTypingPattern` si desea que el punto de conexión de guardar patrón de escritura de TypingDNA se llame mediante Azure AD B2C (a través de la API).

El ejemplo del repositorio contiene una API (TypingDNA-API-Interface) que está configurada con las siguientes propiedades.

- Modo de entrenamiento: si el usuario tiene menos de dos patrones guardados, solicitar siempre MFA.

- Si el usuario tiene de 2 a 5 patrones guardados y el valor de `net_score` es inferior a 50, solicitar MFA.

- Si el usuario tiene más de 5 patrones guardados y el valor de `net_score` es inferior a 65, solicitar MFA.

Estos umbrales deben ajustarse al caso de uso.

- Una vez que la API haya evaluado el elemento `net_score`, debe devolver una notificación booleana a B2C, `promptMFA`.

- La notificación `promptMFA` se usa dentro de una condición previa para ejecutar de forma condicional MFA de Azure AD.

```xml

<OrchestrationStep Order="3" Type="ClaimsExchange">

    <Preconditions>

        <Precondition Type="ClaimsExist" ExecuteActionsIf="true">

            <Value>isActiveMFASession</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>promptMFA</Value>

            <Value>False</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

    </Preconditions>

    <ClaimsExchanges>

        <ClaimsExchange Id="PhoneFactor-Verify" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" />

    </ClaimsExchanges>

</OrchestrationStep>

```

## <a name="onboard-with-typingdna"></a>Incorporación a TypingDNA

1. Regístrese en TypingDNA [aquí](https://www.typingdna.com/)
2. Inicie sesión en el panel de TypingDNA y obtenga la **clave de API** y el **secreto de API**. Se necesitarán en la configuración de la interfaz de API más adelante.

## <a name="integrate-typingdna-with-azure-ad-b2c"></a>Integración de TypingDNA con Azure AD B2C

1. Hospede [TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) en el proveedor de hospedaje que prefiera.
2. Reemplace todas las instancias de `apiKey` y `apiSecret` en la solución [TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) por las credenciales del panel de TypingDNA.
3. Hospede los archivos HTML en el proveedor que prefiera siguiendo los requisitos de CORS [aquí](./customize-ui-with-html.md#3-configure-cors)
4. Reemplace los elementos LoadURI de las definiciones de contenido `api.selfasserted.tdnasignup` y `api.selfasserted.tdnasignin` en el archivo `TrustFrameworkExtensions.xml` por el URI de los archivos HTML hospedados, respectivamente.
5. Cree una clave de directiva B2C en el marco de la experiencia de identidad en la hoja de Azure AD de **Azure Portal**. Use la opción `Generate` y asigne a esta clave el nombre `tdnaHashedId`.
6. Reemplazo de TenantId en los archivos de directiva
7. Reemplace los elementos ServiceURL en todos los perfiles técnicos de la API de REST de TypingDNA (REST-TDNA-VerifyUser, REST-TDNA-SaveUser, REST-TDNA-CheckUser) por el punto de conexión de la [API TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
8. Cargue los [archivos de directiva](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/policy) en el inquilino.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de B2C y elija el marco de la experiencia de identidad.
2. Seleccione el **flujo de usuario** que creó anteriormente.
3. Seleccione **Ejecutar** el flujo de usuario.

    a. **Aplicación**: seleccione la aplicación registrada (en el ejemplo es JWT).

    b. **URL de respuesta**: seleccione la dirección URL de redireccionamiento.

    c. Seleccione **Ejecutar flujo de usuario**.
  
4. Recorra el flujo de registro y cree una cuenta.
5. Cerrar sesión
6. Recorra el flujo de inicio de sesión.
7. El JWT resultante mostrará los resultados de TypingDNA.

## <a name="live-version"></a>Versión en directo

• MFA se ha deshabilitado en esta versión de prueba, pero puede ver el resultado de si se hubiera solicitado MFA por la notificación `promptMFA` después de la autenticación.

• Regístrese [aquí](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SU_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login) e inicie sesión [aquí](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SI_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de AAD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en AAD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)