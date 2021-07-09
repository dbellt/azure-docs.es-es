---
title: 'Asistentes de voz en Windows: introducción'
titleSuffix: Azure Cognitive Services
description: Información general de los asistentes de voz en Windows, incluidas las funcionalidades y los recursos de desarrollo disponibles.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: 989e4e48b03f976a647a542771ade053d9229ec7
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110058913"
---
# <a name="voice-assistants-on-windows"></a>Asistentes de voz en Windows

En Windows 10, versión 2004 y posteriores, las aplicaciones del asistente de voz pueden aprovechar las API ConversationalAgent de Windows para lograr una experiencia del asistente habilitado para voz completa.

## <a name="voice-assistant-features"></a>Características del asistente de voz

Las aplicaciones de agente de voz se pueden activar pronunciando una palabra clave para habilitar una experiencia con manos libres controlada con la voz. La activación de voz funciona cuando se cierra la aplicación y cuando se bloquea la pantalla.

Además, Windows proporciona un conjunto de opciones de privacidad de activación de voz que permite a los usuarios controlar la activación de voz y la activación del bloqueo de cada aplicación.

Tras la activación de voz, Windows administrará varios agentes activos correctamente y notificará a cada asistente de voz en caso de interrupción o desactivación. Esto permite que las aplicaciones administren correctamente las interrupciones y otros eventos entre agentes.

## <a name="how-does-voice-activation-work"></a>¿Cómo funciona la activación por voz?

El tiempo de ejecución de activación del agente (AAR) es el proceso continuo de Windows que administra la activación de la aplicación al pronunciar palabra clave o pulsar un botón. Se inicia con Windows siempre que exista al menos una aplicación en el sistema que se haya registrado con el sistema. Las aplicaciones interactúan con AAR a través de las API ConversationalAgent de Windows SDK.

Cuando el usuario pronuncia una palabra clave, el observador de palabra clave de software o hardware observador del sistema notifica a AAR que se ha detectado una palabra clave y proporciona un identificador de palabra clave. AAR, a su vez, envía una solicitud a BackgroundService para iniciar la aplicación con el identificador de la aplicación correspondiente.

### <a name="registration"></a>Registro

La primera vez que se ejecuta una aplicación activada por voz, se registra su identificador de la aplicación y la información de palabras clave a través de las API ConversationalAgent. AAR registra todas las configuraciones de la asignación global con el observador de palabra clave de hardware o software en el sistema, lo que permite detectar la palabra clave de la aplicación. La aplicación también [se registra con el servicio en segundo plano](/windows/uwp/launch-resume/register-a-background-task).

Tenga en cuenta que esto significa que una aplicación no se puede activar por voz hasta que se ha ejecutado una vez y se ha permitido completar el registro.

### <a name="receiving-an-activation"></a>Recepción de una activación

Al recibir la solicitud de AAR, el servicio en segundo plano inicia la aplicación. La aplicación recibe una señal a través del método de ciclo de vida de OnBackgroundActivated en `App.xaml.cs` con un argumento de evento único. Este argumento indica a la aplicación que se activó mediante AAR y que debe iniciar la comprobación de palabra clave.

Si la aplicación comprueba correctamente la palabra clave, puede hacer que aparezca una solicitud en primer plano. Si esta solicitud se realiza correctamente, la aplicación muestra la interfaz de usuario y continúa su interacción con el usuario.

AAR sigue señalando las aplicaciones activas cuando se pronuncia su palabra clave. Sin embargo, en lugar de enviar una señal a través del método de ciclo de vida de `App.xaml.cs`, lo hace a través de un evento de las API ConversationalAgent.

### <a name="keyword-verification"></a>Comprobación de la palabra clave

El observador de palabra clave que desencadena el inicio de la aplicación logra un bajo consumo de energía al simplificar el modelo de palabra clave. Esto permite que el observador de palabra clave esté "siempre activo" sin un alto impacto en el consumo de energía, pero también significa que el observador de palabra clave probablemente tendrá un gran número de "aceptaciones falsas", en que detectará una palabra clave aunque no se haya pronunciado ninguna. Este es el motivo por el que el sistema de activación por voz inicia la aplicación en segundo plano: para dar a la aplicación la oportunidad de comprobar que se ha hablado de la palabra clave antes de interrumpir la sesión actual del usuario. AAR guarda el audio desde unos segundos antes de que se detecte la palabra clave y hace que sea accesible para la aplicación. La aplicación puede utilizarlo para ejecutar un observador de palabra clave más confiable en el mismo audio.

## <a name="next-steps"></a>Pasos siguientes

- **Revisar las directrices de diseño:** Nuestras [instrucciones de diseño](windows-voice-assistants-best-practices.md) diseñan el trabajo clave necesario para proporcionar las mejores experiencias posibles para la activación por voz en Windows 10.
- **Visitar la página Introducción:** Comience [aquí](how-to-windows-voice-assistants-get-started.md) para conocer los pasos para empezar a implementar asistentes de voz en Windows, desde el establecimiento del entorno de desarrollo hasta una introducción a la guía de implementación.
- **Probar la aplicación de ejemplo**: Para disfrutar de estas funcionalidades de primera mano, visite la página del [ejemplo del asistente de voz de UWP](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample) y siga los pasos para que se ejecute el cliente de ejemplo.