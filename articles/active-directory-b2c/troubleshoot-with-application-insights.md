---
title: Solución de problemas de las directivas personalizadas con Application Insights
titleSuffix: Azure AD B2C
description: Configuración de Application Insights para realizar el seguimiento de la ejecución de las directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/05/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 074bffb8614be1f71ba1956fd5a238bc19354c58
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028750"
---
# <a name="collect-azure-active-directory-b2c-logs-with-application-insights"></a>Recopilación de registros de Azure Active Directory B2C con Application Insights

En este artículo se proporcionan los pasos para recopilar registros de Active Directory B2C (Azure AD B2C) de forma que pueda diagnosticar problemas con sus directivas personalizadas. Application Insights proporciona un modo de diagnosticar excepciones y visualizar problemas de rendimiento de la aplicación. Azure AD B2C incluye una característica para enviar datos a Application Insights.

Los registros de actividad detallados aquí **SOLO** deben estar habilitados durante el desarrollo de las directivas personalizadas.

> [!WARNING]
> No establezca `DeploymentMode` en `Development` en entornos de producción. Los registros recopilan todas las notificaciones que se envían a los proveedores de identidad y se reciben de estos. Usted, como desarrollador, asume la responsabilidad de los datos personales recopilados en los registros de Application Insights. Estos registros detallados solo se recopilan cuando la directiva se coloca en **MODO DE DESARROLLADOR**.

## <a name="set-up-application-insights"></a>Configuración de Application Insights

Si aún no tiene una, cree una instancia de Application Insights en su suscripción.

> [!TIP]
> Se puede usar una sola instancia de Application Insights para varios inquilinos de Azure AD B2C. Después, en la consulta, puede filtrar por inquilino o nombre de directiva. Para obtener más información, [consulte los ejemplos de registros en Application Insights](#see-the-logs-in-application-insights).

Para usar una instancia de salida de Application Insights en su suscripción, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene la suscripción de Azure (no el directorio de Azure AD B2C).
1. Abra el recurso de Application Insights que creó anteriormente.
1. En la página **Información general**, anote la **Clave de instrumentación**.

Para crear una instancia de Application Insights en su suscripción, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene la suscripción de Azure (no el directorio de Azure AD B2C).
1. Seleccione **Crear un recurso** en el panel de navegación izquierdo.
1. Busque y seleccione **Application Insights** y, luego, seleccione **Crear**.
1. Complete el formulario, seleccione **Revisar y crear** y seleccione **Crear**.
1. Cuando la implementación se haya completado, seleccione **Ir al recurso**.
1. En **Configurar**, en el menú de Application Insights, seleccione **Propiedades**.
1. Registre la **CLAVE DE INSTRUMENTACIÓN** para su uso en un paso posterior.

## <a name="configure-the-custom-policy"></a>Configuración de la directiva personalizada

1. Abra el archivo de usuario de confianza (RP), por ejemplo *SignUpOrSignin.xml*.
1. Agregue los siguientes atributos al elemento `<TrustFrameworkPolicy>`:

   ```xml
   DeploymentMode="Development"
   UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
   ```

1. Si aún no existe, agregue un nodo secundario `<UserJourneyBehaviors>` al nodo `<RelyingParty>`. Debe colocarse después de `<DefaultUserJourney ReferenceId="UserJourney Id" from your extensions policy, or equivalent (for example:SignUpOrSigninWithAAD" />`.
1. Agregue el siguiente nodo como nodo secundario del elemento `<UserJourneyBehaviors>`. Asegúrese de reemplazar `{Your Application Insights Key}` por la **Clave de instrumentación** de Application Insights que registro anteriormente.

    ```xml
    <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
    ```

    * `DeveloperMode="true"` indica a Application Insights que acelere la telemetría a través de la canalización de procesamiento. Es bueno para el desarrollo, pero está restringido en volúmenes elevados. En producción, establezca `DeveloperMode` en `false`.
    * `ClientEnabled="true"` envía el script del lado cliente de ApplicationInsights para realizar un seguimiento de la vista de página y de los errores del lado cliente. Puede verlos en la tabla **browserTimings** en el portal de Application Insights. Mediante el establecimiento de `ClientEnabled= "true"`, agrega Application Insights al script de la página y obtiene los intervalos de carga de página y de las llamadas AJAX, los recuentos, los detalles de las excepciones del explorador y los errores de AJAX, así como los recuentos de usuarios y sesiones. Este campo es **opcional** y está establecido en `false` de forma predeterminada.
    * `ServerEnabled="true"` envía el JSON UserJourneyRecorder como evento personalizado a Application Insights.

    Por ejemplo:

    ```xml
    <TrustFrameworkPolicy
      ...
      TenantId="fabrikamb2c.onmicrosoft.com"
      PolicyId="SignUpOrSignInWithAAD"
      DeploymentMode="Development"
      UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
    >
    ...
    <RelyingParty>
      <DefaultUserJourney ReferenceId="UserJourney ID from your extensions policy, or equivalent (for example: SignUpOrSigninWithAzureAD)" />
      <UserJourneyBehaviors>
        <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      </UserJourneyBehaviors>
      ...
    </TrustFrameworkPolicy>
    ```

1. Cargue la directiva.

## <a name="see-the-logs-in-application-insights"></a>Visualización de registros en Application Insights

Hay un breve retraso, normalmente inferior a cinco minutos, antes de que pueda ver nuevos registros en Application Insights.

1. Abra el recurso de Application Insights que ha creado en [Azure Portal](https://portal.azure.com).
1. En la página **Información general**, seleccione **Registros**.
1. Abra una nueva pestaña en Application Insights.

A continuación se muestra una lista de consultas que puede usar para ver los registros:

| Consultar | Descripción |
|---------------------|--------------------|
| `traces` | Obtiene todos los registros generados por Azure AD B2C. |
| `traces | where timestamp > ago(1d)` | Obtiene todos los registros generados por Azure AD B2C para el último día.|
| `traces | where message contains "exception" | where timestamp > ago(2h)`|  Obtiene todos los registros con errores de las últimas dos horas.|
| `traces | where customDimensions.Tenant == "contoso.onmicrosoft.com" and customDimensions.UserJourney  == "b2c_1a_signinandup"` | Obtiene todos los registros generados por el inquilino *contoso.onmicrosoft.com* de Azure AD B2C, y el recorrido del usuario es *b2c_1a_signinandup*. |
| `traces | where customDimensions.CorrelationId == "00000000-0000-0000-0000-000000000000"`| Obtiene todos los registros generados por Azure AD B2C para un identificador de correlación. Reemplace CorrelationId por el identificador de correlación. | 

Las entradas pueden ser largas. Exporte a un archivo CSV para realizar un examen más detallado.

Para más información sobre las consultas, vea [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/log-query-overview.md).

## <a name="see-the-logs-in-vs-code-extension"></a>Consulte los registros en la extensión de VS Code

Es aconsejable que instale la [extensión de Azure AD B2C](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c) para [VS Code](https://code.visualstudio.com/). Con la extensión de Azure AD B2C, el nombre de directiva organiza los registros se organizan por nombre de directiva, identificador de correlación (Application Insights presenta el primer dígito del identificador de correlación) y la marca de tiempo del registro. Esta característica le ayuda a encontrar el registro correspondiente en función de la marca de tiempo local y a ver el recorrido del usuario tal como lo ha ejecutado Azure AD B2C.

> [!NOTE]
> La comunidad ha desarrollado la extensión de VS Code para Azure AD B2C para ayudar a los desarrolladores de identidad. La extensión no es compatible con Microsoft y se pone a disposición estrictamente como es.

### <a name="set-application-insights-api-access"></a>Establecimiento del acceso a Application Insights API

Después de configurar Application Insights y configurar la directiva personalizada, debe obtener el identificador de Application Insights **API** y crear la **clave de API**. La extensión de Azure AD B2C usa tanto el identificador de la API como la clave de API se para leer los eventos de Application Insights (datos de telemetría). Las claves de API deben administrarse como las contraseñas. Manténgala en secreto.

> [!NOTE]
> Azure AD B2C utiliza la clave de instrumentación de Application Insights que creó anteriormente para enviar datos de telemetría a Application Insights. La clave de instrumentación solo se usa en la directiva de Azure AD B2C, no en la extensión de VS Code.

Para obtener el identificador y la clave de Application Insights:

1. En Azure Portal, abra el recurso Application Insights de la aplicación.
1. Seleccione **Configuración** y luego **Acceso de API**.
1. Copie el **identificador de la aplicación**.
1. Seleccione **Create API Key** (Crear clave de API).
1. Active la casilla **Read telemetry** (Leer telemetría).
1. Copie la **clave** antes de cerrar la hoja Create API Key (Crear clave de API) y guárdela en un lugar seguro. Si pierde la clave, deberá crear otra.

    ![Captura de pantalla que muestra cómo crear una clave de acceso de API.](./media/troubleshoot-with-application-insights/application-insights-api-access.png)

### <a name="set-up-azure-ad-b2c-vs-code-extension"></a>Configuración de la extensión VS Code de Azure AD B2C

Ahora que el identificador y la clave de Azure Application Insights API, puede configurar la extensión de VS Code para leer los registros. La extensión VS Code de Azure AD B2C proporciona dos ámbitos para la configuración:

- **Configuración global de usuario:** configuración que se aplica globalmente a cualquier instancia de VS Code que se abra.
- **Configuración del área de trabajo**: la configuración se almacena en el área de trabajo y solo se aplica cuando se abre el área de trabajo (mediante la **carpeta abierta** de VS Code).

1. En el explorador de **seguimiento de Azure AD B2C**, haga clic en el icono **Configuración**.

    ![Captura de pantalla que muestra cómo seleccionar la configuración de Application Insights.](./media/troubleshoot-with-application-insights/app-insights-settings.png)

1. Proporcione el **identificador** y la **clave** de Azure Application Insights.
1. Haga clic en **Guardar**

Después de guardar la configuración los registros de Application Insights aparecen en la ventana de **seguimiento de Azure AD B2C (App Insights).**

![Captura de pantalla de la extensión de Azure AD B2C para vscode, que presenta el seguimiento de Azure Application Insights.](./media/troubleshoot-with-application-insights/vscode-extension-application-insights-trace.png)


## <a name="configure-application-insights-in-production"></a>Configuración de Application Insights en producción

Para mejorar el rendimiento del entorno de producción y mejorar la experiencia del usuario, es importante configurar la directiva para omitir los mensajes que no son importantes. Use la siguiente configuración para enviar solo los mensajes de error críticos a su instancia de Application Insights. 

1. Establezca el atributo `DeploymentMode` de [TrustFrameworkPolicy](trustframeworkpolicy.md) en `Production`. 

   ```xml
   <TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0"
   TenantId="yourtenant.onmicrosoft.com"
   PolicyId="B2C_1A_signup_signin"
   PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_signup_signin"
   DeploymentMode="Production"
   UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights">
   ```

1. Establezca `DeveloperMode` de [JourneyInsights](relyingparty.md#journeyinsights) en `false`.

   ```xml
   <UserJourneyBehaviors>
     <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="false" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
   </UserJourneyBehaviors>
   ```
   
1. Cargue y pruebe la directiva.



## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [solucionar los problemas de las directivas personalizadas de Azure AD B2C](troubleshoot-custom-policies.md)
