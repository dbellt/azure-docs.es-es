---
title: Seguimiento del comportamiento del usuario con Application Insights
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo habilitar registros de eventos en Application Insights desde recorridos del usuario de Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 01/29/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 2cde44ddb49ede8002b8a25ab47ae92ccd602a9d
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226377"
---
# <a name="track-user-behavior-in-azure-ad-b2c-by-using-application-insights"></a>Seguimiento del comportamiento del usuario en Azure Active Directory B2C mediante Application Insights

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

En Azure Active Directory B2C (Azure AD B2C), puede enviar los datos de eventos directamente a [Application Insights](../azure-monitor/app/app-insights-overview.md) mediante la clave de instrumentación proporcionada a Azure AD B2C. Con un perfil técnico de Application Insights, puede obtener registros de eventos detallados y personalizados para los recorridos del usuario para:

- Obtener información detallada sobre el comportamiento del usuario.
- Solucionar problemas con sus propias directivas durante la fase de desarrollo o producción.
- Medir el rendimiento.
- Crear notificaciones de Application Insights.

## <a name="overview"></a>Información general

Para habilitar los registros de eventos personalizados, agregue un perfil técnico de Application Insights. En el perfil técnico, se define la clave de instrumentación de Application Insights, el nombre del evento y las notificaciones que se van a registrar. Para publicar un evento, agregue el perfil técnico como un paso de orquestación en un [recorrido del usuario](userjourneys.md).

Al usar Application Insights, tenga en cuenta lo siguiente:

- Hay un breve retraso, normalmente inferior a cinco minutos, antes de que los nuevos registros estén disponibles en Application Insights.
- Azure AD B2C le permite elegir las notificaciones que registrar. No incluya notificaciones con datos personales.
- Para registrar una sesión de usuario, puede usar un Id. de correlación para unificar eventos.
- Llame al perfil técnico de Application Insights directamente desde un [recorrido](userjourneys.md) o un [subrecorrido del usuario](subjourneys.md). No use un perfil técnico de Application Insights como [perfil técnico de validación](validation-technical-profile.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Cuando usa Application Insights con Azure AD B2C, todo lo que necesita hacer es crear un recurso y obtener la clave de instrumentación. Para más información, vea [Creación de recursos en Application Insights](../azure-monitor/app/create-new-resource.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Asegúrese de que está usando el directorio que tiene su suscripción de Azure. Seleccione el filtro **Directorio y suscripciones** en el menú superior y elija el directorio que contiene su suscripción de Azure. Este inquilino no es el inquilino de Azure AD B2C.
1. Elija **Crear un recurso** en la esquina superior izquierda de Azure Portal y, a continuación, busque y seleccione **Application Insights**.
1. Seleccione **Crear**.
1. En **Nombre**, escriba un nombre para el recurso.
1. En **Tipo de aplicación**, seleccione **Aplicación web ASP.NET**.
1. En **Grupo de recursos**, seleccione un grupo existente o escriba un nombre para un nuevo grupo.
1. Seleccione **Crear**.
1. Abra el nuevo recurso de Application Insights, expanda **Información esencial** y copie la clave de instrumentación.

![Captura de pantalla que muestra la clave de instrumentación en la pestaña Información general de Application Insights.](./media/analytics-with-application-insights/app-insights.png)

## <a name="define-claims"></a>Definición de notificaciones

Una notificación proporciona un almacenamiento temporal de datos durante la ejecución de una directiva de Azure AD B2C. Las notificaciones se declaran en el [elemento ClaimsSchema](claimsschema.md).

1. Abra el archivo de extensiones de la directiva. El archivo podría ser similar a `SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`** .
1. Busque el elemento [BuildingBlocks](buildingblocks.md). Si no ve el elemento, agréguelo.
1. Busque el elemento **ClaimsSchema**. Si no ve el elemento, agréguelo.
1. Agregue las notificaciones siguientes al elemento **ClaimsSchema**:

   ```xml
   <ClaimType Id="EventType">
     <DisplayName>Event type</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="EventTimestamp">
     <DisplayName>Event timestamp</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="PolicyId">
     <DisplayName>Policy Id</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="Culture">
     <DisplayName>Culture ID</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="CorrelationId">
     <DisplayName>Correlation Id</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="federatedUser">
     <DisplayName>Federated user</DisplayName>
     <DataType>boolean</DataType>
   </ClaimType>
   <ClaimType Id="parsedDomain">
     <DisplayName>Domain name</DisplayName>
     <DataType>string</DataType>
     <UserHelpText>The domain portion of the email address.</UserHelpText>
   </ClaimType>
   <ClaimType Id="userInLocalDirectory">
     <DisplayName>userInLocalDirectory</DisplayName>
     <DataType>boolean</DataType>
   </ClaimType>
   ```

## <a name="add-new-technical-profiles"></a>Incorporación de nuevos perfiles técnicos

Los perfiles técnicos pueden considerarse funciones en la directiva personalizada. Estas funciones usan el enfoque de [inclusión de perfil técnico](technicalprofiles.md#include-technical-profile), donde un perfil técnico incluye otro perfil técnico y cambia la configuración o agrega una nueva funcionalidad. En la siguiente tabla se definen los perfiles técnicos que se usan para abrir una sesión y publicar eventos.

| Perfil técnico | Tarea |
| ----------------- | -----|
| AppInsights-Common | El perfil técnico común con la configuración típica. Incluye la clave de instrumentación de Application Insights, una colección de notificaciones que se van a registrar y un modo de desarrollador. En los otros perfiles técnicos se incluye el perfil técnico común y se agregan más notificaciones, como el nombre del evento. |
| AppInsights-SignInRequest | Registra un evento **SignInRequest** con un conjunto de notificaciones cuando se ha recibido una solicitud de inicio de sesión. |
| AppInsights-UserSignUp | Registra un evento **UserSignUp** cuando el usuario activa la opción de registro en un recorrido de inicio de sesión o registro. |
| AppInsights-SignInComplete | Registra un evento **SignInComplete** de autenticación correcta cuando se ha enviado un token a la aplicación de usuario de confianza. |

Abra el archivo *TrustFrameworkExtensions.xml* desde el módulo de inicio. Agregue los perfiles técnicos al elemento **ClaimsProvider**:

```xml
<ClaimsProvider>
  <DisplayName>Application Insights</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AppInsights-Common">
      <DisplayName>Application Insights</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- The ApplicationInsights instrumentation key, which you use for logging the events -->
        <Item Key="InstrumentationKey">xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</Item>
        <Item Key="DeveloperMode">false</Item>
        <Item Key="DisableTelemetry ">false</Item>
      </Metadata>
      <InputClaims>
        <!-- Properties of an event are added through the syntax {property:NAME}, where NAME is the property being added to the event. DefaultValue can be either a static value or a value that's resolved by one of the supported DefaultClaimResolvers. -->
        <InputClaim ClaimTypeReferenceId="EventTimestamp" PartnerClaimType="{property:EventTimestamp}" DefaultValue="{Context:DateTimeInUtc}" />
        <InputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="{property:TenantId}" DefaultValue="{Policy:TrustFrameworkTenantId}" />
        <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
        <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:CorrelationId}" DefaultValue="{Context:CorrelationId}" />
        <InputClaim ClaimTypeReferenceId="Culture" PartnerClaimType="{property:Culture}" DefaultValue="{Culture:RFC5646}" />
      </InputClaims>
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-SignInRequest">
      <InputClaims>
        <!-- An input claim with a PartnerClaimType="eventName" is required. This is used by the AzureApplicationInsightsProvider to create an event with the specified value. -->
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInRequest" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-UserSignUp">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="UserSignUp" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
    
    <TechnicalProfile Id="AppInsights-SignInComplete">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInComplete" />
        <InputClaim ClaimTypeReferenceId="federatedUser" PartnerClaimType="{property:FederatedUser}" DefaultValue="false" />
        <InputClaim ClaimTypeReferenceId="parsedDomain" PartnerClaimType="{property:FederationPartner}" DefaultValue="Not Applicable" />
        <InputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="{property:IDP}" DefaultValue="Local" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

> [!IMPORTANT]
> Cambie la clave de instrumentación del perfil técnico de `AppInsights-Common` por el GUID que su recurso de Application Insights proporciona.

## <a name="add-the-technical-profiles-as-orchestration-steps"></a>Incorporación de los perfiles técnicos como pasos de orquestación

Agregue nuevos pasos de orquestación que hagan referencia a los perfiles técnicos.

> [!IMPORTANT]
> Después de agregar los nuevos pasos de orquestación, vuelva a enumerar los pasos de forma secuencial sin omitir los enteros de 1 a N.

1. Llame a `AppInsights-SignInRequest` como el segundo paso de orquestación. Este paso realiza un seguimiento de la recepción de una solicitud de registro o de inicio de sesión.

   ```xml
   <!-- Track that we have received a sign in request -->
   <OrchestrationStep Order="2" Type="ClaimsExchange">
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackSignInRequest" TechnicalProfileReferenceId="AppInsights-SignInRequest" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

1. Antes del paso de orquestación `SendClaims`, agregue un nuevo paso que llame a `AppInsights-UserSignup`. Se desencadena cuando el usuario selecciona el botón de suscripción en un recorrido de inicio de sesión o registro.

   ```xml
   <!-- Handles the user selecting the sign-up link in the local account sign-in page -->
   <OrchestrationStep Order="8" Type="ClaimsExchange">
     <Preconditions>
       <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
         <Value>newUser</Value>
         <Action>SkipThisOrchestrationStep</Action>
       </Precondition>
       <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
         <Value>newUser</Value>
         <Value>false</Value>
         <Action>SkipThisOrchestrationStep</Action>
       </Precondition>
     </Preconditions>
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackUserSignUp" TechnicalProfileReferenceId="AppInsights-UserSignup" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

1. Después del paso de orquestación `SendClaims`, llame a `AppInsights-SignInComplete`. Este paso muestra un recorrido completado correctamente.

   ```xml
   <!-- Track that we have successfully sent a token -->
   <OrchestrationStep Order="10" Type="ClaimsExchange">
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackSignInComplete" TechnicalProfileReferenceId="AppInsights-SignInComplete" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

## <a name="upload-your-file-run-the-policy-and-view-events"></a>Carga del archivo, ejecución de la directiva y visualización de eventos

Cargue el archivo *TrustFrameworkExtensions.xml*. Luego, llame a la directiva de usuario de confianza desde su aplicación o use **Ejecutar ahora** en Azure Portal. Espere a que los eventos están disponibles en Application Insights.

1. Abra el recurso de **Application Insights** en su inquilino de Azure Active Directory.
1. Seleccione **Uso** y, a continuación, **Eventos**.
1. Establezca **During** (Durante) en **Last hour** (Última hora) y **By** (por) en **3 minutes** (3 minutos). Es posible que tenga que actualizar la ventana para ver los resultados.

![Captura de pantalla que muestra estadísticas de eventos de Application Insights.](./media/analytics-with-application-insights/app-ins-graphic.png)

## <a name="collect-more-data"></a>Recopilar más datos

Para ajustarse a sus necesidades empresariales, puede que desee registrar más notificaciones. Para agregar una notificación, primero [defina una notificación](#define-claims) y, después, agregue la notificación a la colección de notificaciones de entrada. Las notificaciones que agregue al perfil técnico **AppInsights-Common** aparecerán en todos los eventos. Las notificaciones que agregue a un perfil técnico específico aparecerán solo en ese evento. El elemento de notificación de entrada contiene los atributos siguientes:

- **ClaimTypeReferenceId** es la referencia a un tipo de notificación.
- **PartnerClaimType** es el nombre de la propiedad que aparece en Azure Insights. Use la sintaxis de `{property:NAME}`, donde `NAME` es una propiedad que se agrega al evento.
- **DefaultValue** es un valor predefinido que se debe registrar, como un nombre de evento. Si una notificación que se usa en el recorrido del usuario está vacía, se usa el valor predeterminado. Por ejemplo, la notificación `identityProvider` se establece mediante los perfiles técnicos de la federación, como Facebook. Si la notificación está vacía, indica que el usuario ha iniciado sesión con una cuenta local. Por lo tanto, el valor predeterminado se establece en **Local**. También puede registrar una [resolución de notificación](claim-resolver-overview.md) con un valor contextual, como el id. de aplicación o la dirección IP del usuario.

### <a name="manipulate-claims"></a>Manipulación de notificaciones

Puede usar [transformaciones de notificaciones de entrada](custom-policy-overview.md#manipulating-your-claims) para modificar las notificaciones de entrada o generar otras nuevas antes de enviarlas a Application Insights. En el ejemplo siguiente, el perfil técnico incluye la `CheckIsAdmin` la transformación de notificaciones de entrada.

```xml
<TechnicalProfile Id="AppInsights-SignInComplete">
  <InputClaimsTransformations>  
    <InputClaimsTransformation ReferenceId="CheckIsAdmin" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="isAdmin" PartnerClaimType="{property:IsAdmin}"  />
    ...
  </InputClaims>
  <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
</TechnicalProfile>
```

### <a name="add-events"></a>Agregar eventos

Para agregar un evento, cree un nuevo perfil técnico que incluya el perfil técnico de `AppInsights-Common`. A continuación, agregue el nuevo perfil técnico como paso de orquestación al [recorrido del usuario](custom-policy-overview.md#orchestration-steps). Use el elemento [Precondition](userjourneys.md#preconditions) para desencadenar el evento cuando esté listo. Por ejemplo, informe del evento solo cuando los usuarios se ejecuten mediante autenticación multifactor.

```xml
<TechnicalProfile Id="AppInsights-MFA-Completed">
  <InputClaims>
     <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="MFA-Completed" />
  </InputClaims>
  <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
</TechnicalProfile>
```

>[!Important]
>Al agregar un evento al recorrido del usuario, recuerde volver a numerar los pasos de la orquestación secuencialmente.

```xml
<OrchestrationStep Order="8" Type="ClaimsExchange">
  <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
    <Value>isActiveMFASession</Value>
    <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="TrackUserMfaCompleted" TechnicalProfileReferenceId="AppInsights-MFA-Completed" />
  </ClaimsExchanges>
</OrchestrationStep>
```

## <a name="enable-developer-mode"></a>Habilitar el modo de desarrollador

Al usar Application Insights para definir eventos, puede indicar si está habilitado el modo de desarrollador. El modo de desarrollador controla cómo se almacenan en búfer los eventos. En un entorno de desarrollo con un volumen de eventos mínimo, al habilitar el modo de desarrollador, los eventos se envían inmediatamente a Application Insights. El valor predeterminado es `false`. No habilite el modo de desarrollador en entornos de producción.

Para habilitar el modo de desarrollador, cambie los metadatos de `DeveloperMode` a `true` en el perfil técnico `AppInsights-Common`:

```xml
<TechnicalProfile Id="AppInsights-Common">
  <Metadata>
    ...
    <Item Key="DeveloperMode">true</Item>
  </Metadata>
</TechnicalProfile>
```

## <a name="disable-telemetry"></a>Deshabilitar la telemetría

Para deshabilitar los registros de Application Insights, cambie los metadatos de `DisableTelemetry` a `true` en el perfil técnico `AppInsights-Common`:

```xml
<TechnicalProfile Id="AppInsights-Common">
  <Metadata>
    ...
    <Item Key="DisableTelemetry">true</Item>
  </Metadata>
</TechnicalProfile>
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [crear paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights](../azure-monitor/app/tutorial-app-dashboards.md).

::: zone-end