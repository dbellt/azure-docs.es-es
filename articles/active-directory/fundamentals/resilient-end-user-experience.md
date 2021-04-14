---
title: Experiencia del usuario final resistente mediante Azure AD B2C | Microsoft Docs
description: Métodos para generar resistencia en la experiencia del usuario final mediante Azure AD B2C
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: gargi-sinha
ms.author: gasinh
manager: martinco
ms.reviewer: ''
ms.date: 11/30/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53cdbfb3ba531330c37f86233cbed990da4efaf4
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258391"
---
# <a name="resilient-end-user-experience"></a>Experiencia del usuario final resistente

La experiencia de registro e inicio de sesión del usuario final se compone de los siguientes elementos:

- Interfaces con las que interactúa el usuario, como CSS, HTML y JavaScript.

- Flujos de usuario y directivas personalizadas que haya creado, como el registro, el inicio de sesión y la edición del perfil.

- Proveedores de identidades (IDP) de la aplicación, como el nombre de usuario y la contraseña de la cuenta local, Outlook, Facebook y Google.

![En esta imagen se muestran componentes de experiencia del usuario final](media/resilient-end-user-experiences/end-user-experience-architecture.png)

## <a name="choose-between-user-flow-and-custom-policy"></a>Elección entre el flujo de usuario y la directiva personalizada  

Para ayudarle a configurar las tareas de identidad más comunes, Azure AD B2C proporciona [flujos de usuario](../../active-directory-b2c/user-flow-overview.md) configurables e integrados. También puede crear sus propias [directivas personalizadas](../../active-directory-b2c/custom-policy-overview.md), para obtener la máxima flexibilidad. Sin embargo, es recomendable que use las directivas personalizadas solo para abordar escenarios complejos.

### <a name="how-to-decide-between-user-flow-and-custom-policy"></a>Cómo decidir entre el flujo de usuario y la directiva personalizada

Elija los flujos de usuario integrados si pueden cumplir sus requisitos de negocios. Dado que Microsoft los ha probado exhaustivamente, puede minimizar las pruebas necesarias para validar la funcionalidad, el rendimiento o la escala a nivel de directiva de estos flujos de usuario de identidad. Recuerde que todavía tiene que probar la funcionalidad, el rendimiento y la escala de las aplicaciones.

Si [elige directivas personalizadas](../../active-directory-b2c/user-flow-overview.md) debido a sus requisitos de negocio, asegúrese de que realiza pruebas a nivel de directiva en cuanto a la funcionalidad, el rendimiento o la escala, además de las pruebas del nivel de la aplicación.

Consulte el artículo [Comparación de los flujos de usuario y las directivas personalizadas](../../active-directory-b2c/user-flow-overview.md#comparing-user-flows-and-custom-policies) para ayudarle a decidir.

## <a name="choose-multiple-idps"></a>Elección de varios IDP

Al usar un [proveedor de identidades externo](../../active-directory-b2c/technical-overview.md#external-identity-providers) como Facebook, asegúrese de que tiene un plan de reserva en caso de que el proveedor externo deje de estar disponible.

### <a name="how-to-set-up-multiple-idps"></a>Cómo configurar varios IDP

Como parte del proceso de registro del proveedor de identidades externo, incluya una notificación de identidad comprobada, como el número de teléfono móvil o la dirección de correo electrónico del usuario. Confirme las notificaciones comprobadas en la instancia de directorio subyacente de Azure AD B2C. Si el proveedor externo no está disponible, vuelva a las notificaciones de identidad comprobadas y use el número de teléfono como método de autenticación. Otra opción consiste en enviar al usuario un código de acceso de un solo uso para permitirle iniciar sesión.

 Siga estos pasos para [compilar rutas de autenticación alternativas](https://github.com/azure-ad-b2c/samples/tree/master/policies/idps-filter):

 1. Configure la directiva de registro para permitir el inicio de sesión mediante una cuenta local e IDP externos.

 2. Configure una directiva de perfil que permita a los usuarios [vincular la otra identidad a su cuenta](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/account-linking) después de iniciar sesión.

 3. Notifique y permita que los usuarios [cambien a una IDP alternativa](../../active-directory-b2c/customize-ui-with-html.md#configure-dynamic-custom-page-content-uri) si se produce alguna interrupción.

## <a name="availability-of-multi-factor-authentication"></a>Disponibilidad de la autenticación multifactor

Al usar un [servicio de teléfono para la autenticación multifactor (MFA)](../../active-directory-b2c/phone-authentication-user-flows.md), asegúrese de agregar un proveedor de servicios alternativo. Es posible que su proveedor de servicios de telecomunicaciones o de telefonía local experimente interrupciones en el servicio.

### <a name="how-to-choose-an-alternate-mfa"></a>Cómo elegir una MFA alternativa  

El servicio Azure AD B2C usa un proveedor de MFA integrado basado en el teléfono para ofrecer códigos de acceso de un solo uso (OTPs) temporales. Estos códigos se envían mediante llamadas de voz y mensajes de texto al número de teléfono que el usuario registró previamente. Los siguientes métodos alternativos están disponibles para diversos escenarios:

Al usar los **flujos de usuario**, existen dos métodos para crear resistencia:

- **Cambiar la configuración del flujo de usuario**:  tras detectar una interrupción en la entrega de OTP basada en el teléfono, debe cambiar el método de entrega de OTP basado en el teléfono a otro método que esté basado en el correo electrónico; a continuación, vuelva a implementar el flujo de usuario, sin cambiar las aplicaciones.

![captura de pantalla que muestra el inicio de sesión y el registro](media/resilient-end-user-experiences/create-sign-in.png)

- **Cambiar aplicaciones**: en cada tarea de identidad como el registro y el inicio de sesión, debe definir dos conjuntos de flujos de usuario. Configure el primer conjunto para usar un OTP basado en el teléfono y el segundo para usar un OTP basado en el correo electrónico. Si detecta una interrupción en la entrega del OTP basado en el teléfono, cambie y vuelva a implementar las aplicaciones para cambiar del primer conjunto de flujos de usuario al segundo, sin que ello afecte a los flujos de usuario.  

Al usar **directivas personalizadas**, existen cuatro métodos para crear resistencia. La lista siguiente está en orden de complejidad y tendrá que volver a implementar las directivas actualizadas.

- **Habilitar la selección del usuario para un OTP basado en el teléfono o un OTP basado en el correo electrónico**: exponga ambas opciones a los usuarios y permita que seleccionen automáticamente una de ellas. No es necesario realizar cambios en las directivas ni en las aplicaciones.

- **Cambiar dinámicamente entre el OTP basado en el teléfono y el OTP basado en el correo electrónico**:  recopile la información de teléfono y correo electrónico en el registro. Defina de antemano la directiva personalizada para cambiarla de forma condicional si se produce una interrupción del servicio de teléfono; así podrá cambiar la entrega de OTP basada en el teléfono a la entrega basada en el correo electrónico. No es necesario realizar cambios en las directivas ni en las aplicaciones.

- **Usar una aplicación de autenticación**: actualice la directiva personalizada para usar una [aplicación de autenticación](https://github.com/azure-ad-b2c/samples/tree/master/policies/custom-mfa-totp). Si su MFA normal es un OTP basado en el teléfono o en el correo electrónico, vuelva a implementar las directivas personalizadas para usar la aplicación de autenticación.

>[!Note]
>Los usuarios deben configurar la integración de la aplicación de autenticación durante el registro.

- **Usar las preguntas de seguridad**: si ninguno de los métodos anteriores es aplicable, implemente las preguntas de seguridad como una copia de seguridad. Configure las preguntas de seguridad de los usuarios durante la incorporación o la edición de perfiles y almacene las respuestas en una base de datos independiente que no sea el directorio. Este método no cumple el requisito de MFA de tipo "algo que tiene" (por ejemplo, el teléfono), pero ofrece una opción secundaria de tipo "algo que sabe".

## <a name="use-a-content-delivery-network"></a>Uso de una red de entrega de contenido

Las redes de entrega de contenido (CDN) son más eficaces y menos costosas que los almacenes de blobs para la interfaz de usuario personalizada del flujo de usuarios. El contenido de la página web se entrega más rápido desde una red geográficamente distribuida de servidores de alta disponibilidad.  

Pruebe periódicamente la disponibilidad de la red CDN y el rendimiento de la distribución de contenido a través de un escenario de un extremo a otro y las pruebas de carga. Si está planeando realizar un aumento en breve debido al tráfico que generan las promociones o los periodos vacacionales, revise las estimaciones de las pruebas de carga.
  
## <a name="next-steps"></a>Pasos siguientes

- [Recursos de resistencia para desarrolladores de Azure AD B2C](resilience-b2c.md)
  
  - [Interfaces resistentes con procesos externos](resilient-external-processes.md)
  - [Resistencia a través de los procedimientos recomendados para desarrolladores](resilience-b2c-developer-best-practices.md)
  - [Resistencia mediante la supervisión y el análisis](resilience-with-monitoring-alerting.md)
- [Aumento de la resistencia en la infraestructura de autenticación](resilience-in-infrastructure.md)
- [Aumento de la resistencia de la autenticación y autorización en las aplicaciones](resilience-app-development-overview.md)