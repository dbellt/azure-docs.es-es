---
title: Autenticación y autorización
description: Obtenga información sobre el soporte de autenticación y autorización integrado en Azure App Service y Azure Functions, y cómo puede ayudarle a proteger la aplicación frente al acceso no autorizado.
ms.assetid: b7151b57-09e5-4c77-a10c-375a262f17e5
ms.topic: article
ms.date: 03/29/2021
ms.reviewer: mahender
ms.custom: seodec18, fasttrack-edit, has-adal-ref
ms.openlocfilehash: 1b6e600fcaf32a115af14be2444144fee099d635
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106075345"
---
# <a name="authentication-and-authorization-in-azure-app-service-and-azure-functions"></a>Autenticación y autorización en Azure App Service y Azure Functions

Azure App Service incluye funcionalidades de autenticación y autorización integradas (lo que a veces se denomina "Easy Auth") para que pueda proporcionar inicio de sesión a los usuarios y acceder a los datos escribiendo una cantidad mínima de código (o directamente sin código) en la aplicación web, API RESTful y back-end móvil, así como [Azure Functions](../azure-functions/functions-overview.md). En este artículo se describe cómo App Service le ayuda a simplificar la autenticación y autorización para la aplicación.

## <a name="why-use-the-built-in-authentication"></a>¿Por qué usar la autenticación integrada?

No es necesario usar esta característica para la autenticación y autorización. Puede usar las características de seguridad agrupadas en el marco de trabajo web de su elección o puede escribir sus propias utilidades. Sin embargo, deberá asegurarse de que la solución se mantiene actualizada con las actualizaciones más recientes del explorador, el protocolo y la seguridad.

La implementación de una solución segura para la autenticación (que permite el inicio de sesión de los usuarios) y la autorización (que proporciona acceso a los datos seguros) puede suponer un esfuerzo considerable. Debe asegurarse de seguir los procedimientos recomendados y los estándares del sector, así como mantener la implementación actualizada. La característica de autenticación integrada para App Service y Azure Functions puede ahorrar tiempo y esfuerzo, ya que proporciona autenticación integrada con proveedores de identidades federados, lo que le permite centrarse en el resto de la aplicación.

- Azure App Service le permite integrar numerosas funcionalidades de autenticación en su aplicación web o API sin implementarlas usted mismo.
- Está integrado directamente en la plataforma y no requiere ningún idioma, SDK, experiencia en seguridad ni ningún código en particular.
- Puede integrar con varios proveedores de inicio de sesión. Por ejemplo, Azure AD, Facebook, Google y Twitter.

## <a name="identity-providers"></a>Proveedores de identidades

App Service usa la [identidad federada](https://en.wikipedia.org/wiki/Federated_identity), en la cual un proveedor de identidades de terceros almacena las identidades de usuario y el flujo de autenticación para usted. De forma predeterminada, están disponibles los siguientes proveedores de identidades:

| Proveedor | Punto de conexión de inicio de sesión | Guía de procedimientos |
| - | - | - |
| [Plataforma de identidad de Microsoft](../active-directory/fundamentals/active-directory-whatis.md) | `/.auth/login/aad` | [Inicio de sesión con la Plataforma de identidad de Microsoft para App Service](configure-authentication-provider-aad.md) |
| [Facebook](https://developers.facebook.com/docs/facebook-login) | `/.auth/login/facebook` | [Inicio de sesión con Facebook para App Service](configure-authentication-provider-facebook.md) |
| [Google](https://developers.google.com/identity/choose-auth) | `/.auth/login/google` | [Inicio de sesión con Google para App Service](configure-authentication-provider-google.md) |
| [Twitter](https://developer.twitter.com/en/docs/basics/authentication) | `/.auth/login/twitter` | [Inicio de sesión con Twitter para App Service](configure-authentication-provider-twitter.md) |
| Cualquier proveedor de [OpenID Connect](https://openid.net/connect/) (versión preliminar) | `/.auth/login/<providerName>` | [Inicio de sesión con OpenID Connect para App Service](configure-authentication-provider-openid-connect.md) |

Cuando habilita la autenticación y autorización con uno de estos proveedores, su punto de conexión de inicio de sesión está disponible para la autenticación de usuarios y para la validación de tokens de autenticación del proveedor. Se puede proporcionar a los usuarios cualquier número de estas opciones de inicio de sesión.

## <a name="considerations-for-using-built-in-authentication"></a>Consideraciones sobre el uso de la autenticación integrada

La habilitación de esta característica hará que todas las solicitudes a la aplicación se redirijan automáticamente a HTTPS, con independencia del valor de configuración de App Service para aplicar HTTPS. Puede deshabilitarlo con la configuración de  `requireHttps` en la configuración de V2. Sin embargo, recomendamos seguir con HTTPS, y debe asegurarse de que ningún token de seguridad se transmita a través de conexiones HTTP no seguras.

App Service puede usarse para la autenticación con o sin restringir el acceso al contenido del sitio y a las API. Para restringir el acceso a la aplicación solo a los usuarios autenticados, establezca **Acción necesaria cuando la solicitud no está autenticada** para iniciar sesión con uno de los proveedores de identidades configurados. Para autenticar pero no restringir el acceso, establezca **Acción necesaria cuando la solicitud no está autenticada** en "Permitir solicitudes anónimas (ninguna acción)".

> [!NOTE]
> Debe otorgar al registro de cada aplicación su propio permiso y consentimiento. Evite el uso compartido de permisos entre entornos mediante registros de aplicación independientes para ranuras de implementación independientes. Al probar nuevo código, esta práctica puede ayudar a evitar que los problemas afecten a la aplicación de producción.

## <a name="how-it-works"></a>Funcionamiento

[Arquitectura de características en Windows (implementación sin contenedor)](#feature-architecture-on-windows-non-container-deployment)

[Arquitectura de características en Linux y contenedores](#feature-architecture-on-linux-and-containers)

[Flujo de autenticación](#authentication-flow)

[Comportamiento de la autorización](#authorization-behavior)

[Notificaciones de usuario y aplicación](#user-and-application-claims)

[Almacén de tokens](#token-store)

[Registro y seguimiento](#logging-and-tracing)

#### <a name="feature-architecture-on-windows-non-container-deployment"></a>Arquitectura de características en Windows (implementación sin contenedor)

El módulo de autenticación y autorización se ejecuta en el mismo espacio aislado que el código de aplicación. Cuando está habilitado, cada solicitud HTTP entrante pasa a través de él antes de que el código de aplicación lo controle.

:::image type="content" source="media/app-service-authentication-overview/architecture.png" alt-text="Diagrama de arquitectura que muestra las solicitudes que intercepta un proceso en el espacio aislado del sitio que interactúa con los proveedores de identidades antes de permitir el tráfico al sitio implementado" lightbox="media/app-service-authentication-overview/architecture.png":::

Este módulo controla varios aspectos de la aplicación:

- Autentica a los usuarios con el proveedor especificado
- Valida, almacena y actualiza tokens
- Administra la sesión autenticada
- Inyecta información de identidad en los encabezados de solicitud

El módulo se ejecuta por separado del código de aplicación y se configura mediante los parámetros de la aplicación. No se necesitan SDK, idiomas específicos o cambios en el código de aplicación. 

#### <a name="feature-architecture-on-linux-and-containers"></a>Arquitectura de características en Linux y contenedores

El módulo de autenticación y autorización se ejecuta en un contenedor independiente, aislado del código de la aplicación. Con lo que se conoce como [patrón de embajador](/azure/architecture/patterns/ambassador), interactúa con el tráfico entrante para realizar una funcionalidad similar a la de Windows. Dado que no se ejecuta en proceso, no es posible la integración directa con marcos de lenguaje específicos. Sin embargo, la información pertinente que necesita su aplicación se pasa con encabezados de solicitud, como se explica a continuación.

#### <a name="authentication-flow"></a>Flujo de autenticación

El flujo de autenticación es el mismo para todos los proveedores, pero varía en función de si desea iniciar sesión con el SDK del proveedor:

- Sin el SDK del proveedor: la aplicación delega el inicio de sesión federado a App Service. Por lo general, suele ser el caso de las aplicaciones de explorador, que pueden presentar la página de inicio de sesión del proveedor al usuario. El código del servidor administra el proceso de inicio de sesión, por lo que también se denomina _flujo dirigido por el servidor_ o _flujo de servidor_. Este caso se aplica a las aplicaciones de explorador. También se aplica a las aplicaciones nativas que proporcionan inicio de sesión a los usuarios mediante el SDK de cliente de Mobile Apps porque el SDK abre una vista web para proporcionar inicio de sesión a los usuarios con autenticación de App Service.
- Con el SDK del proveedor: la aplicación inicia manualmente la sesión del usuario con el proveedor y luego envía el token de autenticación a App Service para su validación. Por lo general, suele ser el caso de las aplicaciones sin explorador, que no pueden presentar la página de inicio de sesión del proveedor al usuario. El código de aplicación administra el proceso de inicio de sesión, por lo que también se denomina _flujo dirigido por el cliente_ o _flujo de cliente_. Este caso se aplica a las API REST, [Azure Functions](../azure-functions/functions-overview.md) y los clientes de explorador de JavaScript, así como a las aplicaciones de explorador que necesitan más flexibilidad en el proceso de inicio de sesión. También se aplica a las aplicaciones móviles nativas que proporciona inicio de sesión a los usuarios con el SDK del proveedor.

Las llamadas desde una aplicación de explorador de confianza en App Service a otra REST API de App Service o [Azure Functions](../azure-functions/functions-overview.md) se pueden autenticar mediante el flujo dirigido por el servidor. Para obtener más información, consulte [Personalización de la autenticación y autorización en Azure App Service](app-service-authentication-how-to.md).

En la tabla siguiente se muestran los pasos del flujo de autenticación.

| Paso | Sin el SDK del proveedor | Con el SDK del proveedor |
| - | - | - |
| 1. Inicio de sesión del usuario | Redirige el cliente a `/.auth/login/<provider>`. | El código de cliente proporciona inicio de sesión al usuario directamente con el SDK del proveedor y recibe un token de autenticación. Para información, consulte la documentación del proveedor. |
| 2. Autenticación posterior | El proveedor redirige el cliente a `/.auth/login/<provider>/callback`. | El código de cliente [publica el token del proveedor](app-service-authentication-how-to.md#validate-tokens-from-providers) en `/.auth/login/<provider>` para la validación. |
| 3. Establecer la sesión autenticada | App Service agrega una cookie autenticada a la respuesta. | App Service devuelve su propio token de autenticación al código de cliente. |
| 4. Servir contenido autenticado | El cliente incluye la cookie de autenticación en las solicitudes posteriores (controladas automáticamente por explorador). | El código de cliente presenta el token de autenticación en el encabezado `X-ZUMO-AUTH` (controlado automáticamente por SDK de cliente de Mobile Apps). |

Para los exploradores del cliente, App Service puede dirigir automáticamente todos los usuarios no autenticados a `/.auth/login/<provider>`. También se pueden presentar a los usuarios uno o varios vínculos `/.auth/login/<provider>` para iniciar sesión en la aplicación con sus proveedores preferidos.

<a name="authorization"></a>

#### <a name="authorization-behavior"></a>Comportamiento de la autorización

En [Azure Portal](https://portal.azure.com), puede configurar App Service con varios comportamientos cuando la solicitud entrante no esté autenticada. Los encabezados siguientes describen las opciones.

**Permitir solicitudes no autenticadas**

Esta opción traslada la autorización del tráfico sin autenticar al código de aplicación. Para las solicitudes autenticadas, App Service también transfiere información de autenticación en los encabezados HTTP.

Esta opción proporciona más flexibilidad a la hora de controlar las solicitudes anónimas. Por ejemplo, le permite [presentar varios proveedores de inicio de sesión](app-service-authentication-how-to.md#use-multiple-sign-in-providers) a los usuarios. Sin embargo, debe escribir código.

**Requerir autenticación**

Esta opción rechazará el tráfico no autenticado hacia la aplicación. Este rechazo puede ser una acción de redireccionamiento a uno de los proveedores de identidades configurados. En estos casos, un cliente del explorador se redirige a `/.auth/login/<provider>` para el proveedor que elija. Si la solicitud anónima procede de una aplicación móvil nativa, la respuesta devuelta es `HTTP 401 Unauthorized`. También puede configurar el rechazo para que sea `HTTP 401 Unauthorized` o `HTTP 403 Forbidden` para todas las solicitudes.

Con esta opción, no es necesario escribir ningún código de autenticación en la aplicación. Una autorización más precisa, como la autorización específica de rol, se puede controlarse mediante la inspección de las notificaciones del usuario (consulte [Access user claims](app-service-authentication-how-to.md#access-user-claims) (Acceso a las notificaciones de usuario)).

> [!CAUTION]
> Este método de restricción del acceso se aplica a todas las llamadas a la aplicación, lo que puede no ser deseable para las aplicaciones que necesitan una página de inicio disponible públicamente, como muchas aplicaciones de una sola página.

> [!NOTE]
> De forma predeterminada, cualquier usuario del inquilino de Azure AD puede solicitar un token para la aplicación desde Azure AD. Puede [configurar la aplicación en Azure AD](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md) si desea restringir el acceso a la aplicación a un conjunto definido de usuarios.


#### <a name="user-and-application-claims"></a>Notificaciones de usuario y aplicación

En todos los marcos de lenguaje, App Service hace que las notificaciones del token de entrada (tanto si proceden de un usuario final autenticado como de una aplicación cliente) estén disponibles para el código. Para ello, se insertan en los encabezados de solicitud. Para las aplicaciones de ASP.NET 4.6, App Service rellena [ClaimsPrincipal.Current](/dotnet/api/system.security.claims.claimsprincipal.current) con las notificaciones del usuario autenticado, de forma que usted puede seguir el patrón de código de .NET estándar, incluido el atributo `[Authorize]`. De forma similar, para las aplicaciones PHP, App Service rellena la variable `_SERVER['REMOTE_USER']`. En el caso de las aplicaciones Java, se puede acceder a las notificaciones [desde el servlet Tomcat](configure-language-java.md#authenticate-users-easy-auth).

Para [Azure Functions](../azure-functions/functions-overview.md), `ClaimsPrincipal.Current` no está relleno para el código .NET, pero todavía puede encontrar las notificaciones de usuario en los encabezados de solicitudes, u obtener el objeto `ClaimsPrincipal` del contexto de la solicitud o incluso a través de un parámetro de enlace. Consulte [Uso de identidades de cliente](../azure-functions/functions-bindings-http-webhook-trigger.md#working-with-client-identities) para más información.

Para más información, consulte [Access user claims](app-service-authentication-how-to.md#access-user-claims) (Acceso a las notificaciones de usuario).

En la actualidad, ASP.NET Core no admite el rellenado del usuario actual con la característica de autenticación o autorización. Sin embargo, existen [componentes de middleware de código abierto de terceros](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth) que ayudan a enmendar las carencias.

#### <a name="token-store"></a>Almacén de tokens

App Service proporciona un almacén de tokens integrado, que es un repositorio de tokens que están asociados a los usuarios de las aplicaciones web, API o aplicaciones móviles nativas. Al habilitar la autenticación con cualquier proveedor, este almacén de tokens pasa a estar inmediatamente disponible para la aplicación, si el código de aplicación necesita acceder a los datos de estos proveedores en nombre del usuario, como:

- publicar en la escala de tiempo de Facebook del usuario autenticado
- leer los datos corporativos del usuario mediante Microsoft Graph API

Normalmente, debe escribir código para recopilar, almacenar y actualizar estos tokens en la aplicación. Con el almacén de tokens, simplemente [recupera los tokens](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) cuando los necesita e [indica a App Service que los actualice](app-service-authentication-how-to.md#refresh-identity-provider-tokens) cuando dejan de ser válidos. 

Los tokens de identificador, los tokens de acceso y los tokens de actualización se almacenaron en caché durante la sesión autenticada y solamente el usuario asociado puede acceder a ellos.  

Si no necesita trabajar con tokens en la aplicación, puede deshabilitar el almacén de tokens en la página **Autenticación/Autorización** de la aplicación.

#### <a name="logging-and-tracing"></a>Registro y seguimiento

Si [habilita el registro de aplicaciones](troubleshoot-diagnostic-logs.md), verá los seguimientos de autenticación y autorización directamente en los archivos de registro. Si ve un error de autenticación que no esperaba, puede encontrar cómodamente todos los detalles examinando los registros de aplicaciones existentes. Si habilita el [seguimiento de solicitudes erróneas](troubleshoot-diagnostic-logs.md), puede ver exactamente qué rol puede haber desempeñado el módulo de autenticación y autorización en una solicitud errónea. En los registros de seguimiento, busque las referencias a un módulo denominado `EasyAuthModule_32/64`.

## <a name="more-resources"></a>Más recursos

- [Procedimientos: Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](configure-authentication-provider-aad.md)
- [Uso avanzado de la autenticación y autorización en Azure App Service](app-service-authentication-how-to.md)

Ejemplos:
- [Tutorial: Incorporación de la autenticación a una aplicación web que se ejecuta en Azure App Service](scenario-secure-app-authentication-app-service.md)
- [Tutorial: Autenticación y autorización de usuarios de un extremo a otro en Azure App Service (Windows)](tutorial-auth-aad.md)
- [Integración de .NET Core de Azure AppService EasyAuth (de terceros)](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth)
- [Puesta en funcionamiento de la autenticación de Azure App Service con .NET Core (de terceros)](https://github.com/kirkone/KK.AspNetCore.EasyAuthAuthentication)
