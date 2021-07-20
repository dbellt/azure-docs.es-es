---
title: ¿Qué es Azure Communication Services?
description: Obtenga información sobre cómo Azure Communication Services le ayuda a desarrollar experiencias de usuario enriquecidas con comunicaciones en tiempo real.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: da099fe09b44a8c7aeecdfd7118e936ce3f0ff91
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113113074"
---
# <a name="what-is-azure-communication-services"></a>¿Qué es Azure Communication Services?

Azure Communication Services son servicios basados en la nube con API REST y SDK de biblioteca cliente disponibles para ayudarle a integrar la comunicación en sus aplicaciones. Puede agregar características de comunicación a las aplicaciones sin ser un experto en tecnologías de comunicación, como la codificación multimedia y las redes en tiempo real. Azure Communication Services admite varios formatos de comunicación:

1. Llamadas de voz y vídeollamadas
1. Chat de texto enriquecido
1. sms

Las aplicaciones tanto de llamadas de voz como de vídeollamadas pueden interactuar con la red de telefonía conmutada pública (RTC). Puede adquirir números de teléfono directamente mediante las API REST y los SDK de Azure Communication Services o Azure Portal. El enrutamiento directo de Azure Communication Services le permite usar SIP y los controladores de perímetro de sesión para conectar sus propios operadores de RTC y traer sus propios números de teléfono.

Además de las API REST, las [bibliotecas cliente de Azure Communication Services](./concepts/sdk-options.md) están disponibles para varias plataformas y lenguajes, incluidos exploradores web (JavaScript), iOS (Swift), Java (Android) y Windows (.NET). Azure Communication Services no depende de identidades y usted controla cómo se identifican y autentican los usuarios finales.

Entre los escenarios para Azure Communication Services se incluyen:

- **Negocio a consumidor (B2C).** Los empleados y servicios de una empresa interactúan con los consumidores mediante voz, vídeo y chat de texto enriquecido en un explorador personalizado o una aplicación móvil. Una organización puede enviar y recibir mensajes SMS, u [operar un sistema de respuesta de voz interactivo (IVR)](https://github.com/microsoft/botframework-telephony/blob/main/EnableTelephony.md) con un número de teléfono que adquiera mediante Azure. La [integración con Microsoft Teams](./quickstarts/voice-video-calling/get-started-teams-interop.md) se puede usar para conectar a los consumidores a reuniones de Teams hospedadas por empleados. Esto resulta idóneo para escenarios remotos de asistencia sanitaria, banca y soporte técnico de productos en los que es posible que los empleados ya conozcan Teams.
- **Consumidor a consumidor (C2C).** Cree espacios sociales atractivos para la interacción de consumidor a consumidor con voz, vídeo y chat de texto enriquecido. Se puede crear cualquier tipo de interfaz de usuario con los SDK de Azure Communication Services, o bien puede utilizar los ejemplos de aplicaciones completos y el kit de herramientas de interfaz de usuario para ayudarle a empezar a trabajar rápidamente.

Para más información, consulte nuestro [vídeo de Microsoft Mechanics](https://www.youtube.com/watch?v=apBX7ASurgM) o los recursos vinculados a continuación.

## <a name="common-scenarios"></a>Escenarios frecuentes

<br>

| Resource                               |Descripción                           |
|---                                    |---                                   |
|**[Creación de un recurso de Communication Services](./quickstarts/create-communication-resource.md)**|Empiece a utilizar Azure Communication Services mediante Azure Portal o el SDK de Communication Services para aprovisionar su primer recurso de Communication Services. Una vez que tenga la cadena de conexión del recurso de Communication Services, puede aprovisionar los primeros tokens de acceso de usuario.|
|**[Obtención de un número de teléfono](./quickstarts/telephony-sms/get-phone-number.md)**|Use Azure Communication Services para aprovisionar y liberar números de teléfono. Estos números de teléfono se pueden usar para iniciar o recibir llamadas de teléfono y crear soluciones de SMS.|
|**[Envío de un SMS desde la aplicación](./quickstarts/telephony-sms/send.md)**| Las API y los SDK de SMS de Azure Communication Services se utilizan para enviar y recibir mensajes SMS desde las aplicaciones de servicio.|

Después de crear un recurso de Communication Services, puede empezar a crear escenarios de cliente, como las llamadas de voz y vídeo o el chat de texto:

| Resource                               |Descripción                           |
|---                                    |---                                   |
|**[Creación del primer token de acceso de usuario](./quickstarts/access-tokens.md)**|Los tokens de acceso de usuario autentican los clientes en el recurso de Azure Communication Services. Estos tokens se aprovisionan y se vuelven a emitir mediante las API y los SDK de identidad de Communication Services.|
|**[Introducción a las llamadas de voz y vídeo](./quickstarts/voice-video-calling/getting-started-with-calling.md)**| Azure Communication Services permite agregar llamadas de voz y vídeo a las aplicaciones nativas o del explorador mediante Calling SDK. |
|**[Adición de llamadas telefónicas a una aplicación](./quickstarts/voice-video-calling/pstn-call.md)**|Con Azure Communication Services puede agregar funcionalidades de llamadas telefónicas a la aplicación.|
|**[Incorporación de una aplicación de llamadas a una reunión de Teams](./quickstarts/voice-video-calling/get-started-teams-interop.md)**|Azure Communication Services se puede usar para crear experiencias de reunión personalizadas con interacción con Microsoft Teams. Los usuarios de sus soluciones de Communication Services pueden interactuar con los participantes de Teams mediante la voz, el vídeo, el chat y el uso compartido de la pantalla.|
|**[Introducción al chat](./quickstarts/chat/get-started.md)**|Chat SDK de Azure Communication Services se usa para agregar chats de texto enriquecido en tiempo real a las aplicaciones.|
|**[Conexión de un bot de Microsoft a un número de teléfono](https://github.com/microsoft/botframework-telephony)**|El canal de telefonía es un canal de Microsoft Bot Framework que permite al bot interactuar con los usuarios por teléfono. Aprovecha la tecnología de Microsoft Bot Framework combinada con Azure Communication Services y los servicios de Voz de Azure.  |


## <a name="samples"></a>Ejemplos

En los siguientes ejemplos se muestra el uso completo de Azure Communication Services. Use estos ejemplos para poner en marcha sus propias soluciones de Communication Services.
<br>

| Nombre del ejemplo                               | Descripción                           |
|---                                    |---                                   |
|**[Muestra de elementos principales de una llamada grupal](./samples/calling-hero-sample.md)**| Descargue un ejemplo de aplicación diseñado para llamadas grupales para exploradores, dispositivos iOS y Android. |
|**[Muestra de elementos principales de un chat grupal](./samples/chat-hero-sample.md)**| Descargue un ejemplo de aplicación diseñado para un chat de texto grupal para exploradores. |
|**[Ejemplo de llamada web](./samples/web-calling-sample.md)**| Descargue un ejemplo de aplicación web diseñado para llamadas de audio, vídeo y RTC. |


## <a name="platforms-and-sdk-libraries"></a>Plataformas y bibliotecas del SDK

Obtenga más información sobre los SDK de Azure Communication Services con los recursos siguientes. Las API REST están disponibles para la mayoría de las funcionalidades si desea crear sus propios clientes o acceder de otro modo al servicio a través de Internet.

| Resource                               | Descripción                           |
|---                                    |---                                   |
|**[Bibliotecas del SDK y API REST](./concepts/sdk-options.md)**|Las funcionalidades de Azure Communication Services están organizadas conceptualmente en seis áreas, cada una representada por un SDK. Puede decidir qué bibliotecas del SDK usar en función de sus necesidades de comunicación en tiempo real.|
|**[Introducción al SDK de llamadas](./concepts/voice-video-calling/calling-sdk-features.md)**|Consulte la información general del SDK de llamadas de Communication Services.|
|**[Introducción al SDK de chat](./concepts/chat/sdk-features.md)**|Consulte la información general del SDK de chat de Communication Services.|
|**[Introducción al SDK de SMS](./concepts/telephony-sms/sdk-features.md)**|Consulte la información general del SDK de SMS de Communication Services.|

## <a name="other-microsoft-communication-services"></a>Otros servicios de comunicación de Microsoft

Hay otros dos productos de comunicación de Microsoft que puede considerar utilizar y que no pueden interactuar directamente con Communication Services en este momento:

 - [Las API de comunicación en la nube de Microsoft Graph](/graph/cloud-communications-concept-overview) permiten a las organizaciones crear experiencias de comunicación asociadas a usuarios de Azure Active Directory con licencias de Microsoft 365. Esto es idóneo para las aplicaciones asociadas a Azure Active Directory o en contextos de ampliación de la experiencia de productividad en Microsoft Teams. También hay API para compilar aplicaciones y personalización dentro de la [experiencia de equipos.](/microsoftteams/platform/?preserve-view=true&view=msteams-client-js-latest)

 - [Azure PlayFab Party](/gaming/playfab/features/multiplayer/networking/) simplifica la incorporación de comunicación de datos y chat de baja latencia a los juegos. Aunque puede potenciar el chat de juegos y los sistemas de red con Communication Services, PlayFab es una opción personalizada y gratuita de Xbox.


## <a name="next-steps"></a>Pasos siguientes

 - [Creación de un recurso de Communication Services](./quickstarts/create-communication-resource.md)
