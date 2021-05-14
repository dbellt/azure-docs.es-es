---
title: Tutorial para configurar Azure Active Directory B2C con BioCatch
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con BioCatch para identificar usuarios peligrosos y fraudulentos
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/20/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: f9518466aacddee9e31d8bc15f3b89c1f214ab58
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108738006"
---
# <a name="tutorial-configure-biocatch-with-azure-active-directory-b2c"></a>Tutorial: Configuración Azure Active Directory B2C con BioCatch

En este tutorial de ejemplo, aprenderá a integrar la autenticación de Azure Active Directory (AD) B2C con [BioCatch](https://www.biocatch.com/) para mejorar aún más la posición de seguridad de la administración de identidades y acceso de clientes (CIAM). BioCatch analiza los comportamientos digitales físicos y cognitivos de un usuario para generar información que distinga entre clientes legítimos y ciberdelincuentes.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- Una cuenta de [BioCatch](https://www.biocatch.com/contact-us).

## <a name="scenario-description"></a>Descripción del escenario

La integración de BioCatch incluye los siguientes componentes:

- **Una aplicación o servicio web**: el usuario navega primero a este servicio web. Este servicio web crea una instancia de un identificador de sesión de cliente único que se envía a BioCatch. A continuación, el identificador de sesión de cliente comienza inmediatamente a transmitir las características de comportamiento del usuario a BioCatch.

- **Un método**: envía el identificador de sesión de cliente único a Azure AD B2C. En el ejemplo proporcionado, JavaScript se usa para introducir el valor en un campo HTML oculto.

- **Una interfaz personalizada de Azure AD B2C**: oculta un campo HTML para la entrada del identificador de sesión de cliente desde JavaScript, si se usa el método anterior.

- **Directiva personalizada de Azure AD B2C**

  - Toma el identificador de sesión de cliente personalizado de la interfaz de usuario en forma de notificación. Esto se logra a través de un perfil técnico autoafirmado.

  - Se integra con BioCatch mediante un proveedor de notificaciones de API REST y pasa el identificador de sesión de cliente a la plataforma BioCatch.

  - Se devuelven varias notificaciones personalizadas desde BioCatch para que la lógica de la directiva personalizada actúe en consecuencia.

  - Un elemento userjourney, que evalúa una notificación devuelta; por ejemplo, un riesgo de sesión, y ejecuta condicionalmente una acción, como invocar la autenticación multifactor (MFA).

![Diagrama de la arquitectura de BioCatch.](media/partner-biocatch/biocatch-architecture-diagram.png)

| Paso  | Descripción |
|:---|:-----------------------|
|1a     | El usuario navega al servicio web. A continuación, el servicio web devuelve valores HTML, CSS o JavaScript y se configura para cargar el SDK de JavaScript de BioCatch. JavaScript del lado cliente configura o establece el identificador de sesión de cliente para el SDK de BioCatch. Como alternativa, el servicio web puede configurar previamente el identificador de sesión de cliente y enviarlo al cliente.        |
|1b    |  Configure el SDK de JavaScript de BioCatch con instancias en la plataforma BioCatch. Comience inmediatamente a enviar características de comportamiento del usuario a BioCatch desde el dispositivo cliente, mediante el identificador de sesión de cliente del paso 1a.    |
|2     |  El usuario intenta registrarse o iniciar sesión y se le redirige a Azure AD B2C.      |
|3a    | Parte del elemento userjourney es un proveedor de notificaciones autoafirmado, que toma el identificador de sesión de cliente como entrada. Este campo está oculto en la pantalla. Puede usar JavaScript para introducir el identificador de sesión en el campo. Seleccione el botón *Siguiente* para continuar con el proceso de registro o inicio de sesión.|
|3b     | El identificador de sesión de cliente se envía a la plataforma BioCatch para determinar una puntuación de riesgo. |
|3c     | BioCatch devuelve información sobre la sesión, como la puntuación de riesgo, y una recomendación sobre qué hacer: permitir o bloquear. |
|3d    |El elemento userjourney tiene un paso de comprobación condicional, que actúa sobre las notificaciones devueltas.|
| 4   | En función del resultado de la comprobación condicional, se invoca una acción como *Requerir autenticación para actualización a edición superior*.|
|5    | En cualquier momento desde que el usuario llega por primera vez a la página del servicio web, este puede usar el identificador de sesión de cliente para consultar la API de BioCatch con el objetivo de determinar la puntuación de riesgo y la información de sesión en tiempo real. |

## <a name="onboard-with-biocatch"></a>Incorporación con BioCatch

Póngase en contacto con [BioCatch](https://www.biocatch.com/contact-us) y cree una cuenta.

## <a name="configure-the-custom-ui"></a>Configuración de la interfaz de usuario personalizada

Se recomienda ocultar el campo de identificador de sesión de cliente. Use CSS, JavaScript o cualquier otro método para ocultar el campo. Con fines de pruebas, puede mostrar el campo. Por ejemplo, JavaScript se usa para ocultar el campo de entrada como:

```JavaScript
document.getElementById("clientSessionId").style.display = 'none';
```

## <a name="configure--azure-ad-b2c-identity-experience-framework-policies"></a>Configuración de las directivas de Identity Experience Framework de Azure AD B2C

1. Establezca la [configuración de directiva personalizada](./tutorial-create-user-flows.md?pivots=b2c-custom-policy).

2. Cree un archivo que herede del archivo de extensiones.

    ```XML
    <BasePolicy> 

        <TenantId>tenant.onmicrosoft.com</TenantId> 

        <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId> 

      </BasePolicy> 
    ```

3. Cree una referencia a la interfaz de usuario personalizada para ocultar el cuadro de entrada, en el recurso BuildingBlocks.

    ```XML
    <ContentDefinitions> 

        <ContentDefinition Id="api.selfasserted"> 

            <LoadUri>https://domain.com/path/to/selfAsserted.cshtml</LoadUri> 

            <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri> 

          </ContentDefinition> 

        </ContentDefinitions>
    ```

4. Agregue las siguientes notificaciones en el recurso BuildingBlocks.

    ```XML
    <ClaimsSchema> 

          <ClaimType Id="riskLevel"> 

            <DisplayName>Session risk level</DisplayName> 

            <DataType>string</DataType>       

          </ClaimType> 

          <ClaimType Id="score"> 

            <DisplayName>Session risk score</DisplayName> 

            <DataType>int</DataType>       

          </ClaimType> 

          <ClaimType Id="clientSessionId"> 

            <DisplayName>The ID of the client session</DisplayName> 

            <DataType>string</DataType> 

            <UserInputType>TextBox</UserInputType> 

          </ClaimType> 

    <ClaimsSchema> 
    ```

5. Configure el proveedor de notificaciones autoafirmado para el campo de identificador de sesión de cliente.

    ```XML
    <ClaimsProvider> 

          <DisplayName>Client Session ID Claims Provider</DisplayName> 

          <TechnicalProfiles> 

            <TechnicalProfile Id="login-NonInteractive-clientSessionId"> 

              <DisplayName>Client Session ID TP</DisplayName> 

              <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" /> 

              <Metadata> 

                <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item> 

              </Metadata> 

              <CryptographicKeys> 

                <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" /> 

              </CryptographicKeys> 

            <!—Claim we created earlier --> 

              <OutputClaims> 

                <OutputClaim ClaimTypeReferenceId="clientSessionId" Required="false" DefaultValue="100"/> 

              </OutputClaims> 

            <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" /> 

            </TechnicalProfile> 

          </TechnicalProfiles> 

        </ClaimsProvider> 
    ```

6. Configure el proveedor de notificaciones de la API REST para BioCatch. 

    ```XML
    <TechnicalProfile Id="BioCatch-API-GETSCORE"> 

          <DisplayName>Technical profile for BioCatch API to return session information</DisplayName> 

          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" /> 

          <Metadata> 

            <Item Key="ServiceUrl">https://biocatch-url.com/api/v6/score?customerID=<customerid>&amp;action=getScore&amp;uuid=<uuid>&amp;customerSessionID={clientSessionId}&amp;solution=ATO&amp;activtyType=<activity_type>&amp;brand=<brand></Item> 

            <Item Key="SendClaimsIn">Url</Item> 

            <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item> 

            <!-- Set AuthenticationType to Basic or ClientCertificate in production environments --> 

            <Item Key="AuthenticationType">None</Item> 

            <!-- REMOVE the following line in production environments --> 

            <Item Key="AllowInsecureAuthInProduction">true</Item> 

          </Metadata> 

          <InputClaims> 

            <InputClaim ClaimTypeReferenceId="clientsessionId" /> 

          </InputClaims> 

          <OutputClaims> 

            <OutputClaim ClaimTypeReferenceId="riskLevel" /> 

            <OutputClaim ClaimTypeReferenceId="score" /> 

          </OutputClaims> 

          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" /> 

        </TechnicalProfile> 

      </TechnicalProfiles>
    ```

    > [!Note]
    > BioCatch proporcionará la dirección URL, el identificador de cliente y el identificador de usuario único (uuID) que se van a configurar. La notificación SessionID del cliente se pasa como un parámetro querystring a BioCatch. Puede elegir el tipo de actividad; por ejemplo, *MAKE_PAYMENT*.

7. Configure el elemento userjourney y siga el ejemplo.

   1. Obtenga el valor de clientSessionID como una notificación.

   1. Llame a la API de BioCatch para obtener la información de la sesión.

   1. Si el valor de *risk* (riesgo) de la notificación devuelta es *low* (bajo), omita el paso de MFA; de lo contrario, fuerce la autenticación multifactor para los usuarios.

    ```XML
    <OrchestrationStep Order="8" Type="ClaimsExchange"> 

              <ClaimsExchanges> 

                <ClaimsExchange Id="clientSessionIdInput" TechnicalProfileReferenceId="login-NonInteractive-clientSessionId" /> 

              </ClaimsExchanges> 

            </OrchestrationStep> 

            <OrchestrationStep Order="9" Type="ClaimsExchange"> 

              <ClaimsExchanges> 

                <ClaimsExchange Id="BcGetScore" TechnicalProfileReferenceId=" BioCatch-API-GETSCORE" /> 

              </ClaimsExchanges> 

            </OrchestrationStep> 

            <OrchestrationStep Order="10" Type="ClaimsExchange"> 

              <Preconditions> 

                <Precondition Type="ClaimEquals" ExecuteActionsIf="true"> 

                  <Value>riskLevel</Value> 

                  <Value>LOW</Value> 

                  <Action>SkipThisOrchestrationStep</Action> 

                </Precondition> 

              </Preconditions> 

              <ClaimsExchanges> 

                <ClaimsExchange Id="PhoneFactor-Verify" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" /> 

              </ClaimsExchanges>  

    ```

8. Configure el usuario de confianza (opcional).

    Resulta útil pasar la información devuelta por BioCatch a la aplicación como notificaciones en el token, específicamente *riskLevel* y *score*.

    ```XML
    <RelyingParty> 

        <DefaultUserJourney ReferenceId="SignUpOrSignInMfa" /> 

        <UserJourneyBehaviors> 

          <SingleSignOn Scope="Tenant" KeepAliveInDays="30" /> 

          <SessionExpiryType>Absolute</SessionExpiryType> 

          <SessionExpiryInSeconds>1200</SessionExpiryInSeconds> 

          <ScriptExecution>Allow</ScriptExecution> 

        </UserJourneyBehaviors> 

        <TechnicalProfile Id="PolicyProfile"> 

          <DisplayName>PolicyProfile</DisplayName> 

          <Protocol Name="OpenIdConnect" /> 

          <OutputClaims> 

            <OutputClaim ClaimTypeReferenceId="displayName" /> 

            <OutputClaim ClaimTypeReferenceId="givenName" /> 

            <OutputClaim ClaimTypeReferenceId="surname" /> 

            <OutputClaim ClaimTypeReferenceId="email" /> 

            <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" /> 

            <OutputClaim ClaimTypeReferenceId="identityProvider" />                 

            <OutputClaim ClaimTypeReferenceId="riskLevel" /> 

            <OutputClaim ClaimTypeReferenceId="score" /> 

            <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" /> 

          </OutputClaims> 

          <SubjectNamingInfo ClaimType="sub" /> 

        </TechnicalProfile> 

      </RelyingParty> 

    ```

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

Siga estos pasos para agregar los archivos de directivas a Azure AD B2C.

1. Inicie sesión en  [**Azure Portal**](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.

2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio + suscripción**  del menú superior y elija el directorio que contiene el inquilino.

3. Elija  **Todos los servicios**  en la esquina superior izquierda de Azure Portal, y busque y seleccione Azure AD B2C.

4. Vaya a  **Azure AD B2C** > **Identity Experience Framework**.

3. Cargue todos los archivos de directiva en el inquilino.

## <a name="test-the-solution"></a>Probar la solución

1. [Registre una aplicación ficticia que redirija a JWT.MS](./tutorial-register-applications.md?tabs=app-reg-ga).  

2. En **Identity Experience Framework**, seleccione la directiva que creó.

3. En la ventana de la directiva, seleccione la aplicación ficticia JWT.MS y elija **Ejecutar ahora**.

4. Recorra el flujo de registro y cree una cuenta. El token devuelto a JWT.MS debería tener dos notificaciones para riskLevel y score. Siga el ejemplo.  

    ```JavaScript
    { 

      "typ": "JWT", 

      "alg": "RS256", 

      "kid": "_keyid" 

    }.{ 

      "exp": 1615872580, 

      "nbf": 1615868980, 

      "ver": "1.0", 

      "iss": "https://tenant.b2clogin.com/12345678-1234-1234-1234-123456789012/v2.0/", 

      "sub": "12345678-1234-1234-1234-123456789012", 

      "aud": "12345678-1234-1234-1234-123456789012", 

      "acr": "b2c_1a_signup_signin_biocatch_policy", 

      "nonce": "defaultNonce", 

      "iat": 1615868980, 

      "auth_time": 1615868980, 

      "name": "John Smith", 

      "email": "john.smith@contoso.com", 

      "given_name": "John", 

      "family_name": "Smith", 

      "riskLevel": "LOW", 

      "score": 275, 

      "tid": "12345678-1234-1234-1234-123456789012" 

    }.[Signature]  

    ```

## <a name="additional-resources"></a>Recursos adicionales

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy&tabs=applications)
