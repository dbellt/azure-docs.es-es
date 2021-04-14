---
title: 'Azure Communication Services: problemas conocidos'
description: Obtenga información acerca de Azure Communication Services.
author: rinarish
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: troubleshooting
ms.service: azure-communication-services
ms.openlocfilehash: b9ed71a8fc9346ecd454eba98dcbb3b13186eba2
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106276049"
---
# <a name="known-issues-azure-communication-services-calling-sdks"></a>Problemas conocidos: instancias de Calling SDK de Azure Communication Services
En este artículo se proporciona información sobre las limitaciones y los problemas conocidos relativos a las instancias de Calling SDK de Azure Communication Services.

> [!IMPORTANT]
> Hay varios factores que pueden afectar a la calidad de la experiencia de llamada. Consulte la documentación sobre los **[requisitos de red](https://docs.microsoft.com/azure/communication-services/concepts/voice-video-calling/network-requirements)** para más información sobre los procedimientos recomendados de configuración y prueba de la red de Communication Services.


## <a name="javascript-sdk"></a>SDK de JavaScript

En esta sección se proporciona información sobre los problemas conocidos asociados a los SDK de llamada de voz y vídeo JavaScript de Azure Communication Services.

### <a name="refreshing-a-page-doesnt-immediately-remove-the-user-from-their-call"></a>La actualización de una página no quita al usuario inmediatamente de su llamada

Si un usuario está en una llamada y decide actualizar la página, el servicio multimedia de Communication Services no quita este usuario inmediatamente de la llamada. Espera a que el usuario vuelva a unirse. Transcurrido el tiempo de espera del servicio multimedia, se quita al usuario de la llamada.

Es mejor crear experiencias de usuario que no requieran que los usuarios finales actualicen la página de la aplicación durante una llamada. Si un usuario actualiza la página, vuelva a usar el mismo identificador de usuario de Communication Services después de regresar a la aplicación.

Desde la perspectiva de otros participantes en la llamada, el usuario permanecerá en la llamada durante un tiempo (1 o 2 minutos). Si el usuario se vuelve a unir con el mismo identificador de usuario de Communication Services, se representará con el mismo objeto existente en la colección `remoteParticipants`.

Si antes de actualizar la página, el usuario estaba enviando vídeo, la colección `videoStreams` conservará la información de la secuencia anterior hasta que se agote el tiempo de espera del servicio y la quite. En este escenario, la aplicación puede decidir mantener las nuevas secuencias agregadas a la colección y representar una con el valor de `id` más alto. 


### <a name="its-not-possible-to-render-multiple-previews-from-multiple-devices-on-web"></a>No es posible representar varias vistas previas de varios dispositivos en Web
Esta es una limitación conocida. Para más información, consulte la [introducción a Calling SDK](https://docs.microsoft.com/azure/communication-services/concepts/voice-video-calling/calling-sdk-features).

### <a name="enumerating-devices-isnt-possible-in-safari-when-the-application-runs-on-ios-or-ipados"></a>No es posible enumerar dispositivos en Safari cuando la aplicación se ejecuta en iOS o iPadOS.

Las aplicaciones no pueden enumerar ni seleccionar dispositivos de micrófono o altavoz (por ejemplo, Bluetooth) en Safari en iOS o iPad. Hay una limitación conocida del sistema operativo.

Si usa Safari en macOS, la aplicación no podrá enumerar ni seleccionar altavoces mediante el Administrador de dispositivos de Communication Services. En este escenario, los dispositivos deben seleccionarse mediante el sistema operativo. Si usa Chrome en macOS, la aplicación sí puede enumerar y seleccionar dispositivos mediante el Administrador de dispositivos de Communication Services.

### <a name="audio-connectivity-is-lost-when-receiving-sms-messages-or-calls-during-an-ongoing-voip-call"></a>Se pierde la conectividad de audio al recibir mensajes SMS o llamadas durante una llamada VoIP en curso
Este problema puede producirse por varias razones:

- Algunos exploradores móviles no mantienen la conectividad mientras están en segundo plano. Como consecuencia, la experiencia de llamada puede verse degradada si un evento que envía la aplicación a segundo plano interrumpe la llamada de VoIP. 
- A veces, una llamada de SMS o RTC captura el sonido de audio y no lo devuelve a la llamada de VoIP. Apple ha corregido este problema en las versiones de iOS 14.4.1 y superiores. 

<br/>Biblioteca cliente: Calling (JavaScript)
<br/>Exploradores: Safari, Chrome
<br/>Sistema operativo: iOS, Android

### <a name="repeatedly-switching-video-devices-may-cause-video-streaming-to-temporarily-stop"></a>El cambio repetido de dispositivos de los dispositivos de vídeo puede provocar la detención temporal del streaming de vídeo

El cambio entre dispositivos de vídeo puede hacer que la secuencia de vídeo se pause mientras se adquiere la secuencia desde el dispositivo seleccionado.

#### <a name="possible-causes"></a>Causas posibles
Cambiar entre dispositivos con frecuencia puede provocar una degradación del rendimiento. Se recomienda a los desarrolladores detener una secuencia del dispositivo antes de iniciar otra.

### <a name="bluetooth-headset-microphone-is-not-detected-therefore-is-not-audible-during-the-call-on-safari-on-ios"></a>El micrófono de los auriculares Bluetooth no se detecta, por lo que no es audible durante la llamada en Safari en iOS
Safari en IOS no admite auriculares Bluetooth. El dispositivo Bluetooth no aparecerá entre las opciones de micrófono disponibles y otros participantes no podrán oírle si intenta usar Bluetooth en Safari.

#### <a name="possible-causes"></a>Causas posibles
Se trata de una limitación conocida de los sistemas operativos macOS, iOS e iPadOS. 

Con Safari en **macOS** e **iOS/iPadOS**, no es posible enumerar ni seleccionar dispositivos de altavoz mediante el Administrado de dispositivos de Communication Services, dado que esta característica no se admite en Safari. En este escenario, la selección del dispositivo debe actualizarse a través del sistema operativo.

### <a name="rotation-of-a-device-can-create-poor-video-quality"></a>La rotación de un dispositivo puede crear una calidad de vídeo deficiente
Los usuarios pueden experimentar una calidad de vídeo degradada cuando se rotan los dispositivos.

<br/>Dispositivos afectados: Google Pixel 5, Google Pixel 3a, Apple iPad 8 y Apple iPad X
<br/>Biblioteca cliente: Calling (JavaScript)
<br/>Exploradores: Safari, Chrome
<br/>Sistema operativo: iOS, Android


### <a name="camera-switching-makes-the-screen-freeze"></a>Al cambiar de cámara, la pantalla se bloquea 
Cuando un usuario de Communication Services se une a una llamada mediante Calling SDK de JavaScript y, luego, presiona el botón de cambio de cámara, la interfaz de usuario puede dejar de responder hasta que se actualice la aplicación o hasta que el usuario envíe el explorador a segundo plano.

<br/>Dispositivos afectados: Google Pixel 4A
<br/>Biblioteca cliente: Calling (JavaScript)
<br/>Exploradores: Chrome
<br/>Sistema operativo: iOS, Android


#### <a name="possible-causes"></a>Causas posibles
Bajo investigación.

### <a name="if-the-video-signal-was-stopped-while-the-call-is-in-connecting-state-the-video-will-not-be-sent-after-the-call-started"></a>Si se ha detenido la señal de vídeo mientras la llamada está en el estado de conexión, el vídeo no se enviará después de iniciar la llamada 
Si los usuarios deciden activar o desactivar rápidamente el vídeo mientras la llamada está en estado `Connecting`, puede producirse un problema con la secuencia adquirida en la llamada. Animamos a los desarrolladores a crear sus aplicaciones de forma que no haga falta activar o desactivar el vídeo mientras la llamada está en estado `Connecting`. Este problema puede hacer que el rendimiento de vídeo se degrade en los siguientes escenarios:

 - Si el usuario comienza con audio y, luego, inicia y detiene el vídeo mientras la llamada está en estado `Connecting`.
 - Si el usuario comienza con audio y, luego, inicia y detiene el vídeo mientras la llamada está en estado `Lobby`.

#### <a name="possible-causes"></a>Causas posibles
Bajo investigación.

### <a name="enumeratingaccessing-devices-for-safari-on-macos-and-ios"></a>Enumeración y acceso de dispositivos para Safari en MacOS e iOS 
Si se concede acceso a los dispositivos después de un tiempo, se restablecen los permisos de los dispositivos. Safari en MacOS y en iOS no conservan los permisos durante mucho tiempo a menos que haya una secuencia adquirida. La manera más sencilla de solucionar este problema es llamar a la API DeviceManager.askDevicePermission() antes de llamar a las API de enumeración de dispositivos del Administrador de dispositivos [DeviceManager.getCameras(), DeviceManager.getSpeakers() y DeviceManager.getMicrophones()]. Si los permisos están allí, el usuario no verá nada; de lo contrario, se le volverán a solicitar.

<br/>Dispositivos afectados: iPhone
<br/>Biblioteca cliente: Calling (JavaScript)
<br/>Exploradores: Safari
<br/>Sistema operativo: iOS

###  <a name="sometimes-it-takes-a-long-time-to-render-remote-participant-videos"></a>A veces se tarda mucho tiempo en presentar los vídeos de participantes remotos
Durante una llamada de grupo en curso, el _usuario A_ envía el vídeo y, luego, el _usuario B_ se une a la llamada. A veces, el usuario B no ve el vídeo del usuario A, o el vídeo del usuario A comienza a representarse después de un retraso largo. Este problema puede deberse a que el entorno de red requiere una configuración adicional. Consulte la documentación sobre los [requisitos de red](https://docs.microsoft.com/azure/communication-services/concepts/voice-video-calling/network-requirements) para obtener instrucciones sobre la configuración de red.
