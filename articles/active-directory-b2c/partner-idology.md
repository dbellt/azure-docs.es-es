---
title: Integración de IDology con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar una aplicación de pago en línea de ejemplo en Azure AD B2C con IDology. IDology es un proveedor de pruebas y comprobación de identidades que ofrece varias soluciones.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: df0fcaf7987e30f9c2599346aaef5fcabb4f04e9
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256657"
---
# <a name="tutorial-for-configuring-idology-with-azure-active-directory-b2c"></a>Tutorial para configurar IDology con Azure Active Directory B2C 

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure AD B2C con [IDology](https://www.idology.com/solutions/). IDology es un proveedor de pruebas y comprobación de identidades que ofrece varias soluciones. En este ejemplo, trataremos la solución ExpectID de IDology.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

## <a name="scenario-description"></a>Descripción del escenario

La integración de IDology incluye los siguientes componentes:

- Azure AD B2C: servidor de autorización responsable de comprobar las credenciales del usuario. También se conoce como proveedor de identidades.
- IDology: el servicio IDology toma la entrada proporcionada por el usuario y comprueba la identidad del usuario.
- API de REST personalizada: esta API implementa la integración entre Azure AD y el servicio IDology.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Diagrama de la arquitectura de IDology](media/partner-idology/idology-architecture-diagram.png)

| Paso | Descripción |
|------|------|
|1     | Un usuario llega a la página de inicio de sesión. |
|2     | El usuario selecciona la opción de registro para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario. |
|3     | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario. |
|4     | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de IDology puede consumir. A continuación, envía la información a IDology. |
|5     | IDology utiliza la información y la procesa. A continuación, devuelve el resultado a la API de nivel intermedio. |
|6     | La API de nivel intermedio procesa la información y envía la información pertinente de nuevo a Azure AD B2C. |
|7     | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece la respuesta **Error**, se muestra un mensaje de error al usuario. Si aparece la respuesta **Correcto**, el usuario se autentica y se escribe en el directorio. |
|      |      |

> [!NOTE]
> Azure AD B2C también puede pedir al cliente que realice la autenticación de actualización a edición superior, pero este escenario está fuera del ámbito de este tutorial.

## <a name="onboard-with-idology"></a>Incorporación con IDology

1. IDology proporciona una variedad de soluciones, que puede encontrar [aquí](https://www.idology.com/solutions/). En este ejemplo, usamos ExpectID.

2. Para crear una cuenta de IDology, póngase en contacto con [IDology](https://www.idology.com/request-a-demo/microsoft-integration-signup/).

3. Una vez creada la cuenta, recibirá la información que necesita para la configuración de la API. El proceso se describe en las secciones siguientes.

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

### <a name="part-1---deploy-the-api"></a>Parte 1: implementación de la API

Implemente el [código de la API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/IDology/Api) proporcionado en un servicio de Azure. El código se puede publicar desde Visual Studio siguiendo estas [instrucciones](/visualstudio/deployment/quickstart-deploy-to-azure).

Necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

### <a name="part-2---configure-the-api"></a>Parte 2: configuración de la API 

La aplicación se puede [configurar en App Service en Azure](../app-service/configure-common.md#configure-app-settings). Con este método, los valores se pueden configurar de forma segura sin registrarlos en un repositorio. Debe especificar la configuración siguiente para la API de REST:

| Configuración de la aplicación | Source | Notas |
| :-------- | :------------| :-----------|
|IdologySettings:ApiUsername | Configuración de la cuenta de IDology |     |
|IdologySettings:ApiPassword | Configuración de la cuenta de IDology |     |
|WebApiSettings:ApiUsername |Defina un nombre de usuario para la API| Se usa en la configuración de ExtId |
|WebApiSettings:ApiPassword | Defina una contraseña para la API | Se usa en la configuración de ExtId

### <a name="part-3---create-api-policy-keys"></a>Parte 3: creación de claves de directiva de API

Siga este [documento](secure-rest-api.md#add-rest-api-username-and-password-policy-keys) para crear dos claves de directiva: una para el nombre de usuario de la API y otra para la contraseña de la API que definió anteriormente.

La directiva de ejemplo utiliza estos nombres de clave:

* B2C_1A_RestApiUsername
* B2C_1A_RestApiPassword

### <a name="part-4---configure-the-azure-ad-b2c-policy"></a>Parte 4: configuración de la directiva de Azure AD B2C

1. Siga este [documento](tutorial-create-user-flows.md?pivots=b2c-custom-policy#custom-policy-starter-pack) para descargar el [paquete de inicio de LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts) y configure la directiva para el inquilino de Azure AD B2C. Siga las instrucciones hasta que haya completado la sección **Prueba de la directiva personalizada**.

2. Descargue las dos directivas de ejemplo [aquí](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/IDology/policy).

3. Actualice las dos directivas de ejemplo:

   1. Abra ambas directivas:

      1. En la sección `Idology-ExpectId-API`, actualice el elemento de metadatos `ServiceUrl` con la ubicación de la API implementada anteriormente.

      1. Reemplace `yourtenant` por el nombre del inquilino de Azure AD B2C.
      Por ejemplo, si el nombre del inquilino de Azure AD B2C es  `contosotenant`, reemplace todas las instancias de `yourtenant.onmicrosoft.com` por `contosotenant.onmicrosoft.com`.

   1. Abra el archivo TrustFrameworkExtensions.xml:

      1. Busque el elemento `<TechnicalProfile Id="login-NonInteractive">`. Reemplace ambas instancias de `IdentityExperienceFrameworkAppId` por el identificador de la aplicación IdentityExperienceFramework que creó antes.

      1. Reemplace ambas instancias de `ProxyIdentityExperienceFrameworkAppId` por el identificador de la aplicación ProxyIdentityExperienceFramework que creó antes.

4. Reemplace los archivos SignInorSignUp.xml y TrustFrameworkExtensions.xml cargados anteriormente en Azure AD B2C en el paso 1 por las dos directivas de ejemplo actualizadas.

> [!NOTE]
> Como procedimiento recomendado, se sugiere que los clientes agreguen una notificación de consentimiento en la página de colección de atributos. Notifique a los usuarios que la información se enviará a servicios de terceros para la verificación de identidad.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en **Directivas**, seleccione **Flujos de usuario**.

2. Seleccione el **flujo de usuario** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   1. **Aplicación**: seleccione la aplicación registrada (en el ejemplo es JWT).

   1. **URL de respuesta**: seleccione la dirección **URL de redireccionamiento**.

   1. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Cierre la sesión.

6. Recorra el flujo de inicio de sesión.

7. El rompecabezas IDology aparecerá después de introducir **continuar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)
