---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: ddematheu2
manager: chpalm
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 9/1/2020
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: 2a573a4df8abcdc2b32204faadbaa0df8c3dac15
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108313576"
---
El **ejemplo de elementos principales de llamada de grupo para Android** de Azure Communication Services muestra cómo usar el SDK de llamadas de Communication Services para Android para crear una experiencia de llamada de grupo que incluya voz y vídeo. En este inicio rápido de ejemplo, aprenderá a configurar y ejecutar el ejemplo. Se proporciona información general del ejemplo con fines de contextualización.

## <a name="download-code"></a>Descarga de código

Busque el proyecto de este ejemplo en [GitHub](https://github.com/Azure-Samples/communication-services-android-calling-hero). Se puede encontrar una versión del ejemplo con la [interoperabilidad de Teams](../../concepts/teams-interop.md) en una [rama](https://github.com/Azure-Samples/communication-services-android-calling-hero/tree/feature/teams_interop) independiente.

## <a name="overview"></a>Información general

El ejemplo consiste en una aplicación Android nativa que usa los SDK de Azure Communication Services para Android para crear una experiencia de llamada que incluya llamadas de voz y vídeo. La aplicación usa un componente de servidor para aprovisionar los tokens de acceso que, posteriormente, se usan para inicializar el SDK de Azure Communication Services. Para configurar este componente de servidor, siga el tutorial [Creación de un servicio de autenticación de confianza mediante Azure Functions](../../tutorials/trusted-service-tutorial.md).

El ejemplo tendrá una apariencia similar a la siguiente:

:::image type="content" source="../media/calling/landing-page-android.png" alt-text="Captura de pantalla que muestra la página de aterrizaje de la aplicación de ejemplo.":::

Al presionar el botón "Iniciar nueva llamada", la aplicación Android crea una nueva llamada y se une a ella. La aplicación también le permite unirse a una llamada existente de Azure Communication Services mediante el identificador de esa llamada.

Después de unirse a una llamada, se le solicitará que conceda permiso a la aplicación para acceder a la cámara y el micrófono. También se le pedirá que facilite un nombre para mostrar.

:::image type="content" source="../media/calling/pre-call-android.png" alt-text="Captura de pantalla que muestra la pantalla anterior a la llamada en la aplicación de ejemplo.":::

Una vez que configure el nombre para mostrar y los dispositivos, podrá unirse a la llamada. Verá el panel de llamada principal en el que reside la experiencia de llamada.

:::image type="content" source="../media/calling/main-app-android.png" alt-text="Captura de pantalla que muestra la pantalla principal de la aplicación de ejemplo.":::

Componentes de la pantalla principal de llamada:

- **Galería multimedia**: la fase principal en la que se muestran los participantes. Si un participante tiene habilitada la cámara, aquí se muestra su fuente de vídeo. Cada participante tiene un icono individual con su nombre para mostrar y la secuencia de vídeo (si la hay). La galería admite varios participantes y se actualiza al agregarlos o al quitarlos de la llamada.
- **Barra de acciones**: Aquí es donde se encuentran los principales controles de llamada. Estos controles permiten activar o desactivar el vídeo y el micrófono, compartir la pantalla y abandonar la llamada.

A continuación encontrará más información sobre los requisitos previos y los pasos para configurar el ejemplo.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio) en ejecución en el equipo.
- Un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../quickstarts/create-communication-resource.md).
- Una función de Azure que ejecuta el [punto de conexión de autenticación](../../tutorials/trusted-service-tutorial.md) para capturar tokens de acceso.

## <a name="running-sample-locally"></a>Ejecución local del ejemplo

El ejemplo de llamada de grupo se puede ejecutar localmente mediante Android Studio. Los desarrolladores pueden usar su dispositivo físico o un emulador para probar la aplicación.

### <a name="before-running-the-sample-for-the-first-time"></a>Antes de ejecutar el ejemplo por primera vez

1. Abra Android Studio y seleccione `Open an Existing Project`.
2. Abra la carpeta `AzureCalling` dentro de la versión descargada para el ejemplo.
3. Expanda las aplicaciones o recursos para actualizar `appSettings.properties`. Establezca el valor de la clave `communicationTokenFetchUrl` para que sea la dirección URL del punto de conexión de autenticación configurado como requisito previo.

### <a name="run-sample"></a>Ejecución del ejemplo

Compile y ejecute el ejemplo en Android Studio.

## <a name="optional-securing-an-authentication-endpoint"></a>(Opcional) Protección de un punto de conexión de autenticación

Con fines de demostración, este ejemplo usa un punto de conexión de acceso público de forma predeterminada para capturar un token de Azure Communication Services. En escenarios de producción, se recomienda usar un punto de conexión propio protegido para aprovisionar los tokens.

Mediante configuración adicional, este ejemplo permite conectarse a un punto de conexión protegido de **Azure Active Directory** (Azure AD) de manera que se solicite el inicio de sesión de usuario para que la aplicación capture un token de Azure Communication Services. Consulte los siguientes pasos:

1. Habilite la autenticación de Azure Active Directory en la aplicación.  
   - [Registro de la aplicación en Azure Active Directory (con la configuración de la plataforma Android)](../../../active-directory/develop/tutorial-v2-android.md) 
    - [Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](../../../app-service/configure-authentication-provider-aad.md)

2. Vaya a la página de información general de la aplicación registrada en Registros de aplicaciones, en Azure Active Directory. Anote los valores de `Package name`, `Signature hash`, `MSAL Configutaion`.

:::image type="content" source="../media/calling/aad-overview-android.png" alt-text="Configuración de Azure Active Directory en Azure Portal.":::

3. Edite `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` y establezca `isAADAuthEnabled` para habilitar Azure Active Directory.
4. Edite `AndroidManifest.xml` y establezca `android:path` en el hash de firma del almacén de claves. (Opcional. El valor actual usa el hash del almacén de claves debug.keystore incluido. Si se usa un almacén de claves diferente, esto deberá actualizarse).
   ```
   <activity android:name="com.microsoft.identity.client.BrowserTabActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:host="com.azure.samples.communication.calling"
                    android:path="/Signature hash" <!-- do not remove /. The current hash in AndroidManifest.xml is for debug.keystore. -->
                    android:scheme="msauth" />
            </intent-filter>
        </activity>
   ```
5. Copie la configuración de MSAL para Android desde Azure Portal y péguela en `AzureCalling/app/src/main/res/raw/auth_config_single_account.json`. Incluya "account_mode": "SINGLE".
   ```
      {
         "client_id": "",
         "authorization_user_agent": "DEFAULT",
         "redirect_uri": "",
         "account_mode" : "SINGLE",
         "authorities": [
            {
               "type": "AAD",
               "audience": {
               "type": "AzureADMyOrg",
               "tenant_id": ""
               }
            }
         ]
      }
   ```

6. Edite `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` y establezca el valor de la clave `communicationTokenFetchUrl` para que sea la dirección URL del punto de conexión de autenticación.
7. Edite `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` y establezca el valor de la clave `aadScopes` desde los ámbitos `Expose an API` de `Azure Active Directory`.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
>[Descargue el ejemplo de GitHub](https://github.com/Azure-Samples/communication-services-android-calling-hero).

Para más información, consulte los siguientes artículos.

- Familiarícese con el [uso del SDK de llamadas](../../quickstarts/voice-video-calling/calling-client-samples.md).
- Más información sobre [cómo funciona la llamada](../../concepts/voice-video-calling/about-call-types.md)

### <a name="additional-reading"></a>Lecturas adicionales

- [GitHub de Azure Communication](https://github.com/Azure/communication): encuentre más ejemplos e información en la página oficial de GitHub.
- [Ejemplos](./../overview.md): encuentre más ejemplos en nuestra página de información general de ejemplos.
- [Características de llamada de Azure Communication Services](../../concepts/voice-video-calling/calling-sdk-features.md): para más información sobre el SDK de llamada para Android, consulte [SDK de llamadas de Azure Communication Services](https://search.maven.org/artifact/com.azure.android/azure-communication-calling).