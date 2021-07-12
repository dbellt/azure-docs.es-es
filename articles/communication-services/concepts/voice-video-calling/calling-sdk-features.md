---
title: Información general de Calling SDK de Azure Communication Services.
titleSuffix: An Azure Communication Services concept document
description: Proporciona información general sobre Calling SDK.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 7173c625ef56ea9e4d4d43d90adc18ba4fe167ab
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113109294"
---
# <a name="calling-sdk-overview"></a>Información general de Calling SDK

[!INCLUDE [SDP Plan B Deprecation Notice](../../includes/plan-b-sdp-deprecation.md)]

Calling SDK permite que los dispositivos de usuario final impulsen experiencias de comunicación por voz y vídeo. En esta página se proporcionan descripciones detalladas de las características de llamadas, incluida información sobre la compatibilidad del explorador y la plataforma. Para empezar de inmediato, consulte los [inicios rápidos de llamada](../../quickstarts/voice-video-calling/getting-started-with-calling.md) o el [ejemplo de elementos principales de llamada](../../samples/calling-hero-sample.md). 

Una vez que haya iniciado el desarrollo, consulte la [página de problemas conocidos](../known-issues.md) para identificar los errores en los que estamos trabajando.

Principales características de Calling SDK:

- **Direccionamiento** : Azure Communication Services proporciona [identidades](../identity-model.md) genéricas que se usan para dirigirse a puntos de conexión de comunicación. Los clientes usan estas identidades para autenticarse en el servicio y comunicarse entre sí. Estas identidades se usan en las API de llamada que proporcionan a los clientes visibilidad de quién está conectado a una llamada (la lista).
- **Cifrado** : la instancia de Calling SDK que hace la llamada cifra el tráfico y evita manipulaciones de la conexión. 
- **Administración de dispositivos y medios**: la instancia de Calling SDK proporciona funciones para enlazar con dispositivos de audio y vídeo, codifica contenido para una transmisión eficaz a través del plan de TI de comunicaciones y muestra el contenido en los dispositivos de salida y las vistas que especifique. Las API también se proporcionan con fines de uso compartido de la aplicación y la pantalla.
- **RTC**: la instancia de Calling SDK puede recibir e iniciar llamadas de voz con el sistema tradicional de telefonía pública conmutada, [mediante el uso de números de teléfono que haya obtenido en Azure Portal](../../quickstarts/telephony-sms/get-phone-number.md) o mediante programación.
- **Reuniones de Teams**: la instancia de Calling SDK puede [unirse a reuniones de Teams](../../quickstarts/voice-video-calling/get-started-teams-interop.md) e interactuar con el plano de datos de voz y vídeo de Teams. 
- **Notificaciones** : la instancia de Calling SDK proporciona API que permiten a los clientes recibir notificaciones de una llamada entrante. En los casos en que la aplicación no se esté ejecutando en primer plano, hay patrones disponibles para [activar notificaciones emergentes](../notifications.md) e informar a los usuarios finales de una llamada entrante. 

## <a name="detailed-capabilities"></a>Funcionalidad habilitada 

En la lista siguiente se presenta el conjunto de características que están disponibles actualmente en las instancias de Calling SDK de Azure Communication Services.


| Grupo de características | Capacidad                                                                                                          | JS  | Windows | Java (Android) | Objective-C (iOS) |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------- | ----------------- |
| Funcionalidades principales | Realizar una llamada uno a uno entre dos usuarios                                                                           | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Realizar una llamada de grupo con más de dos usuarios (hasta 350)                                                       | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Promocionar una llamada uno a uno con dos usuarios a una llamada de grupo con más de dos usuarios                                 | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Unirse a una llamada de grupo después de que se haya iniciado                                                                              | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Invitar a otro participante de VoIP a unirse a una llamada de grupo en curso                                                       | ✔️   | ✔️       | ✔️              | ✔️                 |
| Control durante la llamada  | Activar o desactivar el vídeo                                                                                              | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Desactivar/activar audio del micrófono                                                                                                     | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Cambiar entre las cámaras                                                                                              | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Retención/reanudación local                                                                                                  | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Altavoz activo                                                                                                      | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Elegir el altavoz para llamadas                                                                                            | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Elegir el micrófono para llamadas                                                                                         | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Mostrar el estado de un participante<br/>*Inactivo, elementos multimedia iniciales, conectando, conectado, en espera, en la sala de espera, desconectado*         | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Mostrar el estado de una llamada<br/>*Elementos multimedia iniciales, entrante, conectando, llamando, conectada, en espera, desconectando, desconectada* | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Mostrar si un participante está silenciado                                                                                      | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Mostrar el motivo por el que un participante abandonó una llamada                                                                       | ✔️   | ✔️       | ✔️              | ✔️                 |
| Uso compartido de la pantalla    | Compartir la pantalla completa desde la aplicación                                                                 | ✔️   | ❌       | ❌              | ❌                 |
|                   | Compartir una aplicación específica (desde la lista de aplicaciones en ejecución)                                                | ✔️   | ❌       | ❌              | ❌                 |
|                   | Compartir una pestaña del explorador web desde la lista de pestañas abiertas                                                                  | ✔️   | ❌       | ❌              | ❌                 |
|                   | El participante puede ver el uso compartido de pantalla remota                                                                            | ✔️   | ✔️       | ✔️              | ✔️                 |
| Lista            | Enumerar participantes                                                                                                   | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Quitar un participante                                                                                                | ✔️   | ✔️       | ✔️              | ✔️                 |
| RTC              | Realizar una llamada uno a uno con un participante de RTC                                                                     | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Realizar una llamada de grupo con participantes de RTC                                                                           | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Promocionar una llamada uno a uno con un participante de RTC a una llamada de grupo                                                 | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Llamada saliente de una llamada de grupo como participante de RTC                                                                    | ✔️   | ✔️       | ✔️              | ✔️                 |
| General           | Probar el micrófono, el altavoz y la cámara con un servicio de prueba de audio (disponible llamando a 8:echo123).                   | ✔️   | ✔️       | ✔️              | ✔️                 |
| Administración del dispositivo | Solicitar permiso para usar audio y/o vídeo                                                                       | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Obtener la lista de cámaras                                                                                                     | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Establecer cámara                                                                                                          | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Obtener la cámara seleccionada                                                                                                 | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Obtener la lista de micrófonos                                                                                                 | ✔️   | ✔️       | ❌              | ❌                 |
|                   | Establecer micrófono                                                                                                      | ✔️   | ✔️       | ❌              | ❌                 |
|                   | Obtener el micrófono seleccionado                                                                                             | ✔️   | ✔️       | ❌              | ❌                 |
|                   | Obtener la lista de altavoces                                                                                                   | ✔️   | ✔️       | ❌              | ❌                 |
|                   | Establecer el altavoz                                                                                                         | ✔️   | ✔️       | ❌              | ❌                 |
|                   | Obtener el altavoz seleccionado                                                                                                | ✔️   | ✔️       | ❌              | ❌                 |
| Representación de vídeo   | Representar un único vídeo en muchos lugares (cámara local o flujo remoto)                                                  | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Establecer y actualizar el modo de escalado                                                                                           | ✔️   | ✔️       | ✔️              | ✔️                 |
|                   | Representar secuencias de vídeo remoto                                                                                          | ✔️   | ✔️       | ✔️              | ✔️                 |


## <a name="calling-sdk-streaming-support"></a>Compatibilidad con streaming de Calling SDK
Calling SDK de Communication Services admite las siguientes configuraciones de streaming:

| Límite                                                         | Web                         | Windows/Android/iOS        |
| ------------------------------------------------------------- | --------------------------- | -------------------------- |
| **Número de secuencias salientes que se pueden enviar simultáneamente**     | 1 vídeo o 1 uso compartido de pantalla | 1 vídeo + 1 uso compartido de pantalla |
| **número de secuencias entrantes que se pueden representar simultáneamente** | 1 vídeo o 1 uso compartido de pantalla | 6 vídeo + 1 uso compartido de pantalla |

## <a name="calling-sdk-timeouts"></a>Tiempos de espera de Calling SDK

Los siguientes tiempos de espera se aplican a las instancias de Calling SDK de Communication Services:

| Acción                                                                      | Tiempo de espera en segundos |
| --------------------------------------------------------------------------- | ------------------ |
| Participante de reconexión/eliminación                                               | 120                |
| Agregar o quitar una nueva modalidad de una llamada (iniciar/detener el uso compartido de la pantalla o un vídeo) | 40                 |
| Tiempo de espera de la operación de transferencia de llamadas                                             | 60                 |
| Tiempo de espera del establecimiento de llamadas entre dos personas                                              | 85                 |
| Tiempo de espera del establecimiento de llamadas de grupo                                            | 85                 |
| Tiempo de espera del establecimiento de llamadas RTC                                             | 115                |
| Tiempo de espera de promoción de una llamada entre dos personas para que sea una llamada de grupo                                    | 115                |

## <a name="javascript-calling-sdk-support-by-os-and-browser"></a>Compatibilidad de Calling SDK de JavaScript según el sistema operativo y el explorador

En la tabla siguiente se representa el conjunto de exploradores compatibles que están disponibles actualmente. Se admiten las tres versiones más recientes del explorador, a menos que se indique lo contrario.

| Plataforma     | Chrome | Safari | Edge (Chromium) | Notas                                                                                                                                                                                                       |
| ------------ | ------ | ------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Android      | ✔️      | ❌      | ❌               | No se admite el uso compartido de pantalla saliente.                                                                                                                                                                   |
| iOS          | ❌      | ✔️      | ❌               | Las aplicaciones iOS en Safari no pueden enumerar ni seleccionar dispositivos de micrófono y altavoz (por ejemplo, Bluetooth); se trata de una limitación del sistema operativo y siempre hay un único dispositivo. No se admite el uso compartido de pantalla saliente. |
| macOS        | ✔️      | ✔️      | ❌               | Son necesarios Safari 14+ y macOS 11+ para la compatibilidad con vídeo saliente.                                                                                                                                                     |
| Windows      | ✔️      | ❌      | ✔️               |                                                                                                                                                                                                             |
| Ubuntu/Linux | ✔️      | ❌      | ❌               |                                                                                                                                                                                                             |

* Se admiten en las versiones de Safari 13.1 +, las llamadas uno a uno no se admiten en Safari.
* Salvo que se especifique lo contrario, se admiten las tres últimas versiones de cada explorador.

## <a name="android-calling-sdk-support"></a>Compatibilidad con el SDK de llamada de Android

* Compatibilidad con Android API Nivel 21 o superior

* Compatibilidad con Java 7 o posterior

* Compatibilidad con Android Studio 2.0

## <a name="ios-calling-sdk-support"></a>Compatibilidad con el SDK de llamada de iOS

* Compatibilidad con iOS 10.0+ en tiempo de compilación y iOS 12.0+ en tiempo de ejecución

* Xcode 12.0+

## <a name="calling-client---browser-security-model"></a>Cliente que llama: modelo de seguridad del explorador

### <a name="user-webrtc-over-https"></a>WebRTC de usuario sobre HTTPS

Las API de WebRTC, como `getUserMedia`, requieren que la aplicación que llama a estas API se atienda a través de HTTPS.

Para el desarrollo local, puede usar `http://localhost`.

### <a name="embed-the-communication-services-calling-sdk-in-an-iframe"></a>Inserción de Calling SDK de Communication Services en un iframe

Una nueva [directiva de permisos (también denominado directiva de características)](https://www.w3.org/TR/permissions-policy-1/#iframe-allow-attribute) se está adoptando en diversos exploradores. Esta directiva afecta a los escenarios de llamada al controlar cómo las aplicaciones pueden acceder a la cámara y el micrófono de un dispositivo mediante un elemento iframe entre orígenes.

Si quiere usar un iframe para hospedar parte de la aplicación desde un dominio diferente, debe agregar el atributo `allow` con el valor correcto a su iframe.

Por ejemplo, este iframe permite el acceso a la cámara y el micrófono:

```html
<iframe allow="camera *; microphone *">
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las llamadas](../../quickstarts/voice-video-calling/getting-started-with-calling.md)

Para más información, consulte los siguientes artículos.
- Familiarización con los [flujos de llamada](../call-flows.md) generales
- Más información sobre los [tipos de llamada](../voice-video-calling/about-call-types.md)
- [Planeación de una solución RTC](../telephony-sms/plan-solution.md)
