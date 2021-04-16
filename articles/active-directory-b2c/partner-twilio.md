---
title: Aplicación Twilio Verify con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar una aplicación de pago en línea de ejemplo en Azure AD B2C con la API de Twilio Verify. Cumpla los requisitos de transacciones de PSD2 (Directiva de servicios de pago 2) a través de la vinculación dinámica y la autenticación sólida del cliente.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9a7f22b05238fb495422a02edd6ca382b4df041a
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257813"
---
# <a name="integrating-twilio-verify-app-with-azure-active-directory-b2c"></a>Integración de la aplicación Twilio Verify con Azure Active Directory B2C

En este tutorial, aprenderá a integrar una aplicación de pago en línea de ejemplo en Azure Active Directory B2C (Azure AD B2C) con la API de Twilio Verify. Con la aplicación Twilio Verify, los clientes de Azure AD B2C pueden cumplir los requisitos para las transacciones de PSD2 (Directiva de servicios de pago 2) a través de la vinculación dinámica y la autenticación sólida del cliente.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
* Una [cuenta de prueba](https://www.twilio.com/try-twilio) en Twilio.

## <a name="scenario-description"></a>Descripción del escenario

Los componentes siguientes forman la solución Twilio:

- [Aplicación web de demostración de .NET](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App) PSD2, que ofrece la posibilidad de iniciar sesión o registrarse, y realizar una transacción ficticia de alto riesgo.
- [Directiva de inicio de sesión y registro](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy) combinada de Azure AD B2C.
- Directiva de Azure AD B2C integrada con la API de Twilio Verify mediante `id_token_hint`.
- Aplicación web de .NET, que hospeda un punto de conexión `.well-known` de OpenIdConnect para permitir la validación de un objeto `id_token_hint`.


    ![flujo de Twilio](media/partner-twilio/twilio-flow.png)

| Paso | Descripción |
|------|------|
| 1     | El usuario inicia el proceso de inicio de sesión o registro en la aplicación de demostración de PSD2. El usuario se autentica mediante la directiva de inicio de sesión y registro de Azure AD B2C. Se devuelve un token en la aplicación. En el registro, el número de teléfono del usuario se comprueba por SMS o teléfono y se registra en su cuenta de Azure AD B2C.     |
| 2     | El usuario inicia una transacción de alto riesgo, como una transferencia de 50,00 USD. El token de acceso actual del usuario se evalúa para PolicyId con el fin de determinar si el usuario ya se ha autenticado a través de una directiva personalizada de actualización a edición superior.     |
| 3     | La aplicación registra el valor de la transacción y el beneficiario, 50,00 USD y John Doe, y genera un token firmado. Este token se denomina `id_token_hint` y contiene la notificación `amount:$500, payee:john doe`. El token `id_token_hint` se envía junto con la solicitud a la directiva personalizada de Azure AD B2C, que se integra con Twilio.     |
| 4     | Azure AD B2C comprueba la firma del token id_token_hint mediante la comprobación del punto de conexión `/.well-known` de OpenId Connect de las aplicaciones. Después de realizar la comprobación, extrae las notificaciones de este token, especialmente `amount` y `payee`. El usuario verá una página para comprobar su número de teléfono móvil a través de un mensaje SMS.     |
| 5     | El usuario solicita que se compruebe su número de teléfono a través de un mensaje SMS, y Azure AD B2C realiza una solicitud de API de REST al punto de conexión de la API de Twilio Verify. También envía la transacción `amount` y `payee` como parte del proceso PSD2 para generar el código de acceso de un solo uso (OTP). Twilio envía un mensaje SMS al número de teléfono registrado del usuario.     |
| 6     |  El usuario escribe el OTP recibido en el mensaje SMS y lo envía a Azure AD B2C. Azure AD B2C realiza una solicitud de API con este OTP a la API de Twilio Verify para comprobar que el OTP es correcto. Por último, se emite un token a la aplicación, con un nuevo PolicyId que indica que el usuario ha actualizado su autenticación a una edición superior.    |
|      |      |

## <a name="onboard-with-twilio"></a>Incorporación a Twilio

1. Obtenga una [cuenta de prueba](https://www.twilio.com/try-twilio) en Twilio.

2. Compre un número de teléfono en Twilio como se describe en [este artículo](https://support.twilio.com/hc/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console).

3. Vaya a la [API de Verify](https://www.twilio.com/console/verify/services) en la consola de Twilio y siga las [instrucciones](https://www.twilio.com/docs/verify/verifying-transactions-psd2) para crear un servicio y habilitar la opción PSD2.  

## <a name="configure-the-psd2-demo-app"></a>Configuración de la aplicación de demostración de PSD2

1. Abra la solución B2C-WebAPI-DotNet y reemplace los valores siguientes por sus propios valores específicos del inquilino en el archivo web.config:

    ```xml
   <add key="ida:Tenant" value="yourtenant.onmicrosoft.com" />
   <add key="ida:TenantId" value="d6f33888-0000-4c1f-9b50-1590f171fc70" />
   <add key="ida:ClientId" value="6bd98cc8-0000-446a-a05e-b5716ef2651b" />
   <add key="ida:ClientSecret" value="secret" />
   <add key="ida:AadInstance" value="https://yourtenant.b2clogin.com/tfp/{0}/{1}" />
   <add key="ida:RedirectUri" value="https://your hosted psd2 demo app url/" />
   ```

2. La [aplicación web](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App) también hospeda el generador de sugerencias de token de identificador y el punto de conexión de metadatos.
   - Cree el certificado de firma tal y como se describe en esta [descripción de ejemplo](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#creating-a-signing-certificate).
   - Actualice las líneas siguientes en función del certificado del archivo web.config:
   
   ```xml
   <add key="ida:SigningCertThumbprint" value="4F39D6014818082CBB763E5BA5F230E545212E89" />
   <add key="ida:SigningCertAlgorithm" value="RS256" />
   ```

3. Cargue la aplicación de demostración en el proveedor de hospedaje que prefiera. La guía de Azure App Service se proporciona en [esta descripción del ejemplo](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#hosting-the-application-in-azure-app-service), incluidas las instrucciones para cargar el certificado.

4. Actualice el registro de la aplicación Azure AD B2C. Para ello, agregue una dirección URL de respuesta equivalente a la dirección URL en la que se hospeda la aplicación.

5. Abra los [archivos de directivas](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy) y reemplace todas las instancias de  `contoso` por su nombre de inquilino.

6. Busque el perfil técnico de la API de REST de Twilio **Custom-SMS-Enroll**. Actualice el objeto `ServiceURL` con su AccountSID de Twilio y el número De al número de teléfono que ha adquirido.

7. Busque los perfiles técnicos de la API de REST de Twilio, **TwilioRestAPI-Verify-Step1** y **TwilioRestAPI-Verify-Step2**, y actualice el objeto `ServiceURL` con su AccountSID de Twilio.

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

Agregue los archivos de directiva a Azure AD B2C:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.

2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.

3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.

4. Navegue a **Azure AD B2C** > **Marco de experiencia de identidad** > **Claves de directiva**.

5. Agregue una nueva clave con el nombre **B2cRestTwilioClientId**. Seleccione **manual** y proporcione el valor de AccountSID de Twilio.

6. Agregue una nueva clave con el nombre **B2cRestTwilioClientSecret**. Seleccione **manual** y proporcione el valor de AUTH TOKEN de Twilio.

7. Cargue todos los archivos de directiva en el inquilino.

8. Personalice la cadena en la transformación de notificaciones GenerateOTPMessageEnrol para el texto del mensaje SMS de registro.

## <a name="test-the-solution"></a>Probar la solución

* Vaya a la aplicación y pruebe las acciones de inicio de sesión, registro y envío de dinero.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- Consulte los [ejemplos de código de integración de Twilio](https://github.com/azure-ad-b2c/samples/tree/master/policies/twilio-mfa-psd2) en GitHub.  

- [Directivas personalizadas de AAD B2C](custom-policy-overview.md)

- [Introducción a las directivas personalizadas en AAD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)
