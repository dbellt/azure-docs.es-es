---
title: 'Inicio rápido: Configuración de la plataforma para usar C# en Unity con el SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar C# en Unity con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 95b0c127d005efe95fa2b7fdb58c69602a98b217
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165121"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para [Unity](https://unity3d.com/).

> [!NOTE]
> El SDK de Voz para Unity es compatible con Windows Desktop (x86 y x64) o con la Plataforma universal de Windows (x86, x64, ARM/ARM64), Android (x86, ARM32/64), iOS (simulador de x64 y ARM64) y Mac (x64).

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="prerequisites"></a>Requisitos previos

Esta guía de inicio rápido requiere:

- En Windows, necesita [Microsoft Visual C++ Redistributable para Visual Studio 2019](https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0) para su plataforma. Durante la primera instalación es posible que deba reiniciar.
- [Unity 2018.3 o posterior](https://store.unity.com/) con [Unity 2019.1 que agrega compatibilidad con UWP ARM64](https://blogs.unity3d.com/2019/04/16/introducing-unity-2019-1/#universal).
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/). La versión 15.9 o posterior de Visual Studio 2017 también es aceptable.
- Para obtener compatibilidad con Windows ARM64, instale las [herramientas de compilación opcionales para ARM64 y el SDK de Windows 10 para ARM64](https://blogs.windows.com/buildingapps/2018/11/15/official-support-for-windows-10-on-arm-development/).
- En Android, necesita un dispositivo Android basado en ARM (API 23: Android 6.0 Marshmallow o posterior) habilitado para el desarrollo con un micrófono que funcione.
- En iOS, necesita un dispositivo iOS (ARM64) habilitado para el desarrollo con un micrófono que funcione.
- En macOS, necesita un dispositivo Mac (x64) y la versión LTS más reciente de Unity 2019 (o superior) para la compatibilidad integrada con acceso al micrófono en la configuración del reproductor de Unity.

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

Para instalar Speech SDK para Unity, siga estos pasos:

1. Descargue y abra el [SDK de Voz para Unity](https://aka.ms/csspeech/unitypackage), que se incluye como un paquete de recursos de Unity (.unitypackage) y que ya debe estar asociado con Unity. Cuando se abra el paquete de recursos, aparecerá el cuadro de diálogo **Import Unity Package** (Importar paquete de Unity). Es posible que tenga que crear y abrir un proyecto vacío para que funcione este paso.

   [![Cuadro de diálogo Import Unity Package en el editor de Unity](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-unity-01-import.png)](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-unity-01-import.png#lightbox)

1. Asegúrese de que se seleccionan todos los archivos y seleccione **Import** (Importar). Transcurridos unos instantes, el paquete de recursos de Unity se importa en el proyecto.

Para más información sobre la importación de paquetes de recursos en Unity, consulte la [documentación de Unity](https://docs.unity3d.com/Manual/AssetPackages.html).

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]
