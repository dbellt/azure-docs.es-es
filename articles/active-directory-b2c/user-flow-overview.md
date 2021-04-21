---
title: Flujos de usuario y directivas personalizadas en Azure Active Directory B2C | Microsoft Docs
titleSuffix: Azure AD B2C
description: Obtenga más información sobre los flujos de usuario integrados y el marco extensible de directivas personalizadas de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/08/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2e4dbc5178bec3a5b1f0931267465879f604f36f
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226015"
---
# <a name="user-flows-and-custom-policies-overview"></a>Información general sobre los flujos de usuario y las directivas personalizadas

En Azure AD B2C, puede definir la lógica de negocios que siguen los usuarios para obtener acceso a su aplicación. Por ejemplo, puede determinar la secuencia de pasos que siguen los usuarios cuando inician sesión, se registran, editan un perfil o restablecen una contraseña. Después de completar la secuencia, el usuario adquiere un token y obtiene acceso a la aplicación. 

En Azure AD B2C, hay dos formas de ofrecer experiencias de usuario relacionadas con la identidad:

* Los **flujos de usuario** son directivas configurables, predefinidas e integradas que se proporcionan para que pueda crear experiencias de registro, inicio de sesión y edición de directivas en cuestión de minutos.

* Las **directivas personalizadas** le permiten crear sus propios recorridos de usuario para escenarios complejos de experiencia de identidad.

En la siguiente captura de pantalla se muestra la interfaz de configuración de flujos de usuario con relación a archivos de configuración de directivas personalizadas.

![En la captura de pantalla se muestra la interfaz de configuración de flujos de usuario con relación a archivos de configuración de directivas personalizadas.](media/user-flow-overview/user-flow-vs-custom-policy.png)

En este artículo se proporciona una breve descripción general de los flujos de usuario y las directivas personalizadas, con el fin de ayudarle a decidir el método que mejor se adecúa a sus necesidades empresariales.

## <a name="user-flows"></a>Flujos de usuario

Para configurar las tareas de identidad más comunes, Azure Portal incluye varias directivas predefinidas y configurables, denominadas *flujos de usuario*.

Puede configurar valores de flujo de usuario como estos para controlar los comportamientos de la experiencia de identidad en las aplicaciones:

* Los tipos de cuenta usados para el inicio de sesión, como cuentas de redes sociales (por ejemplo, Facebook), o cuentas locales que usan una dirección de correo electrónico y una contraseña para el inicio de sesión.
* Los atributos que se recopilan del consumidor, como el nombre, el código postal o el país o región de residencia.
* Azure AD Multi-Factor Authentication (MFA)
* Personalización de la interfaz de usuario
* El conjunto de notificaciones en un token que recibe la aplicación después de que el usuario completa el flujo de usuario.
* Administración de sesiones
* Y mucho más…

En la práctica, la mayoría de los escenarios de identidad comunes para apps se pueden definir e implementar mediante flujos de usuario. Se recomienda usar los flujos de usuario integrados, a menos que tenga escenarios complejos de recorrido del usuario que requieran toda la flexibilidad de las directivas personalizadas.

## <a name="custom-policies"></a>Directivas personalizadas

Las directivas personalizadas son archivos de configuración que definen el comportamiento de la experiencia de usuario del inquilino de Azure AD B2C. Mientras que en el portal de Azure AD B2C se predefinen flujos de usuario para las tareas de identidad más comunes, un desarrollador de identidad puede editar las directivas personalizadas en su totalidad a fin de realizar muchas tareas diferentes.

Una directiva personalizada es totalmente configurable y se controla mediante directivas. Orquesta la confianza entre entidades en protocolos estándar. Por ejemplo, OpenID Connect, OAuth, SAML y otros no estándar, como intercambios de notificaciones de sistema a sistema basados en la API de REST. El marco crea experiencias propias fáciles de usar.

Las directivas personalizadas le ofrecen la posibilidad de crear recorridos de usuario con cualquier combinación de pasos. Por ejemplo:

* Federación con otros proveedores de identidades
* Desafíos de autenticación multifactor (MFA) propios y de terceros
* Recopilación de cualquier dato proporcionado por el usuario
* Integración con sistemas externos mediante la comunicación de la API REST

Cada recorrido del usuario se define mediante una directiva, y puede crear tantas como sea necesario con el fin de posibilitar la mejor experiencia de usuario para su organización.

![Diagrama que muestra un ejemplo de un recorrido de usuario complejo habilitado por IEF](media/user-flow-overview/custom-policy-diagram.png)

Una directiva personalizada se define mediante varios archivos XML que se hacen referencia entre sí en una cadena jerárquica. Los elementos XML definen el esquema de notificaciones, las transformaciones de notificaciones, las definiciones de contenido, los proveedores de notificaciones, los perfiles técnicos, los pasos de orquestación del recorrido del usuario y otros aspectos de la experiencia de identidad.

La gran flexibilidad de las directivas personalizadas es principalmente adecuada cuando es necesario crear escenarios de identidad complejos. Los desarrolladores que configuran las directivas personalizadas deben definir las relaciones de confianza con mucho detalle para incluir puntos de conexión de metadatos, definiciones exactas de intercambio de notificaciones, además de configurar secretos, claves y certificados según lo necesite cada proveedor de identidades.

Más información sobre las directivas personalizadas en [Directivas personalizadas de Azure Active Directory B2C](custom-policy-overview.md).

## <a name="comparing-user-flows-and-custom-policies"></a>Comparación de las directivas personalizadas y los flujos de usuario

En la siguiente tabla se proporciona una comparación detallada de los escenarios posibles con los flujos de usuario y las directivas personalizadas de Azure AD B2C.

| Context | Flujos de usuario | Directivas personalizadas |
|-|-------------------|-----------------|
| Usuarios de destino | Todos los desarrolladores de aplicaciones con o sin conocimientos sobre la identidad. | Profesionales de la identidad, integradores de sistemas, consultores y equipos internos de identidad. Se sienten cómodos con los flujos de OpenID Connect y comprenden a los proveedores de identidades y la autenticación basada en notificaciones. |
| Método de configuración | Azure Portal con una interfaz de usuario (UI) fácil de usar. | Edición directa de archivos XML y posterior carga en Azure Portal. |
| Personalización de la interfaz de usuario | [Personalización completa de la interfaz de usuario](customize-ui-with-html.md), incluido HTML, CSS y [JavaScript](javascript-and-page-layout.md).<br><br>[Compatibilidad multilingüe](language-customization.md) con cadenas personalizadas. | Iguales |
| Personalización de atributos | Atributos estándar y personalizados. | Iguales |
| Administración de tokens y sesiones | [Personalización de tokens](configure-tokens.md) y del [comportamiento de sesiones](session-behavior.md). | Iguales |
| Proveedores de identidades | [Proveedor local](identity-provider-local.md) o [social](add-identity-provider.md) predefinido, como la federación con inquilinos de Azure Active Directory. | OIDC, OAUTH y SAML basados en estándares.  También es posible la autenticación mediante la integración con API REST. |
| Tareas de identidad | [Registro o inicio de sesión](add-sign-up-and-sign-in-policy.md) con cuentas locales o muchas cuentas de redes sociales.<br><br>[Autoservicio de restablecimiento de contraseña](add-password-reset-policy.md).<br><br>[Edición del perfil](add-profile-editing-policy.md).<br><br>Autenticación multifactor.<br><br>Flujos de token de acceso. | Completar las mismas tareas que con los flujos de usuario usando proveedores de identidades personalizados o usar ámbitos personalizados.<br><br>Aprovisionar una cuenta de usuario en otro sistema en el momento del registro.<br><br>Enviar un mensaje de bienvenida con su propio proveedor de servicios de correo electrónico.<br><br>Usar un almacén de usuario externo a Azure AD B2C.<br><br>Validar la información proporcionada por el usuario con un sistema de confianza mediante el uso de una API. |

## <a name="application-integration"></a>Integración de aplicaciones

Puede crear multitud de flujos de usuario o directivas personalizadas de diferentes tipos en su inquilino y usarlos en sus aplicaciones según sea necesario. Tanto los flujos de usuario como las directivas personalizadas se pueden reutilizar entre aplicaciones. Esta flexibilidad le permite definir y modificar las experiencias de identidad con cambios mínimos o ningún cambio en el código. 

Cuando un usuario quiere iniciar sesión en su aplicación, esta inicia una solicitud de autorización a un punto de conexión proporcionado por un flujo de usuario o una directiva personalizada. El flujo de usuario o la directiva personalizada definen y controlan la experiencia del usuario. Al completar un flujo de usuario, Azure AD B2C genera un token y, después, redirige al usuario de vuelta a la aplicación.

![Aplicación móvil con flechas que muestran el flujo entre ella y la página de inicio de sesión de Azure AD B2C](media/user-flow-overview/app-integration.png)

Varias aplicaciones pueden usar el mismo flujo de usuario o directiva personalizada. Una sola aplicación puede usar varios flujos de usuario o directivas personalizadas.

Por ejemplo, para iniciar sesión en una aplicación, la aplicación usa el flujo de usuario de *registro o inicio de sesión*. Una vez que el usuario ha iniciado sesión, es posible que quiera editar su perfil. Para ello, la aplicación inicia otra solicitud de autorización, esta vez usando el flujo de usuario de *edición del perfil*.

La aplicación desencadena un flujo de usuario mediante una solicitud de autenticación HTTP estándar que incluye un nombre de flujo o de directiva personalizada. Se recibe un [token](tokens-overview.md) personalizado como respuesta.


## <a name="next-steps"></a>Pasos siguientes

- Para crear los flujos de usuario recomendados, siga las instrucciones en el [Tutorial: Creación de un flujo de usuario](tutorial-create-user-flows.md).
- Obtenga más información sobre las [versiones de flujos de usuario de Azure AD B2C](user-flow-versions.md).
- Obtenga más información sobre las [directivas personalizadas de Azure AD B2C](custom-policy-overview.md).
