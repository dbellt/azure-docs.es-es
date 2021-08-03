---
title: Introducción a la biblioteca de interfaz de usuario
titleSuffix: An Azure Communication Services concept document
description: Conozca la Biblioteca de interfaz de usuario de Azure Communication Services.
author: ddematheu2
manager: chrispalm
services: azure-communication-services
ms.author: dademath
ms.date: 05/11/2021
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 4176bdf61cafdf50aef71e76b832a0e669f1535c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468810"
---
# <a name="ui-library-overview"></a>Introducción a la Biblioteca de interfaz de usuario

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

> [!NOTE]
> Puede encontrar documentación detallada sobre la Biblioteca de interfaz de usuario en el [ libro de historias sobre la Biblioteca de interfaz de usuario ](https://azure.github.io/communication-ui-library). Ahí encontrará documentación conceptual adicional, inicios rápidos y ejemplos.

Azure Communication Services: la Biblioteca de interfaz de usuario facilita la creación de experiencias de usuario de comunicación modernas mediante Azure Communication Services. Proporciona una biblioteca de componentes de interfaz de usuario preparados para la producción que puede colocar en sus aplicaciones:

- **Composiciones**: estos componentes son soluciones inmediatas que implementan escenarios de comunicación comunes.
  Puede agregar rápidamente experiencias de chat o videollamadas a las aplicaciones.
  Las composiciones son componentes de orden superior de código abierto creadas mediante componentes de interfaz de usuario.

- **Componentes de la interfaz de usuario**: estos componentes son bloques de creación de código abierto que permiten crear una experiencia de comunicación personalizada.
  Los componentes se ofrecen para las funcionalidades de llamadas y de chat que se pueden combinar para generar experiencias.

Todas estas bibliotecas cliente de interfaz de usuario utilizan los recursos y el [lenguaje de diseño de Fluent de Microsoft](https://developer.microsoft.com/fluentui/). La interfaz de usuario de Fluent proporciona una capa fundamental para la Biblioteca de interfaz de usuario y se usa activamente en todos los productos de Microsoft.

Junto con los componentes de la interfaz de usuario, la Biblioteca de interfaz de usuario expone una biblioteca cliente con estado para realizar llamadas y conversar.
Este cliente es independiente de cualquier marco de administración de estados específico y se puede integrar con administradores de estado comunes, como Redux o React Context.
Esta biblioteca cliente con estado se puede usar con los componentes de la interfaz de usuario para pasar propiedades y métodos para que los componentes de la interfaz de usuario representen datos. Para más información, consulte [Introducción a los clientes sin estado](https://azure.github.io/communication-ui-library/?path=/story/stateful-client-what-is-stateful--page).

## <a name="installing-ui-library"></a>Instalación de la Biblioteca de interfaz de usuario

Los clientes con estado se encuentran como parte del paquete `@azure/communication-react`. 

```bash
npm i --save @azure/communication-react
```

## <a name="composites-overview"></a>Introducción a las composiciones

Las composiciones son componentes de nivel superior formados por componentes de interfaz de usuario que ofrecen soluciones inmediatas para escenarios de comunicación comunes mediante Azure Communication Services.
Los desarrolladores pueden crear fácilmente instancias de la composición mediante un token de acceso de Azure Communication Services y la configuración necesaria atribuida para la llamada o el chat.

| Compuesto    | Casos de uso  | 
| ------------ | ---------- |
| [CallComposite](https://azure.github.io/communication-ui-library/?path=/docs/composites-callcomposite--basic-example) | Experiencia de llamada que permite a los usuarios iniciar una llamada o unirse a ella. Dentro de la experiencia, los usuarios pueden configurar sus dispositivos, participar en la llamada con vídeo y ver a otros participantes, incluidos los participantes con el vídeo encendido. La interoperabilidad de Teams se consigue con la funcionalidad de sala de espera para que el usuario espere a ser admitido. |
| [ChatComposite](https://azure.github.io/communication-ui-library/?path=/docs/composites-chatcomposite--basic-example)    | Experiencia de chat en la que el usuario puede enviar y recibir mensajes. Los eventos de subproceso, como escribir, leer o la entrada y salida de los participantes, se muestran al usuario como parte del subproceso de chat.                                                                                                                          |

## <a name="ui-component-overview"></a>Introducción a los componentes de interfaz de usuario

Componentes de interfaz de usuario puros que pueden usar los desarrolladores para crear experiencias de comunicación, desde unir iconos de vídeo en una cuadrícula para mostrar los participantes remotos hasta organizar componentes para ajustarse a las especificaciones de las aplicaciones.
Los componentes de interfaz de usuario admiten personalización para proporcionarles el aspecto y la sensación adecuados que coincidan con la personalización de marca y el estilo de las aplicaciones.

| Área    | Componente    | Descripción       |
| ------- | ------------ | ----------------- |
| Llamar | [Diseño de cuadrícula](https://azure.github.io/communication-ui-library/?path=/story/ui-components-gridlayout--grid-layout-component)                | Componente de cuadrícula para organizar los iconos de vídeo en una cuadrícula NxN.                                            |
|         | [Icono de vídeo](https://azure.github.io/communication-ui-library/?path=/story/ui-components-videotile--video-tile-component)                   | Componente que muestra la secuencia de vídeo cuando está disponible y un componente estático predeterminado cuando no lo está.        |
|         | [Barra de control](https://azure.github.io/communication-ui-library/?path=/story/ui-components-controlbar--control-bar-component)                | Contenedor para organizar DefaultButtons para enlazar con acciones de llamada específicas, como silencio o uso compartido de pantalla. |
|         | [VideoGallery](https://azure.github.io/communication-ui-library/?path=/story/ui-components-video-gallery--video-gallery)                                           | Componente inmediato de la galería de vídeos que cambia dinámicamente a medida que se agregan participantes.               |
| Chat    | [Subproceso de mensaje](https://azure.github.io/communication-ui-library/?path=/story/ui-components-messagethread--message-thread-component)       | Contenedor que representa mensajes de chat, mensajes del sistema y mensajes personalizados.                          |
|         | [Cuadro de envío](https://azure.github.io/communication-ui-library/?path=/story/ui-components-sendbox--send-box-component)                         | Componente de entrada de texto con un botón de envío discreto.                                                   |
|         | [Indicador de estado del mensaje](https://azure.github.io/communication-ui-library/?path=/story/ui-components-message-status-indicator--message-status-indicator)        | Componente de recepción de lectura de varios estados para mostrar el estado del mensaje enviado.                                   |
|         | [Indicador de escritura](https://azure.github.io/communication-ui-library/?path=/story/ui-components-typingindicator--typing-indicator-component) | Componente de texto para representar a los participantes que escriben activamente en un subproceso.                      |
| Comunes  | [Elemento de participante](https://azure.github.io/communication-ui-library/?path=/story/ui-components-participantitem--participant-item-component) | Componente común para representar una llamada o un participante de chat, incluido el avatar y el nombre para mostrar.            |
|         | [Lista de participantes](https://azure.github.io/communication-ui-library/?path=/story/ui-components-participant-list--participant-list)                                 | Componente común para representar una lista de participantes de llamada o chat, incluidos el avatar y el nombre para mostrar       |


## <a name="what-ui-artifact-is-best-for-my-project"></a>¿Qué artefacto de interfaz de usuario es el mejor para mi proyecto?

Comprender estos requisitos le ayudará a elegir la biblioteca cliente adecuada:

- **¿Cuánta personalización desea?** Las bibliotecas cliente principales de Azure Communication no tienen experiencia de usuario y están diseñadas para que pueda compilar cualquier experiencia de usuario que quiera. Los componentes de la Biblioteca de interfaz de usuario proporcionan recursos de interfaz de usuario a cambio de una personalización reducida.
- **¿A qué plataformas quiere dirigirse?** Cada plataforma tiene diferentes funcionalidades.

Se pueden encontrar detalles sobre la disponibilidad de características de la [Biblioteca de interfaz de usuario aquí](https://azure.github.io/communication-ui-library/?path=/story/use-cases--page), pero las ventajas principales se resumen a continuación.

| Biblioteca cliente / SDK  | Complejidad de la implementación | Capacidad de personalización | Llamar | Chat | [Interoperabilidad de equipos](../teams-interop.md) |
| --------------------- | ------------------------- | --------------------- | ------- | ---- | ----------------------------------------------------------------------------------------------------- |
| Componentes compuestos  | Bajo                       | Bajo                   | ✔       | ✔    | ✔                                                                                                     |
| Componentes básicos       | Media                    | Media                | ✔       | ✔    | ✔                                                                                                     |
| Bibliotecas cliente principales | Alto                      | Alto                  | ✔       | ✔    | ✔                                                                                                     |

> [!div class="nextstepaction"]
> [Visite el libro de historias de la Biblioteca de interfaz de usuario](https://azure.github.io/communication-ui-library).
