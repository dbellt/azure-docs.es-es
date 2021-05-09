---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: ddematheu2
manager: chpalm
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: 84bb00b6ef5d0bf68a55b7eba70ccb8eb697a571
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108313559"
---
El **ejemplo de elementos principales de llamada de grupo para iOS** de Azure Communication Services muestra cómo usar Calling SDK de Communication Services para iOS para crear una experiencia de llamada de grupo que incluya voz y vídeo. En este inicio rápido de ejemplo, aprenderá a configurar y ejecutar el ejemplo. Se proporciona información general del ejemplo con fines de contextualización.

## <a name="download-code"></a>Descarga de código

Busque el proyecto de este ejemplo en [GitHub](https://github.com/Azure-Samples/communication-services-ios-calling-hero). Se puede encontrar una versión del ejemplo con la [interoperabilidad de Teams](../../concepts/teams-interop.md) en una [rama](https://github.com/Azure-Samples/communication-services-ios-calling-hero/tree/feature/teams_interop) independiente.

## <a name="overview"></a>Información general

El ejemplo consiste en una aplicación iOS nativa que usa los SDK de Azure Communication Services para iOS para crear una experiencia de llamada que incluya llamadas de voz y vídeo. La aplicación usa un componente de servidor para aprovisionar los tokens de acceso que, posteriormente, se usan para inicializar el SDK de Azure Communication Services. Para configurar este componente de servidor, siga el tutorial [Creación de un servicio de autenticación de confianza mediante Azure Functions](../../tutorials/trusted-service-tutorial.md).

El ejemplo tendrá una apariencia similar a la siguiente:

:::image type="content" source="../media/calling/landing-page-ios.png" alt-text="Captura de pantalla que muestra la página de aterrizaje de la aplicación de ejemplo.":::

Al presionar el botón "Iniciar nueva llamada", la aplicación iOS crea una nueva llamada y se une a ella. La aplicación también le permite unirse a una llamada existente de Azure Communication Services mediante el identificador de esa llamada.

Después de unirse a una llamada, se le solicitará que conceda permiso a la aplicación para acceder a la cámara y el micrófono. También se le pedirá que facilite un nombre para mostrar.

:::image type="content" source="../media/calling/pre-call-ios.png" alt-text="Captura de pantalla que muestra la pantalla anterior a la llamada en la aplicación de ejemplo.":::

Una vez que configure el nombre para mostrar y los dispositivos, podrá unirse a la llamada. Verá el panel de llamada principal en el que reside la experiencia de llamada.

:::image type="content" source="../media/calling/main-app-ios.png" alt-text="Captura de pantalla que muestra la pantalla principal de la aplicación de ejemplo.":::

Componentes de la pantalla principal de llamada:

- **Galería multimedia**: la fase principal en la que se muestran los participantes. Si un participante tiene habilitada la cámara, aquí se muestra su fuente de vídeo. Cada participante tiene un icono individual con su nombre para mostrar y la secuencia de vídeo (si la hay). La galería admite varios participantes y se actualiza al agregarlos o al quitarlos de la llamada.
- **Barra de acciones**: Aquí es donde se encuentran los principales controles de llamada. Estos controles permiten activar o desactivar el vídeo y el micrófono, compartir la pantalla y abandonar la llamada.

A continuación encontrará más información sobre los requisitos previos y los pasos para configurar el ejemplo.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../quickstarts/create-communication-resource.md).
- Una función de Azure que ejecuta el [punto de conexión de autenticación](../../tutorials/trusted-service-tutorial.md) para capturar tokens de acceso.

## <a name="running-sample-locally"></a>Ejecución local del ejemplo

El ejemplo de llamada de grupo se puede ejecutar localmente mediante XCode. Los desarrolladores pueden usar su dispositivo físico o un emulador para probar la aplicación.

### <a name="before-running-the-sample-for-the-first-time"></a>Antes de ejecutar el ejemplo por primera vez

1. Instale las dependencias mediante la ejecución de `pod install`.
2. Abra `AzureCalling.xcworkspace` en XCode.
3. Actualice `AppSettings.plist`. Establezca el valor de la clave de `communicationTokenFetchUrl` para que sea la dirección URL del punto de conexión de autenticación.

### <a name="run-sample"></a>Ejecución del ejemplo

Compile y ejecute el ejemplo en XCode.

## <a name="optional-securing-an-authentication-endpoint"></a>(Opcional) Protección de un punto de conexión de autenticación

Con fines de demostración, en este ejemplo se usa de forma predeterminada un punto de conexión de acceso público para capturar un token de acceso de Azure Communication Services. En escenarios de producción, se recomienda usar un punto de conexión propio protegido para aprovisionar los tokens.

Mediante alguna configuración adicional, este ejemplo permite conectarse a un punto de conexión protegido de **Azure Active Directory** (Azure AD) de manera que se solicite el inicio de sesión de usuario para que la aplicación capture un token de acceso de Azure Communication Services. Consulte los siguientes pasos:

1. Habilite la autenticación de Azure Active Directory en la aplicación.  
   - [Registro de la aplicación en Azure Active Directory (con la configuración de la plataforma iOS/macOS)](../../../active-directory/develop/tutorial-v2-ios.md) 
    - [Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](../../../app-service/configure-authentication-provider-aad.md)
2. Vaya a la página de información general de la aplicación registrada en Registros de aplicaciones, en Azure Active Directory. Anote los valores de `Application (client) ID`, `Directory (tenant) ID`, `Application ID URI`.

:::image type="content" source="../media/calling/aad-overview.png" alt-text="Configuración de Azure Active Directory en Azure Portal.":::

3. Abra `AppSettings.plist` en Xcode y agregue los siguientes valores de clave:
   - `communicationTokenFetchUrl`: la dirección URL para solicitar el token de Azure Communication Services. 
   - `isAADAuthEnabled`: un valor booleano que indica si se requiere o no la autenticación de token de Azure Communication Services.
   - `aadClientId`: el identificador de la aplicación (cliente).
   - `aadTenantId`: el identificador del directorio (inquilino).
   - `aadRedirectURI`: el URI de redirección debe tener el formato `msauth.<app_bundle_id>://auth`.
   - `aadScopes`: una matriz de ámbitos de permisos solicitados a los usuarios con fines de autorización. Agregue `<Application ID URI>/user_impersonation` a la matriz para otorgar acceso al punto de conexión de autenticación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
>[Descargue el ejemplo de GitHub](https://github.com/Azure-Samples/communication-services-ios-calling-hero).

Para más información, consulte los siguientes artículos.

- Familiarícese con el [uso del SDK de llamadas](../../quickstarts/voice-video-calling/calling-client-samples.md).
- Más información sobre [cómo funciona la llamada](../../concepts/voice-video-calling/about-call-types.md)

### <a name="additional-reading"></a>Lecturas adicionales

- [GitHub de Azure Communication](https://github.com/Azure/communication): encuentre más ejemplos e información en la página oficial de GitHub.
- [Ejemplos](./../overview.md): encuentre más ejemplos en nuestra página de información general de ejemplos.
- [Características de llamada de Azure Communication Services](../../concepts/voice-video-calling/calling-sdk-features.md): para más información sobre el SDK de llamada para iOS, consulte [Calling SDK de Azure Communication Services](https://github.com/Azure/Communication/releases/).