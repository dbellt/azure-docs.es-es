---
title: 'Inicio rápido: Configuración de la plataforma para usar C# en Xamarin con el SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar C# en Xamarin con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 0f895744ed589e32c86348b17ddc57b5820b14fc
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165922"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para [Xamarin](/xamarin/get-started/what-is-xamarin), una plataforma de código abierto para compilar aplicaciones modernas y de alto rendimiento para iOS, Android y Windows con .NET. Si desea simplemente que el nombre del paquete comience por su cuenta, ejecute `Install-Package Microsoft.CognitiveServices.Speech` en la consola de NuGet.

> [!NOTE]
> El SDK de Voz para Xamarin es compatible con Windows Desktop (x86 y x64) o con la Plataforma universal de Windows (x86, x64, ARM/ARM64), Android (x86, ARM32/64) e iOS (simulador de x64 y ARM64).

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="prerequisites"></a>Requisitos previos

Esta guía de inicio rápido requiere:

* En Windows, necesita [Microsoft Visual C++ Redistributable para Visual Studio 2019](https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0) para su plataforma. Durante la primera instalación es posible que deba reiniciar.
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)

## <a name="create-a-visual-studio-project-and-install-the-speech-sdk"></a>Creación de un proyecto de Visual Studio e instalación del SDK de Voz

[!INCLUDE [](~/includes/cognitive-services-speech-service-quickstart-xamarin-create-proj.md)]

El SDK de Voz ya está instalado. Ahora puede eliminar o volver a usar el proyecto "helloworld" que creó en los pasos anteriores.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]
