---
title: 'Inicio rápido: Configuración de la plataforma para usar Java en Windows, Linux o macOS con el SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar Java en Windows, Linux o macOS con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.custom: devx-track-java
ms.author: erhopf
ms.openlocfilehash: d5780c6ad924e8a6517837dff23e3a4e7ae4d0e1
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111546480"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para Java 8 JRE de 64 bits. Si desea simplemente que el nombre del paquete comience por su cuenta, el SDK de Java no está disponible en el repositorio central de Maven. Si usa Gradle o un archivo de dependencia `pom.xml`, debe agregar un repositorio personalizado que apunte a `https://csspeechstorage.blob.core.windows.net/maven/` (consulte la siguiente información sobre el nombre del paquete).

> [!NOTE]
> Para el SDK de dispositivos de Voz y el dispositivo Roobo, consulte [SDK de dispositivos de voz](~/articles/cognitive-services/speech-service/speech-devices-sdk.md).

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

- El paquete del SDK de Voz para Java está disponible para estos sistemas operativos:
  - Windows: solo 64 bits
  - Mac: macOS X versión 10.13 o posterior
  - Linux: consulte la lista de [distribuciones y arquitecturas de destino de Linux admitidas](~/articles/cognitive-services/speech-service/speech-sdk.md).

## <a name="prerequisites"></a>Prerrequisitos

- [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) o [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

- [IDE de Java para Eclipse](https://www.eclipse.org/downloads/) (es necesario tener ya instalado Java)

- En Windows, necesita [Microsoft Visual C++ Redistributable para Visual Studio 2019](https://support.microsoft.com/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0) para su plataforma. Durante la primera instalación es posible que deba reiniciar.

- En Linux, consulte los [requisitos del sistema y las instrucciones de configuración](~/articles/cognitive-services/speech-service/speech-sdk.md#get-the-speech-sdk).

## <a name="gradle-config"></a>Configuración de Gradle

Las configuraciones de Gradle requieren un repositorio personalizado y una referencia explícita a la extensión `.jar` de las dependencias. 

```groovy
// build.gradle

repositories {
    maven {
        url "https://csspeechstorage.blob.core.windows.net/maven/"
    }
}

dependencies {
    implementation group: 'com.microsoft.cognitiveservices.speech', name: 'client-sdk', version: "1.17.0", ext: "jar"
}
```

## <a name="create-an-eclipse-project-and-install-the-speech-sdk"></a>Creación de un proyecto de Eclipse e instalación del SDK de Voz

[!INCLUDE [](~/includes/cognitive-services-speech-service-quickstart-java-create-proj.md)]

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]
