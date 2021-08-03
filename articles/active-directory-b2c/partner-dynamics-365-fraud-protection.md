---
title: Tutorial para configurar Azure Active Directory B2C con Microsoft Dynamics 365 Fraud Protection
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con Microsoft Dynamics 365 Fraud Protection para identificar una cuenta peligrosa y fraudulenta
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 5/12/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 219cb793da7835922ad707d0ad1ee7e122990ba8
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962181"
---
# <a name="tutorial-configure-microsoft-dynamics-365-fraud-protection-with-azure-active-directory-b2c"></a>Tutorial: Configuración de Microsoft Dynamics 365 Fraud Protection con Azure Active Directory B2C

En este tutorial de ejemplo, aprenderá a integrar [Microsoft Dynamics 365 Fraud Protection](/dynamics365/fraud-protection/overview) (DFP) con Azure Active Directory (AD) B2C.

Microsoft DFP proporciona a las organizaciones la capacidad de evaluar el riesgo de los intentos de creación de cuentas e inicios de sesión fraudulentos. El cliente puede usar la evaluación de Microsoft DFP para bloquear o investigar los intentos sospechosos de crear nuevas cuentas falsas o poner en peligro las cuentas existentes.

En este ejemplo se muestra cómo incorporar la huella digital del dispositivo de Microsoft DFP y cómo crear cuentas y puntos de conexión de la API de evaluación de inicio de sesión en una directiva personalizada de Azure AD B2C.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md). El inquilino está vinculado a la suscripción de Azure.

- Obtener una [suscripción](https://dynamics.microsoft.com/pricing/#Sales) a Microsoft DFP. También puede configurar una [versión de cliente de prueba](https://dynamics.microsoft.com/ai/fraud-protection/signin/?RU=https%3A%2F%2Fdfp.microsoft.com%2Fsignin).

## <a name="scenario-description"></a>Descripción del escenario

La integración de Microsoft DFP incluye los componentes siguientes:

- **Inquilino de Azure AD B2C**: autentica al usuario y actúa como cliente de Microsoft DFP. Hospeda un script de huellas digitales que recopila datos de identificación y diagnóstico de todos los usuarios que ejecutan una directiva de destino. Los bloqueos o desafíos posteriores de los intentos de inicio de sesión o registro se basan en el resultado de evaluación de las reglas que devuelve Microsoft DFP.

- **Plantillas de interfaz de usuario personalizadas**: se usan para personalizar el contenido HTML de las páginas que representa Azure AD B2C. Estas páginas incluyen el fragmento de código de JavaScript necesario para la huella digital de Microsoft DFP.

- **Servicio de huellas digitales de Microsoft DFP**: script insertado dinámicamente, que registra la telemetría del dispositivo y los detalles del usuario autoafirmado para crear una huella digital identificable de forma única para el usuario que se va a usar más adelante en el proceso de toma de decisiones.

- **Puntos de conexión de la API de Microsoft DFP**: proporciona el resultado de la decisión y acepta un estado final que refleja la operación emprendida por la aplicación cliente. Azure AD B2C se comunica directamente con los puntos de conexión de Microsoft DFP mediante conectores de API REST. La autenticación de API se produce mediante la concesión de client_credentials al inquilino de Azure AD en el que Microsoft DFP tiene licencia y en el que está instalado para obtener un token de portador.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra el diagrama de la arquitectura de Microsoft Dynamics 365 Fraud Protection](./media/partner-dynamics365-fraud-protection/microsoft-dynamics-365-fraud-protection-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. Los usuarios optan por iniciar sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a la API de Microsoft DFP y pasa los atributos de usuario.
| 3. | Una vez que Microsoft DFP consume la información y la procesa, devuelve el resultado a Azure AD B2C.
| 4. | Azure AD B2C recibe información de la API de Microsoft DFP. Si aparece una respuesta de error, se muestra un mensaje de error al usuario. Si aparece la respuesta Correcto, el usuario se autentica y se escribe en el directorio.

## <a name="set-up-the-solution"></a>Configuración de la solución

1. [Cree una aplicación de Facebook](./identity-provider-facebook.md#create-a-facebook-application) configurada para permitir la federación a Azure AD B2C.
2. [Agregue el secreto de Facebook](./tutorial-create-user-flows.md?pivots=b2c-custom-policy#create-the-facebook-key) que creó como una clave de directiva de Identity Experience Framework.

## <a name="configure-your-application-under-microsoft-dfp"></a>Configuración de la aplicación en Microsoft DFP

[Configure el inquilino de Azure AD](/dynamics365/fraud-protection/integrate-real-time-api) para usar Microsoft DFP.

## <a name="set-up-your-custom-domain"></a>Configuración del dominio personalizado

En un entorno de producción, debe usar un [dominio personalizado para Azure AD B2C](./custom-domain.md?pivots=b2c-custom-policy) y para el [servicio de huella digital de Microsoft DFP](/dynamics365/fraud-protection/device-fingerprinting#set-up-dns). El dominio de ambos servicios debe estar en la misma zona DNS raíz para evitar que la configuración de privacidad del explorador bloquee las cookies entre dominios y no es necesario en un entorno que no sea de producción.

El siguiente es un ejemplo:

| Entorno | Servicio | Domain |
|:------------|:---------------|:---------------|
| Desarrollo | Azure AD B2C | contoso-dev.b2clogin.com |
| Desarrollo | Servicio de huella digital de Microsoft DFP | fpt.dfp.microsoft-int.com |
| UAT | Azure AD B2C | contoso-uat.b2clogin.com |
| UAT | Servicio de huella digital de Microsoft DFP | fpt.dfp.microsoft.com |
| Producción | Azure AD B2C | login.contoso.com |
| Producción | Servicio de huella digital de Microsoft DFP | fpt.login.contoso.com |

## <a name="deploy-the-ui-templates"></a>Implementación de las plantillas de la interfaz de usuario

1. Implemente las [plantillas de la interfaz de usuario de Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/ui-templates) proporcionadas en un servicio de hospedaje de Internet de acceso público como Azure Blob Storage.

2. Reemplace el valor `https://<YOUR-UI-BASE-URL>/` por la dirección URL raíz de la ubicación de implementación.

  >[!NOTE]
  >Más adelante necesitará la dirección URL base para configurar las directivas de Azure AD B2C.

3. En el archivo `ui-templates/js/dfp.js`, reemplace `<YOUR-DFP-INSTANCE-ID>` por el identificador de la instancia de Microsoft DFP.

4. Asegúrese de que CORS está habilitado para el nombre de dominio `https://{your_tenant_name}.b2clogin.com` o `your custom domain` de Azure AD B2C.

Consulte la [documentación de personalización de la interfaz de usuario](./customize-ui-with-html.md?pivots=b2c-custom-policy) para más información.

## <a name="azure-ad-b2c-configuration"></a>Configuración de Azure AD B2C

### <a name="add-policy-keys-for-your-microsoft-dfp-client-app-id-and-secret"></a>Incorporación de claves de directiva para el identificador y el secreto de la aplicación cliente de Microsoft DFP

1. En el inquilino de Azure AD donde está configurado Microsoft DFP, cree una [aplicación de Azure AD y conceda el consentimiento del administrador](/dynamics365/fraud-protection/integrate-real-time-api#create-azure-active-directory-applications).
2. Cree un valor secreto para este registro de aplicación y anote el valor del identificador y el secreto de cliente de la aplicación.
3. Guarde los valores de identificador y secreto de cliente como [claves de directiva en el inquilino de Azure AD B2C](./policy-keys-overview.md).

 >[!NOTE]
 >Más adelante necesitará las claves de directiva para configurar las directivas de Azure AD B2C.

### <a name="replace-the-configuration-values"></a>Reemplazo de los valores de configuración

En las [directivas personalizadas](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/policies) proporcionadas, busque los marcadores de posición siguientes y reemplácelos por los valores correspondientes de la instancia.

| Marcador de posición | Reemplazar por | Notas |
| :-------- | :------------| :-----------|
|{Settings:Production} | Si se deben implementar las directivas en el modo de producción | `true` o `false`  |
|{Settings:Tenant} | Su nombre corto de inquilino |  `your-tenant` - desde your-tenant.onmicrosoft.com  |
| {Settings:DeploymentMode}    |  Modo de implementación de Application Insights que se usará   |  `Production` o `Development`  |
|  {Settings:DeveloperMode}    | Si se deben implementar las directivas en el modo de desarrollador de Application Insights      |   `true` o `false`    |
|  {Settings:AppInsightsInstrumentationKey}  |  Clave de instrumentación de la instancia de Application Insights*   |  `01234567-89ab-cdef-0123-456789abcdef` |
|  {Settings:IdentityExperienceFrameworkAppId}  | Identificador de aplicación de la aplicación IdentityExperienceFramework configurada en el inquilino de Azure AD B2C  | `01234567-89ab-cdef-0123-456789abcdef`|
|  {Settings:ProxyIdentityExperienceFrameworkAppId}  |   Identificador de aplicación de la aplicación ProxyIdentityExperienceFramework configurada en el inquilino de Azure AD B2C |   `01234567-89ab-cdef-0123-456789abcdef`|
| {Settings:FacebookClientId}  | Identificador de la aplicación de Facebook que configuró para la federación con B2C    | `000000000000000`   |
|   {Settings:FacebookClientSecretKeyContainer}  | Nombre de la clave de directiva con la que guardó el secreto de la aplicación de Facebook    |  `B2C_1A_FacebookAppSecret` |
|   {Settings:ContentDefinitionBaseUri} |  Punto de conexión en el que implementó los archivos de interfaz de usuario   | `https://<my-storage-account>.blob.core.windows.net/<my-storage-container>`   |
|  {Settings:DfpApiBaseUrl}   |  La ruta de acceso base para la instancia de la API de DFP, que se encuentra en el portal de DFP   | `https://tenantname-01234567-89ab-cdef-0123-456789abcdef.api.dfp.dynamics.com/v1.0/`   |
| {Settings:DfpApiAuthScope} | El ámbito client_credentials del servicio de API de DFP | `https://api.dfp.dynamics-int.com/.default or https://api.dfp.dynamics.com/.default` |
| {Settings:DfpTenantId} | El identificador del inquilino de Azure AD (no B2C) donde DFP tiene licencia y está instalado | `01234567-89ab-cdef-0123-456789abcdef` o `consoto.onmicrosoft.com` |
| {Settings:DfpAppClientIdKeyContainer} | Nombre de la clave de directiva en la que se guarda el identificador de cliente de DFP | `B2C_1A_DFPClientId` |
| {Settings:DfpAppClientSecretKeyContainer} | Nombre de la clave de directiva en la que se guarda el secreto de cliente de DFP | `B2C_1A_DFPClientSecret` |

*Application Insights se puede configurar en cualquier inquilino o suscripción de Azure AD. Este valor es opcional, pero se [recomienda para ayudar con la depuración](./troubleshoot-with-application-insights.md).

>[!NOTE]
>Agregue notificación de consentimiento a la página colección de atributos. Notifique que la información de la identidad del usuario y la telemetría de los usuarios se registrarán para fines de protección de la cuenta.

## <a name="configure-the-azure-ad-b2c-policy"></a>Configuración de la directiva de Azure AD B2C

1. Vaya a la [directiva de Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/policies) en la carpeta Directivas.

2. Siga este [documento](./tutorial-create-user-flows.md?pivots=b2c-custom-policy?tabs=applications#custom-policy-starter-pack) para descargar el [paquete de inicio de LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

3. Configure la directiva para el inquilino de Azure AD B2C.

>[!NOTE]
>Actualice las directivas proporcionadas para que se relacionen con su inquilino específico.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Identity Experience Framework**.

2. Seleccione el valor de **SignUpSignIn** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. **Aplicación**: seleccione la aplicación registrada (el ejemplo es JWT).

   b. **URL de respuesta**: seleccione la **dirección URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará al servicio Microsoft DFP durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

>[!NOTE]
>Actualice las reglas directamente en el portal de Microsoft DFP si usa el [motor de reglas de Microsoft DFP](/dynamics365/fraud-protection/rules).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Ejemplos de Microsoft DFP](https://github.com/Microsoft/Dynamics-365-Fraud-Protection-Samples)

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy)