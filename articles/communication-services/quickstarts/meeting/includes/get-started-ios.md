---
title: 'Inicio rápido: Incorporación de la unión a una reunión de Microsoft Teams a una aplicación iOS mediante Azure Communication Services'
description: En este inicio rápido, aprenderá a usar la biblioteca de integración de Teams de Azure Communication Services para iOS.
author: palatter
ms.author: palatter
ms.date: 01/25/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 671c86790a3c90f948edb574bc015c0f41c5fbdf
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109858031"
---
En este inicio rápido, aprenderá cómo unirse a una reunión de Microsoft Teams mediante la biblioteca insertada de Teams de Azure Communication Services para iOS.

## <a name="prerequisites"></a>Requisitos previos

Para realizar este inicio rápido, debe cumplir los siguientes requisitos previos:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.
- Tener instalado [CocoaPods](https://cocoapods.org/) para el entorno de compilación.

## <a name="setting-up"></a>Instalación

### <a name="creating-the-xcode-project"></a>Creación del proyecto de Xcode

En Xcode, cree un proyecto de iOS y seleccione la plantilla **App** (Aplicación). Usaremos guiones gráficos de UIKit. Durante este inicio rápido, no se van a crear pruebas. No dude en desactivar **Incluir pruebas**.

:::image type="content" source="../media/ios/xcode-new-project-template-select.png" alt-text="Captura de pantalla que muestra la selección de la plantilla New Project (Nuevo proyecto) en Xcode.":::

Dé un nombre al proyecto `TeamsEmbedGettingStarted`.

:::image type="content" source="../media/ios/xcode-new-project-details.png" alt-text="Captura de pantalla que muestra los detalles del nuevo proyecto en Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Instalación del paquete y las dependencias con CocoaPods

1. Cree un Podfile para la aplicación:

```
platform :ios, '12.0'
use_frameworks!

target 'TeamsEmbedGettingStarted' do
    pod 'AzureCommunication', '~> 1.0.0-beta.11'
end

azure_libs = [
'AzureCommunication',
'AzureCore']

post_install do |installer|
    installer.pods_project.targets.each do |target|
    if azure_libs.include?(target.name)
        puts "Adding BUILD_LIBRARY_FOR_DISTRIBUTION to #{target.name}"
        target.build_configurations.each do |config|
        xcconfig_path = config.base_configuration_reference.real_path
        File.open(xcconfig_path, "a") {|file| file.puts "BUILD_LIBRARY_FOR_DISTRIBUTION = YES"}
        end
    end
    end
end
```

2. Ejecute `pod install`.
3. Abra el archivo `.xcworkspace` generado con Xcode.

### <a name="request-access-to-the-microphone-camera-etc"></a>Solicite acceso al micrófono, la cámara, etc.

Para acceder al hardware del dispositivo, actualice la lista de propiedades de información de la aplicación. Establezca el valor asociado al elemento `string` que se incluirá en el cuadro de diálogo que el sistema usa para solicitar al usuario acceso.

Haga clic con el botón derecho en la entrada `Info.plist` del árbol del proyecto y seleccione **Abrir como** > **Código fuente**. Agregue las líneas siguientes a la sección `<dict>` de nivel superior y guarde el archivo.

```xml
<key>NSCameraUsageDescription</key>
<string></string>
<key>NSContactsUsageDescription</key>
<string></string>
<key>NSMicrophoneUsageDescription</key>
<string></string>
```

### <a name="add-the-teams-embed-framework"></a>Adición del marco de integración de Teams

1. Descargue el paquete `MicrosoftTeamsSDK` para iOS.
2. Cree la carpeta `Frameworks` en la raíz del proyecto. Por ejemplo, `\TeamsEmbedGettingStarted\Frameworks\`
3. Copie los marcos `TeamsAppSDK.framework` y `MeetingUIClient.framework` descargados y otros marcos que se proporcionan en la agrupación de versiones en esta carpeta.
4. Agregue los marcos al destino del proyecto en la pestaña General. Use `Add Other` -> `Add Files...` para desplazarse a los archivos del marco y agregarlos.

:::image type="content" source="../media/ios/xcode-add-frameworks.png" alt-text="Captura de pantalla que muestra los marcos agregados en Xcode.":::

5. Si todavía no está, agregue `$(PROJECT_DIR)/Frameworks` a `Framework Search Paths` en la pestaña de configuración de la compilación de destino del proyecto. Para encontrar la configuración, debe cambiar el filtro de `basic` a `all`; también puede usar la barra de búsqueda de la derecha.

:::image type="content" source="../media/ios/xcode-add-framework-search-path.png" alt-text="Captura de pantalla que muestra la ruta de búsqueda del marco en Xcode.":::

### <a name="turn-off-bitcode"></a>Desactivación de Bitcode

Establezca la opción `Enable Bitcode` en `No` en la opción `Build Settings` del proyecto. Para encontrar la configuración, debe cambiar el filtro de `Basic` a `All`; también puede usar la barra de búsqueda de la derecha.

:::image type="content" source="../media/ios/xcode-bitcode-option.png" alt-text="Captura de pantalla que muestra la opción Bitcode en Xcode.":::


### <a name="turn-on-voice-over-ip-background-mode"></a>Active el modo en segundo plano de voz sobre IP.

Seleccione el destino de la aplicación y haga clic en la pestaña Capabilities (Funcionalidades).

Para activar `Background Modes`, haga clic en `+ Capabilities` en la parte superior y seleccione `Background Modes`.

Active la casilla `Voice over IP`.

:::image type="content" source="../media/ios/xcode-background-voip.png" alt-text="Captura de pantalla que muestra VOIP habilitado en Xcode.":::

### <a name="add-a-window-reference-to-appdelegate"></a>Adición de una referencia de ventana a AppDelegate

Abra el archivo **AppDelegate.swift** del proyecto y agregue una referencia para "window" (ventana).

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
```

### <a name="add-a-button-to-the-viewcontroller"></a>Adición de un botón a ViewController

Cree un botón en la devolución de llamada `viewDidLoad` en **ViewController.swift**.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    let button = UIButton(frame: CGRect(x: 100, y: 100, width: 200, height: 50))
    button.backgroundColor = .black
    button.setTitle("Join Meeting", for: .normal)
    button.addTarget(self, action: #selector(joinMeetingTapped), for: .touchUpInside)
    
    button.translatesAutoresizingMaskIntoConstraints = false
    self.view.addSubview(button)
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
```

Cree una salida para el botón en **ViewController.swift**.

```swift
@IBAction func joinMeetingTapped(_ sender: UIButton) {

}
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Abra el archivo **ViewController.swift** del proyecto y agregue una declaración `import` en la parte superior del archivo para importar los elementos `AzureCommunication library` y `MeetingUIClient`. 

```swift
import UIKit
import AzureCommunication
import MeetingUIClient
```

Reemplace la implementación de la clase `ViewController` por un sencillo botón para permitir al usuario unirse a una reunión. En este inicio rápido, asociaremos la lógica de negocios al botón.

```swift
class ViewController: UIViewController {

    private var meetingUIClient: MeetingUIClient?
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Initialize meetingClient
    }

    @IBAction func joinMeetingTapped(_ sender: UIButton) {
        joinMeeting()
    }
    
    private func joinMeeting() {
        // Add join meeting logic
    }
}
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca de integración de Teams de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| MeetingUIClient | MeetingUIClient es el punto de entrada principal a la biblioteca de integración de Teams. |
| MeetingUIClientMeetingJoinOptions | MeetingUIClientMeetingJoinOptions se usa para las opciones configurables, como el nombre para mostrar. |
| MeetingUIClientGroupCallJoinOptions | MeetingUIClientMeetingJoinOptions se usa para las opciones configurables, como el nombre para mostrar. |
| MeetingUIClientTeamsMeetingLinkLocator | MeetingUIClientTeamsMeetingLinkLocator se usa para establecer la dirección URL de la reunión para unirse a una reunión. |
| MeetingUIClientGroupCallLocator | MeetingUIClientGroupCallLocator se usa para establecer el identificador de grupo al que se va a unir. |
| MeetingUIClientCallState | MeetingUIClientCallState se utiliza para notificar los cambios en el estado de la llamada. Estas son las opciones: `connecting`, `waitingInLobby`, `connected` y `ended`. |
| MeetingUIClientDelegate | MeetingUIClientDelegate se usa para recibir eventos, como cambios en el estado de la llamada. |
| MeetingUIClientIdentityProviderDelegate | MeetingUIClientIdentityProviderDelegate se usa para asignar los detalles del usuario a los usuarios de una reunión. |
| MeetingUIClientUserEventDelegate | MeetingUIClientUserEventDelegate proporciona información sobre las acciones del usuario en la interfaz de usuario. |

## <a name="create-and-authenticate-the-client"></a>Creación y autenticación del cliente

Inicialice una instancia de `MeetingUIClient` con un token de acceso de usuario, lo que permite la unión a reuniones. Agregue el código siguiente a la devolución de llamada `viewDidLoad` en **ViewController.swift**:

```swift
do {
    let communicationTokenRefreshOptions = CommunicationTokenRefreshOptions(initialToken: "<USER_ACCESS_TOKEN>", refreshProactively: true, tokenRefresher: fetchTokenAsync(completionHandler:))
    let credential = try CommunicationTokenCredential(withOptions: communicationTokenRefreshOptions)
    meetingUIClient = MeetingUIClient(with: credential)
}
catch {
    print("Failed to create communication token credential")
}
```

Reemplace `<USER_ACCESS_TOKEN>` por un token de acceso de usuario válido para el recurso. Consulte la documentación relativa al [token de acceso de usuario](../../access-tokens.md) si aún no tiene ningún token disponible.

### <a name="setup-token-refreshing"></a>Actualización del token de instalación

Cree un método `fetchTokenAsync`; Después, agregue la lógica `fetchToken` para obtener el token de usuario.

```swift
private func fetchTokenAsync(completionHandler: @escaping TokenRefreshHandler) {
    func getTokenFromServer(completionHandler: @escaping (String) -> Void) {
        completionHandler("<USER_ACCESS_TOKEN>")
    }
    getTokenFromServer { newToken in
        completionHandler(newToken, nil)
    }
}
```

Reemplace `<USER_ACCESS_TOKEN>` por un token de acceso de usuario válido para el recurso.

## <a name="join-a-meeting"></a>Unirse a una reunión

El método `join` se establece como la acción que se llevará a cabo cuando se pulse el botón *Unirse a la reunión*. Actualice la implementación para unirse a una reunión con `MeetingUIClient`:

```swift
private func joinMeeting() {
    let meetingJoinOptions = MeetingUIClientMeetingJoinOptions(displayName: "John Smith", enablePhotoSharing: true, enableNamePlateOptionsClickDelegate: true)
    let meetingLocator = MeetingUIClientTeamsMeetingLinkLocator(meetingLink: "<MEETING_URL>")
    meetingUIClient?.join(meetingLocator: meetingLocator, joinCallOptions: meetingJoinOptions, completionHandler: { (error: Error?) in
        if (error != nil) {
            print("Join meeting failed: \(error!)")
        }
    })
}
```

Reemplace `<MEETING URL>` por un vínculo a la reunión de Microsoft Teams.

### <a name="get-a-microsoft-teams-meeting-link"></a>Obtención de un vínculo de reunión de Microsoft Teams

Se puede recuperar un vínculo a la reunión de Teams mediante instancias de Graph API. Esto se detalla en la [documentación de Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
El SDK de llamadas de Communication Services acepta un vínculo completo a la reunión de Microsoft Teams. Este vínculo se devuelve como parte del recurso `onlineMeeting`, al que se accede bajo la propiedad [`joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). También puede obtener la información necesaria de la reunión de la dirección URL **Unirse a la reunión** de la propia invitación a la reunión de Teams.

## <a name="run-the-code"></a>Ejecución del código

Para compilar y ejecutar la aplicación en el simulador de iOS, seleccione **Producto** > **Ejecutar** o use el método abreviado de teclado (&#8984;-R).

:::image type="content" source="../media/ios/quick-start-join-meeting.png" alt-text="Aspecto final de la aplicación de inicio rápido":::

:::image type="content" source="../media/ios/quick-start-meeting-joined.png" alt-text="Apariencia final cuando se ha unido la reunión":::

> [!NOTE]
> La primera vez que se una a una reunión, el sistema le solicitará acceso al micrófono. En una aplicación de producción, debe usar la API `AVAudioSession` para [comprobar el estado del permiso](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) y actualizar correctamente el comportamiento de la aplicación cuando no se conceda el permiso.

## <a name="add-localization-support-based-on-your-app"></a>Adición de compatibilidad con la localización en función de la aplicación

El SDK de Microsoft Teams admite más de 100 cadenas y recursos. El paquete del marco contiene los idiomas base e inglés. El resto se incluyen en el archivo `Localizations.zip` que viene con el paquete.

#### <a name="add-localizations-to-the-sdk-based-on-what-your-app-supports"></a>Agregue localizaciones al SDK en función de lo que admita la aplicación.

1. Determine el tipo de localizaciones que admite la aplicación. Para ello, vaya al proyecto de Xcode de la aplicación, seleccione la pestaña de información y consulte la lista de localizaciones.
2. Descomprima el archivo Localizations.zip incluido con el paquete.
3. Copie las carpetas de localización de la carpeta descomprimida en función de lo que admita la aplicación en la raíz de TeamsAppSDK.framework.


## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/teams-embed-ios-getting-started).
