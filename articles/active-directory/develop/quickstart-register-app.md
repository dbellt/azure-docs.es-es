---
title: 'Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft | Azure'
description: En este inicio rápido, aprenderá cómo registrar una aplicación mediante la plataforma de identidad de Microsoft.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 05/04/2021
ms.author: marsma
ms.custom: aaddev, identityplatformtop40, contperf-fy21q1, contperf-fy21q2
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 48d214dfc256a1ed8b770df2ff54646de17b80fe
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748752"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft

En este inicio rápido, va a registrar una aplicación en Azure Portal con el fin de que la plataforma de identidad de Microsoft pueda proporcionar servicios de autenticación y autorización para la aplicación y sus usuarios.

La plataforma de identidad de Microsoft realiza la administración de identidades y acceso (IAM) solo para las aplicaciones registradas. Tanto si se trata de una aplicación cliente (por ejemplo, móvil o web) como de una API web que respalda una aplicación cliente, al registrarlas se establece una relación de confianza entre la aplicación y el proveedor de identidades, es decir, la plataforma de identidad de Microsoft.

> [!TIP]
> Para registrar una aplicación en Azure AD B2C, siga los pasos que se indican en [Tutorial: Registro de una aplicación web en Azure AD B2C](../../active-directory-b2c/tutorial-register-applications.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Finalización del inicio rápido [Configuración de un inquilino](quickstart-create-new-tenant.md).

## <a name="register-an-application"></a>Registro de una aplicación

El registro de la aplicación establece una relación de confianza entre la aplicación y la plataforma de identidad de Microsoft. La confianza es unidireccional: la aplicación confía en la plataforma de identidad de Microsoft y no al revés.

Siga estos pasos para crear el registro de la aplicación:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino donde desea registrar una aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
1. Escriba un **Nombre** para mostrar para la aplicación. Los usuarios de la aplicación pueden ver el nombre para mostrar cuando usan la aplicación, por ejemplo, durante el inicio de sesión.
   Puede cambiar el nombre para mostrar en cualquier momento y varios registros de aplicaciones pueden compartir el mismo nombre. El identificador de la aplicación (cliente) generado automáticamente del registro de la aplicación, y no su nombre para mostrar, identifica de forma única la aplicación en la plataforma de identidades.
1. Especifique qué personas pueden usar la aplicación. A veces, se conoce a estas personas como _público de inicio de sesión_.

   | Tipos de cuenta admitidos                                                      | Descripción                                                                                                                                                                                                                                                                                                                                                                                 |
   | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | **Solo las cuentas de este directorio organizativo**                           | Seleccione esta opción si va a desarrollar una aplicación para que la utilicen usuarios (o invitados) de _su_ inquilino.<br><br>A menudo denominada aplicación de _línea de negocio_, se trata de una aplicación de _un solo inquilino_ en la plataforma de identidad de Microsoft.                                                                                                                                          |
   | **Cuentas en cualquier directorio organizativo**                                 | Seleccione esta opción si desea que los usuarios de _cualquier_ inquilino de Azure Active Directory (Azure AD) pueda usar la aplicación. Esta opción es adecuada si, por ejemplo, va a desarrollar una aplicación de software como servicio (SaaS) que desea proporcionar a varias organizaciones.<br><br>Este tipo de aplicación se denomina _multiinquilino_ en la plataforma de identidad de Microsoft. |
   | **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales** | Seleccione esta opción para establecer como destino el mayor conjunto posible de clientes.<br><br>Al seleccionar esta opción, estará registrando una aplicación _multiinquilino_ que también admite usuarios con _cuentas personales de Microsoft_.                                                                                                                                                                               |
   | **Cuentas personales de Microsoft**                                              | Seleccione esta opción si va a crear una aplicación solo para usuarios con cuentas personales de Microsoft. Las cuentas personales de Microsoft abarcan las cuentas de Skype, Xbox, Live y Hotmail.                                                                                                                                                                                                      |

1. No escriba nada en **URI de redirección (opcional)** . Configurará un URI de redirección en la sección siguiente.
1. Seleccione **Registrar** para completar el registro inicial de la aplicación.

   :::image type="content" source="media/quickstart-register-app/portal-02-app-reg-01.png" alt-text="Captura de pantalla de Azure Portal en un explorador web que muestra el panel Registrar una aplicación":::.

Cuando finaliza el registro, en Azure Portal se muestra el panel **Información general** del registro de la aplicación. Verá el **id. de aplicación (cliente)** . Este valor, también conocido como _Id. de cliente_, identifica de forma única la aplicación en la plataforma de identidad de Microsoft.

> [!IMPORTANT]
> De forma predeterminada, los nuevos registros de aplicaciones están ocultos a los usuarios. Cuando esté listo para que los usuarios puedan ver la aplicación en su [página Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md), puede habilitarla. Para habilitar la aplicación, en Azure Portal vaya a **Azure Active Directory** > **Aplicaciones empresariales** y seleccione la aplicación. Después, en la página **Propiedades**, cambie **¿Es visible para los usuarios?** a Sí.

El código de la aplicación, o lo que es más común, una biblioteca de autenticación que se usa en la aplicación, también se sirve del identificador de cliente. El identificador forma parte de la validación de los tokens de seguridad que recibe de la plataforma de identidad.

:::image type="content" source="media/quickstart-register-app/portal-03-app-reg-02.png" alt-text="Captura de pantalla de Azure Portal en un explorador web que muestra el panel de información general del registro de una aplicación.":::

## <a name="add-a-redirect-uri"></a>Incorporación de un URI de redirección

Un _URI de redirección_ es la ubicación a la que la plataforma de identidad de Microsoft redirige el cliente de un usuario y envía tokens de seguridad tras la autenticación.

En una aplicación web de producción, por ejemplo, el URI de redirección suele ser un punto de conexión público en el que se ejecuta la aplicación, por ejemplo, `https://contoso.com/auth-response`. Durante la fase de desarrollo, también es habitual incorporar el punto de conexión en el que se ejecuta la aplicación localmente, como `https://127.0.0.1/auth-response` o `http://localhost/auth-response`.

Para agregar y modificar los URI de redirección de las aplicaciones registradas, especifique los parámetros en [Configuraciones de plataforma](#configure-platform-settings).

### <a name="configure-platform-settings"></a>Configuración de los valores de plataforma

Los valores de cada tipo de aplicación, incluidos los URI de redirección, se especifican en **Configuraciones de plataforma** en Azure Portal. Algunas plataformas, como las de **aplicaciones web** y **aplicaciones de página única**, requieren que se especifique manualmente un URI de redirección. Para otras plataformas, como las de aplicaciones móviles y de escritorio, es posible elegir entre los URI de redirección que se generan automáticamente al configurar las demás opciones.

Para configurar los valores de la aplicación según la plataforma o el dispositivo de destino:

1. En Azure Portal, seleccione la aplicación en **Registros de aplicaciones**.
1. En **Administrar**, seleccione **Autenticación**.
1. En **Configuraciones de plataforma**, seleccione **Agregar una plataforma**.
1. En **Configurar plataformas**, seleccione el icono del tipo de aplicación (plataforma) para configurar los valores.

   :::image type="content" source="media/quickstart-register-app/portal-04-app-reg-03-platform-config.png" alt-text="Captura de pantalla del panel de configuración de la plataforma en Azure Portal." border="false":::

   | Plataforma                            | Parámetros de configuración                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
   | ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | **Web**                             | Escriba un **URI de redirección** para la aplicación. Este URI es la ubicación a la que la plataforma de identidad de Microsoft redirige el cliente de un usuario y envía tokens de seguridad tras la autenticación.<br/><br/>Seleccione esta plataforma para las aplicaciones web estándar que se ejecuten en un servidor.                                                                                                                                                                                                                                                                   |
   | **Aplicación de página única**         | Escriba un **URI de redirección** para la aplicación. Este URI es la ubicación a la que la plataforma de identidad de Microsoft redirige el cliente de un usuario y envía tokens de seguridad tras la autenticación.<br/><br/>Seleccione esta plataforma si va a desarrollar una aplicación web del lado del cliente con JavaScript, o bien, con un marco como Angular, Vue.js, React.js o Blazor WebAssembly.                                                                                                                                                                                    |
   | **iOS/macOS**                     | Escriba el **ID de agrupación** de la aplicación. Lo encontrará en **Configuración de la compilación** o, en Xcode, en _Info.plist_.<br/><br/>Al especificar un **ID de agrupación**, se genera un URI de redirección.                                                                                                                                                                                                                                                                                                                                                              |
   | **Android**                         | Escriba el valor en **Nombre del paquete** de la aplicación. Lo encontrará en el archivo _AndroidManifest.xml_. Además, genere y especifique el valor de **Hash de firma**.<br/><br/>Al especificar estos valores, se genera un URI de redirección.                                                                                                                                                                                                                                                                                                                            |
   | **Aplicaciones móviles y de escritorio** | Seleccione uno de los valores de **URI de redirección sugeridos**. O bien, especifique un valor para **URI de redireccionamiento personalizado**.<br/><br/>En el caso de las aplicaciones de escritorio que usan el explorador insertado, se recomienda<br/>`https://login.microsoftonline.com/common/oauth2/nativeclient`<br/><br/>En el caso de las aplicaciones de escritorio que usan el explorador del sistema, se recomienda<br/>`http://localhost`<br/><br/>Seleccionar esta plataforma para las aplicaciones móviles que no utilicen la biblioteca de autenticación de Microsoft (MSAL) más reciente o que no usen un agente. Seleccione también esta plataforma para las aplicaciones de escritorio. |

1. Seleccione **Configurar** para completar la configuración de la plataforma.

### <a name="redirect-uri-restrictions"></a>Restricciones aplicables a los URI de redirección

Existen algunas restricciones con respecto al formato de los URI de redirección que se agregan al registro de una aplicación. Para más información sobre estas restricciones, consulte [Restricciones y limitaciones del identificador URI de redirección (dirección URL de respuesta)](reply-url.md).

## <a name="add-credentials"></a>Adición de credenciales

Las credenciales se usan en las [aplicaciones cliente confidenciales](msal-client-applications.md) que acceden a una API web. Ejemplos de aplicaciones cliente confidenciales son, entre otras, las [aplicaciones web](scenario-web-app-call-api-overview.md), las [API web](scenario-protected-web-api-overview.md) o las [aplicaciones demonio y de tipo servicio](scenario-daemon-overview.md). Las credenciales permiten que la aplicación se autentique a sí misma, por lo que no se requiere la interacción del usuario en tiempo de ejecución.

Puede agregar certificados y secretos de cliente (una cadena) como credenciales al registro de la aplicación cliente confidencial.

:::image type="content" source="media/quickstart-register-app/portal-05-app-reg-04-credentials.png" alt-text="Captura de pantalla de Azure Portal que muestra el panel de certificados y secretos durante el registro de una aplicación.":::

### <a name="add-a-certificate"></a>Incorporación de un certificado

Un certificado, que a veces se denomina _clave pública_, es el tipo de credenciales recomendado. Proporciona más garantía que un secreto de cliente. Para más información sobre el uso de un certificado como método de autenticación en la aplicación, consulte [Credenciales de certificado para la autenticación de aplicaciones en la plataforma de identidad de Microsoft](active-directory-certificate-credentials.md).

1. En Azure Portal, seleccione la aplicación en **Registros de aplicaciones**.
1. Seleccione **Certificados y secretos** > **Cargar certificado**.
1. Seleccione el archivo que quiere cargar. Debe ser uno de los siguientes tipos de archivo: _.cer_, _.pem_, _.crt_.
1. Seleccione **Agregar**.

### <a name="add-a-client-secret"></a>Agregar un secreto de cliente

El secreto de cliente también se conoce como _contraseña de la aplicación_. Es un valor de cadena que la aplicación puede usar en lugar de un certificado para la identidad. El secreto de cliente es el más fácil de usar de los dos tipos de credenciales. A menudo se usa durante el desarrollo, pero se considera menos seguro que un certificado. Use certificados cuando las aplicaciones se ejecuten en el entorno de producción.

Para más información sobre las recomendaciones de seguridad de aplicaciones, consulte [Procedimientos recomendados y recomendaciones de la plataforma de identidad de Microsoft](identity-platform-integration-checklist.md#security).

1. En Azure Portal, seleccione la aplicación en **Registros de aplicaciones**.
1. Seleccione **Certificados y secretos** > **Nuevo secreto de cliente**.
1. Agregue una descripción para el secreto de cliente.
1. Seleccione una duración.
1. Seleccione **Agregar**.
1. _Registre el valor del secreto_ para su uso en el código de la aplicación cliente. Este valor secreto no se _volverá a mostrar_ una vez que abandone esta página.

Por motivos de seguridad, Microsoft limita la creación de secretos de cliente para más de 24 meses y recomienda establecerlo en un valor inferior a 12 meses.

## <a name="next-steps"></a>Pasos siguientes

Las aplicaciones cliente suelen necesitar acceso a los recursos de una API web. Puede proteger la aplicación cliente mediante la plataforma de identidad de Microsoft. También puede usar la plataforma para autorizar el acceso con permisos con ámbito a la API web.

Pase al siguiente inicio rápido de la serie con el fin de crear otro registro de aplicación para la API web y exponer sus ámbitos.

> [!div class="nextstepaction"]
> [Configuración de una aplicación para exponer una API web](quickstart-configure-app-expose-web-apis.md)
