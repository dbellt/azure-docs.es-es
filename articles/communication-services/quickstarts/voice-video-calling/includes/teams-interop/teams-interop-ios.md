---
title: 'Inicio rápido: unión a una reunión de Teams desde una aplicación de iOS'
description: En este tutorial aprenderá a unirse a una reunión de Teams mediante Calling SDK de Azure Communication Services para iOS.
author: chpalm
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: fb22ce8e649fe70569f75a77d1a952ed1addd5dd
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111993305"
---
En este inicio rápido aprenderá a unirse a una reunión de Teams mediante Calling SDK de Azure Communication Services para iOS.

## <a name="prerequisites"></a>Requisitos previos

- Una [aplicación de iOS para llamadas telefónicas de Communication Services](../../getting-started-with-calling.md).
- Una [implementación de Teams](/deployoffice/teams-install).

Usaremos beta.12 del SDK de AzureCommunicationCalling para este inicio rápido, por lo que es necesario actualizar el podfile y volver a instalar los pods. 

Reemplace el podfile por el siguiente código para Podfile y guárdelo (asegúrese de que "target" coincide con el nombre del proyecto):

   ```
   platform :ios, '13.0'
   use_frameworks!

   target 'AzureCommunicationCallingSample' do
     pod 'AzureCommunicationCalling', '1.0.0-beta.12'
   end
   ```
Elimine la carpeta Pods, Podfile.lock y el archivo `.xcworkspace.`.

Ejecute `pod install` y abra `.xcworkspace` con Xcode.

## <a name="add-the-teams-ui-controls-and-enable-the-teams-ui-controls"></a>Incorporación y habilitación de los controles de la interfaz de usuario de Teams

Reemplace el código de ContentView.swift por el siguiente fragmento de código. El cuadro de texto se utilizará para especificar el contexto de reunión de Teams y el botón se usará para la unión a la reunión especificada:

```swift

import SwiftUI
import AzureCommunicationCalling
import AVFoundation

struct ContentView: View {
    @State var meetingLink: String = ""
    @State var callStatus: String = ""
    @State var message: String = ""
    @State var recordingStatus: String = ""
    @State var callClient: CallClient?
    @State var callAgent: CallAgent?
    @State var call: Call?
    @State var callObserver: CallObserver?

    var body: some View {
        NavigationView {
            Form {
                Section {
                    TextField("Teams meeting link", text: $meetingLink)
                    Button(action: joinTeamsMeeting) {
                        Text("Join Teams Meeting")
                    }.disabled(callAgent == nil)
                    Button(action: leaveMeeting) {
                        Text("Leave Meeting")
                    }.disabled(call == nil)
                    Text(callStatus)
                    Text(message)
                    Text(recordingStatus)
                }
            }
            .navigationBarTitle("Calling Quickstart")
        }.onAppear {
            // Initialize call agent
            var userCredential: CommunicationTokenCredential?
            do {
                userCredential = try CommunicationTokenCredential(token: "<USER ACCESS TOKEN>")
            } catch {
                print("ERROR: It was not possible to create user credential.")
                self.message = "Please enter your token in source code"
                return
            }

            self.callClient = CallClient()

            // Creates the call agent
            self.callClient?.createCallAgent(userCredential: userCredential!) { (agent, error) in
                if error != nil {
                    self.message = "Failed to create CallAgent."
                    return
                } else {
                    self.callAgent = agent
                    self.message = "Call agent successfully created."
                }
            }
        }
    }

    func joinTeamsMeeting() {
        // Ask permissions
        AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
            if granted {
                let joinCallOptions = JoinCallOptions()
                let teamsMeetingLinkLocator = TeamsMeetingLinkLocator(meetingLink: self.meetingLink)
                self.callAgent?.join(with: teamsMeetingLinkLocator, joinCallOptions: joinCallOptions) {(call, error) in
                    if (error == nil) {
                        self.call = call
                        self.callObserver = CallObserver(self)
                        self.call!.delegate = self.callObserver
                        self.message = "Teams meeting joined successfully"
                    } else {
                        print("Failed to get call object")
                        return
                    }
                }
            }
        }
    }


    func leaveMeeting() {
        if let call = call {
            call.hangUp(options: nil, completionHandler: { (error) in
                if error == nil {
                    self.message = "Leaving Teams meeting was successful"
                } else {
                    self.message = "Leaving Teams meeting failed"
                }
            })
        } else {
            self.message = "No active call to hangup"
        }
    }
}

class CallObserver : NSObject, CallDelegate {
    private var owner:ContentView
    init(_ view:ContentView) {
        owner = view
    }

    public func call(_ call: Call, didChangeState args: PropertyChangedEventArgs) {
        owner.callStatus = CallObserver.callStateToString(state: call.state)
        if call.state == .disconnected {
            owner.call = nil
            owner.message = "Left Meeting"
        } else if call.state == .inLobby {
            owner.message = "Waiting in lobby !!"
        } else if call.state == .connected {
            owner.message = "Joined Meeting !!"
        }
    }
    
    public func call(_ call: Call, didChangeRecordingState args: PropertyChangedEventArgs) {
        if (call.isRecordingActive == true) {
            owner.recordingStatus = "This call is being recorded"
        }
        else {
            owner.recordingStatus = ""
        }
    }


    private static func callStateToString(state: CallState) -> String {
        switch state {
        case .connected: return "Connected"
        case .connecting: return "Connecting"
        case .disconnected: return "Disconnected"
        case .disconnecting: return "Disconnecting"
        case .earlyMedia: return "EarlyMedia"
        case .none: return "None"
        case .ringing: return "Ringing"
        case .inLobby: return "InLobby"
        default: return "Unknown"
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```

## <a name="get-the-teams-meeting-link"></a>Obtención del vínculo de la reunión de Teams

El vínculo de la reunión de Teams se puede recuperar mediante las Graph API. Esto se detalla en la [documentación de Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
El SDK de llamada de Communication Services acepta un vínculo a toda la reunión de Teams. Este vínculo se devuelve como parte del recurso `onlineMeeting`, al que se puede acceder bajo la [propiedad `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). También puede obtener la información de la reunión necesaria en la dirección URL **Join Meeting** (Unirse a la reunión) de la propia invitación a la reunión en Teams.

## <a name="launch-the-app-and-join-teams-meeting"></a>Inicio de la aplicación y unión a una reunión de Teams

Para compilar y ejecutar la aplicación en el simulador de iOS, seleccione **Producto** > **Ejecutar** o use el método abreviado de teclado (&#8984;-R).

:::image type="content" source="../../media/ios/acs-join-teams-meeting-quickstart.png" alt-text="Captura de pantalla que muestra la aplicación completada.":::

Inserte el contexto de Teams en el cuadro de texto y presione *Join Teams Meeting* (Unirse a la reunión de Teams) para unirse a la reunión de Teams desde dentro de la aplicación de Communication Services.
