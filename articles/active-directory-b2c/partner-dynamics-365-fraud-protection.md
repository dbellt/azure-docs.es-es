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
ms.date: 02/10/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 8a94500f1669e47065f6ea9789b18442e0c8349e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108127018"
---
# <a name="tutorial-configure-microsoft-dynamics-365-fraud-protection-with-azure-active-directory-b2c"></a>Tutorial: Configuración de Microsoft Dynamics 365 Fraud Protection con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar [Microsoft Dynamics 365 Fraud Protection](/dynamics365/fraud-protection/overview) con Azure Active Directory (AD) B2C.

Microsoft DFP proporciona a los clientes la posibilidad de evaluar si los intentos de crear cuentas e iniciar sesión en el ecosistema del cliente son fraudulentos. El cliente puede usar la evaluación de Microsoft DFP para bloquear o cuestionar los intentos sospechosos de crear nuevas cuentas falsas o poner en peligro las cuentas existentes. La protección de cuentas incluye inteligencia artificial con huellas digitales de dispositivos, API para la evaluación de riesgos en tiempo real, experiencia de reglas y listas para optimizar la estrategia de riesgos según las necesidades empresariales de cada cliente, y un cuadro de mandos para supervisar la eficacia y las tendencias de protección contra fraudes en el ecosistema del cliente.

En este ejemplo, se van a integrar las características de protección de cuentas de Microsoft DFP con un flujo de usuario de Azure AD B2C. El servicio toma huellas digitales de todos los inicios de sesión o los intentos de inicio de sesión y supervisará cualquier comportamiento sospechoso anterior o presente. Azure AD B2C invoca un punto de conexión de decisión de Microsoft DFP, que devuelve un resultado en función de todos los comportamientos pasados y presentes del usuario identificado, así como las reglas personalizadas especificadas en el servicio de Microsoft DFP. Azure AD B2C toma una decisión de aprobación según este resultado y la vuelve a pasar a Microsoft DFP.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md). El inquilino está vinculado a la suscripción de Azure.

- Obtener una [suscripción](https://dynamics.microsoft.com/pricing/#Sales) a Microsoft DFP. También puede configurar una [versión de cliente de prueba](https://dynamics.microsoft.com/ai/fraud-protection/signin/?RU=https%3A%2F%2Fdfp.microsoft.com%2Fsignin).

## <a name="scenario-description"></a>Descripción del escenario

La integración de Microsoft DFP incluye los componentes siguientes:

- **Inquilino de Azure AD B2C**: autentica al usuario y actúa como cliente de Microsoft DFP. Hospeda un script de huellas digitales que recopila datos de identificación y diagnóstico de todos los usuarios que ejecutan una directiva de destino. Posteriormente, bloquea o cuestiona los intentos de inicio de sesión o de registro si Microsoft DFP los considera sospechosos.

- **App Service personalizado**: una aplicación web que cumple dos propósitos.

  - Sirve páginas HTML que se usarán como interfaz de usuario de Identity Experience Framework. Responsable de insertar el script de huellas digitales de Microsoft Dynamics 365.

  - Un controlador de API con puntos de conexión RESTful que conecta Microsoft DFP a Azure AD B2C. El procesamiento de datos del manipulador, la estructura y cumple los requisitos de seguridad de ambos.

- **Servicio de huellas digitales de Microsoft DFP**: script insertado dinámicamente, que registra la telemetría del dispositivo y los detalles del usuario autoafirmado para crear una huella digital identificable de forma única para el usuario que se va a usar más adelante en el proceso de toma de decisiones.

- **Puntos de conexión de la API de Microsoft DFP**: proporciona el resultado de la decisión y acepta un estado final que refleja la operación emprendida por la aplicación cliente. Azure AD B2C no se comunica directamente con los puntos de conexión debido a la variación de los requisitos de seguridad y de carga de API y, en su lugar, usa App Service como intermediario.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra el diagrama de la arquitectura de Microsoft Dynamics 365 Fraud Protection](./media/partner-dynamics365-fraud-protection/microsoft-dynamics-365-fraud-protection-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. Los usuarios optan por iniciar sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario.
| 3. | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de Microsoft DFP puede consumir. Luego, se envía a la API de Microsoft DFP.
| 4. | Una vez que Microsoft DFP consume la información y la procesa, devuelve el resultado a la API de nivel intermedio.
| 5. | La API de nivel intermedio procesa la información y envía la información pertinente de vuelta a Azure AD B2C.
| 6. | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece una respuesta de error, se muestra un mensaje de error al usuario. Si aparece la respuesta Correcto, el usuario se autentica y se escribe en el directorio.

## <a name="set-up-the-solution"></a>Configuración de la solución

1. [Cree una aplicación de Facebook](./identity-provider-facebook.md#create-a-facebook-application) configurada para permitir la federación a Azure AD B2C.
2. [Agregue el secreto de Facebook](./tutorial-create-user-flows.md?pivots=b2c-custom-policy#create-the-facebook-key) que creó como una clave de directiva de Identity Experience Framework.

## <a name="configure-your-application-under-microsoft-dfp"></a>Configuración de la aplicación en Microsoft DFP

[Configure el inquilino de Azure AD](/dynamics365/fraud-protection/integrate-real-time-api) para usar Microsoft DFP.

## <a name="deploy-to-the-web-application"></a>Implementación en la aplicación web

### <a name="implement-microsoft-dfp-service-fingerprinting"></a>Implementación de la huella digital del servicio de Microsoft DFP

La [huella digital de dispositivos de Microsoft DFP](/dynamics365/fraud-protection/device-fingerprinting) es un requisito para la protección de la cuenta de Microsoft DFP.

>[!NOTE]
>Además de las páginas de la interfaz de usuario de Azure AD B2C, el cliente también puede implementar el servicio de huellas digitales dentro del código de la aplicación para una generación de perfiles de dispositivos más integral. El servicio de huellas digitales del código de la aplicación no se incluye en este ejemplo.

### <a name="deploy-the-azure-ad-b2c-api-code"></a>Implementación del código de la API de Azure AD B2C

Implemente el [código de la API proporcionado](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/API) en un servicio de Azure. El código se puede [publicar desde Visual Studio](/visualstudio/deployment/quickstart-deploy-to-azure).

Configure CORS y agregue **Origen permitido** `https://{your_tenant_name}.b2clogin.com`

>[!NOTE]
>Más adelante, necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

Para más información, consulte la [documentación de App Service](../app-service/app-service-web-tutorial-rest-api.md).

### <a name="add-context-dependent-configuration-settings"></a>Incorporación de configuración dependiente del contexto

Ajuste la configuración de la aplicación en [App Service en Azure](../app-service/configure-common.md#configure-app-settings). Esto permite configurar los valores de manera segura sin registrarlos en un repositorio. La API Rest necesita la configuración siguiente:

| Configuración de la aplicación | Source | Notas |
| :-------- | :------------| :-----------|
|FraudProtectionSettings:InstanceId | Configuración de Microsoft DFP |     |
|FraudProtectionSettings:DeviceFingerprintingCustomerId | Id. de cliente de huellas digitales de dispositivos de Microsoft |     |
| FraudProtectionSettings:ApiBaseUrl |  La dirección URL base del portal de Microsoft DFP   | Quite "-int" para llamar a la API de producción en su lugar|
|  TokenProviderConfig: Recurso  | La dirección URL base: `https://api.dfp.dynamics-int.com`     | Quite "-int" para llamar a la API de producción en su lugar|
|   TokenProviderConfig:ClientId       |Id. de la aplicación cliente de Azure AD del comerciante de Fraud Protection      |       |
| TokenProviderConfig:Authority | https://login.microsoftonline.com/<directory_ID> | Autoridad de inquilino de Azure AD del comerciante de Fraud Protection |
| TokenProviderConfig:CertificateThumbprint* | La huella digital del certificado que se va a usar para autenticarse en la aplicación cliente de Azure AD del comerciante |
| TokenProviderConfig:ClientSecret* | El secreto de la aplicación cliente de Azure AD del comerciante | Se recomienda usar un administrador de secretos |

* Establezca solo 1 de los 2 parámetros marcados en función de si se autentica con un certificado o un secreto, como una contraseña.

## <a name="azure-ad-b2c-configuration"></a>Configuración de Azure AD B2C

### <a name="replace-the-configuration-values"></a>Reemplazo de los valores de configuración

En las [directivas personalizadas](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/Policies) proporcionadas, busque los marcadores de posición siguientes y reemplácelos por los valores correspondientes de la instancia.

| Marcador de posición | Reemplazar por | Notas |
| :-------- | :------------| :-----------|
|{your_tenant_name} | Su nombre corto de inquilino |  "suinquilino" de suinquilino.onmicrosoft.com   |
|{your_tenantId} | Id. de inquilino del inquilino de Azure AD B2C |  01234567-89ab-cdef-0123-456789abcdef   |
|  {your_tenant_IdentityExperienceFramework_appid}    |   Identificador de aplicación de la aplicación IdentityExperienceFramework configurada en el inquilino de Azure AD B2C    |  01234567-89ab-cdef-0123-456789abcdef   |
|  {your_tenant_ ProxyIdentityExperienceFramework _appid}     |  Identificador de aplicación de la aplicación ProxyIdentityExperienceFramework configurada en el inquilino de Azure AD B2C      |   01234567-89ab-cdef-0123-456789abcdef     |
|  {your_tenant_extensions_appid}   |  Identificador de aplicación de la aplicación de almacenamiento del inquilino   |  01234567-89ab-cdef-0123-456789abcdef  |
|   {your_tenant_extensions_app_objectid}  | Identificador de objeto de la aplicación de almacenamiento del inquilino    | 01234567-89ab-cdef-0123-456789abcdef   |
|   {your_app_insights_instrumentation_key}  |   Clave de instrumentación de la instancia de App Insights*  |   01234567-89ab-cdef-0123-456789abcdef |
|  {your_ui_base_url}   | Punto de conexión en la instancia de App Service desde donde se atienden los archivos de la interfaz de usuario    | `https://yourapp.azurewebsites.net/B2CUI/GetUIPage`   |
|   {your_app_service_url}  | Dirección URL de su instancia de App Service    |  `https://yourapp.azurewebsites.net`  |
|   {your-facebook-app-id}  |  Identificador de aplicación de la aplicación de Facebook que configuró para la federación con Azure AD B2C   | 000000000000000   |
|  {your-facebook-app-secret}   |  Nombre de la clave de directiva con la que guardó el secreto de la aplicación de Facebook   | B2C_1A_FacebookAppSecret   |

*App Insights puede estar en un inquilino diferente. Este paso es opcional. Quite los TechnicalProfiles y OrchestrationSteps correspondientes si no son necesarios.

### <a name="call-microsoft-dfp-label-api"></a>Llamada a Label API de Microsoft DFP

Los clientes deben [implementar Label API](/dynamics365/fraud-protection/integrate-ap-api). Consulte la [API de Microsoft DFP](https://apidocs.microsoft.com/services/dynamics365fraudprotection#/AccountProtection/v1.0) para más información.

`URI: < API Endpoint >/v1.0/label/account/create/<userId>`

El valor de userID debe ser el mismo que el del valor de configuración de Azure AD B2C correspondiente (ObjectID).

>[!NOTE]
>Agregue notificación de consentimiento a la página colección de atributos. Notifique que la información de la identidad del usuario y la telemetría de los usuarios se registrarán para fines de protección de la cuenta.

## <a name="configure-the-azure-ad-b2c-policy"></a>Configuración de la directiva de Azure AD B2C

1. Vaya a la [directiva de Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Dynamics-Fraud-Protection/Policies) en la carpeta Directivas.

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