---
title: Prueba de una aplicación de Comandos personalizados
titleSuffix: Azure Cognitive Services
description: En este artículo, aprenderá distintos enfoques para probar una aplicación de Comandos personalizados.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.custom: devx-track-csharp
ms.openlocfilehash: d97ab241120ddc4c8e7434591b866312ea7708ae
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111890121"
---
# <a name="test-your-custom-commands-application"></a>Prueba de una aplicación de Comandos personalizados

En este artículo, aprenderá distintos enfoques para probar una aplicación de Comandos personalizados.

## <a name="test-in-the-portal"></a>Prueba en el portal

La prueba en el portal es la forma más sencilla y rápida de comprobar si la aplicación de comandos personalizada funciona según lo previsto. Una vez que la aplicación se haya entrenado correctamente, haga clic en el botón `Test` para iniciar la prueba.

> [!div class="mx-imgBorder"]
> ![Prueba en el portal](media/custom-commands/create-basic-test-chat.png)

## <a name="test-with-windows-voice-assistant-client"></a>Prueba con el cliente del asistente de voz de Windows

El cliente del asistente de voz de Windows es una aplicación Windows Presentation Foundation (WPF) de C# que facilita probar las interacciones con el bot antes de crear una aplicación cliente personalizada.

La herramienta puede aceptar un id. de aplicación de comando personalizado. Permite probar los escenarios de finalización de tareas o de comando y control hospedados en el servicio de Comandos personalizados.

Para configurar el cliente, extraiga el [cliente del asistente de voz de Windows](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/clients/csharp-wpf).

> [!div class="mx-imgBorder"]
> ![Creación de perfil de WVAC](media/custom-commands/conversation.png)

## <a name="test-programatically-with-the-cognitive-services-voice-assistant-test-tool"></a>Prueba mediante programación con la herramienta de prueba del Asistente para voz de Cognitive Services

La herramienta de prueba del asistente para voz es una aplicación de consola de C# para .NET Core, configurable, que permite realizar pruebas de regresión funcional de un extremo a otro para el Asistente para voz de Microsoft. 

La herramienta se puede ejecutar manualmente como un comando de consola o de manera automatizada como parte de una canalización de CI/CD de Azure DevOps para evitar regresiones en el bot.

Para aprender a configurar la herramienta, consulte [Herramienta de prueba del asistente para voz](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/main/clients/csharp-dotnet-core/voice-assistant-test).

## <a name="test-with-speech-sdk-enabled-client-applications"></a>Prueba con aplicaciones cliente habilitadas para el SDK de Voz

El kit de desarrollo de software (SDK) de voz expone muchas de las funcionalidades del servicio de voz, lo que permite desarrollar aplicaciones habilitadas para la voz. Está disponible en muchos lenguajes de programación en la mayoría de las plataformas.

Para configurar una aplicación cliente de Plataforma universal de Windows (UWP) con el SDK de Voz, e integrarla con la aplicación de comandos personalizada:  
- [Cómo: Integración con una aplicación cliente mediante el SDK de Voz](./how-to-custom-commands-setup-speech-sdk.md)
- [Cómo: Envío de actividad a una aplicación cliente](./how-to-custom-commands-send-activity-to-client.md)
- [Cómo: Configuración de puntos de conexión web](./how-to-custom-commands-setup-web-endpoints.md)

Para otros lenguajes de programación y plataformas:
- [Lenguajes de programación, plataformas y capacidades de escenario del SDK de Voz](./speech-sdk.md)
- [Ejemplos de código del asistente de voz](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Consulte los ejemplos en GitHub](https://aka.ms/speech/cc-samples)
