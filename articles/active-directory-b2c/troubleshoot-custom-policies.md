---
title: Solución de problemas de directivas personalizadas en Azure Active Directory B2C
description: Información sobre los métodos para resolver errores al trabajar con directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/25/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6298c8483c44472fe6f52f3e48b5c529c2d978a5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110457499"
---
# <a name="troubleshoot-azure-ad-b2c-custom-policies"></a>Solución de problemas de directivas personalizadas de Azure AD B2C

Si usa [directivas personalizadas](custom-policy-overview.md) de Azure Active Directory B2C (Azure AD B2C), es posible que experimente desafíos con el formato XML del lenguaje de directivas o problemas en tiempo de ejecución. En este artículo se describen algunas herramientas y sugerencias que pueden ayudarle a detectar y solucionar problemas. 

Este artículo se centra en la solución de problemas de configuración de directivas personalizadas de Azure AD B2C. No trata sobre las aplicaciones de usuarios de confianza ni sobre su biblioteca de identidades.

## <a name="azure-ad-b2c-correlation-id-overview"></a>Información general del identificador de correlación de Azure AD B2C

El identificador de correlación de Azure AD B2C es un valor de identificador único que se adjunta a las solicitudes de autorización. Pasa por todos los pasos de orquestación que toma un usuario. Con el identificador de correlación, puede:

- Identificar la actividad de inicio de sesión en la aplicación y realizar un seguimiento del rendimiento de la directiva.
- Buscar los registros de Azure Application Insights de la solicitud de inicio de sesión.
- Pasar el identificador de correlación a la API REST y utilizarlo para identificar el flujo de inicio de sesión. 

El identificador de correlación se cambia cada vez que se establece una nueva sesión. Al depurar las directivas, asegúrese de cerrar las pestañas del explorador existentes. O abra un nuevo explorador en modo privado.

### <a name="get-the-azure-ad-b2c-correlation-id"></a>Obtención del identificador de correlación de Azure AD B2C

Puede encontrar el identificador de correlación en la página de registro o inicio de sesión de Azure AD B2C. En el explorador, seleccione **Ver código fuente**. La correlación aparece como un comentario en la parte superior de la página.

![Captura de pantalla de la vista de código fuente de la página de inicio de sesión de Azure AD B2C.](./media/troubleshoot-custom-policies/find-azure-ad-b2c-correlation-id.png)

Copie el identificador de correlación y, a continuación, continúe con el flujo de inicio de sesión. Use el identificador de correlación para observar el comportamiento de inicio de sesión. Para más información, consulte [Solución de problemas con Application Insights](#troubleshooting-with-application-insights).

### <a name="echo-the-azure-ad-b2c-correlation-id"></a>Visualización del identificador de correlación de Azure AD B2C

Puede incluir el identificador de correlación en los tokens de Azure AD B2C. Para incluir el identificador de correlación:

1. Abra el archivo de extensiones de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Busque el elemento [BuildingBlocks](buildingblocks.md). Si el elemento no existe, agréguelo.
1. Busque el elemento [ClaimsSchema](claimsschema.md). Si el elemento no existe, agréguelo.
1. Agregue la notificación Id. de correlación al elemento **ClaimsSchema**.  

    ```xml
    <!-- 
    <BuildingBlocks>
      <ClaimsSchema> -->
        <ClaimType Id="correlationId">
          <DisplayName>correlation ID</DisplayName>
          <DataType>string</DataType>
        </ClaimType>
      <!-- 
      </ClaimsSchema>
    </BuildingBlocks>-->
    ```

1. Abra el archivo de usuario de confianza de la directiva. Por ejemplo, el archivo <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. La notificación de salida se agregará al token después de un recorrido del usuario correcto y se envía a la aplicación. Modifique el elemento de perfil técnico en la sección de usuario de confianza para agregar `correlationId` como una notificación de salida.
 
    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
      <TechnicalProfile Id="PolicyProfile">
        <DisplayName>PolicyProfile</DisplayName>
        <Protocol Name="OpenIdConnect" />
        <OutputClaims>
          ...
          <OutputClaim ClaimTypeReferenceId="correlationId" DefaultValue="{Context:CorrelationId}" />
        </OutputClaims>
        <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```


## <a name="troubleshooting-with-application-insights"></a>Solución de problemas con Application Insights

Para diagnosticar problemas con las directivas personalizadas, utilice [Application Insights](troubleshoot-with-application-insights.md). Application Insights realiza un seguimiento de la actividad del recorrido del usuario de la directiva personalizada. Proporciona una manera de diagnosticar excepciones y observar el intercambio de notificaciones entre Azure AD B2C y los distintos proveedores de notificaciones definidos por los perfiles técnicos, como proveedores de identidades, servicios basados en API, el directorio de usuarios de Azure AD B2C y otros servicios.  

Se recomienda instalar la [extensión de Azure AD B2C](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c) para [VS Code](https://code.visualstudio.com/). Con la extensión de Azure AD B2C, los registros se organizan automáticamente por nombre de directiva, identificador de correlación (Application Insights presenta el primer dígito del identificador de correlación) y la marca de tiempo del registro. Esta característica le ayuda a encontrar el registro correspondiente en función de la marca de tiempo local y a ver el recorrido del usuario tal como lo ha ejecutado Azure AD B2C. 

> [!NOTE]
> - Hay un breve retraso, normalmente inferior a cinco minutos, antes de que pueda ver nuevos registros en Application Insights.
> - La comunidad ha desarrollado la extensión de Visual Studio Code para Azure AD B2C para ayudar a los desarrolladores de soluciones de identidades. La extensión no tiene soporte técnico de Microsoft y está disponible estrictamente tal cual.

Un único flujo de inicio de sesión puede emitir más de un seguimiento de Azure Application Insights. En la captura de pantalla siguiente, la directiva *B2C_1A_signup_signin* tiene tres registros. Cada registro representa parte del flujo de inicio de sesión.

En la captura de pantalla siguiente se muestra la extensión de Azure AD B2C para VS Code con el explorador de seguimientos de Azure Application Insights.

![Captura de pantalla de la extensión de Azure AD B2C para VS Code con un seguimiento de Azure Application Insights.](./media/troubleshoot-custom-policies/vscode-extension-application-insights-trace.png)

### <a name="filter-the-trace-log"></a>Filtrado del registro de seguimientos

Con el foco en el explorador de seguimientos de Azure AD B2C, comience a escribir el primer dígito del identificador de correlación o una hora que desee buscar. Verá un cuadro de filtro en la parte superior derecha del explorador de seguimientos de Azure AD B2C que muestra lo que ha escrito hasta ahora y se resaltarán los registros de seguimiento coincidentes.  

![Captura de pantalla de la extensión de Azure AD B2C con el resaltado del filtro del explorador de seguimientos.](./media/troubleshoot-custom-policies/vscode-extension-application-insights-highlight.png)

Al mantener el puntero sobre el cuadro de filtro y seleccionar **Enable Filter on Type** (Habilitar filtro al escribir), solo se mostrarán los registros de seguimiento coincidentes. Use el **botón de borrado "X"** para borrar el filtro.

![Captura de pantalla del filtro del explorador de seguimientos de la extensión de Azure AD B2C.](./media/troubleshoot-custom-policies/vscode-extension-application-insights-filter.png)

### <a name="application-insights-trace-log-details"></a>Detalles del registro de seguimientos de Application Insights

Al seleccionar un seguimiento de Azure Application Insights, la extensión abre la ventana **Application Insights details** (Detalles de Application Insights) con la siguiente información:

- **Application Insights**: información genérica sobre el registro de seguimientos, incluido el nombre de la directiva, el identificador de correlación, el identificador de seguimiento de Azure Application Insights y la marca de tiempo del seguimiento.
- **Perfiles técnicos**: lista de perfiles técnicos que aparecen en el registro de seguimientos.
- **Notificaciones**: lista alfabética de las notificaciones que aparecen en el registro de seguimientos y sus valores. Si una notificación aparece varias veces en el registro de seguimientos con valores diferentes, un signo `=>` designa el valor más reciente. Puede revisar estas notificaciones para determinar si los valores de notificación esperados están establecidos correctamente. Por ejemplo, si tiene una condición previa que comprueba un valor de la notificación, la sección de notificaciones puede ayudarle a determinar por qué un flujo esperado se comporta de forma diferente.
- **Transformación de notificaciones:** lista de transformaciones de notificaciones que aparecen en el registro de seguimientos. Cada transformación de notificaciones contiene las notificaciones de entrada, los parámetros de entrada y las notificaciones de salida. La sección de transformación de notificaciones proporciona información sobre los datos enviados y el resultado de la transformación de notificaciones.
- **Tokens**: lista de tokens que aparecen en el registro de seguimientos. Los tokens incluyen el elemento OAuth federado subyacente y los tokens del proveedor de identidades de OpenId Connect. El token del proveedor de identidades federado proporciona detalles sobre cómo devuelve las notificaciones a Azure AD B2C el proveedor de identidades para que pueda asignar las notificaciones de salida del perfil técnico del proveedor de identidades. 
- **Excepciones**: lista de excepciones o errores graves que aparecen en el registro de seguimientos.
- **Código JSON de Application Insights**: datos sin procesar que devuelve Application Insights.

En la captura de pantalla siguiente se muestra un ejemplo de la ventana de detalles del registro de seguimientos de Application Insights.  

![Captura de pantalla del informe de seguimientos de Azure AD B2C de la extensión de Azure AD B2C.](./media/troubleshoot-custom-policies/vscode-extension-application-insights-report.png)

## <a name="troubleshoot-jwt-tokens"></a>Solución de problemas de tokens JWT

Para fines de validación y depuración, los desarrolladores pueden decodificar los tokens JWT mediante un sitio como [https://jwt.ms](https://jwt.ms). Cree una aplicación de prueba que pueda redirigir a `https://jwt.ms` para la inspección de tokens. Si todavía no lo ha hecho, [registre una aplicación web](tutorial-register-applications.md) y [habilite la concesión implícita de token de identificador](tutorial-register-applications.md#enable-id-token-implicit-grant). 

![Captura de pantalla de la vista previa del token JWT.](./media/troubleshoot-custom-policies/jwt-token-preview.png)

Use **Ejecutar ahora** y `https://jwt.ms` para probar sus directivas con independencia de su aplicación web o móvil. Este sitio web actúa como una aplicación de usuario de confianza. Muestra el contenido del JSON Web Token (JWT) que la directiva de Azure AD B2C genera.

## <a name="troubleshoot-saml-protocol"></a>Solución de problemas del protocolo SAML

Para ayudar a configurar y depurar la integración con el proveedor de servicios, puede usar una extensión del navegador para el protocolo SAML; por ejemplo, la [extensión SAML DevTools](https://chrome.google.com/webstore/detail/saml-devtools-extension/jndllhgbinhiiddokbeoeepbppdnhhio) para Chrome, [SAML-Tracer](https://addons.mozilla.org/es/firefox/addon/saml-tracer/) para Firefox o las [herramientas para desarrolladores de Edge o IE](https://techcommunity.microsoft.com/t5/microsoft-sharepoint-blog/gathering-a-saml-token-using-edge-or-ie-developer-tools/ba-p/320957).

En la captura de pantalla siguiente se muestra cómo presenta la extensión SAML DevTools la solicitud SAML que Azure AD B2C envía al proveedor de identidades y la respuesta de SAML.

![Captura de pantalla del registro de seguimientos del protocolo SAML.](./media/troubleshoot-custom-policies/saml-protocol-trace.png)

Con estas herramientas, puede comprobar la integración entre la aplicación y Azure AD B2C. Por ejemplo:

- Puede comprobar si la solicitud SAML contiene una firma y determinar qué algoritmo se usa para iniciar sesión en la solicitud de autorización.
- Puede comprobar si Azure AD B2C devuelve un mensaje de error.
- Compruebe si la sección de aserción está cifrada.
- Obtenga el nombre de las notificaciones que devuelve el proveedor de identidades.

También puede realizar un seguimiento del intercambio de mensajes entre su explorador cliente y Azure AD B2C con [Fiddler](https://www.telerik.com/fiddler). Puede ayudarle a detectar un error en el recorrido del usuario en los pasos de la orquestación.

## <a name="troubleshoot-policy-validity"></a>Solución de problemas de validez de las directivas

Cuando termine de desarrollar la directiva, cargue la directiva en Azure AD B2C. Puede haber algunos problemas con la directiva. Use los métodos siguientes para garantizar la integridad o validez de la directiva.

El error más común al configurar directivas personalizadas es el formato incorrecto del código XML. Es fundamental contar con un buen editor XML. Permite mostrar XML nativo y contenido codificado por colores, rellenar previamente los términos comunes, mantener los elementos XML indexados y realizar una validación con un esquema XML.

Se recomienda usar [Visual Studio Code](https://code.visualstudio.com/). A continuación, instale una extensión para XML, como [Compatibilidad con lenguaje XML de Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml). La extensión para XML permite validar el esquema XML antes de cargar el archivo XML mediante la definición del esquema [XSD](https://raw.githubusercontent.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/master/TrustFrameworkPolicy_0.3.0.0.xsd) de la directiva personalizada.

Puede usar la estrategia de asociación de archivos XML para enlazar al archivo XML el esquema XSD mediante la adición de la siguiente configuración al archivo `settings.json` de VS Code. Para ello:

1. En VS Code, haga clic en **Configuración.** . Para más información, consulte [Configuración de usuarios y áreas de trabajo](https://code.visualstudio.com/docs/getstarted/settings).
1. Busque **fileAssociations** y, en **Extensión**, seleccione **XML**.
1. Seleccione **Editar en settings.json**.

    ![Captura de pantalla de la validación del esquema XML en VS Code.](./media/troubleshoot-custom-policies/xml-validation.png)
1. En el archivo settings.json, agregue el siguiente código JSON:

    ```json
    "xml.fileAssociations": [
      {
        "pattern": "**.xml",
        "systemId": "https://raw.githubusercontent.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/master/TrustFrameworkPolicy_0.3.0.0.xsd"
      }
    ]
    ```

En el ejemplo siguiente se muestra un error de validación de XML. Al mover el mouse sobre el nombre del elemento, la extensión enumera los elementos esperados.

![Captura de pantalla del indicador de error de validación del esquema XML en VS Code.](./media/troubleshoot-custom-policies/xml-validation-error.png)

En el siguiente caso, el elemento `DisplayName` es correcto. Pero está en el orden incorrecto. El elemento `DisplayName` debe estar antes del elemento `Protocol`. Para corregir el problema, mueva el mouse sobre el elemento `DisplayName` hasta el orden correcto de los elementos.

![Captura de pantalla del error de orden de la validación del esquema XML en VS Code.](./media/troubleshoot-custom-policies/xml-validation-order-error.png)

## <a name="upload-policies-and-policy-validation"></a>Validación de directivas y directivas de carga

La validación del archivo de directiva XML se realiza automáticamente durante la carga. La mayoría de los errores hacen que se produzca un error en la carga. En la validación se incluye el archivo de directiva que se está cargando. Además, se incluye la cadena de archivos a la que hace referencia el archivo de carga (el archivo de directiva de usuario de confianza, el archivo de extensiones y el archivo base).

> [!TIP]
> Azure AD B2C ejecuta una validación adicional para la directiva de usuario de confianza. Si tiene un problema con la directiva, incluso si edita solo la directiva de extensión, también es una buena práctica cargar la directiva de usuario de confianza. 

Esta sección contiene los errores comunes de validación y las soluciones probables.

### <a name="schema-validation-error-found-has-invalid-child-element-name"></a>Se ha encontrado un error de validación del esquema ...tiene un elemento secundario no válido "{name}"

La directiva contiene un elemento XML no válido o el elemento XML es válido pero parece estar en el orden incorrecto. Para corregir este tipo de error, consulte la sección [Solución de problemas de validez de las directivas](#troubleshoot-policy-validity).

### <a name="there-is-a-duplicate-key-sequence-number"></a>Hay una secuencia de claves duplicada "{number}" 

Un [recorrido](userjourneys.md) o un [subrecorrido](subjourneys.md) del usuario constan de una lista ordenada de pasos de orquestación que se ejecutan en secuencia. Después de cambiar el recorrido, vuelva a numerar los pasos de forma secuencial sin omitir ningún número entero del 1 a N.

> [!TIP]
> Puede usar el comando de la [extensión de Azure AD B2C](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c) para [VS Code](https://code.visualstudio.com/) `(Shift+Ctrl+r)` para volver a numerar todos los pasos de orquestación de recorridos y subrecorridos del usuario de la directiva.

### <a name="was-expected-to-have-step-with-order-number-but-it-was-not-found"></a>...se esperaba el paso con el orden "{number}", pero no se encontró...

Consulte el error anterior.

### <a name="orchestration-step-order-number-in-user-journey-name-is-followed-by-a-claims-provider-selection-step-and-must-be-a-claims-exchange-but-it-is-of-type"></a>El paso de orquestación con el orden "{number}" en el recorrido del usuario "{name}" ... va seguido de un paso de selección del proveedor de notificaciones y debe ser un intercambio de notificaciones, pero es de tipo...

Los tipos de pasos de orquestación `ClaimsProviderSelection` y `CombinedSignInAndSignUp` contienen una lista de opciones entre las que el usuario puede elegir. Deben ir seguidos del tipo `ClaimsExchange` con uno o varios intercambios de notificaciones.

Los siguientes pasos de orquestación provocan este tipo de error. El segundo paso de orquestación debe ser de tipo `ClaimsExchange`, no de tipo `ClaimsProviderSelection`.

```xml
<!-- 
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>-->
      <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
        <ClaimsProviderSelections>
          <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange"/>
          <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmailExchange"/>
        </ClaimsProviderSelections>
        <ClaimsExchanges>
          <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email"/>
        </ClaimsExchanges>
      </OrchestrationStep> 
      
      <OrchestrationStep Order="2" Type="ClaimsProviderSelection">
        ...
      </OrchestrationStep>
      ...
    <!--
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys> -->
```

### <a name="step-number-with-2-claims-exchanges-it-must-be-preceded-by-a-claims-provider-selection-in-order-to-determine-which-claims-exchange-can-be-used"></a>... paso {number} con 2 intercambios de notificaciones. Debe ir precedido de una selección del proveedor de notificaciones para determinar qué intercambio de notificaciones se puede usar

Un tipo de paso de orquestación `ClaimsExchange` debe tener un único elemento `ClaimsExchange`, a menos que el paso anterior sea de tipo `ClaimsProviderSelection` o `CombinedSignInAndSignUp`. Los siguientes pasos de orquestación provocan este tipo de error. El sexto paso contiene dos intercambios de notificaciones. 

```xml
<!-- 
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>-->
      ...
      <OrchestrationStep Order="5" Type="ClaimsExchange">
        ...
        <ClaimsExchanges>
          <ClaimsExchange Id="SelfAsserted-Social" TechnicalProfileReferenceId="SelfAsserted-Social"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="6" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="Call-REST-First-API" TechnicalProfileReferenceId="Call-REST-First-API"/>
          <ClaimsExchange Id="Call-REST-Second-API" TechnicalProfileReferenceId="Call-REST-Second-API"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      ...
    <!--
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys> -->
```

Para corregir este tipo de error, use dos pasos de orquestación. Cada paso de orquestación con un intercambio de notificaciones.

```xml
<!-- 
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>-->
      ...
      <OrchestrationStep Order="5" Type="ClaimsExchange">
        ...
        <ClaimsExchanges>
          <ClaimsExchange Id="SelfAsserted-Social" TechnicalProfileReferenceId="SelfAsserted-Social"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="6" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="Call-REST-First-API" TechnicalProfileReferenceId="Call-REST-First-API"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="7" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="Call-REST-Second-API" TechnicalProfileReferenceId="Call-REST-Second-API"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      ...
    <!--
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys> -->
```

### <a name="there-is-a-duplicate-key-sequence-name"></a>Hay una secuencia de claves duplicada "{name}"

Un recorrido tiene varios elementos `ClaimsExchange` con el mismo `Id`. Los pasos siguientes provocan este tipo de error. El identificador *AADUserWrite* aparece dos veces en el recorrido del usuario.

```xml
<!-- 
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>-->
      ...
      <OrchestrationStep Order="7" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="AADUserWrite" TechnicalProfileReferenceId="AAD-UserWriteUsingAlternativeSecurityId"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="8" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="AADUserWrite" TechnicalProfileReferenceId="Call-REST-API"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      ...
    <!--
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys> -->
```

Para corregir este tipo de error, cambie el intercambio de notificaciones del octavo paso de orquestación por un nombre único, como *Call-REST-API*.

```xml
<!-- 
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>-->
      ...
      <OrchestrationStep Order="7" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="AADUserWrite" TechnicalProfileReferenceId="AAD-UserWriteUsingAlternativeSecurityId"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="8" Type="ClaimsExchange">
        <ClaimsExchanges>
          <ClaimsExchange Id="Call-REST-API" TechnicalProfileReferenceId="Call-REST-API"/>
        </ClaimsExchanges>
      </OrchestrationStep>
      ...
    <!--
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys> -->
```

### <a name="makes-a-reference-to-claimtype-with-id-claim-name-but-neither-the-policy-nor-any-of-its-base-policies-contain-such-an-element"></a>...hace referencia a ClaimType con el identificador "{claim name}", pero ni la directiva ni ninguna de sus directivas base contienen este tipo de elemento.

Este tipo de error se produce cuando la directiva hace referencia a una notificación que no está declarada en el [esquema de notificaciones](claimsschema.md). Las notificaciones se deben definir en al menos uno de los archivos de la directiva. 

Por ejemplo, un perfil técnico con la notificación de salida *schoolId*. Pero la notificación de salida *schoolId* nunca se declara en la directiva ni en una directiva antecesora.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="schoolId" />
  ...
</OutputClaims>
```

Para corregir este tipo de error, compruebe si el valor de `ClaimTypeReferenceId` está mal escrito o no existe en el esquema. Si la notificación se define en la directiva de extensiones, pero también se usa en la directiva base. Asegúrese de que la notificación esté definida en la directiva en la que se usa o en una directiva de nivel superior.

La adición de la notificación al esquema de notificaciones resuelve este tipo de error.

```xml
<!--
<BuildingBlocks>
  <ClaimsSchema> -->
    <ClaimType Id="schoolId">
      <DisplayName>School name</DisplayName>
      <DataType>string</DataType>
      <UserHelpText>Enter your school name</UserHelpText>
      <UserInputType>TextBox</UserInputType>
    </ClaimType>
  <!-- 
  </ClaimsSchema>
</BuildingBlocks> -->
```

### <a name="makes-a-reference-to-a-claimstransformation-with-id"></a>...hace referencia a un elemento claimsTransformation con el identificador...

La causa de este error es similar a la del error de notificación. Consulte el error anterior.

### <a name="user-is-currently-logged-as-a-user-of-yourtenantonmicrosoftcom-tenant"></a>El usuario ha iniciado sesión actualmente como un usuario del inquilino "yourtenant.onmicrosoft.com"...

Ha iniciado sesión con una cuenta de un inquilino que es diferente al de la directiva que intenta cargar. Por ejemplo, ha iniciado sesión con admin@contoso.onmicrosoft.com, mientras que el elemento `TenantId` de la directiva está establecido en `fabrikam.onmicrosoft.com`.

```xml
<TrustFrameworkPolicy ...
  TenantId="fabrikam.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin"
  PublicPolicyUri="http://fabrikam.onmicrosoft.com/B2C_1A_signup_signin">

  <BasePolicy>
    <TenantId>fabrikam.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  ...
</TrustFrameworkPolicy>
```

- Compruebe que el valor de `TenantId` en los elementos `<TrustFrameworkPolicy\>` y `<BasePolicy\>` coincide con su inquilino de destino de Azure AD B2C.

### <a name="claim-type-name-is-the-output-claim-of-the-relying-partys-technical-profile-but-it-is-not-an-output-claim-in-any-of-the-steps-of-user-journey"></a>El tipo de notificación "{name}" es la notificación de salida del perfil técnico del usuario de confianza, pero no es una notificación de salida de ninguno de los pasos del recorrido del usuario...

Ha agregado una notificación de salida en una directiva de usuario de confianza, pero la notificación de salida no es una notificación de salida de ninguno de los pasos del recorrido del usuario. Azure AD B2C no puede leer el valor de la notificación de la bolsa de notificaciones.

En el ejemplo siguiente, la notificación *schoolId* es una notificación de salida del perfil técnico del usuario de confianza, pero no es una notificación de salida de ninguno de los pasos del recorrido del usuario *SignUpOrSignIn*.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="schoolId" />
      ...
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
``` 

Para corregir este tipo de error, asegúrese de que las notificaciones de salida aparezcan en al menos una colección de notificaciones de salida del perfil técnico de los pasos de orquestación. Si el recorrido del usuario no puede generar la notificación, establezca un valor predeterminado, como una cadena vacía, en el perfil técnico del usuario de confianza.  

```xml
<OutputClaim ClaimTypeReferenceId="schoolId" DefaultValue="" />
```

### <a name="input-string-was-not-in-a-correct-format"></a>La cadena de entrada no tiene el formato correcto

Ha establecido un tipo de valor incorrecto en una notificación de otro tipo. Por ejemplo, ha definido una notificación de número entero.

```xml
<!--
<BuildingBlocks>
  <ClaimsSchema> -->
    <ClaimType Id="age">
      <DisplayName>Age</DisplayName>
      <DataType>int</DataType>
    </ClaimType>
  <!--
  </ClaimsSchema>
</BuildingBlocks> -->
```

Y, después, ha intentado establecer un valor de cadena:

```xml
<OutputClaim ClaimTypeReferenceId="age" DefaultValue="ABCD" />
```

Para corregir este tipo de error, asegúrese de establecer el valor correcto, como `DefaultValue="0"`.


### <a name="tenant-name-already-has-a-policy-with-id-name-another-policy-with-the-same-id-cannot-be-stored"></a>El inquilino "{name}" ya tiene una directiva con el identificador "{name}". No se puede almacenar otra directiva con el mismo identificador.

Ha intentado cargar una directiva en el inquilino, pero ya se ha cargado una directiva con el mismo nombre en dicho inquilino. 

Para corregir este tipo de error, active la casilla **Sobrescribir la directiva personalizada si ya existe** al cargar la directiva.

![Captura de pantalla que muestra cómo sobrescribir la directiva personalizada si ya existe.](./media/troubleshoot-custom-policies/overwrite-custom-policy-if-exists.png)


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la [Recopilación de registros de Azure Active Directory B2C con Application Insights](troubleshoot-with-application-insights.md).

